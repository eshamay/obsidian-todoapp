---
title: "Local backend vs. Obsidian plugin — architecture decision (pulled from work-organizer wiki)"
type: source
status: active
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: work-organizer-local-backend-vs-obsidian-plugin
    hash: 6faa3913fe18
    ingested: 2026-09-19
aliases: [local-backend-vs-plugin-decision]
tags: [source, obsidian-plugin-api, architecture, external-prior-art]
---

The most Obsidian-plugin-API-relevant page found in the sibling `work-organizer` wiki (a different project's own architecture decision, dated 2026-09-18). Decided to build that project's command-center as a real Obsidian plugin (Phase 1), using Obsidian's Plugin API directly (including Dataview/Bases where they fit), with a standalone backend (Phase 2) contingent on future feature needs rather than a fixed timeline.

## Citation

Pulled from `work-organizer/wiki/decisions/local-backend-vs-obsidian-plugin.md`, dated 2026-09-18. See `raw/work-organizer--local-backend-vs-obsidian-plugin.md` in this wiki for the full original text.

## Key claims

- Building directly against Obsidian's Plugin API (not just the Local REST API hedge — see [[concepts/obsidian-extension-mechanisms]]) was the chosen approach for that project's own v1, specifically to get custom objects/views/UI elements beyond what Dataview/Bases render natively.
- The Local REST API plugin was named as a possible prototyping shortcut/hedge (validate a data/view model quickly against a running Obsidian instance) rather than the production mechanism.
- The phased-build framing (plugin first, standalone backend later, contingent not scheduled) is itself a reusable decision *pattern* independent of which specific app is being built — worth noting as prior art even though the decision itself was made for a different app.

## Relevance to this project

`todoapp-blocks-plugin` is already committed to the Plugin API path (it's a fork of an existing Obsidian plugin, not a build-vs-buy decision in progress) — so this source's conclusion doesn't change this project's direction, but its named tradeoffs (custom UI control vs. Dataview/Bases' native-but-limited views) corroborate [[concepts/obsidian-extension-mechanisms]]'s claim that only the Plugin API gives full custom-UI control.
