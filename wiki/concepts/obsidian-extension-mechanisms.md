---
title: "Obsidian extension mechanisms: Plugin API vs. Local REST API vs. Dataview/Bases"
type: concept
status: draft
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: work-organizer-research-local-markdown-apps
    hash: 5c42382169b0
    ingested: 2026-09-19
aliases: [obsidian-extension-points, local-rest-api-vs-plugin-api]
tags: [concept, obsidian-ecosystem, plugin-api, local-rest-api, dataview, bases]
---

Four distinct ways to build custom behavior on top of an Obsidian vault, not mutually exclusive:

1. **In-process `Plugin` (Plugin API)** — this project's approach. Full access to `Workspace`/`Vault`/`Editor`/`Modal`/etc., runs inside Obsidian itself, ships as a community plugin. See [[workspace-leaf-view-model]] for the specific Workspace/Leaf/View internals this project already relies on.
2. **Local REST API plugin** — a community plugin (`coddingtonbear/obsidian-local-rest-api`) that turns a *running* Obsidian instance into a local HTTP server (`127.0.0.1:27124`) exposing CRUD over vault files. Out-of-process: any external tool/script/webapp can read/write the vault without being an Obsidian plugin itself, at the cost of requiring Obsidian to actually be running.
3. **Dataview** — an in-vault query DSL/plugin. Frontmatter properties become queryable; queries render live, inline, inside notes. No external process, no custom UI beyond what Dataview's query blocks render.
4. **Bases** — Obsidian's own native (non-third-party) database-view core plugin. Table/kanban/cards/map views over note properties, built into Obsidian itself.

For a plugin like this one that wants custom interactive UI (task lists, modals, inline editing) rather than just queryable views, option 1 (Plugin API) is the only mechanism that gives full UI control — options 3–4 render views but don't provide arbitrary custom interaction, and option 2 requires a separate always-on process and a UI built outside Obsidian entirely.

Source: [[sources/research-local-markdown-apps]] (external prior art, pulled from a different project's own build-architecture survey — that survey's conclusion favored a standalone backend for *their* use case, not adopted here).
