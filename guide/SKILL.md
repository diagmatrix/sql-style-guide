---
name: sql-style-guide
description: Write, review and reformat SQL following diagmatrix's SQL style guide (left-aligned keywords, 4-space indentation, arguments below SELECT/WHERE/HAVING, aligned AS keywords, uppercase keywords and lowercase functions, snake_case plural tables, CTEs over subqueries, named constraints, and more). Use this skill whenever writing any SQL — queries, views, CTEs, CREATE TABLE statements or migrations — and whenever the user asks to review, lint, check, clean up, tidy, format or "fix the style of" SQL, even if they don't mention the style guide by name.
---

# SQL style guide

This skill applies a specific SQL style guide. The full guide lives next to this file, one file per topic (index in `README.md`), each rule with an explanation and Bad/Good examples. The checklist below condenses every rule so you can usually work from it alone; open the topic file when a rule's exact scope or an edge case matters. Ignore the *Source breakdown* section under each rule: it records which external guides support the rule and doesn't change how the rule is applied.

The guide describes itself as meant "to inform, not dictate": when following a rule would clearly make the SQL worse (efficiency, business constraints, a dialect that doesn't support the construct), deviate and say why. Consistency within a project matters more than any single rule.

## What you might be asked to do

**Write SQL.** Follow the checklist from the start, so the code needs no style pass afterwards.

**Review SQL.** Check the SQL against the checklist and report each problem. Use this format, grouped by rule and citing its code (e.g. `LAY-03`), most impactful first:

```text
### <Code> <Rule name> (<reference file>)
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

### Layout (`layout.md`)

- **LAY-01** Left-align all clause keywords at the same margin. No "river" (right-aligned) keywords.
- **LAY-02** Each major clause (`SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, ...) starts its own line.
- **LAY-03** `SELECT`, `WHERE` and `HAVING`: every argument on its own line below the keyword, indented one level, **even a single one** (`SELECT` / `*`).
- **LAY-03** `FROM`, `JOIN`, `GROUP BY`, `ORDER BY` (and other clauses): arguments stay on the keyword line (`FROM orders`, `GROUP BY region, country_code`).
- **LAY-04** Indent with 4 spaces.
- **LAY-05** `ON` goes on the line after its `JOIN`, indented one level, even with one condition. Extra join conditions follow as `AND` lines at the same indentation.
- **LAY-06** An opening parenthesis that starts a multi-line construct ends its line; content indented one level; closing parenthesis on its own line, aligned with the line that opened it.
- **LAY-07** CTEs: first one on the `WITH` line (`WITH active_customers AS (`), later ones at the same level as `WITH`; body indented one level; `)` on its own line; a blank line between CTEs and after the last one.
- **LAY-08** Exactly one space between tokens (around operators, after commas). No padding to align things, except the two alignment rules below.
- **LAY-09** Align `AS` keywords in a `SELECT` list: pad expressions so every `AS` sits one space after the longest aliased expression. Columns without alias are left alone.
- **LAY-10** In `CREATE TABLE`, align data types one space after the longest column name, and column constraints one space after the longest type. Named table constraints at the end: pad names so the `CHECK (...)` / `FOREIGN KEY (...)` bodies line up.
- **LAY-11** No spaces inside parentheses, none between a function name and `(`.
- **LAY-12** `AND` / `OR` start the next line, at the same indentation as the first condition.
- **LAY-13** A blank line after each `;` between statements.
- **LAY-14** Blank lines between logically distinct sections (between CTEs, before the main query, between groups of conditions).
- **LAY-15** Trailing commas, never leading commas.
- **LAY-16** Lines under 120 characters. To break a long expression, wrap it in parentheses, break after `(`, indent, start each continuation line with its operator, close `)` on its own line. Break inside a function's own parentheses instead of adding new ones. Still unreadable → move it into a CTE.
- **LAY-17** Long `IN` lists: one value per line, indented, inside `IN (` ... `)`.
- **LAY-18, LAY-19, LAY-20** `CASE`: multi-line layout with each `WHEN ... THEN ...` indented under `CASE`, `ELSE` at the `WHEN` level, `END` aligned with `CASE`. A single short `WHEN` may stay on one line. Inside a function call, `CASE` and `END` go on their own lines inside the parentheses.
- **LAY-21** Window functions on one line when they fit under 120 characters; otherwise `OVER (` ends the line, each sub-clause (`PARTITION BY`, `ORDER BY`, `ROWS BETWEEN`) on its own indented line, `)` aligned with the function.

### Keywords and functions (`keywords.md`)

- **KWD-01** Reserved words in UPPERCASE (`SELECT`, `AS`, `IS`, `CASE`, ...).
- **KWD-02** Full keyword forms instead of abbreviations (`INNER JOIN`, not `JOIN`).
- **KWD-03** Prefer ANSI functions over vendor-specific ones when an ANSI equivalent exists (`extract(YEAR FROM x)` over `date_part('year', x)`).
- **KWD-04** Functions in lowercase (`count()`, `coalesce()`, `string_agg()`).
- **KWD-05** Built-in data types in UPPERCASE (`INTEGER`, `VARCHAR(255)`, `TIMESTAMP`). User-defined types follow naming rules (lowercase snake_case).
- **TYP-02** Boolean literals are written `true` / `false` (lowercase), as in `IS true`.

### Naming (`naming.md`)

