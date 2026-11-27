# Expressions and conditions

## EXP-01 Combine multiple boolean conditions with `BETWEEN` and `IN()`

Use `BETWEEN` instead of combining multiple `AND` conditions to express a range.
Use `IN()` instead of chaining `OR` clauses to match against a set of values.
Both make the intent clearer and are easier to scan.

### Example

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

## EXP-02 Use `CASE` for conditional logic

Use `CASE` expressions for inline conditional logic. When the same expression is
compared against multiple values, use the simple
`CASE <expression> WHEN <value>` form rather than repeating the expression in
every `WHEN` branch.

### Example

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

## EXP-03 Be explicit in boolean conditions

Write boolean comparisons using `IS true` and `IS false` rather than relying on
implicit boolean evaluation. Why use `IS` instead of `=`? Unlike `= true`, the
`IS` form always returns a definite boolean, never propagating `NULL`. When the
column is `NULL`, `column = true` evaluates to `NULL` (which SQL treats as
falsy), while `column IS true` evaluates to `FALSE`. This distinction matters:
`= true` and `IS true` behave the same in a `WHERE` clause, but `IS true` makes
the null behaviour explicit and works correctly in `CASE` expressions and
derived columns where a `NULL` result would otherwise silently propagate.

### Example

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

## EXP-04 Use `!=` instead of `<>`

Prefer `!=` over `<>` for the not-equal operator. Both are valid SQL, but `!=`
is more common in other programming languages and reads more naturally as "not
equal".

### Example

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

## EXP-05 Avoid redundant parentheses

Do not add parentheses around expressions that do not require them. Extra
parentheses suggest a precedence concern where there is none, and add visual
noise that makes expressions harder to read.

The only exception is grouping an expression that is too long for one line and
has to span several (see [LAY-16 Keep lines under 120
characters](layout.md#lay-16-keep-lines-under-120-characters)).

### Example

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
