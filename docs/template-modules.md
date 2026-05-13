# Template modules — `00/js/modules/`

The 3G template module layer. All 3G templates (07, 09–14) ship the same JavaScript; only LESS differs per template, so everything here applies to every storefront skin.

This page documents the authored files. The vendored jQuery plugins (`jquery.colorbox.js`, `jquery.cloud-zoom.1.0.2.js`, `jquery-mobile.custom.js`, `jquery.ui.touch-punch.js`) are excluded.

For the `shoptet.*` exposure pattern, see [shoptet-namespace.md](shoptet-namespace.md).

---

## animations.js

[Source](../00/js/modules/animations.js)

**Purpose:** CSS-based fade-in helper with reduced-motion support.
**Module style:** ES module + `shoptet.scripts.registerFunction`.
**Exposes:** `shoptet.animations.fadeIn`.

| Function | Signature | Description |
| --- | --- | --- |
| `fadeIn(el, cleanupMs)` | `(el: HTMLElement, cleanupMs?: number) => void` | Adds `.fade-in` then `.start` to trigger CSS transition; respects `prefers-reduced-motion`; optionally cleans up classes after `cleanupMs`. |

**Dependencies:** `shoptet.scripts`, `requestAnimationFrame`.

---

## cloudZoomInit.js

[Source](../00/js/modules/cloudZoomInit.js)

**Purpose:** Initialise the vendored cloud-zoom plugin on product images and re-bind on image swap.
**Module style:** jQuery on document-ready.
**Exposes:** nothing on `shoptet`; binds DOM events.

**Dependencies:** jQuery, `$.fn.CloudZoom`, `shoptet.config.cloudZoomOptions`.

---

## cookieBar.js

[Source](../00/js/modules/cookieBar.js)

