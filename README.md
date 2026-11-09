# VelosoLabs Agent Skills

Agent skills that teach AI coding agents to use the **VelosoLabs account
MCP** (`mcp.velosolabs.com`) correctly and safely. Modeled on
`supabase/agent-skills`.

## Install

```bash
npx skills add velosolabs/agent-skills
```

This installs the `velosolabs` skill. Pair it with the VelosoLabs account
MCP server (OAuth, tenant-scoped). The skill covers:

- **Tenant model** — every call is scoped to one tenant; never cross tenants.
- **OAuth scopes** — `tenant:read`, `connectors:read/write`,
  `grants:read/write`, `keys:read/write`, `billing:read`, `learn:*` — and the
  **gated-write rule** (writes with real-world effects go behind an explicit
  confirm / the tenant's **Push** button, same as the broker).
- **Workflows** — connectors (SAM.gov, BuildingConnected, company-site,
  Claude), app grants (WinnowBid, BidSidekick, …), entitlements & products,
  API keys, billing reads.
- **Guardrails** — never surface secrets (results only), respect scopes,
  confirm before mutations, degrade to the App API before the MCP ships.

## Layout

```
skills/velosolabs/SKILL.md
skills/velosolabs/references/{scopes,products,connectors,workflows,guardrails}.md
```
