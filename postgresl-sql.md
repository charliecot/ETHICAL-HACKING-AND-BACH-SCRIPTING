# SQL AND POSTGRESQL — COMPLETE COMMAND REFERENCE

A practical learning guide with syntax, explanations, examples, and comments.

---

# TABLE OF CONTENTS

1. SQL, PostgreSQL, and psql
2. Create a sample database
3. SELECT
4. FROM
5. WHERE and comparison operators
6. AND, OR, and NOT
7. LIKE and pattern matching
8. ORDER BY
9. LIMIT and OFFSET
10. DISTINCT
11. Aggregate functions
12. GROUP BY
13. HAVING
14. JOIN
15. Subqueries
16. CASE
17. COALESCE and NULLIF
18. INSERT
19. UPDATE
20. DELETE
21. CREATE TABLE and constraints
22. ALTER TABLE
23. DROP, TRUNCATE, and DELETE compared
24. Indexes
25. Common Table Expressions (CTEs)
26. UNION, INTERSECT, and EXCEPT
27. Window functions
28. Transactions
29. EXPLAIN and query performance
30. PostgreSQL data types
31. Useful SQL functions
32. PostgreSQL psql commands
33. Database and user administration
34. PostgreSQL with Python
35. Common mistakes
36. Recommended learning order
37. Official documentation

---

# 1. SQL, POSTGRESQL, AND PSQL

## What is SQL?

SQL stands for Structured Query Language. It is used to communicate with relational databases.

SQL lets you:

- Create databases and tables.
- Insert, read, update, and delete records.
- Filter and sort information.
- Combine information from multiple tables.
- Control access to database objects.
- Manage transactions.

## What is PostgreSQL?

PostgreSQL is a relational database management system that understands SQL and provides additional features.

## What is psql?

`psql` is a terminal-based application for connecting to PostgreSQL and running SQL commands.

Example:

```bash
psql -U postgres -d mydatabase
```

Meaning:

- `psql`: start the PostgreSQL terminal client.
- `-U postgres`: connect as the `postgres` user.
- `-d mydatabase`: connect to the `mydatabase` database.

Remember: SQL commands and psql commands are different.

```sql
-- SQL command: runs inside PostgreSQL
SELECT * FROM employees;
```

```text
-- psql meta-command: runs inside the psql terminal
\dt
```

---

# 2. CREATE A SAMPLE DATABASE

We'll use a small company database throughout this guide.

## Create a database

Run this command from a suitable PostgreSQL connection:

```sql
CREATE DATABASE company_db;
```

Connect to it from `psql`:

```text
\c company_db
```

## Create a departments table

```sql
CREATE TABLE departments (
    department_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL UNIQUE
);
```

Explanation:

- `CREATE TABLE`: creates a table.
- `department_id`: unique identifier for each department.
- `GENERATED ALWAYS AS IDENTITY`: PostgreSQL generates the ID automatically.
- `PRIMARY KEY`: identifies each row uniquely.
- `VARCHAR(100)`: text with a maximum length of 100 characters.
- `NOT NULL`: a value must be supplied.
- `UNIQUE`: duplicate department names are not allowed.

## Create an employees table

```sql
CREATE TABLE employees (
    employee_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(150) UNIQUE,
    salary NUMERIC(12, 2) NOT NULL CHECK (salary >= 0),
    department_id INTEGER REFERENCES departments(department_id),
    hire_date DATE NOT NULL DEFAULT CURRENT_DATE,
    is_active BOOLEAN NOT NULL DEFAULT TRUE
);
```

Important constraints:

- `PRIMARY KEY`: uniquely identifies a record.
- `FOREIGN KEY`: links a record to another table.
- `REFERENCES departments(department_id)`: department IDs must exist in the departments table when provided.
- `CHECK (salary >= 0)`: prevents negative salaries.
- `DEFAULT CURRENT_DATE`: supplies today's date when no hire date is specified.

## Insert sample departments

```sql
INSERT INTO departments (department_name)
VALUES
    ('Engineering'),
    ('Marketing'),
    ('Finance'),
    ('Human Resources');
```

## Insert sample employees

```sql
INSERT INTO employees (
    first_name,
    last_name,
    email,
    salary,
    department_id,
    hire_date,
    is_active
)
VALUES
    ('Alice', 'Brown', 'alice@example.com', 650000.00, 1, '2024-01-15', TRUE),
    ('Bob', 'Smith', 'bob@example.com', 450000.00, 2, '2023-06-10', TRUE),
    ('Carol', 'Jones', 'carol@example.com', 700000.00, 1, '2022-03-20', TRUE),
    ('David', 'White', 'david@example.com', 350000.00, 3, '2025-02-01', TRUE),
    ('Eva', 'Green', 'eva@example.com', 500000.00, 2, '2024-09-12', FALSE),
    ('Frank', 'Black', 'frank@example.com', 800000.00, 3, '2021-07-05', TRUE);
```

Note: These salary values are sample numbers for learning, not salary recommendations.

---

# 3. SELECT — RETRIEVE DATA

`SELECT` chooses which columns or expressions to return.

## Basic syntax

```sql
SELECT column1, column2
FROM table_name;
```

## Select one column

```sql
SELECT first_name
FROM employees;
```

## Select multiple columns

```sql
SELECT first_name, last_name, salary
FROM employees;
```

## Select every column

```sql
SELECT *
FROM employees;
```

`*` means all columns. It is useful for learning and exploration, but explicitly listing columns is often better in application code.

## Give a column an alias

```sql
SELECT
    first_name AS name,
    salary AS monthly_salary
FROM employees;
```

`AS` gives a column or expression a temporary output name. It does not rename the actual database column.

## Calculate values

```sql
SELECT
    first_name,
    salary,
    salary * 12 AS annual_salary
FROM employees;
```

## Use an expression

```sql
SELECT
    first_name || ' ' || last_name AS full_name
FROM employees;
```

In PostgreSQL, `||` concatenates strings.

