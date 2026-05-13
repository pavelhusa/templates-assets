# Shared libs — `shared/js/libs/`

Cross-template JavaScript: framework, data, storefront features, UI helpers. Everything here loads in every 3G storefront bundle and is the foundation that the `00/js/modules/` layer sits on top of.

For the `shoptet.*` exposure pattern, see [shoptet-namespace.md](shoptet-namespace.md).

Subdirectories of `shared/js/libs/` are documented separately:

- [shared-validator.md](shared-validator.md) — `validator/`
- [shared-ajax.md](shared-ajax.md) — `ajax/`
- [shared-theme-editor.md](shared-theme-editor.md) — `themeEditor/`

---

## scripts.js

[Source](../shared/js/libs/scripts.js)

**Purpose:** The registration framework — function namespacing, event signaling, custom-callback orchestration. Every other lib depends on this loading first.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.scripts.*`.

### Core functions

| Function | Signature | Description |
| --- | --- | --- |
| `applyFunction(fn, args)` | `(fn, args) => unknown` | Execute a registered function with namespace tracking, pre/post events, and exception swallowing. |
| `handleFunctionCall(fn, args, namespace)` | `(fn, args, ns) => void` | Dispatch a `CustomEvent` named `ns + fn.name` before execution. |
| `handleFunctionCallback(fn, args, namespace)` | `(fn, args, ns) => void` | Run any registered custom callback for `ns + fn.name`. |
| `setCustomCallback(fnName, cb)` | `(fnName: string, cb: (args) => void) => void` | Chain a callback to run after every invocation of the named function. |
| `setCustomFunction(fnName, fn)` | `(fnName: string, fn: Function) => void` | Override the implementation of a `shoptet.custom.*` hook. |
| `signal(event, element, eventSource, globalEvent)` | `(event, el, source, global?) => void` | Low-level event dispatch shared by the four `signal*` helpers. |
| `signalDomLoad(event, element)` | `(event, el) => void` | Fire a DOM-load event, plus the umbrella `ShoptetDOMContentLoaded`. |
| `signalDomUpdate(event, element)` | `(event, el) => void` | Fire a DOM-update event (cart count etc.). |
| `signalCustomEvent(event, element)` | `(event, el) => void` | Fire a whitelisted custom business event. |
| `signalNativeEvent(event, element)` | `(event, el) => void` | Re-dispatch a native event wrapped as a `CustomEvent`. |
| `registerFunction(fn, lib)` | `(fn, lib: string) => void` | Wrap `fn` and expose as `shoptet[lib][fn.name]`. |

### Config

- `shoptet.scripts.libs` — map of `libName → string[]` of function names that lib publishes.
- `shoptet.scripts.availableDOMLoadEvents` — whitelist for `signalDomLoad`.
- `shoptet.scripts.availableDOMUpdateEvents` — whitelist for `signalDomUpdate`.
- `shoptet.scripts.availableCustomEvents` — whitelist for `signalCustomEvent`.
- `shoptet.scripts.monitoredFunctions` — array of every namespaced function name.
- `shoptet.scripts.arguments` — last-invocation `arguments` per namespaced function.
- `shoptet.scripts.customCallbacks` — registered post-call callbacks.

**Dependencies:** `shoptet.dev`.

See [shoptet-namespace.md](shoptet-namespace.md) for the conceptual overview of how registration and signaling fit together.

---

## dev.js

[Source](../shared/js/libs/dev.js)

**Purpose:** Developer-mode helpers — event monitoring, deprecation warnings.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.dev.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `enableEventsMonitoring(reload?, expires?)` | `(reload?: boolean, expires?: object) => void` | Log every registered function call to console; persisted via cookie. |
| `disableEventsMonitoring()` | `() => void` | Turn the monitor off. |
| `printMonitoringInfo()` | `() => void` | Print current dev config to console. |
| `printEventInfo(key)` | `(key: string) => void` | Show stored arguments for the named function. |
| `deprecated(deadline, functionName, replaceWith?, customMessage?)` | `(deadline, fnName, replaceWith?, msg?) => void` | Emit a console warning about an upcoming removal. |

Config: `shoptet.dev.config` = `{ name, version, log, monitorEvents, shoptetDevMode, suppressRegFn }`.

Only attaches its monitor when `shoptet.abilities.about.generation === 3`.

**Dependencies:** `shoptet.cookie`, `shoptet.scripts`.

---

## a11y.js

[Source](../shared/js/libs/a11y.js)

**Purpose:** Expose a live `prefers-reduced-motion` flag for other modules to read.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.a11y.reducedMotion` — boolean, updates automatically when the media query changes.

