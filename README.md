# SQL Style Guide

This is my SQL style guide for development.

The rules live in [`guide/`](guide/README.md), one file per topic, each with an explanation and Bad/Good examples.

## Examples

A query:

```sql
/* Customers who spent more than 500 EUR in 2025, biggest spenders first */
WITH paid_orders AS (
    SELECT
        customer_id,
        total_eur
    FROM orders
    WHERE
        is_paid IS true
        AND ordered_at BETWEEN TIMESTAMP '2025-01-01T00:00:00' AND TIMESTAMP '2025-12-31T23:59:59'
),

customer_totals AS (
    SELECT
        customer_id,
        count(*)       AS total_orders,
        sum(total_eur) AS total_spent_eur
    FROM paid_orders
    GROUP BY customer_id
    HAVING
        sum(total_eur) > 500
)

SELECT
    customers.id,
    customers.full_name,
    customer_totals.total_orders,
    customer_totals.total_spent_eur
FROM customers
INNER JOIN customer_totals
    ON customers.id = customer_totals.customer_id
ORDER BY customer_totals.total_spent_eur DESC
```

A table:

```sql
CREATE TABLE bookings (
    id               INTEGER       NOT NULL,
    room_id          INTEGER       NOT NULL,
    guest_id         INTEGER       NOT NULL,
    check_in_date    DATE          NOT NULL,
    check_out_date   DATE          NOT NULL,
    nightly_rate_eur NUMERIC(8, 2) NOT NULL,
    is_paid          BOOLEAN       NOT NULL DEFAULT false,
    created_at       TIMESTAMP     NOT NULL,

    PRIMARY KEY (id),
    FOREIGN KEY (room_id) REFERENCES rooms (id),
    FOREIGN KEY (guest_id) REFERENCES guests (id),
    CONSTRAINT chk_bookings_nightly_rate_eur_min CHECK (nightly_rate_eur > 0),
    CONSTRAINT chk_bookings_dates_order          CHECK (check_out_date > check_in_date)
);
```

## Use it with AI coding agents

The guide is also packaged as an [agent skill](skills/sql-style-guide/SKILL.md) that writes, reviews and reformats SQL following these rules. Install it with the [`skills`](https://github.com/vercel-labs/skills) CLI:

```bash
# Current project only
npx skills add diagmatrix/sql-style-guide

# All your projects
npx skills add diagmatrix/sql-style-guide -g
```

Add `-a claude-code` (or another agent's name) to install it for one agent only.

Once it is installed, ask for SQL as usual ("write a query that...", "review `reports/churn.sql`", "clean up this migration without changing what it does"). Agents don't always open a skill for quick SQL tasks, so if you want it applied every time, add a line like this to your agent instructions (for example `CLAUDE.md` or `AGENTS.md`):

```
Whenever you write, edit or review SQL, use the sql-style-guide skill first.
```
