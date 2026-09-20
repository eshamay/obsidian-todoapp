---
title: "Reusable task-status schema (7 fields) — external prior art"
type: finding
status: draft
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: work-organizer-research-status-data-model
    hash: 1ed4557d2396
    ingested: 2026-09-19
  - id: work-organizer-status-data-model-recommendation
    hash: 06873b78382a
    ingested: 2026-09-19
aliases: [task-status-schema, status-data-model-recommendation]
tags: [finding, task-data-model, status-modeling, schema, entity-model]
---

A concrete, field-level task-status schema assembled by the sibling `work-organizer` wiki's own four-part research thread ([[concepts/task-status-modeling]]), pulled in here as prior art for `todoapp-blocks-plugin`'s own entity-model work — **not yet adopted by this project**, this page records the external recommendation, not a decision made here.

> [!NOTE]
> This page merges two source pulls that cover the identical schema: the `sources/` capstone memo ([[sources/research-status-data-model]]) and the original wiki's own `findings/` recommendation page it was assembled into (`work-organizer/wiki/findings/status-data-model-recommendation.md`, pulled as `raw/work-organizer--status-data-model-recommendation.md`). No separate page was created for the second pull — it corroborates rather than adds new schema content, beyond the terminal-state note below.

One refinement the original project's own audit applied on top of this schema: the primary `status` enum's terminal state should be `done` **and** `cancelled` as two distinct values (not one generic "closed"), matching how a `Project`-level entity in that project already distinguished the two — worth carrying forward if this project's own status enum gets a terminal state at all.

See [[comparisons/task-type-vs-item-entity]] for how this schema's fields line up against this project's current `Task` type field-by-field. The `done`/`cancelled` terminal-state split is independently corroborated by [[comparisons/task-type-vs-mainstream-todo-apps]] — Things (Cultured Code) documents the identical two-boolean split (`completed`/`canceled`) in a real consumer product, not just the other project's own internal audit.

## The schema

| Field | Purpose |
|---|---|
| `status` | Primary-phase enum (e.g. backlog / in-progress / done / cancelled). |
| `status_category` | Small, frozen top-level category underneath `status`, per [[sources/research-status-extensible-taxonomies]]'s extensibility mechanism. |
| `blocked` | Independent boolean flag — orthogonal to `status`, per [[concepts/task-status-modeling]]. |
| `blocked_reason` | Free text/enum explaining why, when `blocked` is true. |
| `needs_followup` | Independent boolean flag, orthogonal to both `status` and `blocked`. |
| `followup_reason` | Free text/enum explaining why, when `needs_followup` is true. |
| `status_changed` | Denormalized timestamp of the last `status` transition (not full history — see below). |

The four daily-use "buckets" in the original project's own framing (in-flight / needs-followup / blocked / backlog) are modeled as *queries* over these fields, never as separately stored values — avoiding the flat-enum trap [[concepts/task-status-modeling]] opens with.

## What this schema deliberately defers

The original wiki's own resolution explicitly separated the **field schema** (locked) from **full transition-history recording** (event-note-per-transition + git backstop, per [[sources/research-status-markdown-history]]) — the latter was knowingly deferred in that project, not solved by the denormalized `status_changed` field alone. Carrying the same distinction forward here: adopting this schema's fields doesn't imply adopting event-note history-recording in the same step.

## Relevance to `todoapp-blocks-plugin`

Current `Task` type (`plugin/src/main.tsx`) has `completed: boolean` — a flat model with no in-progress/blocked/followup distinction at all. This schema is the most directly actionable piece of prior art in this wiki's status-research pull if the entity-model expansion (one of this project's two stated workstreams — see the project's own `CLAUDE.md`) wants a richer status concept than a checkbox. Not a decision — a candidate.
