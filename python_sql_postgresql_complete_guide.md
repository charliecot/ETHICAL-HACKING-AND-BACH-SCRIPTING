# Python + SQL + PostgreSQL: Complete Practical Reference

## Table of Contents

1. SQL, Python, and PostgreSQL
2. Install database drivers
3. SQL command categories
4. Create a database and tables
5. SQL data types
6. CRUD commands
7. Filtering, sorting, and pagination
8. Aggregate functions and grouping
9. Joins
10. Subqueries and common table expressions
11. Constraints and indexes
12. Transactions
13. PostgreSQL-specific features
14. Connect Python to SQLite
15. Connect Python to PostgreSQL with Psycopg
16. SQL parameter styles
17. Tuples, lists, dictionaries, and result rows
18. Insert multiple records
19. Fetching results in different formats
20. Update and delete safely
21. Error handling and resource management
22. SQL injection and security
23. Prepared statements and query composition
24. PostgreSQL JSON and arrays
25. Dates and times
26. Useful PostgreSQL commands
27. Python database project
28. Best practices
29. Cheat sheet
30. Official documentation

---

# 1. SQL, Python, and PostgreSQL

## What is SQL?

SQL means **Structured Query Language**. It is used to define, read, insert, update, and delete data in relational databases.

Example:

```sql
SELECT username, email
FROM users
WHERE active = TRUE;
```

This asks the database to return the username and email of active users.

## What is PostgreSQL?

PostgreSQL is a relational database management system (RDBMS) that understands SQL and adds its own features, including JSON/JSONB, arrays, rich data types, window functions, and advanced indexing.

## What does Python do?

Python connects to a database through a database driver or library. Python sends SQL statements and receives results.

```text
Python application
      |
      | SQL statement + parameters
      v
Database driver (for example, Psycopg)
      |
      v
PostgreSQL server
      |
      | result rows / success / error
      v
Python application
```

Important distinction:

- SQL syntax is interpreted by the database.
- Python syntax is interpreted by Python.
- Psycopg is a Python driver for PostgreSQL.
- SQLite is a separate embedded database; it is useful for learning but is not PostgreSQL.

---

# 2. Install database drivers

Use the Python interpreter or virtual environment that will run your program.

## SQLite

Python includes `sqlite3`, so no additional package is normally needed.

```python
import sqlite3
```

## PostgreSQL with Psycopg 3

```powershell
py -m pip install "psycopg[binary]"
```

On Linux/WSL:

```bash
python3 -m pip install "psycopg[binary]"
```

Import it with:

```python
import psycopg
```

The package is named `psycopg`; the optional `binary` extra provides packaged client components for common installations.

## Other PostgreSQL libraries

- **Psycopg 3:** a direct PostgreSQL driver.
- **SQLAlchemy:** SQL toolkit and ORM supporting PostgreSQL and other databases.
- **Django ORM:** Django's database abstraction layer; it can use PostgreSQL via a suitable driver.
- **asyncpg:** asynchronous PostgreSQL driver.

Choose a library based on your application. The examples below focus on Psycopg 3 and standard SQL.

---

# 3. SQL command categories

SQL statements are commonly grouped into categories. The categories are useful for learning, although terminology and classification can vary.

| Category | Meaning | Common commands |
|---|---|---|
| DDL | Data Definition Language | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| DML | Data Manipulation Language | `INSERT`, `UPDATE`, `DELETE`, `MERGE` |
| DQL | Data Query Language (often used as a teaching category) | `SELECT` |
| DCL | Data Control Language | `GRANT`, `REVOKE` |
| Transaction control | Controls transactions | `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

Examples:

```sql
-- DDL: create a table
CREATE TABLE customers (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    username TEXT NOT NULL
);

-- DML: insert a row
INSERT INTO customers (username)
VALUES ('nzegge');

-- DQL: query rows
SELECT id, username
FROM customers;

-- DCL: grant permissions (run only with appropriate privileges)
GRANT SELECT ON customers TO reporting_user;

