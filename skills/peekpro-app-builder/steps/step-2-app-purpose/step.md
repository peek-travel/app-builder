# Step 2 — Identify the app's purpose (and mock it up)

**Goal:** understand what the app should accomplish, in enough detail to design it — then make
it concrete with an **interactive HTML mockup** the user can click through and react to. Only
do this **after** the stack is settled (step 1).

> Interaction rule: ask **one question at a time**. Let the user think out loud; offer
> example features and trade-offs as you go.

## Part A — Discover

- **The feature/problem** — what gap in Peek Pro does this fill? (waitlist, abandoned-booking
  recovery, dynamic pricing, custom checkout, reporting, reseller/channel sync, …)
- **Who uses it** — the Peek Pro account staff, end customers/guests, the app developer, or a
  mix? (This maps to the two surfaces — client-facing vs. admin.)
- **What triggers it** — a Peek **event/webhook** (booking created, timeslot filled, booking
  abandoned, …), a schedule, or a user action? Most "reactive" apps are webhook-driven.
- **What it reads/changes in Peek** — products, availability/timeslots, bookings, orders,
  payments, customers? (See `references/peek-api.md` for the resource map.)
- **Success criteria / scope for v1** — keep the first version tight.

## Part B — Generate an interactive mockup (iterative loop)

Once the user has described what they want, **generate a single-file `index.html` mockup** of
the proposed app and have them review and interact with it. Iterate until they're happy.

### The loop

1. Build `index.html` (one self-contained file) in the user's project from what you've learned.
2. Tell the user to open it in a browser; invite them to click through and react.
3. Collect feedback (one question at a time for anything unclear) and revise the file.
4. Repeat until the user is satisfied that it reflects what they want.

### When something is unclear — show variants in the SAME file

If you're unsure about a layout, flow, or option, **render multiple versions in the one
`index.html`** so the user can compare directly — e.g. wrap each variant in an `ody-tabs`
tab ("Variant A" / "Variant B"). Let the user pick; then collapse to the chosen direction.

### Use the Odyssey design system (Peek's UI components)

Odyssey ships framework-agnostic **`<ody-*>` web components** via npm
(`@peektravel/app-utilities`). Use them for the mockup so it looks like a real Peek app.

1. **First, load the current component docs** (they're the source of truth and change over
   time) — fetch and read:
   `https://cdn.jsdelivr.net/npm/@peektravel/app-utilities/docs/ui.md`
   It lists every component, its tag, attributes, and usage conventions.
2. **Wire the CDN includes** into the `<head>` of `index.html` (exact snippet):

   ```html
   <head>
     <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=IBM+Plex+Sans:wght@400;500;600&display=swap">
     <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@peektravel/app-utilities/dist/ui/tokens.css">
     <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@peektravel/app-utilities/dist/ui/odyssey.css">
     <script type="module" src="https://cdn.jsdelivr.net/npm/@peektravel/app-utilities/dist/ui/index.js"></script>
   </head>
   ```

3. **Build with the components per `ui.md`.** Key conventions: scalars (strings/booleans) are
   **attributes**; rich data (arrays/objects) are set as **JS properties** on the element
   (`el.columns = [...]`, `el.data = [...]`); events come as `CustomEvent` (`addEventListener`,
   read `event.detail`). **Wrap settings/page UI in `<ody-page-container>`** (the standard
   responsive wrapper).

`steps/step-2-app-purpose/mockup-template.html` in this folder is a ready starter (head wired
up + `ody-page-container` + an `ody-tabs` variant scaffold) — copy it into the user's project
and build on it.

> If `ui.md` is unreachable, say so and fall back to a clean, neutral HTML mockup (don't
> invent `ody-*` attributes); note that it should be re-skinned with Odyssey once available.

## Output of this step

A concise problem statement (feature, users/surfaces, trigger(s), Peek resources) **plus** an
agreed-upon interactive `index.html` mockup — enough to research and plan in step 3.

## Artifacts in this folder

- `mockup-template.html` — single-file Odyssey starter for the mockup loop.
- Add requirements-brief templates or example mockups for common app types over time.
