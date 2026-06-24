---
name: saas-ui-redesign
description: This skill should be used when the user asks to "redesign the UI", "modernize the interface", "make it look like a SaaS app", "add a sidebar", "move the nav to the left", "use a left navigation", or "polish the UI / make it look professional" for the inventory-management Vue 3 client. It converts the app's top navigation bar into a left vertical sidebar, introduces a CSS design-token system for consistent spacing and color, and applies a polished modern-SaaS look.
---

# SaaS UI Redesign (inventory-management client)

Redesign the Vue 3 frontend in `client/` from a horizontal top-nav layout into a modern SaaS interface: a fixed **left vertical sidebar**, a **design-token system** that removes hardcoded/duplicated styling, and a **responsive drawer** for small screens. This skill is tailored to this app's actual files and conventions — follow it directly rather than inventing new patterns.

## Design principles

- **Left sidebar, not top tabs.** A fixed-width (`260px`) full-height sidebar holds the brand, vertical nav, and footer controls. The active route is marked by a left accent bar plus a soft background — not an underline.
- **Tokens over literals.** Color, spacing, radius, shadow, layout, and z-index live in CSS variables (`client/src/styles/tokens.css`), seeded from the existing palette so adoption is visually lossless. New UI references tokens; legacy components migrate incrementally.
- **One source of truth for nav.** Sidebar items are rendered from a data array (`navItems`) of `{ path, label, icon }`, keeping routes, labels, and icons in one place instead of repeated `<router-link>` markup.
- **Icons via lucide-vue-next.** Each nav item shows a lucide icon at `18px`.
- **Preserve behavior.** Keep i18n (`t()`) keys, the `useFilters` / `useAuth` / `useI18n` composables, the global modals, and the global utility classes (`.card`, `.badge`, `.stat-card`, `.table-container`, table styles) that every view depends on.

## Target files

- `client/src/styles/tokens.css` — **new**, the design-token `:root` block. Drop in the copy at `references/design-tokens.css`.
- `client/src/main.js` — import the tokens stylesheet once, before `App.vue`.
- `client/src/App.vue` — replace the `<header class="top-nav">` region with the sidebar shell; migrate the layout `<style>` to tokens; add mobile drawer styles. See `examples/AppShell.example.vue`.
- `client/src/components/FilterBar.vue` — it was sticky at `top: 70px` (old nav height); reposition to the content column.
- `client/package.json` — add `lucide-vue-next`.

## Procedure

Work top-down; verify the app still builds after each file. The granular before/after for every edit is in `references/redesign-playbook.md` — consult it while editing.

1. **Install icons.** From `client/`: `npm install lucide-vue-next`.
2. **Add tokens.** Create `client/src/styles/tokens.css` from `references/design-tokens.css`. Import it in `main.js` with `import './styles/tokens.css'` placed above `import App from './App.vue'`.
3. **Rebuild the shell in `App.vue`.** Replace the top nav with the sidebar layout from `examples/AppShell.example.vue`:
   - Change `.app` from `flex-direction: column` to `row`.
   - Add `<aside class="sidebar">` with three regions: `.sidebar-brand`, `.sidebar-nav`, `.sidebar-footer`.
   - Render nav from a `navItems` computed (see below). Move `<LanguageSwitcher />` and `<ProfileMenu>` into `.sidebar-footer`.
   - Wrap `<FilterBar />` + `<main class="main-content">` in a `.content-area` flex column.
   - Add a `.mobile-bar` (hamburger + brand) and `.sidebar-backdrop`, both hidden above `768px`.
4. **Reposition `FilterBar`.** Set `.filters-bar` sticky `top: 0` (and `top: 56px` under the mobile breakpoint), swap `z-index: 90` for `var(--z-filterbar)`, and remove the `max-width: 1600px; margin: 0 auto` centering from `.filters-container` so it aligns to the content column.
5. **Migrate touched styles to tokens** as you go; don't rewrite untouched component styles.

## Nav data array