**Purpose:** Show / hide the cookie consent bar depending on viewport size.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.cookieBar.setCookieBarVisibility`.

| Function | Signature | Description |
| --- | --- | --- |
| `setCookieBarVisibility(visible)` | `(visible: boolean) => void` | Adds/removes the visibility class on the bar based on screen resolution and the flag. |

**Dependencies:** jQuery, `shoptet.layout.detectResolution`, `shoptet.config.breakpoints`.

---

## dialogs.js

[Source](../00/js/modules/dialogs.js)

**Purpose:** Open / close native `<dialog>` elements with animation, focus management and modal vs non-modal behaviour. Uses `[data-dialog-id]` triggers and `[data-dialog-close]` close buttons.
**Module style:** ES module + IIFE on `shoptet`; registered via `shoptet.scripts.registerFunction`.
**Exposes:** `shoptet.dialog.initDialogs`, `shoptet.dialog.openDialog`, `shoptet.dialog.closeDialog`.

| Function | Signature | Description |
| --- | --- | --- |
| `initDialogs()` | `() => void` | Bind document-level click handler for `[data-dialog-id]` (open) and `[data-dialog-close]` (close). Called once at module load. |
| `openDialog(dialog, triggerEl)` | `(dialog: HTMLDialogElement, triggerEl: HTMLElement \| null) => void` | Open via `showModal()` / `show()` based on `.js-dialog--modal` class; remembers opener for focus return; sets up Escape / backdrop close and focus trap (modal only). |
| `closeDialog(dialog)` | `(dialog: HTMLDialogElement) => void` | Animate-out via `.is-closing`, await `transitionend` on `.dialog__wrapper`, then `dialog.close()`; honour `shoptet.a11y.reducedMotion`; return focus to opener. |

**Dependencies:** ES module `ensure`/`maybe` from `shared/js/typeAssertions`, `shoptet.focusManagement.setupFocusTrap`, `shoptet.focusManagement.focusFirst`, `shoptet.a11y.reducedMotion`, `shoptet.scripts`.

---

## discussions-and-ratings.js

[Source](../00/js/modules/discussions-and-ratings.js)

**Purpose:** AJAX-paginated discussions and product ratings with accessibility announcements.
**Module style:** IIFE with jQuery event handlers on document ready.
**Exposes:** nothing public — pure DOM event binding.

**Bound events:**

- Click on `.discussions-link` / `.more-discussions` — loads more discussions via AJAX.
- Click on `.ratings-link` / `.more-ratings` — loads more ratings.
- Submit on the discussion form — posts the comment and re-renders the thread.

**Dependencies:** jQuery, `shoptet.common.createDocumentFromString`, `shoptet.animations.fadeIn`, `shoptet.helpers.focusFirst`, `shoptet.screenReader.createLoadingAnnouncer`, `shoptet.validator.removeErrorMessage`, `shoptet.scripts.signalCustomEvent`, `shoptet.scripts.signalDomLoad`, `window.showSpinner`, `window.hideSpinner`.

---

## filters.js

[Source](../00/js/modules/filters.js)

**Purpose:** Product-listing filters: price slider, parameter toggles, AJAX page-section updates with browser history state.
**Module style:** ES module; registers functions on `window`.
**Exposes:** `window.priceFilter`, `window.makeFilterAjaxRequest`, `window.moveFilters`, `window.detectFilters`, `window.parseFilterValuesFromContent`, `window.formatFilterValues`.

| Function | Signature | Description |
| --- | --- | --- |
| `priceFilter(min, max)` | `(min: number, max: number) => void` | Initialise the jQuery UI price slider. |
| `makeFilterAjaxRequest(url, pushHistoryState, successCallback, element, event)` | `(url, push, cb?, el?, event?) => void` | Apply filters via AJAX GET, update listing region, manage history state. |
| `moveFilters($el, targetLocation)` | `($el: jQuery, target: 'default'\|'content') => void` | Move the filter DOM block between sidebar and inline content. |
| `detectFilters()` | `() => void` | Decide whether filters need re-positioning based on viewport visibility. |
| `parseFilterValuesFromContent()` | `() => { min, max }` | Read min/max price values from the listing HTML. |
| `formatFilterValues(min, max)` | `(min: number, max: number) => string` | Currency/number-format the values for display. |

**Dependencies:** jQuery, jQuery UI slider, `shoptet.helpers`, `shoptet.common.createDocumentFromString`, `shoptet.tracking.trackProductsFromPayload`, `shoptet.products.splitWidgetParameters`, `shoptet.skipLinks.initSkipLinks`, `shoptet.topProducts.initTopProducts`, `shoptet.screenReader.createLoadingAnnouncer`, `shoptet.config`, `window.showSpinner` / `hideSpinner` / `dismissMessages`, Fetch API.

---

## focus-visible.js

[Source](../00/js/modules/focus-visible.js)

**Purpose:** Safari workaround for `:focus-visible` on radio inputs during keyboard navigation.
**Module style:** ES module with direct DOM event listeners.
**Exposes:** nothing public.

**Dependencies:** `shared/js/typeAssertions`.

---

## focusManagement.js

[Source](../00/js/modules/focusManagement.js)

**Purpose:** Centralised focus utilities — find first / last focusable descendant, focus a root with a temporary `tabindex="-1"` if needed, set up Tab focus traps and dialog pass-through focus (Tab on opener moves into dialog; Shift+Tab from first item or Tab from last item closes the dialog).
**Module style:** ES module + IIFE on `shoptet`; registered via `shoptet.scripts.registerFunction`. Also installs deprecation shims on `shoptet.helpers.findFirstFocusable` and `shoptet.helpers.focusFirst` (sunset 2026-06-30).
**Exposes:** `shoptet.focusManagement.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `findFirstFocusable(wrapper)` | `(wrapper: HTMLElement) => HTMLElement \| null` | First focusable descendant (anchor / button / input / select / textarea / `[tabindex>=0]`). |
| `findLastFocusable(wrapper)` | `(wrapper: HTMLElement) => HTMLElement \| null` | Last focusable descendant. |
| `focusFirst(root, tryInside?, scrollToEl?, isDialog?)` | `(root, tryInside?: boolean, scrollToEl?: boolean, isDialog?: boolean) => void` | Focus `root` or its first focusable descendant; dialog mode picks `[autofocus]` → `aria-labelledby` target → first focusable → dialog itself; optionally scrolls (never when `isDialog`). |
| `setupFocusTrap(root, signal)` | `(root: HTMLElement, signal: AbortSignal) => void` | Tab / Shift+Tab loops focus inside `root` until `signal` aborts. |
| `setupDialogPassThroughFocus(root, opener, windowTarget, onClose)` | `(root, opener, windowTarget: string, onClose: () => void) => void` | Tab on last item or Shift+Tab on opener closes the dialog; Tab on opener (while open) moves focus into the dialog. |

**Dependencies:** ES module `ensure`/`maybe` from `shared/js/typeAssertions`, `shoptet.scroll.scrollToEl`, `shoptet.dev.deprecated`, `shoptet.scripts`.

---

## globalFunctions.js

[Source](../00/js/modules/globalFunctions.js)

The biggest single file in this directory. Mostly attaches utilities to `window` for use by inline Smarty templates; a few attach to `shoptet.global.*`. Grouped by concern below.

**Module style:** mixed jQuery / vanilla JS; IIFE on `shoptet`.

### Messaging, spinner

| Function | Signature | Description |
| --- | --- | --- |
| `window.showMessage(content, type, id, cancel, overlay, parent)` | `(html: string, type: string, id?, cancel?, overlay?, parent?) => void` | Render a notification with optional cancel action and overlay. |
| `window.hideMsg(action?)` | `(action?: 'remove') => void` | Hide / remove notification element. |
| `window.dismissMessages()` | `() => void` | Auto-hide notifications after `shoptet.config.dismissTimeout`. |
| `window.cancelAction(id)` | `(id: string) => void` | Cancel a queued action and hide the message. |
| `window.showSpinner()` | `() => void` | Display the loading-spinner overlay. |
| `window.hideSpinner()` | `() => void` | Hide the loading spinner with fade-out. |
| `window.initTooltips()` | `() => void` | Initialise Bootstrap tooltips on `.show-tooltip` elements. |
| `window.fixTooltipAfterChange(element)` | `(el: HTMLElement) => void` | Update tooltip content and visibility after the underlying value changes. |

