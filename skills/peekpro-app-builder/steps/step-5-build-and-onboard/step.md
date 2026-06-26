# Step 5 — Build the app AND onboard the user (in parallel)

**Goal:** once signed off (step 4), run **two tracks at the same time.** Do **not** block the
build while the user creates accounts.

> Interaction rule: when an implementation choice is genuinely ambiguous, ask **one question
> at a time** — but keep building everything that isn't blocked.

## Track A — build (you, in the background)

Scaffold and implement the app per the plan, the matching `references/<stack>.md`, and your
step-3 research. Follow the fixed-layer rules from `references/peek-api.md`:

- **Client-facing surface: auth from install settings/token — never build your own login**
  (it's installed in Peek Pro, accessed by Peek Pro users). Handle the three IDs (user ID,
  partner/account ID, install ID). The **admin surface may have its own auth** — the no-login
  rule applies only to the client-facing part.
- **Scope all data to `installDataId`** (`currentInstallDataId` on the account object).
- **SDK over raw GraphQL** (Node SDK preferred; warn if the stack forces raw GraphQL).
- **Webhooks (if used):** implement the endpoint(s) per `references/webhooks.md` — and
  **pull the live webhook doc** it links for the exact current config/parser API. Register in
  the registry/app config, **verify the delivery yourself** (the package won't), ack fast,
  idempotent handlers. Use the npm package's standard booking query + `parseBookingWebhook` /
  `parseWaiverWebhook` parsers (Node); validate fields you rely on (parsers return empty
  fields, don't throw). Remember booking events carry **state, not change**: maintain a
  seen-before store keyed on the never-changing, **normalized** booking/order IDs (scoped to
  `installDataId`) to derive created/cancelled/rescheduled, and compare stored field values to
  detect specific changes.
- Build **both surfaces**: the **client-facing** app (installed per Peek account) and the
  **admin** surface (installs, logs, ops for the developer).
- **Build the UI with Odyssey** — the same `<ody-*>` components used for the step-2 mockup
  (see `references/design-guidelines.md`; load `docs/ui.md` live for the current contract).
  - **Node (default):** import them from the **`@peektravel/app-utilities`** npm package
    (`import '@peektravel/app-utilities/ui'` + its `tokens.css` / `odyssey.css`). This is the
    preferred path and pairs with the Node-first SDK.
  - **Not Node / not using that npm package:** **fall back to the Odyssey web components via
    the CDN includes** (the `<head>` snippet) — the same components, just loaded from
    jsDelivr instead of bundled. Don't hand-roll a different UI kit.
- Use **placeholders / env vars** for any secret the user is still acquiring, so the build
  keeps moving while Track B catches up.

## Track B — onboarding walkthrough (the user, in parallel)

Give the user a clear, ordered checklist of what **they** must set up by hand, interleaved
with your progress. Use `onboarding-checklist.md` in this folder as the basis (copy it into
the user's project and tailor it to the plan). It covers, at minimum:

- **Vercel** — create the account/project; connect the repo; configure env vars.
- **Supabase** — create the project; get the project URL + anon/service keys; basic
  schema/RLS.
- **Peek credentials** — Development Hub access; register the app; the **build-time MCP**
  values `PEEK_MCP_URL` / `PEEK_MCP_TOKEN`; understand the app's **runtime** Peek tokens
  arrive via the install/settings flow (not a login the user makes).
- **Other API/secret keys** the plan calls for, and **where each goes** (host secret store /
  env — never committed).

## Keep the tracks in sync

Whenever your code starts needing a given account or key, make sure the user has the matching
setup step in front of them. Don't let either track silently get ahead of the other.

## Output of this step

A working app (both surfaces) wired to Peek with placeholders resolved as the user supplies
real credentials, ready for validation in step 6.

## Artifacts in this folder

- `onboarding-checklist.md` — the per-account/secret checklist to copy into the user's project
  and tailor. Add deployment scripts or env templates here over time.
