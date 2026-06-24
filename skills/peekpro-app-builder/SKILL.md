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
   marked "ASK THE MCP."

Produce **one correct, opinionated recommendation** by reconciling all three, and **surface
conflicts honestly** (e.g. "you picked a Python/Django host, but the only Peek SDK is Node —
here's the risk and what I'd change").

## How to interact (applies to every step)

**Always ask when something is unclear — never guess or assume.** Whenever the goal, scope,
stack, a Peek detail, or the right approach is ambiguous, stop and ask the user.

**Ask one question at a time.** Pose a single question, share the relevant options and your
recommendation, and let the user chat it through with you before moving on. Don't batch
several questions into one message — a back-and-forth conversation per question produces
better answers than a long questionnaire. Only proceed once the current question is resolved.

## Procedure

Follow these steps in order. Do not skip steps 1 and 2, and do not build before sign-off
(step 4).

### 1. Stack & technology FIRST (ask before anything else)

Before discussing the app itself, ask the user what they want to build on:

- **Language / framework**
- **Hosting platform** (e.g. Vercel, Firebase, Fly.io, Cloudflare, AWS, …)
- **Database** (e.g. Supabase/Postgres, Firestore, PlanetScale, …)

**If the user is unsure or has no preference, recommend this default and explain why in a
line or two:**

> **Next.js (React + TypeScript) on Vercel, with Supabase (Postgres) for data — and React
> client components + Supabase Realtime for any real-time features** (e.g. a live-updating
> waitlist or availability view).

This default is deliberate: Next.js is **Node/TypeScript**, so it gets Peek's first-class
**Node SDK**; Vercel + Supabase are fast to stand up and have strong defaults; React/Supabase
Realtime covers live UI when the app needs it.

Then apply the **language gate**:

- **Node/TypeScript is first-class** (the default above satisfies this). Peek currently ships
  its API-translation SDK for **Node only**. Strongly prefer it.
- **Any other language (Python, Ruby/Rails, Go, …):** there is **no Peek SDK yet**, so
  talking to Peek means **raw GraphQL — which Peek explicitly discourages** (misuse can harm
  the installed account's infrastructure). **Warn the user clearly**, explain the trade-off,
  and recommend Node. If they still proceed, continue with caveats.

### 2. Identify the app's purpose

Now ask **what they're trying to accomplish** — the feature/problem (waitlist, abandoned-
booking recovery, dynamic pricing, custom checkout, reporting, …), who uses it, and what
should trigger it. Get enough detail to design: the user-facing behavior, the events that
drive it, and what it needs to read or change in Peek.

### 3. Research and assemble the plan

With the stack and purpose known, gather everything needed and draft a concrete plan:

- **Load references on demand (progressive disclosure)** — read with the Read tool, only
  what applies, to keep context lean:
  - **Always:** `references/peek-api.md` (the fixed Peek rules).
  - **Matching stack:** `references/node.md` (preferred) or `references/python.md` /
    `references/rails.md`.
  - **If building UI:** `references/design-guidelines.md`.
- **Hybrid layer — query the Peek MCP for live facts this purpose needs:** which
  **webhooks/events** fire for the feature, which **APIs / SDK methods / tools** are
  available, the relevant **schema**, and the install/settings/auth contract. These are the
  `ASK THE MCP` markers in the references — never guess them. If the MCP is **unconfigured or
  unreachable**, fall back to the references and **flag each would-be lookup** as "verify
  against the Peek MCP / Development Hub before shipping."
- **Moving layer — research current best practices live:** web-search the up-to-date,
  security-first setup for the chosen stack/host (project structure, secrets, deployment).
  Peek data can include **sensitive PII**, so weigh storage/logging/transit carefully.

Then write the **plan**: recommended architecture; how it maps onto Peek's app model
(install endpoint, settings/auth, the two surfaces, which webhooks vs. SDK calls); the data
model **scoped to an `installDataId`** (see the identity/data-scoping rule below and in
`peek-api.md`); security/PII approach; any conflicts from the language gate; and the **list of
accounts and keys the user will need to acquire**.

### 4. Get sign-off

Present the plan and get the user's **explicit approval before building**. Incorporate their
feedback and revise until they sign off.

### 5. Build the app AND onboard the user — in parallel

Once signed off, run **two tracks at the same time**. Do **not** block the build waiting for
the user to finish creating accounts.

- **Track A — build (you, in the background):** scaffold and implement the app per the plan,
  the stack reference, and your live research. Wire Peek integration per `peek-api.md` (SDK
  over raw GraphQL; identity from install settings; verify webhook signatures; never build
  your own login). Build **both surfaces**: the **client-facing** app (installed per Peek
  account) and the **admin** surface (for the developer — installs, logs, ops). Use
  placeholders/env vars for any secret the user is still acquiring so the build keeps moving.
- **Track B — onboarding walkthrough (the user, in parallel):** give the user a clear,
  ordered checklist of what *they* must set up by hand, interleaved with your progress:
  - **Vercel** — create the account/project; how to connect the repo and configure env vars.
  - **Supabase** — create the project; where to get the project URL and the anon/service
    keys; basic schema/RLS setup.
  - **Peek credentials** — obtain Development Hub access; the **build-time MCP** values
    (`PEEK_MCP_URL`, `PEEK_MCP_TOKEN`); and understand that the **app's** Peek auth tokens
    arrive at runtime via the install/settings flow (not a login the user creates).
  - **Other API/secret keys** the plan calls for, and **where each one goes** (the host's
    secret store / env — never committed).

  Keep the tracks in sync: whenever your code starts needing a given account or key, make
  sure the user has the matching setup step in front of them.

### 6. Validate

Where the MCP supports it, validate generated calls against a **sandbox** account before
suggesting production. Confirm secrets aren't committed and PII handling matches the plan.

## Hard rules (do not violate)

- **Never build the app's own login/auth.** Identity comes from Peek via the install
  settings/token. The app always knows who's calling because Peek tells it.
- **Prefer the Node SDK; avoid raw GraphQL.** Raw GraphQL against an installed account is
  risky. If no SDK exists for the chosen language, warn rather than quietly hand-roll it.
- **Treat Peek data as sensitive PII.** Security-first choices for storage, logging, transit.
- **Scope all app data to an `installDataId`.** Peek passes three IDs (user ID, partner/
  account ID, install ID); the **install ID does not rotate across reinstalls**. Mint
  `installDataId = install ID + timestamp`, track it as `currentInstallDataId` on the account,
  and scope every record to it — so reinstalls start clean and an uninstall wiper knows what
  to delete. See `peek-api.md` "Identity & data scoping."
- **Don't invent Peek endpoint/schema/event details.** If it's volatile, ask the MCP; if the
  MCP is down, flag it as TODO-verify. Stable rules live in `references/`.
- **Don't build before sign-off (step 4), and don't gather the app's purpose before the
  stack (steps 1 → 2).**
- **When unclear, ask — one question at a time.** Never assume; let the user discuss each
  question and its options with you before moving on (see "How to interact").

## Agent-neutral note

The instructions and references here follow the open Agent Skills standard and work in other
agents (Codex, Cursor) too. Only `.claude-plugin/plugin.json` and the install flow are
Claude-specific.
