# Shared utilities — `shared/js/`

The root of `shared/js/` holds two small files plus a `modules/` directory of vendored libraries that are not authored in this repo.

For the `shoptet.*` exposure pattern, see [shoptet-namespace.md](shoptet-namespace.md).

---

## typeAssertions.js

[Source](../shared/js/typeAssertions.js)

**Purpose:** Runtime type validation — predicates that throw on mismatch, enabling JSDoc / `ts-check` style narrowing without a TypeScript compile step.
**Module style:** ES module (`export`).
**Exposes:** four named exports.

| Function | Signature | Description |
| --- | --- | --- |
| `ensure(value, condition)` | `<T>(value: unknown, condition: Predicate<T>) => T` | Throw `TypeError` if `condition(value)` is false; otherwise return `value` typed as `T`. |
| `ensureEvery(values, condition)` | `<T>(values: unknown[], condition: Predicate<T>) => T[]` | Throw if any element fails; logs per-element failures. |
| `maybe(value, condition)` | `<T>(value: unknown, condition: Predicate<T>) => T \| undefined` | Non-throwing variant — returns the value if the predicate passes, `undefined` otherwise. |
| `isHTMLElement` | `Predicate<HTMLElement>` | Built-in predicate: `value instanceof HTMLElement`. |

**Dependencies:** none.

---

## dataLayerHelper.js

[Source](../shared/js/dataLayerHelper.js)

**Purpose:** Re-export wrapper around the Google Data Layer helper that lives in the upstream public assets.
**Module style:** ES module re-export.
**Exposes:** whatever the wrapped helper exports.

The file simply imports `public/cms/libs/googleDataLayerHelper/dataLayerHelper`. The actual helper is shipped from elsewhere in the upstream monorepo; this file is the bridge so bundlers can resolve a stable import path within this repo's scope.

**Dependencies:** the upstream `dataLayerHelper` module.

---

## shared/js/dev/contrastScanner.dev.js

[Source](../shared/js/dev/contrastScanner.dev.js)

**Purpose:** Developer-only colour-contrast scanner — runs axe-core against the live page and highlights elements that fail the `color-contrast` WCAG rule with a red outline plus a label showing the contrast ratio.
**Module style:** ES module that runs at load time. Not imported by any production bundle in this repo; intended to be loaded manually in dev to audit a page.
**Exposes:** nothing public — pure side-effect script.

Behaviour:

- Lazily inject a `<style id="contrast-scanner-styles">` with the `[data-contrast-violation="true"]` outline and label rules.
- On `DOMContentLoaded` and on the custom `ShoptetDOMContentLoaded` event, run `axe.run(document, { runOnly: { type: 'rule', values: ['color-contrast'] } })`.
- For every violating node, set `data-contrast-violation="true"` and `data-contrast-violation-label="<ratio>"`; force `position: relative` if needed so the label appears.
- Guarded by a `contrastScanning` flag so concurrent scans don't pile up.

**Dependencies:** `axe-core` (resolved by the upstream bundler).

---

## shared/js/modules/ — vendored libraries

The `shared/js/modules/` directory contains two third-party libraries that this repo only carries verbatim. They are imported by name from the bundle entries but their internals are out of scope for this documentation.

| File | Origin | Purpose |
| --- | --- | --- |
| [picturefill.js](../shared/js/modules/picturefill.js) | Scott Jehl / Filament Group (v2.2.0) | Responsive image polyfill for `<picture>` and `srcset`. Exposes `window.picturefill`. |
| [select2.js](../shared/js/modules/select2.js) | select2 jQuery plugin | Enhanced `<select>` UI with search / multi-select. Imported by [`checkout-3g.js`](../00/js/checkout-3g.js). |

For details about either library, consult the upstream project documentation rather than reading the bundled source here.
