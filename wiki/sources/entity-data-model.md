---
title: "A unified todo/followup/blocker Item entity model (pulled from work-organizer wiki)"
type: source
status: active
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: work-organizer-entity-data-model
    hash: 8d63b7494a3c
    ingested: 2026-09-19
aliases: [entity-data-model-pull]
tags: [source, task-data-model, entity-model, external-prior-art]
---

Prior-art research pulled from the sibling `work-organizer` wiki (dated 2026-09-18, locked/decided status there) — a 4-entity domain model (`Item`/`Person`/`Project`/`Area`) with literal YAML frontmatter, designed for that project's own v1 Obsidian plugin.

## Citation

Pulled from `work-organizer/wiki/decisions/entity-data-model.md`, dated 2026-09-18. See `raw/work-organizer--entity-data-model.md` for the full original text (13 entities total documented there; only `Item` is directly relevant here — `Person`/`Project`/`Area` model that project's stakeholder/project-tracking needs, out of scope for a todo plugin).

## Key claims

- The `Item` entity is a **unified** todo/followup/blocker/work-item type — one entity type covers what a pure todo app might otherwise split into separate task/reminder/blocker concepts.
- `Item` fields (per the original, `Person`/`Project`/`Area` fields omitted as out of scope here): `status`, `status_category` (matches [[findings/task-status-schema]]'s schema — same research thread, same project), `blocked` + `blocked_reasons[]` + `blocked_by[]` (an array of blocking relations, not a single reason string — see [[concepts/task-status-modeling]]'s "blocked is a relation, not a status" claim, concretized here as a real link-array field), `needs_followup` + `followup_direction`, `context[]` (GTD-style @context tags), `due_date`, `priority`, `project`/`area` links, `gus_ref` (Salesforce-specific, not relevant here).

## Relevance to this project

`todoapp-blocks-plugin`'s current `Task` type (`plugin/src/main.tsx`) is a flat model: `title`, `projectId`, `due`, `priority` (1-4), `completed: boolean`, plus a note-file link. Compared to `Item`, the biggest structural gaps are: no `blocked`/`blocked_by` relation, no `needs_followup` flag, no `context[]` tags, and `completed` is a boolean rather than a status-plus-category pair. This is the most concrete, closest-to-directly-applicable prior art in the whole pull for this project's own "expand the entity model" workstream — `Item` was designed for a superset of a todo app's needs (a full command-center), so not every field applies, but the todo-relevant subset (`status`/`blocked`/`blocked_by`/`needs_followup`/`context`) is a strong starting point.
