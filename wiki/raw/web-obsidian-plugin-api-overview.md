# Obsidian Plugin API Overview (docs.obsidian.md)

Summary of the official Obsidian Developer Documentation covering the Plugin API: plugin lifecycle and manifest, the `Workspace`, `Vault`, and `Editor` APIs, `Modal`/`Setting` UI building blocks, `MarkdownView`/`MarkdownRenderer`, the Commands API, `PluginSettingTab`, event/interval registration and cleanup, mobile compatibility, and plugin release/versioning. Each bullet cites the exact page it was drawn from.

## Plugin Class Lifecycle / Getting Started

- A plugin's core class extends the `Plugin` class from the `obsidian` package; "The Plugin class defines the lifecycle of a plugin and exposes the operations available to all plugins." (https://docs.obsidian.md/Plugins/Getting+started/Anatomy+of+a+plugin)
- `onload()` "runs whenever the user starts using the plugin in Obsidian. This is where you'll configure most of the plugin's capabilities." (https://docs.obsidian.md/Plugins/Getting+started/Anatomy+of+a+plugin)
- `onunload()` executes when a plugin is disabled; "any resources that your plugin is using must be released here to avoid affecting the performance of Obsidian after your plugin has been disabled." (https://docs.obsidian.md/Plugins/Getting+started/Anatomy+of+a+plugin)
- Example lifecycle usage adds UI in `onload()`, e.g. `this.addRibbonIcon('dice', 'Greet', () => { new Notice('Hello, world!'); });`. (https://docs.obsidian.md/Plugins/Getting+started/Build+a+plugin)
- Local plugin development: clone the sample repo, run `npm install`, run `npm run dev` for continuous rebuilding, enable the plugin under Settings → Community plugins, and use the "Reload app without saving" command after code changes. (https://docs.obsidian.md/Plugins/Getting+started/Build+a+plugin)
- The plugin's folder name must match the `id` field in `manifest.json`. (https://docs.obsidian.md/Plugins/Getting+started/Build+a+plugin)
- Obsidian must be restarted whenever `manifest.json` itself changes (unlike source code changes, which only need a reload). (https://docs.obsidian.md/Plugins/Getting+started/Build+a+plugin)
- Plugin status/logs can be inspected via the browser console (Ctrl+Shift+I on Windows/Linux, Cmd-Option-I on macOS), Console tab. (https://docs.obsidian.md/Plugins/Getting+started/Anatomy+of+a+plugin)

## manifest.json Fields

- Required fields for all manifests: `author` ("The author's name."), `minAppVersion` ("The minimum required Obsidian version."), `name` (display identifier; must use only Basic Latin characters, no special punctuation, no duplicate names, cannot include "Obsidian" variants), and `version` (Semantic Versioning `x.y.z`). (https://docs.obsidian.md/Reference/Manifest)
- Optional fields for all manifests: `authorUrl` ("A URL to the author's website.") and `fundingUrl` (a single URL string or an object of labeled URL pairs for financial support). (https://docs.obsidian.md/Reference/Manifest)
- Plugin-only required fields: `description` ("A description of your plugin.") and `id` (unique identifier — lowercase letters and hyphens only, cannot end in "plugin", cannot contain "obsidian"; should match the plugin's folder name for local development). (https://docs.obsidian.md/Reference/Manifest)
- `isDesktopOnly` is a boolean flag indicating the plugin requires desktop-only functionality such as Node.js or Electron APIs. (https://docs.obsidian.md/Reference/Manifest)

## versions.json / Plugin Compatibility Resolution

- Plugin versions must follow semantic versioning in the exact format `x.y.z` (e.g. `1.0.0`). (https://docs.obsidian.md/Plugins/Releasing/Submit+your+plugin)
- When creating a GitHub release, "The 'Tag version' of the release must match the version in your `manifest.json`." (https://docs.obsidian.md/Plugins/Releasing/Submit+your+plugin)
- Community plugin submission requires uploading `main.js`, `manifest.json`, and `styles.css` as binary attachments to each GitHub release; the community directory processes the manifest from the repository's default branch. (https://docs.obsidian.md/Plugins/Releasing/Submit+your+plugin)
- `versions.json` acts as a compatibility resolver: "If your `manifest.json` requires a version of Obsidian that's higher than the running app, your `versions.json` will be consulted to find the latest version of your plugin that is compatible." This lets a user on an older Obsidian build still install a working (older) release of the plugin instead of being blocked entirely. (https://github.com/obsidianmd/obsidian-releases)

## Workspace API

- `activeLeaf` "Indicates the currently focused leaf, if one exists" (nullable); `activeEditor` is the current editor component (nullable if no editor is present). (https://docs.obsidian.md/Reference/TypeScript+API/Workspace)
- `rootSplit` is the main workspace container; `leftSplit`/`rightSplit` are the sidebar containers (sidedocks or mobile drawers); `layoutReady` is a boolean indicating whether app layout has finished initializing. (https://docs.obsidian.md/Reference/TypeScript+API/Workspace)
- `getLeaf()` creates or retrieves leaves and supports modes such as `'tab'`, `'split'`, `'window'`, or reusing an existing leaf. (https://docs.obsidian.md/Reference/TypeScript+API/Workspace)
- `createLeafBySplit()` generates a new leaf adjacent to the current one; `iterateAllLeaves()` traverses all leaves across the main area, sidebars, and floating windows; `setActiveLeaf()` activates a specific leaf; `getLeavesOfType()` filters leaves by view type. (https://docs.obsidian.md/Reference/TypeScript+API/Workspace)
- `getActiveViewOfType()` gets "the currently active view of a given type." (https://docs.obsidian.md/Reference/TypeScript+API/Workspace)
- `onLayoutReady()` queues a callback for execution once layout initialization completes. (https://docs.obsidian.md/Reference/TypeScript+API/Workspace)
- `workspace.on()` supports events including `'active-leaf-change'`, `'file-open'`, `'layout-change'`, `'resize'`, `'window-open'`, `'window-close'`, plus editor and context-menu events. (https://docs.obsidian.md/Reference/TypeScript+API/Workspace)

## Vault API

- The `Vault` class extends the `Events` class to support reactive updates and provides file/folder lifecycle management. (https://docs.obsidian.md/Reference/TypeScript+API/Vault)
- Reading: `read()` for direct disk access, `cachedRead()` for display-only purposes. (https://docs.obsidian.md/Reference/TypeScript+API/Vault)
- Writing/creating: `create()`, `createBinary()`, `modify()`, `modifyBinary()`; appending via `append()` and `appendBinary()`. (https://docs.obsidian.md/Reference/TypeScript+API/Vault)
- Manipulation: `rename()`, `copy()`, `delete()`; atomic read-modify-save is combined in a single `process()` call. (https://docs.obsidian.md/Reference/TypeScript+API/Vault)
- Folder management: `createFolder()`, `getRoot()`, `getAllFolders()`. (https://docs.obsidian.md/Reference/TypeScript+API/Vault)
- File querying: `getAbstractFileByPath()` returns a `TFile` or `TFolder` — disambiguate using `instanceof TFile` or `instanceof TFolder`; `getFiles()`, `getMarkdownFiles()`, and `getAllLoadedFiles()` filter by type. (https://docs.obsidian.md/Reference/TypeScript+API/Vault)
- The `adapter` property provides low-level data access (added 0.9.7); `configDir` reveals the configuration folder path (typically `.obsidian`). (https://docs.obsidian.md/Reference/TypeScript+API/Vault)
- Event listeners track `create`, `modify`, `delete`, and `rename` operations. (https://docs.obsidian.md/Reference/TypeScript+API/Vault)

## Editor API

- The `Editor` class provides a unified interface bridging CodeMirror 5 and 6, with cursor and selection management. (https://docs.obsidian.md/Reference/TypeScript+API/Editor)
- Cursor methods: `getCursor(side)` retrieves the current cursor position; `setCursor(pos, ch)` sets a new cursor location; `scrollIntoView(range, center)` ensures cursor visibility. (https://docs.obsidian.md/Reference/TypeScript+API/Editor)
- Selection methods: `getSelection()` returns the currently selected text; `replaceSelection(replacement, origin)` modifies selected content; `listSelections()` returns all active selection ranges; `setSelection(anchor, head)` creates a single selection; `setSelections(ranges, main)` manages multiple selections; `somethingSelected()` checks whether any text is selected. (https://docs.obsidian.md/Reference/TypeScript+API/Editor)
- `transaction(tx, origin)` (added in v0.13.0) enables "grouped edits and atomic operations," letting multiple changes be applied as one cohesive `EditorTransaction` unit rather than as individual edits. (https://docs.obsidian.md/Reference/TypeScript+API/Editor)

## Modal Class

- `Modal` implements `CloseableComponent`; its constructor accepts an `app` parameter. (https://docs.obsidian.md/Reference/TypeScript+API/Modal)
- Key elements: `contentEl` (content HTMLElement), `titleEl` (title HTMLElement), `modalEl` (main modal element), `containerEl` (container element). (https://docs.obsidian.md/Reference/TypeScript+API/Modal)
- Lifecycle: `onOpen()` runs when the modal opens; `onClose()` runs when the modal closes. (https://docs.obsidian.md/Reference/TypeScript+API/Modal)
- Control methods: `open()` — "Show the modal on the active window. On mobile, the modal will animate on screen."; `close()` — "Hide the modal." (https://docs.obsidian.md/Reference/TypeScript+API/Modal)
- Configuration helpers: `setTitle(title)`, `setContent(content)`, and `setCloseCallback(callback)` (added in v1.10.0). (https://docs.obsidian.md/Reference/TypeScript+API/Modal)

## Setting Component

- `Setting`'s constructor takes a `containerEl` (HTMLElement); `setName(name)` and `setDesc(desc)` label the setting, and `setHeading()` marks a setting as a section header. (https://docs.obsidian.md/Reference/TypeScript+API/Setting)
- Chainable input-control methods: `addText(cb)`, `addTextArea(cb)`, `addToggle(cb)`, `addSlider(cb)`, `addDropdown(cb)`, `addButton(cb)`, `addColorPicker(cb)`, `addSearch(cb)`, `addMomentFormat(cb)`, `addProgressBar(cb)`, `addComponent(cb)`. (https://docs.obsidian.md/Reference/TypeScript+API/Setting)
- Utilities: `setClass(cls)`, `setDisabled(disabled)`, `setTooltip(tooltip, options)`, `then(cb)`, `clear()`. Each method returns the `Setting` instance for fluent chaining. (https://docs.obsidian.md/Reference/TypeScript+API/Setting)
- In a `PluginSettingTab`, "`new Setting(containerEl)` appends a setting to the container element. The Setting class provides methods like `setName` and `setDesc` to label the setting, plus a family of `add…` methods for each control type." (https://docs.obsidian.md/Plugins/User+interface/Settings)

## MarkdownView / MarkdownRenderer

- `MarkdownView` extends `TextFileView` and implements `MarkdownFileInfo`, serving as the primary interface for displaying markdown files in the editor. (https://docs.obsidian.md/Reference/TypeScript+API/MarkdownView)
- Key properties: `editor` (an `Editor` instance), `file` (`TFile | null`), `currentMode`/`previewMode` (editing vs. rendering modes), `data` (in-memory content), `hoverPopover`. (https://docs.obsidian.md/Reference/TypeScript+API/MarkdownView)
- Key methods: `getViewType()`, `getViewData()`/`setViewData()`, `getMode()`, `showSearch()`, `clear()`. It inherits lifecycle/component management from `Component`, `View`, and `TextFileView`. (https://docs.obsidian.md/Reference/TypeScript+API/MarkdownView)
- `MarkdownRenderer` is an abstract class (added v0.9.7) extending `MarkdownRenderChild` and implementing `MarkdownPreviewEvents` and `HoverParent`; properties include `app` (the `App` instance), `file` (abstract readonly `TFile`), and `hoverPopover`. (https://docs.obsidian.md/Reference/TypeScript+API/MarkdownRenderer)
- Its primary static method is `render(app, markdown, el, sourcePath, component)`, which "Renders Markdown string to an HTML element," taking the `App` instance, the markdown string, the target `el`, the `sourcePath`, and an associated `component`. (https://docs.obsidian.md/Reference/TypeScript+API/MarkdownRenderer)
- A `renderMarkdown(markdown, el, sourcePath, component)` static method also exists with similar functionality; the documentation lists `render()` alongside it, suggesting `render()` is the preferred/current method. (https://docs.obsidian.md/Reference/TypeScript+API/MarkdownRenderer)

## Commands API

- Plugins register commands via `addCommand()`, typically inside `onload()`, specifying an `id`, `name`, and an execution handler. (https://docs.obsidian.md/Plugins/User+interface/Commands)
- Basic commands use `callback()` for actions executable at any time. (https://docs.obsidian.md/Plugins/User+interface/Commands)
- Conditional commands use `checkCallback()`, which runs twice — first with `checking: true` to validate preconditions, then with `checking: false` to execute — and "you need to perform the check during both calls" to account for changing conditions. (https://docs.obsidian.md/Plugins/User+interface/Commands)
- Editor-specific commands use `editorCallback()`, which receives the active editor and view and only appears in the Command Palette when an editor is active; `editorCheckCallback()` is the conditional variant. (https://docs.obsidian.md/Plugins/User+interface/Commands)
- Default hotkeys are assigned via the `hotkeys` property with `modifiers` and `key` values; the special `'Mod'` modifier adapts to the platform (Ctrl on Windows/Linux, Cmd on macOS). The docs warn: "Avoid setting default hot keys for plugins that you intend for others to use" due to conflict risk. (https://docs.obsidian.md/Plugins/User+interface/Commands)

## Settings Tabs (PluginSettingTab)

- A settings tab extends `PluginSettingTab` and is registered via `addSettingTab()` in the plugin's `onload()`; persistence is handled through `loadData()`/`saveData()`. (https://docs.obsidian.md/Plugins/User+interface/Settings)
- Modern approach (Obsidian 1.13.0+): override `getSettingDefinitions()` to declare settings declaratively; each definition can specify a `control` type (toggle, text, number, dropdown, etc.). "Each `control` definition's `key` names a property on `this.plugin.settings`. Obsidian reads the current value, writes changes back, and calls `saveData()` automatically." (https://docs.obsidian.md/Plugins/User+interface/Settings)
- Legacy imperative approach: override `display()` to construct individual `Setting` rows manually, and use `hide()` to clean up resources (observers, timers, events) when the tab closes. (https://docs.obsidian.md/Plugins/User+interface/Settings)

## Events / Lifecycle Registration and Cleanup

- `registerEvent()` automatically detaches event handlers when a plugin unloads, rather than requiring manual subscription management for events like file creation. (https://docs.obsidian.md/Plugins/Events)
- "Any registered event handlers need to be detached whenever the plugin unloads. The safest way to make sure this happens is to use the `registerEvent()` method." (https://docs.obsidian.md/Plugins/Events)
- `registerInterval()` wraps `window.setInterval()` calls to guarantee they are cleaned up on plugin unload, preventing memory leaks from intervals that would otherwise run indefinitely. (https://docs.obsidian.md/Plugins/Events)
- Without these helpers, developers must manually track and tear down all subscriptions in `onunload()`; the Plugin/Component class internally tracks registered resources and manages their teardown automatically during unload. (https://docs.obsidian.md/Plugins/Events)

## Mobile Compatibility

- Setting `"isDesktopOnly": true` in the manifest restricts a plugin to desktop, preventing mobile installation for plugins that require Node.js or Electron APIs. (https://docs.obsidian.md/Plugins/Getting+started/Mobile+development)
- "The Node.js API, and the Electron API aren't available on mobile devices." Calls to these APIs can cause a plugin to crash on mobile; developers should use platform detection to provide mobile-safe alternatives. (https://docs.obsidian.md/Plugins/Getting+started/Mobile+development)
- The `Platform` utility supports conditional code paths, e.g. `import { Platform } from 'obsidian'; if (Platform.isIosApp) { /* iOS-specific code */ } if (Platform.isAndroidApp) { /* Android-specific code */ }`. (https://docs.obsidian.md/Plugins/Getting+started/Mobile+development)
- Testing options: desktop emulation via `this.app.emulateMobile(true)` in the Developer Tools console; real-device testing on Android requires USB debugging via Chrome DevTools, and on iOS requires iOS 16.4+, macOS, and Safari Web Inspector. (https://docs.obsidian.md/Plugins/Getting+started/Mobile+development)
- Regex caveat: lookbehind regex patterns only work on iOS 16.4+, which affects behavior on older iOS devices. (https://docs.obsidian.md/Plugins/Getting+started/Mobile+development)

## Sources

- https://docs.obsidian.md/Plugins/Getting+started/Build+a+plugin
- https://docs.obsidian.md/Plugins/Getting+started/Anatomy+of+a+plugin
- https://docs.obsidian.md/Plugins/Getting+started/Mobile+development
- https://docs.obsidian.md/Reference/Manifest
- https://docs.obsidian.md/Reference/TypeScript+API/Workspace
- https://docs.obsidian.md/Reference/TypeScript+API/Vault
- https://docs.obsidian.md/Reference/TypeScript+API/Editor
- https://docs.obsidian.md/Reference/TypeScript+API/Modal
- https://docs.obsidian.md/Reference/TypeScript+API/Setting
- https://docs.obsidian.md/Reference/TypeScript+API/MarkdownView
- https://docs.obsidian.md/Reference/TypeScript+API/MarkdownRenderer
- https://docs.obsidian.md/Plugins/User+interface/Commands
- https://docs.obsidian.md/Plugins/User+interface/Settings
- https://docs.obsidian.md/Plugins/Events
- https://docs.obsidian.md/Plugins/Releasing/Submit+your+plugin
- https://github.com/obsidianmd/obsidian-releases
