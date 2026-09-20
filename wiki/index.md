# Index

## Concepts

- [[concepts/workspace-leaf-view-model]] — Obsidian's Workspace/WorkspaceLeaf/View object graph; the public `createLeafInParent`/`openFile`/`detach`/`View.containerEl` surface behind embedding a real editor outside the normal tab area.
- [[concepts/undocumented-api-fallback-pattern]] — feature-probe + isolate-the-undocumented-access + try/catch + clean-fallback pattern for any plugin technique that leans on non-public host API surface.
- [[concepts/obsidian-extension-mechanisms]] — four ways to extend/query a vault (Plugin API, Local REST API, Dataview, Bases) and why Plugin API is the only one giving full custom UI control.

## Sources

- [[sources/obsidian-embedded-editor-research]] — first-party session findings on embedding Obsidian's live-preview `MarkdownView` inside a custom `Modal`, verified against two real open-source plugins.
- [[sources/research-local-markdown-apps]] — Obsidian-ecosystem survey (Local REST API, Dataview, Bases) pulled from the work-organizer wiki.
- [[sources/local-backend-vs-obsidian-plugin]] — a different project's decision to build directly against the Plugin API (Phase 1) vs. a standalone backend (contingent Phase 2), pulled from the work-organizer wiki; corroborates Plugin API as the only full-custom-UI mechanism.

## Comparisons

(none yet)

## Findings

(none yet)

## Examples

- [[examples/embed-markdownview-in-modal]] — worked code for the WorkspaceLeaf-embedding technique, as applied in `TaskNoteModal` (`plugin/src/main.tsx`).