---

# 4. FROM — CHOOSE THE DATA SOURCE

`FROM` identifies the table or other data source to read.

```sql
SELECT *
FROM employees;
```

Read from a table with an alias:

```sql
SELECT e.first_name, e.salary
FROM employees AS e;
```

Here, `e` is a table alias.

Aliases are especially useful when joining tables.

---

# 5. WHERE — FILTER ROWS

`WHERE` returns only rows that satisfy a condition.

## Greater than

```sql
SELECT *
FROM employees
WHERE salary > 500000;
```

## Less than

```sql
SELECT *
FROM employees
WHERE salary < 500000;
```

## Equal to

```sql
SELECT *
FROM employees
WHERE department_id = 1;
```

## Not equal to

```sql
SELECT *
FROM employees
WHERE department_id <> 1;
```

PostgreSQL also supports `!=`, but `<>` is the standard SQL operator.

## Comparison operators

| Operator | Meaning |
|---|---|
| `=` | Equal to |
| `<>` | Not equal to |
| `!=` | Not equal to; supported by PostgreSQL |
| `>` | Greater than |
| `<` | Less than |
| `>=` | Greater than or equal to |
| `<=` | Less than or equal to |

## BETWEEN

Checks whether a value falls within a range, including both endpoints.

```sql
SELECT *
FROM employees
WHERE salary BETWEEN 400000 AND 700000;
```

Equivalent to:

```sql
SELECT *
FROM employees
WHERE salary >= 400000
  AND salary <= 700000;
```

## IN

Matches any value in a list.

```sql
SELECT *
FROM employees
WHERE department_id IN (1, 3);
```

## NOT IN

Excludes values in a list.

```sql
SELECT *
FROM employees
WHERE department_id NOT IN (1, 3);
```

Be careful when a `NOT IN` list or subquery contains `NULL`. It can cause unexpected results. `NOT EXISTS` is often a safer alternative for subqueries.

## IS NULL

Checks whether a value is missing.

```sql
SELECT *
FROM employees
WHERE department_id IS NULL;
```

## IS NOT NULL

```sql
SELECT *
FROM employees
WHERE department_id IS NOT NULL;
```

Important: Do not use `= NULL` to test for missing values. Use `IS NULL`.

---

# 6. AND, OR, AND NOT

## AND

Every condition must be true.

```sql
SELECT *
FROM employees
WHERE salary > 400000
  AND is_active = TRUE;
```

## OR

At least one condition must be true.

```sql
SELECT *
FROM employees
WHERE department_id = 1
   OR department_id = 3;
```

## NOT

Negates a condition.

```sql
SELECT *
FROM employees
WHERE NOT is_active;
```

This selects employees whose `is_active` value is false.

## Combine conditions with parentheses

```sql
SELECT *
FROM employees
WHERE is_active = TRUE
  AND (department_id = 1 OR department_id = 3);
```

Parentheses make the intended logic explicit.

SQL generally evaluates `NOT` before `AND`, and `AND` before `OR`. Use parentheses when the logic could be unclear.

---

# 7. LIKE AND PATTERN MATCHING

`LIKE` matches text patterns.

## Wildcard characters

| Pattern | Meaning |
|---|---|
| `%` | Zero or more characters |
| `_` | Exactly one character |

## Names starting with A

```sql
SELECT *
FROM employees
WHERE first_name LIKE 'A%';
```

## Names ending with n

```sql
SELECT *
FROM employees
WHERE last_name LIKE '%n';
```

## Names containing ar

```sql
SELECT *
FROM employees
WHERE first_name LIKE '%ar%';
```

## A single-character wildcard

```sql
SELECT *
FROM employees
WHERE first_name LIKE '_ob';
```

This matches a three-character name ending in `ob`, such as `Bob`.

## Case-insensitive matching in PostgreSQL

```sql
SELECT *
FROM employees
WHERE first_name ILIKE 'a%';
```

`ILIKE` is PostgreSQL-specific and matches without distinguishing uppercase from lowercase.

## PostgreSQL regular-expression matching

```sql
-- Case-sensitive regular expression
SELECT *
FROM employees
WHERE first_name ~ '^A';
```

```sql
-- Case-insensitive regular expression
SELECT *
FROM employees
WHERE first_name ~* '^a';
```

Useful PostgreSQL operators:

- `~`: case-sensitive regular-expression match.
- `~*`: case-insensitive regular-expression match.
- `!~`: does not match a case-sensitive regular expression.
- `!~*`: does not match a case-insensitive regular expression.

---

# 8. ORDER BY — SORT RESULTS

## Ascending order

```sql
SELECT *
FROM employees
ORDER BY salary ASC;
```

`ASC` means ascending. It is the default direction.

## Descending order

```sql
SELECT *
FROM employees
ORDER BY salary DESC;
```

## Sort by multiple columns

```sql
SELECT *
FROM employees
ORDER BY department_id ASC, salary DESC;
```

Rows are first sorted by department. Within each department, employees are sorted by salary from highest to lowest.

## Control NULL placement

```sql
SELECT *
FROM employees
ORDER BY department_id ASC NULLS LAST;
```

PostgreSQL supports `NULLS FIRST` and `NULLS LAST`.

---

# 9. LIMIT AND OFFSET

`LIMIT` restricts how many rows are returned.

```sql
SELECT *
FROM employees
LIMIT 3;
```

## OFFSET

Skips a number of rows.

```sql
SELECT *
FROM employees
ORDER BY employee_id
LIMIT 3 OFFSET 2;
```

This skips the first two rows and returns the next three.

## Pagination example

Page 1:

```sql
SELECT *
FROM employees
ORDER BY employee_id
LIMIT 5 OFFSET 0;
```

Page 2:

```sql
SELECT *
FROM employees
ORDER BY employee_id
LIMIT 5 OFFSET 5;
```

General formula:

```text
OFFSET = (page_number - 1) * page_size
```

