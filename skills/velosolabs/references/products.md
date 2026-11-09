# VelosoLabs — product ids

Grant, entitlement and broker-app calls use the **product id**. Resolve any
display name to its id before calling; never send a display name as an id.

| Product id         | Display name      | Notes                                        |
| ------------------ | ----------------- | -------------------------------------------- |
| `wp-ai-poster`     | Bylinely          | WP AI posting; env prefix `WP_AI_POSTER`     |
| `bid-scout`        | WinnowBid         | Construction bid scout; a broker app         |
| `bidsidekick`      | BidSidekick       | Bid assistant; a broker app                  |
| `address-api`      | Address API       | Address lookup/validation                    |
| `edgelark`         | Edgelark          | Cloudflare-account manager app               |
| `sprigboard`       | Sprigboard        | Landscape-plan studio (uses VL sign-in)      |
| `velosolabs-app`   | VelosoLabs (app)  | First-party account app                      |
| `tradeline`        | Tradeline         |                                              |
| `voltwren`         | Voltwren          |                                              |
| `gizmohaus`        | Gizmohaus         |                                              |
| `moda-marieclaire` | Moda MarieClaire  |                                              |
| `pennerly-pro`     | Pennerly Pro      |                                              |

## Broker apps

Apps that call the broker/gateway (registered in `GATEWAY_BROKER_APPS` with an
Ed25519 public key + an allow-list of ops) include `bid-scout` (WinnowBid) and
`bidsidekick` (BidSidekick). When you `grants.grant_app`, you authorize a
product id to invoke specific connector ops through the broker for this
tenant. Grant the **minimum** op set the app needs — never the whole
connector namespace "to be safe".
