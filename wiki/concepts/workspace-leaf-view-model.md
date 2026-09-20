---
title: "Obsidian's Workspace / WorkspaceLeaf / View model"
type: concept
status: draft
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: obsidian-embedded-editor-research
    hash: 4659e92f37c8
    ingested: 2026-09-19
  - id: web-obsidian-plugin-api-overview
    hash: 85e25066b12f
    ingested: 2026-09-19
aliases: [workspaceleaf, workspace-model, obsidian-view-model]
tags: [concept, obsidian-api, workspace, workspaceleaf, markdownview]
---

Obsidian's UI is built from a small object graph: a `Workspace` owns a tree of splits/tab-groups; each tab is a `WorkspaceLeaf`; each leaf hosts exactly one `View` (e.g. `MarkdownView` for a note, other view types for other panes). The `View`'s own DOM lives at `View.containerEl` — a public, typed property. The `WorkspaceLeaf` itself also has a DOM wrapper (its tab chrome), but that wrapper property is **not** part of the public API surface.

This split matters for anything that wants to *reuse* Obsidian's real editing surface outside the normal workspace tab area (e.g. inside a `Modal`): a plugin can create a leaf detached from the visible layout via `Workspace.createLeafInParent(parent, index)` (public, `@since 0.9.11`), open a file into it via `WorkspaceLeaf.openFile(file, openState)` (public), then move only the `View`'s DOM (`leaf.view.containerEl`) — not the leaf's own wrapper — into wherever it's needed. Cleanup is `WorkspaceLeaf.detach()` (public), which also flushes any pending autosave.

`MarkdownViewModeType` has exactly two values: `'source'` and `'preview'`. "Live Preview" (Obsidian's default in-source rendering mode, as opposed to raw markdown source or the read-only rendered view) is not a third API-level mode — it's `'source'` mode rendered according to the user's global editor setting. There is no per-embed override to force live-preview specifically; you get whichever the user's global setting produces under `'source'` mode.

See [[sources/obsidian-embedded-editor-research]] for the concrete embedding sequence and [[examples/embed-markdownview-in-modal]] for the worked code, and [[undocumented-api-fallback-pattern]] for how to handle the one non-public property this technique still depends on (`leaf.containerEl`, distinct from `leaf.view.containerEl`).

**Official corroboration:** [[sources/web-obsidian-plugin-api-overview]] confirms `Workspace.getLeaf()`, `.createLeafBySplit()`, `.getLeavesOfType()`, and the `.rootSplit`/`.leftSplit`/`.rightSplit` container properties are all documented, public API (`docs.obsidian.md/Reference/TypeScript+API/Workspace`) — the embedding technique's use of `createLeafInParent`/`rootSplit` sits alongside this same official surface, not entirely outside it; only `leaf.containerEl` itself remains undocumented. `workspace.on()` also supports `'active-leaf-change'`, `'file-open'`, `'layout-change'` events, relevant if this technique needs to react to focus changes while an embed is open.
