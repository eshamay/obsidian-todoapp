# Obsidian Plugin Developer Tooling and Release Process

Research summary on the Obsidian plugin developer toolchain: the official TypeScript API repo, the sample-plugin scaffold, the esbuild build pipeline, and the community-plugin submission/review/versioning process on `obsidian-releases`. Each claim below is cited to the exact URL fetched. Some documentation pages referenced by cross-links (`Developer policies`, `Submission requirements for plugins`, `Plugin review guidelines`) returned 404 at the time of this research and are called out explicitly rather than cited with fabricated content.

## `obsidianmd/obsidian-api`

- The repo provides "Type definitions for the latest Obsidian API," including `obsidian.d.ts` plus related typing files (`canvas.d.ts`, `publish.d.ts`) that give plugin code TypeScript types for the Obsidian app surface. Source: https://github.com/obsidianmd/obsidian-api
- The repo tracks changes in a `CHANGELOG.md` and has a long commit history on `master`, consistent with an incrementally-versioned typings package rather than a stable-forever spec. Source: https://github.com/obsidianmd/obsidian-api
- Plugins consume the typings via the npm package named `obsidian`, and at runtime a plugin's bundled `main.js` accesses the actual API via `require('obsidian')` (the host app injects the real implementation; the npm package supplies types/dev-time surface only). Source: https://github.com/obsidianmd/obsidian-api
- Official guidance and the canonical starting point for plugin development is documented at docs.obsidian.md, and the sample plugin template is the reference consumer of the `obsidian` package. Source: https://github.com/obsidianmd/obsidian-api
- API questions/issues are directed to the Obsidian developer forum rather than filed as GitHub issues against this repo, indicating community-driven support rather than a formal issue-tracker SLA. Source: https://github.com/obsidianmd/obsidian-api

## `obsidianmd/obsidian-sample-plugin` scaffold

This is the canonical scaffold this project's `todoapp-blocks-plugin` fork already follows structurally.

- `main.ts` — the plugin's TypeScript entry point; compiles down to `main.js`. The sample implements ribbon icons, modals, a settings tab, and event listeners as example patterns. Source: https://github.com/obsidianmd/obsidian-sample-plugin
- `manifest.json` — plugin metadata (id, name, version, minimum Obsidian app version, etc.); must exist both in the repo root and be attached as a release asset. Source: https://github.com/obsidianmd/obsidian-sample-plugin
- `package.json` — Node project config: dependencies plus the `dev`/`build` npm scripts that drive the esbuild pipeline. Source: https://github.com/obsidianmd/obsidian-sample-plugin
- `styles.css` — plugin stylesheet, bundled and shipped alongside the compiled JS as an optional release asset. Source: https://github.com/obsidianmd/obsidian-sample-plugin
- `versions.json` — maps each released plugin version to the minimum compatible Obsidian app version, supporting older-Obsidian compatibility resolution. Source: https://github.com/obsidianmd/obsidian-sample-plugin
- `version-bump.mjs` — automation script invoked on `npm version` that synchronizes `manifest.json`'s `version` field and appends an entry to `versions.json`. Full verbatim content retrieved:
  ```js
  import { readFileSync, writeFileSync } from 'fs';

  const targetVersion = process.env.npm_package_version;

  // read minAppVersion from manifest.json and bump version to target version
  const manifest = JSON.parse(readFileSync('manifest.json', 'utf8'));
  const { minAppVersion } = manifest;
  manifest.version = targetVersion;
  writeFileSync('manifest.json', JSON.stringify(manifest, null, '\t'));

  // update versions.json with target version and minAppVersion from manifest.json
  // but only if the target version is not already in versions.json
  const versions = JSON.parse(readFileSync('versions.json', 'utf8'));
  if (!(targetVersion in versions)) {
      versions[targetVersion] = minAppVersion;
      writeFileSync('versions.json', JSON.stringify(versions, null, '\t'));
  }
  ```
  Source: https://raw.githubusercontent.com/obsidianmd/obsidian-sample-plugin/master/version-bump.mjs. It reads `npm_package_version` (set by npm during `npm version`), copies it into `manifest.json`, and adds a `version → minAppVersion` entry to `versions.json` only if that version isn't already present.
- Manual local install/beta-test convention: copy the compiled `manifest.json`, `main.js`, and `styles.css` into a vault's `.obsidian/plugins/<your-plugin-id>/` folder. Source: https://github.com/obsidianmd/obsidian-sample-plugin
- Community listing requires following the official guidelines and opening a PR against `obsidian-releases`. Source: https://github.com/obsidianmd/obsidian-sample-plugin

## esbuild build pipeline (`esbuild.config.mjs`)

- The config branches on `process.argv[2] === 'production'`:
  - **Production**: runs a single one-shot rebuild (no watch), disables sourcemaps (`sourcemap: false`), and enables minification.
  - **Development** (default): calls `context.watch()` for continuous rebuild on file change, uses inline sourcemaps (`sourcemap: 'inline'`) for in-browser debugging, and skips minification to keep output readable.
  Source: https://raw.githubusercontent.com/obsidianmd/obsidian-sample-plugin/master/esbuild.config.mjs
