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
  - id: work-organizer-research-status-mainstream-tools
    hash: 25bdefb8e427
    ingested: 2026-09-19
  - id: work-organizer-research-status-extensible-taxonomies
    hash: eee62f1f71de
    ingested: 2026-09-19
  - id: work-organizer-research-status-markdown-history
    hash: 14f3c7685324
    ingested: 2026-09-19
  - id: work-organizer-research-status-data-model
    hash: 1ed4557d2396
    ingested: 2026-09-19
aliases: [status-modeling, orthogonal-flags-status]
tags: [concept, task-data-model, status-modeling, entity-model]
---

Status-plus-orthogonal-flags: one primary-phase `status:` enum (e.g. `backlog`/`in-progress`/`done`/`cancelled`) plus independent boolean/flag fields (`blocked`, `needs_followup`, etc.) that can be true at any phase — rather than either (a) a single flat enum that can't represent "in-progress AND blocked" simultaneously, or (b) a full state machine that's more machinery than the domain needs.

Grounded in statechart "orthogonal region" theory (independent concurrent state dimensions) and corroborated by mainstream tools keeping "blocked" off the primary status field: Jira/Linear/GitHub Issues all model blocking as a relation or label, not a status value (Trello's lack of any native dependency mechanism is the corroborating negative case), and Jira/Linear both preserve a small fixed status-*category* set underneath user-editable status names — a two-layer model, not a single flat enum.

Directly relevant prior art for `todoapp-blocks-plugin`'s own task entity model (currently a flat `completed: boolean` — `plugin/src/main.tsx` `Task` type), pulled in via the sibling `work-organizer` wiki's own status-modeling research thread rather than derived fresh here.

**Extensibility:** if a status enum is added, [[sources/research-status-extensible-taxonomies]] recommends a frozen top-level category layer (small, never-extended) over an open additive sub-status string, plus an `unknown` sentinel and free-form tags as a pressure valve — so new status values can be added later without a breaking migration of this project's plain-JSON data files (`.todoapp/<id>.json`).

**Transition history:** [[sources/research-status-markdown-history]] recommends layering three mechanisms — frontmatter for current state only, a linked event note per transition as the live queryable history, git commit history as the tamper-evident backstop. Note this project's vault is not necessarily a git repo (unlike this wiki's own parent repo), so the git-backstop layer may not carry over unchanged.

**Concrete schema:** [[findings/task-status-schema]] assembles all of the above into one field-level 7-field schema (`status`/`status_category`/`blocked`/`blocked_reason`/`needs_followup`/`followup_reason`/`status_changed`) — the single most directly reusable artifact of this research thread.

Sources: [[sources/research-status-state-machine]], [[sources/research-status-mainstream-tools]], [[sources/research-status-extensible-taxonomies]], [[sources/research-status-markdown-history]], [[sources/research-status-data-model]] (external prior art).
