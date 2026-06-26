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
   install handshake, the three IDs + `installDataId` data scoping, auth-via-settings,
   sandbox/prod, the two surfaces, webhooks + the Node SDK, "never touch raw GraphQL," PII
   handling. Loaded from `references/`. Trust these.
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
several questions into one message. Only proceed once the current question is resolved.

## Steps (this skill is organized as a sequence of step folders)

The full instructions live under `steps/`, one folder per step. Each folder contains a
`step.md` with everything about that step — and room for **artifacts/templates** (plan
templates, checklists, schemas, scripts) used by that step.

**How to run them:** do the steps in order. **When you begin a step, `Read` that step's
`step.md` first**, then follow it. Load only the current step's file (and the references it
points to) to keep context lean — this is the progressive-disclosure pattern. Don't skip
steps 1–2, and don't build before sign-off (step 4).

| Step | Folder | What it covers |
| --- | --- | --- |
| 1 | `steps/step-1-stack-selection/step.md` | Ask the stack/tech first; recommend a default if unsure; apply the Node-first language gate. |
| 2 | `steps/step-2-app-purpose/step.md` | Discover what the app should accomplish, who uses it, what triggers it. |
| 3 | `steps/step-3-research-and-plan/step.md` | Load references, query the MCP, research the stack live, and draft the plan (uses `plan-template.md`). |
| 4 | `steps/step-4-sign-off/step.md` | Present the plan and get explicit user approval before building. |
| 5 | `steps/step-5-build-and-onboard/step.md` | Build the app (Track A) **in parallel** with walking the user through account/secret setup (Track B; uses `onboarding-checklist.md`). |
| 6 | `steps/step-6-validate/step.md` | Validate against a Peek sandbox; confirm secrets and PII handling. |

The `references/` files are the fixed-layer knowledge those steps pull in:
`peek-api.md` (always), `node.md`/`python.md`/`rails.md` (matching stack), and
`design-guidelines.md` (only when building UI).

## Hard rules (do not violate)

- **Never build the app's own login/auth.** Identity comes from Peek via the install
  settings/token. The app always knows who's calling because Peek tells it.
- **Prefer the Node SDK; avoid raw GraphQL.** Raw GraphQL against an installed account is
  risky. If no SDK exists for the chosen language, warn rather than quietly hand-roll it.
- **Treat Peek data as sensitive PII.** Security-first choices for storage, logging, transit.
- **Scope all app data to an `installDataId`.** Peek passes three IDs (user ID, partner/
  account ID, install ID); the **install ID does not rotate across reinstalls**. Mint
  `installDataId = install ID + timestamp`, track it as `currentInstallDataId` on the account,
  and scope every record to it. See `references/peek-api.md` "Identity & data scoping."
- **Don't invent Peek endpoint/schema/event details.** If it's volatile, ask the MCP; if the
  MCP is down, flag it as TODO-verify. Stable rules live in `references/`.
- **Don't build before sign-off (step 4), and don't gather the app's purpose before the
  stack (steps 1 → 2).**
- **When unclear, ask — one question at a time.** Never assume (see "How to interact").

## Agent-neutral note

The instructions and references here follow the open Agent Skills standard and work in other
agents (Codex, Cursor) too. Only `.claude-plugin/plugin.json` and the install flow are
Claude-specific.
