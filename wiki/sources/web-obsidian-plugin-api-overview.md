---
title: "Official Obsidian Plugin API overview (docs.obsidian.md)"
type: source
status: active
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: web-obsidian-plugin-api-overview
    hash: 85e25066b12f
    ingested: 2026-09-19
aliases: [obsidian-plugin-api-docs]
tags: [source, obsidian-api, plugin-dev, official-docs]
---

Web research pulled directly from the official Obsidian Developer Documentation (docs.obsidian.md), covering the Plugin API's core surfaces: plugin lifecycle/manifest, `Workspace`, `Vault`, `Editor`, `Modal`/`Setting`, `MarkdownView`/`MarkdownRenderer`, Commands, `PluginSettingTab`, event/interval cleanup, mobile compatibility, and release/versioning.

## Citation

16 pages fetched from docs.obsidian.md, full list in `raw/web-obsidian-plugin-api-overview.md`. Primary pages: Plugins/Getting+started/{Anatomy+of+a+plugin, Build+a+plugin, Mobile+development}; Reference/Manifest; Reference/TypeScript+API/{Workspace, Vault, Editor, Modal, Setting, MarkdownView, MarkdownRenderer}; Plugins/User+interface/{Commands, Settings}; Plugins/Events; Plugins/Releasing/Submit+your+plugin; github.com/obsidianmd/obsidian-releases (for `versions.json`, since docs.obsidian.md's own "Plugin versions" page 404s — a documentation gap on Obsidian's own site).

## Key claims

- `manifest.json` required fields: `author`, `minAppVersion`, `name`, `version` (all manifests) plus `description`, `id` (plugins only — `id` must be lowercase+hyphens, can't end in "plugin" or contain "obsidian"). `isDesktopOnly` flags Node/Electron-dependent plugins.
- `versions.json` is the compatibility resolver: if a user's Obsidian is older than `manifest.json`'s `minAppVersion`, Obsidian consults `versions.json` to install the latest plugin version still compatible with that older app build, rather than blocking install entirely.
- **`Vault` has higher-level file methods (`read`/`modify`/`create`/`process`) distinct from the lower-level `adapter` property** — `process()` combines an atomic read-modify-save in one call. This project's own `TaskNoteModal`/`TodoStore` (`plugin/src/main.tsx`) currently reads/writes note files via `app.vault.adapter.read`/`.write` directly, bypassing these higher-level methods — see note on [[concepts/obsidian-vault-file-io]].
- `Workspace.getLeaf()`, `.createLeafBySplit()`, `.getLeavesOfType()`, `.rootSplit`/`.leftSplit`/`.rightSplit` are all official, documented API — corroborates and extends what [[concepts/workspace-leaf-view-model]] already captured from community-plugin code alone.
- `Editor.transaction(tx, origin)` (since v0.13.0) groups multiple edits into one atomic `EditorTransaction`.
- `MarkdownRenderer.render(app, markdown, el, sourcePath, component)` is the documented way to render markdown to HTML without a live editor — the read-only-preview alternative to this project's chosen embedded-live-editor approach (see [[sources/obsidian-embedded-editor-research]]).
- `registerEvent()`/`registerInterval()` auto-detach on plugin unload — the documented cleanup pattern this project's own `TaskNoteModal.onClose()` (leaf detach + active-leaf restore) already follows in spirit for its own resource (the embedded leaf).
- `PluginSettingTab` has two approaches: legacy imperative `display()` (manual `Setting` row construction + `hide()` cleanup) and a newer (1.13.0+) declarative `getSettingDefinitions()`. This project's `plugin/src/settings.ts` is currently an empty stub — direct relevance if settings UI gets built.
- Mobile: `Platform.isIosApp`/`Platform.isAndroidApp` for conditional code paths; Node/Electron APIs are unavailable on mobile; lookbehind regex only works on iOS 16.4+. Relevant to [[open-knowledge-gaps#gap-001]] (embedded-editor technique unverified on mobile).
