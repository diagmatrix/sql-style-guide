# Naming rules

These rules apply mostly to names chosen by developers, and may not apply when using replicated data from a source that does not adhere to the rules. In those cases, it is better to keep the original structure for clarity reasons (for example, not renaming columns to use `snake_case` in tables that are replicas of endpoints of APIs that use `camelCase`).

## General naming rules

### Use snake case

Always use snake case (lowercase with underscores, like `snake_case`) for naming.

##### Example

Bad:
```sql
SELECT
    firstName,
    last_name,
    Department
FROM employees
```

Good:
```sql
SELECT
    first_name,
    last_name,
    department
FROM employees
```

#### Source breakdown

 - **For (9)**: Holywell, Celko, PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab, Mazur, Supabase.
 - **Against (0)**
 - **Not mentioned (0)**

### Names must be descriptive, consistent and unambiguous

Use descriptive names and consistent across tables (for example, `cohort` should mean the same and have the same data type everywhere). Do not use descriptive prefixes that explain the role or type of the identifier (`tbl_`, `sp_`, `vw_`, ...). The exception is table constraint names, which may start with a prefix for the constraint type (`chk_`, `fk_`, ...), as the type is part of what the name has to convey.

Besides, avoid abbreviations unless they are commonly understood (for example, only use `urn` if the whole team understands that it means "unique reference number").

Generic column names such as `id`, `name` or `type` are allowed inside their own table, as the table already states what they identify: use `users.id`, not `users.user_id`. Prefixing them repeats the table name and makes the column look like a reference to another table. Foreign keys do carry the referenced entity as a prefix (`courses.user_id`), so joins read naturally: `ON courses.user_id = users.id`.

When a query selects a generic column from a table other than the one its result describes, give it a meaningful alias in `table_column` form (`users.id AS user_id`, `courses.name AS course_name`). This keeps the result set unambiguous. Generic columns of the table the result is about (such as `students.id` in a query or CTE that returns students) keep their name.

##### Example

Bad:
```sql
SELECT
    enrolments.pid,
    enrolments.created_at,
    tbl_courses.course_name,
    tbl_courses.date_creation,
    tbl_courses.cohort
FROM enrolments
INNER JOIN tbl_courses
    ON enrolments.course_id = tbl_courses.course_id
```

Good:
```sql
SELECT
    enrolments.participant_id,
    enrolments.created_at AS enrolment_created_at,
    courses.name          AS course_name,
    courses.created_at    AS course_created_at,
    courses.cohort
FROM enrolments
INNER JOIN courses
    ON enrolments.course_id = courses.id
```

#### Source breakdown

Regarding descriptive names:
 - **For (4)**: Holywell, Celko, Mozilla, Supabase.
 - **Against (0)**
 - **Not mentioned (5)**: PostgreSQL, PG.ai, Brooklyn, GitLab, Mazur.

Regarding consistent naming across tables:
 - **For (1)**: Celko.
 - **Against (0)**
 - **Not mentioned (8)**: PostgreSQL, PG.ai, Brooklyn, GitLab, Mazur, Mozilla, Supabase, Holywell.

Regarding not using descriptive prefixes:
 - **For (3)**: Holywell, Celko, Supabase.
 - **Against (1)**: PG.ai.
 - **Not mentioned (5)**: PostgreSQL, Brooklyn, Mazur, Mozilla, GitLab.
 
Regarding avoiding abbreviations:
 - **For (2)**: Holywell, Celko.
 - **Against (0)**
 - **Not mentioned (7)**: PostgreSQL, PG.ai, Brooklyn, GitLab, Mazur, Mozilla, Supabase.

Regarding no vague identifiers:
 - **For (3)**: Holywell, Celko, GitLab.
 - **Against (1)**: PG.ai.
 - **Not mentioned (5)**: PostgreSQL, Brooklyn, Mazur, Mozilla, Supabase.

