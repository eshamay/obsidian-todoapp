> Pulled from work-organizer wiki (`raw/research-status-data-model.md`), dated 2026-09-18 — research for a different project (work-organizer command-center); reused here as prior art.

# Status Data Model: The Recommended Schema (gap-005 synthesis)

**Audience:** command-center design — resolves the open design question in [[open-knowledge-gaps]] gap-005<br>
**Purpose:** one concrete `status:`-like schema for the four daily-work buckets, usable almost as-is when the entity/data model is designed next<br>
**Draws on:** four independent research fronts — [[sources/research-status-state-machine]], [[sources/research-status-mainstream-tools]], [[sources/research-status-extensible-taxonomies]], [[sources/research-status-markdown-history]]<br>
**Date:** 2026-09-18

## Summary

Four research fronts approached gap-005 from different angles — statechart/state-machine theory, how mainstream trackers actually ship, extensible-enum/API-schema guidance, and markdown-native history patterns — and converged on the same shape. That four-way convergence is the strongest-signal recommendation this wiki has produced on the status question: it is not one source's opinion, it is the answer four separate lines of inquiry independently landed on. The shape is a **status-plus-orthogonal-flags hybrid**: one small primary-phase field governed by a frozen category layer, plus independent boolean flags for the genuinely separate dimensions (blocked, needs-followup), plus a linked event-note per transition holding the queryable history — frontmatter carries current state only, never the log. This composes with the design already decided in [[findings/command-center-design-recommendation]] (GTD's list model for the four buckets, [[concepts/frontmatter-as-database]] for querying) rather than competing with it.

## The recommended schema

### Item frontmatter

Every work item carries this on its own note:

```yaml
---
status: in-flight        # primary phase — small documented open enum
status_category: open    # frozen top-level category (derivable from status)
blocked: false           # orthogonal flag — valid at ANY phase
blocked_reason:          # free text; only meaningful when blocked: true
needs_followup: false    # orthogonal flag — valid at ANY phase
followup_reason:         # free text; e.g. "waiting on infra review"
status_changed: 2026-09-18  # date the primary status last changed
---
```

Field-by-field:

| Field | Type | Example values | Role |
|---|---|---|---|
| `status` | string (documented open enum) | `in-flight`, `backlog`, `done`, `unknown` | Region A — the primary workflow phase. One value at a time. |
| `status_category` | string (frozen closed set) | `open`, `closed` (and `unknown` fallback) | The stable layer all views/rollups group by; never gains or loses members. |
| `blocked` | boolean | `true` / `false` | Region B — independent of phase. An item is `in-flight` AND `blocked: true` at once. |
| `blocked_reason` | string | `"waiting on infra review"` | Captures the *why* for the blocking dimension. |
| `needs_followup` | boolean | `true` / `false` | Region C — independent of phase and of blocked. |
| `followup_reason` | string | `"ping stakeholder Fri"` | The *why* for the follow-up dimension. |
| `status_changed` | date | `2026-09-18` | Cheap denormalized "when did phase last move" for at-a-glance views; the event notes are the full record. |

Why this shape and not the alternatives: a **flat enum** cannot hold two independent dimensions without exploding into fused values like `in-flight-blocked` for every combination (the state-machine report's central point, echoing statechart theory's "state explosion" — four independent switches need 8 parallel states versus 16 atomic ones). A **full transition-enforcing state machine** buys illegal-transition prevention that a personal tool does not need and adds machinery (transition tables, guards) not worth carrying (YAGNI). The **hybrid** keeps each field flat and queryable while representing compound states honestly — it is the practical encoding of what statechart theory calls *orthogonal regions* (an entity is in one state from each dimension simultaneously).

### How the four daily buckets map onto it

The four buckets are a **query over these fields, not a stored value** — which is what lets one item appear in more than one bucket at once (in-flight AND blocked):

| Daily bucket | Query condition |
|---|---|
| **in-flight** | `status == "in-flight"` (regardless of flags — a blocked-but-active item still shows here) |
| **needs-followup** | `needs_followup == true` |
| **blocked / delegate** | `blocked == true` |
| **backlog** | `status == "backlog"` |

An item that is `status: in-flight`, `blocked: true` surfaces in **both** the in-flight and blocked views — the overlapping-states requirement (a) satisfied directly. This is exactly how Jira/Linear/GitHub separate phase from blocking (mainstream-tools report): all three model "blocked" as a separate relation or flag, never as a competing value inside the status field, precisely so an issue can be "In Progress" and "blocked by X" simultaneously.

### The transition event-note (history)

Each status change writes one small note whose data lives entirely in frontmatter (so every field is first-class queryable — no Dataview inline-field or `FLATTEN` workaround):

```yaml
---
type: status-change
item: "[[Project X]]"
date: 2026-09-18
from: in-flight
to: blocked
field: blocked          # which axis moved: status | blocked | needs_followup
reason: waiting on infra review
---
```

The parent item's frontmatter holds only current state; this event note is the append-only history. Requirement (b) — a record of when/why status changed — satisfied. This is the markdown-history report's clear recommendation, and it mirrors how Jira/Linear/GitHub all treat status-change history as a first-class automatic log rather than something reconstructed from comments.

### How "what changed this week" gets queried

A Dataview query over the events folder, filtered by date, grouped by item:

