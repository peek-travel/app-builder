# Python build guide (stub — NOT first-class yet)

> ⚠️ **Warn the user first.** There is **no Peek API-translation SDK for Python today**
> (Node only). Without an SDK, talking to Peek means **raw GraphQL**, which Peek explicitly
> discourages because misuse can harm the installed account's infrastructure. Recommend
> **Node** (`node.md`). Only proceed in Python if the user accepts this trade-off, and keep
> the GraphQL surface minimal and read-mostly where possible.

If the user proceeds anyway, follow this stub and **research current best practices live**
for their framework (FastAPI, Django, Flask) and host.

> `ASK THE MCP` = fetch live from the `peek-pro` MCP. `TODO(verify)` = confirm with Peek.

## 1. Project init

- Pick a framework fitting the host. Provide routes for: **install endpoint**, **webhook
  endpoint**, the **client-facing** surface, and the **admin** surface.

## 2. HTTP / GraphQL client

```bash
# No Peek SDK for Python. Use a vetted GraphQL/HTTP client.
pip install httpx  # or gql — TODO(verify) any Peek-recommended client
```

`ASK THE MCP` for the current GraphQL endpoint + schema. Keep queries narrow; avoid
expensive/mutating operations unless necessary, and flag risk per the warning above.

## 3. Auth wiring (client-facing surface: use Peek identity — never your own login)

- Persist the account id + token from Peek's **install** payload (`ASK THE MCP` for shape).
- Build an authorized client per account from the stored token. Identify the caller from
  Peek's settings/token, not a local user model.
- This applies to the **client-facing** surface only. The **admin** surface (developer's own
  area, not installed in Peek Pro) **may** use its own auth.

## 4. Example call

```python
# Illustrative only. ASK THE MCP for the real schema/operations.
# Prefer the smallest query that does the job; verify signatures on webhooks.
```

## 5. Webhooks, secrets, PII

- See `references/webhooks.md` for the full contract. The npm parsers + standard query are
  **Node-only** — in Python, **port them as a template**: replicate the package's booking
  GraphQL field selection and its payload→model parsing (incl. ID normalization and the
  "events carry state, not change" handling). Treat the Node code as the canonical spec.
- Verify the delivery yourself (`ASK THE MCP` for scheme); make handlers idempotent.
- Use the host's secret store; never commit tokens. Treat guest/payment data as sensitive
  PII (see `peek-api.md`).

## 6. Testing (target ≥90% line coverage)

- Set up **pytest** with **coverage.py** (`pytest --cov`) from the start; aim for **≥90% line
  coverage** with meaningful tests.
- Prioritize the critical Peek logic: your ported webhook parsing, the **state-not-change**
  create/update derivation, **ID normalization**, `installDataId` scoping, and auth/token
  handling. Fail the build below the threshold.

`TODO(verify)`: revisit this stub when a Python SDK becomes available — the repo/skill will
be updated then.
