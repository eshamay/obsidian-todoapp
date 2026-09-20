> Pulled from work-organizer wiki (`findings/status-data-model-recommendation.md`), dated 2026-09-18 — research for a different project (work-organizer command-center); reused here as prior art.

---
title: "Status data model — recommended schema (gap-005 capstone)"
type: finding
status: draft
created: 2026-09-18
updated: 2026-09-18
sources:
  - id: research-status-data-model
    hash: 43e6a9e4b40c
    ingested: 2026-09-18
  - id: research-status-state-machine
    hash: df84dfb0fa2a
    ingested: 2026-09-18
  - id: research-status-mainstream-tools
    hash: 293c4ed04703
    ingested: 2026-09-18
  - id: research-status-extensible-taxonomies
    hash: bfce91e87402
    ingested: 2026-09-18
  - id: research-status-markdown-history
    hash: 54fb2208fd09
    ingested: 2026-09-18
aliases: [status-data-model-recommendation, recommended-status-schema, status-schema, gap-005-schema]
tags: [finding, recommendation, status-modeling, schema, gap-005]
---

> [!IMPORTANT]
> Adopt a concrete seven-field frontmatter schema — `status`, `status_category`, `blocked`, `blocked_reason`, `needs_followup`, `followup_reason`, `status_changed` — plus a linked event-note-per-transition for history. This is the single answer four independent research fronts (state-machine theory, mainstream-tool conventions, extensible-taxonomy guidance, markdown-native history patterns) converged on for [[open-knowledge-gaps|gap-005]], assembled from [[sources/research-status-data-model]].

> [!NOTE]
> **Not yet adopted.** Gap-005 stays **open** pending the user's own review — see [[open-knowledge-gaps]] gap-005. This page exists separately from [[concepts/status-modeling]] because that page documents the *pattern* (four dimensions, argued individually across four sibling sources); this page is the *capstone* — one concrete, ready-to-use schema, following this wiki's existing convention of giving decisive cross-source recommendations their own `findings/` page (see [[findings/command-center-design-recommendation]]) rather than burying a full schema inside a `concepts/` page.

No single one of the four status-research sources produced a usable schema on its own — each answered one sub-question of gap-005 (overlapping states, real-world precedent, extensibility, history-recording). [[sources/research-status-data-model]] is the first to assemble all four into one schema a data model can adopt almost as-is.

## The recommended schema

### Item frontmatter

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

| Field | Type | Example values | Role |
|---|---|---|---|
| `status` | string (documented open enum) | `in-flight`, `backlog`, `done`, `unknown` | Region A — the primary workflow phase. One value at a time. |
| `status_category` | string (frozen closed set) | `open`, `closed` (and `unknown` fallback) | The stable layer all views/rollups group by; never gains or loses members. |
| `blocked` | boolean | `true` / `false` | Region B — independent of phase. An item is `in-flight` AND `blocked: true` at once. |
| `blocked_reason` | string | `"waiting on infra review"` | Captures the *why* for the blocking dimension. |
| `needs_followup` | boolean | `true` / `false` | Region C — independent of phase and of blocked. |
| `followup_reason` | string | `"ping stakeholder Fri"` | The *why* for the follow-up dimension. |
| `status_changed` | date | `2026-09-18` | Cheap denormalized "when did phase last move" for at-a-glance views; the event notes are the full record. |

A flat enum can't hold `status` and `blocked` as independent facts without exploding into fused values (`in-flight-blocked`, `backlog-blocked`, …) — the statechart "state explosion" [[concepts/status-modeling]] already names. A full transition-enforcing state machine buys illegal-transition prevention this tool doesn't need. The hybrid keeps every field flat and queryable while still representing compound states honestly.

### How the four daily buckets map onto it

The four buckets are a **query over these fields, not a stored value** — the mechanism that lets one item appear in more than one bucket at once:

| Daily bucket | Query condition |
|---|---|
| **in-flight** | `status == "in-flight"` (regardless of flags — a blocked-but-active item still shows here) |
| **needs-followup** | `needs_followup == true` |
| **blocked / delegate** | `blocked == true` |
| **backlog** | `status == "backlog"` |

