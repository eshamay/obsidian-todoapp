---
title: "Embedding Obsidian's live-preview editor inside a custom Modal — session findings"
type: source
status: active
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: obsidian-embedded-editor-research
    hash: 4659e92f37c8
    ingested: 2026-09-19
aliases: [embedded-editor-research, workspaceleaf-modal-technique]
tags: [source, obsidian-api, plugin-dev, workspaceleaf, markdownview, modal]
---

First-party research notes captured directly during a `todoapp-blocks-plugin` build session (2026-09-19), while replacing the task-note editor's plain `<textarea>` with a real embedded live-preview editor. Not a web source — this is the project's own verified findings, cross-checked against two real open-source plugins' code.

## Citation

Session notes, `todoapp-blocks-plugin` build session, 2026-09-19. Local file (`raw/session-obsidian-embedded-editor-research.md`). Verified against:
- `chrisgurney/obsidian-note-toolbar` — `src/Api/NtbModal.ts` — https://github.com/chrisgurney/obsidian-note-toolbar
- `likemuuxi/obsidian-modal-opener` — `src/modal.ts` — https://github.com/likemuuxi/obsidian-modal-opener
- `obsidianmd/obsidian-api` — `obsidian.d.ts`, installed version 1.13.0 — https://github.com/obsidianmd/obsidian-api

## Key claims

- Obsidian's `Modal` class has no built-in rich markdown editor widget — a plugin wanting a real live-preview (CM6-based) editing surface inside a popup must either open the file in a normal workspace tab, or embed Obsidian's own `MarkdownView` inside arbitrary DOM via a `WorkspaceLeaf` embedding technique.
- The embedding sequence — `Workspace.createLeafInParent(parent, index)` → hide the leaf's own tab wrapper → `WorkspaceLeaf.openFile(file, {state: {mode}})` → re-parent `View.containerEl` (not `leaf.containerEl`) into target DOM → `WorkspaceLeaf.detach()` on cleanup — is real, working code in two independent open-source plugins, not a theoretical technique.
- Of that sequence, only `leaf.containerEl` (the leaf's own tab-wrapper, distinct from `leaf.view.containerEl`) is genuinely undocumented — absent from `WorkspaceLeaf`'s and `WorkspaceItem`'s public class body in `obsidian.d.ts`. Everything else (`createLeafInParent`, `openFile`, `detach`, `View.containerEl`) is public, typed, versioned API.
- `MarkdownViewModeType` has exactly two values, `'source'` and `'preview'` — there is no separate "live preview" mode flag; live preview is `'source'` mode rendered under the user's global editor setting.
- `TextFileView.requestSave` (debounced ~2s autosave) plus `onUnloadFile`/leaf-detach flushing pending saves means an embedded `MarkdownView` autosaves like any normal Obsidian tab — no manual Save button is needed.
- `.metadata-container` (Properties/frontmatter panel) and `.view-header` (native file title bar) are real, addressable CSS classes Obsidian's core renders, confirmed via a shipped plugin's own `styles.css` — not part of any typed/versioned contract, but a stable-enough DOM hook to hide redundant native UI chrome inside an embed.
- Because the risk surface is small but real (one property, two CSS class names), the correct pattern is a feature probe (`typeof workspace.createLeafInParent !== "function"`) plus a try/catch around the whole mount sequence that falls back to a plain-textarea editor on any failure, rather than crashing.
- `eslint-plugin-obsidianmd`'s `no-static-styles-assignment` rule flags direct `element.style.display = ...` — use `element.addClass(...)` / CSS classes instead, even for a one-off hide.

## Open items

- Whether this embedding technique behaves correctly on Obsidian mobile (touch keyboard, toolbar overlap) was not verified this session. See [[open-knowledge-gaps#gap-001]].