Other modules (e.g. [dialogs.js](template-modules.md#dialogsjs), [animations.js](template-modules.md#animationsjs)) consult this flag to skip transitions.

**Dependencies:** `window.matchMedia('(prefers-reduced-motion: reduce)')`.

---

## custom.js

[Source](../shared/js/libs/custom.js)

**Purpose:** Theme override hooks — default implementations for `postSuccessfulValidation` and `postFailedValidation`, plus `shoptet.custom.config` for config injection.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.custom.*`.

| Member | Signature | Description |
| --- | --- | --- |
| `postSuccessfulValidation(form, args)` | `(form, args) => boolean` | Override-able hook fired after successful form validation. |
| `postFailedValidation(form, args)` | `(form, args) => boolean` | Override-able hook fired after failed validation. |
| `config` | `Record<string, unknown>` | Pre-seeded config object read by `config.js`. |

Templates customise by injecting into `window.shoptet.custom.config` before the bundle runs, or by calling `shoptet.scripts.setCustomFunction('shoptet.custom.postSuccessfulValidation', fn)`.

**Dependencies:** none (read by other libs).

---

## config.js

[Source](../shared/js/libs/config.js)

**Purpose:** Runtime configuration constants, seeded from `shoptet.custom.config` and falling back to built-in defaults.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.config.*` (read-only at module level).

Selected values:

| Key | Default | Purpose |
| --- | --- | --- |
| `animationDuration` | 300 | Default ms for fades and slides. |
| `submenuTimeout` | 150 | Submenu show/hide delay. |
| `dismissTimeout` | 6000 | How long a notification stays before auto-hide. |
| `updateQuantityTimeout` | 1000 | Debounce for quantity input changes. |
| `adminBarTimeout` | 800 | Admin-bar hover cleanup. |
| `image360RotationSpeed` | 100 | 360° viewer frame interval. |
| `breakpoints` | `{ xs, sm, md, lg, xl }` | Responsive breakpoint thresholds. |

**Dependencies:** `shoptet.custom.config`.

---

## runtime.js

[Source](../shared/js/libs/runtime.js)

**Purpose:** Window-resize debounce and the `--vh` CSS variable update.
**Module style:** IIFE on `shoptet`; registers functions via `shoptet.scripts.registerFunction`.
**Exposes:** `shoptet.runtime.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `resizeEnd()` | `() => void` | Fires after a 300 ms quiet window on resize; dispatches `resizeEnd`, updates `--vh`, clears `shoptet.layout` cache. |

State: `shoptet.runtime.resize` = `{ delta, rtime, timeout, windowWidth, windowHeight }`.

**Dependencies:** `shoptet.scripts`, `shoptet.layout`, `shoptet.config`.

---

## modal.js

[Source](../shared/js/libs/modal.js)

**Purpose:** Thin wrapper around the vendored jQuery Colorbox plugin.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.modal.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `open(options)` | `(options) => void` | Open a Colorbox modal. |
| `close()` | `() => void` | Close the active modal. |
| `resize(options?)` | `(options?) => void` | Resize the open modal manually. |
| `shoptetResize()` | `() => void` | Auto-size the modal per breakpoint and content. |

Config: `shoptet.modal.config` exposes preset widths (`widthXs/Sm/Md/Lg`), CSS classes, overlay opacity, max dimensions, and `initialHeight`. The event name `'ShoptetModalResizeDone'` is published as `shoptet.modal.resizeDoneEvent`.

**Dependencies:** jQuery Colorbox, `shoptet.layout`, `shoptet.abilities`, `shoptet.config`.

---

## common.js

[Source](../shared/js/libs/common.js)

**Purpose:** DOM and form utilities used everywhere.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.common.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `getSelectValue(select)` | `(select: HTMLSelectElement) => string` | Selected `<option>` value. |
| `getCheckedInputValue(containingElement)` | `(el) => string \| null` | Checked radio value within `el`. |
| `createDocumentFromString(string)` | `(html: string) => Document` | Parse HTML string into a `Document`. |
| `serializeData(data)` | `(obj: Record<string, unknown>) => string` | Convert object to query-string. |
| `serializeForm(form)` | `(form: HTMLFormElement \| FormData) => string` | URL-encoded form serialisation. |
| `createEventNameFromFormAction(action)` | `(action: string) => string` | Map a form `action` URL to a `Shoptet*` event name. |
| `fitsToParentWidth(el, paddingRight?)` | `(el, padding?: number) => boolean` | True if `el` does not overflow its parent. |
| `addClassToElements(elements, className)` | `(els, className) => void` | Batch `classList.add`. |
| `removeClassFromElements(elements, className)` | `(els, className) => void` | Batch `classList.remove`. |
| `moveCursorToEnd(el)` | `(el: HTMLInputElement) => void` | Place caret at end of input. |
| `throttle(func, wait, options?)` | `(fn, wait, opts?) => Function` | Throttle a function (leading / trailing control). |

Constants: `shoptet.common.keyCodes` = `{ backspace: 8, enter: 13, escape: 27 }`.

**Dependencies:** jQuery, `DOMParser`.

---

## cookie.js

[Source](../shared/js/libs/cookie.js)

**Purpose:** Cookie get / create with structured expiration.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.cookie.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `get(cookieName)` | `(name: string) => string \| false` | Return the cookie value, or `false` if absent. |
| `create(name, value, expires, path?)` | `(name, value, expires: CookieExpiration, path?: string) => void` | Set a cookie; `expires` is `{ years?, months?, days?, hours?, minutes?, seconds? }`; automatically adds `Secure` on HTTPS. |

**Dependencies:** `shoptet.dev`.

---

## consent.js

[Source](../shared/js/libs/consent.js)

**Purpose:** GDPR cookie consent — getter, setter, modal, callback registry, gated script loader.
**Module style:** IIFE on `shoptet`; uses Fetch API.
**Exposes:** `shoptet.consent.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `get()` | `() => Record<string, boolean> \| false` | Read the consent cookie. |
| `set(agreements)` | `(agreements) => void` | Save consent, sync to backend, fire registered callbacks. |
| `isSet()` | `() => boolean` | True if consent has been recorded. |
| `isAccepted(agreementType)` | `(type: string) => boolean` | True if the given category was accepted. |
| `onAccept(event)` | `(event: { type, fn }) => void` | Register a function to run when consent is accepted. |
| `openCookiesSettingModal()` | `() => void` | Open the consent preferences modal. |
| `cookiesConsentSubmit(value)` | `(value: 'all'\|'reject'\|'selection'\|'none') => void` | Process the user's consent choice. |

The module also auto-runs `window.initCookieConsent()` which loads scripts that were gated on consent.

**Dependencies:** `shoptet.ajax`, `shoptet.config`, `shoptet.modal`, `shoptet.scripts`, Fetch API.

---

## helpers.js

[Source](../shared/js/libs/helpers.js)

**Purpose:** Number formatting, currency, locale, touch detection, focus management.
**Module style:** IIFE on `shoptet`; also extends `Number.prototype`.
**Exposes:** `shoptet.helpers.*` plus prototype methods.

| Function | Signature | Description |
| --- | --- | --- |
| `toFloat(value)` | `(v: string \| number) => number` | Parse a locale-formatted decimal to a JS number. |
| `toLocaleFloat(value, decimals, trim)` | `(v, decimals, trim) => string` | Format a number to a locale string. |
| `resolveDecimalSeparator(separator?)` | `(sep?: string) => string` | Locale decimal character. |
| `resolveThousandSeparator(separator?)` | `(sep?: string) => string` | Locale thousand character. |
| `resolveDecimalPlaces(places?)` | `(places?: number) => number` | Currency decimal places. |
| `resolveCurrencySymbol(symbol?)` | `(symbol?: string) => string` | Currency code or symbol. |
| `resolveCurrencySymbolPosition(left?)` | `(left?: boolean) => 'left'\|'right'` | Currency symbol placement. |
| `formatNumber(decimals, decSep, thSep)` | `(decimals, decSep, thSep) => string` | Format-as-`Number.prototype` method. |
| `formatAsCurrency(symbol, position, decimals, decSep, thSep)` | `(...) => string` | Format with currency symbol. |
| `roundForSk(price)` | `(price: number) => number` | Round to nearest 0.05 (Slovakia). |
| `roundForHu(price)` | `(price: number) => number` | Round to nearest 5 (Hungary). |
| `roundForDocument(rounding, decimalPlaces)` | `(rounding, decimals) => number` | Round per backend setting. |
| `resolveMinimumAmount(decimals)` | `(decimals: 0\|1\|2\|3) => number` | Minimum order quantity (`1`, `0.1`, `0.01`, `0.001`). |
| `isTouchDevice()` | `() => boolean` | Detect touch capability. |
| `enableUpdatePreview(updateCode)` | `(code: string) => void` | Set the update-preview cookie. |
| `findFirstFocusable(wrapper)` | `(wrapper: HTMLElement) => HTMLElement \| null` | First tab-stop within `wrapper`. |
| `focusFirst(root, tryInside?, scrollToEl?)` | `(root, tryInside?, scrollToEl?) => void` | Focus the first focusable element with optional scroll. |

`Number.prototype` extensions: `ShoptetFormatNumber`, `ShoptetFormatAsCurrency`, `ShoptetRoundForDocument`.

Deprecated: `shoptet.helpers.isApplePayAvailable()` — delegates to `shoptet.layout.showApplePay()`.

**Dependencies:** `shoptet.config`, `shoptet.cookie`, `shoptet.variantsCommon`, jQuery.

---

## images.js

[Source](../shared/js/libs/images.js)

**Purpose:** Lazy-load images with retina support.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.images.unveil`.

| Function | Signature | Description |
| --- | --- | --- |
| `unveil()` | `() => void` | Convert `data-src` / `data-src-retina` into `src` (selects retina when `devicePixelRatio > 1`); sets `loading` attribute. |

**Dependencies:** none.

---

## tracking.js

[Source](../shared/js/libs/tracking.js)

**Purpose:** Bridge to analytics: Google Analytics (UA and GA4), Facebook Pixel (browser + CAPI), Glami, TikTok, GTM dataLayer.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.tracking.*`.

**Form-action handling**

- `getFormAction(formAction)` — classify a form action URL (add / remove / update / coupon).
- `handleAction(form, response)` — main orchestrator called from cart / form submissions.

**Google Analytics (UA)**

- `trackGoogleCart(gaData, formAction)`
- `trackGoogleProductDetail(gaData, action)`

**Google Analytics 4 (gtag)**

- `trackGtagCart(product, formAction, response)`
- `trackGtagProductDetail(product)`
- `createGtagItem(product, index, list, cartInfo)` — build a GA4 item object.
- `trackListings(lists)` — fire `view_item_list` events.

**Facebook Pixel**

- `trackFacebookPixel(fbPixelData, formAction)`
- `trackFacebookPixelApi(eventName, action, data)` — server-side CAPI.

**Other pixels**

- `trackGlamiPixel(productData, formAction)`
- `trackTikTokPixel(productData, formAction)`

**GTM dataLayer**

- `updateGoogleEcommerce(data, formAction)`
- `processTrackingContainer(trackingContainerJson)`
- `updateCartDataLayer(response)`
- `updateDataLayerCartInfo(response)`
- `trackProductsFromPayload(payload)` — used by filters/search to fire impressions from AJAX payloads.

**Dependencies:** `shoptet.ajax`, `shoptet.config`, `shoptet.consent`, `shoptet.scripts`, third-party globals `gtag`, `fbq`, `glami`, `ttq`, `dataLayer`.

---

## image360.js

[Source](../shared/js/libs/image360.js)

**Purpose:** 360° product image viewer with drag/touch interaction, playback controls, fullscreen toggle.
**Module style:** IIFE on `shoptet`; vanilla JS.
**Exposes:** auto-initialises on `.image360` elements; no public function-level API.

Behaviours:

- Drag / touch / mouse rotation.
- Play, pause, prev, next, fullscreen toggle buttons.
- DOM class management drives UI state.

**Dependencies:** `shoptet.config`, `shoptet.helpers.isTouchDevice`, `shoptet.common.keyCodes`, `shoptet.ajax`, `shoptet.messages`.

---

## stockAvailabilities.js

[Source](../shared/js/libs/stockAvailabilities.js)

**Purpose:** Lazy-load stock availability tooltips for product variants.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.stockAvailabilities.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `getDeliveryPointName(stockCode)` | `(code: string) => string` | Display name of the stocking location. |
| `getDeliveryPointAmount(stockCode, productId, variantCode)` | `(code, productId, variantCode) => number` | Quantity at that location. |
| `getStockAvailabilities(productIds)` | `(ids: number[]) => void` | Fetch availability via AJAX and cache. |
| `setStockAvailabilities()` | `() => void` | Populate tooltips from cached data. |
| `attachEventListeners()` | `() => void` | Register hover handlers. |

**Dependencies:** `shoptet.ajax`, `shoptet.config`.

---

## cart.js

[Source](../shared/js/libs/cart.js)

**Purpose:** Shopping-cart AJAX operations, quantity updates, UI synchronisation. The largest single file in `shared/js/libs/`.
**Module style:** IIFE on `shoptet`; jQuery-heavy.
**Exposes:** `shoptet.cart.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `updateCartButton(count, price)` | `(count, price) => void` | Update the header cart UI. |
| `getCartContent(hide?, callback?)` | `(hide?, cb?) => void` | Fetch cart HTML via AJAX. |
| `getAdvancedOrder()` | `() => void` | Open the advanced-order modal. |
| `functionsForStep1()` | `() => void` | Initialise checkout step 1. |
| `handleCartPostUpdate(action, el)` | `(action, el) => void` | Fire cart-update events. |
| `ajaxSubmitForm(action, form, callingFunctions, replaceContent, displaySpinner, eventElement)` | generic | Generic cart form submission. |
| `updateQuantityInCart($input, timeout)` | `($input, timeout) => void` | Debounced quantity change. |
| `removeItemFromCart($el)` | `($el) => void` | Remove a line item. |
| `toggleRelatedProducts($target)` | `($target) => void` | Toggle related-items visibility. |

The file also binds many event handlers: quantity, discount coupon, free-gift selection, variant validation.

**Dependencies:** jQuery, `shoptet.ajax`, `shoptet.config`, `shoptet.csrf`, `shoptet.modal`, `shoptet.tracking`, DOM mutation events.

---

## cartShared.js

[Source](../shared/js/libs/cartShared.js)

**Purpose:** Modern ES-style API for cart actions (add / remove / update). Branches on `shoptet.abilities.about.generation` to fall back to legacy 1G/2G AJAX.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.cartShared.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `addToCart(payload, silent?, configUrlType?)` | `(payload: Payload, silent?, urlType?) => void` | Add item(s) to cart. Payload accepts `priceId` / `productId` / `productCode` plus optional `amount`. |
| `removeFromCart(payload, silent?)` | `(payload, silent?) => void` | Remove a line item. |
| `updateQuantityInCart(payload, silent?)` | `(payload, silent?) => void` | Update a line item's quantity. |

**Dependencies:** `shoptet.abilities`, `shoptet.config`, `shoptet.cart`, `shoptet.ajax`.

---

## checkout.js

[Source](../shared/js/libs/checkout.js)

**Purpose:** Ordering process step 2 — sticky sidebar layout, VAT validation, country / region change handling.
**Module style:** IIFE on `shoptet`; jQuery-heavy.
**Exposes:** `shoptet.checkout.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `toggleAnotherShipping(scroll)` | `(scroll: boolean) => void` | Delegate to `shoptet.global.toggleAnotherShipping`. |
| `compareHeight($a, $b)` | `($a, $b) => boolean` | True if the two elements' heights differ. |
| `fixSidebar()` | `() => void` | Sticky-position the sidebar (fixed / relative / static per scroll position). |
| `handleWithSidebar()` | `() => void` | Re-evaluate sidebar position when content height changes. |

Also binds handlers for: billing-form VAT validation via AJAX, shipping address selection, country change.

**Dependencies:** jQuery, `shoptet.ajax`, `shoptet.config`, `shoptet.layout`, `shoptet.validator`, `shoptet.tracking`.

---

## checkoutShared.js

[Source](../shared/js/libs/checkoutShared.js)

**Purpose:** Shared checkout logic — payment / shipping method selection, price updates, VAT validation, delivery-point management, external-shipping wiring.
**Module style:** IIFE on `shoptet`; jQuery-heavy.
**Exposes:** `shoptet.checkoutShared.*` (>20 functions).

Highlights:

- `getStatedValues()` — read currently selected shipping / payment IDs.
- `setActiveShippingAndPayments()` — mark the selected methods.
- `updatePrice()` — recalculate totals after a method change.
- `displayApplePay()` — show / hide the Apple Pay button.
- `setupDeliveryShipping()` — initialise external shipping providers via `shoptet.externalShipping[...]`.
- `payu()`, `twisto()` — payment gateway setup.
- 15+ more for billing / shipping relationship management, price summaries, external shipping orchestration.

**Dependencies:** jQuery, `shoptet.ajax`, `shoptet.modal`, `shoptet.config`, `shoptet.layout`.

---

## expressCheckout.js

[Source](../shared/js/libs/expressCheckout.js)

**Purpose:** Full express-checkout flow (guest checkout via Shoptet Pay gateway).
**Module style:** IIFE on `shoptet`; modern JS with TypeScript-style JSDoc.
**Exposes:** `shoptet.expressCheckout.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `initExpressCheckout(disableSpinner?)` | `(noSpinner?) => void` | Load the express-checkout modal. |
| `rerenderExpressCheckoutModal(response, onComplete?, onClosed?, options?)` | `(...) => void` | Update modal content after an action. |
| `changeProductQuantity(amount, itemId, priceId)` | `(amount, itemId, priceId) => void` | Update a cart item from inside the modal. |
| `updateShippingMethod(value)` | `(value: string) => void` | Switch shipping method and recalculate. |
| `updateBillingMethod(value)` | `(value: string) => void` | Switch payment method. |
| `addDiscountCoupon()` | `() => void` | Apply a discount code. |
| `removeDiscountCoupon()` | `() => void` | Remove the applied discount. |
| `getPaymentResult(cartId)` | `(cartId) => void` | Poll for payment completion. |
| `initInvoicingStep()` | `() => void` | Wire up invoice-form validation. |
| `confirmOrder(button)` | `(btn: HTMLElement) => void` | Final order confirmation. |

Also: keydown / form-change listeners, step navigation.

**Dependencies:** `shoptet.ajax`, `shoptet.modal`, `shoptet.validator`, `shoptet.layout`, `shoptet.scripts`.

---

## expressCheckoutGateway.js

[Source](../shared/js/libs/expressCheckoutGateway.js)

**Purpose:** Shoptet Pay gateway iframe listener — polls payment status, reacts to `paymentFinished` event.
**Module style:** Standalone (no IIFE wrapper); async / await.
**Exposes:** runs at module load; no public functions.

Behaviour:

- `checkIfGatewayPaymentFinished(cartId)` — internal poller hitting `/action/ExpressCheckout/checkCartPayment/`.
- `listenForGatewayPaymentFinished(container)` — register a `shoptet-pay-gw:paymentFinished` listener.
- Handles PENDING → redirect, success, failure states.

**Dependencies:** Fetch API.

---

## cofidis.js

[Source](../shared/js/libs/cofidis.js)

**Purpose:** Cofidis payment-plan calculator UI.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.cofidis.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `getElements()` | `() => jQuery` | All Cofidis buttons on the page. |
| `setMinPayment(el, minPayment)` | `(el, min) => void` | Display minimum payment requirement. |
| `calculator($newPriceHolder, $cofidis)` | `($price, $cofidis) => void` | Update the payment-plan URL with the new price. |
| `handleClick(e)` | `(e: Event) => void` | Open the Cofidis calculator popup. |
| `addCalculatorListeners()` | `() => void` | Register click handlers. |

**Dependencies:** jQuery, `shoptet.ajax`, `shoptet.config`.

---

## adminBar.js

[Source](../shared/js/libs/adminBar.js)

**Purpose:** Admin-bar hover and auto-collapse behaviour.
**Module style:** IIFE on `shoptet`; vanilla JS.
**Exposes:** nothing public; auto-initialises on `.admin-bar`.

Hover handlers on `.admin-bar #bar-menu > li` with an 800 ms timeout to auto-close.

**Dependencies:** `shoptet.config`, jQuery.

---

## csrf.js

[Source](../shared/js/libs/csrf.js)

**Purpose:** CSRF token injection, validation, expiration.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.csrf.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `isTokenExpired()` | `() => boolean` | True if the token is past its validity window. |
| `injectToken(form)` | `(form: HTMLFormElement) => void` | Insert or update the hidden CSRF field. |
| `validateToken()` | `() => void` | Synchronous AJAX server-side validation. |

State: `shoptet.csrf.token`, `enabled`, `tokenValidity` (6h 59m).

**Dependencies:** jQuery, `shoptet.modal`, `shoptet.config`.

---

## csrfLink.js

[Source](../shared/js/libs/csrfLink.js)

**Purpose:** Submit CSRF-protected link clicks as POST forms or AJAX.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.csrfLink.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `getFunctionFromString(string)` | `(name: string) => Function \| undefined` | Resolve a function from a string lookup. |
| `loadDataAttributes($elem)` | `($el) => { [name]: value }` | Extract `data-*` attribute pairs. |

Auto-binds handlers on `.csrf-post-js` (form submit) and `.csrf-post-ajax-js` (AJAX) links.

**Dependencies:** jQuery, `shoptet.csrf`, `shoptet.scripts`.

---

## layout.js

[Source](../shared/js/libs/layout.js)

**Purpose:** Viewport and device-capability detection.
**Module style:** IIFE on `shoptet`; ES module imports from typeAssertions.
**Exposes:** `shoptet.layout.*` plus legacy `window.*` aliases.

| Function | Signature | Description |
| --- | --- | --- |
| `clearCache(key)` | `(key: string) => void` | Clear a cached viewport / resolution result. |
| `getScrollBarWidth()` | `() => number` | System scrollbar width in px. |
| `getViewHeight()` | `() => number` | Viewport height in `vh` (1/100). |
| `detectResolution(resolution)` | `(width: number) => boolean` | True if `window.innerWidth > resolution`. |
| `showApplePay()` | `() => boolean` | True if Apple Pay is usable in the browser. |

Legacy aliases on `window`: `getScrollBarWidth`, `getViewHeight`, `detectResolution`.

**Dependencies:** `shared/js/typeAssertions`, DOM, `window.ApplePaySession`.

---

## phoneInput.js

[Source](../shared/js/libs/phoneInput.js)

**Purpose:** International phone-number input — country flag picker, dropdown navigation, country-code translation.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.phoneInput.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `handleFlags(el)` | `(el) => void` | Register flag click / key handlers. |
| `interconnectFlagsWithSelect()` | `() => void` | Bind flag picker to country `<select>`. |
| `hideCountriesSelect(el)` | `(el) => void` | Close the flag dropdown. |
| `setSelectedCountry(el, parentGroup, signal)` | `(el, group, signal) => void` | Mark the chosen flag as selected and update the underlying select. |
| `setLastPreferredCountry(parentGroup)` | `(group) => void` | Update the preferred-flag list. |
| `selectSelectedOption(parentGroup, el, select)` | `(group, el, select) => void` | Update the select element to match. |
| `handleKeyCodes(e, el)` | `(e, el) => void` | Arrow keys, Enter, Escape in the picker. |
| `positionCountriesSelect(el)` | `(el) => void` | Flip the dropdown above the input if it would be clipped below. |
| `translateKeys(key)` | `(key: string) => string` | Convert numeric keys to Czech diacritic shortcuts (`Ě→2`, `Š→3`, …). |

State: `phoneInputKeyup`, `pressedKeys`, `matchedElement`.

**Dependencies:** `shoptet.common.keyCodes`, `shoptet.scripts`, `shoptet.validator.events`.

---

## applePaySdk.js

[Source](../shared/js/libs/applePaySdk.js)

**Purpose:** Conditionally load Apple's official Apple Pay JS SDK on the checkout page when the native browser check has not already approved it. Imported only from [`checkout-3g.js`](../00/js/checkout-3g.js).
**Module style:** Standalone (no IIFE wrapper); top-level code that runs at import time.
**Exposes:** nothing public — pure side-effect script.

Behaviour:

- Skip entirely if `shoptet.layout.showApplePay()` already returns true or `shoptet.config.initApplePaySdk` is false.
- Otherwise inject `<script src="https://applepay.cdn-apple.com/jsapi/1.latest/apple-pay-sdk.js" crossorigin="anonymous">`.
- On `script.onload`, clear the `'showApplePay'` cache key, re-check availability, and either add `apple-pay-available` to the `<html>` element or call `shoptet.checkoutShared.removeApplePay(true)`.

**Dependencies:** `shoptet.layout.showApplePay`, `shoptet.layout.clearCache`, `shoptet.config.initApplePaySdk`, `shoptet.checkoutShared.removeApplePay`.

---

## xyDiscounts.js

[Source](../shared/js/libs/xyDiscounts.js)

**Purpose:** Show / hide X+Y discount flags based on variant selection.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.xyDiscounts.updateFlags`.

| Function | Signature | Description |
| --- | --- | --- |
| `updateFlags(codeId)` | `(codeId: string) => void` | Toggle discount-flag visibility for a variant. |

**Dependencies:** jQuery, `shoptet.scripts`.
