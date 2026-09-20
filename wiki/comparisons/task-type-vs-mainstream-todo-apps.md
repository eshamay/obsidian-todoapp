---
title: "todoapp-blocks-plugin's Task type vs. six mainstream todo apps/plugins"
type: comparison
status: draft
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: web-mainstream-todo-app-data-models
    hash: a0a17111f4c8
    ingested: 2026-09-19
aliases: [task-vs-mainstream-apps]
tags: [comparison, task-data-model, entity-model]
---

This project's current `Task` type (`plugin/src/main.tsx`: `title`, `projectId`, `due`, `priority: 1|2|3|4`, `completed: boolean`, `notePath`) against six mainstream tools' documented task data models.

| Concept | This project | Todoist | Things | Obsidian Tasks | Dataview | Kanban |
|---|---|---|---|---|---|---|
| Scheduled vs. hard-deadline split | no (`due` only) | yes (`due` vs. `deadline`) | yes (`when` vs. `deadline`) | 5 date types (created/scheduled/start/due/done) | inherits Tasks' emoji dates | no |
| Priority scale | numeric 1-4 | numeric 1-4 (inverted vs. UI) | none documented | 6-level emoji (5 explicit + implicit default) | inherits Tasks' | none |
| Tags/labels | no | `labels[]` | `tags[]` | inline `#tag` | `tags` field | no |
| Subtask/dependency model | no | `parent_id` (tree) | `checklist-items` (inline, not true subtasks) | `id`/`dependsOn` (general dependency graph) | `children`/`parent` (generic list nesting) | none (lane position only) |
| Recurrence | no | natural-language string + `is_recurring` | not documented | 🔁 natural-language rule | inherits Tasks' | no |
| Time estimation | no | `duration`/`duration_unit` | not documented | not documented | not documented | no |
| Frontmatter/page inheritance | no (flat object) | n/a (API, not markdown) | n/a | no | **yes** — tasks inherit all page frontmatter | n/a |
| Status granularity | 1 boolean (`completed`) | 1 boolean | 2 booleans (`completed`/`canceled`) | checkbox + 🏁 on-completion behavior | 3-way (`checked`/`completed`/`fullyCompleted`) | checkbox + lane position |

## What this surfaces

- **Two independently-arrived-at products (Todoist, Things) both split "scheduled" from "hard deadline"** — the strongest, most-corroborated gap in this project's current model, which has only one `due` field doing both jobs.
- **Tagging/labeling is universal** across every tool surveyed except this project and Kanban (which uses lane position instead) — `Task` has no analogous field at all, only `projectId`.
- **Dataview's frontmatter-inheritance model** is structurally the most different idea here — not a field this project is missing, but a different *shape* (tasks-as-page-extensions vs. tasks-as-flat-objects) worth being aware of if this project ever wants richer per-task metadata without bloating the `Task` type itself.
- **Completed vs. cancelled** (Things' two-boolean split) echoes the same `done`/`cancelled` terminal-state distinction already flagged on [[findings/task-status-schema]] from the entity-model pull — now corroborated by an actual consumer app, not just the other project's own audit.

Not a decision — a comparison surfacing candidates for this project's own entity-model expansion workstream, alongside [[comparisons/task-type-vs-item-entity]].
