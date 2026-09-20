> Pulled from work-organizer wiki (`raw/research-status-mainstream-tools.md`), dated 2026-09-18 — research for a different project (work-organizer command-center); reused here as prior art.

# How Mainstream Task-Tracking Tools Model Status

**Compiled:** 2026-09-18<br>
**Purpose:** Comparative reference for gap-005 — modeling status in a local markdown+YAML command-center, where an item may be simultaneously in-flight AND blocked, changes must be dated/explained, and new states must be addable without breaking existing data.<br>
**Scope:** Jira, Linear, GitHub Issues/Projects, Trello/Kanban.

## Summary

Across all four tools, the decisive design pattern is the same: **primary workflow phase is one axis, and "blocked/waiting" is modeled on a separate axis** — never as a competing value in the single status field. Jira, Linear, and GitHub all express "blocked" as a *relationship to another item* (a dependency link) rather than a status, precisely so an issue can be "In Progress" and "blocked by #42" at once. Trello, having no native dependency, forces the two axes together (a "Blocked" list) or bolts blocking on via labels/Power-Ups. Status-change history is a first-class, automatic audit trail in Jira, Linear, and GitHub; in plain Trello it is a manually-read activity feed. Extensibility is strong everywhere for adding *named* states, but Jira/Linear/GitHub all preserve a small fixed set of **status categories** (To Do / In Progress / Done) underneath the user-editable names — which is the key lesson for a schema meant to outlive its own vocabulary.

## Jira

**1. Core status concept.** A Jira status marks an issue's current place in a *workflow*. Statuses are not global free-standing values; they live inside a workflow, and workflows are bound to issue types per project through a **workflow scheme** (the set of associations between workflows and work types, reusable across projects). A workflow is statuses plus **transitions**, where each transition is one-directional — moving both ways between two statuses requires two transitions. Every status also carries one of three fixed **status categories** (To Do, In Progress, Done) that drive board columns and reporting regardless of the custom name. [1][2]

**2. In-progress-but-blocked.** Jira supports two idioms. Some teams add a literal "Blocked" status to the workflow — but that collapses the two axes and loses the underlying phase. The recommended idiom is the **"is blocked by" issue link** (a dependency relationship to another issue), which leaves the primary status untouched, so an issue stays "In Progress" while carrying a visible blocker link. Flags and link types express the waiting relationship separately from the status field. [2]

**3. Status-change history.** Every issue has a built-in **History/Activity** record. Field changes — including each status transition, with the from-value, to-value, actor, and timestamp — are captured automatically; nothing is bolted on. (A separate admin-level audit log tracks configuration changes.) [2]

**4. Extensibility.** New statuses and transitions can be added to a workflow at any time, and because each maps to a fixed category, existing boards/reports keep working. The cost is migration discipline: reassigning workflow schemes to issue types is an administrative operation, so status taxonomy is extensible but centrally governed rather than casual. [1][2]

## Linear

**1. Core status concept.** Linear issues have a single **workflow state**, and every state belongs to one of five fixed **categories**: Backlog, Unstarted, Started, Completed, Canceled (plus a system-reserved Duplicate, and Triage acting as an inbox-style category). Workflows are **per-team**: each team gets a default sequence (Backlog → Todo → In Progress → Done → Canceled) and can rename, recolor, reorder, and add states — but the five categories themselves are fixed. [3][4]

**2. In-progress-but-blocked.** "Blocked" is explicitly **a relation, not a workflow status.** Linear models Blocking / Blocked-by / Related / Duplicate as issue relations shown with colored flags (orange "blocked by", red "blocks"). An issue can sit in any state such as In Progress while carrying a "blocked by" relation; when the blocker resolves, the link demotes to Related. This is the cleanest native example of the two-axis pattern. [4][5]

**3. Status-change history.** Linear records state changes automatically in an issue's activity timeline, and exposes state-transition timestamps that power cycle-time analytics — so "when did this become Started/Completed" is queryable, not reconstructed from comments. [3]

**4. Extensibility.** Adding a state within an existing category is low-friction and non-breaking, because views and automations key off the category, not the specific name. Renaming/reordering is safe for the same reason. The one hard boundary is the five categories, which cannot be extended. [3]

## GitHub Issues / Projects

**1. Core status concept.** GitHub has two layers. A raw **issue** has only `open`/`closed` state (with a closed *reason*: completed vs. not planned). Richer workflow status lives in **Projects**, where **Status is a single-select custom field** used as the board's column field. Board columns and the Status value stay synchronized — dragging a card rewrites its Status. Projects also support other custom single-select and iteration fields as the column axis. [6][7]

**2. In-progress-but-blocked.** GitHub separates the axes three ways: (a) the Status field for phase, (b) **labels** (free-form, e.g. a "blocked" label) for lightweight flagging, and (c) **issue dependencies** — a native blocked-by / blocking relationship — plus sub-issues and linked pull requests for hierarchy and closure. An issue can be Status="In Progress" and simultaneously "blocked by" another issue via the dependency relation. [8][7]

