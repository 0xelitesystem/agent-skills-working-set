---
name: api-docs-from-openapi
description: Use when the user wants to convert an OpenAPI (Swagger) spec into Markdown reference documentation. Triggers on "generate docs from this OpenAPI", "convert this swagger to markdown", "make API reference from spec", or when the user shares an OpenAPI 3.x file and asks for human-readable docs.
---

# API Docs from OpenAPI

Convert an OpenAPI 3.x spec into clean Markdown reference docs.

## Process

1. **Confirm the spec.** Accept either:
   - Pasted YAML or JSON
   - A file path
   - A URL (only fetch if user explicitly says to)
   
   Verify it's OpenAPI 3.x. If 2.0 (Swagger), ask whether to upgrade conceptually or stop and recommend a converter.

2. **Top-of-doc structure:**
   ```
   # {info.title}
   
   {info.description}
   
   **Version:** {info.version}
   **Base URL:** {servers[0].url}
   
   ## Authentication
   {auth section based on securitySchemes}
   
   ## Endpoints
   ...
   ```

3. **Group endpoints by tag.** If no tags, group by path prefix (`/users/*`, `/orders/*`).

4. **For each endpoint:**

   ```markdown
   ### `{METHOD} /path`
   
   {summary}
   
   {description, if more detailed than summary}
   
   **Path parameters**
   | Name | Type | Required | Description |
   |---|---|---|---|
   | id | string | yes | The user ID |
   
   **Query parameters**
   (only if any; same table format)
   
   **Request body** (if any)
   - Content-Type: `application/json`
   - Schema: `User` (link to schema section)
   
   ```json
   {sample request body}
   ```
   
   **Responses**
   
   `200 OK` -- {description}
   ```json
   {sample response body}
   ```
   
   `400 Bad Request` -- {description}
   
   `404 Not Found` -- {description}
   ```

5. **Schemas section at the end.** For each named schema:

   ```markdown
   ### `User`
   
   {description}
   
   | Field | Type | Required | Description |
   |---|---|---|---|
   | id | string (uuid) | yes | Unique identifier |
   | email | string (email) | yes | Contact email |
   | created_at | string (date-time) | yes | ISO 8601 timestamp |
   ```

6. **Generate sample request/response bodies from the schemas.** Don't leave `{...}` placeholders. Use realistic example values (`"John Doe"`, valid UUIDs, recent dates).

7. **Cross-link.** Schema references in tables become Markdown links to the schema section.

## What NOT to do

- **MUST NOT** include the entire raw spec in the output. The user wanted prose; they have the spec already.
- **MUST NOT** invent endpoints, parameters, or behavior not in the spec. If the spec is sparse, the docs will be sparse -- note that.
- **MUST NOT** generate sample values that look like real PII (real-looking names, real email addresses, real-looking SSNs).
- **MUST NOT** silently omit deprecated endpoints. Mark them: `### \`POST /old-thing\` (warning) Deprecated`. (Use the word "Deprecated", not an emoji, in the actual output.)

## Output format

Default to a single Markdown file. Offer to split per-tag if the result exceeds ~1000 lines.

## Edge cases

- **Spec has no `examples`**: generate plausible ones from the schema. Mark sample values clearly so the user knows they're synthetic.
- **Spec uses `oneOf` / `anyOf`**: render as "One of:" with sub-bullets. Don't pick one arbitrarily.
- **Spec has 50+ endpoints**: ask if the user wants per-tag files or one big file.
- **Authentication uses multiple schemes**: list each, indicate which endpoints use which.
