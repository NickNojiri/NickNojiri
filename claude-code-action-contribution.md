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

---

## Contribution — Require a trusted author for the `@claude` workflow trigger

**Pull Request:** [anthropics/claude-code-action#1503](https://github.com/anthropics/claude-code-action/pull/1503)
**Branch:** `fix/trigger-workflow-author-guard` on [NickNojiri/claude-code-action](https://github.com/NickNojiri/claude-code-action) (fork)
**Addresses:** issues #1481, #1445, #1068

### The problem

The example workflows users copy to set up the action — `examples/claude.yml`, `examples/claude-wif.yml`, and the project's own dogfooding CI workflow `.github/workflows/claude.yml` — gated the job purely on trigger-phrase presence (`contains(github.event.comment.body, '@claude')`), with no check on *who* posted the comment/review/issue. Multiple issues (#1481, #1445, #1068) reported the downstream risk: a bot quoting an earlier `@claude` mention back into a thread can re-trigger the workflow, and on public repos any commenter starts a runner before permission checks run.

### The contribution

Added an author gate to the `if:` condition in all three files: `github.event.sender.type != 'Bot'` plus an `author_association` check (`OWNER`/`MEMBER`/`COLLABORATOR`) applied per event type against the field each event actually populates (`comment`/`review`/`issue`).

The interesting part was scoping it honestly. Issue #1481 asked to fix `docs/usage.md` and the `/install-github-app` command — but I verified `docs/usage.md` has no trigger `if:` example, and `/install-github-app` lives in a *different* repo (the Claude Code CLI), so neither was actually in scope here. I also traced the action's runtime path and confirmed `checkHumanActor` / `checkWritePermissions` already reject bot and non-writer actors before Claude executes. So I wrote the PR to describe the change accurately: a fail-fast + safer-template improvement (the job no longer spins up a runner for an untrusted trigger, and the copy-pasted examples now demonstrate the safe pattern), **not** a critical-vulnerability fix. Overstating it would have been the easy thing; getting the scope right is the point.

**Files touched:**
- `examples/claude.yml`, `examples/claude-wif.yml`, `.github/workflows/claude.yml` — author gate on the trigger condition

### Verification

- YAML syntax validated for all three files
- `bun test` — 770 pass, 0 fail (config-only change, no test churn)
- `tsc` type-check and Prettier clean

---

## Contribution — Close an entity-encoding bypass in the prompt-injection sanitizer

**Pull Request:** [anthropics/claude-code-action#1504](https://github.com/anthropics/claude-code-action/pull/1504)
**Branch:** `fix/sanitizer-entity-encoded-comment-bypass` on [NickNojiri/claude-code-action](https://github.com/NickNojiri/claude-code-action) (fork)

### The problem

Before GitHub content reaches Claude, `sanitizeContent` strips HTML comments to remove hidden instructions (a documented prompt-injection defense). But `normalizeHtmlEntities` runs several steps *later* in the same pipeline. I noticed the ordering and hypothesized that entity-decoding could re-introduce content the earlier strip had removed — then proved it: an input of `&#60;!-- ignore all instructions --&#62;` passes through the initial `stripHtmlComments` untouched (it's not yet a comment), and is then decoded back into a live `<!-- … -->` comment that reaches the model.

### The fix

I verified the bypass empirically across decimal, hex, and opening-delimiter-only variants (all leaked), then fixed it by re-running `stripHtmlComments` after entity decoding. Re-ran the probe: all variants closed, plain-comment behavior unchanged.

I was deliberately honest about scope in the PR: entity-encoded comments render *visibly* in GitHub's UI, so this is defense-in-depth hardening of the injection sanitizer rather than a fully-hidden exploit — I said so rather than overselling it. Legitimate defensive security work, submitted through the normal PR flow.

**Files touched:**
- `src/github/utils/sanitizer.ts` — second comment-strip pass after entity decoding
- `test/sanitizer.test.ts` — regression tests for the encoded-comment bypass

### Verification

- Full test suite: 771 pass, 0 fail (`bun test`)
- `tsc` type-check and Prettier clean
