---
title: "todoapp-blocks-plugin's Task type vs. work-organizer's Item entity"
type: comparison
status: draft
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: work-organizer-entity-data-model
    hash: 8d63b7494a3c
    ingested: 2026-09-19
aliases: [task-vs-item-comparison]
tags: [comparison, task-data-model, entity-model]
---

This project's current `Task` type (`plugin/src/main.tsx`) against [[sources/entity-data-model]]'s `Item` entity — the todo-relevant subset only (Person/Project/Area fields from the original omitted as out of scope).

| Concern | `Task` (this project, current) | `Item` (external prior art) |
|---|---|---|
| Completion state | `completed: boolean` | `status` (enum) + `status_category` — see [[findings/task-status-schema]] |
| Blocking | none | `blocked: boolean` + `blocked_reasons[]` + `blocked_by[]` (relation array, not a string) |
| Followup | none | `needs_followup: boolean` + `followup_direction` |
| Contexts/tags | none | `context[]` (GTD-style @context tags) |
| Due date | `due?: string` | `due_date` — equivalent |
| Priority | `priority: 1\|2\|3\|4` | `priority` — equivalent |
| Grouping | `projectId: string` | `project`/`area` links |
| Notes/body | `notePath?: string` → real vault `.md` file | (not specified in the pulled excerpt) |

## Gaps this surfaces

The starkest gap is **blocking as a boolean-with-no-relation** vs. `Item`'s array-of-links (`blocked_by[]`) — [[concepts/task-status-modeling]] already flags "blocked should be a relation, not a status value" as mainstream-tool consensus (Jira/Linear/GitHub all model it as a relation/label), and `Item` is the first source in this wiki to show a *concrete* field shape for that relation rather than just the principle.

Second gap: no `context[]` equivalent at all — `Task` has no tagging/categorization mechanism beyond its single `projectId`.

Not a decision — a comparison surfacing candidates for this project's own entity-model expansion workstream, to be decided separately from this research wiki.
