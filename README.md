# Peek Pro App Builder

A Claude Code **plugin** that helps developers build **apps that extend Peek Pro** — the
tours & activities booking platform. It bakes in Peek's (sparsely documented) app model and
reconciles it with up-to-date best practices for whatever stack you choose, so you ship a
correct integration instead of guessing.

Typical apps: waitlist, abandoned-booking recovery, dynamic pricing, custom checkout,
reseller/channel sync, custom reporting — anything Peek Pro doesn't do natively.

## Install

```text
/plugin marketplace add peek-travel/peekpro-builder
/plugin install peekpro-builder@peek-pro
```

Then just describe what you want to build ("build a Peek Pro waitlist app") and the
`peekpro-app-builder` skill activates automatically.

## How it works: three layers

This plugin deliberately splits knowledge by **how often it changes**, so it stays correct
over time:

| Layer | Where it lives | Why |
| --- | --- | --- |
| **Fixed** — stable Peek rules (install handshake, auth-via-settings, two surfaces, webhooks, "no raw GraphQL", PII handling) | **Baked** into `skills/peekpro-app-builder/references/` | Rarely changes; safe to bundle so it's never hallucinated |
| **Moving** — current best practices for your hosting/database/language | **Researched live** via web search at build time | Platform docs age fast; baking them goes stale in weeks |
| **Hybrid** — volatile Peek facts (live GraphQL schema, webhook/event catalog, install & settings contract, Node SDK surface, "what's available now") | **Served by the Peek MCP** (`.mcp.json`) | Peek-internal but changes often; kept out of static docs so the skill doesn't rot |

The skill's real job is the **synthesis**: gather your stack → research current best
practices → load Peek's fixed rules → query the MCP for live Peek facts → produce one
opinionated, security-first recommendation and build it.

> **Static vs. live, in one line:** skills are *instructions*; the MCP *executes / fetches
> live*. No live API calls live in the skill, and no static docs live in the MCP.

## Build-time only (the apps you build don't depend on this plugin)

The bundled MCP is a **build-time helper** — it helps Claude generate correct code. The app
you ship talks to Peek **directly** (via the Node SDK) and has **no runtime dependency** on
this plugin or the MCP.

## Stack support

- **Node / TypeScript is first-class** — the only language with Peek's API-translation SDK
  today. Strongly preferred.
- **Other languages (Python, Rails, …)** have stub guides and a clear warning: with no SDK,
  you'd be on **raw GraphQL**, which Peek discourages (it can harm the installed account's
  infrastructure). The skill recommends Node and explains the trade-off. As SDKs ship for
  more languages, this repo and skill will be updated.

## Configure the Peek MCP

`.mcp.json` declares a remote (HTTP) Peek MCP server using two placeholders:

- `PEEK_MCP_URL` — base URL of your Peek MCP backend (issued by the Peek **Development Hub**).
- `PEEK_MCP_TOKEN` — bearer token for that backend.

Set them as environment variables before launching, e.g.:

```bash
export PEEK_MCP_URL="https://<your-peek-mcp-backend>"
export PEEK_MCP_TOKEN="<token-from-the-development-hub>"
```

> **The Peek MCP backend is under active development.** Until it's live, leave `PEEK_MCP_URL`
> unset — the skill detects the server is unavailable, falls back to the baked references,
> and flags anything that would normally be a live lookup as "verify before shipping." The
> plugin is fully installable in the meantime.

## Filling the TODO placeholders

The reference files use two markers where real Peek details are still needed:

- **`ASK THE MCP`** — live data the Peek MCP should serve (schema, webhook catalog, install
  & settings contract, SDK surface). Wire these up as the MCP backend comes online.
- **`TODO(verify)`** — specifics the Peek team must confirm (endpoint shapes, signing
  scheme, Development Hub URLs, design system). Replace with verified facts; don't invent.

Design/style guidelines (`references/design-guidelines.md`) are intentionally a **TBD
placeholder** until Peek's design system is published.

## Repository layout

```text
.
├── .claude-plugin/
│   ├── plugin.json          # plugin manifest (only this folder is Claude-specific)
│   └── marketplace.json     # marketplace catalog (this repo is its own marketplace)
├── .mcp.json                # remote Peek MCP server config (PEEK_MCP_URL / PEEK_MCP_TOKEN)
├── skills/
│   └── peekpro-app-builder/
│       ├── SKILL.md         # the reconciliation procedure (stack-agnostic orchestrator)
│       └── references/
│           ├── peek-api.md          # canonical Peek app/API knowledge (fixed layer)
│           ├── node.md              # Node/TS build guide (preferred)
│           ├── python.md            # Python stub + warning
│           ├── rails.md             # Rails stub + warning
│           └── design-guidelines.md # TBD placeholder
├── README.md
└── LICENSE
```

## Agent-neutral by design

The skill and references follow the open [Agent Skills](https://agentskills.io) standard and
work in other agents (Codex, Cursor) too — they're plain Markdown instructions plus an
`.mcp.json`. Only `.claude-plugin/plugin.json` and the `/plugin` install flow are
Claude-specific.

## License

MIT — see [LICENSE](./LICENSE).
