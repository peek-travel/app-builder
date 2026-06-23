---
name: peekpro-app-builder
description: >-
  Build an app that extends Peek Pro. Use when the user wants to build a Peek Pro app,
  integrate with the Peek Pro API, create a Peek booking/tours/activities integration,
  add functionality Peek Pro lacks (waitlist, abandoned-booking recovery, dynamic pricing,
  custom checkout, reseller/channel sync, reporting), handle Peek webhooks, use the Peek
  GraphQL API or Peek Node SDK, register an app in the Peek Development Hub, or publish to
  the Peek App Store. Triggers on "Peek Pro", "PeekPro", "Peek app", "Peek booking",
  "Peek integration", "Peek API", "Peek webhook".
---

# Build a Peek Pro app

You are helping the user build an **app that extends Peek Pro** — a tours & activities
booking platform. Apps add functionality Peek Pro lacks natively (waitlist, abandoned
bookings, dynamic pricing, custom flows, reporting, reseller sync, …).

Peek Pro is **sparsely documented**, so do not rely on model memory for Peek specifics.
This skill carries the canonical knowledge and tells you when to look things up live.

## The one thing to understand: this skill reconciles three layers

Your job is **synthesis**, not recital. Three sources of truth meet at build time:

1. **Fixed layer — baked references (stable Peek rules).** How Peek apps *work*: the
   install handshake, auth-via-settings, sandbox/prod, the two surfaces, webhooks + the
   Node SDK, "never touch raw GraphQL," PII handling. Loaded from `references/`. Trust these.
2. **Moving layer — live web search (current stack best practices).** Best practices for
   *whatever* hosting/database/language the user picks — including stacks you've never heard
   of. This ages fast, so **research it fresh at build time**. Never bake it.
3. **Hybrid layer — the Peek MCP (volatile Peek facts).** Peek's own frequently-changing
   integration data: current GraphQL schema, webhook/event catalog, install + settings
   payload shapes, Node SDK surface, "what's available to integrate right now." This lives
   in the **MCP**, not the references, so the skill never goes stale. Query it for anything
   marked "ask the MCP."

Produce **one correct, opinionated recommendation** by reconciling all three, and **surface
conflicts honestly** (e.g. "you picked a Python/Django host, but the only Peek SDK is Node —
here's the risk and what I'd change").

## Procedure

Follow these steps in order. Do not skip step 1.

### 1. Gather the stack UP FRONT (ask, don't assume)

Ask the user, before writing anything:

- **Language / framework** — what they want to build in.
- **Hosting platform** — e.g. Vercel, Firebase, Fly.io, Cloudflare, AWS, Render, …
- **Database** — e.g. Supabase/Postgres, Firestore, PlanetScale/MySQL, Mongo, …

Then apply the **language gate**:

- **Node/TypeScript is first-class.** Peek currently ships its API-translation SDK for
  **Node only**. Strongly prefer it. Load `references/node.md`.
- **Any other language (Python, Ruby/Rails, Go, …):** there is **no Peek SDK yet**, which
  means talking to Peek would require **raw GraphQL — which Peek explicitly discourages**
  because misuse can harm the installed account's infrastructure. **Warn the user clearly**,
  explain the trade-off, and recommend Node. If they still proceed, load the matching
  `references/<lang>.md` stub and treat raw-GraphQL guidance as best-effort with caveats.

### 2. Load only what you need (progressive disclosure)

Read these reference files with the Read tool — only the ones that apply, to keep context lean:

- **Always:** `references/peek-api.md` — the Peek app model, auth, resources, webhooks, SDK,
  GraphQL caution. This is the fixed layer.
- **Matching stack only:** `references/node.md` (preferred) or `references/python.md` /
  `references/rails.md` (with warnings per step 1).
- **When building UI:** `references/design-guidelines.md` — Peek's design/style conventions.

Do **not** preload every reference. Load on demand.

### 3. Research the moving layer (live, every build)

Web-search for **current** best practices for the user's exact stack at build time —
project structure, secrets management, deployment, and especially **security**, because
Peek data can include **sensitive PII** (guest names, contact info, payment metadata).
Platform specifics (Vercel/Firebase/Fly/…) change constantly and are deliberately **not**
baked into references — look them up now. Prefer official docs from the last ~12 months.

### 4. Query the hybrid layer (the Peek MCP) for live Peek facts — never guess

For anything Peek-specific that changes — **current GraphQL schema, the webhook/event
catalog and payload shapes, the install/settings contract, the Node SDK surface, what's
available to integrate** — call the bundled **`peek-pro` MCP** instead of inventing it.
The references mark these spots with **"ASK THE MCP."**

If the MCP is **not configured or unreachable** (the backend is still in development): say
so, fall back to the baked references, and **explicitly flag every value that would normally
be a live lookup** as "verify against the Peek MCP / Development Hub before shipping." Never
silently fill a TODO with a guess.

### 5. Reconcile → recommend → build

1. State the recommended architecture in a few lines: stack + how it maps onto Peek's app
   model (install endpoint, settings/auth, which surfaces, webhooks vs. SDK calls).
2. Call out any **conflicts** from the language gate or security review.
3. Scaffold the project following the stack reference + your live research.
4. Wire Peek integration per `peek-api.md` (SDK over raw GraphQL; auth from settings; verify
   webhook signatures; never build your own login).
5. Build **both surfaces** the app needs: the **client-facing** side (installed per Peek
   account) and the **admin** side (for the app developer — installs, logs, ops).

### 6. Validate

Where the MCP supports it, validate generated calls against a **sandbox** account before
suggesting production. Confirm secrets are not committed and PII handling matches step 3.

## Hard rules (do not violate)

- **Never build the app's own login/auth.** Identity comes from Peek via the install
  settings/token. The app always knows who's calling because Peek tells it.
- **Prefer the Node SDK; avoid raw GraphQL.** Raw GraphQL against an installed account is
  risky. If no SDK exists for the chosen language, warn rather than quietly hand-roll it.
- **Treat Peek data as sensitive PII.** Security-first choices for storage, logging, transit.
- **Don't invent Peek endpoint/schema/event details.** If it's volatile, ask the MCP; if the
  MCP is down, flag it as TODO-verify. Stable rules live in `references/`.

## Agent-neutral note

The instructions and references here follow the open Agent Skills standard and work in other
agents (Codex, Cursor) too. Only `.claude-plugin/plugin.json` and the install flow are
Claude-specific.