**3. Status-change history.** Each issue has an automatic timeline recording labeling, closing, reopening, and cross-references, with actor and timestamp. Project field changes (including Status) are likewise recorded per item. History is built-in, not comment-based. [8]

**4. Extensibility.** Highly extensible and low-ceremony: add Status options or whole new single-select fields per project without migration. The tradeoff is that Status options are **project-scoped and unopinionated** — there is no enforced category underneath, so consistency across projects and automation robustness depend on convention rather than a guaranteed taxonomy. [6][7]

## Trello / General Kanban

**1. Core status concept.** In Trello, **status is position** — the **list** (column) a card sits in *is* its stage. There is no separate status field; a card is in exactly one list. Swimlanes/list ordering give the board its workflow shape. [9]

**2. In-progress-but-blocked.** This is Trello's weak spot. Native Trello has **no blocked/blocking dependency**. Teams either (a) create a dedicated "Blocked" list — which conflates phase and blocking, since a card can only be in one list — or (b) apply a **"Blocked" label** (labels are multi-valued and independent of lists) to flag a blocked card while it stays in its phase list, or (c) add a dependency Power-Up. Only options (b)/(c) preserve the two-axis separation. [10]

**3. Status-change history.** Card movement between lists is recorded in the card's **activity feed** with actor and timestamp, so the raw events exist — but there is no structured status-transition report; querying "how long in each list" requires reading the feed or a Power-Up/analytics add-on. [9]

**4. Extensibility.** Adding lists or labels is trivial and instant, with no schema or migration. The flip side is zero governance: nothing distinguishes a "phase" list from any other, no fixed categories exist, and automation (Butler rules) that keys off exact list/label names can silently break when names change. [9][10]

## Comparison

| Tool | 1. Core status concept | 2. In-progress + blocked | 3. Status-change history | 4. Taxonomy extensibility |
|------|------------------------|--------------------------|--------------------------|---------------------------|
| **Jira** | Status inside a workflow; bound to issue type via workflow scheme; 3 fixed categories under custom names [1][2] | "is blocked by" **issue link** (separate axis); a literal Blocked status is possible but discouraged [2] | Built-in per-issue History: every transition with from/to/actor/time [2] | Add statuses/transitions freely; categories fixed; scheme reassignment is a governed admin step [1][2] |
| **Linear** | Single workflow state; 5 fixed categories; per-team workflows [3][4] | "Blocked" is a **relation, not a status**; flags coexist with any state [4][5] | Automatic activity timeline; transition timestamps power cycle-time [3] | Add states within a category non-breaking; 5 categories are the hard ceiling [3] |
| **GitHub** | Issue = open/closed; Projects **Status = single-select field** = board column [6][7] | Three axes: Status + labels + native **dependencies** (blocked-by) + sub-issues [8][7] | Automatic issue timeline + per-item project field-change history [8] | Very extensible per project; no enforced category underneath (convention-dependent) [6][7] |
| **Trello** | **Status = list/column position**; one list per card; no status field [9] | No native dependency; use a **"Blocked" label** or Power-Up to keep phase separate; a Blocked list conflates axes [10] | Card activity feed logs moves (actor/time); no structured transition report natively [9] | Add lists/labels instantly, zero migration; also zero governance/categories [9][10] |

**Takeaway for gap-005:** model a **primary status with a small fixed category set** (mirroring Jira/Linear — e.g. in-flight / needs-followup / blocked / backlog as *categories*, with room for named sub-states beneath), keep **"blocked" as a separate boolean/relation axis** rather than a status value so an item can be in-flight AND blocked, and store status changes as an **append-only dated log** (the Jira/Linear built-in-history model) rather than overwriting a single field.

## Citations

1. Atlassian — *What are issue statuses, priorities, and resolutions?* https://support.atlassian.com/jira-cloud-administration/docs/what-are-issue-statuses-priorities-and-resolutions/
2. Atlassian — *Jira workflows overview* (statuses, transitions, workflow schemes, resolutions). https://www.atlassian.com/software/jira/guides/workflows/overview
3. Linear — *Configuring workflows* (status categories, per-team states). https://linear.app/docs/configuring-workflows
4. Linear — *Issue relations* ("Blocked is a relation, not a status"). https://linear.app/docs/issue-relations
5. Linear — *Docs home.* https://linear.app/docs
6. GitHub Docs — *About the Status field* (Projects). https://docs.github.com/en/issues/planning-and-tracking-with-projects/understanding-fields/about-the-status-field
7. GitHub Docs — *Changing the layout of a view* (Status as column field, custom single-select fields). https://docs.github.com/en/issues/planning-and-tracking-with-projects/customizing-views-in-your-project/changing-the-layout-of-a-view
8. GitHub Docs — *About issues* (open/closed, labels, dependencies/blocked-by, sub-issues, linked PRs). https://docs.github.com/en/issues/tracking-your-work-with-issues/about-issues
9. Atlassian — *Creating and managing lists* (Trello). https://support.atlassian.com/trello/docs/creating-and-managing-lists/
10. Atlassian — *Adding labels to cards* (Trello; labels distinct from lists, multi-valued). https://support.atlassian.com/trello/docs/adding-labels-to-cards/