- **NAM-01, NAM-03** snake_case; only lowercase letters, digits and underscores; start with a letter; no trailing or double underscores; never quoted identifiers.
- **NAM-02** Descriptive, consistent across tables, unambiguous. No type/role prefixes (`tbl_`, `sp_`, `vw_`). Exception: constraint names may carry a type prefix (`chk_`, `fk_`, ...).
- **NAM-02** Avoid abbreviations unless the whole team understands them.
- **NAM-04** Never use reserved words as names (`group`, `union`, `rank`, `order`, `user`, ...).
- **NAM-05, NAM-06** Tables: collective or plural nouns (`staff`, `orders`); singular only for single-row tables. Relationship tables get a real name when one exists (`enrolments`, not `student_courses`).
- **NAM-07, NAM-09** Columns: singular (`goal_count`, not `goals`). Don't repeat the table name (`products.name`, not `products.product_name`; avoid `grades.grade`).
- **NAM-08** A table's own primary key is `id`; `_id` marks foreign keys and other identifiers (`orders.customer_id`).
- **NAM-08** Standard affixes: `_id`, `_status`, `_total`/`total_`, `_name`, `_size`, `_date` (date), `_at` (timestamp), `_addr`/`_address`, `is_`/`has_` (booleans), unit suffixes (`_eur`, `_ms`), `_month`/`_day`.
- **NAM-02** Generic columns (`id`, `name`, `type`) selected from a table other than the one the result describes get a `table_column` alias (`schools.name AS school_name`). Columns of the table the result is about keep their name (`customers.id` in a query returning customers).
- **NAM-10, NAM-11** CTE and subquery names are descriptive noun phrases (`active_customers`, not `tmp`, `d`, `sq`), with no `cte_` prefix or `_cte` suffix.

### Aliasing and references (`aliasing.md`)

- **ALI-01** Always write `AS`, for columns and tables.
- **ALI-02** Every aggregate or computed column gets an alias (`count(*) AS total_orders`).
- **ALI-03** Prefer full table names. Alias only long or ambiguous names (or self-joins), with a meaningful word, never initials or `t1`/`x` (`school_district_areas AS districts`).
- **ALI-04** Column aliases follow column naming rules and say what the value is (`average_score`, not `scores_1`).
- **ALI-05, ALI-06** In queries with more than one table, qualify every column reference. In single-table queries, don't.

### Comments (`comments.md`)

- **CMT-01** `/* */` for every written comment, one line or several; multi-line comments use ` * ` line prefixes.
- **CMT-01** `--` only for commented-out code, one per line. Prefer deleting dead code over keeping it.
- **CMT-02, CMT-03** Comment non-obvious intent and business rules, not what the code plainly says. Explain CTEs whose logic is surprising.

### Data types and literals (`data-types.md`)

- **TYP-01** `NUMERIC` / `DECIMAL` for money and exact values; `REAL` / `FLOAT` only for genuine floating-point maths.
- **TYP-02** `BOOLEAN` for flags, with `true`/`false`, never `1`/`0`.
- **TYP-03** Defaults match the column type (`DEFAULT false` on a `BOOLEAN`, `DEFAULT 0` on a `NUMERIC`).
- **TYP-04** ISO 8601 date/time values, written with typed literals: `DATE '2024-09-01'`, `TIMESTAMP '2025-07-31T23:59:59'`.

### Expressions and conditions (`expressions.md`)

- **EXP-01** Use `BETWEEN` for ranges and `IN (...)` for value sets instead of chains of `AND` / `OR`.
- **EXP-02** `CASE`: use the simple form (`CASE status WHEN ...`) when comparing one expression.
- **EXP-03** Booleans compared with `IS true` / `IS false`, never implicit (`WHERE is_active`) or `NOT flag`.
- **EXP-04** `!=`, not `<>`.
- **EXP-05** No redundant parentheses (except to wrap a long expression across lines).

### Query structure (`structure.md`)

- **STR-01** Prefer CTEs over nested subqueries.
- **STR-02** Avoid correlated subqueries; use a CTE plus `JOIN`.
- **STR-03** One logical step per CTE.
- **STR-04** Always `ON`, never `USING`.
- **STR-05** In the condition, the table introduced first goes first (`ON customers.id = orders.customer_id`).
- **STR-06** For `INNER JOIN`, row filters belong in `WHERE`; `ON` holds only the relationship. (Not for `LEFT JOIN`: moving them changes the result.)

### Clauses (`clauses.md`)

- **CLS-01** `UNION ALL` unless removing duplicates is actually required.
- **CLS-02** Deduplicate with `SELECT DISTINCT` (written `SELECT` / `DISTINCT` / columns), not `GROUP BY` on every column.
- **CLS-03** No `ORDER BY` unless the result's order is part of what's required.
- **CLS-04** `GROUP BY` uses column names or select-list aliases, never positions (`GROUP BY 1, 2`).
- **CLS-05** Filters that don't depend on aggregates go in `WHERE`, not `HAVING`.
- **CLS-06** Group by a select-list alias rather than repeating the expression. Not standard SQL: it works in PostgreSQL, MySQL, BigQuery, Snowflake, but not SQL Server or Oracle, where you compute the expression in a CTE or subquery and group by its column.
- **CLS-07** Non-aggregated columns first in `SELECT`, aggregates last.

### Schema objects (`schema.md`)

- **DDL-01** Every table has at least one key.
- **DDL-02** Name constraints explicitly (`CONSTRAINT chk_products_price_min CHECK (price > 0)`). `PRIMARY KEY`, `FOREIGN KEY` and `UNIQUE` may use generated names.
- **DDL-03, DDL-04** Table-level constraints go after the columns, separated by a blank line; `PRIMARY KEY` is the first of them.
- **DDL-05** One rule per `CHECK`; guard numeric ranges (at least `> 0` where a value must be positive).
- **DDL-06** Validate text formats with `LIKE` / `SIMILAR TO`.
- **DDL-07** List the columns explicitly; never `SELECT *` in a view.
- **DDL-08** Only create a view for a clear, recurring purpose, not as a mirror of a table.

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
