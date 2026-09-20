# Log

## [2026-09-19] bootstrap | todoapp-blocks-plugin research wiki

- Domain: Obsidian plugin API/dev-tooling research + task-tracking-app/data-model research, in service of the `todoapp-blocks-plugin` Obsidian todo plugin.
- Scenario template: research.
- Expected volume: small (<50 sources).
- Source types: markdown, url.
- Folders created: `concepts/`, `sources/`, `comparisons/`, `findings/`, `examples/`, `raw/`. Explicitly skipped: `decisions/`, `methodologies/`, `people/`, `outputs/` (user declined outputs/ — purely internal research feeding plugin code, no external deliverables planned).
- Retrieval tier config: `tier_threshold_sources: 50`, `staleness_days: 120` (shorter than a typical medium-volume wiki, since API/tooling docs move faster than general methodology research).
- Bounded-write-authority table emitted to `CLAUDE.md`.
- Purpose template applied: Research scenario, scoped to Obsidian-API + task-data-model research feeding this plugin's UI/UX and entity-model work; explicitly not a general PKM-methodology wiki (that's the sibling `work-organizer` wiki) and not this plugin's own decision log.

## [2026-09-19] ingest | Embedding Obsidian's live-preview editor inside a custom Modal — session findings
- source: [[sources/obsidian-embedded-editor-research]]
- raw: `raw/session-obsidian-embedded-editor-research.md` (sha256: 4659e92f37c8)
- pages touched: [[index]], [[open-knowledge-gaps]]
- new pages: [[sources/obsidian-embedded-editor-research]], [[concepts/workspace-leaf-view-model]], [[concepts/undocumented-api-fallback-pattern]], [[examples/embed-markdownview-in-modal]]
- contradictions raised: 0
- fact conflicts: 0
- gaps drafted: 1 (gap-001: embedded-editor technique unverified on Obsidian mobile)
- rationale: First ingest into a brand-new wiki, so this source establishes the initial page-type/wikilink pattern rather than merging into existing pages. Captured this session's own hands-on Obsidian-API research (the WorkspaceLeaf/MarkdownView embedding technique used to fix `TaskNoteModal`) as durable prior art before it would otherwise only live in chat history — split into a foundational API-model concept, a reusable risk-management pattern concept, and a worked-code example, per the wiki's concepts/examples taxonomy.

## [2026-09-19] ingest | Local-first, markdown-native personal knowledge app architectures (pulled from work-organizer wiki)
- source: [[sources/research-local-markdown-apps]]
- raw: `raw/work-organizer--research-local-markdown-apps.md` (sha256: 5c42382169b0)
- pages touched: [[index]]
- new pages: [[sources/research-local-markdown-apps]], [[concepts/obsidian-extension-mechanisms]]
- contradictions raised: 0
- fact conflicts: 0
- gaps drafted: 0
- rationale: Pulled from the sibling work-organizer wiki (per user request) for its Obsidian-ecosystem content only — the original's own build-architecture recommendation (standalone backend over Obsidian plugin) was for a different project and is explicitly not adopted here. Extracted the reusable comparison (Plugin API vs. Local REST API vs. Dataview vs. Bases) into a new concept page distinct from this wiki's own [[concepts/workspace-leaf-view-model]].

## [2026-09-19] ingest | Local backend vs. Obsidian plugin — architecture decision (pulled from work-organizer wiki)
- source: [[sources/local-backend-vs-obsidian-plugin]]
- raw: `raw/work-organizer--local-backend-vs-obsidian-plugin.md` (sha256: 6faa3913fe18)
- pages touched: [[index]], [[concepts/obsidian-extension-mechanisms]]
- new pages: [[sources/local-backend-vs-obsidian-plugin]]
- contradictions raised: 0
- fact conflicts: 0
- gaps drafted: 0
- rationale: Corroborates [[concepts/obsidian-extension-mechanisms]]'s Plugin-API-for-full-custom-UI claim from a different angle (a real project's own decision reasoning) — merged as an additional source on that concept page rather than a disconnected new one, since the actual reusable content overlaps with [[sources/research-local-markdown-apps]].

## [2026-09-19] ingest | Task status: flat-enum vs. state-machine vs. status-plus-flags (pulled from work-organizer wiki)
- source: [[sources/research-status-state-machine]]
- raw: `raw/work-organizer--research-status-state-machine.md` (sha256: cf778f274a36)
- pages touched: [[index]]
- new pages: [[sources/research-status-state-machine]], [[concepts/task-status-modeling]]
- contradictions raised: 0
- fact conflicts: 0
- gaps drafted: 0
- rationale: First of the status-research thread pulled from work-organizer; establishes the shared [[concepts/task-status-modeling]] page that subsequent status-research sources in this thread will accumulate onto, rather than each creating a disconnected concept page. Directly relevant prior art since `todoapp-blocks-plugin`'s own Task type is currently a flat `completed: boolean`.

