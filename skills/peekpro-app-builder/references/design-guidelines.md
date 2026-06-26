# Peek Pro design & style guidelines — Odyssey design system

Apps that render UI should use **Odyssey**, Peek's design system, so they look and feel native
to Peek Pro. Odyssey ships **framework-agnostic web components** (`<ody-*>` tags) via npm as
`@peektravel/app-utilities`. They use light DOM, are dependency-free, and work in vanilla
HTML, React, Vue, Angular, and Svelte.

## Always load the live component docs first

Odyssey evolves, so **do not rely on memory for component names/attributes** — fetch and read
the current docs at build time:

```
https://cdn.jsdelivr.net/npm/@peektravel/app-utilities/docs/ui.md
```

It is the source of truth: every component, its tag, attributes, and usage. (This is a
"moving" detail intentionally kept out of this file so the skill doesn't go stale.)

## How to include Odyssey

### In a quick single-file mockup / static page (CDN)

```html
<head>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=IBM+Plex+Sans:wght@400;500;600&display=swap">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@peektravel/app-utilities/dist/ui/tokens.css">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@peektravel/app-utilities/dist/ui/odyssey.css">
  <script type="module" src="https://cdn.jsdelivr.net/npm/@peektravel/app-utilities/dist/ui/index.js"></script>
</head>
```

### In a bundled app (npm)

```ts
import '@peektravel/app-utilities/ui';          // registers all <ody-*> elements
import '@peektravel/app-utilities/ui/tokens.css';
import '@peektravel/app-utilities/ui/odyssey.css';
```

## Usage conventions (from ui.md)

- **Scalars → attributes:** strings/booleans set as HTML attributes
  (`<ody-button variant="primary" left-icon="plus">Add</ody-button>`).
- **Rich data → JS properties:** arrays/objects/functions set on the element object
  (`table.columns = [...]; table.data = [...]`).
- **Events → `CustomEvent`:** `addEventListener(type, e => e.detail)`.
- **Content → light-DOM children:** the component renders your child nodes.
- **Page/settings UI must be wrapped in `<ody-page-container>`** — the standard responsive
  page wrapper (designed around ~868px narrow / ~1310px wide).

## Theming / tokens

Override design tokens in CSS when needed (don't hardcode brand colors):
`--color-<name>-<shade>` (e.g. `--color-interaction-300`), typography `--ody-font-family` /
`--ody-font-weight-*`, layout `--layout-top-bar-height`, `--ody-shadow-base`. Some components
also accept inline color via attributes (e.g. `bar-color="var(--color-success-300)"`).

## Where this is used in the flow

- **Step 2** uses Odyssey to generate the interactive `index.html` mockup
  (`steps/step-2-app-purpose/mockup-template.html` is a wired-up starter).
- **Step 5** uses Odyssey to build the real client-facing and admin surfaces.

## Still open / TODO(verify)

- `TODO(verify)`: brand assets beyond the component set (logo usage rules), and any required
  layout conventions specific to the client-facing vs. admin surfaces.
- `TODO(verify)`: accessibility requirements Peek mandates for published apps.
- If Odyssey is unreachable, fall back to clean, neutral, accessible HTML and re-skin with
  Odyssey later — do **not** invent `ody-*` attributes.
