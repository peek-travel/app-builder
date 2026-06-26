# Peek Pro webhooks — contract & how to interact

How an app **listens to events** from Peek Pro. The patterns below are stable fixed-layer
knowledge. Exact configuration — the booking **GraphQL query string**, config field names,
and the precise parser API — **changes over time and must NOT be hard-coded here.**

## Always pull the latest webhook docs first

Before configuring or implementing webhooks, fetch and read Peek's official, current doc:

```
https://cdn.jsdelivr.net/npm/@peektravel/app-utilities/docs/webhooks.md
```

It is the **source of truth for the exact, best configuration** — the canonical booking
GraphQL query the npm package supplies, the app-config field names/values, and the exact
parser function signatures. Use it for anything concrete; use this file for the stable shape
and the caveats. `ASK THE MCP` for live, account-specific data (and the signature scheme if
the doc doesn't pin it).

## What's available today

Two webhooks: **booking events** and **waiver events** (details below).

## How webhooks are wired (two halves, deliberately split)

Setup has two parts that must agree with each other:

1. **Registration — external, one-time (app config / the registry).** You declare the webhook,
   its target endpoint, and the **payload spec** in the app's configuration (e.g. `app.json` /
   App Store config; the **registry** is documented separately — `TODO(verify)` full registry
   docs). For bookings the spec is a **GraphQL field selection** that shapes the payload; for
   waivers it's a **fixed format** (no query). The **exact config keys/values and the
   canonical booking query string live in the live doc — pull them from there, don't bake.**
2. **Endpoint — you implement it.** Peek POSTs the event to a URL your app exposes. Build this
   route/handler yourself.
3. **Parsing — in your receiver code, via the npm package.** Use the package's **pure parser
   functions** (`@peektravel/app-utilities`) to transform the delivery into a clean model
   instead of reading raw JSON. They are pure transforms — **no auth, no network.** Confirm
   the exact function names/signatures in the live doc; conceptually:
   `parseBookingWebhook(body) → Booking` and `parseWaiverWebhook(body) → Waiver`.

### Endpoint implementation rules (apply to every webhook)

- **Authenticating the delivery is YOUR responsibility.** The package/parsers do **not**
  verify that a request really came from Peek — verify it yourself before trusting it.
  `ASK THE MCP` / live doc for the scheme + header; `TODO(verify)` algorithm.
- **Parsers never throw on malformed input** — they yield **empty fields** rather than errors,
  so a bad delivery won't crash you, but you must **validate the fields you depend on**.
- **Parsers accept multiple input shapes** — the `{ booking: … }` / `{ waiver: … }` envelope,
  a bare node, or a JSON-string body. Pass the body through; don't pre-shape it.
- **Acknowledge fast, process safely; be idempotent.** The doc gives no retry/dedup guarantee,
  so assume at-least-once delivery and possible redelivery.
- **Scope stored data to `installDataId`** (see `peek-api.md`), keyed by the stable IDs below.

---

## Booking events

### Payload is shaped by a registered GraphQL query

When you register the booking webhook you provide a **GraphQL field selection** that Peek runs
to build the payload — the app's way of saying *"this is the data I care about."* Because the
query is complex and the payload shape is **not fixed**, registration and parsing must agree:

- **Use the canonical/standard booking query that the npm package supplies** (don't
  hand-write it). The exact query string is **volatile — get it from the live doc**, not from
  here.
- **Parse with the package's booking parser** (`parseBookingWebhook → Booking`). The booking
  parser **auto-detects guests and the price breakdown** from the payload, and optional fields
  populate when present — nothing to keep in sync.

### The critical caveat: events carry state, not change

**The booking webhook fires on both create *and* update, delivering the *same* payload shape,
and the parser handles them identically — it does not surface which event fired.** The event
contains only the booking's **current state**, with no diff and no event type.

So the app can't tell from an event alone whether the booking was **created, cancelled,
rescheduled, or edited**. To derive meaning, track state yourself:

- **Act only on *new* bookings:** keep a **cache/database of bookings you've already seen**;
  on each event, if you've seen the ID, ignore it; if not, it's new.
- **Detect a *specific change* (e.g. a reschedule):** **store the old value of the field(s)
  you care about** and compare the incoming value on each event. (`ASK THE MCP` / live doc for
  the exact field paths in the model.)

### Stable keys for your stores / lookups / caches

- **The booking ID and the order ID never change.** Use them as keys for caches, databases,
  and lookups (combine with `installDataId` scoping).
- **Normalize IDs before using them as keys.** Booking/order IDs have two formats — internal
  `b_123abc`/`o_123abc` (lowercase + underscore) and display `B-123ABC`/`O-123ABC` (uppercase
  + dash). Always convert any ID you receive to the **internal lowercase-underscore form
  first** (lowercase + replace `-` with `_`) so your store keys match. See `peek-api.md`
  "Booking & order ID formats."

---

## Waiver events

The waiver webhook fires when a **waiver agreement signature is created** (the
`agreement_signature_created` trigger).

- **Fixed payload — no GraphQL query to register.** Unlike bookings, the shape is predefined
  by Peek, so there's no field-selection step.
- **Parse with the package's waiver parser** (`parseWaiverWebhook → Waiver`). Like the booking
  parser it's a pure transform and never throws on malformed input.
- Apply the same **endpoint rules** (you verify the delivery, ack fast, idempotent, scope to
  `installDataId`). If you need new-vs-seen logic, use the same seen-before cache pattern on a
  stable identifier (`ASK THE MCP` / live doc for the waiver's ID field; a referenced booking's
  booking/order IDs remain stable).

---

## Summary for the build (steps 3 & 5)

- **Pull the live webhook doc** for the exact config and parser API before implementing.
- Decide which webhook(s) the feature needs (step 3); for bookings that choice drives the
  registered GraphQL field selection.
- Implement the endpoint(s) (step 5): **you** verify the delivery, ack fast, idempotent.
- Use the npm package's **standard booking query + `parseBookingWebhook` / `parseWaiverWebhook`
  parsers** (Node) — don't hand-write the query or raw-parse the payload, and validate the
  fields you rely on (parsers return empty fields rather than throwing).
- Treat events as **state, not change**: maintain a store keyed on the never-changing,
  **normalized** booking/order IDs (scoped to `installDataId`) to derive created/cancelled/
  rescheduled.
