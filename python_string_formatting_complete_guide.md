# Python String Formatting — Complete Guide
## From basic concatenation to f-strings, dictionaries, templates, and SQL parameters

This guide covers Python string formatting and explains the important difference between formatting ordinary text and binding values to SQL queries.

> **Remember:** use f-strings for normal Python messages. Use your database driver's parameter placeholders for SQL values. Do not insert untrusted values into SQL using f-strings or concatenation.

## Contents
1. Concatenation and `str()`
2. Old `%` formatting
3. Named `%` formatting with dictionaries
4. `.format()` and `**dictionary`
5. F-strings
6. Number formatting
7. Date and time formatting
8. `string.Template`
9. Formatting collections
10. Escaping characters
11. SQL parameter binding
12. SQLite examples
13. PostgreSQL/Psycopg examples
14. Multiple records
15. Common errors and best practices
16. Quick reference
17. Exercises and answers

---

# 1. What is string formatting?

String formatting creates text that contains values from variables or expressions.

```python
username = "Charles"
age = 25
message = f"My name is {username} and I am {age} years old."
print(message)
```

Output: `My name is Charles and I am 25 years old.`

# 2. Concatenation and `str()`

Concatenation joins strings using `+`.

```python
first_name = "Charles"
last_name = "Ngong"
print(first_name + " " + last_name)
```

You cannot directly concatenate a string and an integer:

```python
age = 25
# print("Age: " + age)  # TypeError
print("Age: " + str(age))
```

`str(value)` converts a value to its text representation.

```python
print(str(25))       # "25"
print(str(3.14))     # "3.14"
print(str(True))     # "True"
```

Related conversions:
- `str(value)`: value -> text.
- `int(value)`: compatible value -> integer.
- `float(value)`: compatible value -> floating-point number.
- `repr(value)`: representation often useful for debugging.

# 3. Old-style `%` formatting

The `%` operator can place values into placeholders.

```python
username = "Charles"
age = 25
message = "Name: %s, Age: %d" % (username, age)
print(message)
```

Output: `Name: Charles, Age: 25`

The tuple `(username, age)` supplies values in order.

| Placeholder | Meaning | Example |
|---|---|---|
| `%s` | String representation | `"Charles"` |
| `%r` | Object representation | `"'Charles'"` |
| `%d` or `%i` | Integer formatting | `25` |
| `%f` | Float, normally six decimal places | `3.140000` |
| `%.2f` | Float to two decimal places | `3.14` |
| `%e` | Scientific notation | `3.14e+00` |
| `%x` | Integer in hexadecimal | `ff` for 255 |
| `%o` | Integer in octal | `377` for 255 |
| `%%` | Literal percent sign | `%` |

Example:

```python
name = "Charles"
age = 25
price = 19.987

print("Name: %s" % name)
print("Age: %d" % age)
print("Price: %.2f" % price)
print("Progress: 80%%")
```

Output:

```text
Name: Charles
Age: 25
Price: 19.99
Progress: 80%
```

## A one-item tuple needs a comma

```python
a = ("Charles")   # string, not a tuple
b = ("Charles",) # one-item tuple
```

# 4. Named `%` formatting with dictionaries: `%(username)s`

A dictionary lets you look up values by key instead of position.

```python
user = {
    "username": "Charles",
    "age": 25
}

message = "Name: %(username)s, Age: %(age)d" % user
print(message)
```

Output: `Name: Charles, Age: 25`

Break down `%(username)s`:
- `%`: begins an old-style formatting placeholder.
- `(username)`: dictionary key to look up.
- `s`: format the value as a string.

For `%(age)d`, `(age)` is the key and `d` requests integer formatting.

Example:

```python
product = {"name": "Laptop", "price": 450000, "quantity": 3}

message = (
    "Product: %(name)s | Price: %(price)d | Quantity: %(quantity)d"
    % product
)
print(message)
```

