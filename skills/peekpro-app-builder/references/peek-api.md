# Peek Pro — canonical app & API knowledge (fixed layer)

Stable facts about how Peek Pro apps work. This is shared across every stack. Volatile,
frequently-changing details are intentionally **not** hard-coded here — they are marked
**`ASK THE MCP`** (query the bundled `peek-pro` MCP) or **`TODO(verify)`** (confirm against
the Development Hub) so this file never goes stale.

> Convention:
> - **`ASK THE MCP`** — live data the Peek MCP should serve. Don't guess it.
> - **`TODO(verify)`** — a detail the Peek team must confirm; treat as unknown until then.

---

## What a Peek Pro app is

An app **extends** Peek Pro with functionality it lacks natively (waitlist,
abandoned-booking recovery, dynamic pricing, custom checkout, reseller/channel sync,
reporting, …). Apps are built and run **outside** Peek, register with Peek, and are
installed by Peek Pro clients.

### Lifecycle

1. **Register** the app in the **Peek Development Hub**. The Hub provisions the
   **environment** (sandbox vs. production) and issues **security keys / credentials**.
   `ASK THE MCP` for the exact registration inputs and what credentials are returned.
   `TODO(verify)` Development Hub URL and onboarding steps.
2. **Publish** to the **Peek App Store**. Any Peek Pro client can then install the app.
3. **Install** — when a client installs, Peek calls the app's **install endpoint** (a
   webhook the app exposes) with install info, including the **three IDs** below and auth
   token(s). `ASK THE MCP` for the exact install payload shape (the field keys, environment,
   scopes). `TODO(verify)` install endpoint contract and required HTTP response.
4. **Configure** — the client customizes the app via **settings**. **Auth tokens are
   delivered through this channel**, so the app always knows *which account/user* is calling.
   `ASK THE MCP` for the settings schema and token format/refresh rules.

## Authentication (client-facing surface: use Peek's, never your own)

> **Scope:** this rule is about the **client-facing surface** — the part of the app installed
> within Peek Pro and accessed by Peek Pro users. The **admin** surface (for the app
> developer/owner) is separate and **may have its own auth** (see "The two surfaces").

- The **client-facing app must not implement its own login / user accounts**. Identity and
  authorization come from Peek via the install + settings flow.
- Each install hands the app **auth token(s)** scoped to that account. The app uses them to
  call Peek and to identify the caller.
- `ASK THE MCP` for: token type (e.g. bearer/OAuth), lifetime, refresh mechanism, and the
  scope/permission model. `TODO(verify)` how sandbox vs. production credentials differ.
- Store tokens as **secrets** (never in the repo, never in client-side code). See the
  stack reference for where secrets live on the chosen platform.

## Identity & data scoping: the three IDs (important — design around this)

On **install** and on **every access**, Peek passes the app three identifiers. Account for
all three (these are confirmed; `ASK THE MCP` only for the exact payload **field names**):

- **User ID** — the current user accessing or installing the app (who is acting right now).
- **Partner ID** (also called **Account ID**) — the Peek Pro **account** the app is installed
  into.
- **Install ID** — the unique **account+app** identifier. **It does NOT rotate:** uninstall
  then reinstall yields the **same** Install ID.

### Strongly recommended: scope data to an `installDataId`, tracked as `currentInstallDataId`

Because the Install ID is stable across reinstalls, keying your data on the Install ID alone
means a reinstalled app inherits **stale data** from the previous install. Instead:

- At each install, mint an **`installDataId` = Install ID + install timestamp**.
- Store it on an account object as **`currentInstallDataId`**, and **scope all of the app's
  and install's data to that `installDataId`**.
- On reinstall, a **new** `installDataId` is created, so the user **starts with a clean
  slate** (old data is no longer referenced by `currentInstallDataId`).
- A **post-uninstall data wiper** then knows exactly which data to remove: everything tied to
  the prior `installDataId`.

This pattern gives clean reinstalls and unambiguous data cleanup. Build it in from the start —
retrofitting data scoping later is painful.

## The two surfaces every app has

1. **Client-facing surface** — what gets installed into each Peek Pro account and shown to
   that client (and/or their guests). Configured per-install via settings.
2. **Admin surface** — for the **app developer/owner**: view installs, read logs, monitor
   health, manage configuration across accounts.

Build both. Keep their auth/permissions separate:
- **Client-facing surface** — acts on behalf of the installing account; identity comes from
  Peek (no own login — see Authentication above).
