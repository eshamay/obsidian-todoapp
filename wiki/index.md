# Index

## Concepts

- [[concepts/workspace-leaf-view-model]] — Obsidian's Workspace/WorkspaceLeaf/View object graph; the public `createLeafInParent`/`openFile`/`detach`/`View.containerEl` surface behind embedding a real editor outside the normal tab area.
- [[concepts/undocumented-api-fallback-pattern]] — feature-probe + isolate-the-undocumented-access + try/catch + clean-fallback pattern for any plugin technique that leans on non-public host API surface.

## Sources

- [[sources/obsidian-embedded-editor-research]] — first-party session findings on embedding Obsidian's live-preview `MarkdownView` inside a custom `Modal`, verified against two real open-source plugins.

## Comparisons

(none yet)

## Findings

(none yet)

## Examples

- [[examples/embed-markdownview-in-modal]] — worked code for the WorkspaceLeaf-embedding technique, as applied in `TaskNoteModal` (`plugin/src/main.tsx`).
