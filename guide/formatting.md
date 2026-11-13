# Formatting

## Left align all keywords, starting on the same character boundary

Align all clause keywords to the same left margin instead of right-aligning them to a common boundary (**river** alignment).

Left-aligned keywords are simpler to write and maintain. No padding arithmetic is needed, and adding a longer keyword (switching `JOIN` to `INNER JOIN`, for example) does not force re-alignment of everything else on the page. It also works without editor tooling, which makes it easier to enforce consistently across a team.

##### Example

Bad:
```sql
SELECT full_name,
       email,
       created_at
  FROM customers
 WHERE country_code = 'ES'
```

Good:
```sql
SELECT
    full_name,
    email,
    created_at
FROM customers
WHERE
    country_code = 'ES'
```

#### Source breakdown

 - **For river (2)**: Celko, Holywell.
 - **For left align (5)**: PG.ai, GitLab, Mozilla, Brooklyn, Mazur.
 - **No mention (2)**: PostgreSQL, Supabase.

## Each major clause on its own line

Each major SQL clause (`SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, etc.) must start on its own line. This separates the logical structure of the query from its arguments and makes the query easier to scan, modify, and review.

##### Example

Bad:
```sql
SELECT full_name, email FROM employees WHERE hire_year > 2020 GROUP BY full_name, email
```

Good:
```sql
SELECT
    full_name,
    email
FROM employees
WHERE
    hire_year > 2020
GROUP BY full_name, email
```

#### Source breakdown

 - **For (7)**: Holywell, PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab, Mazur.
 - **Against (0)**
 - **Not mentioned (2)**: Celko, Supabase.

## Place `SELECT`, `WHERE` and `HAVING` arguments below the keyword

The arguments of `SELECT`, `WHERE` and `HAVING` clauses must each be on their own line below the keyword, indented one level, including the first one. This applies even when the clause has a single argument.

All other clauses (`FROM`, `JOIN`, `GROUP BY`, `ORDER BY`, ...) keep their arguments on the same line as the keyword. `JOIN` conditions are the one exception, and follow [their own rule](#place-join-conditions-below-the-keyword).

##### Example

Bad:
```sql
SELECT name, sku, price
FROM products
WHERE price > 100
```

Bad (single argument sharing the line):
```sql
SELECT count(*) AS total_products
FROM products
WHERE price > 100
```

Good:
```sql
SELECT
    name,
    sku,
    price
FROM products
WHERE
    price > 100
ORDER BY price, name
```

#### Source breakdown

Regarding a single argument also being placed below the root keyword:

 - **For (0)**
 - **Against (3)**: Mozilla, PG.ai, Brooklyn (they allow a single argument to share the line with the keyword).
 - **Not mentioned (6)**: Holywell, Celko, PostgreSQL, GitLab, Mazur, Supabase.

Regarding multiple arguments each on their own line, including the first:

 - **For (4)**: Mozilla, PG.ai, Brooklyn, Mazur.
 - **Against (2)**: Holywell, Celko (the river pattern places the first argument on the same line as the keyword).
 - **Not mentioned (3)**: PostgreSQL, GitLab, Supabase.

---

## Use 4 spaces for indentation

Use **4 spaces** for each level of indentation. This is the most common choice among the sources consulted, providing enough visual separation to clearly distinguish nested levels without excessive horizontal growth.

##### Example

Bad:
```sql
SELECT
  full_name,
  email
FROM patients
WHERE
  age >= 65
```

Good:
```sql
SELECT
    full_name,
    email
FROM patients
WHERE
    age >= 65
```

#### Source breakdown

 - **For 4 spaces (2)**: Brooklyn, Holywell.
 - **For 2 spaces (1)**: GitLab.
 - **Not mentioned (6)**: Celko, PostgreSQL, PG.ai, Mozilla, Mazur, Supabase.

## Place `JOIN` conditions below the keyword

When joining tables, place the `ON` keyword on a new line, indented one level under the `JOIN` keyword, even when there is a single condition. This clearly separates the join condition from the join declaration and makes it easier to identify at a glance.

##### Example

Bad:
```sql
SELECT
    customers.full_name,
    products.name AS product_name
FROM customers
INNER JOIN orders ON customers.id = orders.customer_id AND customers.region_code = orders.region_code
INNER JOIN products ON orders.product_id = products.id
```

Good:
```sql
SELECT
    customers.full_name,
    products.name AS product_name
FROM customers
INNER JOIN orders
    ON customers.id = orders.customer_id
    AND customers.region_code = orders.region_code
INNER JOIN products
    ON orders.product_id = products.id
```

#### Source breakdown

 - **For indented `ON` (1)**: Mozilla.
 - **For inline single / indented multiple (2)**: Brooklyn, Mazur.
 - **Not mentioned (6)**: Holywell, Celko, PostgreSQL, PG.ai, GitLab, Supabase.

## Terminate the line after an open parenthesis

When a parenthesised construct spans multiple lines, the opening parenthesis should terminate the line it starts on. The contents should be indented one level, and the closing parenthesis should sit on its own line, aligned with the opening construct.

##### Example

Bad:
```sql
WITH active_drivers AS (SELECT
    id,
    full_name
    FROM drivers
    WHERE is_active IS true)
SELECT
    *
FROM active_drivers
```

Good:
```sql
WITH active_drivers AS (
    SELECT
        id,
        full_name
    FROM drivers
    WHERE
        is_active IS true
)

SELECT
    *
FROM active_drivers
```

#### Source breakdown

 - **For (3)**: Holywell, PG.ai, Mozilla.
 - **Against (0)**
 - **Not mentioned (6)**: Celko, PostgreSQL, Brooklyn, GitLab, Mazur, Supabase.

## Indent CTE bodies one level inside their parentheses

Start the first CTE on the same line as `WITH`, and each following CTE at the same level as `WITH`. Indent the body of every CTE one level inside its parentheses, and put the closing parenthesis on its own line at the level of the CTE name. Leave a blank line between each CTE, and after the last one, for visual separation.

##### Example

Bad:
```sql
WITH
active_customers AS (
SELECT id, full_name FROM customers WHERE is_active IS true
),
recent_orders AS (
SELECT customer_id, total_amount FROM orders WHERE created_at > '2024-01-01'
)
SELECT
    *
FROM active_customers
INNER JOIN recent_orders
    ON active_customers.id = recent_orders.customer_id
```

Good:
```sql
WITH active_customers AS (
    SELECT
        id,
        full_name
    FROM customers
    WHERE
        is_active IS true
),

recent_orders AS (
    SELECT
        customer_id,
        total_amount
    FROM orders
    WHERE
        created_at > '2024-01-01'
)

SELECT
    *
FROM active_customers
INNER JOIN recent_orders
    ON active_customers.id = recent_orders.customer_id
```

#### Source breakdown

 - **For (2)**: PostgreSQL, Brooklyn.
 - **Against (0)**
 - **Not mentioned (7)**: Holywell, Celko, PG.ai, Mozilla, GitLab, Mazur, Supabase.

## Use one space between all language tokens

Always use exactly one space between language tokens: before and after operators (`=`, `>`, `<`, `!=`, ...) and after commas. Never crowd tokens together or add extra padding spaces to align values across lines. The only exceptions are [aligning `AS` keywords](#align-as-keywords-in-a-select-list) and [aligning data types and constraints in `CREATE TABLE`](#align-data-types-and-constraints-in-create-table).

##### Example

Bad:
```sql
SELECT
    full_name,
    salary*1.05  AS raised_salary,
    bonus/12     AS monthly_bonus
FROM employees
WHERE
    salary>30000
    AND is_active=true
```

Good:
```sql
SELECT
    full_name,
    salary * 1.05 AS raised_salary,
    bonus / 12    AS monthly_bonus
FROM employees
WHERE
    salary > 30000
    AND is_active IS true
```

#### Source breakdown

 - **For (3)**: Holywell, Celko, PostgreSQL.
 - **Against (0)**
 - **Not mentioned (6)**: PG.ai, Mozilla, Brooklyn, GitLab, Mazur, Supabase.

## Align `AS` keywords in a `SELECT` list

When several columns in the same `SELECT` list have an alias, pad the expressions so that all the `AS` keywords line up in the same column, one space after the longest expression. Columns without an alias are left as they are.

Aligned aliases make the output columns of a query easy to scan: the expressions are on the left and the resulting column names on the right.

##### Example

Bad:
```sql
SELECT
    region,
    count(*) AS total_sales,
    avg(amount) AS average_amount,
    max(sold_at) AS last_sold_at
FROM sales
GROUP BY region
```

Good:
```sql
SELECT
    region,
    count(*)     AS total_sales,
    avg(amount)  AS average_amount,
    max(sold_at) AS last_sold_at
FROM sales
GROUP BY region
```

## Align data types and constraints in `CREATE TABLE`

In a `CREATE TABLE` definition, pad the column names so that all the data types start in the same column, one space after the longest column name. Do the same with the data types, so that the column constraints (`NOT NULL`, `DEFAULT`, ...) also start in the same column.

For named table constraints at the end of the definition, pad the constraint names so that the constraint bodies (`CHECK (...)`, `FOREIGN KEY (...)`, ...) line up as well.

As with [aligned `AS` keywords](#align-as-keywords-in-a-select-list), this turns the definition into a table that can be read in columns: names, types and constraints.

##### Example

Bad:
```sql
CREATE TABLE students (
    id INTEGER NOT NULL,
    full_name VARCHAR(255) NOT NULL,
    cohort INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (id),
    CONSTRAINT chk_students_cohort_min CHECK (cohort > 2000),
    CONSTRAINT chk_students_full_name_not_empty CHECK (full_name != '')
);
```

Good:
```sql
CREATE TABLE students (
    id         INTEGER      NOT NULL,
    full_name  VARCHAR(255) NOT NULL,
    cohort     INTEGER      NOT NULL,
    created_at TIMESTAMP    DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (id),
    CONSTRAINT chk_students_cohort_min          CHECK (cohort > 2000),
    CONSTRAINT chk_students_full_name_not_empty CHECK (full_name != '')
);
```

## Don't add extra spaces inside parentheses or before function parentheses

Do not pad the inside of parentheses with spaces. Do not add a space between a function name and its opening parenthesis.

##### Example

Bad:
```sql
SELECT
    count( * )         AS total_transactions,
    coalesce( fee, 0 ) AS fee
FROM transactions
WHERE
    currency IN ( 'EUR', 'USD' )
```

Good:
```sql
SELECT
    count(*)         AS total_transactions,
    coalesce(fee, 0) AS fee
FROM transactions
WHERE
    currency IN ('EUR', 'USD')
```

#### Source breakdown

 - **For (3)**: Brooklyn, Mazur, PostgreSQL.
 - **Against (0)**
 - **Not mentioned (6)**: Holywell, Celko, PG.ai, Mozilla, GitLab, Supabase.

## Put `AND` / `OR` at the beginning of the next line

In a `WHERE`, `HAVING`, or `ON` clause with multiple conditions, place `AND` and `OR` at the **beginning** of the next line rather than at the end of the current one. The operators keep the same indentation as the first condition.

Placing boolean operators at the start of the line makes conditions easier to scan, as each condition begins with a clear logical marker, and the boundary between conditions is immediately visible. It also keeps diffs small: adding, removing or commenting out a condition touches only its own line.

##### Example

Bad:
```sql
SELECT
    *
FROM properties
WHERE
    price > 200000 AND
    is_available IS true AND
    city IS NOT NULL
```

Good:
```sql
SELECT
    *
FROM properties
WHERE
    price > 200000
    AND is_available IS true
    AND city IS NOT NULL
```

#### Source breakdown

 - **For (3)**: PG.ai, Mozilla, Brooklyn.
 - **Against (2)**: Celko, Mazur.
 - **Not mentioned (4)**: Holywell, PostgreSQL, GitLab, Supabase.

## Add new lines after semicolons and keyword definitions

Leave a blank line after each semicolon to separate queries. Also leave a new line after each keyword definition to clearly demarcate logical blocks within a script.

##### Example

Bad:
```sql
SELECT count(*) AS total FROM students; SELECT count(*) AS total FROM courses;
```

Good:
```sql
SELECT
    count(*) AS total
FROM students;

SELECT
    count(*) AS total
FROM courses;
```

#### Source breakdown

 - **For (2)**: Holywell, Celko.
 - **Against (0)**
 - **Not mentioned (7)**: PostgreSQL, PG.ai, Mozilla, Brooklyn, GitLab, Mazur, Supabase.

## Use blank lines between logically distinct sections

Leave a blank line between logically distinct sections of code, such as between CTEs, or between groups of related conditions in a long `WHERE` clause. This follows general coding best practices.

##### Example

Bad:
```sql
WITH active_students AS (
    SELECT
        id,
        full_name
    FROM students
    WHERE
        is_active IS true
)
SELECT
    active_students.full_name,
    count(enrolments.course_id) AS total_courses
FROM active_students
INNER JOIN enrolments
    ON active_students.id = enrolments.student_id
GROUP BY active_students.full_name
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
)

