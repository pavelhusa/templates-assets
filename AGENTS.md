# Agent Guidelines

This document defines how AI agents should work in the `shoptet/templates-assets` repository.

---

## Repository Status

This repository is a **downstream mirror** of the upstream Shoptet CMS4 monorepo. The canonical source for everything here lives in that monorepo, and the public `README.md` asks contributors **not to edit files in this repository directly** — changes made here are intended to be overwritten on the next sync from upstream.

In practice this means: when a user asks an agent to make a change here, treat it as a local edit unless they explicitly say otherwise. For changes meant to ship, the work belongs in the upstream monorepo.

---

## What This Repo Contains

A static set of frontend source assets for the Shoptet storefront templates:

- **LESS** stylesheets (`.less`) — component, layout, and theme styles
- **JavaScript** (`.js`) — ES6 module entry files plus per-feature modules, mixing vanilla JS and jQuery
- **Fonts** and **images** — per-template and shared
- **Vendored libraries** — Bootstrap 3 LESS in `00/css/bootstrap/`, jQueryUI in `shared/jqueryui/`, plus several jQuery plugins (`colorbox`, `cloud-zoom`, `jquery-mobile.custom`, etc.)

What is **not** in this repo:

- Smarty templates (`.tpl`) — they live in the upstream monorepo
- React widgets / TypeScript / Vite / Vitest / Playwright
- Any build tooling: no `package.json`, no `tsconfig`, no Webpack or Vite config, no lockfile
- 2G-generation templates (`_/`, `04/`, `05/`, `06/`, `08/`)

LESS compilation, JS bundling, linting, and tests all run in the upstream monorepo — none of those commands work from this directory.

---

## Directory Structure

```
templates-assets/
├── 00/                       # 3G base — canonical source for shared CSS and ALL 3G JS
│   ├── css/
│   │   ├── bootstrap/        # Bootstrap 3 LESS (vendored)
│   │   ├── modules/          # shared modules + components/
│   │   ├── theme_modules/    # shared theme modules + components/
│   │   ├── uitweaks/
│   │   ├── jqueryui/
│   │   └── libs/
│   ├── js/
│   │   ├── main-3g.js        # main bundle entry (ES module imports)
│   │   ├── main-3g-header.js # header bundle entry
│   │   ├── checkout-3g.js    # checkout bundle entry
│   │   ├── expressCheckout-3g.js
│   │   ├── bootstrap/        # Bootstrap 3 JS (vendored)
│   │   ├── libs/
│   │   └── modules/          # per-feature modules
│   ├── fonts/
│   └── img/
├── 07/ 09/ 10/ 11/ 12/ 13/ 14/   # 3G template skins
│   ├── css/main-XX.less      # template entry — imports from ../../00/css/...
│   ├── css/tokens.less       # template-specific CSS custom properties
│   ├── css/modules/          # template-specific module overrides
│   ├── css/theme_modules/    # template-specific theme overrides
│   ├── css/uitweaks/
│   └── fonts/
├── shared/                   # cross-template assets
│   ├── jqueryui/             # jQuery UI (vendored)
│   ├── js/
│   │   ├── libs/             # checkout, cart, consent, common, etc.
│   │   ├── modules/
│   │   ├── typeAssertions.js # ts-check helpers
│   │   └── dataLayerHelper.js
│   ├── css/themeEditor/
│   └── fonts/
├── README.md
└── AGENTS.md
```

### 3G template codes

| Code | Name | Directory |
|------|------|-----------|
| 00 | 3G base (shared CSS + JS) | `00/` |
| 07 | Techno | `07/` |
| 09 | Tango | `09/` |
| 10 | Waltz | `10/` |
| 11 | Classic | `11/` |
| 12 | Step | `12/` |
| 13 | Disco | `13/` |
| 14 | Samba | `14/` |

---

## LESS Architecture

- **Entry point per template**: `XX/css/main-XX.less` is what the upstream build compiles for that template.
- **Cross-template imports**: each `main-XX.less` pulls shared building blocks from `00/` using relative paths, e.g. `@import "../../00/css/modules/bootstrap";` and `@import "../../00/css/modules/components/banner-benefit";`.
- **Design tokens**: per-template `XX/css/tokens.less` defines CSS custom properties under `:root` (colors, spacing, scroll offsets, etc.). Reusable values belong here, not inline.
- **Shared components**: live under `00/css/modules/components/`. A new shared 3G component goes there, then gets `@import`-ed from each consuming template's `main-XX.less`.
- **Template overrides**: each `XX/css/modules/` and `XX/css/theme_modules/` contains the LESS that differs from the 3G base for that specific skin.
- **Bootstrap**: Bootstrap 3 LESS is vendored under `00/css/bootstrap/`; mixins like `@grid-gutter-width` come from there.

