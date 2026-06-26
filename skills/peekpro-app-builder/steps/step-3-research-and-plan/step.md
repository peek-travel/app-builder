# Step 3 — Research & assemble the plan

**Goal:** with the stack (step 1) and purpose (step 2) known, gather everything needed across
the three layers and draft a concrete plan to present for sign-off (step 4).

> Interaction rule: if anything about scope or approach is ambiguous, ask **one question at a
> time** before finalizing the plan.

## 3a. Load the fixed layer (references, on demand)

Read with the Read tool — only what applies, to keep context lean:

- **Always:** `references/peek-api.md` — the Peek app model, the three IDs + `installDataId`
  scoping, auth, resources, webhooks, SDK, GraphQL caution.
- **If the app listens to events:** `references/webhooks.md` — the booking/waiver webhook
  contract (registry setup, registry-defined GraphQL query, npm standard query + booking-model
  parser, and the "events carry state, not change" caveat).
- **Matching stack:** `references/node.md` (preferred) or `references/python.md` /
  `references/rails.md`.
- **If building UI:** `references/design-guidelines.md`.

## 3b. Query the hybrid layer (the Peek MCP) for THIS purpose

Call the bundled `peek-pro` MCP for the live facts the feature needs — never guess these
(they're the `ASK THE MCP` markers in the references):

- which **webhooks/events** fire for the feature, and their payload shapes + signature scheme;
- which **APIs / SDK methods / tools** are available to read/change the needed resources;
- the relevant **GraphQL schema** and the **install/settings/auth** contract (incl. the exact
  field names for the three IDs).

If the MCP is **unconfigured or unreachable** (the backend is still in development): fall back
to the references and **flag every would-be lookup** as "verify against the Peek MCP /
Development Hub before shipping." Never silently fill a gap with a guess.

## 3c. Research the moving layer (live web search)

Web-search the **current**, security-first setup for the chosen stack/host — project
structure, secrets management, deployment, real-time wiring if needed. Peek data can include
**sensitive PII**, so weigh storage/logging/transit. Prefer official docs from the last ~12
months. Platform specifics are intentionally not baked into references — look them up now.

## 3d. Map the data flow (do this as rigorously as the UI)

Planning has **two equally important halves.** Step 2 mocked the **UI**; now map the **data
flow** — it is just as important, often more. For **every** piece of data the app needs, pin
down:

- **Source** — does it come from Peek (which **API/SDK call**, or which **webhook payload
  field** / npm model field), or is it app-generated/derived?
- **When** — at what point is it requested or received (on install, on a webhook, on a user
  action, on a schedule)?
- **Storage** — where and how is it stored (which table/collection + fields, scoped to
  `installDataId`, keyed on **normalized** booking/order IDs where relevant)?

**Verify availability — never assume.** Confirm each Peek-sourced field actually exists and is
obtainable via the **API / SDK / npm package / webhook payload**, using the MCP and the live
docs (and the npm package's models + standard webhook query). If something you assumed isn't
available, find the real source or change the design **now**.

> Why this matters: an app built on the assumption that a field is available — when it isn't —
> simply won't work. Catch that at planning time, not after it's built. **Don't guess data
> availability.**

## 3e. Write the plan

Use `plan-template.md` in this folder as the structure. The plan must cover **both the UI and
the data flow**:

- recommended **architecture** and how it maps onto Peek's app model (install endpoint,
  settings/auth, the two surfaces, which webhooks vs. SDK calls);
- the **data-flow map** from 3d (each item: source → when → storage → **verified available?**);
- the **data model**, explicitly **scoped to an `installDataId`** (`currentInstallDataId` on
  the account object);
- **security/PII** approach for the chosen host;
- any **conflicts** from the language gate (e.g. non-Node → raw GraphQL risk);
- the **list of accounts and keys** the user must acquire (feeds step 5, Track B);
- a tight **v1 scope**.

## Output of this step

A written plan (saved into the user's project, e.g. `PLAN.md`) ready to present in step 4.

## Artifacts in this folder

- `plan-template.md` — the structure to fill in. Copy it into the user's project and complete
  it. Add research-note templates or example plans here over time.
