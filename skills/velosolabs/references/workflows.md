# VelosoLabs MCP — workflow playbooks

Each playbook: read current state first, name the change, Push-gate every
write, surface results only.

## 1. Connect a connector

1. `connectors.list` → find the connector, read `connected` / `key_saved`.
2. If key-based (e.g. `sam-gov`) and `key_saved:false`: tell the tenant you
   will store their API key (GATED). On confirm/Push, `connectors.set_key`
   with the supplied key. Verify `key_saved:true` in the result. No secret
   is returned.
3. If OAuth-based (e.g. `buildingconnected`, `company-site`) and
   `connected:false`: on confirm/Push, `connectors.connect` to drive the
   handshake; verify `connected:true`.
4. To remove: `connectors.disconnect` (GATED) — warn that dependent grants
   will stop working.

## 2. Grant / revoke an app

1. `grants.list` → see what each product is granted.
2. Decide the **minimum** op set the app needs (see
   `references/connectors.md`).
3. On confirm/Push, `grants.grant_app` with the **product id** (e.g.
   `bid-scout`) and the op allow-list. Read the grant back to confirm.
4. To remove: `grants.revoke` (GATED). Confirm the app loses exactly the
   intended ops.

## 3. Entitlements & products (read-only)

- `products.list` → catalog of product ids ↔ display names.
- `entitlements.list` → what this tenant is entitled to (plan-driven).
  Report; do not attempt to change entitlements here.

## 4. API keys

1. `keys.list` → metadata only (id, label, created, last-used, scopes). Never
   expect or display the secret.
2. `keys.create` / `keys.rotate` (GATED): the secret is shown **once** in the
   result. Relay it to the tenant through the MCP's own secure return path;
   do **not** echo it into chat, a file, a journal, or a commit. State that it
   won't be retrievable again.
3. `keys.revoke` (GATED): confirm which key (by id/label) and that dependents
   may break.

## 5. Billing (read-only)

- `billing.get` → plan, status.
- `billing.usage` → metered usage / period figures.
- Report numbers plainly. Charging and plan changes are **out of scope** —
  do not attempt them; direct the tenant to the account UI.

---

## Worked example A — connect SAM.gov → grant WinnowBid → run a search

(Full narrative in SKILL.md.) Sequence:
`connectors.list` → `connectors.set_key(sam-gov)` [GATED] →
`grants.grant_app(bid-scout, ["sam-gov.search_opportunities"])` [GATED] →
`broker.execute(sam-gov.search_opportunities, {query})` [read] → return
results only.

## Worked example B — grant BidSidekick a BuildingConnected read

1. `connectors.get(buildingconnected)` → ensure `connected:true` (if not, run
   playbook 1 first).
2. `grants.grant_app` with product id `bidsidekick` and ops
   `["buildingconnected.list_opportunities"]` [GATED].
3. The app now runs that op via the broker. Verify with `grants.get`.

## Worked example C — rotate an API key safely

1. `keys.list` → identify the key by label.
2. Confirm with the tenant that rotation invalidates the old secret.
3. `keys.rotate` [GATED] → relay the new one-time secret through the MCP
   return channel. Never paste it anywhere else.

## Worked example D — connect a company-site Claude connector

1. Confirm the tenant has toggled `company-site` **on** in account settings
   (off → endpoints 404/empty by design).
2. `connectors.connect(company-site)` [GATED] to complete OAuth.
3. Grant the relevant product the site's ops as needed [GATED].
