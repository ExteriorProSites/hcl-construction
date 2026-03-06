# Copilot / AI Agent Instructions — Intermediate Website Kit (Eleventy + LESS)

Purpose: make an AI coding agent immediately productive in this repo by documenting the build flow, conventions, and places to look for common changes.

## Quick actions
- Install deps: `npm install`
- Dev (live reload): `npm start` — runs `watch:less`, `watch:eleventy`, and `watch:cms` in parallel; clears `public/` first
- Dev Eleventy only: `npm run watch:eleventy` (sets `ELEVENTY_ENV=DEV`)
- Build for production: `npm run build` (sets `ELEVENTY_ENV=PROD`, runs `build:less` and `build:eleventy`)
- Run CMS locally: `npx decap-server` or `npm run watch:cms`
- Netlify deploys with `npm run build` (see `netlify.toml`)

## High-level architecture
- Static site built with Eleventy. Source: `src/` → Output: `public/` (do not edit `public/`; it's generated)
- Content: `src/content/` (pages and blog posts). Create pages using `src/content/pages/_template.txt` front matter
- Includes/layouts: `src/_includes/` and `src/layouts/` (Nunjucks, layout chaining)
- Global data: `src/_data/` (e.g., `src/_data/client.json` is used by the sitemap plugin)
- Assets:
  - LESS source: `src/assets/less/` → compiled to `src/assets/css/` via `less-watch-compiler` (`build:less`)
  - Compiled CSS is processed as a template by Eleventy (see `.eleventy.js`) and passed through to `public/assets`
  - JS source: `src/assets/js/` → handled by the Eleventy JS extension which runs `esbuild` (see `src/config/javascript.js`), output to `public/assets/js`
- Admin / CMS: Decap CMS (Netlify CMS) under `src/admin` (config in `src/admin/config.yml`)

## Important project-specific patterns & gotchas
- `.eleventy.js` is the central build config: it registers custom template extensions for `.css` and `.js` and the plugins: navigation, sitemap, and (prod-only) minifier.
- `isProduction` is derived from `ELEVENTY_ENV=PROD` in scripts (see `src/config/server.js`). This affects:
  - Whether Eleventy's minifier plugin runs
  - Whether `esbuild` minifies and targets `es6` (see `src/config/javascript.js`)
- JS compilation is intentionally scoped to assets inside `./src/assets/` (see `src/config/javascript.js`): files outside this folder are ignored by the ESBuild pipeline.
- UI navigation is built from page front matter using the Eleventy Navigation plugin. Example frontmatter keys are in `src/content/pages/_template.txt` under `eleventyNavigation`.
- Do not manually edit files in `src/assets/css/` or `public/`; those are generated. Make changes to sources (`src/assets/less/` and `src/assets/js/`).

## Files to check first when working on a task
- `.eleventy.js` — core build rules, plugins, passthroughs
- `package.json` — scripts: `start`, `build`, `watch:*`, etc.
- `src/config/javascript.js` — how JS is bundled (esbuild config and output path)
- `src/config/css.js` — registered CSS template behavior
- `src/config/server.js` — `isProduction` and watch settings used by Eleventy dev server
- `src/_includes/components/header.html` — example usage of `eleventyNavigation`
- `src/content/pages/_template.txt` — canonical page frontmatter example
- `src/admin/config.yml` — Decap CMS collections and media paths
- `netlify.toml` — deploy command and publish directory

## Practical examples (copy-paste)
- Add a new page: copy `src/content/pages/_template.txt`, update `title`, `permalink`, and `eleventyNavigation.key`.
- Add a new stylesheet: add/import into `src/assets/less/root.less` (or modular less files under `src/assets/less/`), then run `npm run build:less` or `npm start` for live compilation.
- Add a new JS entry: create `src/assets/js/<name>.js` — Eleventy+esbuild will bundle it to `public/assets/js/` on build.

## Troubleshooting tips
- If a change doesn't appear: ensure `npm start` is running; inspect `public/` for the generated output to confirm build results.
- If JS bundling doesn't run: confirm the file lives under `src/assets/` and check Eleventy's server log for esbuild errors.
- If production sizing or minification differs: `isProduction` toggles minifier + esbuild minify (run `npm run build` to simulate prod exactly).

## Pull request guidance for AI agents
- Make edits in `src/` (content, assets, includes, config). Never directly edit `public/`.
- Update `README.md` when adding or changing developer-facing workflows.
- If adding build steps or env vars, also update `netlify.toml` (if deploy behavior changes) and `package.json` scripts.

---
If anything here is unclear or you'd like more examples (e.g., common frontmatter patterns or esbuild flags), tell me which area to expand and I will iterate.