# Step 2 — Identify the app's purpose

**Goal:** understand what the app should accomplish, in enough detail to design it. Only do
this **after** the stack is settled (step 1).

> Interaction rule: ask **one question at a time**. Let the user think out loud; offer
> example features and trade-offs as you go.

## Discover

- **The feature/problem** — what gap in Peek Pro does this fill? (waitlist, abandoned-booking
  recovery, dynamic pricing, custom checkout, reporting, reseller/channel sync, …)
- **Who uses it** — the Peek Pro account staff, end customers/guests, the app developer, or a
  mix? (This maps to the two surfaces — client-facing vs. admin.)
- **What triggers it** — a Peek **event/webhook** (booking created, timeslot filled, booking
  abandoned, …), a schedule, or a user action? Most "reactive" apps (waitlist, abandoned
  bookings, dynamic pricing) are webhook-driven.
- **What it reads/changes in Peek** — products, availability/timeslots, bookings, orders,
  payments, customers? (See `references/peek-api.md` for the resource map.)
- **Success criteria / scope for v1** — keep the first version tight.

## Keep in mind for later steps

- Reactive features depend on the **webhook/event catalog** — you'll confirm which events
  exist in step 3 via the MCP (`ASK THE MCP`).
- Anything that stores data must be **scoped to an `installDataId`** (see `peek-api.md`).

## Output of this step

A concise problem statement: the feature, the users/surfaces, the trigger(s), and the Peek
resources involved — enough to research and plan in step 3.

## Artifacts (optional, drop them in this folder)

- A **requirements / purpose brief** template, or example briefs for common app types
  (waitlist, dynamic pricing) can live here.