## [2026-09-19] ingest | How Jira, Linear, GitHub, Trello model task status (pulled from work-organizer wiki)
- source: [[sources/research-status-mainstream-tools]]
- raw: `raw/work-organizer--research-status-mainstream-tools.md` (sha256: 25bdefb8e427)
- pages touched: [[index]], [[concepts/task-status-modeling]]
- new pages: [[sources/research-status-mainstream-tools]]
- contradictions raised: 0
- fact conflicts: 0
- gaps drafted: 0
- rationale: Second of the status-research thread; merged onto [[concepts/task-status-modeling]] as corroborating real-world precedent rather than a disconnected page, per the pattern established on ingest seq 4.

## [2026-09-19] ingest | Extensible status taxonomies: frozen category + open sub-status (pulled from work-organizer wiki)
- source: [[sources/research-status-extensible-taxonomies]]
- raw: `raw/work-organizer--research-status-extensible-taxonomies.md` (sha256: eee62f1f71de)
- pages touched: [[index]], [[concepts/task-status-modeling]]
- new pages: [[sources/research-status-extensible-taxonomies]]
- contradictions raised: 0
- fact conflicts: 0
- gaps drafted: 0
- rationale: Third of the status-research thread; adds the extensibility answer to [[concepts/task-status-modeling]] — how to add new status values later without a breaking migration of this project's own plain-JSON data files.

## [2026-09-19] ingest | Recording status-transition history in markdown-native storage (pulled from work-organizer wiki)
- source: [[sources/research-status-markdown-history]]
- raw: `raw/work-organizer--research-status-markdown-history.md` (sha256: 14f3c7685324)
- pages touched: [[index]], [[concepts/task-status-modeling]]
- new pages: [[sources/research-status-markdown-history]]
- contradictions raised: 0
- fact conflicts: 0
- gaps drafted: 0
- rationale: Fourth of the status-research thread; adds the "how are transitions recorded" answer to [[concepts/task-status-modeling]], with a caveat that this project's vault may not be a git repo (unlike the wiki's own parent repo), so the git-backstop layer needs re-evaluation if adopted.

## [2026-09-19] ingest | Capstone: a concrete seven-field task-status schema (pulled from work-organizer wiki)
- source: [[sources/research-status-data-model]]
- raw: `raw/work-organizer--research-status-data-model.md` (sha256: 1ed4557d2396)
- pages touched: [[index]], [[concepts/task-status-modeling]]
- new pages: [[sources/research-status-data-model]], [[findings/task-status-schema]]
- contradictions raised: 0
- fact conflicts: 0
- gaps drafted: 0
- rationale: Capstone of the status-research thread; created this wiki's first `findings/` page for the concrete field-level schema, the single most directly reusable artifact of the whole pull, explicitly marked as external prior art not yet adopted by this project.

## [2026-09-19] ingest | Status data-model recommendation (pulled from work-organizer wiki, findings/)
- source: (merged into [[findings/task-status-schema]] — no new source page created)
- raw: `raw/work-organizer--status-data-model-recommendation.md` (sha256: 06873b78382a)
- pages touched: [[findings/task-status-schema]]
- new pages: (none)
- contradictions raised: 0
- fact conflicts: 0
- gaps drafted: 0
- rationale: This pull is the original wiki's own `findings/` write-up of the exact same schema already captured via [[sources/research-status-data-model]] (seq 8) — near-total content overlap. Rather than create a duplicate page, added it as a corroborating source citation on [[findings/task-status-schema]] and pulled forward its one genuinely new refinement (terminal `done`/`cancelled` distinction) not already recorded.

## [2026-09-19] ingest | A unified todo/followup/blocker Item entity model (pulled from work-organizer wiki)
- source: [[sources/entity-data-model]]
- raw: `raw/work-organizer--entity-data-model.md` (sha256: 8d63b7494a3c)
- pages touched: [[index]], [[findings/task-status-schema]]
- new pages: [[sources/entity-data-model]], [[comparisons/task-type-vs-item-entity]]
- contradictions raised: 0
- fact conflicts: 0
- gaps drafted: 0
- rationale: Last of the 9 work-organizer pulls, and the most directly applicable to this project's entity-model workstream — genuinely new content (not overlapping the status-research thread beyond `status`/`status_category`), so it earned its own source page and this wiki's first `comparisons/` page, field-by-field against the project's current `Task` type. Confirms [[concepts/task-status-modeling]]'s "blocked is a relation" principle with `Item`'s concrete `blocked_by[]` array field.

