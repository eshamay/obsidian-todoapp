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
