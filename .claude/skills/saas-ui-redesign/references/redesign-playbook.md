# Redesign Playbook — top-nav → SaaS sidebar

Granular, file-by-file steps for converting the inventory-management `client/`
from a horizontal top nav to a left sidebar with a design-token system. Read
this alongside `SKILL.md`; copy structure from `examples/AppShell.example.vue`
and tokens from `references/design-tokens.css`.

## 0. Baseline facts about this app

- Vue 3.4 + Vue Router 4.3, Vite 5, no CSS framework or component library.
- `client/src/App.vue` holds the only **non-scoped** `<style>` — it defines the
  layout plus global utility classes every view reuses: `.page-header`,
  `.stats-grid`, `.stat-card`, `.card`, `.card-header`, `.card-title`,
  `.table-container`, bare `table`/`thead`/`th`/`td`, `.badge` (+ variants),
  `.loading`, `.error`. **These selectors must keep working.**
- Routes live in `client/src/main.js`: `/`, `/inventory`, `/orders`,
  `/demand`, `/spending`, `/reports`.
- Global state via composables: `useFilters`, `useAuth`, `useI18n`.
- Global components rendered in `App.vue`: `FilterBar`, `LanguageSwitcher`,
  `ProfileMenu`, and six modals.
- Palette was hardcoded and duplicated (no CSS variables, no media queries).

## 1. `npm install lucide-vue-next` (run in `client/`)

Adds `lucide-vue-next` to `dependencies`. It ships tree-shakeable single-file
icon components, so importing only the six used icons keeps the bundle small.

## 2. `client/src/styles/tokens.css` (new) + import

Create the file from `references/design-tokens.css`. Token groups: text colors,
surface/line colors, brand/state colors (each with a `-soft` companion),
`--space-1..12`, `--radius-sm/md/lg/full`, `--shadow-sm/md/lg/xl`,
`--font-sans`, `--sidebar-width: 260px`, `--transition-base`, and the z-index
scale `--z-filterbar(80) < --z-sidebar(100) < --z-dropdown(1000) < --z-modal(2000)`.

In `client/src/main.js`, add **above** `import App from './App.vue'`:

```js
import './styles/tokens.css'
```

Importing before `App.vue` ensures `:root` variables resolve when App's styles
load.

## 3. `client/src/App.vue`

### 3a. Template
Replace the entire `<header class="top-nav">…</header>` + sibling
`<FilterBar />` + `<main>` region with the shell from
`examples/AppShell.example.vue`:
- `.mobile-bar` (hamburger + brand) — first child, hidden on desktop.
- `.sidebar-backdrop` — `v-if="sidebarOpen"`.
- `<aside class="sidebar">` → `.sidebar-brand`, `.sidebar-nav` (v-for over
  `navItems`), `.sidebar-footer` (LanguageSwitcher + ProfileMenu).
- `.content-area` wrapping `<FilterBar />` and `<main class="main-content">`.
- Leave the two modal elements (`ProfileDetailsModal`, `TasksModal`) exactly
  as they were.

### 3b. Script
- Extend the Vue import to `import { ref, onMounted, computed, markRaw } from 'vue'`.
- Add the lucide import (six icons + `Menu as MenuIcon`).
- Register `MenuIcon` in `components`.
- Add `const sidebarOpen = ref(false)` and the `navItems` computed (see SKILL.md).
- Return `sidebarOpen` and `navItems` alongside the existing returns.
- Do **not** touch the task logic (`loadTasks`/`addTask`/`deleteTask`/`toggleTask`).

### 3c. Styles
Replace only the layout rules (`body`, `.app`, `.top-nav`, `.nav-container`,
`.logo`, `.subtitle`, `.nav-tabs*`, `.main-content`) with the sidebar +
`.content-area` + mobile rules from the example. Keep every utility class
below it; convert their literal values to tokens where a token matches exactly
(e.g. `#e2e8f0` → `var(--color-border)`, `10px` radius → `var(--radius-lg)`),
leaving values without an exact token as literals to avoid visual drift.

Key structural changes:
- `.app`: `flex-direction: column` → `row`.
- `.main-content`: drop `max-width: 1600px; margin: 0 auto`; keep `flex: 1`
  and token padding so it fills the space beside the sidebar.

## 4. `client/src/components/FilterBar.vue` (scoped style)

- `.filters-bar`: `top: 70px` → `top: 0`; `z-index: 90` → `var(--z-filterbar)`;
  tokenize `background`/`border`.
- `.filters-container`: remove `max-width: 1600px; margin: 0 auto`; keep
  horizontal padding (use `var(--space-8)`).
- Add a `@media (max-width: 768px)` block: `.filters-bar { top: 56px }` and
  `.filters-container { padding: 0 var(--space-4); overflow-x: auto }` so the
  filter row clears the mobile bar and scrolls instead of wrapping.

## 5. Verify

Build (`npm run dev`), then walk the checklist in SKILL.md's Verification
section with Playwright MCP. Watch specifically for: footer dropdown clipping,
any `t('nav.*')` key that renders blank, and console warnings about making a
component reactive (means an icon wasn't `markRaw`-wrapped).

## Optional follow-ups (not required)

- Migrate remaining components' hardcoded colors to tokens for full consistency.
- Add an icon-only collapsed rail on desktop (toggle `--sidebar-width`).
- Add a `nav.reports` i18n key to `en.js` / `ja.js` and switch the literal label.
