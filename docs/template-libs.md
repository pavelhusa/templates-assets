# Template libs — `00/js/libs/`

Template-specific libs that ship with the 3G base. Two flavours:

- The `externalShipping*` files at the root of `00/js/libs/` are **example implementations** showing how to wire a third-party shipping provider into checkout. They are not used in production unless a shop replaces or extends them.
- The `variants/` subdirectory contains the **product variant engine** — the live runtime for single-select, multi-parameter, and unavailable-option handling.

For the `shoptet.*` exposure pattern, see [shoptet-namespace.md](shoptet-namespace.md).

---

## externalShippingOne.js

[Source](../00/js/libs/externalShippingOne.js)

**Purpose:** Example implementation of a custom external shipping method integration.
**Module style:** IIFE on `shoptet`; extends `shoptet.externalShipping`.
**Exposes:** `shoptet.externalShipping[myShippingName]` with `modalContent`, `onComplete`, `onClosed`.

| Member | Signature | Description |
| --- | --- | --- |
| `modalContent` | `HTMLElement` | DOM element containing the shipping-method UI. |
| `onComplete(el)` | `(el) => void` | Called after the modal fully loads; calls `shoptet.modal.resize()`. |
| `onClosed(el)` | `(el) => void` | Called on modal close; dispatches `ShoptetExternalShippingChanged`. |

**Bound events:** `ShoptetBaseShippingInfoObtained`, `ShoptetShippingMethodUpdated`, `ShoptetBillingMethodUpdated`.

**Dependencies:** `shoptet.checkoutShared`, `shoptet.modal`.

---

## externalShippingTwo.js

[Source](../00/js/libs/externalShippingTwo.js)

**Purpose:** Second example with alternate pricing logic — same shape as `externalShippingOne.js`, included to show how shops can register multiple external providers.
**Module style:** IIFE on `shoptet`; extends `shoptet.externalShipping`.
**Exposes:** `shoptet.externalShipping[myShippingName]` with `modalContent`, `onComplete`, `onClosed`.

**Dependencies:** `shoptet.checkoutShared`, `shoptet.modal`.

---

## variants/surcharges.js

[Source](../00/js/libs/variants/surcharges.js)

**Purpose:** Calculate and display surcharge prices for product variants (e.g. engraving, gift wrap).
**Module style:** IIFE on `shoptet`; functions registered via `shoptet.scripts.registerFunction`.
**Exposes:** `shoptet.surcharges.initSurcharges`, `updatePrices`, `getSurchargePrices`, `writePrices`, plus the state object `shoptet.surcharges.values`.

| Function | Signature | Description |
| --- | --- | --- |
| `initSurcharges()` | `() => void` | Initialise listeners from the dataLayer; register `change` handlers on `.surcharge-parameter` inputs. |
| `updatePrices(e)` | `(e: Event) => void` | Recalculate and update surcharge prices; dispatch `ShoptetSurchargesPriceUpdated`. |
| `getSurchargePrices(wrapper)` | `(wrapper: HTMLElement) => void` | Iterate surcharge options, populate `shoptet.surcharges.values` with final and additional prices. |
| `writePrices(wrapper)` | `(wrapper: HTMLElement) => void` | Update price-display DOM nodes by adding the computed surcharge totals. |

State on `shoptet.surcharges.values`: `totalSurchargeFinalPrice`, `totalSurchargeAdditionalPrice`, `customerPriceRatio`.

**Dependencies:** `shoptet.scripts.signalCustomEvent`.

---

## variants/common.js

[Source](../00/js/libs/variants/common.js)

