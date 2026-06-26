# Step 4 — Get sign-off

**Goal:** get the user's **explicit approval of the plan before any building.** This is a
hard gate — do not start step 5 without it.

> Interaction rule: ask **one question at a time** to resolve any disagreement; revise the
> plan until the user is satisfied.

## Do

1. Present the plan from step 3 (the filled-in `plan-template.md` / `PLAN.md`) clearly and
   concisely — architecture, Peek integration, surfaces, data model (installDataId scoping),
   security, accounts/keys, v1 scope.
2. Call out explicitly:
   - any **language-gate conflict** (non-Node → raw-GraphQL risk),
   - anything that is **MCP-unverified** (flagged TODO(verify) because the MCP was
     unavailable),
   - the **accounts/keys** the user will need to set up (preview of step 5, Track B).
3. Ask for changes; incorporate feedback; re-present if the plan materially changes.
4. Get an explicit "yes, build this."

## Output of this step

A signed-off plan. Only then proceed to step 5.

## Artifacts (optional)

- Keep a short **decision log** of what was approved / changed here if useful for the admin
  surface or future updates.
