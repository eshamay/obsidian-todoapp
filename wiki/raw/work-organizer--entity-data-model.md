> Pulled from work-organizer wiki (`decisions/entity-data-model.md`), dated 2026-09-18 — research for a different project (work-organizer command-center); reused here as prior art.

---
title: "Entity/data model — work-organizer command center (locked v1)"
type: decision
status: active
created: 2026-09-18
updated: 2026-09-18
sources: []
aliases: [entity-data-model, domain-model, data-model-proposal, locked-schema]
tags: [decision, data-model, entities, schema, locked]
---

> [!IMPORTANT]
> **Decided 2026-09-18.** Originally proposed from a 28-question, 6-round interview with the user (13 entities' prose shape below). Then audited twice before locking: **Audit 1** checked a draft literal-YAML lock against this wiki's own synthesis pages (3 fixes applied — `blocked_reasons` as an array, real `[[wikilinks]]` instead of freeform strings, an explicit unknown/catch-all bucket in the consuming views). **Audit 2** independently re-derived the entity model from the *primary* external research (GTD/PARA/BASB/Zettelkasten/Johnny-Decimal/eng-leader-PKM/personal-CRM/status-modeling sources directly, not this wiki's synthesis of them) and surfaced real structural gaps, which the user then triaged. This closes [[open-knowledge-gaps]] gap-002.

Direct source of the 11 original entities: the user's own interview answers, not external research. **Area** and **Insight** were added from Audit 2's comparison against the primary research. Where a fact composes with prior research (e.g. gap-005's status-modeling recommendation), that's cross-linked explicitly.

## v1-locked entities (real CRUD/UI in the plugin)

Item, Person, Project, and Area get literal YAML frontmatter locked below and real create/edit UI in the v1 Obsidian plugin. Everything else stays prose-shape only for now.

### Item
One unified entity for what would otherwise be separate todo/followup/blocker/work-item/epic types — "something that needs to get done for one reason or another; a pending item that needs action and has a status." Matches gap-005's researched recommendation: one primary status plus independent orthogonal flags, not separate types or a fused enum.

```yaml
---
type: item
title: ""
status: backlog          # in-flight | backlog | done | cancelled | unknown
status_category: open    # open | closed | unknown — derived from status
blocked: false
blocked_reasons: []      # array of {kind, detail} — supports concurrent blockers
                          #   - kind: resource
                          #     detail: "waiting on budget approval"
                          # kind: person-reply | decision-pending | external-dependency | resource |
                          # implementation-timeline | release-schedule | product-launch | moratorium | unknown
blocked_by: []            # [[wikilinks]] to the specific blocking Person/Decision/Item, if known
needs_followup: false
followup_direction: ""   # i-owe | owed-to-me | self | unknown
followup_reason: ""
context: []                # free-form GTD-style tags, e.g. [computer, phone, errand] — "what can I act on right now"
owner: ""                  # [[Person]] wikilink, optional — items can be owned/delegated to people other than the user
due_date: ""
created_date: 2026-09-18
status_changed: 2026-09-18
priority: ""                # explicit rank, meaningful when project/area-linked, set by others on a backlog
project: ""                 # [[Project]] wikilink, optional
area: ""                     # [[Area]] wikilink, optional — alternative to project for ongoing-responsibility-scoped items
gus_ref: ""                  # v1: reference-only (link + notes); v2: pulled from GUS directly
tags: []
---
```

- **The formality gradient:** the same entity is a lightweight personal todo when free-floating, and functions as a work-item/epic when project/area-linked and/or GUS-referenced — no separate "epic" type.
- **Terminal states:** `done` and `cancelled` are both terminal, under `status_category: closed` — mirrors [[#Project]]'s own completed/archived/abandoned distinction, which the original interview-only design had inconsistently lost for Item (Audit 2 finding #1, highest severity: every dropped/superseded item was being forced into "done," corrupting any retrospective/brag-doc view).
- **`blocked_by` vs. `blocked_reasons`:** `blocked_reasons[]` captures the *category* of why (per the researched taxonomy); `blocked_by[]` captures *who/what specifically* — a structured link, not just a category — mirroring Jira/Linear's "blocked by #42" relation (Audit 2 finding).
- **`context[]`:** the GTD engage-filter half ("what can I act on right now" given tools/location/energy) that the original design only covered via `followup_direction`'s delegation half (Audit 2 finding).
- **Explicitly deferred, not solved by `status_changed`:** the event-note-per-transition history pattern from [[findings/status-data-model-recommendation]] and its git-commit backstop. `status_changed` only records *that* status last moved, not from/to/why. The operational vault won't be a git repo either, so there's no backstop-of-last-resort — a real, acknowledged v1 gap, not a solved problem.
- **Also deferred:** `workstream` link (Workstream isn't v1-locked) and any Project→Item status-cascade logic.
- **Relationships:** optional [[#Project]] or [[#Area]]; [[#Person]] (owner); optional links via `blocked_by`.

### Person
A stakeholder. Spans multiple teams, multiple projects, and multiple overlapping reference documents — many-to-many on every axis, never a single home (confirmed directly by the user).

```yaml
---
type: person
title: ""
stakeholder_type: ""   # exec | architect | developer | manager | product | vendor | informed | unknown
team_current: []        # [[Team Name]] wikilinks — unresolved links are fine; Team isn't v1-locked but a
                          # wikilink still backlinks/queries correctly and resolves once a real Team note exists
org_current: ""
team_history: []        # strings embedding a wikilink + date range, e.g. "[[Platform Team]] (2024-01 – 2025-03)"
tags: []
---
```

- **Team/org membership** carries a two-tier `current` + `history` shape, explicitly requested in the interview.
- **Relationships:** many-to-many [[#Team]] (unresolved links), [[#Project]]/[[#Area]] (via team or direct stakeholder role), [[#Item]] (as owner).

### Project
A container for a bounded (but not always upfront-dated) effort. Holds design, timeline, effort/sizing, stakeholders, collaborators, reference materials, links.

```yaml
---
type: project
title: ""
status: active           # active | on-hold | completed | archived | abandoned | unknown
priority: ""
effort_size: ""            # t-shirt size / dev-time / cost estimate, freeform text in v1
driver: ""                  # [[Person]] wikilink, optional — current owner; handoffs recorded as [[#Decision]] entities, not a bare history field
owning_teams: []           # [[Team Name]] wikilinks — may be one or many, shared, evolves over time
created_date: 2026-09-18
tags: []
---
```

- **Cascade:** setting a project to a terminal status (completed/archived/abandoned) cascades to its workstreams and items — tied, not independently terminal-stated when the parent closes. (Not implemented in v1 — see Item's deferred notes.)
- **Relationships:** has-many [[#Workstream]] (not v1-locked); has-many [[#Item]] (direct, project-scoped); has-many [[#Reference]]; has-many [[#Person]] (stakeholders); has-many [[#Interaction]]; has-many [[#Decision]]; many-to-many [[#Team]].

### Area *(added by Audit 2)*
PARA's ongoing-responsibility concept — e.g. "manage the platform team," "on-call ownership" — distinct from a bounded [[#Project]]. Audit 2 called this the single most decisive fix for this specific user's situation (many concurrent teams/orgs with ongoing, non-bounded duties): "a distinction a flat manual vault typically lacks, which is a common reason such vaults stop scaling" (per `methodologies/para.md`).

```yaml
---
type: area
title: ""
status: active           # active | inactive | archived | unknown — areas don't "complete"; they're ongoing by definition
driver: ""                  # [[Person]] wikilink, optional — current owner of this responsibility
owning_teams: []           # [[Team Name]] wikilinks
created_date: 2026-09-18
tags: []
---
```

- **No terminal "completed" state** — an Area can go inactive or be archived/retired, but the notion of "done" doesn't apply the way it does for a Project.
- **No Workstream sub-division in v1** — Workstreams are Project-specific per the original interview; whether Areas need their own sub-division is an open question for v2.
- **Relationships:** has-many [[#Item]] (alongside/instead of Project); has-many [[#Person]], [[#Team]], [[#Reference]].

## Documented-only entities (prose shape, not locked to literal YAML, not in v1 build)

### Workstream
A sub-unit of exactly one project, encapsulating one aspect of it: its own people/teams, effort, cost, dependencies, timeline, work items/epics, references.

- **Fields:** `title`, `status` (cascades from parent project), `priority`, `effort`/`cost`, `timeline`.
- **Relationships:** belongs-to exactly one [[#Project]]; has-many [[#Item]]; has-many [[#Person]]/[[#Team]]; has-many [[#Reference]].

### Team
Represents a scrum team, a virtual team, or simply the core group of people working a project — not strictly bound to the org chart.

- **Fields:** `name`, `kind` (scrum-team | virtual-team | core-group).
- **Relationships:** has-many [[#Person]] (current + history — see Person); loosely rolls up to an [[#Org]]; many-to-many [[#Project]]/[[#Area]].

### Org
The corporate hierarchy rolling up to a leader — VP, SVP, EVP, or above. Distinct from Team: a Team is the working group, an Org is the reporting structure it sits under.

- **Fields:** `name`, `leader` (link to [[#Person]]).
- **Relationships:** loosely has-many [[#Team]].

### Reference
A shared document or link — the reference-tracking half of the original 8 requirements. Types in scope: design docs/RFCs, decks, sheets, Slack threads, GUS records, email, markdown files, Obsidian vaults, sticki-wikis, ad hoc markdown notes.

- **Fields:** `type`, `url`/`link`, `notes` (the user's own explanatory text), `summary` *(added by Audit 2 — a distilled/synthesized takeaway distinct from raw `notes`, per BASB's Progressive Summarization; matters more once v2 pulls real content)*, `tags`.
- **v1 (now):** link + notes/tags/details only — no content pulled in.
- **v2 (later):** pull and store actual content/parts via provider APIs, contingent on the integration work under [[decisions/llm-independent-core]].
- **Relationships:** many-to-many [[#Project]], [[#Area]], [[#Person]], [[#Decision]], [[#Item]], [[#Interaction]].

### Interaction
One unified entity for meetings, syncups, chats, and any other interpersonal touchpoint — not separate types. A `type`/label attribute distinguishes a calendar-sourced meeting from an ad hoc chat.

- **Fields:** `interaction_type` (meeting | syncup | chat | slack-dm | …), `date`/`time`, `attendees`, `notes`/`agenda`, `transcript` (future), `source` (manual | calendar-integration).
- **v1 (now):** only notable 1:1s get a note; Slack DMs count only when explicitly called out.
- **v2 (later):** pulled automatically from Google Calendar/Workspace.
- **Relationships:** many-to-many [[#Project]], [[#Area]], [[#Person]], [[#Reference]], [[#Item]], [[#Decision]].

### Work-list
A cheap, freely-creatable/archivable named list of Items, capturing execution order — separate from an Item's own `priority` field.

- **Fields:** `title`, `status` (active/archived), `scope` (optional project/area link, or free-floating).
- **List entries are polymorphic:** each entry references either an [[#Item]] or another Work-list (recursive/nested composition).
- **Relationships:** has-many ordered entries; optional [[#Project]]/[[#Area]] scope.

### Decision
An architecturally- or business-significant choice, kept so the user can "reconstruct how we got to a point in a project" and recount the full decision branching later.

- **Fields:** `title`, `why`, `when`, `how`, `who` (Person links), `status` (open | decided).
- **Branching:** a Decision can link to prior Decisions it supersedes or builds on — a decision graph, not a flat list.
- **Used for structural changes too:** a project ownership handoff is recorded as a Decision, not a bare history field.
- **Relationships:** many-to-many [[#Person]], [[#Reference]]; optional [[#Project]]/[[#Area]] link; optional supersedes/related-decision links.

### Daily Journal / Log
A running, brag-document-style daily record, separate from project-specific notes.

- **Fields:** `date`, freeform entries, loose pointers to what was worked on that day.
- **Relationships:** loosely references [[#Project]]/[[#Area]]/[[#Item]]/[[#Person]] mentioned, but is not owned by any of them.

### Insight *(added by Audit 2)*
Zettelkasten/BASB's evergreen/permanent note — the user's own synthesized, durable understanding (e.g. "how our auth pipeline actually works," "my model of X"), distinct from [[#Reference]] (others' material) and [[#Decision]] (a specific choice with rationale). Audit 2 flagged this as a structural gap: "half the researched hybrid (Zettelkasten/BASB/Matuschak) is about this layer; the model under-represents it."

- **Fields (prose shape, not yet locked):** `title`, `status` (draft | evergreen | superseded — mirrors this wiki's own concept-page lifecycle), `tags`, `created_date`, `updated_date`.
- **Its main relationship mechanism is prose wikilinks, not frontmatter fields** — like a personal concept page, its value is largely in the body text and its backlinks, in the Zettelkasten style.
- **Relationships:** many-to-many [[#Reference]] (sources that informed it), [[#Item]], [[#Decision]], [[#Project]]/[[#Area]] (context it's relevant to), other Insights (backlinks).

## Cross-cutting mechanics

- **Storage:** every entity is a markdown file with YAML frontmatter, per [[irrefutable-facts]] and [[concepts/frontmatter-as-database]] — relationships are frontmatter link arrays, not a database.
- **`unknown` sentinel:** every v1-locked enum (Item.status, Item.followup_direction, Item's blocked_reasons kind, Person.stakeholder_type, Project.status, Area.status) carries an explicit `unknown` fallback, per the extensible-taxonomy research's Protobuf/AIP-126-style pattern (Audit 2 finding — previously inconsistent across the model).
- **Status cascade:** Project → Workstream → Item is one-directional on terminal-state changes. Not implemented in v1.
- **Three tiers of "what changed and why," by weight:** Decision entity (significant, narrative changes) → event-note-per-transition (routine Item status/blocked flips, deferred past v1) → current+history fields (lighter tracking, e.g. Person's team/org membership).
- **Formality gradient, not separate types:** Item unifies todo/followup/blocker/work-item/epic; Interaction unifies meeting/syncup/chat. Both explicit user calls from the interview.

## Explicitly declined

A leverage/impact property on Item (Work on What Matters' selection criterion, distinct from the externally-set `priority` rank) — surfaced by Audit 2, explicitly declined by the user. Not added.

## Open questions

- Whether Work-list's polymorphic entry needs an explicit `entry_type` discriminator or can be inferred from the link target's frontmatter `type`.
- Whether `informed` stakeholders get any automated behavior in v1 (current read: no — a marker only until integrations land).
- How Decision's "supersedes" links compose with event-note history when a Decision itself gets revisited.
- Whether Areas need their own Workstream-style sub-division in v2.
- Insight's literal frontmatter schema — not locked yet, prose shape only.

## Related pages

- [[open-knowledge-gaps]] — gap-002 (closed by this page).
- [[findings/status-data-model-recommendation]] — the Item status/history pattern this model adopts (overlapping-states + terminal-state fix; transition-history explicitly deferred).
- [[concepts/status-modeling]], [[concepts/frontmatter-as-database]] — the underlying mechanisms this model composes with.
- [[decisions/local-backend-vs-obsidian-plugin]], [[decisions/llm-independent-core]] — the architecture decisions this data model is built under.
