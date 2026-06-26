# Onboarding checklist — accounts, projects & secrets

> Track B of step 5. Copy into the user's project and tailor to the approved plan. The user
> does these by hand **in parallel** with the build. Tick items as they're completed; never
> commit secret values — store them in the host's secret store / env.

## Hosting — Vercel (default)
- [ ] Create a Vercel account / team.
- [ ] Create a project and connect the app's git repo.
- [ ] Add environment variables (filled in as the items below are obtained).
- [ ] Note the production + preview URLs (needed for the install/webhook endpoints).

## Database — Supabase (default)
- [ ] Create a Supabase project.
- [ ] Copy the **Project URL** → `SUPABASE_URL`.
- [ ] Copy the **anon key** → `SUPABASE_ANON_KEY` (client) and **service role key** →
      `SUPABASE_SERVICE_ROLE_KEY` (server only — never expose to the client).
- [ ] Apply the schema; enable Row Level Security; scope rows to `installDataId`.

## Peek — Development Hub & app registration
- [ ] Get access to the Peek **Development Hub**.  *(TODO(verify): Hub URL.)*
- [ ] Register the app; obtain its **app ID** and **security keys**.
- [ ] Confirm the **sandbox** environment for testing (use it before production).
- [ ] Configure the app's **install endpoint** and **webhook endpoint** URLs (from Vercel).

## Peek — build-time MCP (this plugin)
- [ ] Obtain `PEEK_MCP_URL` and `PEEK_MCP_TOKEN` from the Development Hub.
- [ ] Export them in the environment so the `peek-pro` MCP can start:
      `export PEEK_MCP_URL=… ; export PEEK_MCP_TOKEN=…`
- [ ] *(If the MCP backend isn't live yet, leave unset — the skill falls back to references.)*

## Runtime Peek auth (understand, nothing to create)
- [ ] Confirm understanding: the app's **runtime** Peek auth tokens arrive per-install via the
      **install/settings** flow — the app does **not** create its own login.

## Other keys (from the plan)
- [ ] <e.g. email/SMS provider for waitlist notifications> → `<ENV_VAR>`
- [ ] <payment/other API keys, if any> → `<ENV_VAR>`

## Final secret hygiene
- [ ] All secrets live in Vercel/Supabase env or a secret manager — none in the repo or
      client bundle.
- [ ] No PII or tokens written to logs.
