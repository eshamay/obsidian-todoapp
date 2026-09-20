---
title: "Extensible status taxonomies: frozen category + open sub-status (pulled from work-organizer wiki)"
type: source
status: active
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: work-organizer-research-status-extensible-taxonomies
    hash: eee62f1f71de
    ingested: 2026-09-19
aliases: [status-extensible-taxonomies]
tags: [source, task-data-model, status-modeling, api-design, external-prior-art]
---

Prior-art research pulled from the sibling `work-organizer` wiki (dated 2026-09-18), on how to make a status field extensible for future values without breaking existing consumers — drawing on Azure/Google AIP-126/Zalando/Protobuf API-design guidance.

## Citation

Pulled from `work-organizer/wiki/raw/research-status-extensible-taxonomies.md`, dated 2026-09-18. See `raw/work-organizer--research-status-extensible-taxonomies.md` for the full original text and cited API-design guidance (Microsoft/Azure API Guidelines, Google AIP-126, Zalando REST guidelines, Protobuf best-practices, Stripe's "online migrations" blog).

## Key claims

- Recommends a **frozen top-level status-category layer** (a small, stable, never-extended set) over an **open, additive sub-status string** (new values can be added without a breaking migration), plus an `unknown` default sentinel and free-form tags as a pressure valve for genuinely unanticipated values.
- This is the specific extensibility mechanism API-design guidance converges on for enum-like fields that must evolve over a product's lifetime without breaking existing readers/writers.

## Relevance to this project

Directly answers the "how do we add new status values later without breaking the plugin's own data files" question that any status-field addition to [[concepts/task-status-modeling]] would eventually face — relevant since this project's data model is plain JSON on disk (`.todoapp/<id>.json`, see `plugin/src/main.tsx` `TodoStore`), not a server API, but the same forward-compatibility concern applies to any persisted enum.
