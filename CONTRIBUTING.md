# Contributing a Skill

Thanks for considering a contribution. Here's how to add one.

## Structure

```
skills/
  your-skill-name/
    SKILL.md          (required)
    reference.md      (optional, more docs Claude can pull)
    examples/         (optional, sample inputs/outputs)
```

The folder name must match `^[a-z][a-z0-9-]+$` (lowercase, hyphens, no underscores).

## SKILL.md format

Required frontmatter:

```yaml
---
name: your-skill-name
description: A precise one-sentence description of when Claude should use this skill. Should include the trigger conditions explicitly.
---
```

Body: instructions for Claude. Be opinionated and direct. Use `**MUST**`, `**MUST NOT**`, `**SHOULD**` to signal hard rules vs guidance.

## Quality checklist

Before submitting, confirm:

- [ ] Skill has been tested on at least 3 sample inputs and works as intended
- [ ] `description` reads cleanly and would let Claude pick this skill from a list of similar ones
- [ ] Body uses imperative voice ("Read the file" not "You should read the file")
- [ ] No API keys, internal URLs, or proprietary data
- [ ] No opinions about specific commercial products unless they're objectively the standard (e.g. "use Postgres" is okay; "use Vendor X CRM" is not)
- [ ] Validated through [claude-skills-validator](https://github.com/0xelitesystem/claude-skills-validator)
- [ ] Added a row to the table in `README.md`

## PR checklist

Title format: `add skill: your-skill-name`

PR description should include:

1. What the skill does in one sentence
2. The 3 sample inputs you tested it on (paste the inputs and outputs, or link to them)
3. Why this is different from existing skills in the collection

## What gets rejected

- Skills that just rephrase Claude's defaults
- Skills that hardcode proprietary product knowledge
- Skills that try to do 5 things at once (split them)
- Skills with vague trigger descriptions like "use this for various tasks"

## Updating an existing skill

Same flow. Title format: `update skill: skill-name`. Explain what changed in the PR description.

## Removing a skill

Open an issue first explaining why (e.g. obsolete, replaced by something better).
