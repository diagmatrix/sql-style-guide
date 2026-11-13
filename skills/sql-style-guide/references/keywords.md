# Keyword rules

## Use uppercase for reserved words

Always use **UPPERCASE** for reserved words (`SELECT`, `WHERE`, `AS`, ...). This differentiates the SQL logic from the tables and columns used in the queries and table definitions.

##### Example

Bad:
```sql
select
    address,
    urn as school_urn
from schools
where cohort > 2024
```

Good:
```sql
SELECT
    address,
    urn AS school_urn
FROM schools
WHERE
    cohort > 2024
```

## Full keyword forms for reserved words

Always use the full keyword forms instead of the abbreviation.

##### Example

Bad:
```sql
SELECT
    *
FROM stores
JOIN employees
    ON stores.id = employees.store_id
WHERE
    employees.hire_year > 2020
```

Good:
```sql
SELECT
    *
FROM stores
INNER JOIN employees
    ON stores.id = employees.store_id
WHERE
    employees.hire_year > 2020
```

## Avoid vendor-specific keywords

If an ANSI SQL keyword exists for a function or procedure, use it instead of a vendor-specific one. 

##### Example

Bad:
```sql
SELECT
    full_name,
    /* Specific to PostgreSQL */
    date_part('year', hired_at) AS hire_year
FROM employees
```

Good:
```sql
SELECT
    full_name,
    /* ANSI SQL function */
    extract(YEAR FROM hired_at) AS hire_year
FROM employees
```

## Use lowercase for functions

Treat functions as identifiers and always write them in **lowercase**: `count()` instead of `COUNT()`.

Mixing cases is common: single-word functions tend to be written in uppercase (`COALESCE`, `SUM`, ...), while multi-word ones are written in lowercase (`string_agg`, `json_array_elements`). Always using lowercase removes that inconsistency.

It also keeps uppercase meaningful. The [Bouma](https://en.wikipedia.org/wiki/Bouma) (the outline shape) of an uppercase word is a rectangle, which makes it stand out from regular words. Reserving uppercase for reserved words keeps the structure of the query easy to see; using it for functions too dilutes that.

##### Example

Bad:
```sql
SELECT
    warehouse_id,
    COUNT(*)                        AS total_shipments,
    string_agg(tracking_code, ', ') AS tracking_codes
FROM shipments
GROUP BY warehouse_id
```

Good:
```sql
SELECT
    warehouse_id,
    count(*)                        AS total_shipments,
    string_agg(tracking_code, ', ') AS tracking_codes
FROM shipments
GROUP BY warehouse_id
```

## Use uppercase for data types

Data types are written in **UPPERCASE**, the same as reserved words, both in table definitions and in type casts. Mixing cases makes definitions inconsistent, so pick one and stick to it.

User-defined types (enums, domains, composite types) are identifiers rather than reserved words, so they follow the naming rules instead: lowercase, with underscores.

##### Example

Bad:
```sql
CREATE TABLE users (
    id         integer      PRIMARY KEY,
    email      varchar(255) NOT NULL,
    created_at timestamp    DEFAULT CURRENT_TIMESTAMP,
    is_active  boolean      NOT NULL
)
```

Good:
```sql
CREATE TABLE users (
    id         INTEGER      PRIMARY KEY,
    email      VARCHAR(255) NOT NULL,
    created_at TIMESTAMP    DEFAULT CURRENT_TIMESTAMP,
    is_active  BOOLEAN      NOT NULL
)
```
