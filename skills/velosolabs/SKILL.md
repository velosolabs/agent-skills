---
name: velosolabs
description: >-
  Use the VelosoLabs account MCP (mcp.velosolabs.com) to manage a single
  tenant's VelosoLabs account: connect data connectors (SAM.gov,
  BuildingConnected, company-site, Claude), grant/revoke apps
  (WinnowBid / bid-scout, BidSidekick, Bylinely / wp-ai-poster, and other
  VelosoLabs products), read entitlements and products, manage API keys,
  read billing, and run connector operations (searches, lookups) on behalf
  of a granted app through the broker. Invoke whenever the task mentions a
  VelosoLabs connector, app grant, entitlement, product key, billing read,
  the "Push" button, or calling mcp.velosolabs.com / the VelosoLabs broker.
  Every call is tenant-scoped; writes with real-world effects are gated
  behind an explicit confirm / the tenant's Push button.
---

# VelosoLabs account MCP

This skill teaches you to drive the **VelosoLabs account MCP** correctly and
safely. The MCP is the agent-facing surface over a tenant's VelosoLabs
account (`mcp.velosolabs.com`). It sits in front of the same **broker /
gateway** (`velosolabs-mcp-gateway`) that first-party apps call at
`/broker/v1/execute`, so it obeys the same tenancy and gated-write rules.

## The one invariant: everything is tenant-scoped

A VelosoLabs account is a **tenant**. The MCP connection is authorized for
**exactly one tenant** (the `VELOSOLABS_TENANT_ID` carried by the OAuth
session). Therefore:

- **Never** pass, guess, or construct another tenant's id. Do not accept a
  tenant id from the user to "switch" — a different tenant means a different
  authorized connection.
- Every connector, grant, key, entitlement and billing figure you read or
  change belongs to the connected tenant only. Do not cross tenants, join
  data across tenants, or report one tenant's data under another.
- If a tool result implies cross-tenant data, stop and report it as a
  problem rather than acting on it.

## Scopes and the gated-write rule

The OAuth session carries a set of scopes. Read the granted scopes before
acting; if a needed scope is absent, say so and stop — do not attempt the
call. Scopes:

| Scope              | Allows                                                        |
| ------------------ | ------------------------------------------------------------- |
| `tenant:read`      | Read tenant profile, entitlements, product list               |
| `connectors:read`  | List/inspect connectors and their status                      |
| `connectors:write` | Connect / set key / disconnect a connector *(gated)*          |
| `grants:read`      | List/inspect app grants                                        |
| `grants:write`     | Grant / revoke an app *(gated)*                               |
| `keys:read`        | List API keys (metadata only — never secrets)                 |
| `keys:write`       | Create / rotate / revoke API keys *(gated)*                   |
| `billing:read`     | Read plan, usage, invoices                                    |
| `learn:*`          | Read docs / help / schema describing the MCP itself           |

