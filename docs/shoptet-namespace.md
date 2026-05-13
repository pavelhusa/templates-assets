# The `shoptet` global namespace

Almost every authored JS file in this repo attaches to a single global `shoptet` object. This page explains how that namespace is bootstrapped, how modules register themselves into it, how it dispatches events, and how shop templates extend it without editing this repo.

## Bootstrap

`shoptet` is **not declared in any single canonical file** in this repository. It is pre-injected by the upstream Shoptet CMS on the rendered HTML page before any of the bundled JS executes, populated with at least `shoptet.config`, `shoptet.abilities`, `shoptet.messages`, and `shoptet.content`.

Every module then extends it with an idempotent IIFE:

```js
(function (shoptet) {
    shoptet.scripts = shoptet.scripts || {};
    // ... attach functions to shoptet.scripts
})(shoptet);
```

Examples of this pattern:

- [shared/js/libs/scripts.js](../shared/js/libs/scripts.js) — line 1, wraps everything in `(function (shoptet) { ... })(shoptet);`
- [shared/js/libs/config.js](../shared/js/libs/config.js) — `shoptet.config = shoptet.config || {}` immediately at top.
- [shared/js/libs/custom.js](../shared/js/libs/custom.js) — `shoptet.custom = shoptet.custom || {}` plus `shoptet.custom.config = shoptet.custom.config || {}` so the backend can pre-seed config.

## Load order

The bundle entry [00/js/main-3g.js](../00/js/main-3g.js) imports modules in a specific order that the framework relies on:

1. Vendored Bootstrap and jQuery UI plugins
2. `shared/js/libs/scripts.js` — establishes registration & signaling (must run first)
3. `shared/js/libs/dev.js` — debugging instrumentation
4. `shared/js/libs/custom.js` — extension hooks (defines `shoptet.custom.config` defaults)
5. `shared/js/libs/config.js` — reads from `shoptet.custom.config`, fills `shoptet.config` defaults
6. All other libs (`modal`, `common`, `helpers`, `cart`, `tracking`, etc.)
7. `00/js/modules/*` — template-layer modules that depend on libs already being registered

If a module is reordered before its dependencies are registered, its `registerFunction` calls will throw silently and `shoptet.<lib>.<name>` will be undefined at runtime.

## Sub-namespace catalogue

The following `shoptet.<name>` sub-namespaces appear across the codebase. Each links to the doc page where the source file is detailed.

### Framework / infrastructure

