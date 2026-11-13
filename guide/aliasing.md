# Alias rules

## Always use the `AS` keyword explicitly

Never omit the keyword when aliasing, as it provides readability.

##### Example

Bad:
```sql
SELECT
    customers.full_name                   customer_name,
    coalesce(sum(transactions.amount), 0) total_amount
FROM customers
LEFT JOIN archived_transactions transactions
    ON customers.id = transactions.customer_id
GROUP BY customers.full_name
```

Good:
```sql
SELECT
    customers.full_name                   AS customer_name,
    coalesce(sum(transactions.amount), 0) AS total_amount
FROM customers
LEFT JOIN archived_transactions AS transactions
    ON customers.id = transactions.customer_id
GROUP BY customers.full_name
```

#### Source breakdown

 - **For (9)**: Holywell, Celko, Supabase, Brooklyn, GitLab, Mozilla, Mazur, PG.ai, PostgreSQL.
 - **Against (0)**
 - **Not mentioned (0)**

## Always use aliasing for aggregated and other column expression

Always use an alias name for columns that are the result of aggregation, `CASE` statements, `sum()`, `count()`, ... Otherwise, depending of the database provider, the name could be inherited from the column used in the transformation or the function name.

##### Example

Bad:
```sql
SELECT
    team_name,
    count(*),
    avg(goals)
FROM matches
GROUP BY team_name
```

Good:
```sql
SELECT
    team_name,
    count(*)   AS total_matches,
    avg(goals) AS average_goals
FROM matches
GROUP BY team_name
```

#### Source breakdown

 - **For (4)**: Holywell, Celko, Brooklyn, Mazur.
 - **Against (0)**
 - **Not mentioned (5)**: GitLab, Mozilla, Supabase, PG.ai, PostgreSQL.

## Aim for clarity in table aliasing

Prefer using the full table name over an alias when the name is short enough to be readable. When aliasing is unavoidable (long or ambiguous names, self-joins), use a meaningful alias rather than an abbreviation or the initials of the table words (`schools`, not `s`; `districts`, not `sda`).

In a long query, an abbreviation forces the reader to scroll back to the `FROM` or `JOIN` clause to remember which table it stands for. A meaningful name identifies the table anywhere in the query, and makes the business logic easier to follow for developers new to the codebase.

Never use generic aliases that convey no meaning, such as `x` or `t1`.

##### Example

Bad:
```sql
SELECT
    s.name    AS school_name,
    sda.name  AS district_name,
    s.address AS school_address
FROM schools AS s
INNER JOIN school_district_areas AS sda
    ON s.district_code = sda.code
```

Good:
```sql
SELECT
    schools.name    AS school_name,
    districts.name  AS district_name,
    schools.address AS school_address
FROM schools
INNER JOIN school_district_areas AS districts
    ON schools.district_code = districts.code
```

#### Source breakdown

 - **For (3)**: Brooklyn, Mazur, GitLab.
 - **Against (2)**: Holywell, PostgreSQL.
 - **Not mentioned (4)**: Celko, Mozilla, Supabase, PG.ai.


## Column aliases should semantically relate to what they represent

Aliases should relate in some way to the object or expression they are aliasing. If they are computed values, they should reflect the transformation applied, and should reference the source that is being selected. Basically, a column alias should follow the same naming rules as declared column names in tables.

##### Example

Bad:
```sql
SELECT
    team_name,
    count(*)   AS total,
    avg(goals) AS goals_1
FROM matches
GROUP BY team_name
```

Good:
```sql
SELECT
    team_name,
    count(*)   AS total_matches,
    avg(goals) AS average_goals
FROM matches
GROUP BY team_name
```

#### Source breakdown

 - **For (7)**: Brooklyn, Mazur, GitLab, Supabase, Celko, Holywell, PG.ai.
 - **Against (0)**
 - **Not mentioned (2)**: Mozilla, PostgreSQL.

## Always qualify column references in queries with `JOIN`

In queries with multiple tables, always qualify every column reference with its table name or alias. This improves the clarity and readability of the query, as you are able to tell at a glance where each column comes from.

##### Example

Bad:
```sql
SELECT
    full_name,
    email,
    room_id,
    checked_in_at
FROM guests
INNER JOIN bookings
    ON guests.id = bookings.guest_id
```

Good:
```sql
SELECT
    guests.full_name,
    guests.email,
    bookings.room_id,
    bookings.checked_in_at
FROM guests
INNER JOIN bookings
    ON guests.id = bookings.guest_id
```

#### Source breakdown

 - **For (4)**: Brooklyn, Mazur, GitLab, Holywell.
 - **Against (0)**
 - **Not mentioned (5)**: Mozilla, PostgreSQL, Supabase, Celko, PG.ai.

## Omit table references for single-table queries

In single-table queries, omit the table prefix from column references as they do not provide any meaningful context while increasing the verbosity of the query.

##### Example

Bad:
```sql
SELECT
    schools.address,
    schools.urn AS school_urn
FROM schools
WHERE
    schools.cohort > 2024
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

#### Source breakdown

 - **For (2)**: Brooklyn, Mazur.
 - **Against (0)**
 - **Not mentioned (7)**: Mozilla, PostgreSQL, Supabase, Celko, PG.ai, GitLab, Holywell.