-- Transaction control
BEGIN;
UPDATE customers SET username = 'charles' WHERE id = 1;
ROLLBACK;
```

**Warning:** `DROP`, `TRUNCATE`, and `DELETE` can remove data. Test destructive commands in a practice database first.

---

# 4. Create a database and tables

## Create a PostgreSQL database

Run database-administration commands in `psql` or a database client, using a role with the required permissions.

```sql
CREATE DATABASE learning_db;
```

Connect to it in `psql`:

```text
\connect learning_db
```

`\connect` is a `psql` meta-command, not standard SQL.

## Create a table

```sql
CREATE TABLE products (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(120) NOT NULL,
    category TEXT NOT NULL,
    price NUMERIC(10, 2) NOT NULL CHECK (price >= 0),
    stock INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
    active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

Explanation:

- `id`: automatically generated identifier.
- `PRIMARY KEY`: unique row identifier and not null.
- `VARCHAR(120)`: text up to 120 characters.
- `NUMERIC(10, 2)`: exact decimal number with up to 10 digits total and 2 after the decimal point.
- `CHECK`: enforces a condition.
- `DEFAULT`: value used when an insert omits that column.
- `TIMESTAMPTZ`: timestamp with time-zone-aware semantics; PostgreSQL stores the instant internally and renders it according to the session time zone.

Insert practice data:

```sql
INSERT INTO products (name, category, price, stock)
VALUES
    ('Keyboard', 'Accessories', 25.50, 10),
    ('Mouse', 'Accessories', 12.99, 30),
    ('Monitor', 'Displays', 180.00, 5);
```

View it:

```sql
SELECT * FROM products;
```

Avoid relying on `SELECT *` in application code when the exact columns matter. Naming columns makes code more stable and easier to review.

---

# 5. SQL data types

Common PostgreSQL data types:

| Type | Use | Example |
|---|---|---|
| `SMALLINT` | Small integer | `12` |
| `INTEGER` / `INT` | Regular integer | `500` |
| `BIGINT` | Large integer | `9000000000` |
| `NUMERIC(p,s)` | Exact decimal | `NUMERIC(10,2)` |
| `REAL` | Approximate floating point | `3.14` |
| `DOUBLE PRECISION` | Approximate floating point | `3.1415926` |
| `BOOLEAN` | True/false | `TRUE` |
| `TEXT` | Text | `'Hello'` |
| `VARCHAR(n)` | Text with length limit | `VARCHAR(80)` |
| `DATE` | Calendar date | `'2026-10-27'` |
| `TIME` | Time of day | `'13:45:00'` |
| `TIMESTAMP` | Timestamp without time zone | `'2026-10-27 13:45:00'` |
| `TIMESTAMPTZ` | Time-zone-aware timestamp semantics | `CURRENT_TIMESTAMP` |
| `UUID` | UUID identifier | `'...'` |
| `JSON` | JSON document | `'{"ok": true}'` |
| `JSONB` | Binary-optimized JSON storage | `'{"ok": true}'` |
| `BYTEA` | Binary bytes | binary data |
| `TEXT[]` | Text array | `ARRAY['a','b']` |

Use `NUMERIC` for values such as prices when exact decimal arithmetic is required. Approximate floating-point types can produce rounding effects.

---

# 6. CRUD: Create, Read, Update, Delete

CRUD means Create, Read, Update, and Delete.

## INSERT: create records

```sql
INSERT INTO products (name, category, price, stock)
VALUES ('Webcam', 'Accessories', 45.00, 8);
```

Return the newly created row:

```sql
INSERT INTO products (name, category, price, stock)
VALUES ('Webcam', 'Accessories', 45.00, 8)
RETURNING id, name, price;
```

`RETURNING` is supported by PostgreSQL and is useful for obtaining generated IDs.

Insert multiple rows:

```sql
INSERT INTO products (name, category, price, stock)
VALUES
    ('USB Cable', 'Accessories', 5.50, 100),
    ('Desk Lamp', 'Office', 22.00, 15);
```

## SELECT: read records

```sql
SELECT id, name, price
FROM products;
```

Filter:

```sql
SELECT id, name, price
FROM products
WHERE price < 50;
```

## UPDATE: change records

```sql
UPDATE products
SET price = 29.99
WHERE id = 1;
```

**Always check the `WHERE` clause.** Without it, every row in the table is updated.

Use `RETURNING` to inspect changed rows:

```sql
UPDATE products
SET stock = stock - 1
WHERE id = 1 AND stock > 0
RETURNING id, name, stock;
```

## DELETE: remove records

```sql
DELETE FROM products
WHERE id = 3;
```

Inspect deleted rows:

```sql
DELETE FROM products
WHERE stock = 0
RETURNING id, name;
```

Without a `WHERE` clause, `DELETE FROM products` deletes all rows. It does not remove the table structure.

## DELETE versus TRUNCATE versus DROP

```sql
DELETE FROM products WHERE id = 10;
TRUNCATE TABLE products;
DROP TABLE products;
```

- `DELETE`: removes selected rows, or all rows if no condition is supplied.
- `TRUNCATE`: removes all rows efficiently; PostgreSQL supports options such as `RESTART IDENTITY` and `CASCADE`, which need careful use.
- `DROP TABLE`: removes the table itself, including its definition.

These commands are not interchangeable. Use them deliberately.

---

# 7. Filtering, sorting, and pagination

## WHERE operators

```sql
SELECT * FROM products WHERE price >= 20;
SELECT * FROM products WHERE category = 'Accessories';
SELECT * FROM products WHERE category <> 'Accessories';
SELECT * FROM products WHERE stock BETWEEN 5 AND 20;
SELECT * FROM products WHERE category IN ('Office', 'Displays');
SELECT * FROM products WHERE name LIKE 'M%';
SELECT * FROM products WHERE name ILIKE '%mouse%';
SELECT * FROM products WHERE stock IS NULL;
SELECT * FROM products WHERE stock IS NOT NULL;
```

- `=`: equal.
- `<>`: not equal.
- `>=`, `<=`, `>`, `<`: comparisons.
- `BETWEEN`: inclusive range.
- `IN`: matches one of the supplied values.
- `LIKE`: pattern matching, case-sensitive in PostgreSQL.
- `ILIKE`: case-insensitive pattern matching in PostgreSQL.
- `IS NULL`: checks null values.

`NULL` is not the same as zero, an empty string, or Python's `False`. Use `IS NULL`, not `= NULL`.

## Combine conditions

```sql
SELECT *
FROM products
WHERE category = 'Accessories'
  AND price < 50;
```

```sql
SELECT *
FROM products
WHERE category = 'Displays'
   OR stock > 20;
```

Use parentheses when mixing `AND` and `OR` so the intended logic is explicit.

## ORDER BY

```sql
SELECT id, name, price
FROM products
ORDER BY price ASC;
```

```sql
SELECT id, name, price
FROM products
ORDER BY price DESC, name ASC;
```

## LIMIT and OFFSET

```sql
SELECT id, name
FROM products
ORDER BY id
LIMIT 10 OFFSET 20;
```

This requests up to 10 rows after skipping the first 20 ordered rows. Always use a deterministic `ORDER BY` for pagination.

For very large tables, keyset pagination (for example, `WHERE id > last_seen_id ORDER BY id LIMIT 10`) can be more efficient than large offsets.

---

# 8. Aggregate functions and GROUP BY

Common aggregate functions:

```sql
SELECT COUNT(*) FROM products;
SELECT MIN(price) FROM products;
SELECT MAX(price) FROM products;
SELECT AVG(price) FROM products;
SELECT SUM(stock) FROM products;
```

Group rows:

```sql
SELECT category, COUNT(*) AS product_count
FROM products
GROUP BY category;
```

Calculate inventory value by category:

```sql
SELECT category, SUM(price * stock) AS inventory_value
FROM products
GROUP BY category
ORDER BY inventory_value DESC;
```

Filter groups with `HAVING`:

```sql
SELECT category, COUNT(*) AS product_count
FROM products
GROUP BY category
HAVING COUNT(*) >= 2;
```

Difference:
- `WHERE` filters individual rows before grouping.
- `HAVING` filters the resulting groups.

---

# 9. JOINs

Assume two tables:

```sql
CREATE TABLE customers (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL
);

CREATE TABLE orders (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id BIGINT REFERENCES customers(id),
    total NUMERIC(10, 2) NOT NULL CHECK (total >= 0),
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

## INNER JOIN

Returns matching customer/order pairs:

```sql
SELECT customers.name, orders.id, orders.total
FROM customers
INNER JOIN orders
    ON orders.customer_id = customers.id;
```

## LEFT JOIN

Returns all customers, including customers without orders:

```sql
SELECT customers.name, orders.id, orders.total
FROM customers
LEFT JOIN orders
    ON orders.customer_id = customers.id;
```

For customers with no orders:

```sql
SELECT customers.id, customers.name
FROM customers
LEFT JOIN orders ON orders.customer_id = customers.id
WHERE orders.id IS NULL;
```

## RIGHT JOIN and FULL OUTER JOIN

```sql
SELECT c.name, o.id
FROM customers AS c
RIGHT JOIN orders AS o ON o.customer_id = c.id;
```

```sql
SELECT c.name, o.id
FROM customers AS c
FULL OUTER JOIN orders AS o ON o.customer_id = c.id;
```

- `RIGHT JOIN`: all rows from the right side.
- `FULL OUTER JOIN`: all rows from both sides, matched where possible.

In many codebases, `LEFT JOIN` is preferred for readability; reversing table order can often avoid a `RIGHT JOIN`.

## CROSS JOIN

```sql
SELECT c.name, p.name
FROM customers AS c
CROSS JOIN products AS p;
```

This creates every possible customer/product pair. It can generate a very large result.

## Self join

```sql
CREATE TABLE employees (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name TEXT NOT NULL,
    manager_id BIGINT REFERENCES employees(id)
);

SELECT employee.name AS employee,
       manager.name AS manager
FROM employees AS employee
LEFT JOIN employees AS manager
    ON employee.manager_id = manager.id;
```

---

# 10. Subqueries and common table expressions

## Subquery

Find products above the average price:

```sql
SELECT id, name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

## EXISTS

Find customers who have at least one order:

```sql
SELECT c.id, c.name
FROM customers AS c
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customer_id = c.id
);
```

## Common Table Expression (CTE)

```sql
WITH category_totals AS (
    SELECT category, SUM(price * stock) AS inventory_value
    FROM products
    GROUP BY category
)
SELECT category, inventory_value
FROM category_totals
WHERE inventory_value > 100;
```

A CTE gives a named result that can make a complex query easier to read.

## Window function

Rank products by price within each category:

```sql
SELECT
    id,
    name,
    category,
    price,
    RANK() OVER (
        PARTITION BY category
        ORDER BY price DESC
    ) AS price_rank
FROM products;
```

Unlike ordinary grouping, window functions can return row-level details while calculating rankings or aggregates across related rows.

---

# 11. Constraints and indexes

## Constraints

```sql
CREATE TABLE accounts (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    username TEXT NOT NULL UNIQUE,
    balance NUMERIC(12, 2) NOT NULL CHECK (balance >= 0),
    status TEXT NOT NULL DEFAULT 'active',
    CONSTRAINT valid_status CHECK (status IN ('active', 'suspended', 'closed'))
);
```

- `PRIMARY KEY`: unique row identity.
- `NOT NULL`: a value is required.
- `UNIQUE`: values must be unique.
- `CHECK`: validates a condition.
- `FOREIGN KEY`: maintains a relationship to another table.
- `DEFAULT`: supplies a value if one is not provided.

Add a constraint:

```sql
ALTER TABLE products
ADD CONSTRAINT positive_price CHECK (price >= 0);
```

## Indexes

```sql
CREATE INDEX idx_products_category
ON products(category);
```

An index can speed up lookups but consumes storage and adds overhead to writes. Do not index every column automatically.

Composite index:

```sql
CREATE INDEX idx_products_category_price
ON products(category, price);
```

The order of columns matters for which query patterns the index can support efficiently.

Unique index:

```sql
CREATE UNIQUE INDEX idx_customers_email
ON customers(email);
```

Inspect query plans:

```sql
EXPLAIN
SELECT id, name
FROM products
WHERE category = 'Accessories';
```

`EXPLAIN ANALYZE` actually runs the query while measuring it. Use care with statements that modify data.

---

# 12. Transactions

A transaction groups changes into a unit of work.

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1 AND balance >= 100;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

This example is only a sketch: real transfer logic must check that the first update affected exactly one row, validate both account IDs, and handle errors. Otherwise, the second update could run even if the debit failed.

Undo an uncommitted transaction:

```sql
ROLLBACK;
```

Use a savepoint:

```sql
BEGIN;

UPDATE products SET stock = stock - 1 WHERE id = 1;

SAVEPOINT after_stock_update;

UPDATE products SET price = -1 WHERE id = 1;

ROLLBACK TO SAVEPOINT after_stock_update;

COMMIT;
```

The `CHECK (price >= 0)` constraint would reject the negative price update.

Python drivers generally support transaction management. With Psycopg, a connection context manager can commit on success and roll back on exception. See the Python examples below.

---

# 13. PostgreSQL-specific features

## `RETURNING`

```sql
INSERT INTO products (name, category, price, stock)
VALUES ('Headphones', 'Accessories', 35.00, 12)
RETURNING id, name;
```

## `ON CONFLICT` (upsert)

Suppose a table has a unique SKU:

```sql
CREATE TABLE inventory (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    sku TEXT UNIQUE NOT NULL,
    quantity INTEGER NOT NULL DEFAULT 0
);
```

Insert or update:

```sql
INSERT INTO inventory (sku, quantity)
VALUES ('SKU-001', 5)
ON CONFLICT (sku)
DO UPDATE SET quantity = inventory.quantity + EXCLUDED.quantity
RETURNING id, sku, quantity;
```

`EXCLUDED.quantity` is the value proposed for insertion.

Do not use upsert blindly: choose the conflict target and update behavior that match your business rules.

## `ILIKE`

```sql
SELECT name
FROM products
WHERE name ILIKE '%keyboard%';
```

Case-insensitive pattern matching.

## `DISTINCT ON`

Select the most expensive product per category:

```sql
SELECT DISTINCT ON (category)
       category, name, price
FROM products
ORDER BY category, price DESC, id;
```

`DISTINCT ON` is PostgreSQL-specific. The `ORDER BY` determines which row is retained for each category.

## `FILTER` for aggregates

```sql
SELECT
    COUNT(*) AS total_products,
    COUNT(*) FILTER (WHERE active) AS active_products,
    COUNT(*) FILTER (WHERE stock = 0) AS out_of_stock
FROM products;
```

## JSONB

```sql
CREATE TABLE events (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    payload JSONB NOT NULL
);

INSERT INTO events (payload)
VALUES ('{"type": "login", "user": "nzegge"}');

SELECT payload ->> 'type' AS event_type
FROM events
WHERE payload @> '{"user": "nzegge"}';
```

- `->` extracts a JSON value.
- `->>` extracts a JSON value as text.
- `@>` checks JSONB containment.

## Arrays

```sql
CREATE TABLE articles (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    tags TEXT[] NOT NULL DEFAULT '{}'
);

INSERT INTO articles (tags)
VALUES (ARRAY['python', 'postgresql']);

SELECT *
FROM articles
WHERE tags @> ARRAY['python'];
```

Use arrays when they fit the data model. For many-to-many relationships that need foreign keys and detailed metadata, a separate related table is usually more appropriate.

---

# 14. Python + SQLite: first database program

SQLite is useful for learning because it is included with Python. Its SQL overlaps with PostgreSQL, but not all data types, functions, or SQL features are identical.

```python
import sqlite3
from pathlib import Path

DB_PATH = Path(__file__).resolve().parent / "learning.db"

with sqlite3.connect(DB_PATH) as connection:
    cursor = connection.cursor()

    cursor.execute("""
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            username TEXT NOT NULL UNIQUE,
            email TEXT NOT NULL
        )
    """)

    cursor.execute(
        "INSERT OR IGNORE INTO users (username, email) VALUES (?, ?)",
        ("nzegge", "nzegge@example.com"),
    )

    cursor.execute("SELECT id, username, email FROM users")

    for row in cursor.fetchall():
        print(row)
```

SQLite uses `?` placeholders in this example. Other database drivers use different placeholder conventions.

The `with` block commits successful changes and closes the connection when it exits; on an exception, the transaction is rolled back.

---

# 15. Python + PostgreSQL using Psycopg 3

## Connection settings

For local development, you may use environment variables rather than hardcoding passwords.

Example PowerShell session:

```powershell
$env:PGHOST = "localhost"
$env:PGPORT = "5432"
$env:PGDATABASE = "learning_db"
$env:PGUSER = "your_database_user"
$env:PGPASSWORD = "your_password"
```

Avoid committing real passwords to source control. For production, use your organization's secret-management method or an appropriately protected environment configuration.

## Connect and query

```python
import os
import psycopg

connection_settings = {
    "host": os.environ["PGHOST"],
    "port": os.environ.get("PGPORT", "5432"),
    "dbname": os.environ["PGDATABASE"],
    "user": os.environ["PGUSER"],
    "password": os.environ["PGPASSWORD"],
}

with psycopg.connect(**connection_settings) as connection:
    with connection.cursor() as cursor:
        cursor.execute("SELECT current_database(), current_user")
        row = cursor.fetchone()
        print(row)
```

The connection context commits if the block completes successfully and rolls back if an exception escapes. The connection is closed when the context finishes. The nested cursor context closes the cursor.

## Create a table

```python
with psycopg.connect(**connection_settings) as connection:
    with connection.cursor() as cursor:
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS py_users (
                id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
                username TEXT NOT NULL UNIQUE,
                email TEXT NOT NULL
            )
        """)
```

## Insert with parameters

```python
username = "nzegge"
email = "nzegge@example.com"

with psycopg.connect(**connection_settings) as connection:
    with connection.cursor() as cursor:
        cursor.execute(
            """
            INSERT INTO py_users (username, email)
            VALUES (%s, %s)
            ON CONFLICT (username) DO NOTHING
            RETURNING id, username, email
            """,
            (username, email),
        )

        row = cursor.fetchone()
        print(row)
```

For Psycopg, the normal positional placeholder is `%s`, regardless of the Python value's type. Do **not** put quotation marks around the placeholder. The driver handles value adaptation.

If `ON CONFLICT DO NOTHING` skips an existing username, `RETURNING` produces no row and `fetchone()` returns `None`.

---

# 16. SQL parameter styles: `?`, `%s`, named parameters

Different Python libraries use different placeholder formats.

| Library / API | Common placeholder | Example |
|---|---|---|
| Python `sqlite3` qmark style | `?` | `WHERE id = ?` |
| Psycopg 3 positional | `%s` | `WHERE id = %s` |
| Psycopg 3 named | `%(name)s` | `WHERE id = %(id)s` |
| Some DB-API drivers | `:name` or `:1` | Depends on driver |

SQLite:

```python
cursor.execute(
    "SELECT id, username FROM users WHERE id = ?",
    (user_id,),
)
```

Psycopg:

```python
cursor.execute(
    "SELECT id, username FROM py_users WHERE id = %s",
    (user_id,),
)
```

Psycopg named mapping:

```python
cursor.execute(
    "SELECT id, username FROM py_users WHERE username = %(username)s",
    {"username": username},
)
```

The SQL parameter syntax is defined by the driver, not by Python itself. Check the documentation for your chosen driver.

## Critical rule: never build SQL by concatenating user input

Unsafe:

```python
query = "SELECT * FROM py_users WHERE username = '" + username + "'"
cursor.execute(query)
```

Safe:

```python
cursor.execute(
    "SELECT * FROM py_users WHERE username = %s",
    (username,),
)
```

Parameters protect values from being interpreted as SQL syntax and handle quoting correctly.

---

# 17. Tuples, lists, dictionaries, and named rows

SQL results are returned by the driver. The database does not directly return a Python dictionary unless a driver-specific row factory converts the result.

## A. Tuple rows

Psycopg's default row format is tuple-like:

```python
cursor.execute("SELECT id, username, email FROM py_users")
rows = cursor.fetchall()

for row in rows:
    print(row)
    print("ID:", row[0])
    print("Username:", row[1])
    print("Email:", row[2])
```

Example conceptual result:

```python
[
    (1, "nzegge", "nzegge@example.com"),
    (2, "alice", "alice@example.com"),
]
```

Tuple positions follow the order of columns in the `SELECT` list.

## B. One tuple for parameters

```python
user_id = 5

cursor.execute(
    "SELECT id, username FROM py_users WHERE id = %s",
    (user_id,),
)
```

Why `(user_id,)` and not `(user_id)`?

```python
print(type((5)))   # int
print(type((5,)))  # tuple
```

The comma makes a one-item tuple. The parentheses alone do not.

## C. Lists of tuples for many rows

```python
new_users = [
    ("alice", "alice@example.com"),
    ("bob", "bob@example.com"),
    ("charles", "charles@example.com"),
]
```

Insert with Psycopg 3:

```python
with connection.cursor() as cursor:
    cursor.executemany(
        "INSERT INTO py_users (username, email) VALUES (%s, %s)",
        new_users,
    )
```

`executemany()` executes the operation for each parameter set. For high-volume PostgreSQL inserts, Psycopg's `executemany()` behavior is version-dependent and may not be the fastest approach; use the documented bulk-loading or batching approach for your version and workload.

## D. Dictionary for one record's input

```python
new_user = {
    "username": "maria",
    "email": "maria@example.com",
}
```

Use named placeholders:

```python
cursor.execute(
    """
    INSERT INTO py_users (username, email)
    VALUES (%(username)s, %(email)s)
    RETURNING id, username, email
    """,
    new_user,
)
print(cursor.fetchone())
```

The dictionary keys must match the named placeholders.

## E. List of dictionaries

```python
new_users = [
    {"username": "alice", "email": "alice@example.com"},
    {"username": "bob", "email": "bob@example.com"},
]
```

With Psycopg:

```python
with connection.cursor() as cursor:
    cursor.executemany(
        """
        INSERT INTO py_users (username, email)
        VALUES (%(username)s, %(email)s)
        """,
        new_users,
    )
```

This is convenient when each row has named fields.

## F. Dictionaries as returned rows with Psycopg

Psycopg provides `dict_row`:

```python
import psycopg
from psycopg.rows import dict_row

with psycopg.connect(**connection_settings, row_factory=dict_row) as connection:
    with connection.cursor() as cursor:
        cursor.execute("SELECT id, username, email FROM py_users")

        for row in cursor.fetchall():
            print(row["id"])
            print(row["username"])
            print(row["email"])
```

The result is a mapping-like row, conceptually:

```python
{"id": 1, "username": "nzegge", "email": "nzegge@example.com"}
```

## G. Named tuple-like rows

Psycopg also offers `namedtuple_row`:

```python
from psycopg.rows import namedtuple_row

with psycopg.connect(
    **connection_settings,
    row_factory=namedtuple_row,
) as connection:
    with connection.cursor() as cursor:
        cursor.execute("SELECT id, username, email FROM py_users")

        for row in cursor.fetchall():
            print(row.id)
            print(row.username)
```

Use valid, unique column names for convenient attribute access. SQL aliases can help:

```sql
SELECT id, username AS account_name
FROM py_users;
```

## H. `dict` conversion by column names

You can manually convert tuples to dictionaries:

```python
cursor.execute("SELECT id, username, email FROM py_users")
column_names = [column.name for column in cursor.description]

rows_as_dicts = [
    dict(zip(column_names, row))
    for row in cursor.fetchall()
]
```

Psycopg 3's `cursor.description` column objects expose `.name`. For other drivers, metadata shape can differ.

## Which format should you use?

| Format | Good for |
|---|---|
| Tuple | Compact, fast positional access |
| List of tuples | Many parameter sets with fixed column order |
| Dictionary | One input record with named fields |
| List of dictionaries | Multiple input records with named fields |
| `dict_row` | Result rows accessed by column name |
| `namedtuple_row` | Result rows accessed by attribute |
| Custom object / dataclass | Application logic requiring a typed structure |

Use dictionaries when clarity matters; use tuples when positional data is simple and stable. Do not assume the database itself stores Python tuples or dictionaries—the driver converts data between SQL types and Python types.

---

# 18. Fetching results: `fetchone`, `fetchmany`, `fetchall`

```python
cursor.execute("SELECT id, username FROM py_users ORDER BY id")

first = cursor.fetchone()
print(first)
```

`fetchone()` returns the next row or `None` when no row remains.

```python
cursor.execute("SELECT id, username FROM py_users ORDER BY id")

some_rows = cursor.fetchmany(3)
print(some_rows)
```

`fetchmany(3)` requests up to three rows.

```python
cursor.execute("SELECT id, username FROM py_users ORDER BY id")

all_rows = cursor.fetchall()
print(all_rows)
```

`fetchall()` loads all remaining rows into memory. For a very large result, iterate over the cursor or use an appropriate server-side cursor/batching approach instead.

You can also iterate directly:

```python
cursor.execute("SELECT id, username FROM py_users ORDER BY id")

for row in cursor:
    print(row)
```

---

# 19. Insert multiple records

## Method 1: repeated `execute`

```python
rows = [
    ("alice", "alice@example.com"),
    ("bob", "bob@example.com"),
]

for username, email in rows:
    cursor.execute(
        "INSERT INTO py_users (username, email) VALUES (%s, %s)",
        (username, email),
    )
```

Easy to understand, but repeated statements can be inefficient for large data volumes.

## Method 2: `executemany`

```python
cursor.executemany(
    "INSERT INTO py_users (username, email) VALUES (%s, %s)",
    rows,
)
```

Convenient for repeated statements.

## Method 3: `INSERT ... VALUES` with multiple rows

```sql
INSERT INTO products (name, category, price, stock)
VALUES
    ('Keyboard', 'Accessories', 25.50, 10),
    ('Mouse', 'Accessories', 12.99, 30);
```

## Method 4: bulk loading with PostgreSQL `COPY`

For large imports, PostgreSQL's `COPY` protocol can be more efficient than individual inserts. Psycopg 3 provides `cursor.copy()`.

A simplified CSV file might contain:

```text
name,category,price,stock
Keyboard,Accessories,25.50,10
Mouse,Accessories,12.99,30
```

Use the official Psycopg documentation for the exact `COPY` API and CSV/header handling. Validate input, specify the columns explicitly, and do not treat arbitrary uploaded CSV data as trusted.

---

# 20. Update and delete safely in Python

Update one record:

```python
user_id = 5
new_email = "new-address@example.com"

with psycopg.connect(**connection_settings) as connection:
    with connection.cursor() as cursor:
        cursor.execute(
            "UPDATE py_users SET email = %s WHERE id = %s RETURNING id, username, email",
            (new_email, user_id),
        )
        updated_row = cursor.fetchone()

        if updated_row is None:
            print("No matching user was found.")
        else:
            print("Updated:", updated_row)
```

Delete one record:

```python
user_id = 5

with psycopg.connect(**connection_settings) as connection:
    with connection.cursor() as cursor:
        cursor.execute(
            "DELETE FROM py_users WHERE id = %s RETURNING id, username",
            (user_id,),
        )
        deleted_row = cursor.fetchone()

        if deleted_row is None:
            print("No matching user was found.")
        else:
            print("Deleted:", deleted_row)
```

The connection context commits when the block succeeds. If an exception escapes, it rolls back.

For destructive operations, verify the intended target, use constraints and permissions, and consider transactions or backups.

---

# 21. Error handling and resource management

```python
import psycopg
from psycopg import errors

try:
    with psycopg.connect(**connection_settings) as connection:
        with connection.cursor() as cursor:
            cursor.execute(
                "INSERT INTO py_users (username, email) VALUES (%s, %s)",
                ("unique-user", "unique@example.com"),
            )

except psycopg.errors.UniqueViolation:
    print("That username or email already exists.")

except psycopg.OperationalError as error:
    print("Could not connect to PostgreSQL:", error)

except psycopg.Error as error:
    print("Database error:", error)
```

Catch specific errors when you know how to handle them, then use a broader database exception as a fallback. Avoid exposing raw database error details to untrusted users in a production web application.

For predictable cleanup, use context managers for connections and cursors.

---

# 22. SQL injection and security

SQL injection happens when untrusted input is interpreted as part of SQL syntax.

Unsafe:

```python
username = input("Username: ")
query = f"SELECT * FROM py_users WHERE username = '{username}'"
cursor.execute(query)
```

Safe:

```python
username = input("Username: ")

cursor.execute(
    "SELECT id, username FROM py_users WHERE username = %s",
    (username,),
)
```

## Placeholders are for values, not SQL identifiers

This does not work as intended for dynamic column names:

```python
cursor.execute("SELECT %s FROM py_users", ("username",))
```

The placeholder represents a value, not a column identifier. To choose among columns, use a strict allowlist:

```python
allowed_columns = {"id", "username", "email"}
requested_column = "username"

if requested_column not in allowed_columns:
    raise ValueError("Unsupported column")

query = f"SELECT {requested_column} FROM py_users"
cursor.execute(query)
```

Only insert an identifier into SQL after validating it against trusted choices. For more complex dynamic SQL, use Psycopg's SQL composition utilities.

Also:
- Use a database role with only the permissions the application needs.
- Keep passwords out of source control.
- Use encrypted connections where required.
- Do not log credentials.
- Validate business rules in the application and enforce important invariants with database constraints.

---

# 23. Dynamic SQL and identifiers in Psycopg

Psycopg offers `psycopg.sql` for safely composing identifiers and SQL fragments.

```python
from psycopg import sql

table_name = "py_users"
column_name = "username"

query = sql.SQL("SELECT {} FROM {}").format(
    sql.Identifier(column_name),
    sql.Identifier(table_name),
)

with psycopg.connect(**connection_settings) as connection:
    with connection.cursor() as cursor:
        cursor.execute(query)
        print(cursor.fetchall())
```

- `sql.Identifier()` safely quotes a table or column identifier.
- `%s` placeholders still handle data values.
- Do not use `sql.Identifier()` to represent a normal value such as an email address.

Even with safe identifier composition, apply allowlists to prevent a user from selecting sensitive tables or columns.

---

# 24. PostgreSQL JSON and Python dictionaries

Psycopg supports JSON adaptation through its JSON wrappers.

```python
import json
from psycopg.types.json import Jsonb

event_data = {
    "type": "login",
    "username": "nzegge",
    "success": True,
}

with psycopg.connect(**connection_settings) as connection:
    with connection.cursor() as cursor:
        cursor.execute(
            "INSERT INTO events (payload) VALUES (%s) RETURNING id",
            (Jsonb(event_data),),
        )
        print(cursor.fetchone())
```

This assumes the PostgreSQL `events` table exists with a `payload JSONB` column, as defined earlier.

Query JSONB:

```python
username = "nzegge"

with psycopg.connect(**connection_settings) as connection:
    with connection.cursor() as cursor:
        cursor.execute(
            "SELECT id, payload FROM events WHERE payload ->> 'user' = %s",
            (username,),
        )
        print(cursor.fetchall())
```

JSON/JSONB is useful for flexible document-shaped data, but do not use it as a substitute for relational columns when the data needs strong relationships, constraints, or frequent joins.

---

# 25. Dates and times

PostgreSQL date/time types:

- `DATE`: calendar date.
- `TIME`: time of day.
- `TIMESTAMP`: date and time without time-zone information.
- `TIMESTAMPTZ`: an instant interpreted with time-zone-aware semantics.

Use Python's `datetime` and `date` objects:

```python
from datetime import date, datetime, timezone

today = date.today()
now_utc = datetime.now(timezone.utc)

print(today)
print(now_utc)
```

Pass date/time values as parameters instead of formatting them into SQL strings:

```python
cursor.execute(
    "SELECT id, created_at FROM products WHERE created_at >= %s",
    (now_utc,),
)
```

For systems spanning time zones, prefer aware Python datetimes and define a clear UTC storage/display policy.

---

# 26. Useful PostgreSQL commands (`psql`)

These commands run inside the `psql` interactive terminal; they are not ordinary SQL statements.

| Command | Purpose |
|---|---|
| `\l` | List databases |
| `\c learning_db` | Connect to a database |
| `\dt` | List tables in the current schema/search path |
| `\d products` | Describe a table |
| `\dn` | List schemas |
| `\du` | List roles |
| `\conninfo` | Show current connection details |
| `\timing` | Toggle query timing |
| `\x` | Toggle expanded display |
| `\q` | Quit `psql` |
| `\?` | Help for `psql` meta-commands |
| `\h SELECT` | SQL syntax help for a command |

Ordinary SQL for listing current database/user:

```sql
SELECT current_database(), current_user;
```

Check the PostgreSQL server version:

```sql
SELECT version();
```

---

# 27. Mini-project: product database manager

This example assumes PostgreSQL is running, the `learning_db` database exists, and your environment variables are set as described earlier.

Create `product_manager.py`:

```python
import os
import psycopg
from psycopg.rows import dict_row

DB_CONFIG = {
    "host": os.environ["PGHOST"],
    "port": os.environ.get("PGPORT", "5432"),
    "dbname": os.environ["PGDATABASE"],
    "user": os.environ["PGUSER"],
    "password": os.environ["PGPASSWORD"],
}


def create_table():
    with psycopg.connect(**DB_CONFIG) as connection:
        with connection.cursor() as cursor:
            cursor.execute("""
                CREATE TABLE IF NOT EXISTS app_products (
                    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
                    name TEXT NOT NULL,
                    price NUMERIC(10, 2) NOT NULL CHECK (price >= 0),
                    stock INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0)
                )
            """)


def add_product(name, price, stock):
    with psycopg.connect(**DB_CONFIG) as connection:
        with connection.cursor(row_factory=dict_row) as cursor:
            cursor.execute(
                """
                INSERT INTO app_products (name, price, stock)
                VALUES (%s, %s, %s)
                RETURNING id, name, price, stock
                """,
                (name, price, stock),
            )
            return cursor.fetchone()


def list_products():
    with psycopg.connect(**DB_CONFIG) as connection:
        with connection.cursor(row_factory=dict_row) as cursor:
            cursor.execute("""
                SELECT id, name, price, stock
                FROM app_products
                ORDER BY id
            """)
            return cursor.fetchall()


def update_stock(product_id, new_stock):
    with psycopg.connect(**DB_CONFIG) as connection:
        with connection.cursor(row_factory=dict_row) as cursor:
            cursor.execute(
                """
                UPDATE app_products
                SET stock = %s
                WHERE id = %s
                RETURNING id, name, price, stock
                """,
                (new_stock, product_id),
            )
            return cursor.fetchone()


def delete_product(product_id):
    with psycopg.connect(**DB_CONFIG) as connection:
        with connection.cursor() as cursor:
            cursor.execute(
                "DELETE FROM app_products WHERE id = %s RETURNING id",
                (product_id,),
            )
            return cursor.fetchone() is not None


if __name__ == "__main__":
    create_table()

    created = add_product("Keyboard", 25.50, 10)
    print("Created:", created)

    print("All products:")
    for product in list_products():
        print(product)

    if created:
        updated = update_stock(created["id"], 8)
        print("Updated:", updated)

        # Uncomment only when you want to remove this test record.
        # print("Deleted:", delete_product(created["id"]))
```

Study the separation of responsibilities:
- `create_table()` defines the table.
- `add_product()` inserts and returns a row.
- `list_products()` reads rows as dictionaries.
- `update_stock()` updates one row.
- `delete_product()` deletes one row and reports whether it existed.

For a production application, consider connection pooling, configuration validation, authorization, migrations, tests, and application-specific error handling.

---

# 28. Best practices

1. Use parameterized queries for values.
2. Never concatenate untrusted values into SQL.
3. Use explicit column lists in `SELECT` and `INSERT`.
4. Include `WHERE` in targeted `UPDATE` and `DELETE` operations.
5. Use transactions for related changes.
6. Enforce important rules with constraints.
7. Use `NUMERIC` for exact monetary values.
8. Use aware datetimes and a consistent time-zone policy.
9. Fetch only the rows and columns you need.
10. Do not `fetchall()` huge result sets without considering memory.
11. Use dictionary rows when named access improves readability.
12. Use tuples when the column order is stable and positional access is clear.
13. Use indexes based on real query patterns, then inspect query plans.
14. Use least-privilege database roles.
15. Store secrets outside source code.
16. Test destructive commands in a practice database.
17. Use schema migrations for changes to shared/production databases.
18. Keep database logic organized rather than mixing all SQL into unrelated application code.
19. Handle expected database errors without exposing internal details to users.
20. Back up important databases and regularly test restoration.

---

# 29. Cheat sheet

## SQL basics

```sql
CREATE TABLE example (id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, name TEXT NOT NULL);

INSERT INTO example (name) VALUES ('Alice');

SELECT id, name FROM example WHERE id = 1;

UPDATE example SET name = 'Bob' WHERE id = 1;

DELETE FROM example WHERE id = 1;
```

## Filtering and ordering

```sql
SELECT * FROM products
WHERE price >= 10 AND category IN ('Office', 'Accessories')
ORDER BY price DESC
LIMIT 20;
```

## Grouping

```sql
SELECT category, COUNT(*) AS count
FROM products
GROUP BY category
HAVING COUNT(*) > 1;
```

## PostgreSQL positional parameter

```python
cursor.execute(
    "SELECT id, name FROM products WHERE id = %s",
    (product_id,),
)
```

## PostgreSQL named parameters

```python
cursor.execute(
    "SELECT id, name FROM products WHERE name = %(name)s",
    {"name": product_name},
)
```

## SQLite positional parameter

```python
cursor.execute(
    "SELECT id, name FROM products WHERE id = ?",
    (product_id,),
)
```

## Tuple vs one-item tuple

```python
value = (10)   # int
values = (10,) # tuple
```

## Result formats

```python
row = cursor.fetchone()      # one row
rows = cursor.fetchall()     # all remaining rows
rows = cursor.fetchmany(5)   # up to five rows
```

## Psycopg dictionary row

```python
from psycopg.rows import dict_row

with psycopg.connect(**DB_CONFIG, row_factory=dict_row) as connection:
    with connection.cursor() as cursor:
        cursor.execute("SELECT id, name FROM products")
        print(cursor.fetchall())
```

---

# 30. Official documentation

- PostgreSQL documentation: https://www.postgresql.org/docs/
- PostgreSQL SQL commands: https://www.postgresql.org/docs/current/sql-commands.html
- PostgreSQL `psql`: https://www.postgresql.org/docs/current/app-psql.html
- Psycopg 3 documentation: https://www.psycopg.org/psycopg3/docs/
- Python `sqlite3`: https://docs.python.org/3/library/sqlite3.html
- Python DB-API specification (PEP 249): https://peps.python.org/pep-0249/
- SQLAlchemy: https://docs.sqlalchemy.org/
- Django databases: https://docs.djangoproject.com/en/stable/topics/db/

## Suggested learning order

1. Learn `CREATE TABLE`, `INSERT`, `SELECT`, `UPDATE`, and `DELETE`.
2. Practise `WHERE`, `ORDER BY`, `LIMIT`, and `GROUP BY`.
3. Learn primary keys, foreign keys, and joins.
4. Practise Python tuples and parameterized SQL.
5. Connect Python to SQLite to understand the database workflow.
6. Connect Python to PostgreSQL using Psycopg.
7. Learn dictionary rows, transactions, exceptions, and bulk inserts.
8. Study indexes, query plans, JSONB, and window functions.
9. Build and test the product manager project.