### Layout, scroll, navigation

| Function | Signature | Description |
| --- | --- | --- |
| `window.detectScrolled(direction)` | `(direction: 'up'\|'down') => void` | Toggle `scrolled-up` / `scrolled-down` body classes. |
| `window.addPaddingToOverallWrapper()` | `() => void` | Add bottom padding for positioned footer on mobile. |
| `window.moveElementAfterSelector($what, $where)` | `($what, $where) => void` | jQuery-insert element after another. |
| `window.updateQueryStringParameter(key, value)` | `(k, v) => void` | Update / add a URL query parameter and navigate. |
| `window.unveilElementByHash(elementId)` | `(id: string) => void` | Activate a tab / accordion based on `location.hash`. |
| `window.locationSearchToObject()` | `() => Record<string,string>` | Parse `location.search` into a plain object. |
| `window.getRelativeOffset($el, $parent)` | `($el, $parent) => { top, left }` | Element offset relative to a parent. |
| `window.resizeEndCallback()` | `() => void` | Batch resize handler for layout adjustments. |

### Video

| Function | Signature | Description |
| --- | --- | --- |
| `window.detectVideoBackground($video)` | `($video) => boolean` | Detect whether the header video exists. |
| `window.detectVideoBackgroundHeight($videoWrapper)` | `($wrap) => number` | Return the header video wrapper height. |
| `window.pauseVideo($video)` | `($video) => void` | Pause if not paused. |
| `window.resumeVideo($video)` | `($video) => void` | Play if not playing. |
| `window.handleWithVideo($video, $videoWrapper)` | `($v, $w) => void` | Pause / resume header video based on viewport visibility. |

### Region / shipping (checkout helpers)

| Function | Signature | Description |
| --- | --- | --- |
| `shoptet.global.updateSelectedRegions($el)` | `($el: jQuery) => void` | Update region select options when the country changes. |
| `shoptet.global.toggleRegionsWrapper()` | `() => void` | Show / hide the regions wrapper based on visible selects. |
| `shoptet.global.restoreDefaultRegionSelect($el, val)` | `($el, val) => void` | Restore default region selection. |
| `shoptet.global.toggleAnotherShipping(scroll)` | `(scroll: boolean) => void` | Toggle the alternate shipping address form. |
| `window.toggleContacts($el)` | `($el: jQuery) => void` | Toggle contact-info visibility in checkout. |

### Misc

| Function | Signature | Description |
| --- | --- | --- |
| `window.toggleText($el, text, showText, hideText)` | `($el, currentText, showText, hideText) => void` | Toggle HTML element text between two states. |
| `window.delay(callback, ms)` | `(cb: () => void, ms: number) => void` | Named debounce timeout. |
| `window.setCarouselHeight(selector)` | `(selector: string) => void` | Compute carousel min-height from image aspect ratios. |
| `window.initColorbox()` | `() => void` | Initialise Colorbox on product image galleries. |
| `window.initDatepickers()` | `() => void` | Initialise jQuery UI datepicker on `.datepicker.birthdate`. |

**Dependencies:** jQuery, jQuery UI datepicker, Colorbox, `shoptet.layout`, `shoptet.modal`, `shoptet.config`, `shoptet.messages`, `shoptet.validator`, `shoptet.cookie`, `shoptet.images`, `shoptet.common`, `shoptet.helpers`, `shoptet.scripts`, Bootstrap tab plugin, `scrollToEl`.

---

## layout.js

[Source](../00/js/modules/layout.js)

**Purpose:** Compute the scroll-offset accounting for fixed header / admin bar / nav and publish it as a CSS variable.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.layout.getScrollOffset`.

| Function | Signature | Description |
| --- | --- | --- |
| `getScrollOffset(extra?)` | `(extra?: number) => number` | Pixel offset, sum of fixed header + admin bar + navigation heights plus an optional extra. |

**Dependencies:** `shoptet.layout.detectResolution`, `shoptet.abilities`, `shoptet.config.breakpoints`.

---

## menu.js

[Source](../00/js/modules/menu.js)

**Purpose:** Main menu — open / close, submenus, overflow handling, helper-box collapse.
**Module style:** IIFE on `shoptet` with jQuery event handlers.
**Exposes:** `shoptet.menu.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `toggleMenu()` | `() => void` | Toggle overlay and main menu visibility. |
| `splitMenu()` | `() => void` | Redistribute overflowing items into the menu helper. |
| `splitHelperMenu(i)` | `(i: number) => void` | Manage helper-menu item visibility for `i` visible items. |
| `showSubmenu($el)` | `($el: jQuery) => void` | Display a submenu and handle third-level overflow. |
| `hideSubmenu()` | `() => void` | Hide all open submenus. |
| `updateMenu()` | `() => void` | Re-split when fonts load. |
| `showMenuHelper()` | `() => void` | Show the helper box. |
| `hideMenuHelper()` | `() => void` | Hide the helper box. |
| `hideNavigation()` | `() => void` | Hide navigation window and reset state. |