### Use unquoted alphanumeric names

Use only lowercase alphanumeric characters for names separated by underscores, beginning always with a letter. For readability reasons, they should never end in an underscore or contain multiple consecutive underscores. Always avoid quoted identifiers for names, as they are prone to cause query errors (many database providers are case insensitive, so forgetting the quotes leads to incorrect column assignations).

##### Example

Bad:
```sql
CREATE TABLE "Enrolments" (
    id            INTEGER      PRIMARY KEY,
    1st_grade     NUMERIC(4,2),
    student__name VARCHAR,
    course_id_    INTEGER,
    "Is Active"   BOOLEAN,
    createdat     TIMESTAMP
)
```

Good:
```sql
CREATE TABLE enrolments (
    id           INTEGER      PRIMARY KEY,
    final_grade  NUMERIC(4,2),
    student_name VARCHAR,
    course_id    INTEGER,
    is_active    BOOLEAN,
    created_at   TIMESTAMP
)
```

#### Source breakdown

Regarding the use of only alphanumeric characters, starting with a letter:
 - **For (4)**: Holywell, Celko, PG.ai, Mozilla.
 - **Against (0)**
 - **Not mentioned (5)**: PostgreSQL, Brooklyn, GitLab, Mazur, Supabase.

Regarding no consecutive underscores:
 - **For (2)**: Holywell, Celko.
 - **Against (0)**
 - **Not mentioned (7)**: PostgreSQL, PG.ai, Brooklyn, GitLab, Mazur, Mozilla, Supabase.

Regarding quoted identifiers:
 - **For (3)**: Holywell, Celko, GitLab.
 - **Against (0)**
 - **Not mentioned (6)**: PostgreSQL, PG.ai, Brooklyn, Mazur, Mozilla, Supabase.

### Do not use reserved words as identifiers

Never use words such as `sum`, `count`, ... It will have to be quoted everywhere it is used or it will fail.

##### Example

Bad:
```sql
CREATE TABLE employees (
    id         INTEGER PRIMARY KEY,
    first_name VARCHAR,
    last_name  VARCHAR,
    group      VARCHAR
)
...
CREATE TABLE union (...)
```

Good:
```sql
CREATE TABLE employees (
    id         INTEGER PRIMARY KEY,
    first_name VARCHAR,
    last_name  VARCHAR,
    team_name  VARCHAR
)
...
CREATE TABLE merges (...)
```

#### Source breakdown

 - **For (3)**: Brooklyn, GitLab, Supabase.
 - **Against (0)**
 - **Not mentioned (6)**: Holywell, Celko, PostgreSQL, PG.ai, Mozilla, Mazur.

## Use collective, class or plural nouns for tables

The core reasoning behind this rule is that tables hold many rows, and each row represents one entity of something, so the table name should reflect its plural nature. Use collective nouns if possible (for example, `staff` over `employees`). Otherwise, use plural nouns.

The only exception to this rule would be for "singleton" tables: tables that will only contain and will contain one row. Those should be in singular to reflect their nature.

##### Example

Bad:
```sql
CREATE TABLE acrobats (...);
CREATE TABLE course (...);
```

Good:
```sql
CREATE TABLE troupe (...);
CREATE TABLE courses (...);
```

#### Source breakdown

 - **For (4)**: Holywell, Celko, Mazur, Supabase.
 - **Against (1)**: PG.ai.
 - **Not mentioned (4)**: Brooklyn, GitLab, PostgreSQL, Mozilla.

## Avoid concatenating names for relationship tables

Avoid using concatenated names for relationship tables, preferring a common descriptive name (`syllabus` instead of `course_materials`). If there is no common term, the concatenation will probably be the better fit.

##### Example

Bad:
```sql
CREATE TABLE student_courses (...);
CREATE TABLE actor_films (...);
```

Good:
```sql
CREATE TABLE enrolments (...);
CREATE TABLE castings (...);
```