An item that is `status: in-flight`, `blocked: true` surfaces in **both** the in-flight and blocked views. This is exactly how Jira/Linear/GitHub keep "blocked" off the primary status field (see [[concepts/status-modeling]] §Real-world examples).

### The transition event-note (history)

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

The parent item's frontmatter holds only current state; this event note is the append-only history — the concrete shape [[concepts/status-modeling]] §Recording transition history already recommends.

### Querying "what changed this week"

```dataview
TABLE from, to, reason, date
FROM "events"
WHERE type = "status-change" AND date >= date(today) - dur(7 days)
SORT date DESC
```

Git commit history (one commit per change, per this wiki's own one-ingest-one-commit convention) sits underneath as the tamper-evident backstop — audit-of-last-resort, not the day-to-day query surface.

## Where the four research fronts agree

- **[[sources/research-status-state-machine|State-machine theory]]** → status + orthogonal boolean flags, not a flat enum, not a full state machine. Grounds it in statechart orthogonal regions; cites Jira's status/resolution/priority separation.
- **[[sources/research-status-mainstream-tools|Mainstream tools]]** → independently confirms the same split across Jira, Linear, and GitHub: a primary phase axis plus blocking as a separate relation/flag; status-change history automatic and first-class in all three.
- **[[sources/research-status-extensible-taxonomies|Extensible taxonomies]]** → the extensibility answer: a frozen top-level category layer over an open, additive sub-status string, an `unknown` fallback, and tags as a pressure valve.
- **[[sources/research-status-markdown-history|Markdown history]]** → frontmatter holds current state only; a linked event-note per transition is the live queryable history; git is the backstop.

Four fronts, one shape — overlapping states from orthogonal flags, transition history from event notes, extensibility from frozen-category-over-open-enum.

## How this composes with existing wiki findings

- [[findings/command-center-design-recommendation]] already recommends GTD's list model for the four buckets and says they "should be the primary status facet (`status:` frontmatter), rendered as views, not folders." This page fills in *how* that facet is structured so an item can sit in two buckets at once — the buckets stay views-not-folders; they just query `status` plus two flags instead of one field.
- [[concepts/frontmatter-as-database]] is the mechanism the whole schema rides on: item state is frontmatter properties, the four-bucket views are faceted Dataview/Bases queries, and event notes are just more frontmatter-bearing pages indexed the same way.
- The Obsidian-plugin-first decision ([[findings/command-center-design-recommendation]] Phase 1) is well served: everything above is plain markdown + YAML + Dataview, no database, and carries forward unchanged into a Phase 2 standalone backend if one materializes.

## Open implementation questions (not resolved here)

These are deliberately left open — the user's call, not this synthesis's:

1. **Exact category names.** `open`/`closed` for `status_category` here; the extensible-taxonomies source also floated `blocked` as a top-level category. Whether blocking is a category or purely a flag (this schema makes it purely a flag) needs confirmation.
2. **Whether `done` is a `status` value or its own category.** Modeled here as a terminal `status` value under `status_category: closed`; could instead be a distinct category if closed items need finer sub-states (e.g. `done` vs. `cancelled`).
3. **Event-note folder location and naming.** Whether transition notes live in a dedicated `events/`-style folder and the exact file-naming convention are unspecified.
4. **Whether `status_changed` is worth denormalizing** onto the item, versus relying on event notes alone.

See [[open-knowledge-gaps]] gap-005 for the tracked version of these four questions.

## Sources

- [[sources/research-status-data-model]]
- [[sources/research-status-state-machine]]
- [[sources/research-status-mainstream-tools]]
- [[sources/research-status-extensible-taxonomies]]
- [[sources/research-status-markdown-history]]

## Related pages

- [[concepts/status-modeling]] — the pattern-level page (theory + real-world examples + extensibility + history mechanisms) this schema concretizes into one usable shape.
- [[findings/command-center-design-recommendation]] — the decided design this schema fills in the status facet for.
- [[concepts/frontmatter-as-database]] — the query/storage mechanism this schema is built on.
- [[open-knowledge-gaps]] — gap-005, still open pending the user's review of this schema.
