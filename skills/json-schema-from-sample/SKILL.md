---
name: json-schema-from-sample
description: Use when the user wants to generate a JSON Schema from one or more sample JSON documents. Triggers on "create a schema for this", "generate JSON Schema", "validate this shape", "infer types from this JSON", or when the user pastes JSON and asks how to validate similar documents.
---

# JSON Schema from Sample

Generate a JSON Schema (Draft 2020-12 by default) from sample documents.

## Process

1. **Get samples.** Ideally multiple. One sample produces a schema that's too tight (every field is required, every enum has one value). Ask for 2-5 samples that represent the variation you expect to see.

2. **Confirm schema draft version.** Default to Draft 2020-12. Common alternatives:
   - Draft 7: still widely used, supported by older tooling
   - Draft 4: legacy, only if explicitly required

3. **Walk the samples to infer:**
   - **Types per field.** If a field is sometimes a string and sometimes null, type is `["string", "null"]`. If sometimes string sometimes number, that's worth flagging to the user as likely a bug.
   - **Required fields.** Only fields present in ALL samples are required. Mention which fields varied so the user can decide if the variation is real or coincidental.
   - **Enums.** If a field has a small set (≤10) of distinct string values across all samples, suggest making it an enum. Ask before committing, the samples may not have exhausted the values.
   - **Patterns.** If string values match a clear pattern (UUID, email, ISO date, semver), use the standard `format` keyword (`uuid`, `email`, `date-time`) or a `pattern` regex. Don't over-fit.
   - **Number ranges.** Use `minimum`/`maximum` only if the samples suggest a meaningful bound (e.g. age 0-120). Don't add bounds based on coincidental sample values.
   - **Array item type.** If an array always contains the same shape, use that as `items`. If items are mixed, use `prefixItems` or `oneOf`.
   - **Nested objects.** Recurse with the same logic.

4. **Output the schema with comments explaining choices.** Use `description` fields liberally. JSON Schema doesn't allow `//` comments, so put rationale in `description`.

5. **Below the schema, write a short audit:**

```
Audit:
- 12 fields total
- 8 required, 4 optional (only present in some samples)
- 2 enum candidates: status, priority, confirm complete list
- 1 type ambiguity: user_id is sometimes string, sometimes integer (likely a bug?)
- 0 format guesses below 80% confidence
```

## What NOT to do

- **MUST NOT** add `"additionalProperties": false` by default. It locks down the schema and breaks forward compatibility. Add it only if the user asks.
- **MUST NOT** infer regex patterns from a single sample value. Patterns need at least 3 examples.
- **MUST NOT** use `"format": "X"` for non-standard formats. Only the spec-defined formats: `date-time`, `date`, `time`, `email`, `uuid`, `uri`, `ipv4`, `ipv6`, `hostname`. For others, use `pattern`.
- **MUST NOT** silently drop fields that appear in only one sample. Make them optional and report which ones.

## Common pitfalls to call out to the user

- "All samples have a positive `count`. I made `minimum: 0`. Is `count` ever negative?"
- "Samples never have `null` values. Should null be allowed for any field?"
- "Field `id` is integer in some samples, string in others. This is usually a bug. Which is correct?"

## Output destinations

If samples are large or the schema is going into a project, ask whether to write to a file (`schema.json`) or just paste inline. Default to inline for ≤100 lines, file for larger.
