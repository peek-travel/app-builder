# Step 6 — Validate

**Goal:** confirm the app actually works against Peek and is safe to ship.

> Interaction rule: if a validation result is ambiguous or something fails, surface it and ask
> **one question at a time** about how to proceed.

## Do

- **Sandbox first.** Where the MCP supports it, validate generated calls against a Peek
  **sandbox** account before suggesting production. Never test against a live account.
- **Webhooks:** confirm signature verification works and handlers are **idempotent** (replay
  a test event).
- **Install flow:** exercise the install endpoint with a test payload; confirm the three IDs
  are captured and a fresh `installDataId` / `currentInstallDataId` is created.
- **Reinstall behavior:** confirm a reinstall starts from a clean slate (new `installDataId`)
  and that the uninstall data-wiper targets the prior `installDataId`.
- **Secrets:** confirm none are committed; all live in the host secret store / env.
- **PII:** confirm storage/logging/transit match the plan (no PII or tokens in logs).
- **Both surfaces:** smoke-test the client-facing app and the admin surface.

## If the MCP is unavailable

Validate what you can locally, and clearly list every **TODO(verify)** the user must confirm
against the Peek MCP / Development Hub before going to production. Don't claim it's verified
when it isn't.

## Output of this step

A validated app with a short report of what passed and any remaining TODO(verify) items.

## Artifacts (optional)

- A **validation checklist** or test-payload fixtures can live in this folder and be copied
  into the user's project.