Always use a predictable `ORDER BY` when paginating. For very large tables, keyset pagination can be more efficient than a large offset.

---

# 10. DISTINCT — REMOVE DUPLICATES

```sql
SELECT DISTINCT department_id
FROM employees;
```

Returns each distinct department ID once.

## Distinct combinations

```sql
SELECT DISTINCT department_id, is_active
FROM employees;
```

This returns unique combinations of department and active status.

## PostgreSQL DISTINCT ON

`DISTINCT ON` keeps one row for each value of the specified expression.

```sql
SELECT DISTINCT ON (department_id)
    department_id,
    first_name,
    salary
FROM employees
ORDER BY department_id, salary DESC;
```

This returns one employee per department: the highest-paid employee in each department.

The `ORDER BY` determines which row is retained. PostgreSQL requires the `DISTINCT ON` expressions to match the leftmost `ORDER BY` expressions.

---

# 11. AGGREGATE FUNCTIONS

Aggregate functions calculate a result from multiple rows.

| Function | Purpose |
|---|---|
| `COUNT(*)` | Counts rows |
| `COUNT(column)` | Counts non-NULL values in a column |
| `SUM(column)` | Calculates the sum |
| `AVG(column)` | Calculates the average |
| `MIN(column)` | Finds the minimum |
| `MAX(column)` | Finds the maximum |

## COUNT

```sql
SELECT COUNT(*) AS employee_count
FROM employees;
```

## SUM

```sql
SELECT SUM(salary) AS total_salary
FROM employees;
```

## AVG

```sql
SELECT AVG(salary) AS average_salary
FROM employees;
```

## MIN and MAX

```sql
SELECT
    MIN(salary) AS lowest_salary,
    MAX(salary) AS highest_salary
FROM employees;
```

## Count non-NULL values

```sql
SELECT COUNT(department_id)
FROM employees;
```

`COUNT(department_id)` ignores rows where `department_id` is NULL. `COUNT(*)` counts all rows.

## Count distinct values

```sql
SELECT COUNT(DISTINCT department_id)
FROM employees;
```

---

# 12. GROUP BY — GROUP ROWS

`GROUP BY` combines rows with the same grouping values so aggregate functions can calculate results per group.

## Count employees in each department

```sql
SELECT
    department_id,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department_id;
```

## Average salary per department

```sql
SELECT
    department_id,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department_id;
```

## Multiple grouping columns

```sql
SELECT
    department_id,
    is_active,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department_id, is_active;
```

Important: When you use `GROUP BY`, ordinary selected columns generally need to be grouped or used inside aggregate expressions.

---

# 13. HAVING — FILTER GROUPS

`HAVING` filters groups after grouping and aggregation.

```sql
SELECT
    department_id,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 1;
```

## WHERE versus HAVING

- `WHERE` filters individual rows before grouping.
- `HAVING` filters groups after grouping.

Example:

```sql
SELECT
    department_id,
    AVG(salary) AS average_salary
FROM employees
WHERE is_active = TRUE
GROUP BY department_id
HAVING AVG(salary) > 400000;
```

This first selects active employees, then calculates each department's average salary, then keeps departments whose average exceeds the threshold.

---

# 14. JOIN — COMBINE TABLES

A JOIN combines rows from multiple tables using a relationship or condition.

## INNER JOIN

Returns only rows with matching records in both tables.

```sql
SELECT
    e.first_name,
    e.salary,
    d.department_name
FROM employees AS e
INNER JOIN departments AS d
    ON e.department_id = d.department_id;
```

## LEFT JOIN

Returns all rows from the left table, plus matching rows from the right table. If no match exists, right-table columns are NULL.

```sql
SELECT
    e.first_name,
    d.department_name
FROM employees AS e
LEFT JOIN departments AS d
    ON e.department_id = d.department_id;
```

## Find employees without a matching department

```sql
SELECT e.*
FROM employees AS e
LEFT JOIN departments AS d
    ON e.department_id = d.department_id
WHERE d.department_id IS NULL;
```

This includes employees with no department ID and any rows without a matching department.

## RIGHT JOIN

Returns all rows from the right table and matching rows from the left table.

```sql
SELECT
    e.first_name,
    d.department_name
FROM employees AS e
RIGHT JOIN departments AS d
    ON e.department_id = d.department_id;
```

## FULL OUTER JOIN

Returns matching rows and unmatched rows from both tables.

```sql
SELECT
    e.first_name,
    d.department_name
FROM employees AS e
FULL OUTER JOIN departments AS d
    ON e.department_id = d.department_id;
```

## CROSS JOIN

Returns every possible combination of rows from both tables.

```sql
SELECT
    e.first_name,
    d.department_name
FROM employees AS e
CROSS JOIN departments AS d;
```

If one table has 6 rows and the other has 4, the result has 24 rows.

## JOIN using USING

When both tables share the same join-column name:

```sql
SELECT
    e.first_name,
    d.department_name
FROM employees AS e
JOIN departments AS d
USING (department_id);
```

This works only when both tables have a column named `department_id`.

## ON versus USING

- `ON` lets you write an explicit join condition.
- `USING` is shorthand for joining columns with the same name.

Use `ON` when the columns have different names or when the condition is more complex.

---

# 15. SUBQUERIES

A subquery is a query nested inside another SQL statement.

## Find employees earning above the overall average

```sql
SELECT first_name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

The inner query calculates the average salary. The outer query selects employees earning more than that value.

## IN with a subquery

```sql
SELECT first_name
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE department_name IN ('Engineering', 'Finance')
);
```

## EXISTS

`EXISTS` checks whether a subquery returns at least one row.

```sql
SELECT d.department_name
FROM departments AS d
WHERE EXISTS (
    SELECT 1
    FROM employees AS e
    WHERE e.department_id = d.department_id
);
```

This returns departments with at least one employee.

## NOT EXISTS

```sql
SELECT d.department_name
FROM departments AS d
WHERE NOT EXISTS (
    SELECT 1
    FROM employees AS e
    WHERE e.department_id = d.department_id
);
```

This returns departments without employees.

---

# 16. CASE — CONDITIONAL EXPRESSIONS

`CASE` allows conditional logic inside a query.

```sql
SELECT
    first_name,
    salary,
    CASE
        WHEN salary >= 700000 THEN 'High'
        WHEN salary >= 450000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_category
