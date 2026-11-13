# Data types

## Use `NUMERIC` or `DECIMAL` for monetary and precision values

When storing monetary amounts or any value where exact precision matters, use `NUMERIC` or `DECIMAL`. Never use `REAL` or `FLOAT` for these — they use binary floating-point arithmetic and can introduce rounding errors. Only reach for `REAL` or `FLOAT` when the computation genuinely requires floating-point math (e.g. scientific calculations).

##### Example

Bad:
```sql
CREATE TABLE invoices (
    id       INTEGER   NOT NULL,
    order_id INTEGER   NOT NULL,
    amount   FLOAT     NOT NULL,
    discount REAL      NOT NULL,
    paid_at  TIMESTAMP NOT NULL,

    PRIMARY KEY (id)
);
```

Good:
```sql
CREATE TABLE invoices (
    id       INTEGER        NOT NULL,
    order_id INTEGER        NOT NULL,
    amount   NUMERIC(10, 2) NOT NULL,
    discount DECIMAL(5, 2)  NOT NULL,
    paid_at  TIMESTAMP      NOT NULL,

    PRIMARY KEY (id)
);
```

#### Source breakdown

 - **For (2)**: Holywell, Celko.
 - **Against (0)**
 - **Not mentioned (7)**: PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab, Mazur, Supabase.

## Prefer `BOOLEAN` for logical values: use `true`/`false` instead of `1`/`0`

Use the `BOOLEAN` type for columns that represent a true/false state. Store and compare values using the literals `true` and `false` rather than integer substitutes `1` and `0`. This makes intent explicit and avoids ambiguity when reading queries.

##### Example

Bad:
```sql
SELECT
    full_name,
    is_active
FROM users
WHERE
    is_active = 1
    AND is_verified = 0
```

Good:
```sql
SELECT
    full_name,
    is_active
FROM users
WHERE
    is_active IS true
    AND is_verified IS false
```

#### Source breakdown

 - **For (1)**: GitLab.
 - **Against (0)**
 - **Not mentioned (8)**: Holywell, Celko, PostgreSQL, PG.ai, Mozilla, Brooklyn, Mazur, Supabase.

## Default values should match the column type

A column's default value must be the same type as the column. 

##### Example

Bad:
```sql
CREATE TABLE bookings (
    room_id     INTEGER   NOT NULL,
    guest_id    INTEGER   NOT NULL,
    booked_at   TIMESTAMP NOT NULL,
    is_paid     BOOLEAN   NOT NULL DEFAULT 1,
    total_price NUMERIC   NOT NULL DEFAULT '0',

    PRIMARY KEY (room_id, booked_at)
);
```

Good:
```sql
CREATE TABLE bookings (
    room_id     INTEGER   NOT NULL,
    guest_id    INTEGER   NOT NULL,
    booked_at   TIMESTAMP NOT NULL,
    is_paid     BOOLEAN   NOT NULL DEFAULT false,
    total_price NUMERIC   NOT NULL DEFAULT 0,

    PRIMARY KEY (room_id, booked_at)
);
```

#### Source breakdown

 - **For (2)**: Holywell, Celko.
 - **Against (0)**
 - **Not mentioned (7)**: PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab, Mazur, Supabase.

## Use ISO 8601 format for date/time values and ISO temporal syntax in queries

Store dates and times using ISO 8601 format: `YYYY-MM-DDTHH:MM:SS.SSSSS`. Use ISO temporal syntax when expressing date/time literals in queries rather than vendor-specific formats or implicit casts.

##### Example

Bad:
```sql
SELECT
    *
FROM flights
WHERE
    departed_at > '01/09/2024'
    AND arrived_at < '31/07/2025 23:59:59'
```

Good:
```sql
SELECT
    *
FROM flights
WHERE
    departed_at > DATE '2024-09-01'
    AND arrived_at < TIMESTAMP '2025-07-31T23:59:59'
```

#### Source breakdown

 - **For (4)**: Holywell, Celko, PG.ai, Supabase.
 - **Against (0)**
 - **Not mentioned (5)**: PostgreSQL, Mozilla, Brooklyn, GitLab, Mazur.
