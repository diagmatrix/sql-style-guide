# Query clauses and constructs

## Combine multiple boolean conditions with `BETWEEN` and `IN()`

Use `BETWEEN` instead of combining multiple `AND` conditions to express a range. Use `IN()` instead of chaining `OR` clauses to match against a set of values. Both make the intent clearer and are easier to scan.

##### Example

Bad:
```sql
SELECT
    *
FROM orders
WHERE
    quantity >= 10
    AND quantity <= 50
    AND (
        status = 'shipped'
        OR status = 'pending'
        OR status = 'returned'
    )
```

Good:
```sql
SELECT
    *
FROM orders
WHERE
    quantity BETWEEN 10 AND 50
    AND status IN ('shipped', 'pending', 'returned')
```

## CASE Statements

### Use `CASE` for conditional logic

Use `CASE` expressions for inline conditional logic. When the same expression is compared against multiple values, use the simple `CASE <expression> WHEN <value>` form rather than repeating the expression in every `WHEN` branch.

##### Example

Bad:
```sql
SELECT
    full_name,
    CASE
        WHEN grade_band = 'A' THEN 'Distinction'
        WHEN grade_band = 'B' THEN 'Merit'
        WHEN grade_band = 'C' THEN 'Pass'
        ELSE 'Fail'
    END AS grade_label
FROM assessments
```

Good:
```sql
SELECT
    full_name,
    CASE grade_band
        WHEN 'A' THEN 'Distinction'
        WHEN 'B' THEN 'Merit'
        WHEN 'C' THEN 'Pass'
        ELSE 'Fail'
    END AS grade_label
FROM assessments
```

### Use a multi line `CASE` structure

For `CASE` expressions that span multiple lines, indent each `WHEN` one level under `CASE`. Place `THEN` on the same line as its `WHEN`. Put `ELSE` at the same indentation level as `WHEN`. Close with `END` on its own line, aligned with `CASE`.

##### Example

Bad:
```sql
SELECT
    endpoint,
    CASE WHEN latency_ms >= 1000 THEN 'Slow'
    WHEN latency_ms >= 300 THEN 'Acceptable'
    ELSE 'Fast' END AS speed_label
FROM requests
```

Good:
```sql
SELECT
    endpoint,
    CASE
        WHEN latency_ms >= 1000 THEN 'Slow'
        WHEN latency_ms >= 300 THEN 'Acceptable'
        ELSE 'Fast'
    END AS speed_label
FROM requests
```

### A single short `WHEN`/`THEN` can stay on one line

When a `CASE` has only one `WHEN` branch and the `THEN` value is short, the entire expression can be written on a single line.

##### Example

Bad (several branches on one line):
```sql
SELECT
    endpoint,
    CASE WHEN latency_ms >= 1000 THEN 'Slow' WHEN latency_ms >= 300 THEN 'Acceptable' ELSE 'Fast' END AS speed_label
FROM requests
```

Good:
```sql
SELECT
    endpoint,
    CASE WHEN is_cached IS true THEN 'Cached' END AS cache_label
FROM requests
```

### `CASE` inside a function call

When a `CASE` expression is nested inside a function call, place `CASE` and `END` each on their own lines, indented one level inside the function's parentheses.

##### Example

Bad:
```sql
SELECT
    count(CASE WHEN status = 'failed' THEN id ELSE NULL END) AS failed_payments
FROM payments
```

Good:
```sql
SELECT
    count(
        CASE
            WHEN status = 'failed' THEN id
            ELSE NULL
        END
    ) AS failed_payments
FROM payments
```

## Prefer `UNION ALL` over `UNION`

`UNION ALL` is more performant than `UNION` because it skips the sort and deduplication step. Only use `UNION` when removing duplicate rows is genuinely part of the requirement.

##### Example

Bad:
```sql
SELECT
    student_id,
    course_id
FROM enrolments
WHERE
    status = 'active'
UNION
SELECT
    student_id,
    course_id
FROM archived_enrolments
WHERE
    status = 'active'
```