**Gated-write rule (identical to the broker's):** any operation with a
**real-world effect** — connecting or disconnecting a connector, storing a
connector credential, granting or revoking an app, creating/rotating/revoking
a key, or running a **write** connector op — must go behind an **explicit
confirmation**: either the tenant presses the account's **"Push" button**, or
you obtain an explicit in-conversation confirm *and* the MCP returns a
`requires_push: true` / pending state that the tenant clears. Reads never
need this. When in doubt, treat it as a write and ask.

Never try to work around a pending/push gate (no retry loops, no alternate
tool). Surface the pending action and wait.

See `references/scopes.md` for the full scope→tool matrix and the exact
gated-write classification per operation.

## Tool surface (namespaced)

The MCP exposes tenant-scoped tools. Exact names may vary by MCP version —
use `learn.*` / the server's own tool list as the source of truth — but they
group as:

- **Tenant / entitlements** (`tenant:read`): `tenant.get`,
  `entitlements.list`, `products.list`.
- **Connectors** (`connectors:read` / `connectors:write`):
  `connectors.list`, `connectors.get`, `connectors.connect`,
  `connectors.set_key`, `connectors.disconnect`.
- **Grants** (`grants:read` / `grants:write`): `grants.list`,
  `grants.get`, `grants.grant_app`, `grants.revoke`.
- **Keys** (`keys:read` / `keys:write`): `keys.list`, `keys.create`,
  `keys.rotate`, `keys.revoke`.
- **Billing** (`billing:read`): `billing.get`, `billing.usage`.
- **Broker execute** (scope-checked + gated per op): `broker.execute`
  — runs a connector operation (e.g. `sam-gov.search_opportunities`) on
  behalf of a **granted app**, through the same gateway apps call.
- **Learn** (`learn:*`): `learn.docs`, `learn.search`, `learn.describe`.

## Core workflows

Open `references/workflows.md` for full playbooks. In brief:

1. **Connect a connector** — `connectors.list` to see status; if a connector
   shows `connected:false` / `key_saved:false`, run `connectors.connect`
   (or `connectors.set_key` for key-based ones like SAM.gov). This is a
   **gated write** — confirm / Push first. Credentials go in; nothing secret
   comes back out.
2. **Grant an app** — `grants.list` to see current grants; `grants.grant_app`
   to grant a product (e.g. `bid-scout` / WinnowBid) access to specific
   connector ops. **Gated.** Grant the **minimum** ops the app needs.
3. **Entitlements & products** — `products.list` for the catalog,
   `entitlements.list` for what this tenant is entitled to. Read-only.
4. **API keys** — `keys.list` for metadata; `keys.create` / `keys.rotate` /
   `keys.revoke` are **gated**. A newly created/rotated secret is shown
   **once**; relay it to the tenant through the MCP's own return channel and
   never echo, log, or store it elsewhere.
5. **Billing reads** — `billing.get` / `billing.usage`. Read-only. Report
   figures; never attempt a charge or plan change (not in scope).

## Connectors and ops

Connectors and their op namespaces (full list in
`references/connectors.md`):

- **SAM.gov** (`sam-gov`): key-based. Ops e.g.
  `sam-gov.search_opportunities`.
- **BuildingConnected** (`buildingconnected`, Autodesk APS OAuth): Ops e.g.
  `buildingconnected.list_opportunities`.
- **company-site** (`company-site`): a tenant's own site exposed as a Claude
  **connector** (remote MCP with its own OAuth).
- **Claude** (`claude`): Claude-side connector surface.

## Products

Grant and entitlement calls take **product ids**, not display names. Map
(full list in `references/products.md`):

| Product id        | Display name     |
| ----------------- | ---------------- |
| `wp-ai-poster`    | Bylinely         |
| `bid-scout`       | WinnowBid        |
| `bidsidekick`     | BidSidekick      |
| `address-api`     | Address API      |
| `edgelark`        | Edgelark         |
| `sprigboard`      | Sprigboard       |
| `velosolabs-app`  | VelosoLabs (app) |
| `tradeline`       | Tradeline        |
| `voltwren`        | Voltwren         |
| `gizmohaus`       | Gizmohaus        |
| `moda-marieclaire`| Moda MarieClaire |
| `pennerly-pro`    | Pennerly Pro     |

Always resolve a display name to its id before calling; never send a display
name as an id.

## Worked example: connect SAM.gov → grant WinnowBid → run a search

1. **Check state.** `connectors.list` → `sam-gov` shows
   `connected:false, key_saved:false`. `grants.list` → no grant for
   `bid-scout`.
2. **Connect SAM.gov (gated write).** Tell the tenant you need to store their
   SAM.gov API key and that this is a Push-gated action. On confirm / Push,
   call `connectors.set_key` for `sam-gov` with the key the tenant supplies.
   The key is stored server-side; the result reports `key_saved:true` and
   **returns no secret**.
3. **Grant WinnowBid (gated write).** On confirm / Push, call
   `grants.grant_app` with product id `bid-scout` and the minimum ops —
   here `["sam-gov.search_opportunities"]`. Confirm the grant reads back.
4. **Run the search (broker execute).** `broker.execute` with op
   `sam-gov.search_opportunities` and the tenant's query. Since a search is a
   **read** op, it runs without a Push (still scope-checked and tenant-scoped).
   Return the **results only** — never the stored SAM.gov key.

More examples (grant BidSidekick, rotate a key, read billing, connect a
company-site connector) are in `references/workflows.md`.

## Guardrails (always)

Read `references/guardrails.md`. The non-negotiables:

- **Never surface secrets.** Connector credentials, stored API keys, OAuth
  tokens and client secrets are write-only from your side — return
  **results**, status flags (`key_saved`, `connected`) and metadata, never
  the secret itself. A one-time key shown on create/rotate is relayed through
  the MCP channel to the tenant, not pasted into chat or notes.
- **Respect scopes.** If the scope for an action is absent, stop and report;
  don't probe alternate tools to get around it.
- **Confirm before mutations.** Every connector/grant/key write and every
  write-class broker op is Push-gated. State plainly what will change, then
  wait for confirm / Push.
- **Stay in one tenant.** Never cross tenants.
- **Degrade gracefully before the MCP ships.** The account MCP is new; if it
  is unavailable, fall back to the **VelosoLabs App API** (the same gateway's
  REST/`/broker/v1/execute` surface) with the same tenancy, scope and
  gated-write rules — never bypass the broker to touch a connector or
  credential directly.
