---
title: "Capstone: a concrete seven-field task-status schema (pulled from work-organizer wiki)"
type: source
status: active
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: work-organizer-research-status-data-model
    hash: 1ed4557d2396
    ingested: 2026-09-19
aliases: [status-data-model-capstone]
tags: [source, task-data-model, status-modeling, schema, external-prior-art]
---

Prior-art research pulled from the sibling `work-organizer` wiki (dated 2026-09-18) — the capstone synthesis of that wiki's four-part status-modeling research thread ([[sources/research-status-state-machine]], [[sources/research-status-mainstream-tools]], [[sources/research-status-extensible-taxonomies]], [[sources/research-status-markdown-history]]) into one concrete field-level schema.

## Citation

Pulled from `work-organizer/wiki/raw/research-status-data-model.md`, dated 2026-09-18. See `raw/work-organizer--research-status-data-model.md` for the full original text.

## Key claims

- Assembles a concrete seven-field schema: `status`, `status_category`, `blocked`, `blocked_reason`, `needs_followup`, `followup_reason`, `status_changed`.
- The four daily-use "buckets" (in-flight / needs-followup / blocked / backlog, in that other project's own framing) are modeled as *queries* over these fields, not as stored values themselves.
- Event-note-per-transition was chosen over an in-file changelog for history, per [[sources/research-status-markdown-history]].

## Relevance to this project

The concrete field-level shape — see [[findings/task-status-schema]] — is the single most directly reusable artifact from the entire status-research pull, for whatever richer status concept `todoapp-blocks-plugin`'s own entity-model work eventually adopts (currently just `completed: boolean`).
