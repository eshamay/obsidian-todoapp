# CodeMirror 6 Architecture and Obsidian Editor Extensions

Obsidian's live-preview Source-mode editor is built on CodeMirror 6 (CM6), a from-scratch rewrite that splits the editor into an immutable state model and a separate DOM-rendering view layer. This is the reference material an Obsidian plugin developer would need before customizing editor behavior via CM6 extensions: the state/view split, how extensions compose, the decoration types used to render live-preview syntax hiding, the `StateField`/`StateEffect` mechanism for custom plugin state, `ViewPlugin` for DOM-level components, Obsidian's own public registration API, and real-world community examples.

## EditorState vs. EditorView

- CM6's document and state data structures are immutable, and operations on them are pure functions — the state model itself never mutates in place. (https://codemirror.net/docs/ref/#state.EditorState, https://codemirror.net/docs/ref/#state.Text)
- The view component and command interface wrap that immutable core in an imperative interface: `EditorView` synchronizes the DOM with the current `EditorState`, listens for browser events (typing, key presses, mouse interaction), and translates them into transactions that produce new state values. (https://codemirror.net/docs/ref/#view.EditorView)
- The CM6 guide frames this as a deliberate departure from a conventional library shape, explicitly contrasting it with CM6's own predecessor: "CodeMirror is structured quite a bit differently than your classical JavaScript library (including its own previous versions)." (https://codemirror.net/docs/guide/) CM5 held document, selection, and DOM concerns together in one monolithic editor object; CM6 separates the pure data model (`EditorState`) from the DOM-owning layer (`EditorView`).
- State changes always flow through transactions rather than direct mutation, which is what lets extensions like `StateField` compute new values deterministically from old value + transaction.

## Extension system

- All CM6 features — syntax highlighting, decorations, keymaps, gutters, themes — are supplied as `Extension` values passed into `EditorState.create({extensions: [...]})`, or equivalently into the `EditorView` constructor's `extensions` option, e.g. `new EditorView({extensions: [basicSetup, javascript()], parent: document.body})`. (https://codemirror.net/docs/guide/)
- "Extensions are provided as values (usually imported from some package), or arrays of such values. They can be arbitrarily nested," and are deduplicated during configuration. (https://codemirror.net/docs/ref/#state.EditorStateConfig.extensions)
- The active extension set lives inside `EditorState` itself and can be changed by a transaction; dynamic reconfiguration uses `Compartment.reconfigure`. (https://codemirror.net/docs/ref/#state.Compartment.reconfigure)
- Extension ordering/precedence is controlled first by an explicit precedence category, then by position — see `Prec`. (https://codemirror.net/docs/ref/#state.Prec)
- Extension "kinds" composed this way include: syntax-highlighting packages (e.g. a language's `javascript()` extension), keymaps/commands, `StateField`s, `ViewPlugin`s, decorations, `EditorView.theme()` themes, and configuration facets. (https://codemirror.net/docs/guide/)

## Decorations API

Decorations are the mechanism CM6 uses to render Obsidian's live-preview behavior — e.g. hiding the literal `**` markers around bold text while displaying the text as bold, or replacing a markdown checkbox token with a rendered checkbox widget. Decorations are stored in immutable `DecorationSet`s built on `RangeSet`. (https://codemirror.net/docs/ref/#state.RangeSet)

- **`Decoration.mark(spec)`** — adds style or DOM attributes to the text within a range, without altering document content. Spec fields include `class`/`attributes` (add a CSS class or arbitrary DOM attributes), `tagName` (wrap the range in an element), and `inclusive`/`inclusiveStart`/`inclusiveEnd` (whether the range covers its boundary positions). (https://codemirror.net/docs/ref/#view.Decoration^mark) This is the mechanism for styling markdown syntax markers in place (e.g. dimming `**` rather than hiding it).
- **`Decoration.widget(spec)`** — inserts a DOM element at a position without consuming document text. Spec fields: `widget` (a `WidgetType` instance), `side` (which side of the position the widget renders on), and `block` (inline vs. block widget). (https://codemirror.net/docs/ref/#view.Decoration^widget) Used for things like a rendered checkbox or embed preview injected inline.
- **`Decoration.replace(spec)`** — hides part of the document or replaces it with a given DOM node, and is the direct mechanism behind Obsidian's "hide the `**`/`__`/`#` syntax marker, show only the rendered result" live-preview effect. Spec fields: optional `widget` to draw in place of the hidden range, `inclusive`/`inclusiveStart`/`inclusiveEnd`, and `block`. (https://codemirror.net/docs/ref/#view.Decoration^replace)
- **`Decoration.line(spec)`** — attaches attributes/classes to the DOM element wrapping an entire line, via `attributes` or the `class` shorthand. (https://codemirror.net/docs/ref/#view.Decoration^line)
- Individual decorated ranges are collected into a `DecorationSet` with `Decoration.set(of, sort?)`. (https://codemirror.net/docs/ref/#view.Decoration^set)

## StateField and StateEffect

`StateField` is CM6's mechanism for a plugin to carry custom, immutable state that updates in lockstep with every transaction; `StateEffect` is the signal type used to push explicit, non-document-derived changes into a field.

- `StateField.define(config)` config shape: `create(state)` returns the field's initial value; `update(value, transaction)` computes the new value from the old value and the transaction; `compare?` customizes equality checking; `provide?` lets the field itself contribute further extensions (e.g. decorations); `toJSON?`/`fromJSON?` support serialization. (https://codemirror.net/docs/ref/#state.StateField^define)
- Guide description: "State fields, living inside the purely functional state data structure, must store immutable values," and are "kept in sync with the rest of the state using something like a reducer. Every time the state updates, a function is called with the field's current value and the transaction." (https://codemirror.net/docs/guide/)
- Minimal example from the guide:
  ```js
  let countDocChanges = StateField.define({
    create() { return 0 },
    update(value, tr) { return tr.docChanged ? value + 1 : value }
  })
  ```
- `StateEffect.define(spec?)` creates a `StateEffectType`; its `.of(value)` method produces an effect instance to attach to a transaction (`tr.effects`), and the field's `update()` reads `tr.effects` to react to it. An optional `map` spec field lets the effect's value be remapped through document changes. (https://codemirror.net/docs/ref/#state.StateEffect^define)
- Guide example pairing an effect with a field:
  ```js
  let setFullScreenMode = StateEffect.define<boolean>()
  let fullScreenMode = StateField.define({
    create() { return false },
    update(value, tr) {
      for (let e of tr.effects) if (e.is(setFullScreenMode)) value = e.value
      return value
    }
  })
  ```

## ViewPlugin

`ViewPlugin` is CM6's mechanism for an imperative, DOM-owning component that lives alongside the state model and reacts to view updates (e.g. scroll position, cursor moves, viewport changes) that a pure `StateField` cannot observe.

- `ViewPlugin.fromClass(cls, spec?)` creates a plugin from a class whose constructor takes the `EditorView` (and optional arg) and returns an instance implementing the `PluginValue` interface. (https://codemirror.net/docs/ref/#view.ViewPlugin^fromClass)
- `PluginValue` interface: `update?(update: ViewUpdate)` — notified of updates that happened in the view; `docViewUpdate?(view)` — called when the document view itself updates; `destroy?()` — cleanup when the plugin is discarded. (https://codemirror.net/docs/ref/#view.ViewPlugin^fromClass)
- Guide framing: "View plugins provide a way for extensions to run an imperative component inside the view," and recommends they "should generally not hold (non-derived) state" — best used as "shallow views over the data kept in the editor state." (https://codemirror.net/docs/ref/#view.ViewPlugin, https://codemirror.net/docs/guide/)
- Guide example:
  ```js
  const docSizePlugin = ViewPlugin.fromClass(class {
    constructor(view) {
      this.dom = view.dom.appendChild(document.createElement("div"))
      this.dom.textContent = view.state.doc.length
    }
    update(update) {
      if (update.docChanged) this.dom.textContent = update.state.doc.length
    }
  })
  ```

## Obsidian's public editor-extension registration API

Obsidian exposes a documented, public method on the `Plugin` class specifically for registering CM6 extensions — this is the supported integration point rather than reaching into CM6 internals directly.

- `Plugin.registerEditorExtension(extension: Extension): void` — the `extension` parameter "must be a CodeMirror 6 Extension, or an array of Extensions." (https://docs.obsidian.md/Reference/TypeScript+API/Plugin/registerEditorExtension)
- The Obsidian editor-extensions guide instructs calling it from `onload()`: "To register an editor extension, use registerEditorExtension() in the onload method of your Obsidian plugin," and identifies "View plugins and State fields" as "two of the most common" extension kinds plugin authors register this way. (https://docs.obsidian.md/Plugins/Editor/Editor+extensions)
- Typical usage: `this.registerEditorExtension([examplePlugin, exampleField]);` inside `onload()`. (https://docs.obsidian.md/Plugins/Editor/Editor+extensions)
- For dynamic reconfiguration, the API reference recommends passing an array and mutating it at runtime, then calling `Workspace.updateOptions()` to push the change into all open editors. (https://docs.obsidian.md/Reference/TypeScript+API/Plugin/registerEditorExtension)

## Community plugins using registerEditorExtension

Several widely-used Obsidian community plugins register CM6 `ViewPlugin`/`StateField` extensions through this public API, illustrating what is achievable without touching CM6 internals directly:

- **obsidian-completr** (autocomplete plugin) registers a `StateField` for suggestion markers and a `ViewPlugin`-style update listener: `this.registerEditorExtension(markerStateField);` plus an `EditorView.updateListener` registration. (https://github.com/tth05/obsidian-completr/blob/400fb99279345f8f7424ef58a6076e7a93ac5fdc/src/main.ts, lines 38-39)
- **obsidian-tasks** (task-management plugin) registers a live-preview extension bundle: `this.registerEditorExtension(newLivePreviewExtension(this));`. (https://github.com/obsidian-tasks-group/obsidian-tasks/blob/61ce07a94661807090e7b395cb79f19c10830a68/src/main.ts, line 82)
- **obsidian-banners** (banner-image plugin) registers a paired `ViewPlugin` + `StateField`: `plug.registerEditorExtension([bannerExtender, bannerField]);`, driving the banner's live-preview rendering via `StateEffect`s (`openNoteEffect`, `refreshEffect`, `removeBannerEffect`) dispatched into `bannerField`. (https://github.com/noatpad/obsidian-banners/blob/f5f5aa732752124bcdde95c378a2e8ae243d5066/src/editing/index.ts, lines 11-17)

These examples confirm that `registerEditorExtension` alone is sufficient to add custom decorations, widgets, and reactive state to the live-preview editor — no undocumented API or CM6-internals access is required for this class of customization.

## Sources

- https://codemirror.net/docs/guide/
- https://codemirror.net/docs/ref/#state.EditorState
- https://codemirror.net/docs/ref/#state.Text
- https://codemirror.net/docs/ref/#view.EditorView
- https://codemirror.net/docs/ref/#state.EditorStateConfig.extensions
- https://codemirror.net/docs/ref/#state.Compartment.reconfigure
- https://codemirror.net/docs/ref/#state.Prec
- https://codemirror.net/docs/ref/#state.RangeSet
- https://codemirror.net/docs/ref/#view.Decoration^mark
- https://codemirror.net/docs/ref/#view.Decoration^widget
- https://codemirror.net/docs/ref/#view.Decoration^replace
- https://codemirror.net/docs/ref/#view.Decoration^line
- https://codemirror.net/docs/ref/#view.Decoration^set
- https://codemirror.net/docs/ref/#state.StateField^define
- https://codemirror.net/docs/ref/#state.StateEffect^define
- https://codemirror.net/docs/ref/#view.ViewPlugin^fromClass
- https://codemirror.net/docs/ref/#view.ViewPlugin
- https://docs.obsidian.md/Plugins/Editor/Editor+extensions
- https://docs.obsidian.md/Reference/TypeScript+API/Plugin/registerEditorExtension
- https://github.com/tth05/obsidian-completr/blob/400fb99279345f8f7424ef58a6076e7a93ac5fdc/src/main.ts
- https://github.com/obsidian-tasks-group/obsidian-tasks/blob/61ce07a94661807090e7b395cb79f19c10830a68/src/main.ts
- https://github.com/noatpad/obsidian-banners/blob/f5f5aa732752124bcdde95c378a2e8ae243d5066/src/editing/index.ts
