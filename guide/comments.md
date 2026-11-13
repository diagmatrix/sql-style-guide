# Comments

## Use `/* */` for comments and `--` for commented-out code

Use `/* */` for every written comment, whether it spans one line or several. Use `--` only to comment out lines of code, prefixing each line.

This keeps commented-out code visible in reviews. When a block of code is wrapped in `/* */`, a diff only shows the opening and closing markers, so it is easy to miss that the lines in between are disabled. With `--` on every line, each disabled line is marked.

Avoid committing commented-out code at all: delete it and rely on version control. Use `--` only when a line has to stay disabled for a reason.

##### Example

Bad:
```sql
-- Returns all active users with at least one subscription
-- during the current billing year.
SELECT
    users.email,
    count(subscriptions.plan_id) AS total_subscriptions
FROM users
-- Current subscriptions
INNER JOIN subscriptions
    ON users.id = subscriptions.user_id
WHERE
    users.is_active IS true
/*
    AND users.signup_year > 2024
    AND users.country_code IS NOT NULL
*/
```

Good:
```sql
/*
 * Returns all active users with at least one subscription
 * during the current billing year.
 */
SELECT
    users.email,
    count(subscriptions.plan_id) AS total_subscriptions
FROM users
/* Current subscriptions */
INNER JOIN subscriptions
    ON users.id = subscriptions.user_id
WHERE
    users.is_active IS true
    -- AND users.signup_year > 2024
    -- AND users.country_code IS NOT NULL
```

#### Source breakdown

 - **For `/* */` multi-line + `--` single-line (6)**: Holywell, Celko, PostgreSQL, PG.ai, GitLab, Supabase.
 - **For `/* */` exclusively (1)**: Brooklyn.
 - **Not mentioned (2)**: Mozilla, Mazur.

## Comment only where necessary

Add comments where the intent or logic is not self-evident. Do not comment every line or state what the code already makes obvious. Equally, do not leave complex or surprising logic without explanation.

##### Example

Bad (over-commented):
```sql
SELECT
    name,         /* the product name */
    price         /* the product price */
FROM products     /* from the products table */
WHERE
    is_available IS true /* only available products */
```

Bad (under-commented):
```sql
SELECT
    name,
    price,
    price / 1.21 AS net_price
FROM products
WHERE
    country_code = 'ES'
```

Good:
```sql
SELECT
    name,
    price,
    /* Spanish prices include 21% VAT: remove it to get the net price */
    price / 1.21 AS net_price
FROM products
WHERE
    country_code = 'ES'
```

#### Source breakdown

 - **For (2)**: Holywell, Celko.
 - **Against (0)**
 - **Not mentioned (7)**: PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab, Mazur, Supabase.

## Comment CTEs with confusing or notable logic

When a CTE contains logic that is not immediately obvious, such as a complex filter, a business rule, or a non-trivial join strategy, add a comment above or inside it explaining what it does and why.

##### Example

Bad:
```sql
WITH eligible_students AS (
    SELECT
        student_id
    FROM enrolments
    WHERE
        completed_at IS NOT NULL
        AND grade >= 50
        AND course_id NOT IN (
            SELECT
                id
            FROM courses
            WHERE
                is_pilot IS true
        )
)

SELECT
    *
FROM eligible_students
```

Good:
```sql
/*
 * Students who completed a non-pilot course with a passing grade (≥50).
 * Pilot courses are excluded as they follow a different grading scheme.
 */
WITH eligible_students AS (
    SELECT
        student_id
    FROM enrolments
    WHERE
        completed_at IS NOT NULL
        AND grade >= 50
        AND course_id NOT IN (
            SELECT
                id
            FROM courses
            WHERE
                is_pilot IS true
        )
)

SELECT
    *
FROM eligible_students
```

#### Source breakdown

 - **For (3)**: Brooklyn, GitLab, Supabase.
 - **Against (0)**
 - **Not mentioned (6)**: Holywell, Celko, PostgreSQL, PG.ai, Mozilla, Mazur.
