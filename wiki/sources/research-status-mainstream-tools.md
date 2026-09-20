---
title: "How Jira, Linear, GitHub, Trello model task status (pulled from work-organizer wiki)"
type: source
status: active
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: work-organizer-research-status-mainstream-tools
    hash: 25bdefb8e427
    ingested: 2026-09-19
aliases: [status-mainstream-tools-survey]
tags: [source, task-data-model, status-modeling, comparison, external-prior-art]
---

Prior-art research pulled from the sibling `work-organizer` wiki (dated 2026-09-18), surveying how Jira, Linear, GitHub Issues/Projects, and Trello model task/work-item status.

## Citation

Pulled from `work-organizer/wiki/raw/research-status-mainstream-tools.md`, dated 2026-09-18. See `raw/work-organizer--research-status-mainstream-tools.md` for the full original text and its cited Atlassian/Linear/GitHub/Trello docs.

## Key claims

- Jira, Linear, and GitHub Issues/Projects all keep "blocked" off the primary status field — modeled instead as an issue link/relation/label, not a status value.
- Trello's lack of a native dependency mechanism is the corroborating negative case: without a relation mechanism, "blocked" has nowhere to live except an ad hoc label, reinforcing that a dedicated relation (not a status enum value) is the right shape.
- Jira and Linear both preserve a small, fixed set of status *categories* (e.g. To Do / In Progress / Done) underneath user-editable, per-project custom status names — a two-layer model, not a single flat enum.

## Relevance to this project

Corroborates [[concepts/task-status-modeling]]'s status-plus-orthogonal-flags recommendation with real-world tool precedent across four mainstream tools, and adds the fixed-category-underneath-editable-names pattern as a specific implementation detail worth carrying forward if this project's entity model adds a richer status field.
