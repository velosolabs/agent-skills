# VelosoLabs MCP — guardrails

## 1. Never surface secrets
- Connector API keys, OAuth tokens, client secrets and stored credentials are
  **write-only** from your side. Return **results**, status flags
  (`key_saved`, `connected`), and metadata — never the secret.
- A one-time secret from `keys.create` / `keys.rotate` is relayed to the
  tenant through the MCP's secure return path only. Do **not** echo it into
  chat, logs, files, journals, or commits. Say it is non-retrievable.
- If any tool result appears to leak a secret, stop, do not repeat it, and
  report the leak.

## 2. Respect scopes
- Check granted scopes before acting. Missing scope → report and stop.
- Never probe alternate tools or endpoints to get around an absent scope or a
  gate.

## 3. Confirm before mutations (Push-gated)
- Every connector write, grant write, key write, and write-class broker op is
  GATED. Name the exact change (what, which tenant, effect), then wait for the
  tenant's **Push** / explicit confirm. Proceed only once cleared.

## 4. One tenant, always
- The connection is authorized for a single tenant. Never accept, infer, or
  pass another tenant id. Cross-tenant data in a result is an incident, not an
  input.

## 5. Least privilege on grants
- Grant apps the **minimum** ops they need, never a whole connector namespace
  "to be safe". Prefer revocable, narrow grants.

## 6. Degrade before the MCP ships
- The account MCP is new. If it is unavailable, fall back to the **VelosoLabs
  App API** — the same gateway's REST / `/broker/v1/execute` surface — under
  the **same** tenancy, scope and gated-write rules.
- Never bypass the broker/gateway to touch a connector, credential, or
  external system directly. The broker is the only sanctioned path, MCP or
  App-API.

## 7. SOC2-minded, non-blocking
- Favor access control, least privilege, encryption-in-transit, and an audit
  trail (grants/keys/connector changes are logged by the gateway). Flag a gap
  rather than inventing an unsafe workaround — but never let a compliance
  concern block the tenant's legitimate, confirmed action.