**Dependencies:** jQuery, `shoptet.config`, `shoptet.layout.detectResolution`, `shoptet.abilities.config.navigation_breakpoint`, `window.getRelativeOffset`, `window.hideMsg`.

---

## popups.js

[Source](../00/js/modules/popups.js)

**Purpose:** Slide-in popup windows for cart, search, register, and navigation. Includes focus-trap and ARIA management.
**Module style:** ES module + IIFE on `shoptet`.
**Exposes:** `shoptet.popups.hideContentWindows`, `shoptet.popups.showPopupWindow`.

| Function | Signature | Description |
| --- | --- | --- |
| `hideContentWindows(skipElement?)` | `(skip?: HTMLElement) => void` | Hide all open popups except the optional skipped one; toggles body classes and ARIA. |
| `showPopupWindow(target, show, ariaControls, focusTrapping)` | `(target: string, show: boolean, ariaControls?: string, focusTrap?: boolean) => void` | Open / close the named popup (cart / search / register / navigation). |

Internal helpers: `focusTrap`, `hidePopupWindowCallback`, `focusPopupWindow`.

**Dependencies:** ES module imports from `shared/js/typeAssertions`, jQuery, `shoptet.cookieBar`, `shoptet.cart.getCartContent`, `shoptet.config.animationDuration`, `shoptet.images.unveil`, `shoptet.ajax`, `shoptet.validator.initValidator`, `shoptet.scripts.signalDomLoad`, `window.toggleRequiredAttributes`, `window.initDatepickers`, `window.initTooltips`, `window.clearSearchFocus`, optional `window.fbq`.

---

## productInnerSecondImage.js

[Source](../00/js/modules/productInnerSecondImage.js)

**Purpose:** Swap the product image on hover for a preview effect.
**Module style:** jQuery on document-ready with delegated handlers.
**Exposes:** nothing public.

**Dependencies:** jQuery.

---

## productSorting.js

[Source](../00/js/modules/productSorting.js)

**Purpose:** Product-list sort dropdown with AJAX apply and screen-reader announcements.
**Module style:** ES module + IIFE on `shoptet`; only activates when `shoptet.config.ums_a11y_category_page` is set.
**Exposes:** `shoptet.productSorting.initProductSorting`.

| Function | Signature | Description |
| --- | --- | --- |
| `initProductSorting()` | `() => void` | Initialise / re-initialise sort UI; safe to call multiple times. |

Internal helpers: `sort(btn)`, `registerListeners(root)`.

**Dependencies:** ES module imports, `shoptet.config.ums_a11y_category_page`, `shoptet.screenReader.createLoadingAnnouncer`, `shoptet.messages`, `shoptet.scripts`, `window.makeFilterAjaxRequest`.

---

## products.js

[Source](../00/js/modules/products.js)