- **Admin surface** — the developer's own area; it is **not** installed inside Peek Pro and is
  **not** accessed by Peek Pro users, so it **may use its own authentication** (the
  developer's login/SSO). The "no own login" rule does **not** apply here.

## Talking to Peek: two mechanisms

### A. Webhooks (inbound — "something happened in Peek")

Peek emits events the app subscribes to. Reactive features (waitlist, abandoned bookings,
dynamic pricing) depend on these. **Two webhooks are available today: booking events and
waiver events.** They're configured through the **registry** and delivered to an endpoint the
app implements.

> **See `references/webhooks.md` for the full contract** — registry setup, the
> registry-defined GraphQL query that shapes the booking payload, the npm package's standard
> query + booking-model parser, and the critical caveat that booking events carry **state,
> not change** (use the never-changing booking/order IDs + a seen-before store to derive
> created/cancelled/rescheduled).

- `ASK THE MCP` for the **current webhook/event catalog**: payload shapes, delivery
  guarantees, retry behavior, and **signature/verification** scheme.
- Always **verify webhook signatures** before trusting a payload. `TODO(verify)` signing
  scheme + header name.
- Expect at-least-once delivery; design handlers to be **idempotent**. `TODO(verify)`.

### B. Outbound API — prefer the SDK, avoid raw GraphQL

- Peek exposes a **GraphQL API**, but **raw GraphQL is risky**: misuse can negatively impact
  the installed account's underlying infrastructure. **Do not hand-write GraphQL** unless
  there is genuinely no alternative for the chosen language.
- Instead, use Peek's **API-translation package (SDK)**, which wraps GraphQL safely.
  **Currently available for Node only.** See `node.md`.
- `ASK THE MCP` for: the current GraphQL **schema**, the **SDK surface** (methods, version),
  and how SDK calls map to operations.
- For non-Node languages, see the language stub and the warning in `SKILL.md` step 1.

## Booking & order ID formats — normalize on input

Booking and order IDs come in **two representations**:

- **Internal / canonical** — lowercase with an underscore: **`b_123abc`** (booking),
  **`o_123abc`** (order).
- **Display** — uppercased with a dash (easier for humans): **`B-123ABC`** (booking),
  **`O-123ABC`** (order).

**Whenever you receive a booking or order ID — from the API, a webhook, a URL, user input,
anywhere — normalize it to the internal lowercase-underscore form *first*** (lowercase the
whole string and replace `-` with `_`, e.g. `B-123ABC` → `b_123abc`). Store, compare, and key
caches/DBs/lookups on the **canonical** form; use the display form **only** for showing to
humans. Mixing the two formats as keys causes duplicate or missed records — and remember these
IDs never change, so they're your stable keys (scope them to `installDataId`).

## Core resources (structure — confirm specifics live)

Peek's domain centers on these. **Field-level and endpoint/operation specifics are
volatile — `ASK THE MCP` for the current schema; do not invent field names.**

- **Products / activities** — the bookable experiences (tours, activities, rentals).
  `ASK THE MCP` for product schema and how to list/read products. `TODO(verify)`.
- **Availability / timeslots** — when a product can be booked; capacity per slot.
  `ASK THE MCP` for timeslot schema, how to query availability, capacity semantics.
  `TODO(verify)`.
- **Bookings** — a customer's reserved spot(s) on a timeslot. Central to most apps.
  `ASK THE MCP` for booking schema, create/cancel/modify operations, status values.
  `TODO(verify)`.
- **Orders** — the commercial wrapper around bookings (line items, totals).
  `ASK THE MCP` for order schema and relationship to bookings. `TODO(verify)`.
- **Payments** — charges, refunds, payment status on a booking/order.
  `ASK THE MCP` for payment operations and statuses. **Treat all payment data as sensitive.**
  `TODO(verify)`.
- **Customers / guests** — PII-bearing. Minimize what you store; see security note.
  `ASK THE MCP` for customer schema. `TODO(verify)`.

## Security & PII (applies to every app)

Peek data routinely includes **sensitive PII** (guest names, emails, phone numbers, and
payment-related metadata). Default to:

- Encrypt in transit (HTTPS only) and at rest; restrict who/what can read it.
- Store the **minimum** necessary; prefer referencing Peek IDs over copying PII.
- Keep secrets out of source control and client bundles; use the platform's secret store.
- Don't log PII or tokens. Redact.
- `TODO(verify)` any Peek-specific compliance requirements (data residency, retention).

## Sandbox vs. production

The Development Hub provisions both. Build and validate against **sandbox** first; never
test against a live account. `ASK THE MCP` for how to target sandbox vs. production and any
differences in credentials/endpoints. `TODO(verify)`.

---

### Summary of what the Peek MCP should serve (for the Peek/dev-portal team)

Tier-1 (MVP): current GraphQL schema · webhook/event catalog + payload shapes ·
install-endpoint contract · settings/auth-token contract · Node SDK surface/version.
Tier-2: Development Hub onboarding (register app, sandbox keys, list installs) · sandbox
validation of generated calls. Until these exist, every `ASK THE MCP` above degrades to a
`TODO(verify)` the developer must confirm manually.
