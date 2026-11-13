# Subqueries and CTEs

## Prefer CTEs over nested subqueries

When a query requires intermediate results, use a CTE (`WITH`) rather than a nested subquery. CTEs give each step a name, making the query easier to read, test, and debug.

##### Example

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

#### Source breakdown

 - **For (7)**: Celko, PG.ai, Mozilla, Brooklyn, GitLab, Mazur, Supabase.
 - **Against (0)**
 - **Not mentioned (2)**: Holywell, PostgreSQL.

## Avoid correlated subqueries

A correlated subquery references a column from the outer query, causing it to execute once per row. This leads to poor performance and is almost always replaceable with a `JOIN` or a CTE.

##### Example

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

#### Source breakdown

 - **For (2)**: Celko, PG.ai.
 - **Against (0)**
 - **Not mentioned (7)**: Holywell, PostgreSQL, Mozilla, Brooklyn, GitLab, Mazur, Supabase.

## Each CTE should do one logical unit of work

A CTE should encapsulate a single, well-defined step in the query's logic. Avoid cramming multiple transformations into one CTE. A query made of several simple CTEs is easier to understand and debug than one made of fewer complex ones.

##### Example

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

#### Source breakdown

 - **For (3)**: Brooklyn, GitLab, Supabase.
 - **Against (0)**
 - **Not mentioned (6)**: Holywell, Celko, PostgreSQL, PG.ai, Mozilla, Mazur.

## CTE and subquery names should be descriptive

CTE names should clearly convey what the CTE produces. Prefer names that read like a noun phrase describing the result set  (like `active_students`) rather than abbreviations or vague labels like `data` or `tmp`. Make names as long as they need to be to stay unambiguous, without padding them with unnecessary words.

The same applies to subquery aliases, for example a subquery in a `JOIN` or `FROM` clause.

##### Example

Bad:
```sql
WITH d AS (
...
),

tmp AS (
...
),

final AS (
...
)
```

Good:
```sql
WITH active_students AS (
...
),

completed_enrolments AS (
...
),

student_pass_rates AS (
...
)
```

Bad (subquery):
```sql
SELECT
    students.full_name,
    sq.total_courses
FROM students
INNER JOIN (
    SELECT
        student_id,
        count(course_id) AS total_courses
    FROM enrolments
    GROUP BY student_id
) AS sq
    ON students.id = sq.student_id
```

Good (subquery):
```sql
SELECT
    students.full_name,
    student_course_counts.total_courses
FROM students
INNER JOIN (
    SELECT
        student_id,
        count(course_id) AS total_courses
    FROM enrolments
    GROUP BY student_id
) AS student_course_counts
    ON students.id = student_course_counts.student_id
```

#### Source breakdown

 - **For descriptive (1)**: Brooklyn.
 - **For concise but clear (1)**: GitLab.
 - **Not mentioned (7)**: Holywell, Celko, PostgreSQL, PG.ai, Mozilla, Mazur, Supabase.

## Do not prefix or suffix CTE names with `cte`

Avoid names like `cte_open_tickets` or `open_tickets_cte`. The `WITH` keyword already makes the CTE context clear, so the redundant label adds noise without adding information.

##### Example

Bad:
```sql
WITH cte_open_tickets AS (
    SELECT
        id,
        subject
    FROM tickets
    WHERE
        is_closed IS false
)

SELECT
    *
FROM cte_open_tickets
```

Good:
```sql
WITH open_tickets AS (
    SELECT
        id,
        subject
    FROM tickets
    WHERE
        is_closed IS false
)

SELECT
    *
FROM open_tickets
```

#### Source breakdown

 - **For (1)**: Brooklyn.
 - **Against (0)**
 - **Not mentioned (8)**: Holywell, Celko, PostgreSQL, PG.ai, Mozilla, GitLab, Mazur, Supabase.
