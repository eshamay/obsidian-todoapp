---
title: "Obsidian Vault API: high-level file I/O vs. raw adapter access"
type: concept
status: draft
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: web-obsidian-plugin-api-overview
    hash: 85e25066b12f
    ingested: 2026-09-19
  - id: web-obsidian-dev-tooling-and-release-process
    hash: 2ee40b4c74b6
    ingested: 2026-09-19
aliases: [vault-api, vault-adapter-vs-vault-methods]
tags: [concept, obsidian-api, vault, file-io]
---

Obsidian's `Vault` class offers two layers for touching files: high-level methods (`read()`/`cachedRead()`, `create()`/`modify()`/`process()`, `createBinary()`/`modifyBinary()`, `append()`/`appendBinary()`, `rename()`/`copy()`/`delete()`) that work against `TFile`/`TFolder` objects and participate in Obsidian's normal file-change eventing (`create`/`modify`/`delete`/`rename` events on `Vault`, which extends `Events`); and the lower-level `adapter` property, which does raw path-based reads/writes without going through the `TFile` abstraction.

`process()` is notable: it combines an atomic read-modify-save into one call, avoiding a separate read-then-write race window.

> [!NOTE]
> **This project's own `TaskNoteModal`/`TodoStore` (`plugin/src/main.tsx`) currently uses `app.vault.adapter.read(...)`/`.write(...)` directly** for note-file content, bypassing the higher-level `Vault` methods entirely. This is a real, actionable finding — not adopted or changed here, just recorded as a candidate improvement: moving to `vault.getAbstractFileByPath()` + `vault.read()`/`vault.process()` would let normal Vault file-change events fire (useful if e.g. Obsidian Sync or other plugins want to observe note-file changes) and would use the documented instanceof-`TFile` disambiguation pattern already used elsewhere in this project's own embedded-editor code (see [[examples/embed-markdownview-in-modal]], which does use `getAbstractFileByPath` + `instanceof TFile` for the *opening* leaf, just not for the fallback textarea's raw read/write).

Official Plugin guidelines ([[sources/web-obsidian-dev-tooling-and-release-process]]) go further and name the specific preferred APIs: **prefer the `Editor` API over `Vault.modify()` for the active file; use `Vault.process()` for background edits; and use `FileManager.processFrontMatter()` specifically for YAML frontmatter edits** (rather than string-templating the whole file).

> [!NOTE]
> **Second actionable finding:** this project's `makeNoteFileContent()`/`stripTodoAppNoteMeta()` (`plugin/src/main.tsx`) hand-roll frontmatter generation (a JS template literal building `---\nkey: value\n---`) and stripping (a regex, `text.replace(/^---[\s\S]*?---\s*/, "")`) — the documented `FileManager.processFrontMatter(file, fn)` API exists specifically to read/mutate a file's frontmatter object safely (parsed, not string-matched) without touching the body. Neither finding has been acted on — both are candidate improvements for this project's own code, recorded here as prior art, not a decision.

Sources: [[sources/web-obsidian-plugin-api-overview]], [[sources/web-obsidian-dev-tooling-and-release-process]] (official docs.obsidian.md / obsidianmd GitHub orgs).