```dataview
TABLE from, to, reason, date
FROM "events"
WHERE type = "status-change" AND date >= date(today) - dur(7 days)
SORT date DESC
```

Git commit history (one commit per change, per the project's existing convention) sits underneath as the tamper-evident backstop — recoverable via `git log -L`/`-S` with `--since`, but treated as audit-of-last-resort, not the day-to-day query surface.

## Where the four reports agree (the strong signal)

The convergence is the headline. Each report reached the recommendation from a different direction:

- **[[sources/research-status-state-machine|State-machine theory]]** → status + orthogonal boolean flags, *not* a flat enum and *not* a full state machine. Grounds it in statechart "orthogonal regions" and cites Jira's status/resolution/priority separation as the practitioner proof.
- **[[sources/research-status-mainstream-tools|Mainstream tools]]** → independently confirms the same split across Jira, Linear, and GitHub: a primary phase axis plus blocking modeled as a separate relation/flag, never fused into the status value; and status-change history is an automatic first-class log in all three.
- **[[sources/research-status-extensible-taxonomies|Extensible taxonomies]]** → the extensibility answer: a frozen top-level category layer over an open, additive sub-status string, an `unknown` fallback so consumers never choke on an unrecognized value, and tags as a pressure valve. Satisfies requirement (c) — taxonomy stays extensible without breaking existing data/views.
- **[[sources/research-status-markdown-history|Markdown history]]** → frontmatter holds current state only; a linked event-note per transition (its own frontmatter: item/date/from/to/reason) is the live queryable history via Dataview; git is the backstop audit trail.

Four fronts, one shape. The four requirements map onto the four reports' contributions cleanly: overlapping states ← orthogonal flags (state-machine + mainstream-tools); transition history ← event notes (markdown-history + mainstream-tools); extensibility ← frozen-category-over-open-enum (extensible-taxonomies).

### Requirement (c) — extensibility — in practice

`status` is a **documented open enum**: its known values live in the schema as examples, not as an enforced constraint. New phases are added additively; existing values are never renamed or deleted (Azure/Protobuf/AIP-126 guidance). Every view branches on `status_category` (the frozen layer) with an explicit `unknown` fallback, so a future `status` value degrades into its category rather than vanishing from a view. Free-form tags remain available for the genuinely unanticipated, and a recurring tag can be promoted into a documented sub-status once it earns it.

## The one open implementation choice — and the call

The reports fully resolve everything except **where the transition history physically lives**: an in-file append-only changelog block inside the item note, versus a separate event-note-per-transition.

- **In-file changelog block** — lower file sprawl, everything in one place, diffs cleanly. But a changelog *bullet* is only Dataview-queryable if it carries bracketed inline fields (`- [changed:: 2026-09-18] [to:: blocked] ...`) and is reached with `FLATTEN file.lists` — verbose, and second-class compared with frontmatter.
- **Event-note-per-transition** — one extra file and one extra write per change, but the transition's fields live in frontmatter, so they are first-class queryable with no inline-field or `FLATTEN` gymnastics, and "what changed this week" is a plain `FROM "events"` query.

**Decision: go with the event-note-per-transition.** The whole system is built on [[concepts/frontmatter-as-database]] — the query surface is the product. Paying one extra file per transition to keep history first-class queryable is the right trade for a tool whose central value is faceted queries over frontmatter. The in-file changelog is the documented fallback if file sprawl ever becomes a real problem in daily use.

## How this composes with existing wiki findings

This model is a refinement *under* the decided design, not a competing proposal:

- [[findings/command-center-design-recommendation]] already recommends **GTD's list model** for the four buckets and says the four buckets "should be the primary status facet (`status:` frontmatter), rendered as views, not folders." This synthesis fills in *how* that `status:` facet is actually structured so that an item can sit in two buckets at once — which a single flat `status:` value could not do. The buckets stay views-not-folders; they just query `status` + the two flags instead of one field.
- [[concepts/frontmatter-as-database]] is the mechanism the whole model rides on: item state is frontmatter properties, the four-bucket views are faceted Dataview/Bases queries, and the event notes are just more frontmatter-bearing pages indexed the same way. Nothing here introduces a store outside the frontmatter-as-database pattern.
- The Obsidian-plugin-first decision ([[findings/command-center-design-recommendation]] Phase 1) is well served: everything above is plain markdown + YAML + Dataview, no database, and carries forward unchanged if a Phase 2 standalone backend materializes external items into the same files (each synced item just gets the same `status`/flags/event-note treatment).

## Open questions

gap-005 stays **open pending the user's own review**. These are implementation details deliberately *not* closed here:

1. **Exact category names.** This synthesis uses `open`/`closed` for `status_category`; the extensible-taxonomies report also floated `blocked` as a top-level category. Whether blocking is a category or purely a flag (this synthesis makes it purely a flag) is the user's call to confirm.
2. **Whether `done` is a `status` value or its own category.** Modeled here as a terminal `status` value under `status_category: closed`; could instead be a distinct category if closed items need finer sub-states (e.g. `done` vs. `cancelled`).
3. **Event-note folder location and naming.** Whether transition notes live in a dedicated `events/`-style folder (as written above) and the file-naming convention (e.g. `events/2026-09-18-project-x-blocked.md`) are unspecified.
4. **Whether `status_changed` is worth denormalizing** onto the item, or whether the event notes alone suffice — a minor redundancy-vs-convenience call.
