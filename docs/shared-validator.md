# Shared validator — `shared/js/libs/validator/`

The form-validation framework. The core (`validator.js`) is generic; each other file is a single-purpose validator that registers itself with the framework. The template layer in [00/js/modules/validator.js](../00/js/modules/validator.js) wires these into specific forms — see [template-modules.md § validator.js](template-modules.md#validatorjs).

For the `shoptet.*` exposure pattern, see [shoptet-namespace.md](shoptet-namespace.md).

---

## validator.js

[Source](../shared/js/libs/validator/validator.js)

**Purpose:** Core framework — field registration, error message display / removal, form-level submit gating.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.validator.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `initNewValidator(validator, element, event)` | `(validator, el, event: string) => void` | Attach a validator to an element on a specific event (`change`, `blur`, custom). |
| `formContainsInvalidFields(form)` | `(form: HTMLFormElement) => number` | Count invalid fields in a form. |
| `handleValidators(validators)` | `(validators: Record<string, ValidatorDef>) => void` | Register all validators from a map. |
| `getExistingMessage(elementWrapper)` | `(wrapper) => HTMLElement \| null` | Query existing error messages. |
| `removeErrorMessage(element, messageType, elementWrapper)` | `(el, type?, wrapper?) => void` | Clear an error message. |
| `addErrorMessage(element, messageType)` | `(el, type: string) => void` | Show an error with customisable text. |

Config:

- `shoptet.validator.invalidFieldClasses` — selector for invalid fields.
- `shoptet.validator.events` — list of DOM-ready event names the framework listens for.

**Dependencies:** `shoptet.scripts`, `shoptet.messages`.

---

## required.js

[Source](../shared/js/libs/validator/required.js)

**Purpose:** Validate that required fields have a value.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.validatorRequired.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `validateRequiredField(el)` | `(el: HTMLElement) => boolean` | True if the element has a value. |

Config:

- `shoptet.validatorRequired.messageType` = `'validatorRequired'`.
- `shoptet.validatorRequired.validators` = `{ requiredInputs: { elements, events, validator, fireEvent } }`.

Auto-binds to `.js-validate-required` on `change`, `blur`, `validatedFormSubmit`.

**Dependencies:** `shoptet.validator`, `shoptet.scripts`.

---

## phone.js

[Source](../shared/js/libs/validator/phone.js)

**Purpose:** Validate international phone numbers (format, regional rules, format conversion).
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.validatorPhone.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `validateNumber(el)` | `(el: HTMLInputElement) => void` | Validate phone, detect country code, call backend validate API. |

Config:

- `validateUrl` = `/action/ShoptetValidatePhone/index/`.
- `messageType` = `'validatorInvalidPhoneNumber'`.
- `messageTypeSuggestedRegion` = `'validatorInvalidPhoneNumberSuggestedRegion'`.
- `ajaxPending` — counter exposed for `validator.js` to wait on before form submit.

Auto-binds to `.js-validate-phone` on `change` and `ShoptetPhoneCodeChange`.

**Dependencies:** `shoptet.ajax`, `shoptet.validator`, `shoptet.phoneInput`, `shoptet.scripts`.

---

## zipcode.js

[Source](../shared/js/libs/validator/zipcode.js)

**Purpose:** Validate postal codes per country via a `data-pattern` regex.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.validatorZipCode.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `validateZipCode(el)` | `(el) => boolean` | Test value against `data-pattern`. |
| `updateZipValidPattern($el)` | `($el: jQuery) => void` | Update the pattern when the country select changes. |

Config: `messageType` = `'validatorZipCode'`.

Auto-binds to `.js-validate-zip-code` on `change`, `ShoptetBillZipPatternChange`, `ShoptetDeliveryZipPatternChange`.

**Dependencies:** jQuery, `shoptet.validator`, `shoptet.scripts`.

---

## companyid.js

[Source](../shared/js/libs/validator/companyid.js)

**Purpose:** Validate company / VAT IDs per country via a `data-pattern` regex.
**Module style:** IIFE on `shoptet`.
**Exposes:** `shoptet.validatorCompanyId.*`.

| Function | Signature | Description |
| --- | --- | --- |
| `validateCompanyId(el)` | `(el) => boolean` | Test value against `data-pattern`. |
| `updateCompanyIdValidPattern()` | `() => void` | Update the pattern from the selected country option. |

Config: `messageType` = `'validatorInvalidCompanyId'`; `messageTypeCustomized` includes an example.

Auto-binds to `.js-validate-company-id` on `ShoptetCompanyIdPatternChange` and `change`.

**Dependencies:** jQuery, `shoptet.validator`, `shoptet.scripts`, `shoptet.messages`.

---

## helpers.js

[Source](../shared/js/libs/validator/helpers.js)

**Purpose:** Toggle the `required` attribute on conditional field groups (e.g. registration form, alternate-shipping-address fields).
**Module style:** Global function on `window`.
**Exposes:** `window.toggleRequiredAttributes`.

| Function | Signature | Description |
| --- | --- | --- |
| `toggleRequiredAttributes($el, job, preserveNoJsValidation)` | `($el, job: 'add'\|'remove', preserveNoJsValidation?: boolean) => void` | Add or remove `required` attributes and validation suspension; stores / restores original values; toggles the `.js-validation-suspended` class; manages `autocomplete` toggling. |

**Dependencies:** jQuery, `shoptet.validator`.