FROM employees;
```

SQL checks conditions from top to bottom and returns the result for the first matching condition.

## CASE with status values

```sql
SELECT
    first_name,
    CASE
        WHEN is_active = TRUE THEN 'Active'
        ELSE 'Inactive'
    END AS account_status
FROM employees;
```

## CASE inside ORDER BY

```sql
SELECT *
FROM employees
ORDER BY
    CASE
        WHEN is_active = TRUE THEN 0
        ELSE 1
    END,
    first_name;
```

This places active employees before inactive employees, then sorts names alphabetically.

---

# 17. COALESCE AND NULLIF

## COALESCE

Returns the first non-NULL expression.

```sql
SELECT
    first_name,
    COALESCE(department_id, 0) AS department_id
FROM employees;
```

If `department_id` is NULL, the query displays `0`.

Another example:

```sql
SELECT COALESCE(NULL, NULL, 'Default value');
```

Result:

```text
Default value
```

## NULLIF

Returns NULL if the two expressions are equal; otherwise, it returns the first expression.

```sql
SELECT NULLIF(10, 10);
```

Result: `NULL`.

```sql
SELECT NULLIF(10, 5);
```

Result: `10`.

## Avoid division by zero

```sql
SELECT
    total_sales / NULLIF(order_count, 0) AS average_order_value
FROM sales_summary;
```

If `order_count` is zero, the denominator becomes NULL instead of zero, avoiding a division-by-zero error. The result is NULL in that case.

---

# 18. INSERT — ADD DATA

## Insert one row

```sql
INSERT INTO employees (
    first_name,
    last_name,
    email,
    salary,
    department_id
)
VALUES (
    'Grace',
    'King',
    'grace@example.com',
    600000,
    1
);
```

The identity ID, default hire date, and default active status are generated automatically.

## Insert multiple rows

```sql
INSERT INTO departments (department_name)
VALUES
    ('Sales'),
    ('Customer Support');
```

## RETURNING

PostgreSQL supports `RETURNING` to show inserted values.

```sql
INSERT INTO departments (department_name)
VALUES ('Research')
RETURNING department_id, department_name;
```

## Insert from another query

```sql
INSERT INTO archived_employees (
    employee_id,
    first_name,
    last_name
)
SELECT
    employee_id,
    first_name,
    last_name
FROM employees
WHERE is_active = FALSE;
```

The destination table must exist, and the selected values must match its column types.

---

# 19. UPDATE — MODIFY DATA

## Update one employee

```sql
UPDATE employees
SET salary = 750000
WHERE employee_id = 1;
```

## Update multiple columns

```sql
UPDATE employees
SET
    salary = 720000,
    is_active = TRUE
