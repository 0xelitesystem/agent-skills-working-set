---
name: changelog-writer
description: Use when the user wants to generate human-readable changelog entries. Triggers on "write a changelog", "summarize this release", "what changed since v1.2", "release notes for this PR batch", or when the user provides a list of git commits, PRs, or a diff between two versions and wants prose output.
---

# Changelog Writer

Turn raw git history into a changelog humans actually read.

## Process

1. **Get the input.** Accept any of:
   - List of commit messages (`git log v1.0..HEAD --oneline`)
   - List of PR titles + descriptions
   - A merged diff between two tags/branches
   - A free-form list the user dictates
   
   If only commit subjects are provided, ask whether bodies/PRs are available, they're often where the "why" lives.

2. **Get the audience.** Default to "users of the software" (i.e. external changelog). Ask if it's:
   - **External**: end users, customers. Skip refactors and internal changes. Lead with user-visible impact.
   - **Internal**: engineering team. Include refactors, infra changes, dependency bumps.
   - **Hybrid**: both. Use sections.

3. **Group entries into these sections in order:**
   - **Breaking changes** (anything that requires consumer action)
   - **Added** (new features, new endpoints, new capabilities)
   - **Changed** (behavior changes that aren't breaking)
   - **Fixed** (bugs)
   - **Deprecated** (still works, will go away)
   - **Removed** (gone)
   - **Security** (CVEs, fixes for security issues)
   - **Internal** (only if audience is internal/hybrid)

   Skip sections with no entries; don't write empty headings.

4. **Write each entry as user-facing impact, not implementation.**
   - Bad: "Refactored the AuthService to use DI"
   - Good: "Login flow now reports network errors instead of hanging"
   
   - Bad: "Bumped axios to 1.6.2"
   - Good: (skip for external; for internal: "axios 1.5 → 1.6.2 (security patch)")

5. **Format:**
   - Use Markdown
   - One bullet per entry
   - Past tense ("Added", "Fixed") in the section heading; imperative or descriptive in the bullet
   - Reference PR/issue numbers in parens at the end of each bullet if available
   - Group related bullets together within a section

6. **Top of file: version + date heading.**
   ```
   ## 2.4.0 - 2026-05-15
   ```

7. **Below the changelog, list anything you skipped and why.** "Skipped 12 dependency bumps and 3 internal refactors." Lets the user catch you if you skipped something they wanted included.

## What NOT to do

- **MUST NOT** invent context. If a commit message says "fix bug", say "Fixed a bug", don't fabricate what the bug was.
- **MUST NOT** include merge commits ("Merge branch X into Y") as changelog entries.
- **MUST NOT** be vague. "Various improvements" is useless. Either name the improvement or omit.
- **MUST NOT** include version-bump commits, lint-config commits, formatting-only commits, or CI-only commits in an external changelog.
- **MUST NOT** write more than one entry per atomic change. If a feature shipped via 8 commits, that's one bullet.

## Tone

- External: clear, friendly, no jargon. A new user reading this should understand the impact.
- Internal: dense, technical, assumes context. Engineers reading their own changelog don't need explanations.

## Edge cases

- **No semantic versioning info available**: ask what version this is for. Don't guess.
- **Commits with no clear category**: put under "Changed" or ask the user.
- **Reverts**: list as "Reverted X (introduced in Y)" with the original version.
- **Long change list (>30 entries)**: ask if you should aggressively trim or include everything. For external, almost always trim.
