# TodoApp Blocks Plugin (fork)

## Origin

Fork/derivative of **TodoApp Blocks** by Max Dvorkin — an Obsidian plugin.
Upstream: https://github.com/kaiso12/todoapp

## Goal

Build on top of upstream plugin for personal use. Not aiming for upstream PR compat — free to diverge.

Two work streams:
1. **UI/UX tweaks** — improve usability of existing views/rendering/interactions.
2. **Entity model expansion** — extend how tasks are captured/represented (fields, relations, metadata beyond upstream's model).

## Status

Upstream vendored into `plugin/` (upstream's own `.git` stripped — plain snapshot, not a submodule/subtree). All work happens inside `plugin/`.

Next steps:
- Get local Obsidian dev build loop working (`plugin/`: `npm install`, esbuild via `esbuild.config.mjs`).
- Inventory upstream's current task entity model + UI components (`plugin/src/`) before changing.

## Test vault

Manual testing happens in a real Obsidian vault: `~/Desktop/Obsidian/Work Organizer/`. That vault already has TodoApp Blocks enabled (`.obsidian/community-plugins.json` lists `"todoapp"`) and has real task data at vault root (`.todoapp/life.json`, `TodoApp Notes/life/*.md` — unrelated to and untouched by anything below).

`~/Desktop/Obsidian/Work Organizer/.obsidian/plugins/todoapp` is a **symlink** to this repo's `plugin/` directory (the original real folder was preserved as `todoapp.bak-20260919` alongside it, same layout the sibling `work-organizer-command-center` plugin in that vault already uses). `plugin/` has `main.js`/`manifest.json`/`styles.css` at its root, same layout Obsidian expects, so no copy step is needed.

Test loop:
1. Edit `plugin/src/main.tsx` (or other source).
2. `npm run build` inside `plugin/`.
3. In Obsidian (vault open): reload the app, or disable/re-enable the TodoApp Blocks plugin, to pick up the new `main.js`.
4. Test in the vault directly — existing "life" project/tasks are real data, not throwaway fixtures.

## Remote

Personal repo, for tracking this Obsidian plugin fork: `git@github.com:eshamay/obsidian-todoapp.git` (remote `origin`, branch `main`).

## Conventions

- No PR ever planned back to upstream (kaiso12/todoapp) — free to diverge fully, no compat constraint.
- Commit only when explicitly asked.
- Push only when explicitly asked.
