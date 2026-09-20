> Pulled from work-organizer wiki (`raw/research-local-markdown-apps.md`), dated 2026-09-18 — research for a different project (work-organizer command-center); reused here as prior art.

# Local-First, Markdown-Native Personal Knowledge App Architectures

> **Audience:** Design team for a new local command-center web app <br>
> **Purpose:** Survey architectures and recommend a build approach for a fast, click-driven local web app over an Obsidian-compatible markdown vault <br>
> **Compiled:** 2026-09-18 <br>
> **Status:** Research draft for wiki

## Summary

Every mature tool in this space converges on one principle: **plain markdown files on disk are the source of truth**, and the application is a disposable view layer over them. This is the core of "local-first software" — data lives on the user's device, works offline, and survives the app that created it ([Local-first software, Wikipedia](https://en.wikipedia.org/wiki/Local-first_software)). That principle is exactly the hard requirement already decided for this project, so the design question is not *whether* to store markdown, but *what renders and edits it*: an Obsidian plugin inheriting Obsidian's UI, or a separate local server + web frontend with full UI freedom.

The recommendation below is a **separate local backend + SPA**, because the requirement for fast, customizable, click-through views tailored to active work — plus a roadmap toward Slack/GUS/GitHub/Google integrations — exceeds what Obsidian's plugin surface renders comfortably, while markdown-on-disk keeps Obsidian usable in parallel.

## The landscape

**Obsidian's own ecosystem** is the richest reference. Three pieces matter:

- **Local REST API plugin** exposes full CRUD over vault files (including targeted section edits and binary attachments), full-text and structured (JsonLogic) search, and command execution, over an authenticated HTTPS server at `127.0.0.1:27124` with bearer-token auth ([obsidian-local-rest-api](https://github.com/coddingtonbear/obsidian-local-rest-api)). This is the single most reusable component for the project: it turns a running Obsidian instance into a local backend a custom frontend can call.
- **Dataview** is a live index-and-query engine: it reads YAML frontmatter and inline `[key:: value]` fields, then renders auto-updating tables, lists, calendars, and task queries via its own query language (DQL), inline queries, or JavaScript — without modifying the underlying files ([Dataview docs](https://blacksmithgu.github.io/obsidian-dataview/)). This is the model to imitate for "customizable views."
- **Bases** is now a **core plugin** (not third-party) that creates database-like table, list, card, Kanban, and map views from note properties, saved as `.base` files or embedded code blocks — with all data staying in local markdown and their properties ([Obsidian Bases help](https://obsidian.md/help/bases)). Bases signals Obsidian's official direction toward structured, view-driven use of frontmatter, which is precisely the command-center pattern.

**Other tools** confirm the pattern and its variations:

- **Logseq** is local-first and markdown-based, though it is block-oriented and has been moving toward a database-backed model; treat it as evidence that block granularity and a DB index can coexist with markdown export, not as a drop-in ([Logseq docs](https://docs.logseq.com/)).
- **Foam** is a VS Code extension over a folder of markdown, providing `[[wikilink]]` navigation, backlinks, and graph views with no separate app or proprietary format, leaning on Git for versioning ([Foam docs](http://docs.foam.md/)). It shows how little "app" you need if you accept an existing editor's UI.
- **Dendron** is an open-source, local-first, markdown, VS Code-native tool built around hierarchical note names, schema-driven autocomplete/templates, and link-preserving refactors ([Dendron wiki](https://wiki.dendron.so/)). Its schema system is worth studying for imposing structure on a growing vault.

## The most directly relevant precedent: SilverBullet

**SilverBullet** is the closest existing thing to what this project wants: a self-contained, self-hosted **web application** that stores notes as a versioned set of markdown files on disk ("a Space"), and layers on a live-preview editor, an embedded database with a query language, and a Lua scripting environment for dynamically generating content — page pickers, file trees, wiki-links, queries, and templates all in the browser ([SilverBullet](https://silverbullet.md/)). It is distributed as a single server binary or Docker container. It proves the exact architecture — **local server process + browser SPA over a markdown folder** — is viable and performant, and it is open source, so it is worth studying or forking rather than starting from zero.

## Architectural pattern for a custom build

The reference shape is a small local backend + a fast SPA:

1. **Backend**: a local process (Node/Deno/Go/Rust) that owns file I/O against the vault folder, parses frontmatter + body, and maintains an in-memory or SQLite index for fast queries. SilverBullet's "files on disk + query database" split is the template ([SilverBullet](https://silverbullet.md/)); Dataview's index-over-frontmatter approach is the query model ([Dataview](https://blacksmithgu.github.io/obsidian-dataview/)).
2. **Write discipline**: edits round-trip through markdown + YAML so files stay byte-compatible with Obsidian and clean under Git. Bases and Dataview both demonstrate that keeping all state in markdown properties — never a sidecar database as the authority — is what preserves portability ([Obsidian Bases](https://obsidian.md/help/bases)).
3. **Frontend**: an SPA delivering the click-through, textbox-driven, customizable views. Views are saved queries/layouts over frontmatter, mirroring Bases' `.base` files and Dataview queries.
4. **Integration seam**: because the backend already brokers all reads/writes, later Slack/GUS/GitHub/Google connectors attach at the backend as additional data sources that materialize into markdown notes — no frontend rework.

You can either reuse Obsidian's Local REST API as this backend (fast start, but requires Obsidian running) or write a standalone backend (more work, no Obsidian dependency).

## Recommendation and tradeoffs

**Build a standalone local backend + SPA, keep the vault Obsidian-compatible, and study/fork SilverBullet.**

| Approach | Gains | Costs |
|---|---|---|
| **Obsidian plugin** | Inherits Obsidian's editor, sync, graph, and Bases/Dataview for free; least code | UI constrained to Obsidian's plugin API and rendering; hard to build a bespoke fast command-center layout; integrations run inside Obsidian's lifecycle |
| **Standalone web app** (recommended) | Full UI freedom for tailored, fast views; clean home for Slack/GUS/GitHub/Google connectors; can run headless | Must build editor/index/sync yourself; must stay disciplined about markdown compatibility so Obsidian keeps working in parallel |

The plugin route wins on speed-to-first-version and is the right call if the person is happy living inside Obsidian's panes. But the stated requirements — a *fast, responsive, customizable* command center whose views change with what is being worked on, plus a multi-system integration roadmap — are the things Obsidian's plugin surface constrains most. A standalone app pays more upfront and inherits nothing, yet the markdown-on-disk contract means Obsidian, Git, and any future tool all keep working against the same files ([Local-first software](https://en.wikipedia.org/wiki/Local-first_software)). A useful hedge: prototype against the **Local REST API** first to validate views quickly, then graduate to a standalone backend once the view model stabilizes.

## Sources

- Local-first software — https://en.wikipedia.org/wiki/Local-first_software
- Obsidian Local REST API — https://github.com/coddingtonbear/obsidian-local-rest-api
- Obsidian Dataview — https://blacksmithgu.github.io/obsidian-dataview/
- Obsidian Bases — https://obsidian.md/help/bases
- Logseq docs — https://docs.logseq.com/
- Foam docs — http://docs.foam.md/
- Dendron wiki — https://wiki.dendron.so/
- SilverBullet — https://silverbullet.md/
