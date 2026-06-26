# Step 1 — Stack & technology selection

**Goal:** lock the language/framework, hosting platform, and database **before** discussing
the app itself. Everything downstream (which reference loads, the language gate, the security
research) depends on this.

> Interaction rule: ask **one question at a time** and let the user discuss options with you.
> Don't move to step 2 until the stack is settled.

## Ask for

1. **Language / framework** — what they want to build in.
2. **Hosting platform** — e.g. Vercel, Firebase, Fly.io, Cloudflare, AWS, Render, …
3. **Database** — e.g. Supabase/Postgres, Firestore, PlanetScale/MySQL, Mongo, …

## If the user is unsure or has no preference — recommend this default

> **Next.js (React + TypeScript) on Vercel, with Supabase (Postgres) for data — plus React
> client components + Supabase Realtime for any real-time features** (e.g. a live-updating
> waitlist or availability board).

Why this default:
- **Next.js is Node/TypeScript** → it gets Peek's **first-class Node SDK** (see the language
  gate below).
- **Vercel + Supabase** stand up fast, have strong security defaults, and pair cleanly with
  Next.js (server routes for the install/webhook endpoints, Supabase for data + auth-adjacent
  needs).
- **React + Supabase Realtime** covers live UI when the feature needs it.

Offer it as a recommendation, not a mandate — let the user confirm or steer.

## The language gate (apply after the stack is chosen)

- **Node / TypeScript is first-class.** Peek currently ships its API-translation SDK for
  **Node only**. Strongly prefer it. (The default above already satisfies this.)
- **Any other language (Python, Ruby/Rails, Go, …):** there is **no Peek SDK yet**, so
  talking to Peek means **raw GraphQL — which Peek explicitly discourages** (misuse can harm
  the installed account's infrastructure). **Warn the user clearly**, explain the trade-off,
  and recommend Node. If they still proceed, continue with caveats and note it in the plan.

## Output of this step

A confirmed stack (language/framework + host + database) and an explicit note of any
language-gate warning. Record it so step 3's plan can build on it.

## Artifacts (optional, drop them in this folder)

- A short **stack decision record** template, or example stack write-ups for common
  combinations, can live here as the catalog grows.
