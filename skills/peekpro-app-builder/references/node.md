# Node / TypeScript build guide (PREFERRED stack)

Node is **first-class** for Peek apps: it's the only language with Peek's API-translation
SDK today. Prefer it. Pair with the user's chosen hosting/database and **research current
best practices for that platform live** (see `SKILL.md` step 3) — this file covers the
stable Node + Peek pattern only.

> `ASK THE MCP` = fetch live from the `peek-pro` MCP. `TODO(verify)` = confirm with Peek.

## 1. Project init

- Use TypeScript. Pick the framework that fits the chosen host (e.g. Next.js on Vercel,
  Cloud Functions on Firebase, a plain Node/Express or Hono service on Fly.io). **Research
  the current recommended setup for that host at build time.**
- Structure (adapt to the framework):
  - an **install endpoint** route (receives Peek's install callback),
  - a **webhook endpoint** route (receives Peek events),
  - the **client-facing** surface, and the **admin** surface (separate routes/auth),
  - a thin module wrapping the Peek SDK client.

## 2. Install the Peek SDK

```bash
# TODO(verify): exact package name + install command — ASK THE MCP for the current SDK.
npm install <peek-node-sdk-package>
```

`ASK THE MCP` for: package name, current version, and the client's method surface.

## 3. Auth wiring (client-facing surface: identity comes from Peek — never build your own login)

- On **install**, persist the account id + the issued token(s) from Peek's install payload
  into your secret store, keyed by account. `ASK THE MCP` for the install payload shape.
- On each request/webhook, resolve the calling account and load its token to construct the
  SDK client. Identify the caller from Peek's settings/token, not a local user table.
- This applies to the **client-facing** surface only. The **admin** surface (the developer's
  own area, not installed in Peek Pro) **may** use its own auth.

```ts
// Shape is illustrative — ASK THE MCP for the real SDK client + auth options.
import { PeekClient } from "<peek-node-sdk-package>"; // TODO(verify) import name

function clientForAccount(accountId: string) {
  const token = secrets.get(accountId);            // from the install/settings token
  return new PeekClient({ token /* , environment: sandbox|production */ });
}
```

## 4. Example call (use the SDK — never raw GraphQL)

```ts
// Method names below are placeholders. ASK THE MCP for the real SDK surface.
const client = clientForAccount(accountId);

// e.g. list products / check availability / read a booking
const products = await client.products.list();        // TODO(verify) method
const slots = await client.availability.forDay(date);  // TODO(verify) method
```

Do **not** drop down to raw GraphQL to "fill the gap" — if the SDK lacks a method, `ASK THE
MCP` whether one exists before considering raw GraphQL, and flag the risk to the user.

## 5. Webhooks

- Expose a webhook route; **verify the signature** before processing. `ASK THE MCP` for the
  event catalog + signature scheme. `TODO(verify)` header/algorithm.
- Make handlers **idempotent** (events may be redelivered).

## 6. Testing (target ≥90% line coverage)

- Set up a test runner from the start (e.g. **Vitest** or **Jest**) with a **coverage
  reporter** enabled (`--coverage`); research the current best choice for the framework.
- Aim for **≥90% line coverage** with meaningful tests. Prioritize the critical Peek logic:
  webhook parsing/handling, the **state-not-change** create/update derivation, **ID
  normalization** (`B-123ABC` → `b_123abc`), `installDataId` scoping, and auth/token handling.
- Add a CI step (or npm script) that fails the build below the coverage threshold.

## 7. Secrets & PII

- Use the host's secret manager (e.g. Vercel/Firebase/Fly env + secret stores) — **research
  the current best practice for the chosen host**. Never commit tokens.
- Treat guest/payment data as sensitive PII (see `peek-api.md` security note).
