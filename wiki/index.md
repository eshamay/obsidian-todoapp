# Index

## Concepts

- [[concepts/workspace-leaf-view-model]] — Obsidian's Workspace/WorkspaceLeaf/View object graph; the public `createLeafInParent`/`openFile`/`detach`/`View.containerEl` surface behind embedding a real editor outside the normal tab area.
- [[concepts/undocumented-api-fallback-pattern]] — feature-probe + isolate-the-undocumented-access + try/catch + clean-fallback pattern for any plugin technique that leans on non-public host API surface.
- [[concepts/obsidian-extension-mechanisms]] — four ways to extend/query a vault (Plugin API, Local REST API, Dataview, Bases) and why Plugin API is the only one giving full custom UI control.
- [[concepts/task-status-modeling]] — status-plus-orthogonal-flags: one status enum + independent blocked/followup flags, vs. flat-enum or full-state-machine alternatives.
- [[concepts/obsidian-vault-file-io]] — Vault's high-level file methods vs. raw `adapter` access; flags that this project's own note-file I/O currently bypasses the higher-level API.
- [[concepts/obsidian-ui-building-blocks]] — Modal/Setting/Commands/PluginSettingTab, the core UI-construction API surface; notes this project's settings.ts is still an empty stub.
- [[concepts/obsidian-plugin-guidelines-checklist]] — official community-plugin code-quality guidelines (innerHTML avoidance, Editor/Vault.process/FileManager.processFrontMatter preferences, UI text case, hotkey conventions).
- [[concepts/codemirror6-and-editor-extensions]] — CM6's EditorState/EditorView split, Decorations/StateField/StateEffect/ViewPlugin, and Obsidian's public `registerEditorExtension` API — a fully-documented alternative to undocumented-internals techniques for editor customization.

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
- [[sources/web-obsidian-plugin-api-overview]] — official docs.obsidian.md coverage of Plugin lifecycle, Workspace/Vault/Editor, Modal/Setting, Commands, PluginSettingTab, mobile compatibility, and versioning.
- [[sources/web-obsidian-dev-tooling-and-release-process]] — obsidian-api typings, obsidian-sample-plugin scaffold (matches this project's own build files), esbuild pipeline, community submission process (not applicable to this project), and plugin guidelines.
- [[sources/web-mainstream-todo-app-data-models]] — Todoist API, Things URL scheme, Obsidian Tasks/Dataview/Kanban plugin data models; TickTick has no verifiable public schema.
- [[sources/web-codemirror6-and-obsidian-editor-extensions]] — CodeMirror 6 architecture and Obsidian's public `registerEditorExtension` API, with three community-plugin examples.

## Comparisons

- [[comparisons/task-type-vs-item-entity]] — this project's current `Task` type vs. `Item`'s field shape, field-by-field; surfaces blocking-as-relation and context-tagging as the starkest gaps.
- [[comparisons/task-type-vs-mainstream-todo-apps]] — `Task` vs. Todoist/Things/Tasks-plugin/Dataview/Kanban; surfaces scheduled-vs-deadline split and universal tagging as the starkest gaps.

## Findings

- [[findings/task-status-schema]] — the reusable 7-field task-status schema (`status`/`status_category`/`blocked`/`blocked_reason`/`needs_followup`/`followup_reason`/`status_changed`), external prior art not yet adopted by this project.

## Examples

- [[examples/embed-markdownview-in-modal]] — worked code for the WorkspaceLeaf-embedding technique, as applied in `TaskNoteModal` (`plugin/src/main.tsx`).