This is ordinary Python formatting. Psycopg also uses syntax like `%(username)s` for SQL parameters, but that is handled by the database driver rather than Python's `%` operator. See the SQL section below.

# 5. The `.format()` method

`.format()` fills curly-brace placeholders.

```python
username = "Charles"
age = 25

print("Name: {}, Age: {}".format(username, age))
print("Name: {0}, Age: {1}".format(username, age))
print("Age: {1}, Name: {0}".format(username, age))
```

- `{}`: use the next argument.
- `{0}`: first argument.
- `{1}`: second argument.

Named arguments:

```python
print("Name: {username}, Age: {age}".format(
    username="Charles",
    age=25
))
```

Use a dictionary with `**`:

```python
user = {"username": "Charles", "age": 25}
print("Name: {username}, Age: {age}".format(**user))
```

`**user` unpacks dictionary entries as keyword arguments.

You can access dictionary keys from an argument:

```python
person = {"name": "Charles", "age": 25}
print("{0[name]} is {0[age]} years old.".format(person))
```

# 6. F-strings: recommended for ordinary Python text

F-strings were introduced in Python 3.6. Put `f` before the string and expressions inside `{}`.

```python
username = "Charles"
age = 25

print(f"My name is {username} and I am {age} years old.")
```

Expressions also work:

```python
a = 10
b = 5

print(f"Sum: {a + b}")
print(f"Product: {a * b}")
print(f"a is greater than b: {a > b}")
```

Dictionary access:

```python
user = {"username": "Charles", "age": 25}
print(f"Name: {user['username']}, Age: {user['age']}")
```

Debugging with `=`:

```python
age = 25
print(f"{age=}")  # age=25
```

Conversion flags:

```python
name = "Charles"
print(f"{name!s}")  # calls str(name)
print(f"{name!r}")  # calls repr(name)
print(f"{name!a}")  # calls ascii(name)
```

Use f-strings for normal messages, reports, and debug output. Do not use them to interpolate untrusted values into SQL.

# 7. Number formatting

A format specification follows a colon inside `{}`.

## Decimal places

```python
number = 12.34567
print(f"{number:.2f}")   # 12.35
print(f"{number:.3f}")   # 12.346
print(f"{number:10.2f}") # minimum width 10, 2 decimals
```

Formatting changes the displayed text, not the original variable.

## Alignment and width

```python
word = "Python"
print(f"|{word:<10}|")  # left
print(f"|{word:>10}|")  # right
print(f"|{word:^10}|")  # centered
```

## Fill characters

```python
number = 42
print(f"{number:0>5}")  # 00042
print(f"{number:*^7}")  # **42***
```

## Thousands separators

```python
amount = 1234567.89
print(f"{amount:,}")     # 1,234,567.89
print(f"{amount:,.2f}")  # 1,234,567.89
```

## Percentage

```python
ratio = 0.875
print(f"{ratio:.1%}")  # 87.5%
```

The `%` format type multiplies the displayed value by 100 and appends `%`.

## Binary, octal, hexadecimal

```python
number = 255
print(f"{number:b}")   # 11111111
print(f"{number:o}")   # 377
print(f"{number:x}")   # ff
print(f"{number:X}")   # FF
print(f"{number:#x}")  # 0xff
print(f"{number:#b}")  # 0b11111111
```

These change the representation, not the stored integer.

# 8. Date and time formatting

```python
from datetime import datetime

now = datetime.now()
print(f"Date: {now:%Y-%m-%d}")
print(f"Time: {now:%H:%M:%S}")
print(f"Full: {now:%A, %d %B %Y}")
```

Common directives:

| Directive | Meaning |
|---|---|
| `%Y` | Four-digit year |
| `%y` | Two-digit year |
| `%m` | Month number |
| `%B` | Full month name |
| `%b` | Abbreviated month |
| `%d` | Day of month |
| `%A` | Full weekday |
| `%a` | Abbreviated weekday |
| `%H` | Hour, 24-hour clock |
| `%I` | Hour, 12-hour clock |
| `%M` | Minute |
| `%S` | Second |
| `%p` | AM/PM marker |

