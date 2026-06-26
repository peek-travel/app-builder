# Peek Pro webhooks — contract & how to interact

How an app **listens to events** from Peek Pro. This is fixed-layer knowledge: the patterns
below are stable. Exact payload field names, the signing scheme, and registry mechanics are
volatile — `ASK THE MCP` for those, or `TODO(verify)` if the MCP is unavailable.

## What's available today

Two webhooks:

1. **Booking events** — see [Booking events](#booking-events).
2. **Waiver events** — see [Waiver events](#waiver-events).

## How webhooks are wired (both types)

1. **Register & configure in the registry.** Webhooks are set up through the **registry**
   (Peek's app configuration registry — documented separately; `TODO(verify)` full registry
   docs / URL). Registration is where you declare the webhook, its target endpoint, and —
   for bookings — the GraphQL query that shapes the payload (see below).
2. **Expose an endpoint in your app.** Peek POSTs the event to a URL you implement as part of
   the app. You must build this endpoint (route/handler) yourself.
3. **Process the incoming data.** Parse it, decide what (if anything) changed that you care
   about, and act. See the per-type caveats below.

### Endpoint implementation rules (apply to every webhook)

- **Verify the signature** before trusting a payload. `ASK THE MCP` for the scheme + header;
  `TODO(verify)` algorithm.
- **Acknowledge fast, process safely.** Return a 2xx quickly; do heavier work async if needed.
- **Be idempotent.** Assume at-least-once delivery and possible redelivery.
- **Scope stored data to `installDataId`** (see `peek-api.md` "Identity & data scoping"), keyed
  by the stable IDs noted below.

---

## Booking events

### You define the payload via a GraphQL query (in the registry)

When you register the booking webhook, you provide a **GraphQL query** that Peek Pro runs to
generate the payload you receive. This is the app's way of saying *"this is the data I care
about"* — whatever the query selects is what arrives on the webhook.

These queries get complicated, so:

- **Use the standard GraphQL configuration that ships in the Peek npm package**
  (`@peektravel/app-utilities`) rather than hand-writing the query. `ASK THE MCP` /
  `TODO(verify)` the exact export to use for the standard booking query.
- **Parse the incoming payload with the npm package into its booking model**, instead of
  reading raw JSON by hand. `TODO(verify)` the exact parser/model export.

### The critical caveat: events carry state, not change

**Peek Pro sends an "updated booking" event every time *anything* about the booking
changes — and the event contains only the new booking data, not a diff or an event type.**

So on receipt the app does **not** know *what* happened: it cannot tell from the event alone
whether the booking was **created, cancelled, rescheduled, or edited**. It just gets the
current state.

To derive meaning, the app must track state itself:

- **To act only on *new* bookings:** keep a **cache/database of bookings you've already
  seen**. On each event, look the booking up; if you've seen it, ignore it; if it's new,
  it's a new booking. (This is how you turn "updated" events into "created".)
- **To detect a *specific change* (e.g. a reschedule):** **store the old value of the
  field(s) you care about**, and on each event compare the incoming value to the stored one.
  If it differs, that field changed. (E.g. watch the timeslot/date field to catch reschedules;
  watch status to catch cancellations.) `ASK THE MCP` for the exact field paths in the model.

### Stable keys for your stores / lookups / caches

- **The booking ID and the order ID never change.** Use them as the keys for caches,
  databases, and lookups. (Combine with `installDataId` scoping.)

---

## Waiver events

Waiver events follow the same wiring: **registered/configured in the registry**, delivered to
an **endpoint you implement**, and **processed** by the app.

- `ASK THE MCP` for the waiver event payload shape and which fields are included.
- `TODO(verify)`: whether waiver webhooks also use the registry-defined **GraphQL query**
  mechanism (like bookings) or ship a fixed payload.
- `TODO(verify)`: whether the npm package provides a standard waiver query + model parser
  (prefer it if so, mirroring the booking pattern).
- Apply the same endpoint rules (verify signature, idempotent, ack fast, scope to
  `installDataId`). If you need to distinguish new vs. updated waivers, use the same
  seen-before cache pattern keyed on a stable waiver identifier (`ASK THE MCP` for the ID
  field; the booking/order IDs remain stable if the waiver references a booking).

---

## Summary for the build (steps 3 & 5)

- Decide which webhook(s) the feature needs (step 3), and that data shapes the registry
  GraphQL query.
- Implement the endpoint(s) in the app (step 5): signature-verified, idempotent, fast ack.
- Use the npm package's standard GraphQL config + model parser (Node). Don't hand-roll the
  query or raw-parse the payload.
- Treat events as **state, not change**: maintain a store keyed on the **never-changing
  booking/order IDs** (scoped to `installDataId`) to derive created/cancelled/rescheduled.
