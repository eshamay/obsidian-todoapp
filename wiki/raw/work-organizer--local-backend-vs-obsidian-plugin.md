> Pulled from work-organizer wiki (`decisions/local-backend-vs-obsidian-plugin.md`), dated 2026-09-18 — research for a different project (work-organizer command-center); reused here as prior art.

---
title: "Standalone local backend + SPA vs. Obsidian plugin — command-center UI architecture"
type: decision
status: active
created: 2026-09-18
updated: 2026-09-18
sources:
  - id: research-local-markdown-apps
    hash: abbd7806eba5
    ingested: 2026-09-18
  - id: research-personal-crm-integrations
    hash: 7852359ae771
    ingested: 2026-09-18
  - id: research-synthesis
    hash: deaeedbc320a
    ingested: 2026-09-18
aliases: [local-backend-vs-obsidian-plugin, standalone-spa-decision, silverbullet-pattern]
tags: [decision, architecture, ui, local-first, obsidian, silverbullet, open]
---

> [!IMPORTANT]
> **Decision status: decided (2026-09-18).** See [[#Resolution]] below. The tradeoff table and open questions further down are kept as the research record; the "Proposed resolution — phased build" section's own `[!NOTE]` (still saying "not a decision") is superseded by this one.

Whether the command-center's local web-app frontend should be built as an **Obsidian plugin** (inheriting Obsidian's editor, rendering, and plugin ecosystem) or a **standalone local backend + SPA** (a custom server owning file I/O and an index, with a custom frontend, keeping the vault Obsidian-compatible for parallel use) — the central architectural fork surfaced by [[sources/research-local-markdown-apps]].

## The decision

1. **Obsidian plugin** — build inside Obsidian's plugin API, rendering into Obsidian's own panes, reusing its editor/sync/graph and the Bases/Dataview plugin ecosystem.
2. **Standalone local backend + SPA** (recommended by the source, not yet chosen by the user) — a local server process owns vault file I/O and maintains its own query index; a separate frontend SPA renders click-through, textbox-driven views as saved queries/layouts over frontmatter; the vault stays byte-compatible markdown so Obsidian keeps working alongside it.

This decision, once made, resolves the UI-architecture half of [[open-knowledge-gaps]] gap-001 (tech stack) and bears directly on gap-004 (browser-web-app requirement vs. fully-local requirement).

## Options and tradeoffs

| Approach | Gains | Costs |
|---|---|---|
| **Obsidian plugin** | Inherits Obsidian's editor, sync, graph, and Bases/Dataview for free; least code to a working version | UI constrained to Obsidian's plugin API and rendering; hard to build a bespoke fast command-center layout; integrations run inside Obsidian's lifecycle; not a literal browser web-app (desktop/mobile GUI) |
| **Standalone local backend + SPA** | Full UI freedom for tailored, fast views; clean home for Slack/GUS/GitHub/Google connectors at the backend seam; can run headless; literally browser-served, satisfying a strict reading of requirement 4 | Must build editor/index/sync from scratch; must stay disciplined about markdown/YAML round-tripping so Obsidian keeps working in parallel |

A third path — a hedge, not a separate option — is to **prototype against Obsidian's Local REST API plugin first** (turns a running Obsidian instance into a backend a custom frontend can call, fast start but requires Obsidian running), validate the view model, then graduate to a standalone backend once that model stabilizes.

## Recommendation from research

[[sources/research-local-markdown-apps]] recommends option 2 — build a standalone local backend + SPA, keep the vault Obsidian-compatible, and study or fork **SilverBullet** as the closest existing precedent (a self-hosted web app, single server binary or Docker container, storing notes as versioned markdown with a live-preview editor, embedded query database, and scripting environment). The stated rationale: the requirement for fast, customizable, click-through views that change with active work, plus a roadmap toward Slack/GUS/GitHub/Google integrations, exceeds what Obsidian's plugin surface renders comfortably. The source frames option 1 as the right call only "if the person is happy living inside Obsidian's panes," which is explicitly not yet established for this user.

> [!NOTE]
> This recommendation sits at an angle to the Obsidian-as-destination framing in [[sources/research-zettelkasten-networked-notes]] and [[sources/research-eng-leader-pkm]] — those sources evaluate Obsidian (desktop/mobile GUI + Dataview/Tasks/Kanban/Bases) as the adopted tool of choice among real practitioners and this project's strongest local-markdown candidate. This source doesn't contradict that finding; it reframes Obsidian as a possible *starting point* (via the Local REST API hedge) en route to a custom standalone build, rather than the destination. Both can be true: Obsidian remains the best off-the-shelf fit today, and a standalone backend+SPA remains the better fit for this project's specific customization and integration roadmap once built.

## Integration seam, made concrete

[[sources/research-personal-crm-integrations]] fills in the "later Slack/GUS/GitHub/Google connectors attach at the backend seam" line above with a specific architecture, applicable under either option 1 or option 2: OAuth 2.0 device-code auth (no callback server), polling or outbound-initiated connections (Slack Socket Mode) rather than inbound webhooks (no public endpoint required), and a snapshot-to-markdown cache that writes each synced item as a file with `people:` links back to person notes. **Steampipe** is named as a possible uniform connector across sources. See [[concepts/local-first-software]] §Local-first integration architecture for the full pattern.

This narrows, but doesn't resolve, the decision: a standalone backend has an obvious place to run scheduled polling/Socket-Mode listeners and write snapshots (a background process alongside the file-I/O server); an Obsidian-plugin approach would need those same jobs to run inside Obsidian's plugin lifecycle (or as an external script writing into the vault while Obsidian is closed), which is less natural but not ruled out.

> [!NOTE]
> **Clarified by [[decisions/llm-independent-core]] (2026-09-18):** the OAuth device-code pattern above stays valid, but under Phase 1 (Obsidian plugin) it must be implemented as standalone application logic talking directly to each provider's own OAuth endpoints — not by depending on this environment's MCP adaptors for Slack/Google Workspace/etc. The core must authenticate and sync with zero LLM/AI dependency; an LLM/agent plugin is a later, optional add-on, not part of this seam.

## Proposed resolution — phased build

[[sources/research-synthesis]] proposes a way to act on this decision without forcing an either/or, laid out in full on [[findings/command-center-design-recommendation]]:

- **Phase 1:** build inside Obsidian — adopt PARA folders, the status/stakeholder frontmatter schema, per-person notes, and Dataview/Bases views, optionally prototyped through the Local REST API plugin. This validates the data model and the four-bucket views cheaply, and nothing produced here is throwaway because the markdown-on-disk contract carries forward unchanged.
- **Phase 2:** once the schema and views stabilize, build the standalone local backend + SPA option for the literal browser UI (requirement 4) and to host the Slack/GUS/GitHub/Google connectors at the backend seam (requirement 8). Obsidian keeps working in parallel against the same vault.

> [!NOTE]
> This is a proposed sequencing, not a decision. The "decision status: open" line at the top of this page still stands — the user has not chosen between (or ordered) options 1 and 2. What the synthesis adds is a way both options can be true at once, at different points in time, rather than a resolution.

## Resolution

**Decided by the user, 2026-09-18.** Option 1 first, option 2 later, contingent — not the fixed two-phase timeline the synthesis proposed:

- **Phase 1 (starting point): build as an actual Obsidian plugin.** Use Obsidian's Plugin API directly to add the custom objects, views, and UI elements this project needs (not just the Local REST API hedge as an external-app shortcut — the user wants the real plugin surface, including Dataview/Bases where they fit).
- **Phase 2 (contingent, not scheduled): a custom standalone build, "as needed depending on feature needs."** This is triggered by concrete feature gaps discovered while using the Obsidian-plugin build, once the system is established, the data model is defined and designed, and the UI elements are fully explored — not by a calendar or by finishing Phase 1. It may end up not happening at all if the Obsidian plugin keeps satisfying feature needs.

This resolves gap-001 (see [[open-knowledge-gaps]]) and settles the UI half of gap-004's tension (a local GUI is acceptable — see that gap's own resolution). The runtime/SilverBullet/index-strategy questions below remain genuinely open, but now scoped explicitly to "if and when Phase 2 triggers," not to an imminent decision.

## Open questions (apply only if/when Phase 2 triggers)

- Which backend runtime (Node/Deno/Go/Rust) and index strategy (in-memory vs. SQLite)? Not addressed by the source beyond naming the options.
- Is SilverBullet a fork candidate or purely a reference architecture to study? Not yet evaluated.
- What concrete feature gap(s) in the Obsidian-plugin build would actually trigger starting Phase 2? Not yet defined — worth revisiting once the data model and UI elements from Phase 1 exist.

## Related pages

- [[sources/research-local-markdown-apps]]
- [[sources/research-personal-crm-integrations]]
- [[sources/research-synthesis]]
- [[findings/command-center-design-recommendation]]
- [[comparisons/framework-and-tooling-comparison]]
- [[concepts/local-first-software]]
- [[open-knowledge-gaps]] — gap-001, gap-003, gap-004
