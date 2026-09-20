---
title: "CodeMirror 6 architecture and Obsidian's registerEditorExtension"
type: concept
status: draft
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: web-codemirror6-and-obsidian-editor-extensions
    hash: ec051c66a590
    ingested: 2026-09-19
aliases: [codemirror6-architecture, registerEditorExtension-api, cm6-decorations]
tags: [concept, obsidian-api, codemirror6, editor-extensions]
---

Obsidian's live-preview Source-mode editor is CodeMirror 6 (CM6): an immutable `EditorState` (document + selection + custom fields, updated only via transactions) paired with an imperative `EditorView` (DOM rendering, browser event handling). All editor behavior — syntax highlighting, keymaps, decorations, custom state — composes as `Extension` values.

**The mechanism behind live-preview rendering:** `Decoration.replace` hides a markdown syntax marker (e.g. `**`) and optionally draws a widget in its place — this is literally how Obsidian shows bold text as bold while hiding the asterisks in Source/live-preview mode. `Decoration.mark` styles a range without hiding anything; `Decoration.widget` inserts new DOM without consuming document text.

**Custom plugin state:** `StateField` (reducer-style: `create()` + `update(value, transaction)`) carries state that updates deterministically alongside the document; `StateEffect` is how a plugin pushes an explicit, non-document-derived change into a field (via `tr.effects`). `ViewPlugin.fromClass` is the escape hatch for imperative DOM components reacting to things a pure `StateField` can't see (scroll, viewport) — CM6's own guidance: keep view plugins as thin views over state, not their own source of truth.

**Obsidian's public integration point:** `Plugin.registerEditorExtension(extension)`, called from `onload()`. This is fully documented, public API — no undocumented internals needed, unlike [[undocumented-api-fallback-pattern|the WorkspaceLeaf embedding technique]] this project already uses for the task-note modal. Three real community plugins (obsidian-completr, obsidian-tasks, obsidian-banners) confirm `registerEditorExtension` alone is sufficient for custom decorations/widgets/reactive state in the live-preview editor.

**Not currently used by this project.** Relevant if the UI/UX workstream ever wants in-editor rendering of task metadata (due dates, priority, etc.) directly inside a note's live-preview body, rather than only in the plugin's own custom task-list view.

Source: [[sources/web-codemirror6-and-obsidian-editor-extensions]].
