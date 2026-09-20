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