`strftime()` converts a date/time object to text:

```python
print(datetime.now().strftime("%Y-%m-%d %H:%M:%S"))
```

`strptime()` parses text into a date/time object:

```python
date_text = "2026-10-27"
date_object = datetime.strptime(date_text, "%Y-%m-%d")
print(date_object)
```

Remember: `strftime` = object to string; `strptime` = string to object.

# 9. `string.Template`

`string.Template` uses `$name` placeholders.

```python
from string import Template

template = Template("Name: $username, Age: $age")
user = {"username": "Charles", "age": 25}

print(template.substitute(user))
```

Use `${name}` when the placeholder touches adjacent text:

```python
template = Template("Welcome, ${username}_dev!")
print(template.substitute(username="Charles"))
```

- `substitute()` raises an error if a required key is missing.
- `safe_substitute()` leaves unresolved placeholders instead of raising that missing-key error.

# 10. Formatting collections and objects

```python
languages = ["Python", "JavaScript", "SQL"]
user_tuple = ("Charles", 25)
user_dict = {"username": "Charles", "age": 25}

print(f"Languages: {languages}")
print(f"Tuple: {user_tuple}")
print(f"Dictionary: {user_dict}")
```

For a custom message, access individual fields:

```python
user = {"username": "Charles", "age": 25}
print(f"{user['username']} is {user['age']} years old.")
```

`str()` is designed for readable display; `repr()` is useful for debugging and may show escape sequences.

# 11. Escaping braces and percent signs

To display literal braces in an f-string, double them:

```python
name = "Charles"
print(f"{{username}} = {name}")
# {username} = Charles
```

- `{{` displays `{`.
- `}}` displays `}`.

In old-style `%` formatting, `%%` displays one percent sign:

```python
print("Progress: 80%% complete" % ())
```

In an f-string, a literal percent sign can be written directly:

```python
progress = 80
print(f"Progress: {progress}% complete")
```

# 12. Python formatting versus SQL parameter binding

This distinction is essential.

## Ordinary Python formatting

```python
user = {"username": "Charles", "age": 25}
message = "Name: %(username)s, Age: %(age)d" % user
print(message)
```

Python creates a finished text string.

## SQL parameter binding

```python
cursor.execute(
    """
    INSERT INTO users (username, age)
    VALUES (%(username)s, %(age)s)
    """,
    {"username": "Charles", "age": 25}
)
```

With Psycopg, the SQL and dictionary are passed separately. Psycopg handles the parameter binding. Python does not run `"...%(...)" % user` on the SQL statement first.

Psycopg uses `%s` for both strings and integers. Here `%s` is a driver parameter marker, not the old Python `%s` conversion instruction. The driver adapts the Python values to database types.

**Never construct SQL from untrusted input using f-strings or concatenation:**

```python
# UNSAFE — do not do this:
# sql = f"SELECT * FROM users WHERE username = '{username}'"
```

Use the driver's parameter placeholders. Placeholders are for values, not table names or column names; dynamic SQL identifiers require the database library's safe identifier-composition feature.

# 13. SQLite parameter styles

Python's built-in `sqlite3` supports `?` positional parameters and `:name` named parameters.

## Positional parameters with a tuple

```python
import sqlite3

conn = sqlite3.connect("school.db")
cursor = conn.cursor()

params = ("Charles", 25)
cursor.execute(
    "INSERT INTO users (username, age) VALUES (?, ?)",
    params
)

conn.commit()
conn.close()
```

## Named parameters with a dictionary

```python
import sqlite3

conn = sqlite3.connect("school.db")
cursor = conn.cursor()

user = {"username": "Charles", "age": 25}
cursor.execute(
    """
    INSERT INTO users (username, age)
    VALUES (:username, :age)
    """,
    user
)

conn.commit()
conn.close()
```

The `:username` placeholder matches the dictionary key `"username"`; the colon is not part of the key.