Good:
```sql
SELECT
    student_id,
    course_id
FROM enrolments
WHERE
    status = 'active'
UNION ALL
SELECT
    student_id,
    course_id
FROM archived_enrolments
WHERE
    status = 'active'
```

## Use `SELECT DISTINCT` instead of `GROUP BY` on all columns when deduplicating

When the goal is to remove duplicate rows rather than aggregate, use `SELECT DISTINCT`. Using `GROUP BY` on every column in the select list to achieve the same result obscures intent.

##### Example

Bad:
```sql
SELECT
    customer_id,
    product_id
FROM orders
GROUP BY customer_id, product_id
```

Good:
```sql
SELECT
    DISTINCT
    customer_id,
    product_id
FROM orders
```

## Avoid `ORDER BY` unless necessary to produce the correct result

Do not add `ORDER BY` to a query just for convenience or aesthetics. Sorting is expensive and the results of most queries will be re-processed downstream (by application code, a window function, or another query) where the order is irrelevant or re-applied anyway.

##### Example

Bad:
```sql
SELECT
    agent_id,
    count(id) AS total_tickets
FROM tickets
GROUP BY agent_id
ORDER BY agent_id
```

Good:
```sql
SELECT
    agent_id,
    count(id) AS total_tickets
FROM tickets
GROUP BY agent_id
```

## Always use explicit column names in `GROUP BY`

Use column names rather than positional numbers in `GROUP BY` clauses. Positional references (`GROUP BY 1, 2`) are fragile, as reordering the `SELECT` list silently changes what is being grouped without any error.

