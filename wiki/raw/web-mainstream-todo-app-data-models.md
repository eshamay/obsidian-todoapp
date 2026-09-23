> Compiled: 2026-09-19<br>
> Scope: public API / data-model documentation for mainstream consumer todo apps and Obsidian task-related community plugins<br>
> Purpose: fill the gap left by prior research on Jira/Linear/GitHub/Trello (project-management tools) and one custom entity model — none of that covered consumer todo apps

This note surveys how six consumer-facing task tools and Obsidian plugins publicly document their task data model: Todoist's REST API, TickTick's developer portal, Things' URL scheme, and the Obsidian Tasks / Dataview / Kanban community plugins. Each section lists sourced factual claims, then a short comparison against this project's own `Task` type (`title`, `projectId`, `due`, `priority` 1-4, `completed` boolean, `notePath`).

## Todoist API (developer.todoist.com/api/v1)

- The Task object exposes `content` (task title text) and `description` (additional detail text). — https://developer.todoist.com/api/v1/
- `priority` is an integer where "1 is no priority and 4 is high priority" (note: Todoist's UI inverts this — UI "Priority 1" maps to API value 4). — https://developer.todoist.com/api/v1/
- `labels` is an array of label names/IDs for categorization; `project_id` identifies the owning project; `section_id` identifies an optional section within a project; `parent_id` references the parent task for subtasks. — https://developer.todoist.com/api/v1/
- The `due` object's documented fields are: `string` (human-readable text, e.g. "every day"), `date` (date-only, full-day tasks), `datetime` (date+time, floating or timezone-bound), `timezone` (present only when the due time is timezone-fixed), and `is_recurring` (boolean). Recurrence is carried by the natural-language `string` field plus `is_recurring`, not a separate RRULE field — the docs' "Due dates" guide shows full-day, floating-time, and fixed-timezone variants using exactly this field set with no distinct recurrence-rule property. — https://developer.todoist.com/api/v1/#section/Due-dates
- Todoist has a **separate `deadline` object**, distinct from `due`, with its own fields `date` and `lang` (deadline is date-only, `lang` controls natural-language parsing) — a "when is this due at the latest" concept independent of the scheduled/recurring `due` date. — https://developer.todoist.com/api/v1/#section/Due-dates
- `duration` and `duration_unit` (`"minute"` or `"hour"`) express an estimated time-on-task, independent of due/deadline. — https://developer.todoist.com/api/v1/

**Comparable vs. new fields:** `content`→`title`, `project_id`→`projectId`, `due.date`/`due.datetime`→`due`, `priority` 1-4 maps directly (mind the inversion) onto this project's `priority` 1-4, and task completion state is comparable to `completed`. New concepts not in this project's model: `section_id` (sub-project grouping), `parent_id` (subtasks/hierarchy), `labels` (multi-value tagging distinct from project), `duration`/`duration_unit` (time estimation), and a **second, independent deadline concept** separate from the scheduled due date.

## TickTick (developer.ticktick.com)

- TickTick operates a branded developer portal at `developer.ticktick.com` (page title "TickTick Developer") with a docs path at `developer.ticktick.com/docs/index.html` that contains an `#/openapi` route, implying an OAuth-based "Open API" product exists.
- However, this entire portal — the root, `/docs/index.html`, and the `#/openapi` route — is a client-side-rendered single-page app (webpack/React bundle) with no server-rendered content. Repeated fetch attempts (WebFetch tool, direct `curl`, and a headless-browser tool that errored out) each returned only the empty app shell (`<div id="root"></div>` plus JS bundle references), never the rendered documentation body. — https://developer.ticktick.com/docs/index.html (fetched 2026-09-19, returned empty SPA shell in all attempts)
- **Explicitly: no field-level task schema could be independently verified through publicly accessible, non-JS-executing tooling in this research pass.** Any claim about specific TickTick task field names (e.g., `dueDate`, `priority` value ranges, `repeatFlag`) would be unverified guessing and is deliberately omitted here rather than fabricated.

**Comparable vs. new fields:** cannot be assessed — no verifiable public schema was retrievable in this pass. This is a documented gap, not a comparison.

## Things (Cultured Code) — URL scheme

- The Things URL scheme documents these to-do fields: `title` ("The title of the to-do to add"), `notes` ("The text to use for the notes field of the to-do"), `when` (scheduling — accepts values like `today`, `tomorrow`, `evening`, `anytime`, `someday`, or a specific date), `deadline` ("The deadline to apply to the to-do"), `tags` (comma-separated tag titles), `checklist-items` (up to 100 sub-items per to-do), `list-id`/`list` (the target project or area, by ID or title), `heading` (a heading within a project to file the to-do under), `completed` (boolean completion flag), and `canceled` (boolean cancellation flag, a status distinct from completion). — https://culturedcode.com/things/support/articles/2803573/

**Comparable vs. new fields:** `title`→`title`, `list-id`/`list`→`projectId`, `when`/`deadline`→`due`, `completed`→`completed`. New concepts: Things separates **scheduling (`when`) from a hard `deadline`** (two distinct date concepts, similar in spirit to Todoist's due/deadline split); **`tags`** as a first-class multi-value field; **`checklist-items`** as inline sub-items distinct from subtasks; **`heading`** as a sub-grouping inside a project (comparable to Todoist's `section_id`); and a three-state-like status (`completed` vs `canceled`) rather than a single boolean.

## Obsidian Tasks plugin (publish.obsidian.md/tasks)

- The Tasks plugin's "Emoji Format" encodes fields as trailing emoji + value pairs appended to a standard markdown checkbox line (`- [ ] ...`). Documented date-field emoji: created date ➕, scheduled date ⏳, start date 🛫, due date 📅, done date ✅, cancelled date ❌. — https://publish.obsidian.md/tasks/Reference/Task+Formats/Tasks+Emoji+Format
- Priority is encoded as one of six levels, each with its own emoji: lowest ⏬, low 🔽, normal (no emoji — the implicit default), medium 🔼, high ⏫, highest 🔺. This is a 6-level scheme (5 explicit + 1 implicit default), not a numeric 1-4 range. — https://publish.obsidian.md/tasks/Reference/Task+Formats/Tasks+Emoji+Format
- Recurrence is encoded with 🔁 followed by a natural-language rule (e.g. "every week"). Other documented fields: 🏁 "on completion" (what happens to the recurrence template when the task is completed), 🆔 a task ID, and ⛔ a dependency ("depends on") reference to another task's ID — giving Tasks a general task-dependency graph, not just parent/subtask nesting. — https://publish.obsidian.md/tasks/Reference/Task+Formats/Tasks+Emoji+Format
- Tags are supported via inline `#tag` markdown syntax elsewhere in the task line (not part of the emoji date/field set documented on this specific page).

**Comparable vs. new fields:** due date 📅 → `due`; the checkbox `- [ ]`/`- [x]` state → `completed`; project/notePath is implicit (the task's containing file *is* the notePath in this model, with no separate project concept). New concepts: a **6-level emoji priority scale** (vs. this project's numeric 1-4); **five distinct date types** (created/scheduled/start/due/done, plus cancelled) where this project has only `due`; **recurrence** as a natural-language rule string; and an explicit **`id`/`dependsOn` dependency graph** between tasks, which is a materially different relationship model than a single `parent_id`.

## Obsidian Dataview (blacksmithgu.github.io/obsidian-dataview)

- Dataview treats markdown task checkboxes (`- [ ]` / `- [x]`) as first-class, independently queryable objects (via `TASK` queries), separate from but related to page-level frontmatter metadata (queried via `TABLE`/`LIST` on pages). — https://blacksmithgu.github.io/obsidian-dataview/annotation/metadata-tasks/
- Documented implicit task fields include: `text`/`visual` (task content), `line`/`lineCount` (position in file), `status`, `checked`, `completed`, `fullyCompleted` (completion states at various granularities, e.g. accounting for nested subtasks), `due`, `created`, `start`, `scheduled` (dates parsed from emoji shorthand, i.e. interoperable with the Tasks plugin's syntax), `tags`, `outlinks`, `children` (nested subtasks/list items), `parent` (reference to the parent task), `task` (a boolean discriminator marking the row as a task vs. a plain list item), and `blockId`. — https://blacksmithgu.github.io/obsidian-dataview/annotation/metadata-tasks/
- Critically: **"Tasks inherit all fields from their parent page"** — so page-level frontmatter (e.g. a custom `rating` field) is directly accessible inside a task query, meaning Dataview treats tasks and pages as one connected metadata graph rather than two disjoint models. — https://blacksmithgu.github.io/obsidian-dataview/annotation/metadata-tasks/

**Comparable vs. new fields:** `due`→`due`; `completed`/`checked`/`fullyCompleted`→`completed` (though Dataview's three-way split of completion granularity has no equivalent here); the task's source file is implicitly `notePath`. New concepts: `children`/`parent` as a **generic list-nesting hierarchy** (broader than subtasks — any nested bullet, not just tasks); frontmatter-field **inheritance from the containing page** into every task on that page (no analogous mechanism in this project's flat `Task` type); and treating "task-ness" itself (`task: true/false`) as a discriminator on what is otherwise a generic list-item type.

## Obsidian Kanban plugin (github.com/mgmeyers/obsidian-kanban)

- A Kanban board is a plain markdown note marked by YAML frontmatter `kanban-plugin: board` (the constant `frontmatterKey = 'kanban-plugin'`; `basicFrontmatter` scaffolds exactly `kanban-plugin: board`). — https://github.com/mgmeyers/obsidian-kanban/blob/main/src/parsers/common.ts
- Lanes (columns) are plain markdown level-2 headings: the board serializer emits `## <lane title>` per lane (`lines.push(`## ${laneTitleWithMaxItems(lane.data.title, lane.data.maxItems)}`)`). — https://github.com/mgmeyers/obsidian-kanban/blob/main/src/parsers/formats/list.ts
- Cards are standard markdown checkbox list items nested under a lane heading: the serializer emits `- [${item.data.checkChar}] <content>` — i.e. a normal `- [ ]`/`- [x]` task line, so a card's "done" lane/state is literally the checkbox character plus which lane heading it sits under. — https://github.com/mgmeyers/obsidian-kanban/blob/main/src/parsers/formats/list.ts
- An archive of removed cards is appended at the end of the file behind a lane titled `## Archive`, preceded by a horizontal-rule-like marker constant `archiveString = '***'`. — https://github.com/mgmeyers/obsidian-kanban/blob/main/src/parsers/common.ts and https://github.com/mgmeyers/obsidian-kanban/blob/main/src/parsers/formats/list.ts
- Board-level settings (lane width, sort options, date-trigger config, etc.) are serialized as JSON inside an Obsidian comment block: `%% kanban:settings\n```\n<json>\n```\n%%` appended to the file — configuration lives in-band in the same markdown file, not in a separate config file. — https://github.com/mgmeyers/obsidian-kanban/blob/main/src/parsers/common.ts
- Per-card dates are added via a "date trigger" typed inline in the card text, which opens a calendar picker and inserts a date string into the card's markdown content (documented at a how-to level, not as a discrete schema field). — https://publish.obsidian.md/kanban/How+do+I/Add+a+date+to+a+card.md

**Comparable vs. new fields:** the checkbox state on a card line → `completed`; the card's containing note → `notePath`; a card's lane could loosely map to a coarse status/priority bucket rather than the project concept. New concepts: **lanes as a markdown-heading-encoded status/column dimension** (not present in this project's model at all — there is no swimlane/status-column concept); an **in-band JSON settings blob** co-located with content rather than separate metadata; and an explicit **archive section** as a soft-delete/history mechanism distinct from a `completed` flag.

## Sources

- https://developer.todoist.com/api/v1/
- https://developer.todoist.com/api/v1/#section/Due-dates
- https://developer.ticktick.com/
- https://developer.ticktick.com/docs/index.html
- https://culturedcode.com/things/support/articles/2803573/
- https://publish.obsidian.md/tasks/Reference/Task+Formats/Tasks+Emoji+Format
- https://blacksmithgu.github.io/obsidian-dataview/annotation/metadata-tasks/
- https://github.com/mgmeyers/obsidian-kanban
- https://github.com/mgmeyers/obsidian-kanban/blob/main/src/parsers/common.ts
- https://github.com/mgmeyers/obsidian-kanban/blob/main/src/parsers/formats/list.ts
- https://publish.obsidian.md/kanban/How+do+I/Add+a+date+to+a+card.md
