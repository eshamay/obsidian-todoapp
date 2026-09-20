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
aliases: [vault-api, vault-adapter-vs-vault-methods]
tags: [concept, obsidian-api, vault, file-io]
---

Obsidian's `Vault` class offers two layers for touching files: high-level methods (`read()`/`cachedRead()`, `create()`/`modify()`/`process()`, `createBinary()`/`modifyBinary()`, `append()`/`appendBinary()`, `rename()`/`copy()`/`delete()`) that work against `TFile`/`TFolder` objects and participate in Obsidian's normal file-change eventing (`create`/`modify`/`delete`/`rename` events on `Vault`, which extends `Events`); and the lower-level `adapter` property, which does raw path-based reads/writes without going through the `TFile` abstraction.

`process()` is notable: it combines an atomic read-modify-save into one call, avoiding a separate read-then-write race window.

> [!NOTE]
> **This project's own `TaskNoteModal`/`TodoStore` (`plugin/src/main.tsx`) currently uses `app.vault.adapter.read(...)`/`.write(...)` directly** for note-file content, bypassing the higher-level `Vault` methods entirely. This is a real, actionable finding — not adopted or changed here, just recorded as a candidate improvement: moving to `vault.getAbstractFileByPath()` + `vault.read()`/`vault.process()` would let normal Vault file-change events fire (useful if e.g. Obsidian Sync or other plugins want to observe note-file changes) and would use the documented instanceof-`TFile` disambiguation pattern already used elsewhere in this project's own embedded-editor code (see [[examples/embed-markdownview-in-modal]], which does use `getAbstractFileByPath` + `instanceof TFile` for the *opening* leaf, just not for the fallback textarea's raw read/write).

Source: [[sources/web-obsidian-plugin-api-overview]] (official docs.obsidian.md).