**Purpose:** Common variant utilities used by simple and split variant handlers — cart-button gating, browser-restoration nudges, quantity-tooltip updates.
**Module style:** IIFE on `shoptet`; functions registered via `shoptet.scripts.registerFunction`.
**Exposes:** `shoptet.variantsCommon.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `disableAddingToCart()` | `() => void` | Add `disabled-add-to-cart` class to `<body>`. |
| `enableAddingToCart()` | `() => void` | Remove `disabled-add-to-cart` and `variant-not-chosen` classes. |
| `hasToDisableCartButton()` | `() => boolean` | True if an empty/disabled variant option is selected or no variant is chosen. |
| `handleSubmit($el)` | `($el: jQuery) => boolean` | Validate variant selection on form submit; return false if invalid; show an error message. |
| `handleBrowserValueRestoration()` | `() => void` | Fire manual change events on page load for browser-restored form values. |
| `updateQuantityTooltips($form, min, max)` | `($form, min, max) => void` | Update min/max text in quantity tooltips. |
| `hideQuantityTooltips()` | `() => void` | Hide all quantity-related tooltips. |

State: `shoptet.variantsCommon.reasonToDisable` (string), `noDisplayClasses` (CSS class names).

**Dependencies:** jQuery, `shoptet.scripts.signalNativeEvent`, `shoptet.config`, `shoptet.messages`.

---

## variants/variantsData.js

[Source](../00/js/libs/variants/variantsData.js)

**Purpose:** Fetch and cache dynamic variant metadata (EAN, custom values) via AJAX.
**Module style:** ES module with imports; registered on `shoptet` via `shoptet.scripts.registerFunction`.
**Exposes:** `shoptet.variantsData.fetchData`, `shoptet.variantsData.getProductVariants`.

| Function | Signature | Description |
| --- | --- | --- |
| `fetchData(priceId?)` | `(priceId?: number) => void` | Fetch variant data and update dynamic values; `undefined` resets to default. |
| `getProductVariants()` | `() => Array<{ productId, productVariantId, ean }>` | Return the cached variant list. |

**Dependencies:** `shoptet.ajax.makeAjaxRequest`, `shoptet.messages`, `ensure` from `shared/js/typeAssertions`.

---

## variants/simple.js

[Source](../00/js/libs/variants/simple.js)

**Purpose:** Single-select variant switcher (one `<select>` or one radio group per product).
**Module style:** IIFE on `shoptet`; registered via `shoptet.scripts.registerFunction`.
**Exposes:** `shoptet.variantsSimple.handler`, `shoptet.variantsSimple.switcher`.

| Function | Signature | Description |
| --- | --- | --- |
| `handler()` | `() => void` | Initialise surcharges and bind change listeners to the variant select / radios. |
| `switcher($variant)` | `($variant: jQuery) => void` | Update form `priceId`, images, pricing details, GA/FB tracking, quantity limits. |

State: `shoptet.variantsSimple.loadedVariant` — cached selected variant element.

**Dependencies:** `shoptet.surcharges`, `shoptet.variantsCommon`, `shoptet.variantsData`, `shoptet.tracking`, `shoptet.products`, `shoptet.xyDiscounts`, `shoptet.quantityDiscounts`, jQuery.

---

## variants/split.js

[Source](../00/js/libs/variants/split.js)

**Purpose:** Multi-parameter variant switcher (e.g. colour × size) with AJAX variant lookups.
**Module style:** IIFE on `shoptet`; registered via `shoptet.scripts.registerFunction`.
**Exposes:** `shoptet.variantsSplit.handler`, `shoptet.variantsSplit.getData`, `shoptet.variantsSplit.showVariantDependent`.

| Function | Signature | Description |
| --- | --- | --- |
| `handler()` | `() => void` | Initialise surcharges and split-parameter change listeners. |
| `getData(variantCode, trackProducts)` | `(code: string, track: boolean) => void` | Look up the variant in `shoptet.variantsSplit.necessaryVariantData`; update form, images, pricing, tracking, quantity. |
| `showVariantDependent()` | `() => void` | Show / hide DOM elements scoped to specific parameter combinations using CSS classes. |

State: `shoptet.variantsSplit.necessaryVariantData` — map of `variantCode → metadata`.

**Dependencies:** `shoptet.surcharges`, `shoptet.variantsCommon`, `shoptet.variantsData`, `shoptet.tracking`, `shoptet.products`, `shoptet.xyDiscounts`, `shoptet.quantityDiscounts`, `shoptet.messages`, jQuery.

---

## variants/unavailable.js

[Source](../00/js/libs/variants/unavailable.js)

**Purpose:** Disable or hide unavailable variant option combinations (option still rendered, but greyed out / moved into an "unavailable" optgroup).
**Module style:** IIFE on `shoptet`; registered via `shoptet.scripts.registerFunction`. Initialises on `DOMContentLoaded`.
**Exposes:** `shoptet.variantsUnavailable.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `setupAllParameters(params)` | `(params: string[]) => void` | Process all parameters; hide / show unavailable options; manage message display. |
| `attachEventListeners(el, params)` | `(el, params: string[]) => void` | Attach `change` and `ShoptetSelectedParametersReset` listeners. |
| `getAvailableCombinations(variants, selected, currentParam)` | `(variants, selected, currentParam) => Record<string, true>` | Map of valid option values for the current parameter. |
| `getSelected(params, currentParam)` | `(params, currentParam) => Record<string, string> \| null` | Currently selected parameter values, or null if empty. |
| `getExistingOptions(el)` | `(el) => HTMLElement[]` | Non-placeholder option elements. |
| `getUnavailableOptgroup(el)` | `(el) => HTMLOptGroupElement` | Create or retrieve the "unavailable variants" optgroup. |
| `handleOptions(el, available, existing)` | `(el, available, existing) => void` | Apply / remove unavailable styling and move options to / from the optgroup. |
| `getOption(el, param)` | `(el, param: string) => HTMLElement \| null` | Find the option matching a given parameter value. |
| `moveOptionFromUnavailable(option, unavailableOptgroup)` | `(option, optgroup) => void` | Move an option back to the main select. |
| `areUnavailableOptionsSelected(unavailableOptions)` | `(opts) => boolean` | True if any unavailable option is currently selected. |
| `setupCurrentParameter(el, params, variants)` | `(el, params, variants) => boolean` | Main handler; returns true if an unavailable option ended up selected. |
| `sortOptions(el)` | `(el) => void` | Sort `<select>` options by `data-index`, move the optgroup to the end. |

State: `shoptet.variantsUnavailable.classes` (CSS class mappings), `availableVariantsResource`, `availableVariants`. Supports both `<select>` and radio-input variant interfaces.

**Dependencies:** `shoptet.common`, `shoptet.messages`, `shoptet.scripts.signalCustomEvent`.