# 14. PostgreSQL with Psycopg

Psycopg uses `%s` for positional parameters and `%(name)s` for named dictionary parameters.

## Positional tuple

```python
import psycopg

user = ("Charles", 25)

with psycopg.connect(
    "dbname=mydb user=postgres password=mypassword host=localhost"
) as conn:
    with conn.cursor() as cursor:
        cursor.execute(
            "INSERT INTO users (username, age) VALUES (%s, %s)",
            user
        )
```

## Named dictionary

```python
import psycopg

user = {"username": "Charles", "age": 25}

with psycopg.connect(
    "dbname=mydb user=postgres password=mypassword host=localhost"
) as conn:
    with conn.cursor() as cursor:
        cursor.execute(
            """
            INSERT INTO users (username, age)
            VALUES (%(username)s, %(age)s)
            """,
            user
        )
```

Replace the example database name, user, password, and host with your own configuration. Avoid committing real credentials to a repository.

# 15. Insert multiple records

## List of dictionaries

```python
users = [
    {"username": "Charles", "age": 25},
    {"username": "Alice", "age": 22},
    {"username": "Bob", "age": 30},
]
```

SQLite:

```python
cursor.executemany(
    "INSERT INTO users (username, age) VALUES (:username, :age)",
    users
)
```

Psycopg:

```python
cursor.executemany(
    "INSERT INTO users (username, age) VALUES (%(username)s, %(age)s)",
    users
)
```

Each dictionary represents one record, and its keys match the named placeholders.

## List of tuples

```python
users = [
    ("Charles", 25),
    ("Alice", 22),
    ("Bob", 30),
]
```

SQLite:

```python
cursor.executemany(
    "INSERT INTO users (username, age) VALUES (?, ?)",
    users
)
```

Psycopg:

```python
cursor.executemany(
    "INSERT INTO users (username, age) VALUES (%s, %s)",
    users
)
```

Each tuple supplies values in column order.

## Tuple of tuples

```python
users = (
    ("Charles", 25),
    ("Alice", 22),
    ("Bob", 30),
)
```

This can also be passed to `executemany()` with the same SQL placeholders as the list of tuples.

| Structure | Example | Values identified by |
|---|---|---|
| List of dictionaries | `[{"username": "Charles", "age": 25}]` | Keys |
| List of tuples | `[("Charles", 25)]` | Positions |
| Tuple of tuples | `(("Charles", 25),)` | Positions |

`executemany()` runs a parameterized statement for multiple parameter sets. It does not necessarily create one SQL statement containing all rows; exact behavior depends on the driver.

# 16. Common mistakes

## Mistake: string plus integer

```python
age = 25
# "Age: " + age  # TypeError
print(f"Age: {age}")
```

## Mistake: missing dictionary key

```python
user = {"username": "Charles"}
# "%(age)d" % user  # KeyError: age is missing
```

## Mistake: forgetting the comma in a one-item tuple

```python
params = ("Charles",)  # tuple
params = ("Charles")   # string
```

## Mistake: mixing SQL placeholder styles

Do not assume SQLite's `:username`, Psycopg's `%(username)s`, and f-string `{username}` are interchangeable. Use the syntax expected by the library.

## Mistake: using `%d` as a Psycopg placeholder

For ordinary Python `%` formatting, `%d` means integer formatting. For Psycopg SQL parameters, use `%s` for values of different types.

# 17. Best practices

1. Use f-strings for ordinary Python messages.
2. Learn `%` formatting because you may see it in older code.
3. Use `.format()` when working in code that already uses it.
4. Always use parameterized queries for SQL values.
5. SQLite `sqlite3`: use `?` or `:name`.
6. Psycopg: use `%s` or `%(name)s`.
7. Use dictionaries when named fields improve readability.
8. Use tuples when positional order is clear and consistent.
9. Match tuple value order to SQL column order.
10. Remember that formatting to two decimals changes the display, not the original number.
11. Keep passwords and other secrets out of public source code.