Define in `App.vue` `setup()`. Labels use `t()` so they stay reactive to language switches; icons are wrapped in `markRaw` so Vue does not make the component definitions reactive.

```js
import { LayoutDashboard, Package, ShoppingCart, Wallet, TrendingUp, FileBarChart, Menu as MenuIcon } from 'lucide-vue-next'

const navItems = computed(() => [
  { path: '/', label: t('nav.overview'), icon: markRaw(LayoutDashboard) },
  { path: '/inventory', label: t('nav.inventory'), icon: markRaw(Package) },
  { path: '/orders', label: t('nav.orders'), icon: markRaw(ShoppingCart) },
  { path: '/spending', label: t('nav.finance'), icon: markRaw(Wallet) },
  { path: '/demand', label: t('nav.demandForecast'), icon: markRaw(TrendingUp) },
  { path: '/reports', label: 'Reports', icon: markRaw(FileBarChart) }  // no i18n key — literal, as before
])
```

Render with:

```html
<router-link v-for="item in navItems" :key="item.path" :to="item.path"
  class="nav-item" :class="{ active: $route.path === item.path }"
  @click="sidebarOpen = false">
  <component :is="item.icon" :size="18" class="nav-icon" />
  <span class="nav-label">{{ item.label }}</span>
</router-link>
```

### Nav item → icon mapping

| Route | Label key | Icon |
|-------|-----------|------|
| `/` | `nav.overview` | `LayoutDashboard` |
| `/inventory` | `nav.inventory` | `Package` |
| `/orders` | `nav.orders` | `ShoppingCart` |
| `/spending` | `nav.finance` | `Wallet` |
| `/demand` | `nav.demandForecast` | `TrendingUp` |
| `/reports` | (literal "Reports") | `FileBarChart` |

## Active state

```css
.nav-item.active { color: var(--color-primary); background: var(--color-primary-soft); }
.nav-item.active::before {
  content: ''; position: absolute; left: 0; top: 50%; transform: translateY(-50%);
  width: 3px; height: 60%; border-radius: 0 var(--radius-full) var(--radius-full) 0;
  background: var(--color-primary);
}
```

## Responsive (the app's first breakpoints)

At `<= 768px`: the sidebar becomes a fixed off-canvas drawer (`transform: translateX(-100%)`, `.open` slides it in), a `.mobile-bar` with a hamburger appears, and a `.sidebar-backdrop` closes the drawer on click. A `sidebarOpen` ref toggles it; nav clicks set it back to `false`. The `FilterBar` sticks at `top: 56px` to clear the mobile bar.

## Pitfalls

- **Do not drop i18n.** Reuse the exact `t('nav.*')` keys; only `/reports` is a literal label (it has no key in `en.js` / `ja.js`).
- **Keep the global classes.** `.card`, `.badge`, `.stat-card`, `.table-container`, and the bare `table`/`th`/`td` rules in `App.vue`'s non-scoped `<style>` are consumed by every view and modal — migrate their values to tokens but keep the selectors and computed results identical.
- **Footer dropdowns open near the bottom.** `ProfileMenu` / `LanguageSwitcher` were built to drop down from the top bar. In the sidebar footer, verify their menus remain on-screen; if they clip, have them open upward.
- **Wrap icons in `markRaw`.** Skipping this makes Vue warn about making component definitions reactive.
- **Z-index order.** Use the token scale (`--z-filterbar` < `--z-sidebar` < `--z-dropdown` < `--z-modal`) instead of the old ad-hoc `90 / 100 / 2000`.

## Verification

1. `cd client && npm install && npm run dev` (frontend on `:3000`); ensure the FastAPI backend runs on `:8001`. Confirm a clean build with no console errors.
2. With Playwright MCP against `http://localhost:3000`, confirm: the left sidebar renders with logo + 6 icon nav items + footer controls; each item routes and shows the active highlight; all six views load with data; `LanguageSwitcher` and `ProfileMenu` still work from the footer; `FilterBar` filters still drive the views; and at ~375px wide the sidebar collapses to a working drawer.
3. Capture before/after screenshots.