**Purpose:** Product detail and listing behaviour — image swap, thumbnail layout, ratings stars, related-product carousel, quantity controls, parameter overflow.
**Module style:** IIFE on `shoptet` with jQuery event handlers; vanilla JS for widgets.
**Exposes:** `shoptet.products.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `splitWidgetParameters()` | `() => void` | Hide / show parameter values that overflow their parent width. |
| `setThumbnailsDirection()` | `() => void` | Decide horizontal vs vertical thumbnail layout based on features and resolution. |
| `checkThumbnails(direction, action, reset)` | `(dir, action, reset?) => void` | Calculate thumbnail dimensions and overflow state. |
| `unveilProductVideoTab(href)` | `(href: string) => void` | Load iframe videos when a tab becomes active. |

Internal helpers (selected): `splitSingleWidgetParameter`, `replaceImage`, `highlightActiveThumbnail`, `browseProducts`, `setThumbnails`, `switchThumbnails`, `checkDiscountFlag`, `changeStyle`, `setStyle`, `returnStyle`, `changeQuantity`.

**Dependencies:** jQuery, `shoptet.variantsSimple.handler`, `shoptet.variantsSplit.handler`, `shoptet.variantsCommon`, `shoptet.modal`, `shoptet.ajax`, `shoptet.csrf`, `shoptet.tracking.trackProductsFromPayload`, `shoptet.common`, `shoptet.quantity.updateQuantity`, `shoptet.animations.fadeIn`, `shoptet.images.unveil`, `shoptet.helpers.focusFirst`, `shoptet.scripts.signalCustomEvent`, `shoptet.scripts.signalDomLoad`, `shoptet.screenReader`, `shoptet.config`, `shoptet.abilities`.

---

## quantity.js

[Source](../00/js/modules/quantity.js)

**Purpose:** Quantity input — min/max/step validation, increment/decrement buttons, screen-reader announcements, tooltip feedback.
**Module style:** ES module + IIFE on `shoptet`.
**Exposes:** `shoptet.quantity.updateQuantity`, `shoptet.quantity.announceQuantityLimits`, `shoptet.quantity.enforceAndAnnounceLimits`.

| Function | Signature | Description |
| --- | --- | --- |
| `updateQuantity(el, min, max, decimals, action, callback?)` | `(el, min, max, decimals, action: '+'\|'-', cb?) => void` | Increment or decrement with bounds checking and announcements. |
| `announceQuantityLimits(el, action)` | `(el, action) => void` | Show tooltip and announce limit hit via screen reader. |
| `enforceAndAnnounceLimits(el, event)` | `(el, event) => void` | Validate input on arrow keys and text input events. |

Internal: `calculateQuantity(value, step, decimals, action)`.

**Dependencies:** jQuery, `shoptet.helpers`, `shoptet.variantsCommon.hideQuantityTooltips`, `shoptet.screenReader.announceToScreenReader`, `shoptet.messages`, `shoptet.dev.deprecated`, `shoptet.scripts`.

---

## quantityDiscounts.js

[Source](../00/js/modules/quantityDiscounts.js)

**Purpose:** Quantity-discount table — show pricing for tiered quantities, refresh on variant / quantity changes.
**Module style:** IIFE on `shoptet` with cached DOM references.
**Exposes:** nothing publicly named, but listens to `ShoptetSimpleVariantChange` and similar events.

**Dependencies:** `shoptet.quantity.enforceAndAnnounceLimits`, `shoptet.scripts.signalCustomEvent`, jQuery number-format helper.

---

## ratingStars.js

[Source](../00/js/modules/ratingStars.js)

**Purpose:** Hook up star-rating inputs — sync visual fill on selection, manage per-star `aria-describedby` text, clear error states, pre-select stars from `?preselectStars=...` URL parameter, and unveil the vote form when arriving via the rating email link.
**Module style:** ES module + IIFE on `shoptet`; registered via `shoptet.scripts.registerFunction`. Idempotent: skips forms already marked with `data-rating-stars-init="true"`.
**Exposes:** `shoptet.ratingStars.initRatingStars`.

| Function | Signature | Description |
| --- | --- | --- |
| `initRatingStars(form)` | `(form: HTMLFormElement) => void` | Modal-form entry point — sets up listeners without the email-link unveil branch. |

Internal: `setupRatingStars(form, options)` — the same wiring with `{ enableUnveil }` to handle the in-page tab form on `DOMContentLoaded`.

**Dependencies:** ES module `ensureEvery`/`isHTMLInputElement`/`isHTMLFormElement`/`maybe` from `shared/js/typeAssertions`, `shoptet.scripts.signalCustomEvent` (`ShoptetRatingStarsUnveiled`).

---

## ratingSubpage.js

[Source](../00/js/modules/ratingSubpage.js)

**Purpose:** Drive the rating modal opened from the product detail page — fetch the rating form into a Colorbox modal, wire up validation and `ratingStars`, submit asynchronously, then patch the rating summary blocks on the page from the response payload.
**Module style:** ES module; no IIFE wrapper; runs `init()` on `DOMContentLoaded`. Auto-opens the modal when `?guid=...&preselectStars=...#ratingWrapper` is present (email-link flow) and the shop is in simple-rating mode.
**Exposes:** nothing public — all functions module-private. Document-level submit (`#dialogFormRating`) and click (`.js-ratingDialogTrigger`) handlers.

Key internal functions:

- `openRatingModal()` — open Colorbox with the rating form, init validator and `shoptet.ratingStars.initRatingStars`, announce loading via `shoptet.screenReader.createLoadingAnnouncer`, signal `ShoptetRatingModalOpened`.
- `submitRatingForm(form)` — serialise via `shoptet.common.serializeForm`, POST via `shoptet.ajax.makeAjaxRequest` to `shoptet.config.rateProduct`, show success / error message, refresh the product summary on success, signal `ShoptetRatingUpdated`.
- `updateRatingSummaryFromPayload(response)` — replace every `.stars--productDetail` and `.starsLabel--productDetail` on the page with the payload's clone; signal `ShoptetRatingSummaryUpdated`.
- `autoOpenRatingModalFromEmailLink()` — open the modal automatically when the URL hash is `#ratingWrapper` and simple-rating config is enabled.

**Dependencies:** ES module imports from `shared/js/typeAssertions`, `shoptet.config.rating`, `shoptet.config.rateProduct`, `shoptet.screenReader.createLoadingAnnouncer`, `shoptet.modal`, `shoptet.validator.initValidator` / `formContainsInvalidFields`, `shoptet.ratingStars.initRatingStars`, `shoptet.ajax.makeAjaxRequest`, `shoptet.common.serializeForm`, `shoptet.scripts.signalCustomEvent`, `window.initTooltips`, `window.showMessage`, jQuery (for validator init).

---

## screenReader.js

[Source](../00/js/modules/screenReader.js)

