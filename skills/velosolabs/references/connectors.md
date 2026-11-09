# VelosoLabs — connectors & op namespaces

A **connector** links the tenant's VelosoLabs account to an external data
source. Credentials live server-side in the gateway; your side is write-only
for secrets and read-only for status.

## Status fields

`connectors.list` / `connectors.get` return per-connector status such as:
- `connected` (bool) — OAuth/handshake complete
- `key_saved` (bool) — a required API key is stored (key-based connectors)
- `ops` — the operation namespace available for this connector

Treat these flags as the source of truth; don't assume a connector is live.

## Catalog

| Connector id       | Auth style            | Example ops                                   |
| ------------------ | --------------------- | --------------------------------------------- |
| `sam-gov`          | API key (`set_key`)   | `sam-gov.search_opportunities`                |
| `buildingconnected`| Autodesk APS OAuth    | `buildingconnected.list_opportunities`        |
| `company-site`     | Remote MCP OAuth      | tenant's own site exposed as a Claude connector|
| `claude`           | Claude-side OAuth     | Claude connector surface                      |

### SAM.gov (`sam-gov`)
Key-based. Connect with `connectors.set_key` (GATED) — the tenant supplies a
SAM.gov API key; it is stored and `key_saved` flips to `true`. The key is
never returned. Reads like `sam-gov.search_opportunities` run through
`broker.execute`.

### BuildingConnected (`buildingconnected`)
Autodesk APS OAuth (APS client id/secret live in the gateway). Connect with
`connectors.connect` (GATED) which drives the OAuth handshake. Ops such as
`buildingconnected.list_opportunities` run through `broker.execute`.

### company-site (`company-site`)
Turns the tenant's own site into a **Claude connector** (a remote MCP with
its own OAuth). The tenant toggles it on in their account settings; until it
is on, its endpoints answer empty/404. Respect that off/on state — don't
force-enable.

### Claude (`claude`)
The Claude-side connector surface for the tenant.

## Op classification

Most listed ops are **reads** (searches, lists, lookups) → run directly via
`broker.execute` subject to scope + a live grant. Any op that **writes** to
the external system (submits/edits a record) is **GATED** — Push-confirm
first. If you are unsure whether an op writes, treat it as a write.
