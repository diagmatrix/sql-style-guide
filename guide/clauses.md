# Clauses

## CLS-01 Prefer `UNION ALL` over `UNION`

`UNION ALL` is more performant than `UNION` because it skips the sort and
deduplication step. Only use `UNION` when removing duplicate rows is genuinely
part of the requirement.

### Example

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

## CLS-02 Use `SELECT DISTINCT` instead of `GROUP BY` on all columns when deduplicating

When the goal is to remove duplicate rows rather than aggregate, use
`SELECT DISTINCT`. Using `GROUP BY` on every column in the select list to
achieve the same result obscures intent.

### Example

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

## CLS-03 Avoid `ORDER BY` unless necessary to produce the correct result

Do not add `ORDER BY` to a query just for convenience or aesthetics. Sorting is
expensive and the results of most queries will be re-processed downstream (by
application code, a window function, or another query) where the order is
irrelevant or re-applied anyway.

### Example

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

## CLS-04 Always use explicit column names in `GROUP BY`

Use column names rather than positional numbers in `GROUP BY` clauses.
Positional references (`GROUP BY 1, 2`) are fragile, as reordering the `SELECT`
list silently changes what is being grouped without any error.

For complex expressions, alias them in the `SELECT` list and group by the alias
(see [lateral column
aliasing](#cls-06-use-lateral-column-aliasing-when-grouping-by-name)) instead of
using a positional reference.

### Example

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

## CLS-05 Prefer `WHERE` over `HAVING` when either would suffice

`WHERE` filters rows before aggregation; `HAVING` filters after. When a
condition does not depend on an aggregate result, place it in `WHERE`: it is
more explicit about intent and filters earlier, reducing the rows that need to
be aggregated.

### Example

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

## CLS-06 Use lateral column aliasing when grouping by name

When an expression in `SELECT` has a column alias, you can reference that alias
directly in `GROUP BY` rather than repeating the full expression. This avoids
duplication and makes the query easier to maintain.

Note that referencing a `SELECT` alias in `GROUP BY` is not standard SQL. It is
supported by PostgreSQL, MySQL, BigQuery and Snowflake, among others, but not by
SQL Server or Oracle. In those, compute the expression in a CTE or subquery and
group by its column name instead.

### Example

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

## CLS-07 Place non-aggregated columns first in `SELECT`, aggregated columns last

In a query with aggregation, list the grouping (non-aggregated) columns first,
followed by the aggregated expressions. This mirrors the logical order of
`GROUP BY` and makes the structure of the query easier to parse at a glance.

### Example

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