SELECT
    active_students.full_name,
    count(enrolments.course_id) AS total_courses
FROM active_students
INNER JOIN enrolments
    ON active_students.id = enrolments.student_id
GROUP BY active_students.full_name
```

#### Source breakdown

 - **For (4)**: Holywell, Celko, Brooklyn, Mazur.
 - **Against (0)**
 - **Not mentioned (5)**: PostgreSQL, PG.ai, Mozilla, GitLab, Supabase.

## Use trailing commas

In lists of arguments (columns in `SELECT`, values in `IN`, ...) place commas at the **end** of each line (trailing) rather than the **beginning** (leading).

Trailing commas are the more widely supported convention and feel natural to most developers. A missing comma is already flagged by most IDEs before the query is run.

##### Example

Bad:
```sql
SELECT
      subject
    , priority
    , status
FROM tickets
```

Good:
```sql
SELECT
    subject,
    priority,
    status
FROM tickets
```

#### Source breakdown

 - **For trailing commas (4)**: Celko, Mozilla, PG.ai, Mazur.
 - **For leading commas (1)**: Brooklyn.
 - **Not mentioned (4)**: Holywell, PostgreSQL, GitLab, Supabase.

## Keep lines under 120 characters

Lines should not exceed **120 characters**, to keep queries readable in side-by-side views, code reviews, and editors with limited width. The limit is deliberately generous: in queries with several levels of subqueries, indentation alone takes up a good part of the line. Treat it as a ceiling, not a target.

When a line goes over the limit, wrap the expression in parentheses. Break the line after the opening parenthesis, indent the content one level, and put the closing parenthesis on its own line, aligned with the line that opened it. Inside, break at logical points such as before an operator or after a comma, starting each line with the operator in the same way as `AND` / `OR`. The parentheses group the expression into one visible block, with a clear start and end.

Function calls already have their own parentheses, so break inside those instead (see [Terminate the line after an open parenthesis](#terminate-the-line-after-an-open-parenthesis)). If an expression is still hard to read after breaking it, move it into a CTE.

##### Example

Bad:
```sql
SELECT
    students.full_name,
    assessments.exam_score * 0.5 + assessments.coursework_score * 0.3 + assessments.attendance_score * 0.15 + assessments.participation_score * 0.05 AS final_grade
FROM students
INNER JOIN assessments
    ON students.id = assessments.student_id
```

Good:
```sql
SELECT
    students.full_name,
    (
        assessments.exam_score * 0.5
        + assessments.coursework_score * 0.3
        + assessments.attendance_score * 0.15
        + assessments.participation_score * 0.05
    ) AS final_grade
FROM students
INNER JOIN assessments
    ON students.id = assessments.student_id
```

#### Source breakdown

 - **For (2)**: Brooklyn, GitLab.
 - **Against (0)**
 - **Not mentioned (7)**: Holywell, Celko, PostgreSQL, PG.ai, Mozilla, Mazur, Supabase.
