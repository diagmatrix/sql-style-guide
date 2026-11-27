# Query structure

## STR-01 Prefer CTEs over nested subqueries

When a query requires intermediate results, use a CTE (`WITH`) rather than a nested subquery. CTEs give each step a name, making the query easier to read, test, and debug.

### Example

Bad:

```sql
SELECT
    full_name,
    total_orders
FROM (
    SELECT
        customers.full_name,
        count(orders.id) AS total_orders
    FROM customers
    INNER JOIN orders
        ON customers.id = orders.customer_id
    WHERE
        customers.is_active IS true
    GROUP BY customers.full_name
) AS customer_orders
WHERE
    total_orders > 3
```

Good:

```sql
WITH customer_orders AS (
    SELECT
        customers.full_name,
        count(orders.id) AS total_orders
    FROM customers
    INNER JOIN orders
        ON customers.id = orders.customer_id
    WHERE
        customers.is_active IS true
    GROUP BY customers.full_name
)

SELECT
    full_name,
    total_orders
FROM customer_orders
WHERE
    total_orders > 3
```

### Source breakdown

- **For (7)**: Celko, PG.ai, Mozilla, Brooklyn, GitLab, Mazur, Supabase.
- **Against (0)**
- **Not mentioned (2)**: Holywell, PostgreSQL.

## STR-02 Avoid correlated subqueries

A correlated subquery references a column from the outer query, causing it to execute once per row. This leads to poor performance and is almost always replaceable with a `JOIN` or a CTE.

### Example

Bad:

```sql
SELECT
    full_name,
    (
        SELECT
            count(*)
        FROM tracks
        WHERE
            tracks.artist_id = artists.id
    ) AS total_tracks
FROM artists
```

Good:

```sql
WITH track_counts AS (
    SELECT
        artist_id,
        count(*) AS total_tracks
    FROM tracks
    GROUP BY artist_id
)

SELECT
    artists.full_name,
    track_counts.total_tracks
FROM artists
INNER JOIN track_counts
    ON artists.id = track_counts.artist_id
```

### Source breakdown

- **For (2)**: Celko, PG.ai.
- **Against (0)**
- **Not mentioned (7)**: Holywell, PostgreSQL, Mozilla, Brooklyn, GitLab, Mazur, Supabase.

## STR-03 Each CTE should do one logical unit of work

A CTE should encapsulate a single, well-defined step in the query's logic. Avoid cramming multiple transformations into one CTE. A query made of several simple CTEs is easier to understand and debug than one made of fewer complex ones.

### Example

Bad:

```sql
WITH student_data AS (
    SELECT
        students.full_name,
        count(enrolments.course_id) AS total_courses,
        avg(assessments.score)      AS average_score,
        max(assessments.score)      AS top_score
    FROM students
    INNER JOIN enrolments
        ON students.id = enrolments.student_id
    INNER JOIN assessments
        ON enrolments.id = assessments.enrolment_id
    WHERE
        students.is_active IS true
    GROUP BY students.full_name
)

SELECT
    *
FROM student_data
```

Good:

```sql
WITH active_students AS (
    SELECT
        id,
        full_name
    FROM students
    WHERE
        is_active IS true
),

student_enrolments AS (
    SELECT
        active_students.full_name,
        enrolments.course_id,
        enrolments.id AS enrolment_id
    FROM active_students
    INNER JOIN enrolments
        ON active_students.id = enrolments.student_id
),

student_scores AS (
    SELECT
        student_enrolments.full_name,
        avg(assessments.score) AS average_score,
        max(assessments.score) AS top_score
    FROM student_enrolments
    INNER JOIN assessments
        ON student_enrolments.enrolment_id = assessments.enrolment_id
    GROUP BY student_enrolments.full_name
)

SELECT
    *
FROM student_scores
```

### Source breakdown

- **For (3)**: Brooklyn, GitLab, Supabase.
- **Against (0)**
- **Not mentioned (6)**: Holywell, Celko, PostgreSQL, PG.ai, Mozilla, Mazur.

## STR-04 Use `ON` for all join conditions; avoid `USING`

Always express join conditions with `ON`, even when the joining columns share the same name. Avoid `USING`, as it silently merges the columns into one in the result set, which can cause unexpected behaviour when adding aliases or when the query is refactored.

### Example

Bad:

```sql
SELECT
    customers.full_name,
    orders.total_amount
FROM customers
INNER JOIN orders USING (customer_id)
```

Good:

```sql
SELECT
    customers.full_name,
    orders.total_amount
FROM customers
INNER JOIN orders
    ON customers.id = orders.customer_id
```

### Source breakdown

- **For (2)**: Brooklyn, GitLab.
- **Against  (1)**: PostgreSQL.
- **Not mentioned (6)**: Holywell, Celko, PG.ai, Mozilla, Mazur, Supabase.

## STR-05 Put the first-referenced table immediately after `ON`

In a join condition, write the table that was introduced first (the left-hand table) immediately after the `ON` keyword. This makes fan-out risk visible at a glance: if the left table's key is not on the left side of the condition, a row duplication bug may be hiding.

### Example

Bad:

```sql
SELECT
    doctors.full_name,
    appointments.scheduled_at
FROM doctors
INNER JOIN appointments
    ON appointments.doctor_id = doctors.id
```

Good:

```sql
SELECT
    doctors.full_name,
    appointments.scheduled_at
FROM doctors
INNER JOIN appointments
    ON doctors.id = appointments.doctor_id
```

### Source breakdown

- **For (2)**: Brooklyn, Mazur.
- **Against (0)**
- **Not mentioned (7)**: Holywell, Celko, PostgreSQL, PG.ai, Mozilla, GitLab, Supabase.

## STR-06 For `INNER JOIN`, keep filter conditions in `WHERE`, not in the `JOIN` clause

The `ON` clause should express only the relationship between tables. Row-level filters belong in `WHERE`. Mixing business logic into `ON` obscures intent, as a reader scanning `ON` expects join conditions, not data filters.

As an important note, this rule applies only to `INNER JOIN`. With `LEFT JOIN`, moving a condition from `ON` to `WHERE` changes the semantics and effectively converts it into an inner join.

### Example

Bad:

```sql
SELECT
    drivers.full_name,
    deliveries.tracking_code
FROM drivers
INNER JOIN deliveries
    ON drivers.id = deliveries.driver_id
    AND drivers.is_active IS true
    AND deliveries.delivered_at IS NOT NULL
```

Good:

```sql
SELECT
    drivers.full_name,
    deliveries.tracking_code
FROM drivers
INNER JOIN deliveries
    ON drivers.id = deliveries.driver_id
WHERE
    drivers.is_active IS true
    AND deliveries.delivered_at IS NOT NULL
```

### Source breakdown

- **For (1)**: Brooklyn.
- **Against (0)**
- **Not mentioned (8)**: Holywell, Celko, PostgreSQL, PG.ai, Mozilla, GitLab, Mazur, Supabase.
