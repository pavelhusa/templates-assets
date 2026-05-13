# Shared AJAX — `shared/js/libs/ajax/`

Two files split between the request side (`request.js`) and the response wrapper (`response.js`). Together they form `shoptet.ajax.*`.

For the `shoptet.*` exposure pattern, see [shoptet-namespace.md](shoptet-namespace.md).

---

## request.js

[Source](../shared/js/libs/ajax/request.js)

**Purpose:** `XMLHttpRequest` wrapper that returns a Promise resolving to an `AjaxResponse`, plus a structured success / failure / redirect / complete callback set.
**Module style:** IIFE on `shoptet`; ES module imports; TypeScript-style JSDoc.
**Exposes:** `shoptet.ajax.makeAjaxRequest`, plus the constants below.

| Function | Signature | Description |
| --- | --- | --- |
| `makeAjaxRequest(url, type, data, callbacks?, header?)` | `(url, type: 'GET'\|'POST', data, callbacks?, header?) => Promise<AjaxResponse>` | Send an async request and resolve with the parsed response. Callbacks: `{ success, failed, redirect, complete }`, all optional. |

Constants:

- `shoptet.ajax.requestTypes` = `{ get: 'GET', post: 'POST' }`.
- `shoptet.ajax.pendingClass` = `'ajax-pending-element'` (used to mark in-flight elements).

**Dependencies:** `shoptet.common.serializeData`, `shoptet.ajax.AjaxResponse`, `preserveEditorPreviewUrl` from `shared/js/libs/themeEditor/lib.js`.

---

## response.js

[Source](../shared/js/libs/ajax/response.js)

**Purpose:** The `AjaxResponse` constructor — parses JSON, routes callbacks, handles redirects, surfaces notifications via `window.showMessage`.
**Module style:** Constructor + prototype methods, IIFE-wrapped.
**Exposes:** `shoptet.ajax.AjaxResponse`; legacy `window.AjaxResponse` shim (deprecated, delegates to `shoptet.ajax.AjaxResponse`).

### Constructor

`new shoptet.ajax.AjaxResponse(result, customSettings?)` — parses `result` (JSON string or object). Response shape: `{ code, message, messages[], payload }`.

### Methods

| Method | Signature | Description |
| --- | --- | --- |
| `setCallback(callback, fn)` | `(name: 'success'\|'failed'\|'redirect'\|'complete', fn) => void` | Register a callback after construction. |
| `isFailed()` | `() => boolean` | True if response code is 500. |
| `isSuccess()` | `() => boolean` | True if response code is 200. |
| `isRedirected()` | `() => boolean` | True if payload has `returnUrl` and the code is 301 / 302 / 303. |
| `redirect()` | `() => void` | Navigate to `payload.returnUrl` if `isRedirected()`. |
| `getCode()` | `() => number` | Response code. |
| `getMessage()` | `() => string` | Primary message. |
| `getMessages()` | `() => string[]` | Messages array. |
| `getPayload()` | `() => unknown` | Full payload. |
| `getFromPayload(key)` | `(key: string) => unknown` | Value at `payload[key]`. |
| `showNotification()` | `() => void` | Display success / error toast via `window.showMessage`. |
| `processResult()` | `() => void` | Route to the appropriate callback (success / failed / redirect) and then `complete`. |

**Dependencies:** `shoptet.dev.deprecated`, `window.showMessage`.
