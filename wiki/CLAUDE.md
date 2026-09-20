# wiki/ — sticki-wiki schema

Research base for the `todoapp-blocks-plugin` project: Obsidian plugin API/dev-tooling, and task-tracking-app/data-model prior art, to support building a custom Obsidian todo plugin. Bootstrapped 2026-09-19.

## Config

```yaml
wiki:
  domain: >
    Obsidian plugin API and developer tooling, and task-tracking-app /
    task-data-model research, in service of building a custom Obsidian
    todo-list plugin (fork of kaiso12/todoapp).
  scenario_template: research
  expected_volume: small   # <50 sources
  source_types: [markdown, url]
  page_taxonomy:
    - concepts/
    - sources/
    - comparisons/
    - findings/
    - examples/
  retrieval_tiers:
    tier_threshold_sources: 50
    staleness_days: 120   # shorter than a typical medium-volume wiki — API/tooling docs move faster than general methodology research
  ingest:
    dedup: by-hash-then-by-id
    related_pages_per_ingest: [5, 15]
  graph_weights:
    # slotted, not enforced in v1
    concept_link: 1.0
    source_citation: 1.0
```

## Bounded write authority

| File | Who may write | Notes |
|---|---|---|
| `irrefutable-facts.md` | User only (via `sticki-facts`) | Never auto-promoted |
| `open-knowledge-gaps.md` | Agent may draft (open); user closes (via `sticki-gaps`) | |
| `purpose.md` | User only | Scope/audience/non-goals |
| `todo.md` | Agent end-to-end (via `sticki-todo`) | Operational, no confirmation gate on close |
| `index.md`, `log.md` | Agent, on every ingest | |
| Content pages (`concepts/`, `sources/`, `comparisons/`, `findings/`, `examples/`) | Agent, via `sticki-ingest` / page-writer | |

## Page taxonomy notes

- **concepts/** — Obsidian API/plugin-dev concepts (e.g. Workspace/Leaf/View model, editor embedding, frontmatter-as-data) and task-data-modeling concepts (status modeling, entity schemas).
- **sources/** — reference documents and external research sources: official Obsidian docs, community plugin source, task-app API docs, prior-art pulled from other wikis.
- **comparisons/** — task-tracking-app comparisons (data models, API surfaces, Obsidian-integration options).
- **findings/** — cross-cutting synthesis / recommendations relevant to this plugin's own design.
- **examples/** — concrete Obsidian API usage code examples/snippets pulled from docs or community plugins.

No `decisions/`, `methodologies/`, `people/`, or `outputs/` — this wiki is internal research feeding the plugin's code, not the plugin's own decision log (that lives in the project's own docs/commits) and not a methodology-comparison wiki.

## The `.sticki/` change surface

`.sticki/state.json` tracks `{schema_version, revision, head_sha, updated}`. `.sticki/events.jsonl` is an append-only ingest/change log. Both git-tracked (parent repo `todoapp-blocks-plugin` is already a git repo).