#### Source breakdown

 - **For (2)**: Holywell, Celko.
 - **Against (0)**
 - **Not mentioned (7)**: Brooklyn, GitLab, PostgreSQL, Mozilla, Mazur, Supabase, PG.ai.

## Column naming rules

### Use singular names for columns

A column holds one value per row, so using singular reflects that. An exception can be made for `ARRAY` or `JSON` columns, but those should be avoided where possible.

##### Example

Bad:
```sql
CREATE TABLE players (
    id        INTEGER PRIMARY KEY,
    full_name VARCHAR,
    goals     INTEGER
)
```

Good:
```sql
CREATE TABLE players (
    id         INTEGER PRIMARY KEY,
    full_name  VARCHAR,
    goal_count INTEGER
)
```

#### Source breakdown

 - **For (4)**: Holywell, Celko, Supabase, PostgreSQL.
 - **Against (0)**
 - **Not mentioned (5)**: Brooklyn, GitLab, Mozilla, Mazur, PG.ai.

### Use standardised prefixes and suffixes

Use standardised prefixes and suffixes to convey meaning, making them understandable without the need to inspect their values and data types. 

A non comprehensive list of some standard prefixes and suffixes is as follows:

| Prefix/Suffix | Meaning |
| ------------- | ------- |
| `_id`         | A unique identifier, such as a foreign key (`course_id`) or any other ID |
| `_status`     | A flag or status |
| `_total`, `total_` | Total or sum of values |
| `_name`       | A name |
| `_size`       | A size |
| `_date`       | A date (for example, `2026-04-14`) |
| `_at`         | A date and time (for example, `2026-04-14 00:00:00`) |
| `_addr`, `_address` | An address, physical or intangible |
| `is_`, `has_` | A boolean flag |
| `_eur`, `_ms` | Numeric unit of the data (seconds, litres, ...) |
| `_month`, `_day` | Specific part of a date |

A table's own single-column primary key is named `id`. The `_id` suffix is used for foreign keys that reference another table's `id` (`enrolments.course_id` referencing `courses.id`), and for identifiers in general, such as external or business IDs (`stripe_customer_id`).

##### Example

Bad:
```sql
SELECT
    treatment,
    last_treatment,
    name
FROM patients
```

Good:
```sql
SELECT
    has_treatment,
    last_treatment_date,
    name
FROM patients
```

#### Source breakdown

Regarding general standardised suffixes and prefixes:
 - **For (5)**: Holywell, Celko, Brooklyn, GitLab, Mazur.
 - **Against (0)**
 - **Not mentioned (4)**: PostgreSQL, Mozilla, PG.ai, Supabase.

Regarding bare `id` primary keys with `_id` foreign keys:
 - **For (2)**: Brooklyn, Supabase.
 - **Against (2)**: Holywell, Celko (they avoid bare `id` columns).
 - **Not mentioned (5)**: PostgreSQL, Mozilla, PG.ai, GitLab, Mazur.

### Do not use the table name as a column name or prefix

Do not duplicate the table name in column name. This means two different things:
1. Do not use the table name as a prefix for a column (no `courses.course_name`).
2. Do not name a column identically to its table. A complete identical should never happen per previous rules, as tables should be collective/plural and columns singular, but things like `grades.grade` should be avoided.

##### Example

Bad:
```sql
CREATE TABLE grades (
    id           INTEGER      PRIMARY KEY,
    grade        NUMERIC(3,1),
    grade_letter VARCHAR(1)
)
```

Good:
```sql
CREATE TABLE grades (
    id     INTEGER      PRIMARY KEY,
    score  NUMERIC(3,1),
    letter VARCHAR(1)
)
```

#### Source breakdown

 - **For (3)**: Holywell, Celko, Supabase.
 - **Against (0)**
 - **Not mentioned (6)**: Brooklyn, GitLab, Mozilla, Mazur, PG.ai, PostgreSQL.
