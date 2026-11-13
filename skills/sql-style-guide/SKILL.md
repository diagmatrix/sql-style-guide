---
name: sql-style-guide
description: Write, review and reformat SQL following diagmatrix's SQL style guide (left-aligned keywords, 4-space indentation, arguments below SELECT/WHERE/HAVING, aligned AS keywords, uppercase keywords and lowercase functions, snake_case plural tables, CTEs over subqueries, named constraints, and more). Use this skill whenever writing any SQL — queries, views, CTEs, CREATE TABLE statements or migrations — and whenever the user asks to review, lint, check, clean up, tidy, format or "fix the style of" SQL, even if they don't mention the style guide by name.
---

# SQL style guide

This skill applies a specific SQL style guide. The full guide is bundled in `references/`, one file per topic, each rule with an explanation and Bad/Good examples. The checklist below condenses every rule so you can usually work from it alone; open the reference file when a rule's exact scope or an edge case matters.

The guide describes itself as meant "to inform, not dictate": when following a rule would clearly make the SQL worse (efficiency, business constraints, a dialect that doesn't support the construct), deviate and say why. Consistency within a project matters more than any single rule.

## What you might be asked to do

**Write SQL.** Follow the checklist from the start, so the code needs no style pass afterwards.

**Review SQL.** Check the SQL against the checklist and report each problem. Use this format, grouped by rule, most impactful first:

```
### <Rule name> (<reference file>)
- Line <n>: <what is wrong> → <how to fix it>
```

End with the corrected SQL in full, unless the user only asked for a list. If nothing breaks a rule, say so plainly instead of inventing nitpicks.

**Reformat SQL.** Rewrite it to follow the guide **without changing what it does**. Formatting, keyword case, aliases, comment style and the like are safe. Always rewrite boolean comparisons to `IS true` / `IS false` (`= true`, `= false`, bare `flag`, `NOT flag`): in `WHERE`, `HAVING` and `ON` the rows are identical. In a `SELECT` list or `CASE`, a `NULL` flag now yields `false` instead of `NULL`, which is what the guide intends; mention it in your reply when that happens. Some rules change the query's meaning, behaviour or the schema, so don't apply them silently. Mention them as suggestions instead, unless the user asked for them:

- renaming tables, columns or constraints (other code depends on those names; the guide also says replicated data from external sources may keep its original naming)
- `UNION` → `UNION ALL` (it stops removing duplicates)
- moving conditions between `ON` and `WHERE` (only equivalent for `INNER JOIN`, never for `LEFT JOIN`)
- removing `ORDER BY`, turning subqueries into CTEs, splitting CTEs, replacing correlated subqueries
- vendor-specific → ANSI functions (the results can differ, e.g. `date_trunc` returns a date while `extract` returns a number)

When reformatting a file, keep anything that isn't SQL (surrounding code, migration boilerplate, templating like `{{ ref('x') }}` or `:param` placeholders) exactly as it was.

## Rule checklist

### Formatting (`references/formatting.md`)
- Left-align all clause keywords at the same margin. No "river" (right-aligned) keywords.
- Each major clause (`SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, ...) starts its own line.
- `SELECT`, `WHERE` and `HAVING`: every argument on its own line below the keyword, indented one level, **even a single one** (`SELECT` / `    *`).
- `FROM`, `JOIN`, `GROUP BY`, `ORDER BY` (and other clauses): arguments stay on the keyword line (`FROM orders`, `GROUP BY region, country_code`).
- Indent with 4 spaces.
- `ON` goes on the line after its `JOIN`, indented one level, even with one condition. Extra join conditions follow as `AND` lines at the same indentation.
- An opening parenthesis that starts a multi-line construct ends its line; content indented one level; closing parenthesis on its own line, aligned with the line that opened it.
- CTEs: first one on the `WITH` line (`WITH active_customers AS (`), later ones at the same level as `WITH`; body indented one level; `)` on its own line; a blank line between CTEs and after the last one.
- Exactly one space between tokens (around operators, after commas). No padding to align things, except the two alignment rules below.
- Align `AS` keywords in a `SELECT` list: pad expressions so every `AS` sits one space after the longest aliased expression. Columns without alias are left alone.
- In `CREATE TABLE`, align data types one space after the longest column name, and column constraints one space after the longest type. Named table constraints at the end: pad names so the `CHECK (...)` / `FOREIGN KEY (...)` bodies line up.
- No spaces inside parentheses, none between a function name and `(`.
- `AND` / `OR` start the next line, at the same indentation as the first condition.
- A blank line after each `;` between statements.
- Blank lines between logically distinct sections (between CTEs, before the main query, between groups of conditions).
- Trailing commas, never leading commas.
- Lines under 120 characters. To break a long expression, wrap it in parentheses, break after `(`, indent, start each continuation line with its operator, close `)` on its own line. Break inside a function's own parentheses instead of adding new ones. Still unreadable → move it into a CTE.

### Keywords (`references/keywords.md`)
- Reserved words in UPPERCASE (`SELECT`, `AS`, `IS`, `CASE`, ...).
- Full keyword forms instead of abbreviations (`INNER JOIN`, not `JOIN`).
- Prefer ANSI functions over vendor-specific ones when an ANSI equivalent exists (`extract(YEAR FROM x)` over `date_part('year', x)`).
- Functions in lowercase (`count()`, `coalesce()`, `string_agg()`).
- Built-in data types in UPPERCASE (`INTEGER`, `VARCHAR(255)`, `TIMESTAMP`). User-defined types follow naming rules (lowercase snake_case).
- Boolean literals are written `true` / `false` (lowercase), as in `IS true`.

### Naming (`references/naming.md`)
- snake_case; only lowercase letters, digits and underscores; start with a letter; no trailing or double underscores; never quoted identifiers.
- Descriptive, consistent across tables, unambiguous. No type/role prefixes (`tbl_`, `sp_`, `vw_`). Exception: constraint names may carry a type prefix (`chk_`, `fk_`, ...).
- Avoid abbreviations unless the whole team understands them.
- Never use reserved words as names (`group`, `union`, `rank`, `order`, `user`, ...).
- Tables: collective or plural nouns (`staff`, `orders`); singular only for single-row tables. Relationship tables get a real name when one exists (`enrolments`, not `student_courses`).
- Columns: singular (`goal_count`, not `goals`). Don't repeat the table name (`products.name`, not `products.product_name`; avoid `grades.grade`).
- A table's own primary key is `id`; `_id` marks foreign keys and other identifiers (`orders.customer_id`).
- Standard affixes: `_id`, `_status`, `_total`/`total_`, `_name`, `_size`, `_date` (date), `_at` (timestamp), `_addr`/`_address`, `is_`/`has_` (booleans), unit suffixes (`_eur`, `_ms`), `_month`/`_day`.
- Generic columns (`id`, `name`, `type`) selected from a table other than the one the result describes get a `table_column` alias (`schools.name AS school_name`). Columns of the table the result is about keep their name (`customers.id` in a query returning customers).

### Aliasing (`references/aliasing.md`)
- Always write `AS`, for columns and tables.
- Every aggregate or computed column gets an alias (`count(*) AS total_orders`).
- Prefer full table names. Alias only long or ambiguous names (or self-joins), with a meaningful word, never initials or `t1`/`x` (`school_district_areas AS districts`).
- Column aliases follow column naming rules and say what the value is (`average_score`, not `scores_1`).
- In queries with more than one table, qualify every column reference. In single-table queries, don't.

### Comments (`references/comments.md`)
- `/* */` for every written comment, one line or several; multi-line comments use ` * ` line prefixes.
- `--` only for commented-out code, one per line. Prefer deleting dead code over keeping it.
- Comment non-obvious intent and business rules, not what the code plainly says. Explain CTEs whose logic is surprising.

### Data types (`references/data-types.md`)
- `NUMERIC` / `DECIMAL` for money and exact values; `REAL` / `FLOAT` only for genuine floating-point maths.
- `BOOLEAN` for flags, with `true`/`false`, never `1`/`0`.
- Defaults match the column type (`DEFAULT false` on a `BOOLEAN`, `DEFAULT 0` on a `NUMERIC`).
- ISO 8601 date/time values, written with typed literals: `DATE '2024-09-01'`, `TIMESTAMP '2025-07-31T23:59:59'`.

### Constraints and keys (`references/constraints-keys.md`)
- Every table has at least one key.
- Name constraints explicitly (`CONSTRAINT chk_products_price_min CHECK (price > 0)`). `PRIMARY KEY`, `FOREIGN KEY` and `UNIQUE` may use generated names.
- Table-level constraints go after the columns, separated by a blank line; `PRIMARY KEY` is the first of them.
- One rule per `CHECK`; guard numeric ranges (at least `> 0` where a value must be positive).
- Validate text formats with `LIKE` / `SIMILAR TO`.

### Joins (`references/joins.md`)
- Always `ON`, never `USING`.
- In the condition, the table introduced first goes first (`ON customers.id = orders.customer_id`).
- For `INNER JOIN`, row filters belong in `WHERE`; `ON` holds only the relationship. (Not for `LEFT JOIN`: moving them changes the result.)

### Views (`references/views.md`)
- List the columns explicitly; never `SELECT *` in a view.
- Only create a view for a clear, recurring purpose, not as a mirror of a table.

### Subqueries and CTEs (`references/subqueries-ctes.md`)
- Prefer CTEs over nested subqueries.
- Avoid correlated subqueries; use a CTE plus `JOIN`.
- One logical step per CTE.
- CTE and subquery names are descriptive noun phrases (`active_customers`, not `tmp`, `d`, `sq`), with no `cte_` prefix or `_cte` suffix.

### Queries (`references/queries.md`)
- Use `BETWEEN` for ranges and `IN (...)` for value sets instead of chains of `AND` / `OR`.
- `CASE`: use the simple form (`CASE status WHEN ...`) when comparing one expression; multi-line layout with each `WHEN ... THEN ...` indented under `CASE`, `ELSE` at the `WHEN` level, `END` aligned with `CASE`. A single short `WHEN` may stay on one line. Inside a function call, `CASE` and `END` go on their own lines inside the parentheses.
- `UNION ALL` unless removing duplicates is actually required.
- Deduplicate with `SELECT DISTINCT` (written `SELECT` / `    DISTINCT` / columns), not `GROUP BY` on every column.
- No `ORDER BY` unless the result's order is part of what's required.
- `GROUP BY` uses column names or select-list aliases, never positions (`GROUP BY 1, 2`).
- Filters that don't depend on aggregates go in `WHERE`, not `HAVING`.
- Group by a select-list alias rather than repeating the expression. Not standard SQL: it works in PostgreSQL, MySQL, BigQuery, Snowflake, but not SQL Server or Oracle, where you compute the expression in a CTE or subquery and group by its column.
- Non-aggregated columns first in `SELECT`, aggregates last.
- Booleans compared with `IS true` / `IS false`, never implicit (`WHERE is_active`) or `NOT flag`.
- `!=`, not `<>`.
- Long `IN` lists: one value per line, indented, inside `IN (` ... `)`.
- No redundant parentheses (except to wrap a long expression across lines).
- Window functions on one line when they fit under 120 characters; otherwise `OVER (` ends the line, each sub-clause (`PARTITION BY`, `ORDER BY`, `ROWS BETWEEN`) on its own indented line, `)` aligned with the function.

## Example

A query written to the guide:

```sql
WITH active_customers AS (
    SELECT
        id,
        full_name
    FROM customers
    WHERE
        is_active IS true
),

recent_orders AS (
    SELECT
        customer_id,
        total_amount
    FROM orders
    WHERE
        created_at > DATE '2024-01-01'
)

SELECT
    active_customers.full_name,
    count(recent_orders.customer_id) AS total_orders,
    sum(recent_orders.total_amount)  AS total_spent_eur
FROM active_customers
INNER JOIN recent_orders
    ON active_customers.id = recent_orders.customer_id
GROUP BY active_customers.full_name
```
