# Open Knowledge Gaps

## Index

| ID | Status | Summary |
|---|---|---|
| gap-001 | open | Does the WorkspaceLeaf/MarkdownView embedding technique work correctly on Obsidian mobile (touch keyboard, toolbar overlap)? |

---

## gap-001: Embedded-editor technique on Obsidian mobile

**Status:** open

Drafted by: agent on 2026-09-19.

[[sources/obsidian-embedded-editor-research]] verified the `WorkspaceLeaf`/`MarkdownView` embedding technique ([[concepts/workspace-leaf-view-model]], [[examples/embed-markdownview-in-modal]]) on desktop only. This project's `manifest.json` declares `isDesktopOnly: false`, so mobile support is claimed, but the embedded-editor path (and its fallback) has not been exercised on Obsidian mobile — touch keyboard interaction and toolbar overlap with a leaf embedded inside a `Modal` are unverified.

---

## Closed gaps

(none yet)