**Purpose:** ARIA-live announcements and a `LoadingAnnouncer` class for long-running operations.
**Module style:** IIFE on `shoptet`; class-based.
**Exposes:** `shoptet.screenReader.announceToScreenReader`, `shoptet.screenReader.createLoadingAnnouncer`.

| Function / Class | Signature | Description |
| --- | --- | --- |
| `announceToScreenReader(message, liveType, options)` | `(msg, 'polite'\|'assertive', { clearAfter?: number }) => void` | Inject text into an ARIA-live region with optional auto-clear. |
| `createLoadingAnnouncer(options)` | `(opts) => LoadingAnnouncer` | Factory for a `LoadingAnnouncer` instance. |

**`LoadingAnnouncer` methods:**

| Method | Description |
| --- | --- |
| `begin(wrapper)` | Announce initial message, set `aria-busy`, schedule periodic pings. |
| `end()` | Clear timers, announce success, remove `aria-busy`. |
| `error()` | Announce error assertively, remove `aria-busy`. |
| `ping()` | Announce a "still loading" mid-flight message. |

**Dependencies:** `shoptet.messages`, `shoptet.dev.deprecated`, `shoptet.scripts`.

---

## scroll.js

[Source](../00/js/modules/scroll.js)

**Purpose:** Smooth scroll-to-element with reduced-motion support and message-bar offset.
**Module style:** ES module + IIFE on `shoptet`.
**Exposes:** `shoptet.scroll.scrollToEl`, `window.scrollToEl` (legacy alias).

| Function | Signature | Description |
| --- | --- | --- |
| `scrollToEl($el)` | `($el: HTMLElement \| jQuery) => void` | Smooth-scroll to the element, honour `prefers-reduced-motion`, account for notification bar height. |

**Dependencies:** ES module imports, `requestAnimationFrame`.

---

## search.js

[Source](../00/js/modules/search.js)

**Purpose:** Search whisperer (autocomplete), recommended-products carousel, search-focus state, results pagination.
**Module style:** jQuery on document-ready with `window.*` registrations.
**Exposes:** `window.fulltextSearch`, `window.clearSearchFocus`, `window.checkMinimalLength`, `window.detectRecommended`, `window.hideRecommended`, `window.updateIndicator`, `window.switchRecommended`.

| Function | Signature | Description |
| --- | --- | --- |
| `fulltextSearch($input, $container)` | `($input, $container) => void` | Wire up the AJAX autocomplete and quantity controls in the whisperer. |
| `clearSearchFocus()` | `() => void` | Remove the search-focus state with an intentional delay. |
| `checkMinimalLength($el)` | `($el) => boolean` | Enforce 3-character minimum query. |
| `detectRecommended()` | `() => number` | Number of recommended-product rows. |
| `hideRecommended()` | `() => void` | Remove recommended-products navigation. |
| `updateIndicator(className)` | `(className: string) => void` | Update the carousel position indicator. |
| `switchRecommended(target)` | `(target: 'prev'\|'next') => void` | Navigate the recommended-products carousel. |

**Dependencies:** jQuery, `shoptet.abilities.feature`, `shoptet.common.createDocumentFromString`, `shoptet.images.unveil`, `shoptet.scripts.signalDomLoad`, `shoptet.products.splitWidgetParameters`, `shoptet.products.changeQuantity`, `shoptet.popups.showPopupWindow`, `shoptet.screenReader.createLoadingAnnouncer`, `shoptet.helpers.focusFirst`, `window.delay`, `window.showMessage`, `window.showSpinner` / `hideSpinner`, `window.initTooltips`.

---

## skipLinks.js

[Source](../00/js/modules/skipLinks.js)

**Purpose:** Skip-to-content links — toggle visibility based on tab direction and apply focus to target regions.
**Module style:** ES module with direct DOM listeners.
**Exposes:** `shoptet.skipLinks.initSkipLinks`.

| Function | Signature | Description |
| --- | --- | --- |
| `initSkipLinks()` | `() => void` | Find skip links and targets, register listeners; idempotent. |

Internal: `getSkipLinksAndTargets`, `toggleSkipLinkVisibility`, `registerListeners`.

**Dependencies:** ES module imports, `shoptet.screenReader.announceToScreenReader`, `shoptet.messages`.

---

## slider.js

[Source](../00/js/modules/slider.js)

**Purpose:** Reusable slider/carousel — touch/mouse drag, keyboard, pagination, autoplay, responsive per-page configuration.
**Module style:** ES module + class + IIFE on `shoptet`.
**Exposes:** `shoptet.slider.runSlider`, `shoptet.slider.runProductSlider`.

### `Slider` class

Constructor: `new Slider(options)` — `options` includes `frame`, `slides`, `slidesPerPage`, `autoplay`, `keyboardNav`, callbacks, etc.

