---
title: "Feature-probe + fallback pattern for undocumented API surfaces"
type: concept
status: draft
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: obsidian-embedded-editor-research
    hash: 4659e92f37c8
    ingested: 2026-09-19
aliases: [feature-probe-fallback, graceful-degradation-undocumented-api]
tags: [concept, obsidian-api, plugin-dev, risk-management, fallback]
---

When a plugin technique depends on non-public/undocumented parts of a host API (a property or method absent from the official typed surface, or a CSS class name with no versioned contract), the technique should never be load-bearing on its own — it should degrade cleanly if the host changes.

Pattern, as applied to the [[workspace-leaf-view-model|WorkspaceLeaf embedding technique]]:

1. **Feature probe first.** Check that the public entry point the technique needs actually exists and is callable (e.g. `typeof workspace.createLeafInParent === "function"`) before touching anything. Cheap insurance against a future API removal, even for an old/stable method.
2. **Isolate the one undocumented access behind a single helper.** Don't scatter unchecked casts through the codebase — one small typed helper function (e.g. `(leaf as unknown as { containerEl?: HTMLElement }).containerEl`) makes the actual risk surface auditable at a glance.
3. **Wrap the whole mount/setup sequence in try/catch.** Any failure — the undocumented property missing, an assumed view type not materializing, a DOM operation throwing — should be caught, any partially-created state cleaned up (e.g. `leaf?.detach()`), and control handed to a known-good fallback path.
4. **The fallback must be the previously-working behavior**, not a crash and not a half-broken UI. In the concrete case this pattern was extracted from, the fallback was reverting to a plain `<textarea>` — strictly worse UX than the embedded editor, but never worse than what existed before the enhancement was added.

This is a general risk-management pattern for any plugin/integration code that reaches past a host's officially documented surface for genuine functional gain — not specific to Obsidian, though [[sources/obsidian-embedded-editor-research]] is where it was first captured in this wiki.
