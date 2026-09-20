---
title: "CodeMirror 6 architecture and Obsidian's editor-extension API (web research)"
type: source
status: active
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: web-codemirror6-and-obsidian-editor-extensions
    hash: ec051c66a590
    ingested: 2026-09-19
aliases: [codemirror6, registerEditorExtension]
tags: [source, obsidian-api, codemirror6, editor-extensions, official-docs]
---

Web research on CodeMirror 6 (CM6) — the engine behind Obsidian's live-preview Source-mode editor — and Obsidian's own public API for registering CM6 extensions.

## Citation

codemirror.net/docs/{guide,ref}; docs.obsidian.md/Plugins/Editor/Editor+extensions; docs.obsidian.md/Reference/TypeScript+API/Plugin/registerEditorExtension; three community-plugin GitHub sources (obsidian-completr, obsidian-tasks, obsidian-banners), each cited to an exact commit + line range. Full list in `raw/web-codemirror6-and-obsidian-editor-extensions.md`.

## Key claims

- CM6 splits into an immutable `EditorState` (pure data, transactions produce new values) and an imperative `EditorView` (DOM sync, browser event handling) — a deliberate departure from CM5's monolithic editor object.
- All CM6 features compose as `Extension` values (arbitrarily nestable, deduplicated, ordered via `Prec`) passed into `EditorState.create({extensions: [...]})`.
- **`Decoration.replace`** is the specific mechanism behind Obsidian's live-preview syntax hiding (e.g. hiding `**` while showing bold text) — hides/replaces a document range, optionally with a widget. `Decoration.mark` styles in place without hiding; `Decoration.widget` inserts DOM at a position without consuming text.
- `StateField` carries custom immutable state updated in lockstep with transactions (reducer-style: `create()`/`update(value, tr)`); `StateEffect` is the signal type for pushing explicit changes into a field via `tr.effects`.
- `ViewPlugin.fromClass` provides an imperative, DOM-owning component reacting to view updates (scroll, cursor, viewport) that a pure `StateField` can't observe — CM6's own guidance is that view plugins should stay "shallow views over the data kept in the editor state," not hold their own state.
- **`Plugin.registerEditorExtension(extension)` is a documented, public Obsidian API** — the supported integration point for adding custom CM6 `ViewPlugin`/`StateField` extensions to the live-preview editor, called from `onload()`. Dynamic reconfiguration: mutate the extension array at runtime, then call `Workspace.updateOptions()`.
- Three real community plugins (`obsidian-completr`, `obsidian-tasks`, `obsidian-banners`) all use `registerEditorExtension` alone — confirming this class of editor customization (custom decorations, widgets, reactive state in the live-preview editor) needs **no undocumented API or CM6-internals access**, unlike the `WorkspaceLeaf` embedding technique this project already uses (see [[concepts/undocumented-api-fallback-pattern]]).

## Relevance to this project

If this project's UI/UX workstream ever wants richer in-editor rendering of task syntax (e.g. custom decorations for due dates, priority markers, or inline task metadata directly in a note's live-preview body, rather than only in the plugin's own custom task-list view), `registerEditorExtension` is the fully-public, no-fallback-needed path — a meaningfully different risk profile than the embedded-modal technique. Not currently used anywhere in this project's code.