| Method | Description |
| --- | --- |
| `goTo(index, callback?)` | Slide to a specific index. |
| `next(howManySlides?, callback?)` | Advance by `n` (default 1). |
| `prev(howManySlides?, callback?)` | Go back by `n`. |
| `remove(index, callback?)` | Remove the item at `index` and rebuild. |
| `insert(item, index, callback?)` | Insert at `index` and rebuild. |
| `prepend(item, callback?)` | Add to the start. |
| `append(item, callback?)` | Add to the end. |
| `destroy(restoreMarkup, callback?)` | Tear down and optionally restore original markup. |
| `autoplayStart()` | Start autoplay. |
| `autoplayStop()` | Stop autoplay. |

### Module-level

| Function | Signature | Description |
| --- | --- | --- |
| `runSlider(options)` | `(options) => Slider` | Construct a `Slider` and track it in `activeSliders`. |
| `runProductSlider(selector)` | `(selector: string) => void` | Initialise a product slider with the standard responsive config. |

**Dependencies:** ES module imports, `shoptet.layout.detectResolution`, `shoptet.abilities`, `shoptet.config`, `shoptet.dev.deprecated`, `shoptet.scripts`.

---

## smartLabels.js

[Source](../00/js/modules/smartLabels.js)

**Purpose:** Floating-label behaviour for form inputs — show / hide based on populated state and focus.
**Module style:** jQuery plugin; only registers when `shoptet.abilities.feature.smart_labels` is truthy.
**Exposes:** `$.fn.SmartLabels`.

| Method | Description |
| --- | --- |
| `$('.smart-label-wrapper').SmartLabels(options?)` | Manage label visibility and focus states; also positions the phone-input label. |

**Dependencies:** jQuery, `shoptet.abilities.feature.smart_labels`.

---

## tabsAccordion.js

[Source](../00/js/modules/tabsAccordion.js)

**Purpose:** Accordion behaviour for collapsible content; only active when `shoptet.abilities.feature.tabs_accordion` is truthy.
**Module style:** jQuery on document-ready.
**Exposes:** nothing public.

**Dependencies:** jQuery, `shoptet.abilities.feature.tabs_accordion`, `shoptet.products.unveilProductVideoTab`.

---

## tabsResponsive.js

[Source](../00/js/modules/tabsResponsive.js)

**Purpose:** Responsive navigation — collapse overflowing menu items into a hidden dropdown.
**Module style:** jQuery plugin; only active when `shoptet.abilities.feature.tabs_responsive` is truthy.
**Exposes:** `$.fn.shpResponsiveNavigation`.

| Method | Description |
| --- | --- |
| `$('.responsive-nav').shpResponsiveNavigation()` | Manage responsive menu item redistribution and toggle. |

**Dependencies:** jQuery, `shoptet.abilities.feature.tabs_responsive`.

---

## topNavigationMenu.js

[Source](../00/js/modules/topNavigationMenu.js)

**Purpose:** Top navigation overflow handling — show a trigger when items don't fit.
**Module style:** jQuery on document-ready with `window.*` registrations; only active when `shoptet.abilities.feature.top_navigation_menu` is truthy.
**Exposes:** `window.topMenuFits`, `window.showTopMenuTrigger`, `window.hideTopMenuTrigger`.

| Function | Signature | Description |
| --- | --- | --- |
| `topMenuFits()` | `() => boolean` | Check whether items fit; redistribute overflow. |
| `showTopMenuTrigger()` | `() => void` | Add the trigger-visibility class. |
| `hideTopMenuTrigger()` | `() => void` | Remove the trigger-visibility class. |

**Dependencies:** jQuery, `shoptet.abilities.feature.top_navigation_menu`, `shoptet.layout.detectResolution`, `shoptet.config.breakpoints`, `shoptet.common.fitsToParentWidth`.

---

## topProducts.js

[Source](../00/js/modules/topProducts.js)

**Purpose:** Top-products block — expand / collapse toggle with responsive visibility.
**Module style:** ES module + IIFE on `shoptet`; only active when `shoptet.config.ums_a11y_category_page` is set.
**Exposes:** `shoptet.topProducts.initTopProducts`.

| Function | Signature | Description |
| --- | --- | --- |
| `initTopProducts()` | `() => void` | Initialise / re-initialise the block; removes prior listeners first. |

Internal: `getTopProductElements`, `updateTopProductsState`, `handleToggleClick`.

**Dependencies:** ES module imports, `shoptet.config.ums_a11y_category_page`, `shoptet.layout.detectResolution`, `shoptet.config.breakpoints`, `shoptet.scripts`.

---

## tooltips.js

[Source](../00/js/modules/tooltips.js)

**Purpose:** Next-generation tooltip system intended to replace the jQuery Bootstrap tooltip. Defines a single shared bubble (positioned dynamically, with arrow) plus event-delegated triggers via `.show-tooltip_new` / `.question-tooltip_new`.
**Module style:** ES module. **Status: currently inert.** The registration / global-export block at the bottom of the file is wrapped in `/* ... */` (with a `TODO: Uncomment this after the jQuery tooltips are removed`), so the functions are defined but never attached to `shoptet.tooltips` and the file has no runtime effect today.
**Exposes (once enabled):** `shoptet.tooltips.initTooltips`, `showTooltip`, `hideTooltip`, `updateTooltipBubbleContent`, `repositionTooltipBubble`, plus deprecated `window.initTooltips` / `window.fixTooltipAfterChange` / `$.fn.tooltip` shims (sunset 2026-12-31).

