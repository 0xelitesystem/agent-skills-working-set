---
name: code-review-preflight
description: Use when the user asks for a pre-PR review or self-review of code they've written. Triggers on phrases like "review my changes", "check before I push", "self-review this PR", "what would a reviewer flag here", or when the user pastes a diff or branch summary asking for feedback.
---

# Code Review Preflight

Run the checks a senior reviewer would catch before they catch them.

## Process

1. **Get the diff.** If not provided, ask: which branch vs which base? If the user is in a git repo, prefer `git diff origin/main...HEAD` over `git diff` so context is correct.

2. **Pass through this checklist in order. Report findings as you go, severity-tagged.**

### Critical (blocks PR)

- **Secrets:** any string matching `sk-`, `ghp_`, `xoxb-`, `xoxp-`, `AKIA`, `eyJ` followed by long base64. Even in tests. Even in comments.
- **SQL string concatenation with user input.** Anywhere `f"...{...}"` or `+ user_value +` appears in a SQL context, flag it.
- **Disabled tests** (`.skip`, `xit`, `@pytest.mark.skip`) without an attached issue link or comment explaining why.
- **`TODO` / `FIXME` introduced in this diff.** Old TODOs are someone else's problem. New ones are this PR's.
- **Unhandled `await` rejection paths** (no try/catch, no `.catch()`, no caller that handles it).
- **Logging of sensitive fields:** `password`, `token`, `secret`, `key`, `email` (when in a security-sensitive context) appearing inside log/print calls.

### Important (request changes)

- **Untested code paths.** New function or new branch with no test in the same diff. Exception: pure rendering / pure type changes.
- **Off-by-one risks:** ranges with `<=` vs `<`, slice indexing, pagination math.
- **Mutating shared state** (module-level dicts/lists, default mutable arguments in Python).
- **Magic numbers / strings without a constant** (anything that looks like a configuration value).
- **Catch-all exception handlers** that don't re-raise or log.
- **Imports that are now unused.**
- **Functions over ~50 lines** without an obvious reason; suggest splitting.

### Style (comment, don't block)

- **Naming:** unclear single-letter names outside of loop counters
- **Repeated code blocks** that could be a helper function (3+ occurrences)
- **Comments that restate the code** without adding intent
- **Inconsistent quote style, semicolons, etc.** if the project has a linter, defer to that

3. **Output format:**

```
CRITICAL (N findings)
  file.py:42  Hardcoded API key in test fixture
  ...

IMPORTANT (N findings)
  file.ts:88  New function `parseRange` has no test
  ...

STYLE (N suggestions)
  file.go:23  Consider extracting magic number 86400 as `secondsPerDay`
  ...

OK to merge once Critical and Important are addressed.
```

## What NOT to do

- **MUST NOT** rewrite the code unsolicited. List findings; let the author decide.
- **MUST NOT** invent issues to look thorough. If the diff is clean, say so.
- **MUST NOT** repeat lints the project's linter already catches (assume CI runs them).
- **MUST NOT** comment on style preferences that are subjective unless the project has an explicit style guide.

## Calibration

A 100-line PR usually has 2-5 Important findings. If you're returning 20+ findings on a 100-line PR, you're being noisy. Reduce.
