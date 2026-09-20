---
title: "Task status: flat-enum vs. state-machine vs. status-plus-flags (pulled from work-organizer wiki)"
type: source
status: active
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: work-organizer-research-status-state-machine
    hash: cf778f274a36
    ingested: 2026-09-19
aliases: [status-state-machine-eval]
tags: [source, task-data-model, status-modeling, external-prior-art]
---

Prior-art research pulled from the sibling `work-organizer` wiki (dated 2026-09-18), evaluating how to model task/work-item status: a single flat enum, a full state machine, or a hybrid.

## Citation

Pulled from `work-organizer/wiki/raw/research-status-state-machine.md`, dated 2026-09-18. See `raw/work-organizer--research-status-state-machine.md` for the full original text.

## Key claims

- A single flat `status:` enum can't represent an item that is simultaneously in-flight *and* blocked, and gives no record of when/why it moved between values.
- Recommends a **status-plus-orthogonal-flags hybrid**: a small primary-phase `status:` enum plus independent boolean flags (e.g. `blocked`, `needs_followup`) valid at any phase — grounded in statechart "orthogonal region" theory and corroborated by Jira's production separation of `status`/`resolution`/`priority` into independent fields.

## Relevance to this project

`todoapp-blocks-plugin`'s task entity model (currently `completed: boolean` plus `priority`/`due` — see `plugin/src/main.tsx` `Task` type) is exactly a flat model today. This source's hybrid recommendation is directly relevant prior art if the entity-model expansion (per this project's own goals) adds a richer status concept. See [[concepts/task-status-modeling]].
