# JOIN clauses

## Use `ON` for all join conditions; avoid `USING`

Always express join conditions with `ON`, even when the joining columns share the same name. Avoid `USING`, as it silently merges the columns into one in the result set, which can cause unexpected behaviour when adding aliases or when the query is refactored.

##### Example

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

#### Source breakdown

 - **For (2)**: Brooklyn, GitLab.
 - **Against  (1)**: PostgreSQL.
 - **Not mentioned (6)**: Holywell, Celko, PG.ai, Mozilla, Mazur, Supabase.

## Put the first-referenced table immediately after `ON`

In a join condition, write the table that was introduced first (the left-hand table) immediately after the `ON` keyword. This makes fan-out risk visible at a glance: if the left table's key is not on the left side of the condition, a row duplication bug may be hiding.

##### Example

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

#### Source breakdown

 - **For (2)**: Brooklyn, Mazur.
 - **Against (0)**
 - **Not mentioned (7)**: Holywell, Celko, PostgreSQL, PG.ai, Mozilla, GitLab, Supabase.

## For `INNER JOIN`, keep filter conditions in `WHERE`, not in the `JOIN` clause

The `ON` clause should express only the relationship between tables. Row-level filters belong in `WHERE`. Mixing business logic into `ON` obscures intent, as a reader scanning `ON` expects join conditions, not data filters.

As an important note, this rule applies only to `INNER JOIN`. With `LEFT JOIN`, moving a condition from `ON` to `WHERE` changes the semantics and effectively converts it into an inner join.

##### Example

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

#### Source breakdown

 - **For (1)**: Brooklyn.
 - **Against (0)**
 - **Not mentioned (8)**: Holywell, Celko, PostgreSQL, PG.ai, Mozilla, GitLab, Mazur, Supabase.
