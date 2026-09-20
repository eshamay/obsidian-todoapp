> [!WARNING]
> Edit only by the user. Agents may propose edits for review but must not write this file directly.

Last human edit: 2026-09-19

# Purpose

## Scope

Research Obsidian's plugin API and developer tooling, and task-tracking-app / task-data-model prior art, to support building a custom Obsidian todo-list plugin (`todoapp-blocks-plugin`, forked from kaiso12/todoapp). Findings feed the plugin's own UI/UX and entity-model work (in the parent project, not in this wiki).

In scope:
- Obsidian Plugin API surfaces relevant to this plugin: Workspace/Leaf/View model, Modal, MarkdownView/MarkdownRenderer, Editor, Vault, Commands, Settings, mobile compatibility.
- Obsidian developer tooling: sample-plugin scaffold, esbuild build pipeline, typings/versioning, community-plugin submission/review process.
- Task-tracking-app research: how mainstream tools (Todoist, TickTick, Things, GitHub/Jira/Linear/Trello, Obsidian's own Tasks/Dataview/Kanban plugins) model task data, status, and relations.
- Prior art pulled from other research already done (e.g. the `work-organizer` wiki's task/status-data-model and Obsidian-ecosystem research) and from this project's own hands-on findings (e.g. the `WorkspaceLeaf` embedding technique used to fix the task-note editor).

## Audience

Primarily the user (sole author/consumer). Secondary: future agent sessions working on `todoapp-blocks-plugin`, who should read this wiki before proposing UI/UX or entity-model changes.

## Non-goals

- This wiki is not the plugin itself, and not its decision log — plugin design decisions live in the project's own docs/commits, not here.
- Not a general PKM-methodology wiki — that ground is already covered by the sibling `work-organizer` wiki for a different project; only the Obsidian-API and task-data-model slice of that prior work is pulled in here.
- Not evaluating cloud/team-shared project-management tools as candidates to adopt — the target is this plugin's own design, not a build-vs-buy decision.
