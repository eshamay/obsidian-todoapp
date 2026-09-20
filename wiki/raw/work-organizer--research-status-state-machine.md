> Pulled from work-organizer wiki (`raw/research-status-state-machine.md`), dated 2026-09-18 — research for a different project (work-organizer command-center); reused here as prior art.

# Modeling Work-Item Status: Flat Enum vs. State Machine vs. Status-Plus-Flags Hybrid

**Audience:** command-center design (gap-005)<br>
**Question:** how to represent a work item that is simultaneously in a workflow phase *and* blocked, with a record of why/when it moved<br>
**Date:** 2026-09-18

## Summary

Three shapes compete for modeling work-item status: a **flat enum** (one field, one value), an **explicit state machine** (states plus defined transitions), and a **status-plus-orthogonal-flags hybrid** (a primary phase enum alongside independent flags like `blocked` and `needs_followup`). The recurring lesson across statechart theory, workflow-engine practice, and real ticketing systems is that a single flat enum cannot express two independent dimensions at once without exploding into a combinatorial product of states. For the four-bucket command-center — a primary phase that can be blocked *at any phase* and separately flagged for follow-up — the hybrid is the right shape. It is the practical, storage-friendly encoding of what statechart theory calls **orthogonal regions**.

## The core tradeoff

A **flat enum** is the simplest option: one field, trivially queryable and sortable, one value at a time. A finite state machine is formally "an abstract machine that can be in exactly one of a finite number of states at any given time" ([statecharts.dev](https://statecharts.dev/what-is-a-state-machine.html)). That "exactly one" is the constraint. The moment an item can be both *in-flight* and *blocked*, a single enum forces you to either pick one and lose the other, or invent a fused value like `in_flight_blocked` for every combination. Modeling independent dimensions this way grows the state count as a product, not a sum — statecharts.dev notes five independent booleans already imply 32 states, most invalid ([statecharts.dev](https://statecharts.dev/what-is-a-state-machine.html)).

An **explicit state machine** adds defined transitions on top of states, so each state+event pair maps deterministically to a next state ([Stately](https://stately.ai/docs/state-machines-and-statecharts)). This buys two things a flat enum lacks: it makes illegal transitions unrepresentable, and — if transitions are logged — it gives the history of *when and why* an item moved, which is exactly the audit trail gap-005 wants. The cost is machinery: a transition table, guards, and enforcement code that a personal tool may not want.

The opposite failure mode is worth naming too. Representing status as several independent booleans (`isBlocked`, `isDone`, `needsFollowup`) invites contradictory combinations — the classic bug where an item shows stale data after an error because the flags disagree. Kent C. Dodds' widely-cited write-up shows how separate booleans let a component "show the position if there's no error… [or] only an error, even if subsequent requests succeed," and argues a single explicit `status` "enable[s] our users to know exactly what the state is at any given point" ([kentcdodds.com](https://kentcdodds.com/blog/stop-using-isloading-booleans)). The takeaway is not "avoid flags" but "don't scatter the *primary* phase across booleans" — keep one authoritative phase, and reserve flags for genuinely independent dimensions.

## Statecharts and orthogonal regions

David Harel's statecharts (1987) were designed as "the 'bigger brother' of state machines… to overcome some of the limitations of state machines" ([statecharts.dev](https://statecharts.dev/what-is-a-state-machine.html)). The mechanism that fits "in-flight AND blocked" precisely is the **orthogonal region** (also called a parallel or "and" state). In UML state machines — "an object-based variant of Harel statechart" — "a composite state can contain two or more orthogonal regions… and being in such a composite state entails being in all its orthogonal regions simultaneously" ([Wikipedia: UML state machine](https://en.wikipedia.org/wiki/UML_state_machine)). An entity is in one state from dimension A *and* one from dimension B at the same time.

This is the direct theoretical answer to gap-005. XState frames it with a media-player example: two concurrent regions, one tracking playing/paused and another normal/muted, so the player can be "simultaneously paused and muted" ([Stately](https://stately.ai/docs/parallel-states)). Map that onto the command-center: region A is the primary phase (in-flight / backlog / done), region B is the blocking dimension (clear / blocked), region C is follow-up (none / needs-followup). Critically, orthogonal regions turn combinatorial growth back into linear growth — modeling four independent switches needs "eight states… and eight transitions" as parallel regions versus "16 atomic states… and 64 transitions" if fused ([statecharts.dev](https://statecharts.dev/glossary/parallel-state.html)). UML calls the fused alternative the "state and transition explosion," where "the complexity of a traditional FSM tends to grow much faster than the complexity of the system it describes" ([Wikipedia](https://en.wikipedia.org/wiki/UML_state_machine)).

## What workflow-engine and ticketing practice actually do

Two observations from production systems. First, **BPMN workflow engines** already model true concurrency at runtime: a parallel gateway means "all outgoing sequence flows are followed in parallel, creating one concurrent execution for each," so a single process instance holds multiple active tokens at once ([Camunda](https://docs.camunda.org/manual/latest/reference/bpmn20/gateways/parallel-gateway/)). Concurrent-active-state is a first-class idea in workflow tooling, not an edge case.

Second, and more directly relevant, **ticketing systems separate the primary phase from orthogonal dimensions rather than fusing them into one enum.** Jira keeps `status` ("a work item's current place in the workflow"), `resolution` ("how a work item was completed… set when the status is changed"), and `priority` as three distinct fields ([Atlassian](https://support.atlassian.com/jira-cloud-administration/docs/what-are-issue-statuses-priorities-and-resolutions/)). The `status` is a state machine; resolution and priority are the orthogonal flags. So mature practice favors exactly the hybrid: one governed primary phase plus independent side-dimensions, because blocked-ness and urgency are not workflow positions and should not consume workflow-position values.

## Why status enums grow — and what that implies

Status vocabularies accrete over time because every fused combination and every newly-noticed distinction tempts a new value; that is the state-explosion tendency again ([Wikipedia](https://en.wikipedia.org/wiki/UML_state_machine)). The design implication is *not* to pre-model every future state. Fowler's YAGNI argues against speculative generality, citing the cost of build, delay, carry, and repair, while noting it "does not apply to effort to make the software easier to modify" ([martinfowler.com](https://martinfowler.com/bliki/Yagni.html)). Start with the minimal phase set; keep the phase dimension separate from the flag dimensions so that adding a flag later never forces re-encoding existing rows.

## Recommendation

For "primary phase + can-be-blocked-at-any-phase + needs-followup," use the **status-plus-orthogonal-flags hybrid**, not a flat enum and not a full transition-enforcing state machine:

- `status:` — a small enum for the primary phase (`in-flight`, `backlog`, and a terminal `done`). This is region A.
- `blocked: true/false` + `blocked_reason:` — an independent flag, valid at any phase. Region B.
- `needs_followup: true/false` — a third independent flag. Region C.

The four daily buckets then fall out of a query, not a stored value: *needs-followup* = flag C; *blocked* = flag B; *in-flight* = `status:in-flight` and not blocked; *backlog* = `status:backlog`. This keeps each field flat and queryable (the flat-enum win) while representing compound states honestly (the statechart-orthogonality win), and it matches how Jira and other trackers actually separate these concerns. Add a lightweight `history:` / transition log if the "when and why it moved" requirement matters — that is the one thing flags alone don't give you, and it's the reason to borrow the state-machine's transition-logging idea even while declining its full enforcement machinery. Defer any richer state set until a concrete need appears (YAGNI).

## Citations

- Finite state machine definition, flat-FSM limits, state explosion: https://statecharts.dev/what-is-a-state-machine.html
- Parallel/orthogonal states, linear-vs-exponential complexity: https://statecharts.dev/glossary/parallel-state.html
- XState parallel states, "paused and muted" example: https://stately.ai/docs/parallel-states
- State machines vs. ad-hoc flags, deterministic transitions: https://stately.ai/docs/state-machines-and-statecharts
- UML/Harel orthogonal regions, "state and transition explosion": https://en.wikipedia.org/wiki/UML_state_machine
- Boolean-flag impossible states, single-status argument: https://kentcdodds.com/blog/stop-using-isloading-booleans
- BPMN parallel gateway, concurrent executions/tokens: https://docs.camunda.org/manual/latest/reference/bpmn20/gateways/parallel-gateway/
- Jira status vs. resolution vs. priority (separate orthogonal fields): https://support.atlassian.com/jira-cloud-administration/docs/what-are-issue-statuses-priorities-and-resolutions/
- YAGNI, cost of speculative generality: https://martinfowler.com/bliki/Yagni.html
