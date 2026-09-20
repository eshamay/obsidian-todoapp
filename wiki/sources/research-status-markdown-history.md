---
title: "Recording status-transition history in markdown-native storage (pulled from work-organizer wiki)"
type: source
status: active
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: work-organizer-research-status-markdown-history
    hash: 14f3c7685324
    ingested: 2026-09-19
aliases: [status-markdown-history]
tags: [source, task-data-model, status-modeling, markdown-storage, external-prior-art]
---

Prior-art research pulled from the sibling `work-organizer` wiki (dated 2026-09-18), on how to record *when/why* a status transition happened in markdown-native (frontmatter-based) storage, as opposed to just the current state.

## Citation

Pulled from `work-organizer/wiki/raw/research-status-markdown-history.md`, dated 2026-09-18. See `raw/work-organizer--research-status-markdown-history.md` for the full original text.

## Key claims

- Recommends layering three mechanisms: frontmatter holds **current state only**; a **linked event note per transition** (its own small frontmatter: `type`/`item`/`date`/`from`/`to`/`reason`) is the live, queryable history; **git commit history** is the tamper-evident backstop, not the day-to-day query surface.
- This three-layer split keeps the primary note's frontmatter simple (fast to read/render) while still making "what changed and why" answerable without parsing git log for routine use.

## Relevance to this project

`todoapp-blocks-plugin` doesn't currently record any transition history for task changes (`plugin/src/main.tsx` `TodoStore.save()` just overwrites `.todoapp/<id>.json` wholesale) — this is directly relevant prior art if a future entity-model expansion wants a "what changed and when" view, though note that this project's vault is not necessarily a git repo (unlike this wiki's own parent repo), so the git-backstop layer may not be available the same way.
