---
title: "Obsidian community-plugin guidelines checklist"
type: concept
status: draft
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: web-obsidian-dev-tooling-and-release-process
    hash: 2ee40b4c74b6
    ingested: 2026-09-19
aliases: [plugin-guidelines-checklist, obsidian-code-review-rules]
tags: [concept, obsidian-api, plugin-dev, code-quality]
---

Official docs.obsidian.md guidelines for community plugins — useful as a general code-quality checklist regardless of whether a plugin is actually submitted to the community directory (this project isn't — see this project's own `CLAUDE.md`, "no PR ever planned back to upstream," and no community-listing plan either):

- Use `this.app` (the plugin-instance reference), never the global `app`/`window.app`.
- Avoid `innerHTML`/`outerHTML`/`insertAdjacentHTML` for user-controlled content; use `createEl()`/`createDiv()`/`createSpan()` DOM-construction helpers instead.
- Release resources (event listeners, etc.) on unload; prefer `registerEvent()`/`addCommand()` for auto-cleanup over manual bookkeeping. Don't detach *other* leaves in `Plugin.onunload()` — that's about not disturbing the user's workspace layout when a plugin is disabled, distinct from a `Modal` cleaning up a leaf it itself created (see [[sources/obsidian-embedded-editor-research]]'s `TaskNoteModal.onClose()`, which is the correct place for that).
- UI text: Sentence case, not Title Case; avoid the word "settings" in settings headings; use `setHeading()` instead of raw heading elements.
- Commands: avoid default hotkeys (conflict risk); use the correct callback variant for the situation.
- Data access: prefer `Editor` API over `Vault.modify()` for the active file; `Vault.process()` for background edits; `FileManager.processFrontMatter()` for frontmatter edits (see [[concepts/obsidian-vault-file-io]]); `normalizePath()` on user-supplied paths; avoid iterating all vault files, use targeted lookups.
- No hardcoded inline styles — CSS classes + Obsidian's CSS variables (this project's own eslint config already enforces this via `obsidianmd/no-static-styles-assignment`, encountered directly while building the embedded editor — see [[examples/embed-markdownview-in-modal]]).
- TypeScript style: `const`/`let` over `var`; async/await over raw Promise chains.

Not verified: explicit telemetry-disclosure or code-obfuscation rejection criteria — the pages that would plausibly document these (`Developer+policies`, `Submission+requirements+for+plugins`, `Plugin+review+guidelines`) all 404'd during this research session. Not asserted here without a working citation.

Source: [[sources/web-obsidian-dev-tooling-and-release-process]].
