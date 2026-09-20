---
title: "Mainstream consumer todo-app and Obsidian task-plugin data models (web research)"
type: source
status: active
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: web-mainstream-todo-app-data-models
    hash: a0a17111f4c8
    ingested: 2026-09-19
aliases: [todoist-api, things-url-scheme, obsidian-tasks-plugin, obsidian-dataview-tasks, obsidian-kanban-format]
tags: [source, task-data-model, entity-model, comparison, official-docs]
---

Web research filling the gap left by prior research (Jira/Linear/GitHub/Trello project-management tools, and one custom `Item` entity) — none of that covered consumer todo apps specifically. Surveys Todoist's REST API, TickTick's developer portal, Things' URL scheme, and the Obsidian Tasks/Dataview/Kanban community plugins.

## Citation

developer.todoist.com/api/v1; developer.ticktick.com (no verifiable schema — SPA with no server-rendered docs, confirmed via WebFetch, curl, and a headless-browser attempt); culturedcode.com/things/support; publish.obsidian.md/tasks; blacksmithgu.github.io/obsidian-dataview; github.com/mgmeyers/obsidian-kanban (read directly from TypeScript source when README fetches were incomplete). Full list in `raw/web-mainstream-todo-app-data-models.md`.

## Key claims

- **Todoist**: `due` (scheduled, recurring via natural-language string + `is_recurring`) and a **separate `deadline` object** (date-only "due at the latest," independent of `due`) are two distinct date concepts. Plus `duration`/`duration_unit` for time estimation, `labels` (multi-value tags distinct from `project_id`), and `parent_id` for subtasks.
- **TickTick**: no field-level schema could be verified — the entire developer portal is a client-rendered SPA with no server-rendered content, and this was confirmed (not assumed) via three independent fetch attempts.
- **Things**: separates `when` (scheduling) from `deadline` (hard due date) — same two-date-concept split as Todoist, independently arrived at. Also has `tags`, `checklist-items` (inline sub-items distinct from subtasks), `heading` (sub-grouping inside a project), and a three-state-like status (`completed` vs. `canceled` as separate booleans).
- **Obsidian Tasks plugin**: 6-level emoji priority scale (not numeric 1-4); 5 distinct date types (created/scheduled/start/due/done) plus cancelled; recurrence as a natural-language rule; and an explicit `id`/`dependsOn` field pair giving a general task-dependency graph, not just parent/child nesting.
- **Obsidian Dataview**: tasks are first-class queryable objects with `status`/`checked`/`completed`/`fullyCompleted` as *distinct* completion-granularity fields (not one boolean), and — most structurally significant — **tasks inherit all frontmatter fields from their containing page**, making tasks and pages one connected metadata graph rather than two disjoint models.
- **Obsidian Kanban plugin**: verified directly from TypeScript source (not just README) — lanes are literal `## <heading>` markdown, cards are literal `- [ ]`/`- [x]` checkbox lines under a lane, an `## Archive` lane is a soft-delete mechanism, and board settings are a JSON blob embedded in a `%% kanban:settings ... %%` comment — all state lives in-band in one markdown file.

## Relevance to this project

Every one of the six sources surfaces at least one structural concept `todoapp-blocks-plugin`'s current `Task` type doesn't have. See [[comparisons/task-type-vs-mainstream-todo-apps]] for the full field-by-field breakdown. The two independently-arrived-at "scheduled date vs. hard deadline" splits (Todoist, Things) are notable corroboration of each other from unrelated products.
