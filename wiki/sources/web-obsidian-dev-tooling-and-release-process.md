---
title: "Obsidian plugin developer tooling and community-release process (web research)"
type: source
status: active
created: 2026-09-19
updated: 2026-09-19
sources:
  - id: web-obsidian-dev-tooling-and-release-process
    hash: 2ee40b4c74b6
    ingested: 2026-09-19
aliases: [obsidian-dev-tooling, obsidian-release-process]
tags: [source, obsidian-api, plugin-dev, build-tooling, official-docs]
---

Web research on `obsidianmd/obsidian-api` (typings), `obsidianmd/obsidian-sample-plugin` (scaffold), the esbuild build pipeline, and the `obsidianmd/obsidian-releases` community-plugin submission process, plus official Plugin guidelines. Several docs.obsidian.md pages that would plausibly hold rejection criteria (telemetry disclosure, code obfuscation) returned 404 during research and are explicitly flagged as unverified rather than guessed at.

## Citation

github.com/obsidianmd/{obsidian-api, obsidian-sample-plugin, obsidian-releases}; docs.obsidian.md/Plugins/Releasing/{Plugin+guidelines, Submit+your+plugin, Release+your+plugin+with+GitHub+Actions}; docs.obsidian.md/Plugins/Getting+started/Development+workflow; github.com/TfTHacker/obsidian42-brat. Full list, including 404'd URLs, in `raw/web-obsidian-dev-tooling-and-release-process.md`.

## Key claims

- `obsidian` npm package supplies types/dev-time surface only — at runtime, a plugin's `main.js` calls `require('obsidian')` and the host app injects the real implementation.
- `version-bump.mjs` (verbatim retrieved) runs on `npm version`, syncing `manifest.json.version` from `npm_package_version` and appending a `version → minAppVersion` entry to `versions.json` — **this project's own `plugin/version-bump.mjs` already matches this exactly**, inherited from the sample-plugin scaffold via the upstream fork.
- esbuild config branches on `production` vs. dev: production = one-shot minified build, no sourcemap; dev = `context.watch()`, inline sourcemap, no minification. `obsidian`/`electron`/CodeMirror/Lezer marked external. **This project's own `plugin/esbuild.config.mjs` already matches this pattern.**
- Community submission: only the *initial* listing needs review (via the community.obsidian.md portal, not a hand-edited PR to `community-plugins.json` as commonly assumed); subsequent version updates are self-service via new GitHub releases with a tag matching `manifest.json`'s version. **Not relevant to this project** — no PR back to upstream and no community-directory submission is planned (see this project's own `CLAUDE.md`).
- **Plugin guidelines, several with direct relevance to this project's own code:**
  - "Avoid `innerHTML`/`outerHTML`/`insertAdjacentHTML`" for user-controlled content — use `createEl()`/`createDiv()`/`createSpan()` instead.
  - **"Prefer the `Editor` API over `Vault.modify()` for the active file; use `Vault.process()` for background edits; use `FileManager.processFrontMatter()` for YAML frontmatter edits."** Directly extends [[concepts/obsidian-vault-file-io]] — this project's `makeNoteFileContent()`/`stripTodoAppNoteMeta()` (`plugin/src/main.tsx`) hand-roll YAML frontmatter generation/stripping via string templates and regex, rather than using the documented `FileManager.processFrontMatter()` API.
  - "Use Sentence case in UI," avoid raw `<h1>`/`<h2>`, use `setHeading()`.
  - Don't detach leaves in `onunload()` — this is about a *plugin's* unload hook not touching the user's workspace layout; distinct from this project's `TaskNoteModal.onClose()` detaching a leaf *it itself created* for the embed, which is the correct place for that cleanup, not `onunload()`.
- BRAT (`TfTHacker/obsidian42-brat`) is the standard pre-submission beta-testing tool — install directly from a GitHub repo/branch/commit without manual `.obsidian/plugins/` copying. Not currently used by this project (this project uses a direct symlink into a test vault instead — see this project's own `CLAUDE.md` "Test vault" section).