# 18. Quick reference

| Goal | Syntax | Example |
|---|---|---|
| Join strings | `+` | `"Hello " + name` |
| Convert value to text | `str()` | `str(age)` |
| Old positional formatting | `%s`, `%d` | `"%s is %d" % (name, age)` |
| Old named formatting | `%(key)s` | `"%(username)s" % user` |
| `.format()` positional | `{}` | `"Hello {}".format(name)` |
| `.format()` named | `{name}` | `"{name}".format(name="Charles")` |
| Dictionary unpacking | `**dict` | `"{username}".format(**user)` |
| F-string | `{expression}` | `f"Hello {name}"` |
| Two decimal places | `:.2f` | `f"{price:.2f}"` |
| Thousands separator | `:,` | `f"{amount:,}"` |
| Percentage display | `:.1%` | `f"{ratio:.1%}"` |
| Left aligned | `:<width` | `f"{name:<10}"` |
| Right aligned | `:>width` | `f"{name:>10}"` |
| Centered | `:^width` | `f"{name:^10}"` |
| Format date | `%Y-%m-%d` | `now.strftime("%Y-%m-%d")` |
| Template placeholder | `$name` | `Template("Hi $name")` |
| SQLite positional SQL | `?` | `VALUES (?, ?)` |
| SQLite named SQL | `:name` | `VALUES (:username, :age)` |
| Psycopg positional SQL | `%s` | `VALUES (%s, %s)` |
| Psycopg named SQL | `%(name)s` | `VALUES (%(username)s, %(age)s)` |

# 19. Practice exercises

Use:

```python
product = {
    "name": "Laptop",
    "price": 450000,
    "quantity": 3
}
```

1. Use concatenation to print `Product: Laptop`.
2. Use named `%` formatting to print `Product: Laptop | Price: 450000 | Quantity: 3`.
3. Use `.format(**product)` to create the same output.
4. Use an f-string to create the same output.
5. Display `1234.5678` to two decimal places.
6. Display `0.825` as `82.5%`.
7. Write an SQLite insert using `:name` and `:price`, passing a dictionary separately.
8. Write a Psycopg insert using `%(name)s` and `%(price)s`, passing a dictionary separately.

## Answers

### 1. Concatenation

```python
print("Product: " + product["name"])
```

### 2. Named `%` formatting

```python
print(
    "Product: %(name)s | Price: %(price)d | Quantity: %(quantity)d"
    % product
)
```

### 3. `.format(**product)`

```python
print(
    "Product: {name} | Price: {price} | Quantity: {quantity}".format(**product)
)
```

### 4. F-string

```python
print(
    f"Product: {product['name']} | "
    f"Price: {product['price']} | "
    f"Quantity: {product['quantity']}"
)
```

### 5. Two decimal places

```python
price = 1234.5678
print(f"{price:.2f}")  # 1234.57
```

### 6. Percentage

```python
ratio = 0.825
print(f"{ratio:.1%}")  # 82.5%
```

### 7. SQLite

```python
product_data = {"name": "Laptop", "price": 450000}

cursor.execute(
    "INSERT INTO products (name, price) VALUES (:name, :price)",
    product_data
)
```

### 8. PostgreSQL with Psycopg

```python
product_data = {"name": "Laptop", "price": 450000}

cursor.execute(
    "INSERT INTO products (name, price) VALUES (%(name)s, %(price)s)",
    product_data
)
```

In both SQL examples, values are passed separately to the database driver.

---

# Final recommendation

Study in this order:

1. Concatenation and `str()`.
2. Old `%` formatting, including `%(key)s`.
3. `.format()` and dictionary unpacking with `**`.
4. F-strings and number formatting.
5. Date/time formatting and `string.Template`.
6. SQL parameter binding for SQLite and PostgreSQL.

**Core rule:** Format normal Python text with f-strings. Send SQL values using the database driver's parameter placeholders.
