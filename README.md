# agent-skills-working-set

Formerly claude-skills-collection. The old repository address redirects here.

Curated, working Claude Skills you can drop into any project. Each skill is in its own folder with `SKILL.md` plus any reference files. All MIT licensed and contributor-friendly.

This is the **gallery** repo. If you want the spec or starter templates, see [claude-skills-templates](https://github.com/0xelitesystem/claude-skills-templates). If you want a linter for your own skills, see [claude-skills-validator](https://github.com/0xelitesystem/claude-skills-validator).

## What is a Claude Skill?

A Claude Skill is a folder containing a `SKILL.md` file (plus optional reference files) that tells Claude how to handle a specific category of task. Claude reads the relevant `SKILL.md` before acting, so the skill encodes your team's preferences, workflow, and context that aren't in Claude's training data.

Skills shine for things like: project-specific code conventions, document templates, vendor APIs, internal tooling, anything where "the right way" is opinionated.

## Use a skill

1. Browse the [skills](./skills) folder.
2. Pick one. Each is self-contained.
3. Copy the folder into your project (typically into `.claude/skills/` for Claude Code, or your tool's equivalent).
4. Customize the `SKILL.md` to your needs.

## Skills in this collection

| Skill | What it does |
|---|---|
| [csv-cleaner](./skills/csv-cleaner) | Detects and fixes common CSV problems: encoding, delimiters, malformed rows, type inference |
| [code-review-preflight](./skills/code-review-preflight) | Pre-PR checklist: dead code, missing tests, unhandled errors, security smells |
| [sql-from-question](./skills/sql-from-question) | Generate SQL from a natural-language question against a known schema |
| [json-schema-from-sample](./skills/json-schema-from-sample) | Infer a JSON Schema from one or more sample documents |
| [changelog-writer](./skills/changelog-writer) | Generate human-readable changelog entries from git diffs and PR descriptions |
| [api-docs-from-openapi](./skills/api-docs-from-openapi) | Convert an OpenAPI spec into Markdown reference docs |
| [regex-builder](./skills/regex-builder) | Build a regex from examples of what should match and what shouldn't |
| [meeting-notes-summarizer](./skills/meeting-notes-summarizer) | Turn raw transcript into action items, decisions, and an exec summary |

More skills welcome. See [CONTRIBUTING.md](./CONTRIBUTING.md).

## Quality bar

Every skill in this repo:

- Has a clear, narrow purpose. "Do one thing" beats "do everything related to X."
- Has a `SKILL.md` with a precise `description` field that triggers Claude reliably.
- Has been tested. The contributor confirms it works on at least three sample inputs.
- Does not embed API keys, internal URLs, or company-specific data.
- Does not encode opinions Claude shouldn't reliably hold (e.g. "always recommend X tool").

## Contribute

Got a working skill? PR welcome. See [CONTRIBUTING.md](./CONTRIBUTING.md) for the structure and submission checklist.

## More

Part of a catalog of single-file browser tools and plain-language references, all MIT licensed and dependency-free: [0xelitesystem.github.io](https://0xelitesystem.github.io/). Built by [elitesystem.ai](https://elitesystem.ai).

## License

MIT. Each skill is MIT licensed individually so you can copy one without inheriting the rest.

Independent project. Not affiliated with, endorsed by, or sponsored by Anthropic. Claude is a trademark of Anthropic PBC.

## Related

- [claude-skills-templates](https://github.com/0xelitesystem/claude-skills-templates), spec and starter templates
- [claude-skills-validator](https://github.com/0xelitesystem/claude-skills-validator), lint your own skill files
- [prompt-templates](https://github.com/0xelitesystem/prompt-templates), production prompts