| Function | Signature | Description |
| --- | --- | --- |
| `initTooltips()` | `() => void` | Create / reuse the shared `#tooltip_bubble`; migrate legacy `title` → `data-title`; ensure `type="button"`; default `aria-label` for `.question-tooltip_new`; bind document-level `mouseover` / `mouseout` / `focusout` / `click` / `keydown` (Escape) and `resize`. |
| `showTooltip(button, announceToScreenReader?)` | `(button: HTMLButtonElement, announceToScreenReader?: boolean) => void` | Update content from `data-title`, show the bubble, position it, optionally announce via `shoptet.screenReader`. |
| `hideTooltip(force?)` | `(force?: boolean) => void` | Hide immediately or after a 600 ms delay; clear stored open trigger. |
| `updateTooltipBubbleContent(content)` | `(content: string) => void` | Replace the bubble's text content. |
| `repositionTooltipBubble()` | `() => void` | Re-run positioning for the currently open tooltip (resize handler). |

Internal: `positionTooltip(button)` — viewport-aware placement with arrow flip and edge clamping; `createTooltipBubble()` / `getTooltipBubble()` for lazy DOM creation.

**Dependencies:** ES module imports from `shared/js/typeAssertions`, `shoptet.messages.moreInfo`, optional `shoptet.screenReader.announceToScreenReader`, optional `shoptet.dev.deprecated`, optional `shoptet.scripts.registerFunction` (only used by the commented-out registration block).

---

## unveil.js

[Source](../00/js/modules/unveil.js)

**Purpose:** Deprecated jQuery-plugin wrapper that delegates to `shoptet.images.unveil()`.
**Module style:** jQuery plugin (deprecation shim).
**Exposes:** the `$.fn.unveil` plugin name; logs a deprecation notice via `shoptet.dev.deprecated`.

**Dependencies:** jQuery, `shoptet.dev.deprecated`, `shoptet.images.unveil`.

---

## validator.js

[Source](../00/js/modules/validator.js)

**Purpose:** Template-layer wiring of the validator framework — registers field validators (required, email, password, fullname, address), shows messages, handles async phone validation on submit.
**Module style:** IIFE with jQuery event handlers on document-ready.
**Exposes:** `shoptet.validator.initValidator`, `shoptet.validator.showValidatorMessage`, `shoptet.validator.removeValidatorMessage`, `shoptet.validator.removeErrorMessage`, plus the per-field `validators.*` registry and the `shoptetFormValidator` object.

| Function | Signature | Description |
| --- | --- | --- |
| `initValidator($el, settings?)` | `($el: jQuery, settings?) => void` | Attach validation to forms or fields within `$el`. |
| `showValidatorMessage($el, message, cssClass)` | `($el, message, cssClass) => void` | Display a validation message. |
| `removeValidatorMessage($el)` | `($el) => void` | Remove a previously shown message. |
| `removeErrorMessage(el, messageType)` | `(el, messageType?) => void` | Remove a specific error message type (used by discussions module too). |

**Per-field validators** (internal but invoked by the framework):

| Validator | Validates |
| --- | --- |
| `required` | Non-empty input or checked checkbox/radio. |
| `password` | Confirmation field matches. |
| `email` | Format, TLD length, against a list of common typos in `invalidEmails`. |
| `fullname` | Contains whitespace or special space character. |
| `billStreet` | Street format warning. |
| `billHouseNumber` | House number format. |

The `invalidEmails` array (e.g. `'seznam.com'`, `'gmal.com'`) drives typo detection; treat it as a maintenance list.

**Dependencies:** jQuery, `shoptet.messages`, `shoptet.scripts.signalCustomEvent`, `shoptet.validatorPhone.ajaxPending`, `shoptet.custom.postFailedValidation`, `shoptet.custom.postSuccessfulValidation`, `window.scrollToEl`.

See also [shared-validator.md](shared-validator.md) for the underlying framework and the per-field validators that ship in `shared/js/libs/validator/`.

---

## watchdog.js

[Source](../00/js/modules/watchdog.js)

**Purpose:** Product watchdog signup — customer opts in to be notified about price drops, promotions, stock changes.
**Module style:** IIFE on `shoptet` with jQuery event handlers on document-ready.
**Exposes:** `shoptet.watchdog.initWatchdog`.

| Function | Signature | Description |
| --- | --- | --- |
| `initWatchdog()` | `() => void` | Initialise the watchdog form: load existing subscription data, manage checkboxes, post via AJAX. |

**Dependencies:** jQuery, `shoptet.ajax.AjaxResponse`, `shoptet.config`, `shoptet.modal`, `shoptet.messages`, `shoptet.scripts.signalDomLoad`, `window.showMessage`, `window.hideMsg`, `window.showSpinner` / `hideSpinner`.
