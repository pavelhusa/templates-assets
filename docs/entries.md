# Entry bundles

The four files under `00/js/` are bundle entry points. They contain no feature code — only `import` statements that compose the final bundles produced by the upstream build. The actual JavaScript lives in the imported modules.

The bundles share no manifest; the order of imports inside each entry file is the load order at runtime. See [shoptet-namespace.md § Load order](shoptet-namespace.md#load-order) for why the order matters.

## [00/js/main-3g.js](../00/js/main-3g.js)

Storefront main bundle — every 3G template (07, 09–14) loads this. Compose-only.

Imports, in order:

| # | Import | Role | Documented |
| --- | --- | --- | --- |
| 1–5 | `./bootstrap/transition`, `carousel`, `dropdown`, `tab`, `tooltip` | Vendored Bootstrap 3 JS | not in scope |
| 6–15 | `../../shared/js/jqueryui/*` (core, widget, mouse, position, autocomplete, button, datepicker, menu, selectmenu, slider) | Vendored jQuery UI | not in scope |
| 16 | `./modules/jquery.ui.touch-punch` | Vendored | not in scope |
| 17 | `./modules/jquery.colorbox` | Vendored | not in scope |
| 18 | `../../shared/js/libs/scripts` | Registration & signaling framework | [shared-libs.md#scriptsjs](shared-libs.md#scriptsjs) |
| 19 | `../../shared/js/libs/dev` | Dev helpers, deprecation warnings | [shared-libs.md#devjs](shared-libs.md#devjs) |
| 20 | `../../shared/js/libs/custom` | Theme override hooks | [shared-libs.md#customjs](shared-libs.md#customjs) |
| 21 | `../../shared/js/libs/config` | Config constants | [shared-libs.md#configjs](shared-libs.md#configjs) |
| 22 | `../../shared/js/libs/runtime` | Resize debouncing | [shared-libs.md#runtimejs](shared-libs.md#runtimejs) |
| 23 | `../../shared/js/libs/modal` | Modal wrapper | [shared-libs.md#modaljs](shared-libs.md#modaljs) |
| 24 | `../../shared/js/libs/common` | DOM / form utilities | [shared-libs.md#commonjs](shared-libs.md#commonjs) |
| 25 | `../../shared/js/libs/cookie` | Cookie helpers | [shared-libs.md#cookiejs](shared-libs.md#cookiejs) |
| 26 | `../../shared/js/libs/consent` | GDPR consent | [shared-libs.md#consentjs](shared-libs.md#consentjs) |
| 27 | `../../shared/js/libs/helpers` | Number / currency formatting | [shared-libs.md#helpersjs](shared-libs.md#helpersjs) |
| 28 | `../../shared/js/libs/images` | Lazy-load (`unveil`) | [shared-libs.md#imagesjs](shared-libs.md#imagesjs) |
| 29 | `../../shared/js/libs/tracking` | Analytics integrations | [shared-libs.md#trackingjs](shared-libs.md#trackingjs) |
| 30 | `../../shared/js/libs/image360` | 360° product viewer | [shared-libs.md#image360js](shared-libs.md#image360js) |
| 31 | `../../shared/js/libs/stockAvailabilities` | Stock tooltips | [shared-libs.md#stockavailabilitiesjs](shared-libs.md#stockavailabilitiesjs) |
| 32 | `./libs/variants/surcharges` | Variant surcharges | [template-libs.md#variantssurchargesjs](template-libs.md#variantssurchargesjs) |
| 33 | `./libs/variants/common` | Variant common helpers | [template-libs.md#variantscommonjs](template-libs.md#variantscommonjs) |
| 34 | `./libs/variants/variantsData` | Dynamic variant data | [template-libs.md#variantsvariantsdatajs](template-libs.md#variantsvariantsdatajs) |
| 35 | `./libs/variants/simple` | Single-select variants | [template-libs.md#variantssimplejs](template-libs.md#variantssimplejs) |
| 36 | `./libs/variants/split` | Split (multi-param) variants | [template-libs.md#variantssplitjs](template-libs.md#variantssplitjs) |
| 37 | `./libs/variants/unavailable` | Unavailable-option handling | [template-libs.md#variantsunavailablejs](template-libs.md#variantsunavailablejs) |
| 38 | `../../shared/js/libs/ajax/request` | AJAX request | [shared-ajax.md#requestjs](shared-ajax.md#requestjs) |
| 39 | `../../shared/js/libs/ajax/response` | AjaxResponse | [shared-ajax.md#responsejs](shared-ajax.md#responsejs) |
| 40 | `../../shared/js/libs/cart` | Cart operations | [shared-libs.md#cartjs](shared-libs.md#cartjs) |
| 41 | `../../shared/js/libs/cartShared` | Cart shared API | [shared-libs.md#cartsharedjs](shared-libs.md#cartsharedjs) |
| 42 | `../../shared/js/libs/validator/helpers` | `toggleRequiredAttributes` | [shared-validator.md#helpersjs](shared-validator.md#helpersjs) |
| 43 | `../../shared/js/libs/validator/validator` | Validator core | [shared-validator.md#validatorjs](shared-validator.md#validatorjs) |
| 44 | `../../shared/js/libs/validator/phone` | Phone validator | [shared-validator.md#phonejs](shared-validator.md#phonejs) |
| 45 | `../../shared/js/libs/validator/zipcode` | ZIP validator | [shared-validator.md#zipcodejs](shared-validator.md#zipcodejs) |
| 46 | `../../shared/js/libs/validator/companyid` | Company ID validator | [shared-validator.md#companyidjs](shared-validator.md#companyidjs) |
| 47 | `../../shared/js/libs/validator/required` | Required validator | [shared-validator.md#requiredjs](shared-validator.md#requiredjs) |
| 48 | `../../shared/js/libs/phoneInput` | Phone input UI | [shared-libs.md#phoneinputjs](shared-libs.md#phoneinputjs) |
| 49 | `../../shared/js/libs/cofidis` | Cofidis calculator | [shared-libs.md#cofidisjs](shared-libs.md#cofidisjs) |
| 50 | `../../shared/js/libs/adminBar` | Admin bar UX | [shared-libs.md#adminbarjs](shared-libs.md#adminbarjs) |
| 51 | `../../shared/js/libs/csrf` | CSRF token | [shared-libs.md#csrfjs](shared-libs.md#csrfjs) |
| 52 | `../../shared/js/libs/csrfLink` | CSRF link handler | [shared-libs.md#csrflinkjs](shared-libs.md#csrflinkjs) |
| 53 | `../../shared/js/libs/xyDiscounts` | X+Y discounts | [shared-libs.md#xydiscountsjs](shared-libs.md#xydiscountsjs) |
| 54 | `./modules/focus-visible` | `:focus-visible` Safari shim | [template-modules.md#focus-visiblejs](template-modules.md#focus-visiblejs) |
| 55 | `./modules/focusManagement` | Focus trap & first-focus helpers | [template-modules.md#focusmanagementjs](template-modules.md#focusmanagementjs) |
| 56 | `./modules/layout` | Scroll-offset CSS variable | [template-modules.md#layoutjs](template-modules.md#layoutjs) |
| 57 | `./modules/screenReader` | ARIA-live announcer | [template-modules.md#screenreaderjs](template-modules.md#screenreaderjs) |
| 58 | `./modules/validator` | Validator template-layer wiring | [template-modules.md#validatorjs](template-modules.md#validatorjs) |
| 59 | `./modules/quantity` | Quantity input | [template-modules.md#quantityjs](template-modules.md#quantityjs) |
| 60 | `./modules/jquery.cloud-zoom.1.0.2` | Vendored cloud-zoom plugin | not in scope |
| 61 | `./modules/cloudZoomInit` | Cloud-zoom init | [template-modules.md#cloudzoominitjs](template-modules.md#cloudzoominitjs) |
| 62 | `./modules/tabsAccordion` | Tabs accordion | [template-modules.md#tabsaccordionjs](template-modules.md#tabsaccordionjs) |
| 63 | `./modules/tabsResponsive` | Tabs responsive | [template-modules.md#tabsresponsivejs](template-modules.md#tabsresponsivejs) |
| 64 | `./modules/topNavigationMenu` | Top navigation menu | [template-modules.md#topnavigationmenujs](template-modules.md#topnavigationmenujs) |
| 65 | `./modules/popups` | Popup windows | [template-modules.md#popupsjs](template-modules.md#popupsjs) |
| 66 | `./modules/dialogs` | `<dialog>`-element controller | [template-modules.md#dialogsjs](template-modules.md#dialogsjs) |
| 67 | `./modules/cookieBar` | Cookie bar | [template-modules.md#cookiebarjs](template-modules.md#cookiebarjs) |
| 68 | `./modules/tooltips` | New-style tooltip system (currently inert) | [template-modules.md#tooltipsjs](template-modules.md#tooltipsjs) |
| 69 | `./modules/globalFunctions` | Global utilities | [template-modules.md#globalfunctionsjs](template-modules.md#globalfunctionsjs) |
| 70 | `./modules/menu` | Main menu | [template-modules.md#menujs](template-modules.md#menujs) |
| 71 | `./modules/products` | Product detail / listing | [template-modules.md#productsjs](template-modules.md#productsjs) |
| 72 | `./modules/topProducts` | Top products block | [template-modules.md#topproductsjs](template-modules.md#topproductsjs) |
| 73 | `./modules/discussions-and-ratings` | Discussions & ratings | [template-modules.md#discussions-and-ratingsjs](template-modules.md#discussions-and-ratingsjs) |
| 74 | `./modules/search` | Search whisperer | [template-modules.md#searchjs](template-modules.md#searchjs) |
| 75 | `./modules/filters` | Product filters | [template-modules.md#filtersjs](template-modules.md#filtersjs) |
| 76 | `./modules/productSorting` | Product list sort | [template-modules.md#productsortingjs](template-modules.md#productsortingjs) |
| 77 | `./modules/productInnerSecondImage` | Hover-swap product image | [template-modules.md#productinnersecondimagejs](template-modules.md#productinnersecondimagejs) |
| 78 | `./modules/smartLabels` | Floating labels | [template-modules.md#smartlabelsjs](template-modules.md#smartlabelsjs) |
| 79 | `./modules/slider` | `Slider` class | [template-modules.md#sliderjs](template-modules.md#sliderjs) |
| 80 | `./modules/watchdog` | Product watchdog signup | [template-modules.md#watchdogjs](template-modules.md#watchdogjs) |
| 81 | `./modules/quantityDiscounts` | Quantity discounts table | [template-modules.md#quantitydiscountsjs](template-modules.md#quantitydiscountsjs) |
| 82 | `./modules/unveil` | Deprecated unveil shim | [template-modules.md#unveiljs](template-modules.md#unveiljs) |
| 83 | `./modules/skipLinks` | Skip links | [template-modules.md#skiplinksjs](template-modules.md#skiplinksjs) |
| 84 | `./modules/scroll` | `scrollToEl` | [template-modules.md#scrolljs](template-modules.md#scrolljs) |
| 85 | `./modules/animations` | `fadeIn` | [template-modules.md#animationsjs](template-modules.md#animationsjs) |
| 86 | `./modules/ratingStars` | Rating star input | [template-modules.md#ratingstarsjs](template-modules.md#ratingstarsjs) |
| 87 | `./modules/ratingSubpage` | Rating dialog / sub-page flow | [template-modules.md#ratingsubpagejs](template-modules.md#ratingsubpagejs) |

## [00/js/main-3g-header.js](../00/js/main-3g-header.js)

Header-only bundle, executes before the body to populate viewport-dependent CSS variables early.

| Import | Role | Documented |
| --- | --- | --- |
| `../../shared/js/libs/a11y` | Reduced-motion flag (`shoptet.a11y.reducedMotion`) | [shared-libs.md#a11yjs](shared-libs.md#a11yjs) |
| `../../shared/js/libs/layout` | Viewport detection (`getViewHeight`, `detectResolution`, `getScrollBarWidth`, Apple Pay check) | [shared-libs.md#layoutjs](shared-libs.md#layoutjs) |

## [00/js/checkout-3g.js](../00/js/checkout-3g.js)

Bundle for the checkout (order step 2) page.

| Import | Role | Documented |
| --- | --- | --- |
| `../../shared/js/modules/select2` | Vendored select2 plugin | [shared-utilities.md](shared-utilities.md) |
| `../../shared/js/libs/checkoutShared` | Payments, shipping, prices | [shared-libs.md#checkoutsharedjs](shared-libs.md#checkoutsharedjs) |
| `../../shared/js/libs/checkout` | Sticky sidebar, billing form | [shared-libs.md#checkoutjs](shared-libs.md#checkoutjs) |
| `../../shared/js/libs/applePaySdk` | Conditional loader for the Apple Pay JS SDK | [shared-libs.md#applepaysdkjs](shared-libs.md#applepaysdkjs) |

## [00/js/expressCheckout-3g.js](../00/js/expressCheckout-3g.js)

Bundle for the Shoptet Pay express checkout flow.

| Import | Role | Documented |
| --- | --- | --- |
| `../../shared/js/libs/expressCheckout` | Full express-checkout flow | [shared-libs.md#expresscheckoutjs](shared-libs.md#expresscheckoutjs) |