For complex expressions, alias them in the `SELECT` list and group by the alias (see [lateral column aliasing](#use-lateral-column-aliasing-when-grouping-by-name)) instead of using a positional reference.

##### Example

Bad:
```sql
SELECT
    school_id,
    cohort,
    count(student_id) AS total_students
FROM students
GROUP BY 1, 2
```

Good:
```sql
SELECT
    school_id,
    cohort,
    count(student_id) AS total_students
FROM students
GROUP BY school_id, cohort
```

## Prefer `WHERE` over `HAVING` when either would suffice

`WHERE` filters rows before aggregation; `HAVING` filters after. When a condition does not depend on an aggregate result, place it in `WHERE`: it is more explicit about intent and filters earlier, reducing the rows that need to be aggregated.

##### Example

Bad:
```sql
SELECT
    agency_id,
    count(id) AS total_properties
FROM properties
GROUP BY agency_id
HAVING
    agency_id IS NOT NULL
```

Good:
```sql
SELECT
    agency_id,
    count(id) AS total_properties
FROM properties
WHERE
    agency_id IS NOT NULL
GROUP BY agency_id
```

## Use lateral column aliasing when grouping by name

When an expression in `SELECT` has a column alias, you can reference that alias directly in `GROUP BY` rather than repeating the full expression. This avoids duplication and makes the query easier to maintain.

Note that referencing a `SELECT` alias in `GROUP BY` is not standard SQL. It is supported by PostgreSQL, MySQL, BigQuery and Snowflake, among others, but not by SQL Server or Oracle. In those, compute the expression in a CTE or subquery and group by its column name instead.

##### Example

Bad:
```sql
SELECT
    extract(YEAR FROM sold_at) AS sale_year,
    count(id)                  AS total_sales
FROM sales
GROUP BY extract(YEAR FROM sold_at)
```

Good:
```sql
SELECT
    extract(YEAR FROM sold_at) AS sale_year,
    count(id)                  AS total_sales
FROM sales
GROUP BY sale_year
```

## Place non-aggregated columns first in `SELECT`, aggregated columns last

In a query with aggregation, list the grouping (non-aggregated) columns first, followed by the aggregated expressions. This mirrors the logical order of `GROUP BY` and makes the structure of the query easier to parse at a glance.

##### Example

Bad:
```sql
SELECT
    count(student_id) AS total_students,
    avg(score)        AS average_score,
    school_id,
    cohort
FROM students
GROUP BY school_id, cohort
```

Good:
```sql
SELECT
    school_id,
    cohort,
    count(student_id) AS total_students,
    avg(score)        AS average_score
FROM students
GROUP BY school_id, cohort
```

## Be explicit in boolean conditions

Write boolean comparisons using `IS true` and `IS false` rather than relying on implicit boolean evaluation. Why use `IS` instead of `=`? Unlike `= true`, the `IS` form always returns a definite boolean, never propagating `NULL`. When the column is `NULL`, `column = true` evaluates to `NULL` (which SQL treats as falsy), while `column IS true` evaluates to `FALSE`. This distinction matters: `= true` and `IS true` behave the same in a `WHERE` clause, but `IS true` makes the null behaviour explicit and works correctly in `CASE` expressions and derived columns where a `NULL` result would otherwise silently propagate.

##### Example

Bad:
```sql
SELECT
    *
FROM users
WHERE
    is_active
    AND NOT is_premium
```

Good:
```sql
SELECT
    *
FROM users
WHERE
    is_active IS true
    AND is_premium IS false
```

## Use `!=` instead of `<>`

Prefer `!=` over `<>` for the not-equal operator. Both are valid SQL, but `!=` is more common in other programming languages and reads more naturally as "not equal".

##### Example

Bad:
```sql
SELECT
    *
FROM orders
WHERE
    country_code <> 'ES'
    AND status <> 'cancelled'
```

Good:
```sql
SELECT
    *
FROM orders
WHERE
    country_code != 'ES'
    AND status != 'cancelled'
```

## Break long `IN` lists into multiple indented lines

When an `IN` list contains many values, place each value on its own indented line. This makes the list easier to scan and simplifies adding or removing values in version control diffs.

##### Example

Bad:
```sql
SELECT
    *
FROM students
WHERE
    school_id IN (101, 102, 103, 104, 105, 106, 107, 108)
```

Good:
```sql
SELECT
    *
FROM students
WHERE
    school_id IN (
        101,
        102,
        103,
        104,
        105,
        106,
        107,
        108
    )
```

## Avoid redundant parentheses

Do not add parentheses around expressions that do not require them. Extra parentheses suggest a precedence concern where there is none, and add visual noise that makes expressions harder to read.

The only exception is grouping an expression that is too long for one line and has to span several (see [Keep lines under 120 characters](formatting.md#keep-lines-under-120-characters)).

##### Example

Bad:
```sql
SELECT
    *
FROM products
WHERE
    (is_available IS true)
    AND (price > 20)
    AND (category_id IN (3, 7))
```

Good:
```sql
SELECT
    *
FROM products
WHERE
    is_available IS true
    AND price > 20
    AND category_id IN (3, 7)
```

## Keep window functions on one line when they fit. Expand sub-clauses when they don't

If a window function fits on a single line without exceeding the line length limit, keep it there. When it needs to break across lines, place each sub-clause (`PARTITION BY`, `ORDER BY`, `ROWS/RANGE BETWEEN`) on its own line indented inside `OVER()`, and put the closing parenthesis on its own line at the same indentation level as the function.

##### Example

Bad (fits on one line, but is split):
```sql
SELECT
    full_name,
    row_number() OVER (
        PARTITION BY department_id
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees
```

Bad (too long for one line):
```sql
SELECT
    ticker,
    avg(close_price) OVER (PARTITION BY ticker, exchange ORDER BY traded_on ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS rolling_average_price
FROM stock_prices
```

Good (fits on one line):
```sql
SELECT
    full_name,
    row_number() OVER (PARTITION BY department_id ORDER BY salary DESC) AS salary_rank
FROM employees
```

Good (too long for one line):
```sql
SELECT
    ticker,
    avg(close_price) OVER (
        PARTITION BY ticker, exchange
        ORDER BY traded_on
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS rolling_average_price
FROM stock_prices
```
