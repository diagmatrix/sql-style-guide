# Schema objects

## DDL-01 Every table must have at least one key

Every table must define at least one key to uniquely identify each row. A table
without a key has no reliable way to reference, update, or deduplicate its rows.

### Example

Bad:

```sql
CREATE TABLE enrolments (
    student_id  INTEGER   NOT NULL,
    course_id   INTEGER   NOT NULL,
    enrolled_at TIMESTAMP NOT NULL
);
```

Good:

```sql
CREATE TABLE enrolments (
    student_id  INTEGER   NOT NULL,
    course_id   INTEGER   NOT NULL,
    enrolled_at TIMESTAMP NOT NULL,

    PRIMARY KEY (student_id, course_id)
);
```

### Source breakdown

- **For (2)**: Holywell, Celko.
- **Against (0)**
- **Not mentioned (7)**: PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab, Mazur,
  Supabase.

## DDL-02 Give constraints explicit custom names

Name constraints explicitly so that error messages and schema introspection are
meaningful. The exceptions are `UNIQUE`, `PRIMARY KEY`, and `FOREIGN KEY` —
these can rely on database-generated names.

### Example

Bad:

```sql
CREATE TABLE products (
    id    INTEGER NOT NULL,
    sku   TEXT    NOT NULL,
    price NUMERIC NOT NULL,

    PRIMARY KEY (id),
    CHECK (price > 0)
);
```

Good:

```sql
CREATE TABLE products (
    id    INTEGER NOT NULL,
    sku   TEXT    NOT NULL,
    price NUMERIC NOT NULL,

    PRIMARY KEY (id),
    CONSTRAINT chk_products_price_min CHECK (price > 0)
);
```

### Source breakdown

- **For (2)**: Holywell, Celko.
- **Against (0)**
- **Not mentioned (7)**: PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab, Mazur,
  Supabase.

## DDL-03 Place multi-column constraints near both columns; table-level constraints at the end

Multi-column constraints should be positioned as close as possible to all the
columns they reference. Table-level constraints (those that span multiple
columns or apply to the table as a whole) go at the end of the `CREATE TABLE`
definition, after all column declarations.

### Example

Bad:

```sql
CREATE TABLE loans (
    PRIMARY KEY (member_id, book_id),
    member_id   INTEGER   NOT NULL,
    book_id     INTEGER   NOT NULL,
    borrowed_at TIMESTAMP NOT NULL,
    FOREIGN KEY (member_id) REFERENCES members (id)
);
```

Good:

```sql
CREATE TABLE loans (
    member_id   INTEGER   NOT NULL,
    book_id     INTEGER   NOT NULL,
    borrowed_at TIMESTAMP NOT NULL,

    PRIMARY KEY (member_id, book_id),
    FOREIGN KEY (member_id) REFERENCES members (id)
);
```

### Source breakdown

- **For (2)**: Holywell, Celko.
- **Against (0)**
- **Not mentioned (7)**: PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab, Mazur,
  Supabase.

## DDL-04 Declare `PRIMARY KEY` first, immediately after the column list

The `PRIMARY KEY` constraint should be the first constraint declared in a
`CREATE TABLE` statement, placed right after the last column definition.

### Example

Bad:

```sql
CREATE TABLE projects (
    id        INTEGER NOT NULL,
    name      TEXT    NOT NULL,
    client_id INTEGER NOT NULL,
    FOREIGN KEY (client_id) REFERENCES clients (id),
    PRIMARY KEY (id)
);
```

Good:

```sql
CREATE TABLE projects (
    id        INTEGER NOT NULL,
    name      TEXT    NOT NULL,
    client_id INTEGER NOT NULL,

    PRIMARY KEY (id),
    FOREIGN KEY (client_id) REFERENCES clients (id)
);
```

### Source breakdown

- **For (2)**: Holywell, Celko.
- **Against (0)**
- **Not mentioned (7)**: PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab, Mazur,
  Supabase.

## DDL-05 Keep each constraint single-focused

Each `CHECK()` constraint should validate exactly one rule, do not combine
multiple conditions into a single constraint. Use `CHECK()` constraints to
enforce numerical ranges, with at minimum a `> 0` guard where a value must be
positive.

### Example

Bad:

```sql
CREATE TABLE discounts (
    id           INTEGER NOT NULL,
    product_id   INTEGER NOT NULL,
    percentage   NUMERIC NOT NULL,
    min_quantity INTEGER NOT NULL,

    PRIMARY KEY (id),
    CONSTRAINT chk_discount_values CHECK (percentage >= 0 AND percentage <= 100 AND min_quantity > 0)
);
```

Good:

```sql
CREATE TABLE discounts (
    id           INTEGER NOT NULL,
    product_id   INTEGER NOT NULL,
    percentage   NUMERIC NOT NULL,
    min_quantity INTEGER NOT NULL,

    PRIMARY KEY (id),
    CONSTRAINT chk_discounts_percentage_min   CHECK (percentage >= 0),
    CONSTRAINT chk_discounts_percentage_max   CHECK (percentage <= 100),
    CONSTRAINT chk_discounts_min_quantity_min CHECK (min_quantity > 0)
);
```

### Source breakdown

- **For (2)**: Holywell, Celko.
- **Against (0)**
- **Not mentioned (7)**: PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab, Mazur,
  Supabase.

## DDL-06 Use `LIKE` and `SIMILAR TO` for string format validation in constraints

When constraining the format of a text column, use `LIKE` for simple pattern
matching or `SIMILAR TO` for regex-style patterns. These are portable SQL
standard expressions.

### Example

Bad:

```sql
CREATE TABLE suppliers (
    id         INTEGER NOT NULL,
    email      TEXT    NOT NULL,
    vat_number TEXT    NOT NULL,

    PRIMARY KEY (id)
);
```

Good:

```sql
CREATE TABLE suppliers (
    id         INTEGER NOT NULL,
    email      TEXT    NOT NULL,
    vat_number TEXT    NOT NULL,

    PRIMARY KEY (id),
    CONSTRAINT chk_suppliers_email_format      CHECK (email LIKE '%@%.%'),
    CONSTRAINT chk_suppliers_vat_number_format CHECK (vat_number SIMILAR TO '[A-Z]{2}[0-9]{9}')
);
```

### Source breakdown

- **For (2)**: Holywell, Celko.
- **Against (0)**
- **Not mentioned (7)**: PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab, Mazur,
  Supabase.

## DDL-07 Always explicitly name the columns a view exposes

Never rely on `SELECT *` in a view definition. Always list the columns the view
exposes by name. This prevents silent breakage when the underlying table gains,
loses, or reorders columns.

### Example

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

### Source breakdown

- **For (1)**: Celko.
- **Against (0)**
- **Not mentioned (8)**: Holywell, PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab,
  Mazur, Supabase.

## DDL-08 Avoid view proliferation

Do not create views as a default abstraction layer. Each view should exist
because it solves a specific, recurring problem, such as enforcing access
control, encapsulating a complex join, or standardising a derived metric. If a
view has no clear purpose, it becomes maintenance burden without value.

### Example

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

### Source breakdown

- **For (1)**: Celko.
- **Against (0)**
- **Not mentioned (8)**: Holywell, PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab,
  Mazur, Supabase.