- The bundle marks `obsidian`, `electron`, CodeMirror, and Lezer packages as external (not bundled — resolved at runtime by the host app), includes Node builtins, targets `es2021`, and outputs CommonJS format matching what Obsidian's plugin loader expects from `main.js`. Source: https://raw.githubusercontent.com/obsidianmd/obsidian-sample-plugin/master/esbuild.config.mjs
- `npm run dev` runs the watch/dev build; `npm run build` runs the one-shot production build. Source: https://github.com/obsidianmd/obsidian-sample-plugin

## `obsidianmd/obsidian-releases` and the community submission process

- `community-plugins.json` is the central registry Obsidian's in-app plugin browser reads to populate the list of installable community plugins; its `name`, `author`, and `description` fields back in-app search. Source: https://github.com/obsidianmd/obsidian-releases
- Update mechanics: plugin authors keep `manifest.json`'s `version` current in their own repo; "Obsidian will look for your GitHub releases tagged identically to the version inside manifest.json," pulling `manifest.json`, `main.js`, and `styles.css` (optional) as release assets. `versions.json` lets Obsidian resolve a compatible plugin version for a given app version. Source: https://github.com/obsidianmd/obsidian-releases
- Only the *initial* submission requires review/approval into `community-plugins.json`; subsequent version updates are self-service via new GitHub releases and don't require a new PR. Source: https://docs.obsidian.md/Plugins/Releasing/Submit+your+plugin
- Official submission steps (per docs.obsidian.md):
  1. Publish the plugin's source on GitHub.
  2. Set `manifest.json` `version` to a Semantic Versioning (`x.y.z`) value, then cut a GitHub release whose **tag matches that version exactly**, attaching `main.js`, `manifest.json`, and optionally `styles.css` as binary assets.
  3. Submit via the community directory portal at community.obsidian.md — sign in with an Obsidian account, link/verify the GitHub repo, and add the plugin; the portal reads the manifest from the repo's default-branch HEAD.
  4. Address automated review feedback by fixing the repo and publishing a new incremented-version release.
  Source: https://docs.obsidian.md/Plugins/Releasing/Submit+your+plugin
