# Views

## Always explicitly name the columns a view exposes

Never rely on `SELECT *` in a view definition. Always list the columns the view exposes by name. This prevents silent breakage when the underlying table gains, loses, or reorders columns.

##### Example

Bad:
```sql
CREATE VIEW active_employees AS
SELECT
    *
FROM employees
WHERE
    is_active IS true
```

Good:
```sql
CREATE VIEW active_employees AS
SELECT
    id,
    full_name,
    job_title,
    department_id
FROM employees
WHERE
    is_active IS true
```

#### Source breakdown

 - **For (1)**: Celko.
 - **Against (0)**
 - **Not mentioned (8)**: Holywell, PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab, Mazur, Supabase.

## Avoid view proliferation

Do not create views as a default abstraction layer. Each view should exist because it solves a specific, recurring problem, such as enforcing access control, encapsulating a complex join, or standardising a derived metric. If a view has no clear purpose, it becomes maintenance burden without value.

##### Example

Bad:
```sql
/* A view that just mirrors the base table with no added value */
CREATE VIEW students_view AS
SELECT
    id,
    full_name,
    cohort,
    school_id,
    is_active
FROM students
```

Good:
```sql
/* A view that encapsulates a meaningful, reusable definition */
CREATE VIEW active_students_with_school AS
SELECT
    students.id,
    students.full_name,
    students.cohort,
    schools.name AS school_name
FROM students
INNER JOIN schools
    ON students.school_id = schools.id
WHERE
    students.is_active IS true
```

#### Source breakdown

 - **For (1)**: Celko.
 - **Against (0)**
 - **Not mentioned (8)**: Holywell, PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab, Mazur, Supabase.