### CSS/LESS style

- Mobile-first: start with mobile, layer up via media queries.
- BEM naming: `.block__element--modifier`.
- Max 3 levels of nesting.
- All reusable values in `tokens.less` or shared variable files — no magic numbers.
- No `!important`. No global selectors that aren't scoped to a block.
- Animate via `transform`/`opacity`; avoid animating `top`/`left`.
- Visible focus indicators; WCAG AA contrast (4.5:1) minimum.

---

## JavaScript Architecture

- **All 3G JS lives in `00/js/`.** Templates 07–14 ship no JavaScript of their own — they inherit the same JS bundles.
- **Entry files** (`main-3g.js`, `main-3g-header.js`, `checkout-3g.js`, `expressCheckout-3g.js`) use ES module `import` syntax. The upstream monorepo's bundler resolves these imports; this repo has no bundler config of its own.
- **Module styles in `00/js/modules/`** are a mix of:
  - jQuery plugins (`jquery.colorbox.js`, `jquery.cloud-zoom.1.0.2.js`, `jquery-mobile.custom.js`, `jquery.ui.touch-punch.js`)
  - IIFE modules attaching to a global `shoptet` namespace, e.g. `(function (shoptet) { ... })`
  - Modern modules that `import` helpers from `shared/js/` (e.g. `globalFunctions.js` imports from `shared/js/typeAssertions.js`)
- **`shared/js/typeAssertions.js`** exports runtime type guards (`ensure`, `ensureEvery`, `isHTMLElement`, `maybe`) — used to bolt JSDoc/`ts-check`-style typing onto vanilla JS. There is no TypeScript compilation in this repo.
- **jQuery is still pervasive** in 3G code. New JavaScript should prefer vanilla DOM APIs; jQuery is maintained where it already exists but not extended in fresh code.

---

## Browser Support

- Chrome, Firefox, Safari, Edge: last 2 versions
- Mobile Safari: iOS 12+
- Chrome Mobile: last 2 versions
- Target: `baseline-widely-available`

---

## Demo URLs (Visual Verification)

There is no local dev server in this repo. To see changes rendered, use the upstream-built demo storefronts:

| Code | Template | Production URL | Local Dev URL |
|------|----------|----------------|---------------|
| 14 | Samba | https://samba.shoptet.cz | https://samba.shoptet.cz.test |
| 13 | Disco | https://disco.shoptet.cz | https://disco.shoptet.cz.test |
| 12 | Step | https://step.shoptet.cz | https://step.shoptet.cz.test |
| 11 | Classic | https://classic.shoptet.cz | https://classic.shoptet.cz.test |
| 10 | Waltz | https://waltz.shoptet.cz | https://waltz.shoptet.cz.test |
| 09 | Tango | https://tango.shoptet.cz | https://tango.shoptet.cz.test |
| 07 | Techno | https://techno.shoptet.cz | https://techno.shoptet.cz.test |

---

## Tests, Linters, Build

None of these are configured in this repo. There is no `pnpm test`, no ESLint config to invoke from this directory, and no compilation step. Verification of changes is visual via the demo URLs above; correctness gates run in the upstream monorepo's pipeline.

---

## Commit Conventions

The mirror's own history is mostly squashed `Update` commits, but for any local work follow Conventional Commits:

```
type(scope): subject
```

Types: `feat`, `fix`, `refactor`, `style`, `test`, `docs`, `chore`, `perf`. Subject: imperative mood, lowercase, no trailing period, ≤50 characters.

Examples:

```
feat(samba): add tabs-horizontal component to main-14.less
fix(00/js): debounce search input handler
refactor(shared): extract focus-trap helper into typeAssertions
```

---

## Agent Behavior Summary

- Treat the repo as **read-mostly**: real changes belong upstream. Edit here only when the user explicitly asks.
- Prefer vanilla JS over jQuery for new code; do not introduce new jQuery dependencies.
- Put reusable LESS values in `tokens.less` / shared variable files — never magic numbers, never `!important`.
- Shared 3G components live in `00/`; template-specific overrides live in the template's own `css/modules/` or `css/theme_modules/`.
- Don't reference build/test commands (`pnpm dev`, `pnpm test`, `./docker/dev ...`) as if they run from this directory — they don't.
- Don't invent files or directories that aren't here (no `react/`, no `.tpl`, no 2G template dirs).
