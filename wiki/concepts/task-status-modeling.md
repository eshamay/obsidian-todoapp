---
title: "Task/work-item status modeling: status-plus-orthogonal-flags"
type: concept
status: draft
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: work-organizer-research-status-state-machine
    hash: cf778f274a36
    ingested: 2026-09-19
aliases: [status-modeling, orthogonal-flags-status]
tags: [concept, task-data-model, status-modeling, entity-model]
---

Status-plus-orthogonal-flags: one primary-phase `status:` enum (e.g. `backlog`/`in-progress`/`done`/`cancelled`) plus independent boolean/flag fields (`blocked`, `needs_followup`, etc.) that can be true at any phase — rather than either (a) a single flat enum that can't represent "in-progress AND blocked" simultaneously, or (b) a full state machine that's more machinery than the domain needs.

Grounded in statechart "orthogonal region" theory (independent concurrent state dimensions) and corroborated by mainstream tools keeping "blocked" off the primary status field: Jira/Linear/GitHub Issues all model blocking as a relation or label, not a status value, and Jira/Linear both preserve a small fixed status-*category* set underneath user-editable status names.

Directly relevant prior art for `todoapp-blocks-plugin`'s own task entity model (currently a flat `completed: boolean` — `plugin/src/main.tsx` `Task` type), pulled in via the sibling `work-organizer` wiki's own status-modeling research thread rather than derived fresh here.

Source: [[sources/research-status-state-machine]] (external prior art). See also the capstone concrete schema at [[sources/research-status-data-model]] once ingested.
