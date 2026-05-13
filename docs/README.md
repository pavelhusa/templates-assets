# templates-assets — JavaScript documentation

This directory documents the JavaScript authored for Shoptet 3G storefront templates. It is function-level reference material aimed at both human developers new to the repo and AI agents loading context. Every section links into the source.

**Start here:** [shoptet-namespace.md](shoptet-namespace.md) — the `shoptet` global is the architectural backbone of nearly every file in this repo.

For repo-level conventions, build context, and directory structure, see [../README.md](../README.md) and [../AGENTS.md](../AGENTS.md).

## Scope

- Documented: our authored JavaScript under `00/js/` (excluding vendored jQuery plugins and Bootstrap) and `shared/js/`.
- Not documented: LESS / CSS architecture, the vendored `00/js/bootstrap/`, `shared/js/jqueryui/`, and the bundled jQuery plugins (`jquery.colorbox.js`, `jquery.cloud-zoom.1.0.2.js`, `jquery-mobile.custom.js`, `jquery.ui.touch-punch.js`, `select2.js`, `picturefill.js`).
- Entry bundles are documented as composition only — their imports are listed but feature code lives in the imported modules.

## Module-style legend

Three patterns recur across the codebase:

| Style | Recognised by | Notes |
| --- | --- | --- |
| **IIFE on `shoptet` global** | `(function (shoptet) { ... })(shoptet || (shoptet = {}))` | Predominant pattern. Module extends a sub-namespace of the global `shoptet` object. |
| **ES module** | top-level `import` / `export` | Resolved by the upstream bundler. Newer modules use this; some also still attach functions to `shoptet.*`. |
| **jQuery plugin** | `$.fn.SomeName = function () { ... }` | A handful of UI behaviours register as jQuery plugins (e.g. `SmartLabels`, `shpResponsiveNavigation`). |

## Doc index

| Doc | Covers |
| --- | --- |
| [shoptet-namespace.md](shoptet-namespace.md) | The `shoptet` global: bootstrap, sub-namespace catalogue, registration system, event signaling, custom hooks, `shoptet.abilities` feature flags. |
| [entries.md](entries.md) | Entry bundles: `main-3g.js`, `main-3g-header.js`, `checkout-3g.js`, `expressCheckout-3g.js`. |
| [template-modules.md](template-modules.md) | `00/js/modules/` — the 3G template module layer (popups, menu, products, search, filters, slider, validator, etc.). |
| [template-libs.md](template-libs.md) | `00/js/libs/` — external-shipping example modules and the `variants/` engine. |
| [shared-libs.md](shared-libs.md) | `shared/js/libs/` root — cart, checkout, tracking, modal, helpers, config, scripts, consent, csrf, runtime, etc. |
| [shared-validator.md](shared-validator.md) | `shared/js/libs/validator/` — the validator framework and per-field validators. |
| [shared-ajax.md](shared-ajax.md) | `shared/js/libs/ajax/` — request wrapper and response object. |
| [shared-theme-editor.md](shared-theme-editor.md) | `shared/js/libs/themeEditor/` — preview-iframe scripts. |
| [shared-utilities.md](shared-utilities.md) | `shared/js/typeAssertions.js`, `shared/js/dataLayerHelper.js`, and a pointer to the vendored `shared/js/modules/` files. |

## How modules expose APIs

Three exposure surfaces appear throughout the docs:

- **`shoptet.x.y`** — registered via `shoptet.scripts.registerFunction(fn, 'x')`; see [shoptet-namespace.md](shoptet-namespace.md#registration-system).
- **`window.foo`** — legacy globals, mostly older `00/js/modules/` code (e.g. `window.showSpinner`, `window.scrollToEl`, the `filters.js` `window.priceFilter` family).
- **`export …`** — ES-module exports consumed by other modules in the bundle (e.g. `shared/js/typeAssertions.js`).

Each file entry below lists which of these it uses under **Exposes**.
