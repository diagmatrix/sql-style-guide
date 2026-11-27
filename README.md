# SQL Style Guide

[![Lint](https://github.com/diagmatrix/sql-style-guide/actions/workflows/lint.yml/badge.svg)](https://github.com/diagmatrix/sql-style-guide/actions/workflows/lint.yml)

This is my SQL style guide for development. These guidelines are meant to
inform, not dictate. Every project comes with its own constraints: efficiency
concerns, business rules, specific contexts... There will be times when
following a rule to the letter does more harm than good.

## Rules

The rules live in [`guide/`](guide/), one file per group, each with an
explanation and Bad/Good examples. Every rule has a stable code made of its
group prefix and a number (for example `LAY-03`).

1. [Layout](guide/layout.md) (`LAY`)
2. [Keywords and functions](guide/keywords.md) (`KWD`)
3. [Naming](guide/naming.md) (`NAM`)
4. [Aliasing and references](guide/aliasing.md) (`ALI`)
5. [Comments](guide/comments.md) (`CMT`)
6. [Data types and literals](guide/data-types.md) (`TYP`)
7. [Expressions and conditions](guide/expressions.md) (`EXP`)
8. [Query structure](guide/structure.md) (`STR`)
9. [Clauses](guide/clauses.md) (`CLS`)
10. [Schema objects](guide/schema.md) (`DDL`)

## Acknowledgements

This guide is an adaptation of
[Aircury's SQL style guide](https://github.com/aircury/sql-style-guide), which I
also created. Most of its rules come from other SQL style guides, where a
decision has been made with some reasoning and no other guide contradicts it.
Others come mostly from my opinion or are not a consensus between the guides.
The guides it compares against are:

- *Joe Celko's SQL Programming Style* (Joe Celko, 2005)
- [Simon Holywell's SQL Style Guide](https://www.sqlstyle.guide/)
- The [PostgreSQL documentation](https://www.postgresql.org/docs/current/sql.html)
- [Postgres.ai's rules](https://postgres.ai/rules/rules-pages)
- [Mozilla's SQL Style Guide](https://docs.telemetry.mozilla.org/concepts/sql_style)
- [Brooklyn Data Co.'s SQL Style Guide](https://github.com/brooklyn-data/co/blob/main/sql_style_guide.md)
- [GitLab's SQL Style Guide](https://handbook.gitlab.com/handbook/enterprise-data/platform/sql-style-guide/)
- [Matt Mazur's SQL Style Guide](https://github.com/mattm/sql-style-guide)
- [Supabase's Postgres SQL Style Guide](https://supabase.com/docs/guides/getting-started/ai-prompts/code-format-sql)

To learn more about these sources and where each one stands on every rule,
head to [Aircury's SQL style guide](https://github.com/aircury/sql-style-guide).

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

The guide is also packaged as an [agent skill](guide/SKILL.md) that writes,
reviews and reformats SQL following these rules. Install it with the
[`skills`](https://github.com/vercel-labs/skills) CLI:

```bash
# Current project only
npx skills add diagmatrix/sql-style-guide

# All your projects
npx skills add diagmatrix/sql-style-guide -g
```

Add `-a claude-code` (or another agent's name) to install it for one agent only.

Once it is installed, ask for SQL as usual ("write a query that...", "review
`reports/churn.sql`", "clean up this migration without changing what it does").
Agents don't always open a skill for quick SQL tasks, so if you want it applied
every time, add a line like this to your agent instructions (for example
`CLAUDE.md` or `AGENTS.md`):

```text
Whenever you write, edit or review SQL, use the sql-style-guide skill first.
```

## Contributing

Markdown files are linted with
[markdownlint](https://github.com/DavidAnson/markdownlint), and a GitHub
Actions workflow fails any push or pull request with lint errors. See
[`CONTRIBUTING.md`](CONTRIBUTING.md) for how to run it locally and how rules
are written.
