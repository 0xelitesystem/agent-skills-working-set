---
name: sql-from-question
description: Use when the user asks a question in plain English that requires querying a known database. Triggers when the user provides a schema (DDL, ER diagram description, or table list) and asks "how many...", "what is the average...", "show me users who...", or any analytical question that maps to SQL.
---

# SQL from Question

Generate correct SQL from a natural-language question against a known schema.

## Process

1. **Confirm you have the schema.** If not provided, ask for it. Accept any of:
   - DDL (`CREATE TABLE ...`)
   - Pasted output of `\d table_name` or `DESCRIBE table_name`
   - A list like "users (id, name, email, created_at), orders (id, user_id, total, ...)"
   - Reference to a previously-shared schema in this conversation
   
   If the schema is incomplete (e.g. user mentions "the customers table" but didn't share its columns), ask before guessing.

2. **Confirm the SQL dialect.** Default to PostgreSQL. Ask only if the schema or question uses dialect-specific features and you're unsure (e.g. `LIMIT` vs `TOP`, `CONCAT` vs `||`, window function syntax).

3. **Restate the question in your own words before writing SQL.** This catches misunderstandings before they become wrong queries. Example:
   > You're asking: "Of users who signed up in the last 30 days, how many placed at least one order with a total over $100?" Confirm or correct.

4. **Write the query. Follow these conventions:**
   - Use lowercase keywords (`select`, `from`, `where`) for readability in this conversation; switch to UPPERCASE only if the user's existing code uses uppercase.
   - Always alias tables in joins with short, meaningful names (`users u`, `orders o`).
   - Always qualify columns with table aliases when more than one table is involved.
   - Use `WITH` (CTEs) when the query has 3+ logical steps. Inline subqueries only for trivial cases.
   - Format multi-line. Never one-line a query with more than 2 clauses.
   - Use explicit `JOIN ... ON ...`, never implicit comma joins.
   - For aggregates, name every output column (`count(*) as user_count`, not bare `count(*)`).

5. **Add a comment explaining intent at the top of complex queries.** One sentence.

6. **Below the query, list assumptions you made.** Examples:
   - "Assumed `created_at` is in UTC."
   - "Assumed soft-deleted users have `deleted_at IS NOT NULL` and should be excluded."
   - "Assumed orders.status of 'completed' is the relevant filter."

7. **Suggest one simple variation.** Examples: "If you want this broken down by month, change X to Y." This catches the common follow-up before they ask.

## What NOT to do

- **MUST NOT** write `SELECT *` in production-style queries. List columns explicitly. Exception: ad-hoc exploration queries the user explicitly asked to keep loose.
- **MUST NOT** invent column or table names. If the schema doesn't have what the question needs, say so.
- **MUST NOT** use deprecated syntax (`(+)` joins in Oracle, `RIGHT OUTER JOIN` style if `LEFT OUTER JOIN` would be clearer).
- **MUST NOT** silently change semantics. If the user asks for "active users" and "active" isn't defined in the schema, ask what defines active.

## Edge cases

- **Date/time questions** without timezone info: assume UTC, state the assumption.
- **Questions with "or" or "and"** that could be parsed multiple ways: rephrase the question both ways and ask which is meant.
- **Questions about percentages or ratios**: use `AVG(condition::int)` or `COUNT(...) FILTER (WHERE ...) * 100.0 / COUNT(*)`. Don't truncate integer division.
- **Top-N queries**: use `LIMIT` (or dialect equivalent), not `ROWNUM` or hand-rolled ranking, unless the user wants ties handled (then use `RANK()` or `DENSE_RANK()`).
