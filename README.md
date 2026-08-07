# Hi, I'm Nick Nojiri 👋

Software developer focused on web technologies, currently contributing to open source.

## 🔭 Open Source

Contributing to **[Google Lighthouse](https://github.com/GoogleChrome/lighthouse)**:

- [PR #17125](https://github.com/GoogleChrome/lighthouse/pull/17125) — surface the plugins used during a run in the report footer (issue [#9934](https://github.com/GoogleChrome/lighthouse/issues/9934)). Includes i18n, dark-mode-aware iconography, and unit tests, verified against the project's full lint/type-check/test/i18n CI suite.
- [PR #17127](https://github.com/GoogleChrome/lighthouse/pull/17127) — smoke test coverage proving CSS nesting is handled correctly by the `unused-css-rules` audit (issue [#14718](https://github.com/GoogleChrome/lighthouse/issues/14718)), with regression protection in both directions (unused nested rules counted, used nested rules not falsely flagged).
- **[paulirish/lh-scorecalc #55](https://github.com/paulirish/lh-scorecalc/pull/55) — ✅ merged** — fix stale scores when switching device type in the Lighthouse Score Calculator (issue [#16609](https://github.com/GoogleChrome/lighthouse/issues/16609)); root-caused via Playwright since the tool has no test harness, and fixed at its true upstream source repo after a maintainer redirected my initial PR.
- [PR #17131](https://github.com/GoogleChrome/lighthouse/pull/17131) — clamp used-byte accounting in the unused-CSS audit so overlapping/duplicate coverage ranges can't report negative wasted bytes (relates to [#14718](https://github.com/GoogleChrome/lighthouse/issues/14718)); with regression tests.

→ [Read the full write-up](./lighthouse-contribution.md)

Contributing to **[Anthropic's Claude Code Action](https://github.com/anthropics/claude-code-action)**:

- **[PR #1502](https://github.com/anthropics/claude-code-action/pull/1502) — ✅ merged** — closed a security gap in the content sanitizer: GitHub App user-to-server (`ghu_`) tokens, the one missing prefix from GitHub's documented token formats, now get redacted before issue/PR content reaches the model. Includes regression tests; full suite green (771 tests).
- [PR #1503](https://github.com/anthropics/claude-code-action/pull/1503) — added a trusted-author gate to the `@claude` trigger in the project's example and CI workflows (addresses issues #1481, #1445, #1068), so untrusted or bot triggers fail fast before a runner starts.
- [PR #1504](https://github.com/anthropics/claude-code-action/pull/1504) — closed an entity-encoding bypass in the prompt-injection sanitizer: `&#60;!-- … --&#62;` survived the comment-strip pass and was later decoded back into a live HTML comment reaching the model. Fixed by re-stripping after decoding; regression tests added.
- [PR #1524](https://github.com/anthropics/claude-code-action/pull/1524) — fixed `--permission-mode` in `claude_args` being silently ignored in headless runs: the flag reached the CLI but the required interlock only the SDK emits was missing, so `bypassPermissions` downgraded to `default` and the run failed while reporting success. Extracts the mode into the SDK's first-class option (issue #1512).
- **[PR #1539](https://github.com/anthropics/claude-code-action/pull/1539) — ✅ merged** — found by reading the code and filed as issue #1527: an empty `{{description}}` (emoji/CJK/punctuation-only titles) collapsed a `branch_name_template` to `claude//123`, which fails branch-name validation and aborted the whole run. Collapses empty path segments; regression tests added.

→ [Read the full write-up](./claude-code-action-contribution.md)

## 🛠 Projects

- **[relay](https://github.com/NickNojiri/relay)**
- **[WindowDash](https://github.com/NickNojiri/WindowDash)**
- **[SocialAgent-Team10](https://github.com/NickNojiri/SocialAgent-Team10)**
- **[portfolio](https://github.com/NickNojiri/portfolio)**
- **freeCodeCamp back-end certification** — a series of Node.js/Express microservice and API projects ([issue tracker](https://github.com/NickNojiri/FCC-issue-tracker), [personal library](https://github.com/NickNojiri/FCC-personal-library), [URL shortener](https://github.com/NickNojiri/FCC-URL-shortener-microservice), [and more](https://github.com/NickNojiri?tab=repositories&q=FCC))

## 📫 Contact

- Email: [nickthe20@gmail.com](mailto:nickthe20@gmail.com)
- GitHub: [@NickNojiri](https://github.com/NickNojiri)
