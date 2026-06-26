# Peek Pro app plan — <app name>

> Copy this into the user's project (e.g. `PLAN.md`), fill it in during step 3, and use it to
> get sign-off in step 4. Replace every `<…>` and resolve every TODO before building.

## 1. Summary
- **What it does:** <one-line purpose>
- **Who uses it:** <account staff / guests / app admin>
- **Trigger(s):** <webhook event(s) / schedule / user action>

## 2. Stack
- **Language/framework:** <e.g. Next.js (React + TS)>
- **Hosting:** <e.g. Vercel>
- **Database:** <e.g. Supabase/Postgres>
- **Real-time:** <e.g. Supabase Realtime — or N/A>
- **Language-gate note:** <Node = first-class; or WARNING if non-Node + raw-GraphQL risk>

## 3. Peek integration (confirm via the MCP — mark TODO(verify) if MCP unavailable)
- **Webhooks/events consumed:** <event names + payloads> — `ASK THE MCP`
- **API / SDK calls made:** <SDK methods / operations> — `ASK THE MCP`
- **Resources touched:** <products / availability / bookings / orders / payments / customers>
- **Auth:** identity from install settings/token (NO own login). Three IDs handled: user ID,
  partner/account ID, install ID.

## 4. Surfaces
- **Client-facing:** <what the installing account/guests see>
- **Admin:** <what the app developer sees — installs, logs, ops>

## 5. Data flow (verify EVERY Peek-sourced item — no assumptions)
For each piece of data the app needs, map source → when → storage → and confirm it's actually
available. Don't list a field you haven't verified exists.

| Data item | Source (Peek API/SDK call · webhook field · app-derived) | When (install / webhook / user action / schedule) | Stored where & how (table.field) | Verified available? |
| --- | --- | --- | --- | --- |
| <e.g. guest count> | <booking webhook → parseBookingWebhook → guests> | on booking webhook | `bookings.guest_count` | ✅ webhooks doc / npm model |
| <…> | <…> | <…> | <…> | ⬜ TODO verify via MCP/doc |

> Any row not marked verified must be resolved (find the real source or change the design)
> **before sign-off**. An app built on assumed-but-absent data won't work.

## 6. Data model (scoped to installDataId)
- `installDataId = installId + installTimestamp`; `currentInstallDataId` stored on the account
  object; **all records scoped to `installDataId`**, keyed on **normalized** booking/order IDs
  where relevant.
- **Entities:** <tables/collections and their fields>
- **PII handling:** <what is stored vs. referenced by Peek ID; encryption; retention>

## 7. Security
- Secret storage: <host secret manager>; no secrets in repo/client bundle.
- Webhook delivery verification: <you verify it — scheme: ASK THE MCP>. Idempotent handlers.

## 8. Accounts & keys the user must acquire (drives step 5, Track B)
- [ ] <Hosting account, e.g. Vercel>
- [ ] <Database, e.g. Supabase project + URL/keys>
- [ ] <Peek Development Hub access + app registration>
- [ ] <Build-time MCP: PEEK_MCP_URL / PEEK_MCP_TOKEN>
- [ ] <Any other API keys>

## 9. v1 scope (and explicit non-goals)
- **In:** <…>
- **Out (later):** <…>

## 10. Open questions / risks
- <anything to confirm with the user or the Peek team>
