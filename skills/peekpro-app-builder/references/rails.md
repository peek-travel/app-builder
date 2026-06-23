# Ruby on Rails build guide (stub — NOT first-class yet)

> ⚠️ **Warn the user first.** There is **no Peek API-translation SDK for Ruby today**
> (Node only). Without an SDK, talking to Peek means **raw GraphQL**, which Peek explicitly
> discourages because misuse can harm the installed account's infrastructure. Recommend
> **Node** (`node.md`). Only proceed in Rails if the user accepts this trade-off.

If the user proceeds anyway, follow this stub and **research current Rails + host best
practices live**.

> `ASK THE MCP` = fetch live from the `peek-pro` MCP. `TODO(verify)` = confirm with Peek.

## 1. Project init

```bash
rails new peek_app   # API-only is fine if no server-rendered UI: rails new peek_app --api
```

Provide routes/controllers for: **install endpoint**, **webhook endpoint**, the
**client-facing** surface, and the **admin** surface.

## 2. HTTP / GraphQL client

```ruby
# No Peek SDK for Ruby. Use a vetted GraphQL/HTTP client (e.g. the `graphql-client` or
# `graphlient` gem, or Faraday). TODO(verify) any Peek-recommended client.
```

`ASK THE MCP` for the current GraphQL endpoint + schema. Keep operations minimal; flag the
raw-GraphQL risk per the warning above.

## 3. Auth wiring (use Peek identity — never your own login / Devise)

- On **install**, store the account id + Peek-issued token (`ASK THE MCP` for payload shape),
  e.g. an `Installation` model keyed by account, tokens in encrypted credentials.
- Build a per-account authorized client from the stored token. Identify the caller from
  Peek's settings/token — **do not** add Devise/your own auth.

## 4. Webhooks, secrets, PII

- Verify webhook signatures (`ASK THE MCP` for scheme); make handlers idempotent.
- Use Rails encrypted credentials / the host's secret store; never commit tokens. Treat
  guest/payment data as sensitive PII (see `peek-api.md`).

`TODO(verify)`: revisit this stub when a Ruby SDK becomes available — the repo/skill will be
updated then.
