---
name: regex-builder
description: Use when the user wants to build a regex from positive and negative examples rather than from a description. Triggers on "build a regex that matches X but not Y", "I have these examples that should match...", "I need a pattern that catches these but skips these", or when the user provides labeled examples and wants a working regex.
---

# Regex Builder

Build a regex from examples. Validate against the user's data before claiming it works.

## Process

1. **Get the examples.** Need both:
   - **Positives**: strings that MUST match
   - **Negatives**: strings that MUST NOT match
   
   If the user gives only positives, ask for negatives. Without negatives, the regex `.*` matches everything; the negatives are what shape the pattern.
   
   Aim for at least 5 positives and 5 negatives. If fewer, warn the user that the regex may be over-fit.

2. **Confirm the engine.** Default to JavaScript / PCRE (they're nearly identical for most patterns). Other engines have important differences:
   - **Python `re`**: no possessive quantifiers, different lookbehind rules
   - **Go `regexp`**: no lookarounds, no backreferences (RE2)
   - **POSIX**: no `\d`, `\w`, `\s` shortcuts
   
   If the user's environment isn't clear from context, ask.

3. **Look at the positives. Identify common structure:**
   - Anchors needed? (Does match start at beginning, end at end, or float?)
   - Fixed prefixes/suffixes
   - Variable parts: are they alphanumeric, digits only, contain specific characters?
   - Length: fixed, range, or unbounded?

4. **Check the negatives. Identify what must NOT be allowed:**
   - Different prefix? Use lookahead negation.
   - Different content type? Tighten character class.
   - Different length? Use quantifier bounds.

5. **Build incrementally. Start permissive, add constraints.**
   - First pass: matches all positives but might match some negatives
   - Each iteration: identify which negatives still match and tighten

6. **Test against EVERY positive and EVERY negative.** Report results in a table:

   ```
   POSITIVE    EXPECTED  GOT
   abc-123       match    match  OK
   abc-456       match    match  OK
   ...
   
   NEGATIVE    EXPECTED   GOT
   abc-12        no-match  no-match  OK
   abcd-123      no-match  match    FAIL -- pattern is too loose, fixing
   ```

   If any test fails, tighten and re-test. Don't deliver a pattern that fails any provided example.

7. **Output:**
   - The pattern (with delimiters appropriate to the engine)
   - A plain-English explanation of each part
   - A note about what the pattern does NOT enforce (e.g. "doesn't validate that the date is a real calendar date, only that the format is correct")
   - The example test results table

## What NOT to do

- **MUST NOT** use overly complex constructs when simple ones suffice. `[A-Za-z0-9]` beats `[\w&&[^_]]` for readability.
- **MUST NOT** use catastrophic-backtracking patterns. Avoid nested quantifiers like `(a+)+` or `(a|a)+`.
- **MUST NOT** assume word boundary `\b` does what the user wants without testing -- `\b` behaves differently around non-ASCII characters.
- **MUST NOT** silently choose case-insensitive matching unless the examples make it obvious. If "ABC" is positive and "abc" is negative, casing matters; preserve it.

## Edge cases

- **Examples are URLs / emails / phone numbers**: standard regexes for these are well-known. Suggest the standard pattern AND the user's example-derived pattern; let them pick.
- **Examples have invisible characters** (zero-width spaces, non-breaking spaces): flag this; show hex of suspicious whitespace.
- **Examples vary widely**: maybe regex isn't the right tool. Suggest alternative parsing.
- **Negative examples are subsets of positive examples** (impossible): tell the user the constraints conflict.
