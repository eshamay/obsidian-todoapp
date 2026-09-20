---
title: "Obsidian UI building blocks: Modal, Setting, Commands, PluginSettingTab"
type: concept
status: draft
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: web-obsidian-plugin-api-overview
    hash: 85e25066b12f
    ingested: 2026-09-19
aliases: [obsidian-ui-components, plugin-settings-tab]
tags: [concept, obsidian-api, modal, setting, commands, settings-tab]
---

The core UI-construction surfaces a plugin has beyond raw DOM manipulation:

- **`Modal`** — `contentEl`/`titleEl`/`modalEl`/`containerEl`, `onOpen()`/`onClose()` lifecycle, `open()`/`close()`. This project's `TaskNoteModal` (`plugin/src/main.tsx`) is the only `Modal` subclass in the codebase; see [[sources/obsidian-embedded-editor-research]] for how it goes beyond the basic contract to embed a real `MarkdownView`.
- **`Setting`** — a fluent builder for one labeled row in a settings tab or modal: `setName()`/`setDesc()`/`setHeading()`, chainable `add*` methods (`addText`, `addToggle`, `addDropdown`, `addButton`, etc.), each returning the `Setting` instance.
- **Commands (`addCommand`)** — registers an entry in the Command Palette. `callback()` for always-available actions; `checkCallback()` for conditional ones (called twice: once to check, once to execute — the check must be re-evaluated both times since state can change between calls); `editorCallback()`/`editorCheckCallback()` for editor-context-only commands. Default hotkeys are discouraged for plugins meant for other users, due to conflict risk.
- **`PluginSettingTab`** — two approaches: legacy imperative `display()` (manually construct `Setting` rows, clean up in `hide()`) vs. a newer (Obsidian 1.13.0+) declarative `getSettingDefinitions()` where each definition's `key` maps to a property on the plugin's settings object and Obsidian handles read/write/`saveData()` automatically.

`plugin/src/settings.ts` in this project is currently an empty stub ("Settings UI is not implemented yet") — direct relevance if/when this project builds a settings tab; the declarative `getSettingDefinitions()` approach would be less boilerplate than the legacy `display()` pattern, given this project's `obsidian` package is pinned to 1.13.0 (already meets the version bar).

Source: [[sources/web-obsidian-plugin-api-overview]] (official docs.obsidian.md).