- Pre-submission repo requirements: a root `README.md` (excerpts are shown on the plugin's public listing), a `LICENSE` file governing reuse, and a `manifest.json` describing the plugin, plus general compliance with "developer policies and submission requirements." Source: https://docs.obsidian.md/Plugins/Releasing/Submit+your+plugin
- The plugin `id` in `manifest.json` must be unique across the directory and **must not contain the substring "obsidian."** Source: https://docs.obsidian.md/Plugins/Releasing/Submit+your+plugin
- Obsidian's release tag/version resolution downloads assets from the GitHub release matching the manifest's `version` field. Source: https://docs.obsidian.md/Plugins/Releasing/Submit+your+plugin
- A GitHub Actions release workflow pattern (recommended, sample-plugin style) triggers on pushing a tag matching `manifest.json`'s version (`git push origin <version>`), builds the plugin, and creates a draft GitHub release with that tag/version as the release name. Source: https://docs.obsidian.md/Plugins/Releasing/Release+your+plugin+with+GitHub+Actions
- `obsidian-releases` previously maintained a `plugin-review.md` document of common reviewer feedback; that document now states its guidance has moved and points reviewers/authors to the official Plugin guidelines page instead. Source: https://raw.githubusercontent.com/obsidianmd/obsidian-releases/master/plugin-review.md

## Plugin guidelines (docs.obsidian.md/Plugins/Releasing/Plugin+guidelines)

Concrete, verbatim-quoted rules retrieved from the guidelines page (this page does **not** contain telemetry-disclosure or code-obfuscation rules — see caveat below):

- General: "Avoid using the global app object, `app` (or `window.app`). Instead, use the reference provided by your plugin instance, `this.app`." Avoid unnecessary console logging. Rename placeholder classes (`MyPlugin`, `MyPluginSettings`, `SampleSettingTab`) to the plugin's real names. Source: https://docs.obsidian.md/Plugins/Releasing/Plugin+guidelines
- Security: "Avoid `innerHTML`, `outerHTML` and `insertAdjacentHTML`" when rendering user-controlled input; use DOM-construction APIs like `createEl()`, `createDiv()`, `createSpan()` instead. Source: https://docs.obsidian.md/Plugins/Releasing/Plugin+guidelines
- Resource management: any resources the plugin creates (event listeners, etc.) must be destroyed/released on unload; prefer `registerEvent()` / `addCommand()` for auto-cleanup; don't detach leaves in `onunload()`. Source: https://docs.obsidian.md/Plugins/Releasing/Plugin+guidelines
- UI text: "Use Sentence case in UI" (not Title Case); "Avoid 'settings' in settings headings"; use `setHeading()` instead of raw `<h1>`/`<h2>` elements. Source: https://docs.obsidian.md/Plugins/Releasing/Plugin+guidelines
- Commands: avoid setting default hotkeys; use the correct callback variant (`callback`, `checkCallback`, `editorCallback`, `editorCheckCallback`). Source: https://docs.obsidian.md/Plugins/Releasing/Plugin+guidelines
- Vault/data access: prefer the `Editor` API over `Vault.modify()` for the active file; use `Vault.process()` for background edits; use `FileManager.processFrontMatter()` for YAML frontmatter edits; use `normalizePath()` on user-supplied paths; avoid iterating all vault files — use targeted lookups (`getFileByPath()`, etc.). Source: https://docs.obsidian.md/Plugins/Releasing/Plugin+guidelines
- Styling: no hardcoded inline styles — use CSS classes and Obsidian's CSS variables. TypeScript style: prefer `const`/`let` over `var`, and async/await over raw Promises. Source: https://docs.obsidian.md/Plugins/Releasing/Plugin+guidelines

**Caveat on rejection criteria (telemetry, obfuscation, id-format specifics):** the task asked about explicit rejection reasons such as "no telemetry without disclosure" and "no obfuscated code." The `Plugin+guidelines` page fetched does not contain those items, and several plausible source pages returned 404 during this research: `docs.obsidian.md/Developer+policies`, `docs.obsidian.md/Plugins/Releasing/Developer+policies`, `docs.obsidian.md/Plugins/Releasing/Submission+requirements+for+plugins`, `docs.obsidian.md/Plugins/Releasing/Plugin+review+guidelines`, and `obsidian-releases`' own `CONTRIBUTING.md`. The only confirmed id-related rule is the uniqueness/no-"obsidian"-substring rule cited above from the Submit-your-plugin page. Telemetry-disclosure and code-obfuscation rules could not be independently verified from the pages that were reachable in this session and are therefore omitted rather than asserted without citation.

## Beta-testing conventions

- Manual local beta-test path: place the built `manifest.json`, `main.js`, and `styles.css` into `<vault>/.obsidian/plugins/<plugin-id>/`, matching the sample plugin's own install instructions. Source: https://github.com/obsidianmd/obsidian-sample-plugin
- Manual reload during development: Settings → Community plugins → toggle the plugin off then on to pick up a rebuilt `main.js`. Source: https://docs.obsidian.md/Plugins/Getting+started/Development+workflow
- The community "Hot-Reload" plugin automates this: it "reloads your plugin whenever the source code changes," removing the manual toggle step. Source: https://docs.obsidian.md/Plugins/Getting+started/Development+workflow
- BRAT ("Beta Reviewers Auto-update Tester," by TfTHacker) is the standard tool for pre-submission beta testing: it lets testers add a plugin's GitHub repo path directly, installing/updating it straight from GitHub releases (or specific branches/commits) without the manual copy-to-`.obsidian/plugins` step, so developers can iterate with real testers before their plugin is in the official community directory. Source: https://github.com/TfTHacker/obsidian42-brat

## Sources

- https://github.com/obsidianmd/obsidian-api
- https://github.com/obsidianmd/obsidian-sample-plugin
- https://raw.githubusercontent.com/obsidianmd/obsidian-sample-plugin/master/version-bump.mjs
- https://raw.githubusercontent.com/obsidianmd/obsidian-sample-plugin/master/esbuild.config.mjs
- https://github.com/obsidianmd/obsidian-releases
- https://raw.githubusercontent.com/obsidianmd/obsidian-releases/master/plugin-review.md
- https://docs.obsidian.md/Plugins/Releasing/Plugin+guidelines
- https://docs.obsidian.md/Plugins/Releasing/Submit+your+plugin
- https://docs.obsidian.md/Plugins/Releasing/Release+your+plugin+with+GitHub+Actions
- https://docs.obsidian.md/Plugins/Getting+started/Development+workflow
- https://github.com/TfTHacker/obsidian42-brat

### URLs attempted that returned 404 (not usable as citations)

- https://docs.obsidian.md/Plugins/Releasing/Update+your+plugin
- https://docs.obsidian.md/Plugins/Releasing/Plugin+review+guidelines
- https://docs.obsidian.md/Plugins/Releasing/Submission+requirements+for+plugins
- https://docs.obsidian.md/Plugins/Releasing/Developer+policies
- https://docs.obsidian.md/Developer+policies
- https://docs.obsidian.md/oa/plugin/developer-policies
- https://github.com/obsidianmd/obsidian-releases/blob/master/.github/PULL_REQUEST_TEMPLATE.md
- https://raw.githubusercontent.com/obsidianmd/obsidian-releases/master/CONTRIBUTING.md
- https://raw.githubusercontent.com/obsidianmd/obsidian-releases/master/.github/workflows/validate-plugin-entry.yml
- https://raw.githubusercontent.com/obsidianmd/obsidian-releases/master/.github/PULL_REQUEST_TEMPLATE/plugin.md