WHERE employee_id = 2;
```

## Update based on a condition

```sql
UPDATE employees
SET salary = salary * 1.10
WHERE department_id = 1;
```

This increases salaries in department 1 by 10%.

## RETURNING updated values

```sql
UPDATE employees
SET salary = salary + 25000
WHERE employee_id = 1
RETURNING employee_id, first_name, salary;
```

**Important:** Always check your `WHERE` clause before running an update. Without it, every row can be updated.

```sql
-- Dangerous: updates every employee
UPDATE employees
SET is_active = FALSE;
```

---

# 20. DELETE — REMOVE ROWS

## Delete a particular employee

```sql
DELETE FROM employees
WHERE employee_id = 3;
```

## Delete inactive employees

```sql
DELETE FROM employees
WHERE is_active = FALSE;
```

## RETURNING deleted rows

```sql
DELETE FROM employees
WHERE employee_id = 2
RETURNING *;
```

**Important:** A DELETE statement without a `WHERE` clause removes all rows from the table.

```sql
-- Dangerous: deletes every row
DELETE FROM employees;
```

For important changes, use a transaction so you can inspect the result and roll back if necessary.

---

# 21. CREATE TABLE AND CONSTRAINTS

## General syntax

```sql
CREATE TABLE table_name (
    column_name data_type constraint,
    another_column data_type
);
```

## PRIMARY KEY

A primary key uniquely identifies each row.

```sql
CREATE TABLE customers (
    customer_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL
);
```

## UNIQUE

Prevents duplicate values.

```sql
CREATE TABLE accounts (
    account_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email VARCHAR(150) UNIQUE
);
```

PostgreSQL normally permits multiple NULL values in a UNIQUE column unless you specify a different NULL-handling option.

## NOT NULL

Requires a value.

```sql
CREATE TABLE courses (
    course_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    course_name VARCHAR(100) NOT NULL
);
```

## CHECK

Validates a condition.

```sql
CREATE TABLE products (
    product_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price NUMERIC(10, 2) NOT NULL CHECK (price >= 0)
);
```

## DEFAULT

Supplies a value if one is omitted.

```sql
CREATE TABLE tasks (
    task_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    task_name VARCHAR(100) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

## FOREIGN KEY

Creates a relationship between tables.

```sql
CREATE TABLE orders (
    order_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
    order_date DATE NOT NULL DEFAULT CURRENT_DATE
);
```

The referenced key must exist. A foreign key helps maintain referential integrity.

## ON DELETE actions

Foreign keys can specify what happens when a referenced row is deleted.

```sql
CREATE TABLE order_items (
    order_item_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    order_id INTEGER NOT NULL
        REFERENCES orders(order_id)
        ON DELETE CASCADE,
    product_name VARCHAR(100) NOT NULL
);
```

Common actions:

- `CASCADE`: automatically deletes dependent rows.
- `SET NULL`: sets the foreign-key value to NULL; the column must allow NULL.
- `SET DEFAULT`: sets the column to its default value, which must satisfy the foreign key.
- `RESTRICT`: prevents deletion when dependent rows exist.
- `NO ACTION`: checks the constraint at its normal checking time; it is the default.

Use cascading deletes carefully because one deletion may remove many dependent records.

---

# 22. ALTER TABLE — CHANGE TABLE STRUCTURE

## Add a column

```sql
ALTER TABLE employees
ADD COLUMN phone VARCHAR(20);
```

## Rename a column

```sql
ALTER TABLE employees
RENAME COLUMN phone TO phone_number;
```

## Change a column's data type

```sql
ALTER TABLE employees
ALTER COLUMN salary TYPE NUMERIC(14, 2);
```

PostgreSQL may require a conversion if the old and new types differ significantly.

## Set a default

```sql
ALTER TABLE employees
ALTER COLUMN is_active SET DEFAULT TRUE;
```

## Remove a default

```sql
ALTER TABLE employees
ALTER COLUMN is_active DROP DEFAULT;
```

## Add a constraint

```sql
ALTER TABLE employees
ADD CONSTRAINT salary_positive CHECK (salary >= 0);
```

## Drop a column

```sql
ALTER TABLE employees
DROP COLUMN phone_number;
```

Dropping a column removes that column's data. Review dependencies before doing this.

## Rename a table

```sql
ALTER TABLE employees
RENAME TO staff;
```

If you run this example, later examples that reference `employees` must use the new table name or you should rename it back.

---

# 23. DROP, TRUNCATE, AND DELETE

These commands are different.

## DELETE

Removes rows and can target specific records.

```sql
DELETE FROM employees
WHERE employee_id = 5;
```

## TRUNCATE

Removes all rows efficiently.

```sql
TRUNCATE TABLE employees;
```

To reset an identity sequence too:

```sql
TRUNCATE TABLE employees RESTART IDENTITY;
```

To truncate a table and dependent tables that reference it, PostgreSQL may require `CASCADE`:

```sql
TRUNCATE TABLE employees RESTART IDENTITY CASCADE;
```

Use `CASCADE` with care: it can truncate additional tables.

## DROP TABLE

Removes the table itself, including its data and table definition.

```sql
DROP TABLE employees;
```

Safer form:

```sql
DROP TABLE IF EXISTS employees;
```

## Comparison

| Command | Removes rows? | Keeps table structure? | Supports WHERE? |
|---|---|---|---|
| `DELETE` | Yes | Yes | Yes |
| `TRUNCATE` | All rows | Yes | No |
| `DROP TABLE` | Yes, by removing the table | No | No |

These commands can have significant consequences. Use backups and transactions when appropriate.

---

# 24. INDEXES

An index can help PostgreSQL find rows faster, especially for frequently filtered, joined, or sorted columns. Indexes also require storage and can slow down writes.

## Create an index

```sql
CREATE INDEX idx_employees_salary
ON employees(salary);
```

## Index for a frequently filtered column

```sql
CREATE INDEX idx_employees_department
ON employees(department_id);
```

## Unique index

```sql
CREATE UNIQUE INDEX idx_employees_email
ON employees(email);
```

A unique index prevents duplicate non-NULL values in this example.

## Multi-column index

```sql
CREATE INDEX idx_employees_department_salary
ON employees(department_id, salary DESC);
```

The column order matters. This index is particularly useful for queries that filter by `department_id` and may then sort or filter by salary.

## Partial index

```sql
CREATE INDEX idx_active_employees
ON employees(employee_id)
WHERE is_active = TRUE;
```

A partial index stores entries only for rows matching its condition.

## Drop an index

```sql
DROP INDEX IF EXISTS idx_employees_salary;
```

Do not create indexes on every column automatically. Use query plans and workload patterns to decide which indexes are useful.

---

# 25. COMMON TABLE EXPRESSIONS (CTEs)

A CTE uses `WITH` to give a query a temporary named result.

## Basic CTE

```sql
WITH high_paid_employees AS (
    SELECT *
    FROM employees
    WHERE salary > 600000
)
SELECT first_name, salary
FROM high_paid_employees;
```

The CTE exists for this statement, not as a permanent table.

## Multiple CTEs

```sql
WITH active_employees AS (
    SELECT *
    FROM employees
    WHERE is_active = TRUE
),
high_paid AS (
    SELECT *
    FROM active_employees
    WHERE salary > 600000
)
SELECT first_name, salary
FROM high_paid;
```

## Recursive CTE

Recursive CTEs are useful for hierarchical data, such as an organizational chart or category tree.

```sql
WITH RECURSIVE numbers(n) AS (
    SELECT 1

    UNION ALL

    SELECT n + 1
    FROM numbers
    WHERE n < 5
)
SELECT n
FROM numbers;
```

Result:

```text
1
2
3
4
5
```

The initial query creates the starting row. The recursive query repeatedly generates another row until the stopping condition is reached.

---

# 26. UNION, UNION ALL, INTERSECT, AND EXCEPT

These operators combine the results of compatible SELECT queries.

## UNION

Combines results and removes duplicate rows.

```sql
SELECT first_name AS name
FROM employees
WHERE department_id = 1

UNION

SELECT first_name AS name
FROM employees
WHERE department_id = 2;
```

## UNION ALL

Combines results while keeping duplicates.

```sql
SELECT first_name AS name
FROM employees
WHERE department_id = 1

UNION ALL

SELECT first_name AS name
FROM employees
WHERE department_id = 2;
```

`UNION ALL` is generally faster when duplicate removal is unnecessary.

## INTERSECT

Returns rows present in both query results.

```sql
SELECT employee_id
FROM employees
WHERE salary > 400000

INTERSECT

SELECT employee_id
FROM employees
WHERE is_active = TRUE;
```

## EXCEPT

Returns rows in the first result that are absent from the second.

```sql
SELECT employee_id
FROM employees

EXCEPT

SELECT employee_id
FROM employees
WHERE is_active = TRUE;
```

The queries combined by these operators must return the same number of columns with compatible data types.

---

# 27. WINDOW FUNCTIONS

Window functions calculate values across related rows without combining those rows into one result per group.

## ROW_NUMBER

Assigns a sequential number within each partition.

```sql
SELECT
    first_name,
    department_id,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department_id
        ORDER BY salary DESC
    ) AS row_number
FROM employees;
```

Each department gets its own numbering sequence.

## RANK

Assigns ranks, leaving gaps after ties.

```sql
SELECT
    first_name,
    salary,
    RANK() OVER (
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

## DENSE_RANK

Ranks rows without gaps after ties.

```sql
SELECT
    first_name,
    salary,
    DENSE_RANK() OVER (
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

## AVG OVER

Calculates an average while preserving individual rows.

```sql
SELECT
    first_name,
    department_id,
    salary,
    AVG(salary) OVER (
        PARTITION BY department_id
    ) AS department_average
FROM employees;
```

## Running total

```sql
SELECT
    employee_id,
    first_name,
    salary,
    SUM(salary) OVER (
        ORDER BY employee_id
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM employees;
```

## GROUP BY versus window functions

- `GROUP BY` usually returns one row per group.
- Window functions preserve individual rows while calculating group-related values.

---

# 28. TRANSACTIONS

A transaction groups database operations so they can be committed together or rolled back.

## BEGIN

Starts a transaction.

```sql
BEGIN;
```

## COMMIT

Saves the transaction's changes.

```sql
COMMIT;
```

## ROLLBACK

Cancels uncommitted changes.

```sql
ROLLBACK;
```

## Example

```sql
BEGIN;

UPDATE employees
SET salary = salary + 10000
WHERE employee_id = 1;

-- Inspect the change before committing.
SELECT employee_id, first_name, salary
FROM employees
WHERE employee_id = 1;

-- Save the changes when satisfied.
COMMIT;
```

If you decide not to keep the changes, execute `ROLLBACK` instead of `COMMIT`, while the transaction is still open.

## Savepoints

A savepoint allows you to roll back part of a transaction.

```sql
BEGIN;

UPDATE employees
SET salary = salary + 10000
WHERE employee_id = 1;

SAVEPOINT after_first_update;

UPDATE employees
SET salary = salary + 10000
WHERE employee_id = 2;

-- Undo changes made after the savepoint.
ROLLBACK TO SAVEPOINT after_first_update;

-- Keep the first update.
COMMIT;
```

The second update is undone; the first update remains and is committed.

---

# 29. EXPLAIN AND QUERY PERFORMANCE

`EXPLAIN` shows PostgreSQL's planned execution strategy.

```sql
EXPLAIN
SELECT *
FROM employees
WHERE department_id = 1;
```

## EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE
SELECT *
FROM employees
WHERE department_id = 1;
```

`EXPLAIN ANALYZE` executes the query and reports actual timing and row counts alongside the plan.

Be careful with statements that modify data. `EXPLAIN ANALYZE` executes them too.

For example, if you must analyze a data-modifying statement, you can use a transaction and roll it back:

```sql
BEGIN;

EXPLAIN ANALYZE
UPDATE employees
SET salary = salary + 1
WHERE employee_id = 1;

ROLLBACK;
```

This example executes the update inside the transaction but rolls back its changes.

Common plan terms:

- `Seq Scan`: sequential table scan.
- `Index Scan`: uses an index to find rows.
- `Bitmap Index Scan`: identifies matching index entries for a bitmap scan.
- `Nested Loop`: repeatedly matches rows from two inputs.
- `Hash Join`: joins using a hash table.
- `Sort`: sorts rows.
- `Aggregate`: calculates aggregate results.

An index scan is not always better than a sequential scan. PostgreSQL chooses a plan based on estimated cost and available statistics.

---

# 30. POSTGRESQL DATA TYPES

## Numeric types

| Type | Purpose |
|---|---|
| `SMALLINT` | Small whole numbers |
| `INTEGER` or `INT` | Standard whole numbers |
| `BIGINT` | Large whole numbers |
| `NUMERIC(p,s)` | Exact decimal values |
| `REAL` | Approximate floating-point values |
| `DOUBLE PRECISION` | Higher-precision approximate floating-point values |
| `SERIAL` | Older shorthand for integer IDs backed by a sequence |
| `BIGSERIAL` | Older shorthand for large integer IDs backed by a sequence |

For money-like calculations where exact decimal arithmetic matters, `NUMERIC` is often a suitable choice.

## Text types

| Type | Purpose |
|---|---|
| `CHAR(n)` | Fixed-length character type |
| `VARCHAR(n)` | Variable-length text with a limit |
| `TEXT` | Variable-length text without a declared length limit |

## Date and time types

| Type | Purpose |
|---|---|
| `DATE` | Calendar date |
| `TIME` | Time of day |
| `TIMESTAMP` | Date and time without time zone |
| `TIMESTAMPTZ` | Timestamp with time-zone-aware handling |
| `INTERVAL` | Duration or time interval |

PostgreSQL stores `TIMESTAMPTZ` values internally as instants and displays them according to the session time zone. It does not preserve the original named time zone supplied as input.

## Other important types

| Type | Purpose |
|---|---|
| `BOOLEAN` | True, false, or NULL |
| `UUID` | Universally unique identifier |
| `JSON` | JSON data stored as text-like JSON representation |
| `JSONB` | Binary decomposed JSON, often preferable for querying |
| `BYTEA` | Binary data |
| `ARRAY` | Array of values |
| `INET` | IP network or address |
| `CIDR` | IP network |
| `MACADDR` | MAC address |

Example:

```sql
CREATE TABLE api_events (
    event_id UUID PRIMARY KEY,
    event_data JSONB NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

---

# 31. USEFUL SQL AND POSTGRESQL FUNCTIONS

## String functions

```sql
SELECT
    UPPER(first_name) AS uppercase_name,
    LOWER(last_name) AS lowercase_name,
    LENGTH(first_name) AS name_length,
    TRIM(first_name) AS trimmed_name
FROM employees;
```

Other useful functions:

```sql
SELECT CONCAT('Hello', ' ', 'World');
SELECT SUBSTRING('PostgreSQL' FROM 1 FOR 4);
SELECT REPLACE('Hello World', 'World', 'SQL');
```

## Numeric functions

```sql
SELECT ROUND(123.456, 2);
SELECT ABS(-10);
SELECT CEIL(4.2);
SELECT FLOOR(4.8);
SELECT POWER(2, 3);
```

## Date and time functions

```sql
SELECT CURRENT_DATE;
SELECT CURRENT_TIME;
SELECT CURRENT_TIMESTAMP;
SELECT NOW();
```

Add an interval:

```sql
SELECT CURRENT_DATE + INTERVAL '7 days';
```

Calculate a date difference:

```sql
SELECT DATE '2026-10-21' - DATE '2026-10-01';
```

This returns the number of days between the two dates.

Extract a component:

```sql
SELECT EXTRACT(YEAR FROM CURRENT_DATE);
SELECT EXTRACT(MONTH FROM CURRENT_DATE);
SELECT EXTRACT(DAY FROM CURRENT_DATE);
```

## Type conversion

```sql
SELECT CAST('123' AS INTEGER);
```

PostgreSQL shorthand:

```sql
SELECT '123'::INTEGER;
```

Both convert the text value to an integer.

## Boolean and NULL checks

```sql
SELECT TRUE AND FALSE;
SELECT TRUE OR FALSE;
SELECT NOT TRUE;
SELECT NULL IS NULL;
```

SQL uses three-valued logic: true, false, and unknown. Comparisons involving NULL generally produce unknown rather than true or false.

---

# 32. PSQL META-COMMANDS

These commands run in the `psql` terminal, not as ordinary SQL statements.

| Command | Purpose |
|---|---|
| `\?` | Show psql meta-command help |
| `\h` | Show SQL command help |
| `\h SELECT` | Help for SELECT syntax |
| `\l` | List databases |
| `\c company_db` | Connect to a database |
| `\conninfo` | Display current connection information |
| `\dt` | List tables in the search path |
| `\dt *.*` | List tables across visible schemas |
| `\d employees` | Describe a table |
| `\d+ employees` | Describe a table with additional details |
| `\di` | List indexes |
| `\dv` | List views |
| `\df` | List functions |
| `\dn` | List schemas |
| `\du` | List roles |
| `\dx` | List installed extensions |
| `\x` | Toggle expanded output |
| `\timing` | Toggle query execution timing |
| `\pset pager off` | Disable the output pager |
| `\e` | Edit the current query buffer in an editor |
| `\s` | Show command history |
| `\! clear` | Run the terminal's clear command on compatible systems |
| `\q` | Exit psql |

## Run a SQL file

From the operating system terminal:

```bash
psql -U postgres -d company_db -f script.sql
```

Inside `psql`:

```text
\i script.sql
```

## Save query output

```text
\o output.txt
```

Subsequent query output is sent to the file.

Return output to the terminal:

```text
\o
```

## List tables in a schema

```text
\dt public.*
```

The `public` schema is common, but tables can live in other schemas.

---

# 33. DATABASE AND USER ADMINISTRATION

Run administrative commands using an account with sufficient privileges.

## Create a database

```sql
CREATE DATABASE shop_db;
```

## Create a role that can log in

```sql
CREATE ROLE app_user
WITH LOGIN PASSWORD 'replace_with_a_strong_secret';
```

Use a strong secret, and avoid placing real passwords in source code or public documentation.

## Grant database connection privileges

```sql
GRANT CONNECT ON DATABASE shop_db TO app_user;
```

## Grant schema usage

Connect to `shop_db`, then run:

```sql
GRANT USAGE ON SCHEMA public TO app_user;
```

## Grant table privileges

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON ALL TABLES IN SCHEMA public
TO app_user;
```

## Grant privileges on existing sequences

```sql
GRANT USAGE, SELECT
ON ALL SEQUENCES IN SCHEMA public
TO app_user;
```

Identity columns can depend on sequence privileges in relevant configurations.

## Set privileges for future tables

Run this as the role that will create the tables, or specify the correct object-creating role:

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_user;
```

For future sequences:

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT USAGE, SELECT ON SEQUENCES TO app_user;
```

Default privileges apply to objects subsequently created by the relevant role. They do not automatically grant privileges on existing objects.

## Change a password

```sql
ALTER ROLE app_user
WITH PASSWORD 'replace_with_a_new_strong_secret';
```

## Revoke privileges

```sql
REVOKE DELETE
ON ALL TABLES IN SCHEMA public
FROM app_user;
```

Grant only the privileges an application actually needs. A normal application account usually should not be a PostgreSQL superuser.

---

# 34. CONNECT POSTGRESQL TO PYTHON

Install Psycopg, the PostgreSQL adapter for Python:

```bash
python -m pip install "psycopg[binary]"
```

Example connection and query:

```python
import os
import psycopg

# Store connection credentials in environment variables.
# Configure these in your terminal or development environment.
database_url = os.environ["DATABASE_URL"]

with psycopg.connect(database_url) as conn:
    with conn.cursor() as cur:
        # Use placeholders for values supplied by users.
        cur.execute(
            """
            SELECT employee_id, first_name, salary
            FROM employees
            WHERE department_id = %s
            ORDER BY salary DESC
            """,
            (1,),
        )

        rows = cur.fetchall()

        for row in rows:
            print(row)
```

Example environment variable format:

```text
DATABASE_URL=postgresql://app_user:your_secret@localhost:5432/company_db
```

Configure the environment variable securely rather than hardcoding real credentials.

## Insert safely

```python
import os
import psycopg

database_url = os.environ["DATABASE_URL"]

with psycopg.connect(database_url) as conn:
    with conn.cursor() as cur:
        cur.execute(
            """
            INSERT INTO employees (
                first_name,
                last_name,
                email,
                salary,
                department_id
            )
            VALUES (%s, %s, %s, %s, %s)
            RETURNING employee_id
            """,
            ("Helen", "Moore", "helen@example.com", 550000, 1),
        )

        new_employee_id = cur.fetchone()[0]
        print("Created employee:", new_employee_id)
```

The connection context manager commits a successful transaction and rolls it back if an exception escapes the context.

**Important security rule:** Use parameterized queries for values. Do not build SQL by concatenating untrusted input.

Incorrect:

```python
# UNSAFE: user input is inserted directly into SQL text.
query = f"SELECT * FROM employees WHERE first_name = '{name}'"
```

Correct:

```python
cur.execute(
    "SELECT * FROM employees WHERE first_name = %s",
    (name,),
)
```

Placeholders represent values, not table or column names. For dynamic identifiers, use Psycopg's SQL composition utilities rather than treating an identifier as an ordinary value.

---

# 35. COMMON SQL MISTAKES

## Mistake 1: Forgetting WHERE

Dangerous:

```sql
UPDATE employees
SET salary = 500000;
```

This updates every row.

Safer:

```sql
UPDATE employees
SET salary = 500000
WHERE employee_id = 1;
```

## Mistake 2: Comparing NULL with equals

Incorrect:

```sql
SELECT *
FROM employees
WHERE department_id = NULL;
```

Correct:

```sql
SELECT *
FROM employees
WHERE department_id IS NULL;
```

## Mistake 3: Using WHERE for aggregate filters

Incorrect:

```sql
SELECT department_id, COUNT(*)
FROM employees
WHERE COUNT(*) > 1
GROUP BY department_id;
```

Correct:

```sql
SELECT department_id, COUNT(*)
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 1;
```

## Mistake 4: Forgetting JOIN conditions

A cross join or a join with an incorrect condition can create far more rows than expected.

```sql
SELECT e.first_name, d.department_name
FROM employees AS e
JOIN departments AS d
    ON e.department_id = d.department_id;
```

## Mistake 5: Using UPDATE or DELETE without checking the target rows

Before a change, preview the affected records:

```sql
SELECT *
FROM employees
WHERE employee_id = 1;
```

Then execute the update or delete using the same condition.

## Mistake 6: Assuming rows have a guaranteed order

Without `ORDER BY`, PostgreSQL does not guarantee result order.

## Mistake 7: Concatenating untrusted user input into SQL

Use parameterized queries to reduce SQL injection risk.

## Mistake 8: Confusing a table alias with a permanent rename

```sql
SELECT e.first_name
FROM employees AS e;
```

The alias `e` exists only within this query.

## Mistake 9: Using SELECT * in every application query

Select only the columns you need when possible. This can improve clarity, reduce data transfer, and make applications less sensitive to schema changes.

---

# 36. RECOMMENDED LEARNING ORDER

Study the concepts in this sequence:

1. `SELECT`, `FROM`, and column aliases.
2. `WHERE` and comparison operators.
3. `AND`, `OR`, `NOT`, `IN`, `BETWEEN`, and `IS NULL`.
4. `ORDER BY`, `LIMIT`, and `OFFSET`.
5. `INSERT`, `UPDATE`, and `DELETE`.
6. `COUNT`, `SUM`, `AVG`, `MIN`, and `MAX`.
7. `GROUP BY` and `HAVING`.
8. `INNER JOIN` and `LEFT JOIN`.
9. Subqueries and `EXISTS`.
10. `CASE`, `COALESCE`, and `NULLIF`.
11. Table creation, keys, and constraints.
12. Transactions and savepoints.
13. CTEs and window functions.
14. Indexes and `EXPLAIN ANALYZE`.
15. Roles, privileges, and application connections.

## Practice projects

### Project 1: Student management system

Create tables for students, courses, and enrollments. Practise joins, filtering, and counting enrollments.

### Project 2: E-commerce database

Create products, customers, orders, and order items. Calculate order totals and customer spending.

### Project 3: Employee management

Use departments and employees. Practise aggregate functions, salary ranking, and department reports.

### Project 4: Inventory management

Track products, stock levels, and stock movements. Practise transactions and constraints.

### Project 5: Blog database

Create users, posts, and comments. Practise foreign keys, joins, and counting comments per post.

---

# 37. OFFICIAL DOCUMENTATION

- PostgreSQL documentation: https://www.postgresql.org/docs/
- PostgreSQL SQL commands: https://www.postgresql.org/docs/current/sql-commands.html
- PostgreSQL data types: https://www.postgresql.org/docs/current/datatype.html
- PostgreSQL functions and operators: https://www.postgresql.org/docs/current/functions.html
- PostgreSQL indexes: https://www.postgresql.org/docs/current/indexes.html
- PostgreSQL transactions: https://www.postgresql.org/docs/current/tutorial-transactions.html
- PostgreSQL psql: https://www.postgresql.org/docs/current/app-psql.html
- Psycopg documentation: https://www.psycopg.org/psycopg3/docs/

---

# FINAL SUMMARY

Remember the basic SQL query structure:

```sql
SELECT columns
FROM table_name
JOIN another_table
    ON join_condition
WHERE row_condition
GROUP BY grouping_columns
HAVING group_condition
ORDER BY sorting_columns
LIMIT number_of_rows
OFFSET rows_to_skip;
```

Not every clause is required in every query. When used together, the logical processing order is generally:

1. `FROM` and `JOIN`
2. `WHERE`
3. `GROUP BY`
4. `HAVING`
5. `SELECT`
6. `DISTINCT`
7. `ORDER BY`
8. `LIMIT` and `OFFSET`

This logical order helps explain why, for example, aggregate filters belong in `HAVING` rather than `WHERE`.