| Namespace | Purpose | Documented in |
| --- | --- | --- |
| `shoptet.scripts` | Function registration, event signaling, custom callback orchestration | [shared-libs.md#scriptsjs](shared-libs.md#scriptsjs) |
| `shoptet.dev` | Event monitoring, deprecation warnings, dev-mode debug helpers | [shared-libs.md#devjs](shared-libs.md#devjs) |
| `shoptet.custom` | Override hooks for templates (`postSuccessfulValidation`, `postFailedValidation`, `config`) | [shared-libs.md#customjs](shared-libs.md#customjs) |
| `shoptet.config` | Runtime config (timeouts, breakpoints) seeded from `shoptet.custom.config` | [shared-libs.md#configjs](shared-libs.md#configjs) |
| `shoptet.runtime` | Resize event debouncing and viewport state | [shared-libs.md#runtimejs](shared-libs.md#runtimejs) |
| `shoptet.layout` | Viewport/resolution detection, scrollbar width, Apple Pay availability | [shared-libs.md#layoutjs](shared-libs.md#layoutjs) |
| `shoptet.abilities` | Feature flags injected by backend (`feature.*`, `about.generation`, `elements.*`) | See [Feature flags](#shoptetabilities-feature-flags) below |
| `shoptet.a11y` | Live `prefers-reduced-motion` flag for skipping transitions | [shared-libs.md#a11yjs](shared-libs.md#a11yjs) |
| `shoptet.messages` | Localised message strings injected by backend | (pre-injected; not authored in this repo) |
| `shoptet.content` | HTML fragments injected by backend (modal headers/footers etc.) | (pre-injected; not authored in this repo) |

### Networking / data

| Namespace | Purpose | Documented in |
| --- | --- | --- |
| `shoptet.ajax` | XHR wrapper + `AjaxResponse` constructor | [shared-ajax.md](shared-ajax.md) |
| `shoptet.csrf` | CSRF token injection and validation | [shared-libs.md#csrfjs](shared-libs.md#csrfjs) |
| `shoptet.csrfLink` | CSRF-protected link click handlers | [shared-libs.md#csrflinkjs](shared-libs.md#csrflinkjs) |
| `shoptet.cookie` | Cookie get/create | [shared-libs.md#cookiejs](shared-libs.md#cookiejs) |
| `shoptet.consent` | GDPR cookie consent management | [shared-libs.md#consentjs](shared-libs.md#consentjs) |

### Storefront features

| Namespace | Purpose | Documented in |
| --- | --- | --- |
| `shoptet.cart` | Cart AJAX operations, quantity updates, UI sync | [shared-libs.md#cartjs](shared-libs.md#cartjs) |
| `shoptet.cartShared` | Modern add/remove/update cart API | [shared-libs.md#cartsharedjs](shared-libs.md#cartsharedjs) |
| `shoptet.checkout` | Order step 2 (sticky sidebar, billing/shipping) | [shared-libs.md#checkoutjs](shared-libs.md#checkoutjs) |
| `shoptet.checkoutShared` | Shared checkout logic (payments, shipping, prices) | [shared-libs.md#checkoutsharedjs](shared-libs.md#checkoutsharedjs) |
| `shoptet.expressCheckout` | Express checkout flow | [shared-libs.md#expresscheckoutjs](shared-libs.md#expresscheckoutjs) |
| `shoptet.externalShipping` | External shipping provider hooks | [template-libs.md](template-libs.md) |
| `shoptet.cofidis` | Cofidis payment-plan calculator | [shared-libs.md#cofidisjs](shared-libs.md#cofidisjs) |
| `shoptet.modal` | Modal wrapper around jQuery Colorbox | [shared-libs.md#modaljs](shared-libs.md#modaljs) |
| `shoptet.tracking` | GA UA/GA4, Facebook Pixel, Glami, TikTok, GTM dataLayer | [shared-libs.md#trackingjs](shared-libs.md#trackingjs) |

### Product / variant

| Namespace | Purpose | Documented in |
| --- | --- | --- |
| `shoptet.products` | Product detail/listing image handling, thumbnails, ratings | [template-modules.md#productsjs](template-modules.md#productsjs) |
| `shoptet.variantsCommon` | Cart-button gating, variant browser-restoration | [template-libs.md#variantscommonjs](template-libs.md#variantscommonjs) |
| `shoptet.variantsSimple` | Single-select variant switcher | [template-libs.md#variantssimplejs](template-libs.md#variantssimplejs) |
| `shoptet.variantsSplit` | Multi-parameter variant switcher with AJAX lookups | [template-libs.md#variantssplitjs](template-libs.md#variantssplitjs) |
| `shoptet.variantsData` | Dynamic variant metadata fetch (EAN etc.) | [template-libs.md#variantsvariantsdatajs](template-libs.md#variantsvariantsdatajs) |
| `shoptet.variantsUnavailable` | Disable/hide unavailable variant combinations | [template-libs.md#variantsunavailablejs](template-libs.md#variantsunavailablejs) |
| `shoptet.surcharges` | Variant surcharge price calculation | [template-libs.md#variantssurchargesjs](template-libs.md#variantssurchargesjs) |
| `shoptet.xyDiscounts` | X+Y discount flag toggling | [shared-libs.md#xydiscountsjs](shared-libs.md#xydiscountsjs) |
| `shoptet.stockAvailabilities` | Lazy-loaded stock tooltips | [shared-libs.md#stockavailabilitiesjs](shared-libs.md#stockavailabilitiesjs) |
| `shoptet.quantity` | Quantity input min/max/step handling | [template-modules.md#quantityjs](template-modules.md#quantityjs) |
| `shoptet.watchdog` | Product price/stock alert signup | [template-modules.md#watchdogjs](template-modules.md#watchdogjs) |
| `shoptet.productSorting` | Product list sort dropdown | [template-modules.md#productsortingjs](template-modules.md#productsortingjs) |
| `shoptet.topProducts` | Top-products expand/collapse block | [template-modules.md#topproductsjs](template-modules.md#topproductsjs) |

### UI / navigation

| Namespace | Purpose | Documented in |
| --- | --- | --- |
| `shoptet.menu` | Main menu, submenus, overflow helper | [template-modules.md#menujs](template-modules.md#menujs) |
| `shoptet.popups` | Cart/search/register/navigation popups + focus trap | [template-modules.md#popupsjs](template-modules.md#popupsjs) |
| `shoptet.dialog` | Native `<dialog>` element controller (open / close / focus) | [template-modules.md#dialogsjs](template-modules.md#dialogsjs) |
| `shoptet.focusManagement` | First / last focusable lookup, focus trap, dialog pass-through focus | [template-modules.md#focusmanagementjs](template-modules.md#focusmanagementjs) |
| `shoptet.tooltips` | New-style tooltip system (currently inert — registration commented out) | [template-modules.md#tooltipsjs](template-modules.md#tooltipsjs) |
| `shoptet.ratingStars` | Star rating input wiring | [template-modules.md#ratingstarsjs](template-modules.md#ratingstarsjs) |
| `shoptet.cookieBar` | Cookie bar visibility on small screens | [template-modules.md#cookiebarjs](template-modules.md#cookiebarjs) |
| `shoptet.skipLinks` | Skip-to-content links for keyboard nav | [template-modules.md#skiplinksjs](template-modules.md#skiplinksjs) |
| `shoptet.screenReader` | ARIA-live announcer and loading announcer class | [template-modules.md#screenreaderjs](template-modules.md#screenreaderjs) |
| `shoptet.scroll` | `scrollToEl` with reduced-motion support | [template-modules.md#scrolljs](template-modules.md#scrolljs) |
| `shoptet.animations` | `fadeIn` helper with reduced-motion support | [template-modules.md#animationsjs](template-modules.md#animationsjs) |
| `shoptet.slider` | `Slider` class, product slider initialiser | [template-modules.md#sliderjs](template-modules.md#sliderjs) |
| `shoptet.images` | Lazy-load (`unveil`) with retina detection | [shared-libs.md#imagesjs](shared-libs.md#imagesjs) |
| `shoptet.helpers` | Number/currency formatting, focus, touch detection | [shared-libs.md#helpersjs](shared-libs.md#helpersjs) |
| `shoptet.common` | DOM/form utilities, throttle, event-name helper | [shared-libs.md#commonjs](shared-libs.md#commonjs) |
| `shoptet.phoneInput` | International phone input with flag picker | [shared-libs.md#phoneinputjs](shared-libs.md#phoneinputjs) |

### Validators

| Namespace | Purpose | Documented in |
| --- | --- | --- |
| `shoptet.validator` | Core form-validation framework | [shared-validator.md](shared-validator.md) and [template-modules.md#validatorjs](template-modules.md#validatorjs) |
| `shoptet.validatorRequired` | Required-field validator | [shared-validator.md#requiredjs](shared-validator.md#requiredjs) |
| `shoptet.validatorPhone` | Phone number validator | [shared-validator.md#phonejs](shared-validator.md#phonejs) |
| `shoptet.validatorZipCode` | ZIP/postal code validator | [shared-validator.md#zipcodejs](shared-validator.md#zipcodejs) |
| `shoptet.validatorCompanyId` | Company ID / VAT number validator | [shared-validator.md#companyidjs](shared-validator.md#companyidjs) |

The catalogue is not exhaustive — backend pre-injection may add more (`shoptet.events`, etc.). Use `Object.keys(shoptet)` in DevTools on a real shop page to inspect the live tree.

## Registration system

The registration framework lives in [shared/js/libs/scripts.js](../shared/js/libs/scripts.js). It exists to:

- Give every public function a consistent namespace path (`shoptet.<lib>.<fn>`).
- Wrap each function call so a `CustomEvent` fires before *and* after execution.
- Capture arguments for post-call callbacks and dev-mode monitoring.
- Catch exceptions instead of letting one module break the page.

### `shoptet.scripts.registerFunction(fn, lib)`

Defined at `scripts.js:133-138`. For each function name listed under `shoptet.scripts.libs.<lib>`, the consuming module typically iterates and registers:

```js
shoptet.scripts.libs.runtime.forEach(function (fnName) {
    var fn = eval(fnName);
    shoptet.scripts.registerFunction(fn, 'runtime');
});
```

After registration, calling `shoptet.runtime.resizeEnd()` invokes the registered wrapper, which delegates to `shoptet.scripts.applyFunction(fn, arguments)`.

### `shoptet.scripts.applyFunction(fn, args)`

Defined at `scripts.js:11-29`. Each call:

1. Resolves the namespace from `fn.prototype.shoptetNamespace`.
2. Stores `arguments` in `shoptet.scripts.arguments[namespace + fn.name]` for inspection.
3. Calls `handleFunctionCall()` which dispatches a `CustomEvent(namespace + fn.name)`.
4. Invokes the original function.
5. Calls `handleFunctionCallback()` which runs any registered `customCallbacks[namespace + fn.name]`.
6. Catches and logs (in dev mode) any thrown exception so one module's failure doesn't cascade.

### `shoptet.scripts.libs`

Defined at `scripts.js:140` onward. A static map from `lib` name to an array of function names that lib publishes. Adding a new public function requires adding its name to this map *and* calling `registerFunction` from within the lib.

## Event signaling

Four signal helpers wrap `document.dispatchEvent(new CustomEvent(...))`. They differ in *which* whitelist they validate against and whether they also dispatch a global wrapper event.

| Function | Whitelist | Use it for |
| --- | --- | --- |
| `shoptet.scripts.signalDomLoad(event, el)` | `availableDOMLoadEvents` | DOM content loaded for a region (cart, search results, registration form). Also dispatches the umbrella `ShoptetDOMContentLoaded`. |
| `shoptet.scripts.signalDomUpdate(event, el)` | `availableDOMUpdateEvents` | DOM changed without reload (cart count updated). |
| `shoptet.scripts.signalCustomEvent(event, el)` | `availableCustomEvents` | Business logic events: variant changed, form validated, filters applied, modal resized. |
| `shoptet.scripts.signalNativeEvent(event, el)` | — (no whitelist) | Re-dispatching native events (resize, scroll) wrapped in a `CustomEvent`. |

The three whitelists live in `scripts.js:143-202`:

- `availableDOMLoadEvents` — load events for distinct DOM regions, e.g. `ShoptetDOMCartContentLoaded`, `ShoptetDOMSearchResultsLoaded`, `ShoptetDOMProductRatingsLoaded`.
- `availableDOMUpdateEvents` — DOM-update events, currently `ShoptetDOMCartCountUpdated`.
- `availableCustomEvents` — ~40 custom events: `ShoptetSimpleVariantChange`, `ShoptetCartAddCartItem`, `validatedFormSubmit`, `ShoptetDataLayerUpdated`, etc.

Whitelisting prevents typos from silently doing nothing — an unknown event name is logged and rejected.

## Custom hooks

Three ways a custom theme can override or extend behaviour without editing this repo.

### `shoptet.custom.config`

Used by [shared/js/libs/config.js](../shared/js/libs/config.js) as the "look here first" lookup for every configurable constant. A theme can inject (typically in HTML before the bundle loads):

```js
window.shoptet = window.shoptet || {};
window.shoptet.custom = window.shoptet.custom || {};
window.shoptet.custom.config = { animationDuration: 500, dismissTimeout: 10000 };
```

`config.js` then reads `shoptet.custom.config.animationDuration ?? 300` and exposes the result as `shoptet.config.animationDuration`.

### `shoptet.scripts.setCustomCallback(fnName, cb)`

Defined at `scripts.js:45-56`. Adds a callback that fires **after** every invocation of the named function. Multiple callbacks chain via closure:

```js
shoptet.scripts.setCustomCallback('shoptet.cart.updateCartButton', function (args) {
    // args is the array of arguments the original was called with
});
```

### `shoptet.scripts.setCustomFunction(fnName, fn)`

Defined at `scripts.js:59-68`. Overrides the implementation of a `shoptet.custom.*` hook (such as `postSuccessfulValidation`). Unlike `setCustomCallback`, this replaces; but the new function can call the previous one if it captured a reference, so chaining is possible by convention.

The two built-in hooks declared in [shared/js/libs/custom.js](../shared/js/libs/custom.js) are:

- `shoptet.custom.postSuccessfulValidation(form, args)` — fires when validation passes.
- `shoptet.custom.postFailedValidation(form, args)` — fires when validation fails.

The validator in [00/js/modules/validator.js](../00/js/modules/validator.js) calls these via `shoptet.scripts.setCustomFunction`'s chained-pattern wrapper.

## `shoptet.abilities` feature flags

`shoptet.abilities` is **not** populated by anything in this repo; it is injected by the upstream backend before bundle execution. Modules read from it defensively.

Typical structure:

```js
shoptet.abilities = {
    about:   { generation: 3 },
    feature: { smart_labels: true, tabs_accordion: true, tabs_responsive: true,
               top_navigation_menu: true, fixed_header: true, ... },
    elements: { recapitulation_in_checkout: true, ... },
};
```

Patterns of consumption:

- **Gate whole-module activation.** [00/js/modules/smartLabels.js](../00/js/modules/smartLabels.js), [00/js/modules/tabsAccordion.js](../00/js/modules/tabsAccordion.js), [00/js/modules/tabsResponsive.js](../00/js/modules/tabsResponsive.js), [00/js/modules/topNavigationMenu.js](../00/js/modules/topNavigationMenu.js) all check `shoptet.abilities.feature.<flag>` at top-level and no-op if disabled.
- **Branch generation paths.** [shared/js/libs/cartShared.js](../shared/js/libs/cartShared.js) branches on `shoptet.abilities.about.generation !== 3` to fall back to legacy 1G/2G AJAX functions.
- **Conditional dev tools.** [shared/js/libs/dev.js](../shared/js/libs/dev.js) only attaches its event monitor when `shoptet.abilities.about.generation === 3`.

Treat `shoptet.abilities` as read-only at the module level; modifying it after load won't retrofit features into modules that already self-disabled.
