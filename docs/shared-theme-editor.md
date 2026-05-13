# Shared theme editor — `shared/js/libs/themeEditor/`

Two ES modules that run inside the Shoptet theme-editor preview iframe. The editor parent window communicates with the preview via `window.postMessage`; these scripts are the preview-side counterpart.

For the `shoptet.*` exposure pattern, see [shoptet-namespace.md](shoptet-namespace.md). (These files do not register on `shoptet` — they are pure ES modules.)

---

## lib.js

[Source](../shared/js/libs/themeEditor/lib.js)

**Purpose:** Editor preview-mode detection and URL parameter preservation.
**Module style:** ES module (`export` only).
**Exposes:** named exports only.

| Function | Signature | Description |
| --- | --- | --- |
| `detectEditorPreview()` | `() => boolean` | True if the page is loaded with the `editorPreview` URL parameter. |
| `preserveEditorPreviewUrl(url)` | `(url: string) => string` | Add the `editorPreview` query parameter to outgoing request URLs so they keep editor context. |
| `getNextUrl(urlObject, options)` | `(url: URL, options: { previewId?, deviceMode?: 'mobile'\|'desktop' }) => URL` | Construct the next URL preserving `editorPreview` and `isMobileDevice` parameters across navigation. |

**Dependencies:** `URL` API only.

---

## editorPreview.js

[Source](../shared/js/libs/themeEditor/editorPreview.js)

**Purpose:** Theme-editor preview iframe runtime — intercepts links, navigates within the editor, handles element inspection overlay, communicates with the parent editor window via `postMessage`.
**Module style:** ES module that runs at load time (no explicit exports).
**Exposes:** runs automatically; no public function-level API.

Behaviours:

- Intercepts clicks on same-origin links and sends `pageIsLoading` to the parent before navigation.
- Listens for messages from the parent editor: `reload`, `navigate`, `inspectConfig`, `inspectHover`.
- Emits messages to the parent: `pageLoaded`, `pageIsLoading`, `inspecting`, `inspectHover`.
- Implements an **inspect overlay** showing labels for editable elements based on `data-editorid` attributes.
- Special-cases Bootstrap carousels and tabs (forwards parent commands into the relevant plugin API).
- Detects mobile preview via a media query so the parent can simulate viewport sizes.

**Dependencies:** `detectEditorPreview`, `getNextUrl` from [`./lib.js`](../shared/js/libs/themeEditor/lib.js); optional Bootstrap carousel and tab plugins.
