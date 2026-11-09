# VelosoLabs MCP — scopes & gated-write classification

## Scope → tool matrix

| Tool                     | Required scope      | Class  |
| ------------------------ | ------------------- | ------ |
| `tenant.get`             | `tenant:read`       | read   |
| `entitlements.list`      | `tenant:read`       | read   |
| `products.list`          | `tenant:read`       | read   |
| `connectors.list`        | `connectors:read`   | read   |
| `connectors.get`         | `connectors:read`   | read   |
| `connectors.connect`     | `connectors:write`  | GATED  |
| `connectors.set_key`     | `connectors:write`  | GATED  |
| `connectors.disconnect`  | `connectors:write`  | GATED  |
| `grants.list`            | `grants:read`       | read   |
| `grants.get`             | `grants:read`       | read   |
| `grants.grant_app`       | `grants:write`      | GATED  |
| `grants.revoke`          | `grants:write`      | GATED  |
| `keys.list`              | `keys:read`         | read   |
| `keys.create`            | `keys:write`        | GATED  |
| `keys.rotate`            | `keys:write`        | GATED  |
| `keys.revoke`            | `keys:write`        | GATED  |
| `billing.get`            | `billing:read`      | read   |
| `billing.usage`          | `billing:read`      | read   |
| `broker.execute` (read)  | scope of the op's connector (`connectors:read`) + a live grant | read |
| `broker.execute` (write) | scope of the op's connector (`connectors:write`) + a live grant | GATED |
| `learn.docs`             | `learn:*`           | read   |
| `learn.search`           | `learn:*`           | read   |
| `learn.describe`         | `learn:*`           | read   |

## The gated-write rule in detail

A **GATED** operation has a real-world side effect: it stores a credential,
changes who can access what, mints/kills a secret, or causes an external
write (e.g. submitting/altering data in SAM.gov or BuildingConnected). For
every GATED op:

1. State exactly what will change, in which tenant, and why.
2. Require an explicit confirm. The tenant's **"Push" button** on the
   VelosoLabs account is the canonical confirmation surface — the same
   mechanism the broker uses for first-party apps. The MCP may return
   `requires_push: true` / a pending action id; surface it and wait.
3. Only proceed once the Push/confirm is cleared. Never loop or retry to
   defeat the gate, and never substitute a different tool to avoid it.

Reads (`read` class) run directly, subject only to scope + tenant checks.

## Scope hygiene

- Check granted scopes up front; if a task needs a scope you don't have,
  report the missing scope and stop.
- Request/assume the **minimum** scope for the task. A reporting task needs
  only `*:read` + `billing:read`; don't exercise write scopes you won't use.
- `learn:*` is informational only — never a substitute for a missing data
  scope.
