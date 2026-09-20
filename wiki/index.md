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
- [[sources/research-status-extensible-taxonomies]] — frozen status-category layer + open additive sub-status + `unknown` fallback + tags, for forward-compatible status enums; drawn from API-design guidance (Azure/AIP-126/Zalando/Protobuf).
- [[sources/research-status-markdown-history]] — frontmatter-current-state + linked-event-note-history + git-backstop, three-layer approach to recording status transitions in markdown-native storage.
- [[sources/research-status-data-model]] — capstone synthesis of the four-part status-research thread into one concrete seven-field schema; pulled from the work-organizer wiki.
- [[sources/entity-data-model]] — a unified todo/followup/blocker `Item` entity model, pulled from the work-organizer wiki.

## Comparisons

- [[comparisons/task-type-vs-item-entity]] — this project's current `Task` type vs. `Item`'s field shape, field-by-field; surfaces blocking-as-relation and context-tagging as the starkest gaps.

## Findings

- [[findings/task-status-schema]] — the reusable 7-field task-status schema (`status`/`status_category`/`blocked`/`blocked_reason`/`needs_followup`/`followup_reason`/`status_changed`), external prior art not yet adopted by this project.

## Examples

- [[examples/embed-markdownview-in-modal]] — worked code for the WorkspaceLeaf-embedding technique, as applied in `TaskNoteModal` (`plugin/src/main.tsx`).
