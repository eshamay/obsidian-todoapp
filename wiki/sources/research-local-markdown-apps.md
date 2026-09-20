---
title: "Local-first, markdown-native personal knowledge app architectures (pulled from work-organizer wiki)"
type: source
status: active
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: work-organizer-research-local-markdown-apps
    hash: 5c42382169b0
    ingested: 2026-09-19
aliases: [local-markdown-apps-survey]
tags: [source, obsidian-ecosystem, local-rest-api, dataview, bases, external-prior-art]
---

Prior-art research pulled from the sibling `work-organizer` wiki (dated 2026-09-18, a different project's own build-architecture survey), reused here for its Obsidian-ecosystem content. Surveys local-first markdown app architectures — Obsidian's ecosystem (Local REST API plugin, Dataview, Bases), Logseq, Foam, Dendron, SilverBullet — while evaluating a *different* app's own UI approach.

## Citation

Pulled from `work-organizer/wiki/raw/research-local-markdown-apps.md`, dated 2026-09-18. See `raw/work-organizer--research-local-markdown-apps.md` in this wiki for the full original text and its own cited URLs (Obsidian Local REST API, Dataview, Bases, Logseq, Foam, Dendron, SilverBullet).

## Key claims

- Obsidian's Local REST API plugin exposes full CRUD over vault files via a local HTTP server (`127.0.0.1:27124`) — an out-of-process integration path distinct from writing an in-process plugin against the Plugin API directly.
- Dataview's frontmatter query model turns plain markdown+YAML into a queryable database (live queries rendered inline in notes) without any external server.
- Bases is Obsidian's own native database-view core plugin — table/kanban/cards/map views over note properties, no third-party plugin required.
- The original survey's own conclusion (build a standalone local backend + SPA rather than an Obsidian plugin) is a different app's architecture decision and does not carry over as a recommendation for this project — noted for completeness, not adopted here.

## Relevance to this project

Names three distinct mechanisms for extending/querying an Obsidian vault beyond a straight `Plugin` (in-process): the Local REST API (out-of-process HTTP), Dataview (in-vault query DSL), and Bases (native database views). See [[concepts/obsidian-extension-mechanisms]] for how these compare against this project's own approach (a direct in-process `Plugin`).