## [2026-09-19] ingest | Official Obsidian Plugin API overview (docs.obsidian.md)
- source: [[sources/web-obsidian-plugin-api-overview]]
- raw: `raw/web-obsidian-plugin-api-overview.md` (sha256: 85e25066b12f)
- pages touched: [[index]], [[concepts/workspace-leaf-view-model]]
- new pages: [[sources/web-obsidian-plugin-api-overview]], [[concepts/obsidian-vault-file-io]], [[concepts/obsidian-ui-building-blocks]]
- contradictions raised: 0
- fact conflicts: 0
- gaps drafted: 0
- rationale: First of the web-research fan-out (official docs.obsidian.md). Corroborates [[concepts/workspace-leaf-view-model]]'s embedding technique against the officially documented Workspace surface. Surfaces one concrete finding for this project's own code: `TaskNoteModal`/`TodoStore` uses raw `vault.adapter` I/O instead of the higher-level `Vault` methods (`read`/`process`) — recorded on [[concepts/obsidian-vault-file-io]], not acted on. Also covers Commands/PluginSettingTab, relevant to this project's currently-stubbed `settings.ts`.

## [2026-09-19] ingest | Obsidian plugin developer tooling and community-release process (web research)
- source: [[sources/web-obsidian-dev-tooling-and-release-process]]
- raw: `raw/web-obsidian-dev-tooling-and-release-process.md` (sha256: 2ee40b4c74b6)
- pages touched: [[index]], [[concepts/obsidian-vault-file-io]]
- new pages: [[sources/web-obsidian-dev-tooling-and-release-process]], [[concepts/obsidian-plugin-guidelines-checklist]]
- contradictions raised: 0
- fact conflicts: 0
- gaps drafted: 0
- rationale: Second of the web-research fan-out. Confirms this project's own `version-bump.mjs`/`esbuild.config.mjs` (inherited via the upstream fork) already match the canonical sample-plugin scaffold. Surfaces a second concrete finding for this project's own code — `FileManager.processFrontMatter()` is the documented API for frontmatter edits, vs. this project's hand-rolled template/regex approach in `makeNoteFileContent()`/`stripTodoAppNoteMeta()` — merged onto [[concepts/obsidian-vault-file-io]] alongside the prior `vault.adapter` finding. Community-submission-process content noted as not applicable to this project (no PR/listing planned).

## [2026-09-19] ingest | Mainstream consumer todo-app and Obsidian task-plugin data models (web research)
- source: [[sources/web-mainstream-todo-app-data-models]]
- raw: `raw/web-mainstream-todo-app-data-models.md` (sha256: a0a17111f4c8)
- pages touched: [[index]], [[findings/task-status-schema]]
- new pages: [[sources/web-mainstream-todo-app-data-models]], [[comparisons/task-type-vs-mainstream-todo-apps]]
- contradictions raised: 0
- fact conflicts: 0
- gaps drafted: 0
- rationale: Third of the web-research fan-out, and this wiki's second `comparisons/` page — fills the "consumer todo app" gap the work-organizer pull didn't cover (that pull only had project-management tools and one custom entity model). Surfaces the scheduled-vs-deadline split (corroborated independently by Todoist and Things) and universal tagging as the starkest gaps in this project's current `Task` type. Cross-linked the `completed`/`cancelled` two-boolean split (Things) onto [[findings/task-status-schema]] as independent corroboration of that schema's own terminal-state refinement.

## [2026-09-19] ingest | CodeMirror 6 architecture and Obsidian's editor-extension API (web research)
- source: [[sources/web-codemirror6-and-obsidian-editor-extensions]]
- raw: `raw/web-codemirror6-and-obsidian-editor-extensions.md` (sha256: ec051c66a590)
- pages touched: [[index]], [[concepts/undocumented-api-fallback-pattern]]
- new pages: [[sources/web-codemirror6-and-obsidian-editor-extensions]], [[concepts/codemirror6-and-editor-extensions]]
- contradictions raised: 0
- fact conflicts: 0
- gaps drafted: 0
- rationale: Fourth and last of the web-research fan-out. Documents Obsidian's public `registerEditorExtension` API as a fully-supported alternative path for editor customization, distinct from the undocumented-internals `WorkspaceLeaf` embedding technique this project's own `TaskNoteModal` already uses — cross-linked onto [[concepts/undocumented-api-fallback-pattern]] to make the risk-profile contrast explicit. Not currently used by this project; relevant if the UI/UX workstream wants in-editor task-metadata rendering later. This completes the planned bootstrap + ingest + web-research-fan-out work (14 total ingests).
