# AGENTS.md

Instructions for AI coding agents working in this repository.

## Project Overview

- MYNTFP (`MYNTFP: Material You New Tab for Phones`) is a browser extension that replaces the browser's New Tab page with a personalized, Material You–inspired page (themes, wallpaper, clock, weather, search, shortcuts, todos, bookmarks, AI tools, quotes, translations).
- Plain HTML/CSS/vanilla JavaScript — no framework, no bundler, no package manager, no lockfile. There is no `package.json`, and no test, lint, typecheck, or CI workflow setup in the repo.
- Layout: `index.html` (single page + script lists), `style.css` (all styles), `scripts/*.js` (one vanilla-JS feature module per concern), `locales/*.js` (per-language translation objects), plus static assets (`favicon/`, `fonts/`, `images/`, `svgs/`, `docs/`, `tools/`).

## Commands

- No automated commands exist (nothing to build, lint, or test).
- Verify changes manually by loading the extension:
  - Chromium-based browsers (Chrome, Edge, Brave, Opera): open `chrome://extensions`, enable Developer mode, **Load unpacked**, select the repo folder (uses `manifest.json`).
  - Firefox / Zen: swap `manifest.json` ↔ `manifest(firefox).json` (delete + rename) and load via `about:debugging#/runtime/this-firefox` → **Load Temporary Add-on**. Swap back before committing.
  - Opera: new tab override is unsupported; use the startup-page workaround in `README.md`.

## Architecture Notes

- Script wiring lives in `index.html` under the `<!-- SCRIPTS -->` and `<!-- LANGUAGES SCRIPTS -->` sections. Adding a script or locale requires editing those lists.
- Load order is load-bearing:
  - `scripts/preload.js` is a synchronous `<script>` in `<head>` running as an IIFE before `<body>` exists; it applies saved theme/CSS custom properties before first paint (FOUC prevention) and defines `window.ThemeHelpers`.
  - All other `scripts/*.js` load with `<script defer>`.
  - Locale scripts load after core scripts; `scripts/languages.js` provides the global `translations` map (plus `menuWidths`, `numberMappings`, `rtlLanguages`) used across modules.
- Feature modules share top-level `const`/`function` globals (e.g. `translations`, `isDesktop` in `scripts/browser-utils.js`). Do not rename or shadow shared globals.
- Persistence is browser `localStorage` under well-known keys (e.g. `selectedTheme`, `preferredTheme`, `customThemeColor`, `bgOpacity`, `weatherApiKey`, `hideTips`). Reuse existing keys; do not rename them casually.

## Conventions

- New feature: add module in `scripts/`, include the required copyright header (see `scripts/README.md`), register it in the `<!-- SCRIPTS -->` section of `index.html`, and add an entry under `## [Unreleased]` in `CHANGELOG.md`.
- New locale: follow the step-by-step guide in `locales/README.md` (covers `locales/*.js`, `index.html`, `scripts/languages.js`, `scripts/clock.js`, root `README.md`, and `tools/languagesAnalysis.html`).
- Changelog: Keep a Changelog format with categories `Added`, `Changed`, `Improved`, `Removed`, `Fixed`, `Localized`, `Other`.
- Keep PRs focused (one feature/fix); don't mix unrelated refactors into a feature PR.
- Don't reformat code or apply editor "Format Document" — it makes diffs hard to review.

## Verification Notes

- No automated tests/lint/build exist; manual browser testing is the verification path. Load the unpacked extension and exercise the changed behavior; check Chromium and Firefox where relevant.

## Gotchas

- `manifest.json` is MV3 (Chromium) while `manifest(firefox).json` is the Firefox variant; swapping is required for Firefox temporary loading.
- `preload.js` must remain a synchronous `<script>` in `<head>` before the `<body>` exists.
- Opera does not support overriding the New Tab page.

## References

- `README.md`, `CONTRIBUTING.md`, `CHANGELOG.md`, `CODE_OF_CONDUCT.md`
- `scripts/README.md`, `locales/README.md`
- `.github/pull_request_template.md`, `.github/ISSUE_TEMPLATE/`
- `docs/`, `tools/languagesAnalysis.html`
