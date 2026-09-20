> First-party research notes from a `todoapp-blocks-plugin` build session (2026-09-19), captured while fixing the task-note editor in `plugin/src/main.tsx` (`TaskNoteModal`) to show a full live-preview markdown experience instead of a plain textarea.

# Embedding Obsidian's real live-preview editor inside a custom Modal

## Problem

Obsidian's `Modal` class gives no built-in rich markdown editor widget. A plugin that wants a real live-preview (CM6-based) editing surface inside a popup — not a plain `<textarea>`, not a read-only `MarkdownRenderer.render()` preview — has to either (a) open the file in a normal workspace tab, or (b) embed Obsidian's own `MarkdownView` inside arbitrary DOM via internal APIs. This wiki entry covers (b), the technique actually used.

## Verified technique

Confirmed against two real, MIT-licensed open-source Obsidian plugins that do exactly this:

1. **`chrisgurney/obsidian-note-toolbar`** — `src/Api/NtbModal.ts` (`NtbModal.onOpen`/`onClose`). Comment in source: "adapted from https://github.com/likemuuxi/obsidian-modal-opener (MIT license)." Sequence:
   ```ts
   this.leaf = this.ntb.app.workspace.createLeafInParent(this.ntb.app.workspace.rootSplit, 0);
   if (this.leaf) (this.leaf.containerEl as HTMLElement).hide();
   await this.leaf.openFile(this.content);
   this.contentEl.appendChild(this.leaf.view.containerEl);
   // onClose:
   this.leaf?.detach();
   ```
2. **`likemuuxi/obsidian-modal-opener`** — `src/modal.ts` (`ModalWindow`). Adds:
   - Forces edit vs. preview mode via `openFile(file, { state: { mode } })` where `mode` is `'source' | 'preview'` — Obsidian's `MarkdownViewModeType` has only these two values. "Live Preview" itself is not a separate mode flag; it's how `'source'` mode renders under the user's global editor setting (Source mode vs. Live Preview vs. Reading are really "source, with CM6 decorations on/off" plus "preview," not three independent API modes).
   - Tracks `prevActiveLeaf = app.workspace.getMostRecentLeaf()` before creating the embedded leaf and restores it via `app.workspace.setActiveLeaf(...)` on close, so the embed doesn't permanently steal "current file" focus from things like Backlinks/Outline panes.
   - Registers a `Scope` so normal editor keyboard commands still work while focus is inside the modal.
   - `styles.css` shows the CSS hooks: `.view-header` (title/breadcrumb bar) and `.metadata-container` (Obsidian's Properties/frontmatter panel) — both toggleable with plain CSS, e.g. `body:not(.show-metadata) ... .metadata-container { display: none }`.

## Public vs. undocumented API surface

Most of the sequence is real, typed, `@public` API (per `obsidian.d.ts` from `obsidianmd/obsidian-api`, installed version 1.13.0 in this project):

- `Workspace.createLeafInParent(parent: WorkspaceSplit, index: number): WorkspaceLeaf` — public, `@since 0.9.11`.
- `WorkspaceLeaf.openFile(file: TFile, openState?: OpenViewState): Promise<void>` — public.
- `WorkspaceLeaf.detach(): void` — public, documented cleanup.
- `View.containerEl: HTMLElement` — public. This is what actually gets moved into the modal (`leaf.view.containerEl`), **not** `leaf.containerEl`.
- `TextFileView.requestSave` ("debounced save in 2 seconds from now") and `onUnloadFile` (flushes pending save on teardown) — public, documented. This is what justifies dropping a manual Save button: `MarkdownView` autosaves the same way any normal Obsidian tab does, and `leaf.detach()` flushes it.

Only one property is genuinely undocumented: **`leaf.containerEl`** (the leaf's own tab-wrapper element, distinct from `leaf.view.containerEl`) — it does not appear anywhere in `WorkspaceLeaf`'s or `WorkspaceItem`'s public class body in `obsidian.d.ts`. It's needed only to hide the leaf's own tab chrome so it never flashes as a real tab before its view's DOM is re-parented into the modal. Isolating this one access behind a small typed helper (`(leaf as unknown as { containerEl?: HTMLElement }).containerEl`) keeps the non-public surface auditable to a single line.

Two CSS class names are the only other non-guaranteed surface: `.metadata-container` (Properties panel) and `.view-header` (native file title bar) — both are real, addressable DOM classes Obsidian's core renders, confirmed via the `obsidian-modal-opener` plugin's own shipped `styles.css`, but not part of any typed/versioned contract.

## Fallback pattern

Because the risk surface (`leaf.containerEl`, the two CSS class names) is undocumented, wrap the whole mount sequence in a feature probe + try/catch that degrades to a plain-textarea editor on any failure, rather than crashing the modal:

- Probe: `typeof workspace.createLeafInParent !== "function" || !workspace.rootSplit` → skip straight to fallback.
- try/catch around: leaf creation → hiding the leaf's own wrapper → `openFile` → asserting `leaf.view instanceof MarkdownView` → re-parenting `leaf.view.containerEl` into the modal. Any throw: `leaf?.detach()`, remove any partially-appended DOM, fall back.

This was applied directly in `TaskNoteModal.tryOpenEmbeddedEditor()` (`plugin/src/main.tsx`), with `renderFallbackTextarea()` as the degrade path (today's pre-existing plain-`<textarea>` behavior, unchanged).

## Project environment notes (as of this session)

- `obsidian` npm package pinned `"latest"` in `package.json`, resolves to **1.13.0** typings (`package-lock.json`).
- `manifest.json` declares `minAppVersion: "1.5.0"`.
- No `@codemirror/*` packages are actual runtime dependencies of this plugin — only `preact` (^10.26.9) is a real runtime dep; CodeMirror only appears transitively as a peer-dep of a nested `obsidian` copy pulled in by a lint devDependency (`eslint-plugin-obsidianmd`), irrelevant to the bundled `main.js`.
- `tsconfig.json`: `strict: true`, `noUncheckedIndexedAccess: true`, `skipLibCheck: true`. All the public-API calls above type-checked cleanly against 1.13.0 typings with zero `any`/`@ts-ignore` beyond the one contained cast for `leaf.containerEl`.
- eslint (via `eslint-plugin-obsidianmd`) flags direct `element.style.display = ...` assignment (`obsidianmd/no-static-styles-assignment`) — use `element.addClass(...)`/CSS classes instead of inline styles, even for a one-off hide.

## Open question

Whether this embedding technique behaves correctly on Obsidian mobile (touch keyboard, toolbar overlap) — not verified in this session; `manifest.json`'s `isDesktopOnly: false` means mobile support is claimed but the embedded-editor fallback path has not been exercised there.
