---
title: "Embedding a live MarkdownView inside an Obsidian Modal"
type: example
status: active
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: obsidian-embedded-editor-research
    hash: 4659e92f37c8
    ingested: 2026-09-19
aliases: [embed-markdownview-example, workspaceleaf-modal-code]
tags: [example, obsidian-api, workspaceleaf, markdownview, modal]
---

Worked example of the technique described in [[concepts/workspace-leaf-view-model]], as applied in this project's `TaskNoteModal` (`plugin/src/main.tsx`) and verified against `chrisgurney/obsidian-note-toolbar`'s `NtbModal` and `likemuuxi/obsidian-modal-opener`'s `ModalWindow`.

## Mount sequence (open)

```ts
const workspace = this.app.workspace;
if (typeof workspace.createLeafInParent !== "function" || !workspace.rootSplit) {
  return false; // feature probe failed — caller falls back to plain textarea
}

const file = this.app.vault.getAbstractFileByPath(this.notePath);
if (!(file instanceof TFile)) return false;

let leaf: WorkspaceLeaf | undefined;
try {
  this.prevActiveLeaf = workspace.getMostRecentLeaf();
  leaf = workspace.createLeafInParent(workspace.rootSplit, 0);

  const wrapperEl = leafContainerEl(leaf); // isolates the one undocumented access
  if (!wrapperEl) throw new Error("leaf.containerEl unavailable");
  wrapperEl.addClass("hidden-leaf"); // never use element.style.display directly — eslint-plugin-obsidianmd flags it

  await leaf.openFile(file, { state: { mode: "source" }, active: true });
  if (!(leaf.view instanceof MarkdownView)) throw new Error("leaf.view is not a MarkdownView");

  const host = contentEl.createDiv({ cls: "embedded-leaf-host" });
  host.appendChild(leaf.view.containerEl); // View.containerEl — public. NOT leaf.containerEl.
  this.leaf = leaf;
  return true;
} catch (e) {
  console.error("embedded editor unavailable, falling back", e);
  leaf?.detach();
  return false;
}
```

## Cleanup sequence (close)

```ts
async onClose() {
  if (this.leaf) {
    this.leaf.detach(); // flushes any pending autosave (TextFileView.requestSave)
    this.leaf = undefined;
    if (this.prevActiveLeaf) {
      this.app.workspace.setActiveLeaf(this.prevActiveLeaf, { focus: false }); // restore whatever tab was active before the embed
    }
  }
  this.contentEl.empty();
}
```

## CSS to hide redundant native chrome

```css
.embedded-leaf-host .view-header,
.embedded-leaf-host .metadata-container {
  display: none;
}
```

## Notes

- `leafContainerEl()` helper: `(leaf as unknown as { containerEl?: HTMLElement }).containerEl` — the single contained cast for the one non-public property this technique needs. See [[undocumented-api-fallback-pattern]].
- No manual Save button — `MarkdownView`'s autosave (`TextFileView.requestSave`) plus the flush-on-detach in `onClose()` is sufficient, same guarantee any real Obsidian tab close relies on.
- Full applied version: `plugin/src/main.tsx` — `TaskNoteModal.tryOpenEmbeddedEditor()` / `renderFallbackTextarea()` / `onClose()`.
