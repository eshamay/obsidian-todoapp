> Pulled from work-organizer wiki (`raw/research-status-markdown-history.md`), dated 2026-09-18 — research for a different project (work-organizer command-center); reused here as prior art.

# Recording Status-Change History in a Markdown-Native, Git-Tracked System

> **Audience:** command-center design (gap-005) <br>
> **Purpose:** how to record WHEN and WHY a `status:` field changed without a database <br>
> **Date:** 2026-09-18 <br>
> **Scope:** plain-markdown notes, YAML frontmatter, git-tracked, Obsidian Dataview-queryable

## Summary

Frontmatter is a current-state store, not a history store: Obsidian Properties "hold current state only" and "don't natively track historical changes" ([obsidian.md/help/properties](https://obsidian.md/help/properties)). So a `status:` field alone tells you what a thing *is*, never how it got there. Three markdown-native patterns fill the gap: (1) an in-file append-only changelog block, (2) a separate small note per status-change event, and (3) reconstructing history from git commits. They are not mutually exclusive — the strongest design uses frontmatter for current state, dated event lines/notes for a live queryable history, and git as the tamper-evident backstop. Recommendation is at the end.

## 1. In-file append-only changelog block

The common PKM pattern is a `## History` / `## Changelog` section holding dated bullets like `- 2026-09-18 → blocked (waiting on infra review)`. It is the lowest-friction option — everything stays in one file, diffs cleanly, and reads well.

Its query limitation is the important part. Obsidian Properties (frontmatter) are the only metadata that "applies to the whole page" and is trivially queryable ([Dataview add-metadata](https://blacksmithgu.github.io/obsidian-dataview/annotation/add-metadata/)). Body bullets are *not* invisible to Dataview, but you cannot attach a queryable field to a plain bullet — "Bracketed inline fields are the only way to explicitly add fields to specific list items, YAML frontmatter always applies to the whole page" ([add-metadata](https://blacksmithgu.github.io/obsidian-dataview/annotation/add-metadata/)). So a queryable changelog bullet must carry inline fields: `- [changed:: 2026-09-18] [to:: blocked] waiting on infra`. Dataview then exposes every bullet through the implicit `file.lists` collection — "all your bullet points have all the information described here available" ([metadata on tasks/lists](https://blacksmithgu.github.io/obsidian-dataview/annotation/metadata-tasks/)) — and you reach them with `FLATTEN file.lists AS L` ([data-commands](https://blacksmithgu.github.io/obsidian-dataview/queries/data-commands/)). It works, but it is verbose, and querying *into* body sections is second-class compared with frontmatter.

## 2. Linked "event note" per status change

Here each transition becomes its own small note — e.g. `events/2026-09-18-project-x-blocked.md` — with the transition modeled as frontmatter and a link back to the parent item:

```yaml
---
type: status-change
item: "[[Project X]]"
date: 2026-09-18
from: active
to: blocked
reason: waiting on infra review
---
```

Because the event note keeps its data in *frontmatter*, every field is first-class queryable — no inline-field workaround, no `FLATTEN` gymnastics. This is the same move the Obsidian Tasks plugin makes for dated task lines: it recognizes absolute `YYYY-MM-DD` dates (created ➕, done ✅, due 📅, etc.) and makes each "accessible for filtering, sorting, and grouping" ([Tasks/Dates](https://publish.obsidian.md/tasks/Getting+Started/Dates)). The event-note pattern generalizes that idea from tasks to arbitrary transitions, and turns history into a folder of filterable objects rather than prose buried in a body section. A `FROM "events"` collection filtered by date and grouped by item ([data-commands](https://blacksmithgu.github.io/obsidian-dataview/queries/data-commands/)) directly answers "what changed this week." Cost: more files, and one extra write per transition.

## 3. Git history as the audit trail

Because every note is a git-tracked file and the project already does "one ingest = one git commit," the commit history *already is* a status audit trail — no in-file history required. `git log -p <file>` shows the diff at each commit; `git log -L :status:<file>` (or a line/regex range) will "trace the evolution of the line range… within the file"; and the pickaxe `-S`/`-G` finds commits where a specific value was "added or removed." Date windows come from `--since`/`--until` ([git-log docs](https://git-scm.com/docs/git-log)). Every past value of `status:` is recoverable this way, and the record is tamper-evident (rewriting it means rewriting history).

The tradeoff is queryability. Reconstructing "what changed this week and why" from raw commits is exactly the pain Simon Willison's `git-history` tool was built to relieve: "reading through thousands of commit differences and eyeballing changes… isn't a great way of finding the interesting stories," so the tool "reads through the entire history of a file and generates a SQLite database reflecting changes to that file over time" ([git-history](https://simonwillison.net/2021/Dec/7/git-history/)). Two consequences for this design: git history is *not* live-queryable from inside Obsidian (Dataview cannot read it), and the "why" is only captured if commit messages are disciplined — the diff shows *what* changed, the message must supply the reason. Willison frames the general choice as implicit history (reuse git, minimal storage, harder queries) vs. explicit history (cleaner queries, more upfront schema) ([git-history](https://simonwillison.net/2021/Dec/7/git-history/)).

## 4. Do PKM patterns combine these?

Yes, and the composition is clean. The recurring recommendation is: frontmatter holds only *current* state (Properties are current-state-only anyway — [obsidian.md/help/properties](https://obsidian.md/help/properties)); a linked event note per transition holds the history as frontmatter-native objects; and a Dataview query over the events folder surfaces "what changed this week." This layers directly on the existing frontmatter-as-database pattern — the event notes are just more pages with YAML properties, indexed the same way ([add-metadata](https://blacksmithgu.github.io/obsidian-dataview/annotation/add-metadata/)) — and git sits underneath as the immutable backstop that survives even if a note or event is later edited or deleted.

## Recommendation

Use all three in layers, matched to how often you query each:

1. **Frontmatter = current state only.** Keep `status:` as the single source for "what is it now." Do not try to cram history into it — Properties are current-state-only ([obsidian.md/help/properties](https://obsidian.md/help/properties)).
2. **Event note per transition = the live, queryable history.** Prefer this over an in-file changelog block, because event-note fields live in frontmatter and are first-class queryable, whereas in-body changelog bullets need bracketed inline fields plus `FLATTEN` to query ([add-metadata](https://blacksmithgu.github.io/obsidian-dataview/annotation/add-metadata/), [metadata-tasks](https://blacksmithgu.github.io/obsidian-dataview/annotation/metadata-tasks/)). A `FROM "events" WHERE date >= …` Dataview query answers "what changed this week" ([data-commands](https://blacksmithgu.github.io/obsidian-dataview/queries/data-commands/)). Capture the *why* in a `reason:` field, since git alone will not.
3. **If you want less file sprawl**, fall back to the in-file changelog block with bracketed inline fields — accept the clunkier query path.
4. **Git = the backstop, not the primary query surface.** `git log -p`/`-L`/`-S` with `--since` reconstructs any past value ([git-log](https://git-scm.com/docs/git-log)), and a `git-history`-style pass can lift it into SQLite if deep analysis is ever needed ([git-history](https://simonwillison.net/2021/Dec/7/git-history/)) — but treat it as audit-of-last-resort, not the day-to-day "what changed" view. Its usefulness scales with commit-message discipline, which the one-commit-per-ingest convention already encourages.

Net: frontmatter for *now*, event notes for *the live history you query*, git for *the immutable record* — no database required.

---

Note on sourcing: the deep-research `web_search` tool returned empty for every query and DuckDuckGo served a CAPTCHA/redirect, so I verified claims by fetching canonical documentation directly (Obsidian help, the Dataview docs site, git-scm, and Simon Willison's blog). All citations above resolve to those primary sources.
