# Open Source Contributions: Google Lighthouse

**Project:** [GoogleChrome/lighthouse](https://github.com/GoogleChrome/lighthouse) — Google's automated tool for auditing web page quality (performance, accessibility, SEO, best practices). Used industry-wide, including inside Chrome DevTools and PageSpeed Insights.

---

## Contribution 1 — Display plugins in the report footer

**Issue:** [#9934 — Display plugins in footer of report](https://github.com/GoogleChrome/lighthouse/issues/9934)
**Branch:** `fix/plugin-footer-display` on [NickNojiri/lighthouse](https://github.com/NickNojiri/lighthouse) (fork)
**Pull Request:** _pending — link to be added once opened against GoogleChrome/lighthouse_

### The problem

Lighthouse supports third-party plugins that add extra audit categories to a report (e.g. `lighthouse-plugin-publisher-ads`). When a report was generated using one or more plugins, the report footer gave no indication of which plugins ran — a user reviewing a report had no way to tell it included non-standard, third-party checks.

### The fix

Added a `Plugins: <names>` entry to the report footer's metadata block, shown only when the report includes at least one plugin category (detected via the existing `lighthouse-plugin-` category ID prefix convention already used elsewhere in the codebase for the plugin gauge badge). Reports with no plugins render exactly as before.

**Files touched:**
- `report/renderer/report-renderer.js` — collect plugin category IDs and append the footer meta item
- `report/renderer/report-utils.js` — new localizable `runtimeSettingsPlugins` UI string
- `report/assets/styles.css` — new `.lh-report-icon--plugin` icon, reusing the existing plugin puzzle-piece SVG
- `report/test/renderer/report-renderer-test.js` — unit tests for both the plugin and no-plugin cases
- Generated/regenerated via the project's own tooling: `components.js` (bundled report template), locale files (`en-US.json`, `en-XL.json`), and sample report fixtures (`sample_v2.json`, `sample-flow-result.json`)

**The core change**, in `report/renderer/report-renderer.js`:

```js
const pluginNames = Object.keys(report.categories)
  .filter(categoryId => ReportUtils.isPluginCategory(categoryId));
if (pluginNames.length > 0) {
  metaItems.push(['plugin',
    `${Globals.strings.runtimeSettingsPlugins}: ${pluginNames.join(', ')}`]);
}
```

Deliberately small: it reuses the codebase's existing plugin-detection convention rather than introducing new plumbing, adds nothing to the LHR schema, and cannot affect reports that don't use plugins. Verified visually in both light and dark themes via the project's sample-report build.

### Verification

- All 273 existing + new tests in `report/test` pass, including the accessibility (axe) render test
- `eslint` clean
- `tsc --build` (full project type-check) clean
- `yarn i18n:checks` (CI's localization-strings-collected check) passes

---

## Contribution 2 — Smoke test coverage for CSS nesting

**Issue:** [#14718 — Confirm correct handling of CSS nesting](https://github.com/GoogleChrome/lighthouse/issues/14718)
**Branch:** `tests/css-nesting-coverage` on [NickNojiri/lighthouse](https://github.com/NickNojiri/lighthouse) (fork)
**Pull Request:** _pending — link to be added once opened against GoogleChrome/lighthouse_

### The problem

When Chrome shipped CSS nesting, the Lighthouse team asked for confirmation that their use of the Chrome DevTools Protocol's CSS rule-usage tracking still behaved correctly, plus smoke tests to prevent regressions. The risk: Lighthouse's `unused-css-rules` audit sums used-rule byte ranges with no overlap handling, and nested rules live *inside* their parent rule's source range — a change in how Chrome reports those ranges could silently double-count usage or misreport waste.

### The contribution

Extended the byte-efficiency smoke test fixture with a nested-CSS stylesheet generator (parent rules containing `& .child` nested rules) and two generated stylesheets — one fully used, one fully unused. New expectations assert, against a real Chrome run:

- the unused nested stylesheet is reported at >95% wasted bytes (nested rules are counted as unused), and
- the fully-used nested stylesheet is **not** reported (nested rule usage is tracked; no false waste)

so a regression in nested-rule handling fails the smoke suite in either direction. The empirical runs confirmed current behavior is correct: unused nested rules count at exactly 100% waste and used nested rules produce none, with no double-counting.

**Files touched:**
- `cli/test/fixtures/byte-efficiency/tester.html` — nested-CSS stylesheet generator + two test stylesheets
- `cli/test/smokehouse/test-definitions/byte-efficiency.js` — assertions for the nested-CSS cases

### Verification

- Full `byte-efficiency` smoke suite passes (13/13 assertions) against real Chrome, plus a pre-change baseline run to isolate the effect of the change
- `eslint` clean

---

## Process notes

- Signed Google's [Individual Contributor License Agreement](https://developers.google.com/open-source/cla/individual), required for all Lighthouse contributions
- Followed the project's conventional-commit PR title format (enforced by a commitlint bot), e.g. `report: display plugins in footer`, `tests(smoke): add coverage for CSS nesting in unused-css-rules`
- Worked from a personal fork per the project's standard external-contributor workflow
