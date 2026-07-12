# Open Source Contribution: Google Lighthouse

**Project:** [GoogleChrome/lighthouse](https://github.com/GoogleChrome/lighthouse) — Google's automated tool for auditing web page quality (performance, accessibility, SEO, best practices). Used industry-wide, including inside Chrome DevTools and PageSpeed Insights.

**Issue:** [#9934 — Display plugins in footer of report](https://github.com/GoogleChrome/lighthouse/issues/9934)

**Branch:** `fix/plugin-footer-display` on [NickNojiri/lighthouse](https://github.com/NickNojiri/lighthouse) (fork)

**Pull Request:** _pending — link to be added once opened against GoogleChrome/lighthouse_

## The problem

Lighthouse supports third-party plugins that add extra audit categories to a report (e.g. `lighthouse-plugin-publisher-ads`). When a report was generated using one or more plugins, the report footer gave no indication of which plugins ran — a user reviewing a report had no way to tell it included non-standard, third-party checks.

## The fix

Added a `Plugins: <names>` entry to the report footer's metadata block, shown only when the report includes at least one plugin category (detected via the existing `lighthouse-plugin-` category ID prefix convention already used elsewhere in the codebase for the plugin gauge badge). Reports with no plugins render exactly as before.

**Files touched:**
- `report/renderer/report-renderer.js` — collect plugin category IDs and append the footer meta item
- `report/renderer/report-utils.js` — new localizable `runtimeSettingsPlugins` UI string
- `report/assets/styles.css` — new `.lh-report-icon--plugin` icon, reusing the existing plugin puzzle-piece SVG
- `report/test/renderer/report-renderer-test.js` — unit tests for both the plugin and no-plugin cases
- Generated/regenerated via the project's own tooling: `components.js` (bundled report template), locale files (`en-US.json`, `en-XL.json`), and sample report fixtures (`sample_v2.json`, `sample-flow-result.json`)

## Verification

- All 273 existing + new tests in `report/test` pass, including the accessibility (axe) render test
- `eslint` clean
- `tsc --build` (full project type-check) clean
- `yarn i18n:checks` (CI's localization-strings-collected check) passes

## Process notes

- Signed Google's [Individual Contributor License Agreement](https://developers.google.com/open-source/cla/individual), required for all Lighthouse contributions
- Followed the project's conventional-commit PR title format (enforced by a commitlint bot): `report: display plugins in footer`
- Worked from a personal fork per the project's standard external-contributor workflow
