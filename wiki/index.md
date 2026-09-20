# Index

## Concepts

- [[concepts/workspace-leaf-view-model]] — Obsidian's Workspace/WorkspaceLeaf/View object graph; the public `createLeafInParent`/`openFile`/`detach`/`View.containerEl` surface behind embedding a real editor outside the normal tab area.
- [[concepts/undocumented-api-fallback-pattern]] — feature-probe + isolate-the-undocumented-access + try/catch + clean-fallback pattern for any plugin technique that leans on non-public host API surface.
- [[concepts/obsidian-extension-mechanisms]] — four ways to extend/query a vault (Plugin API, Local REST API, Dataview, Bases) and why Plugin API is the only one giving full custom UI control.
- [[concepts/task-status-modeling]] — status-plus-orthogonal-flags: one status enum + independent blocked/followup flags, vs. flat-enum or full-state-machine alternatives.

## Sources

- [[sources/obsidian-embedded-editor-research]] — first-party session findings on embedding Obsidian's live-preview `MarkdownView` inside a custom `Modal`, verified against two real open-source plugins.
- [[sources/research-local-markdown-apps]] — Obsidian-ecosystem survey (Local REST API, Dataview, Bases) pulled from the work-organizer wiki.
- [[sources/local-backend-vs-obsidian-plugin]] — a different project's decision to build directly against the Plugin API (Phase 1) vs. a standalone backend (contingent Phase 2), pulled from the work-organizer wiki; corroborates Plugin API as the only full-custom-UI mechanism.
- [[sources/research-status-state-machine]] — evaluates flat-enum vs. state-machine vs. status-plus-flags for task status; recommends the hybrid, pulled from the work-organizer wiki.
- [[sources/research-status-mainstream-tools]] — surveys Jira/Linear/GitHub/Trello status modeling; all three richer tools keep "blocked" off the status field, corroborating the hybrid.

## Comparisons

(none yet)

## Findings

(none yet)

## Examples

- [[examples/embed-markdownview-in-modal]] — worked code for the WorkspaceLeaf-embedding technique, as applied in `TaskNoteModal` (`plugin/src/main.tsx`).
