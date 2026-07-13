# Open Source Contribution: Anthropic's Claude Code Action

**Project:** [anthropics/claude-code-action](https://github.com/anthropics/claude-code-action) — Anthropic's official GitHub Action for running Claude Code in CI: it responds to `@claude` mentions on issues and PRs, implements changes, and answers questions about code. TypeScript, ~8k stars.

---

## Contribution — Redact GitHub user-to-server (`ghu_`) tokens

**Pull Request:** [anthropics/claude-code-action#1502](https://github.com/anthropics/claude-code-action/pull/1502)
**Branch:** `fix/redact-ghu-tokens` on [NickNojiri/claude-code-action](https://github.com/NickNojiri/claude-code-action) (fork)

### The problem

Before feeding GitHub content (issue bodies, comments, reviews) to Claude, the action sanitizes it — including redacting any GitHub credentials someone may have pasted. The `redactGitHubTokens` function covered `ghp_` (personal access), `gho_` (OAuth), `ghs_` (installation), `ghr_` (refresh), and `github_pat_` (fine-grained) tokens, but missed **`ghu_`** — GitHub App user-to-server tokens, one of the five [documented GitHub token prefixes](https://github.blog/security/behind-githubs-new-authentication-token-formats/). A `ghu_` token pasted into an issue or PR comment passed through sanitization unredacted into prompt content.

I found the gap by auditing the sanitizer against GitHub's documented token formats, after noticing the project's recent bug reports clustered around string-handling edge cases. A broader fix had been attempted months earlier (PR #1098) but was abandoned by its author before addressing review feedback — so I scoped a minimal, test-covered version of just the uncontroversial missing piece.

### The fix

```ts
// GitHub user-to-server tokens: ghu_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX (40 chars)
content = content.replace(
  /\bghu_[A-Za-z0-9]{36}\b/g,
  "[REDACTED_GITHUB_TOKEN]",
);
```

One pattern, mirroring the four existing 40-character token patterns exactly, plus unit tests covering both a plain occurrence and the `x-access-token:ghu_…@github.com` git-credential URL form.

**Files touched:**
- `src/github/utils/sanitizer.ts` — the `ghu_` redaction pattern
- `test/sanitizer.test.ts` — regression tests

### Verification

- Full test suite: 771 tests pass, 0 failures (`bun test`)
- `tsc` type-check clean
- Prettier formatting clean

### Process notes

- Scoped deliberately after studying why the prior attempt (#1098) stalled: review flagged scope creep and missing tests, so this PR does one documented thing with tests
- Followed the project's conventional-commit format (`fix(sanitizer): …`)
- Worked from a personal fork per the standard external-contributor workflow
