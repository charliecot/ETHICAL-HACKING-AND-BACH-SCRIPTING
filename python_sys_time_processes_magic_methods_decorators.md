# Python Reference Guide: `sys`, `time`, Processes, Magic Methods, and Decorators

> Beginner-friendly notes with examples, practical use cases, and best practices.
>
> **Python version:** Most examples work on modern Python 3. `sys.exception()` requires Python 3.11 or newer.

---

## Table of Contents

1. The `sys` module
2. The `time` module
3. Processes in Python
4. Magic (dunder) methods
5. Function decorators
6. Class decorators
7. Choosing the right tool
8. Practice projects

---

# 1. The `sys` Module

The `sys` module provides access to information about the Python interpreter and the environment in which your script runs.

```python
import sys
```

## 1.1 `sys.argv` — command-line arguments

`sys.argv` is a **list**, not a function. It contains the script name/path followed by arguments supplied on the command line.

Suppose `backup.py` contains:

```python
import sys

print(sys.argv)
```

Run:

```bash
py backup.py report.txt 3
```

Possible output:

```text
['backup.py', 'report.txt', '3']
```

- `sys.argv[0]`: script name or path.
- `sys.argv[1]`: first argument after the script name (`report.txt`).
- `sys.argv[2]`: second argument (`3`).
- `len(sys.argv)`: number of entries, including the script name.

Arguments arrive as strings. Convert them when necessary:

```python
count = int(sys.argv[2])
```

Always check that an argument exists before indexing it:

```python
import sys

if len(sys.argv) < 2:
    print("Usage: py backup.py <filename>")
    sys.exit(2)

filename = sys.argv[1]
print(f"File requested: {filename}")
```

For larger command-line programs, prefer `argparse`, which supports named options, types, defaults, and helpful error messages.

## 1.2 `sys.exit()` — request program termination

`sys.exit()` raises `SystemExit`, normally causing the Python program to stop.

```python
import sys

print("Starting")
sys.exit(0)
print("This line normally will not run")
```

Common exit-code conventions:

- `0`: success.
- Nonzero: failure.
- `1`: often used for a general error.
- `2`: often used for command-line usage errors, including by `argparse`.

These are conventions, not universal meanings for every possible exit code.

```python
import sys

filename = "report.txt"

if not filename:
    print("A filename is required.")
    sys.exit(1)
```

Difference between control-flow statements:

- `return`: leave the current function.
- `break`: leave the current loop.
- `continue`: skip to the next loop iteration.
- `sys.exit()`: request termination of the Python program.

Use `sys.exit()` mainly at the boundary of a command-line program. Inside reusable functions, returning a result or raising an appropriate exception is often better.

Check the last exit code:

Windows Command Prompt:

```bat
echo %ERRORLEVEL%
```

Linux/WSL:

```bash
echo $?
```

## 1.3 `sys.exception()` — currently handled exception

Available in Python 3.11+. Inside an `except` block, it returns the exception currently being handled.

```python
import sys

try:
    result = 10 / 0
except ZeroDivisionError:
    error = sys.exception()
    print(type(error).__name__)
    print(error)
```

Possible output:

```text
ZeroDivisionError
division by zero
```

For many everyday cases, this is simpler:

```python
try:
    result = 10 / 0
except ZeroDivisionError as error:
    print(error)
```

Use `sys.exception()` when you specifically need to retrieve the currently handled exception without binding it with `except ... as error`.

## 1.4 `sys.exc_info()` — exception type, value, and traceback

Inside an exception handler, `sys.exc_info()` returns a tuple of three items:

1. Exception type.
2. Exception instance (value).
3. Traceback object.

```python
import sys

try:
    int("hello")
except ValueError:
    error_type, error_value, traceback = sys.exc_info()

    print("Type:", error_type.__name__)
    print("Message:", error_value)
    print("Traceback available:", traceback is not None)
```

Use this when you need all three components, especially for diagnostic or logging code. For ordinary handling, `except SomeError as error` is generally clearer.

## 1.5 Other useful `sys` attributes and functions

```python
import sys

print(sys.version)       # Python version string
print(sys.platform)      # Platform identifier, e.g. 'win32' or 'linux'
print(sys.executable)    # Path to the Python interpreter running this script
print(sys.path)          # Directories Python searches for imports
print(sys.argv)          # Command-line arguments
print(sys.getsizeof([1, 2, 3]))  # Shallow size of this list object in bytes
```

`sys.getsizeof()` reports the shallow size of an object, not the complete memory used by all objects it refers to.

Standard streams:

```python
import sys

print("Normal output", file=sys.stdout)
print("Error/diagnostic output", file=sys.stderr)
```

- `sys.stdin`: standard input.
- `sys.stdout`: standard output.
- `sys.stderr`: standard error output.

---

# 2. The `time` Module

Import it with:

```python
import time
```

## 2.1 `time.sleep(seconds)` — pause execution

```python
import time

print("Starting")
time.sleep(2)
print("About two seconds later")
```

`time.sleep(0.5)` requests a half-second pause. The actual pause may be longer because of operating-system scheduling.

Useful for simple polling, delays, and demonstrations. It does **not** detect file changes by itself.

## 2.2 `time.time()` — Unix timestamp

Returns the current time as seconds since the Unix epoch (1970-01-01 00:00:00 UTC), usually as a floating-point number.

```python
import time

print(time.time())
```

Use it when you need a Unix timestamp. For measuring durations, prefer `time.perf_counter()` or `time.monotonic()`.

## 2.3 `time.perf_counter()` — measure duration

```python
import time

start = time.perf_counter()
total = sum(range(1_000_000))
elapsed = time.perf_counter() - start

print(total)
print(f"Elapsed: {elapsed:.6f} seconds")
```

Use this for benchmarking code or measuring how long a backup operation takes. The clock is high-resolution and intended for elapsed-time measurement.

## 2.4 `time.monotonic()` — elapsed time that is not affected by wall-clock adjustments

```python
import time

start = time.monotonic()
time.sleep(1)
elapsed = time.monotonic() - start

print(f"Elapsed: {elapsed:.2f} seconds")
```

Useful for timeouts, deadlines, and measuring durations. Its absolute value has no useful calendar meaning; compare differences between readings.

## 2.5 `time.process_time()` — CPU time

Measures CPU time consumed by the current process. It excludes time spent sleeping.

```python
import time

start = time.process_time()
sum(range(2_000_000))
cpu_seconds = time.process_time() - start

print(f"CPU time: {cpu_seconds:.6f} seconds")
```

Use it to estimate CPU work separately from waiting time.

## 2.6 `time.localtime()` and `time.gmtime()`

Both convert a timestamp into a `struct_time`.

```python
import time

print(time.localtime())  # Local system time
print(time.gmtime())     # UTC
```

Access individual fields:

```python
now = time.localtime()
print(now.tm_year)
print(now.tm_mon)
print(now.tm_mday)
print(now.tm_hour)
print(now.tm_min)
print(now.tm_sec)
```

## 2.7 `time.strftime()` — format time into text

```python
import time

stamp = time.strftime("%Y%m%d_%H%M%S")
print(stamp)  # Example: 20261027_162630
```

Useful directives:

| Directive | Meaning | Example |
|---|---|---|
| `%Y` | Four-digit year | `2026` |
| `%y` | Two-digit year | `26` |
| `%m` | Month number | `10` |
| `%d` | Day of month | `27` |
| `%H` | Hour, 24-hour clock | `16` |
| `%I` | Hour, 12-hour clock | `04` |
| `%M` | Minute | `26` |
| `%S` | Second | `30` |
| `%p` | AM/PM marker | `PM` |
| `%A` | Full weekday | `Tuesday` |
| `%B` | Full month | `October` |

## 2.8 `time.strptime()` — parse a formatted string

```python
import time

date_text = "2026-10-27"
parsed = time.strptime(date_text, "%Y-%m-%d")

print(parsed.tm_year, parsed.tm_mon, parsed.tm_mday)
```

- `strftime`: time information → formatted string.
- `strptime`: formatted string → parsed time fields.

For richer date arithmetic, time zones, or formatting, also learn `datetime`.

## 2.9 `time.ctime()`

Produces a human-readable local-time string:

```python
import time

print(time.ctime())
print(time.ctime(time.time()))
```

It is convenient for display, but `strftime()` gives more control over filenames and log formats.

## 2.10 Which clock should I use?

| Goal | Recommended function |
|---|---|
| Get current Unix timestamp | `time.time()` |
| Measure duration / benchmark | `time.perf_counter()` |
| Timeouts and elapsed time | `time.monotonic()` |
| Measure CPU time | `time.process_time()` |
| Pause execution | `time.sleep()` |
| Format local time | `time.strftime()` |
| Read local/UTC time fields | `time.localtime()` / `time.gmtime()` |

---

# 3. Processes in Python

## 3.1 What is a process?

A **process** is a running instance of a program. It has its own process ID (PID), memory space, resources, and execution state.

Examples:
- A running Python script.
- A browser process.
- A text editor process.
- A separate worker launched by a Python program.

A **thread** is an execution path within a process. Threads in the same process share memory. Separate processes normally have separate memory spaces.

## 3.2 The `subprocess` module

Use `subprocess` when your Python program needs to launch another program or command, wait for it, or capture its output.

```python
import subprocess
import sys

result = subprocess.run(
    [sys.executable, "--version"],
    capture_output=True,
    text=True,
    check=True,
)

print(result.stdout.strip())
```

Why this is a good pattern:
- `sys.executable` launches the same Python interpreter running your script.
- Passing a list of arguments avoids manually building a shell command.
- `capture_output=True` captures stdout and stderr.
- `text=True` returns decoded strings instead of bytes.
- `check=True` raises `CalledProcessError` if the child process exits unsuccessfully.

For commands that may fail and whose failure you want to inspect:

```python
import subprocess
import sys

result = subprocess.run(
    [sys.executable, "-c", "print('Child process says hello')"],
    capture_output=True,
    text=True,
)

print("Return code:", result.returncode)
print("Output:", result.stdout)
print("Errors:", result.stderr)
```

A return code of `0` conventionally indicates success; nonzero commonly indicates failure. The exact meaning depends on the program that was launched.

### Avoid unsafe shell commands

Prefer:

```python
subprocess.run(["some_program", "argument"], check=True)
```

Be careful with:

```python
subprocess.run(user_supplied_text, shell=True)
```

If untrusted input is inserted into a shell command, it can cause command injection. Use argument lists and avoid `shell=True` unless you genuinely need shell features and have handled input safely.

## 3.3 Start a process without waiting immediately: `Popen`

Use `Popen` when you need more control over a running child process.

```python
import subprocess
import sys

process = subprocess.Popen([
    sys.executable,
    "-c",
    "import time; time.sleep(2); print('Finished')",
])

print("Child PID:", process.pid)

return_code = process.wait()
print("Exit code:", return_code)
```

- `Popen(...)` starts the child process.
- `process.pid` gives its process ID.
- `process.wait()` waits until it finishes and returns its exit code.

If you redirect output using pipes, remember to read it or use `communicate()`; otherwise a child process can block if a pipe buffer fills.

## 3.4 `multiprocessing` — run Python work in separate processes

Use `multiprocessing` for CPU-heavy work that can benefit from multiple processes, or when you want process-level isolation.

```python
from multiprocessing import Process
import os

def worker():
    print("Worker process PID:", os.getpid())

if __name__ == "__main__":
    process = Process(target=worker)

    process.start()
    process.join()

    print("Worker finished")
```

- `Process(...)` describes the work to run.
- `.start()` launches it.
- `.join()` waits for it to finish.

**Important:** Keep process creation inside `if __name__ == "__main__":` in scripts, particularly on Windows. The child process may import the main module again.

Processes have startup and communication overhead. Do not use them for every small task.

## 3.5 `os.getpid()` and `os.getppid()`

```python
import os

print("Current process:", os.getpid())
print("Parent process:", os.getppid())
```

- `getpid()` returns the current process ID.
- `getppid()` returns the parent process ID where supported.

Useful for debugging process-based programs and system-monitoring tools.

## 3.6 `threading` versus `multiprocessing`

| Tool | Main use | Memory |
|---|---|---|
| `threading` | Concurrent I/O work, such as waiting on files or networks | Threads share process memory |
| `multiprocessing` | CPU-heavy parallel work or process isolation | Separate processes normally have separate memory |
| `subprocess` | Launch an external program or command | Child process is separate |

For many network or file-waiting tasks, threads or asynchronous I/O can be appropriate. For CPU-heavy Python work, multiprocessing can help. Choose based on the actual workload.

---

# 4. Magic Methods (Dunder Methods)

## 4.1 What are magic methods?

Magic methods are special methods whose names begin and end with double underscores, for example:

```python
__init__
__str__
__repr__
__len__
__eq__
```

They are also called **dunder methods**, short for “double underscore.”

Python invokes many of these methods automatically in response to operations such as creating an object, printing it, adding values, comparing objects, or using `len()`.

You normally do not call them directly. Instead, you use the corresponding Python operation.

## 4.2 `__init__()` — initialize an object

```python
class User:
    def __init__(self, username):
        self.username = username

user = User("nzegge")
print(user.username)
```

When `User("nzegge")` creates an instance, Python calls `__init__()` to initialize it.

Technically, `__new__()` creates the instance; `__init__()` initializes the instance that was created.

Use `__init__()` to set initial attributes and establish a valid starting state.

## 4.3 `__str__()` — user-friendly display

```python
class FileBackup:
    def __init__(self, filename):
        self.filename = filename

    def __str__(self):
        return f"Backup for {self.filename}"

backup = FileBackup("report.txt")
print(backup)
```

Output:

```text
Backup for report.txt
```

Python calls `__str__()` when you use `str(backup)` or normally print the object.

Use it when you want a readable display for users.

## 4.4 `__repr__()` — developer-friendly representation

```python
class User:
    def __init__(self, username):
        self.username = username

    def __repr__(self):
        return f"User(username={self.username!r})"

user = User("nzegge")

print(repr(user))
```

Output:

```text
User(username='nzegge')
```

`__repr__()` should ideally provide an unambiguous representation useful for debugging. For some classes, it can resemble a valid Python expression that recreates an equivalent object.

If you define only `__repr__()`, Python can use it as a fallback when `__str__()` is absent.

## 4.5 `__len__()` — support `len(object)`

```python
class Playlist:
    def __init__(self, songs):
        self.songs = songs

    def __len__(self):
        return len(self.songs)

playlist = Playlist(["Song A", "Song B", "Song C"])

print(len(playlist))  # 3
```

Python calls `__len__()` when you use `len(playlist)`.

The method should return a non-negative integer.

Use it when your object represents a collection with a meaningful length.

## 4.6 `__eq__()` — define equality

```python
class Product:
    def __init__(self, product_id):
        self.product_id = product_id

    def __eq__(self, other):
        if not isinstance(other, Product):
            return NotImplemented
        return self.product_id == other.product_id

first = Product("P100")
second = Product("P100")
third = Product("P200")

print(first == second)  # True
print(first == third)  # False
```

Without a custom `__eq__()`, ordinary user-defined objects generally compare by identity rather than by their attributes.

Use it when two separate objects should count as equal according to your domain rules.

Returning `NotImplemented` for an unsupported comparison allows Python to try the other operand's comparison behavior or apply its normal fallback.

## 4.7 `__add__()` — define the `+` operator

```python
class Money:
    def __init__(self, amount):
        self.amount = amount

    def __add__(self, other):
        if not isinstance(other, Money):
            return NotImplemented
        return Money(self.amount + other.amount)

    def __repr__(self):
        return f"Money({self.amount})"

total = Money(10) + Money(5)

print(total)  # Money(15)
```

Python calls `__add__()` for an operation like `first + second`, subject to Python's operator-dispatch rules.

Use it when addition is natural and clearly defined for your type. Do not overload operators just to make code look clever.

## 4.8 `__getitem__()` — support indexing

```python
class TaskList:
    def __init__(self, tasks):
        self.tasks = tasks

    def __getitem__(self, index):
        return self.tasks[index]

tasks = TaskList(["Backup files", "Check disk", "Write log"])

print(tasks[0])  # Backup files
```

Python calls `__getitem__()` for expressions such as `tasks[0]`.

Use it when your object naturally behaves like a sequence or mapping. Implement additional sequence behavior only when it makes sense.

## 4.9 `__enter__()` and `__exit__()` — context managers

These methods enable the `with` statement.

```python
class ManagedMessage:
    def __enter__(self):
        print("Entering the context")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("Leaving the context")
        return False

with ManagedMessage():
    print("Inside the context")
```

Output:

```text
Entering the context
Inside the context
Leaving the context
```

Python calls `__enter__()` when entering the `with` block and `__exit__()` when leaving it, including when an exception occurs.

Returning `False` (or another false value) means an exception is not suppressed. Returning a true value suppresses an exception that occurred inside the block.

Use context managers for resources that need reliable cleanup, such as files, locks, and connections.

For example, ordinary file objects already implement the context-manager protocol:

```python
with open("notes.txt", "w", encoding="utf-8") as file:
    file.write("Hello")
```

The file is closed when the block exits.

## 4.10 `__call__()` — make an instance callable

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor

    def __call__(self, number):
        return number * self.factor

double = Multiplier(2)

print(double(5))  # 10
```

Python calls `__call__()` when you invoke the instance like a function.

Use this when an object needs to behave like a callable while maintaining its own state.

## 4.11 `__iter__()` and `__next__()` — iteration

An object that implements the iterator protocol can be used in a `for` loop. An iterator provides `__iter__()` and `__next__()`.

```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= 0:
            raise StopIteration

        value = self.current
        self.current -= 1
        return value

for number in Countdown(3):
    print(number)
```

Output:

```text
3
2
1
```

This example is its own iterator, so it is one-shot: after exhaustion, it does not restart automatically. For many custom iterables, it is better for `__iter__()` to return a separate iterator or for the method to yield values in a generator.

Use custom iteration when your object represents a stream or sequence that should be traversed with `for`.

## 4.12 `__contains__()` — support `in`

```python
class AllowedExtensions:
    def __init__(self, extensions):
        self.extensions = extensions

    def __contains__(self, extension):
        return extension in self.extensions

allowed = AllowedExtensions({".txt", ".pdf", ".png"})

print(".txt" in allowed)  # True
print(".exe" in allowed)  # False
```

Python can call `__contains__()` for the `in` operator.

Use it when membership has a meaningful custom definition. If your class is simply wrapping a normal collection, exposing or delegating to that collection may be enough.

## 4.13 `__enter__` / `__exit__` versus `try/finally`

These are two ways to guarantee cleanup.

```python
file = open("notes.txt", "w", encoding="utf-8")
try:
    file.write("Hello")
finally:
    file.close()
```

The cleaner equivalent is:

```python
with open("notes.txt", "w", encoding="utf-8") as file:
    file.write("Hello")
```

Prefer an existing context manager or `contextlib` when possible rather than writing custom `__enter__()` and `__exit__()` methods unnecessarily.

## 4.14 Other magic methods worth recognizing

| Method | Associated behavior |
|---|---|
| `__new__()` | Creates an instance |
| `__bool__()` | Truth-value testing, such as `if object:` |
| `__lt__()` | Less-than comparison (`<`) |
| `__le__()` | Less-than-or-equal (`<=`) |
| `__gt__()` | Greater-than comparison (`>`) |
| `__ge__()` | Greater-than-or-equal (`>=`) |
| `__sub__()` | Subtraction (`-`) |
| `__mul__()` | Multiplication (`*`) |
| `__truediv__()` | True division (`/`) |
| `__hash__()` | Hashing for hash-based collections |
| `__setitem__()` | Assignment by key/index, such as `obj[key] = value` |
| `__delitem__()` | Deletion by key/index |
| `__getattr__()` | Fallback for missing attributes |
| `__getattribute__()` | Intercepts normal attribute access; use carefully |
| `__setattr__()` | Intercepts attribute assignment; use carefully |
| `__delattr__()` | Intercepts attribute deletion |
| `__enter__()` | Enters a context manager |
| `__exit__()` | Exits a context manager |

### Best practices for magic methods

1. Implement a magic method only when the operation makes sense for your class.
2. Follow Python's expected behavior and return types.
3. Avoid surprising operator behavior.
4. Be cautious with `__getattribute__()` and `__setattr__()` because they can easily cause recursion or confusing behavior.
5. Avoid implementing `__hash__()` carelessly: objects that compare equal should have equal hashes. Mutable objects whose equality-relevant attributes can change are usually not suitable as hash keys.
6. Use `__repr__()` to make debugging easier.

---

# 5. Function Decorators

## 5.1 What is a decorator?

A decorator is a callable that takes another function or class and returns a replacement or a wrapped version of it.

A **function decorator** commonly adds behavior before or after a function runs without requiring you to rewrite the original function's logic.

The `@decorator_name` syntax is convenient shorthand for applying a decorator.

## 5.2 First understand functions as values

In Python, a function can be stored in a variable or passed to another function.

```python
def greet():
    print("Hello")

another_name = greet
another_name()
```

Output:

```text
Hello
```

Notice that `greet` is assigned without parentheses. Writing `greet()` would call it immediately.

## 5.3 A simple function decorator

```python
def my_decorator(function):
    def wrapper():
        print("Before the function")
        function()
        print("After the function")

    return wrapper

@my_decorator
def greet():
    print("Hello")

greet()
```

Output:

```text
Before the function
Hello
After the function
```

The syntax:

```python
@my_decorator
def greet():
    print("Hello")
```

is essentially equivalent to:

```python
def greet():
    print("Hello")

greet = my_decorator(greet)
```

Step by step:

1. Python creates the original `greet` function.
2. Python passes it to `my_decorator`.
3. The decorator returns `wrapper`.
4. The name `greet` now refers to the wrapper.
5. Calling `greet()` runs the wrapper, which calls the original function.

## 5.4 Decorators that accept arguments

A useful general-purpose decorator should support functions with positional arguments and keyword arguments.

```python
from functools import wraps

def announce(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        print(f"Calling {function.__name__}")
        result = function(*args, **kwargs)
        print(f"Finished {function.__name__}")
        return result

    return wrapper

@announce
def add(a, b):
    return a + b

answer = add(3, 4)
print("Answer:", answer)
```

Output:

```text
Calling add
Finished add
Answer: 7
```

Understand these parts:

- `*args` collects positional arguments into a tuple.
- `**kwargs` collects keyword arguments into a dictionary.
- `function(*args, **kwargs)` passes them to the original function.
- `return result` preserves the original function's return value.
- `@wraps(function)` preserves useful metadata such as the original function's name and docstring.

Without returning `result`, the wrapper would return `None` even if the original function returned a value.

## 5.5 A timing decorator

This is a practical decorator for your projects.

```python
import time
from functools import wraps

def measure_time(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()

        try:
            return function(*args, **kwargs)
        finally:
            elapsed = time.perf_counter() - start
            print(f"{function.__name__} took {elapsed:.6f} seconds")

    return wrapper

@measure_time
def calculate():
    return sum(range(1_000_000))

result = calculate()
print(result)
```

Why use `finally`? It measures and reports elapsed time even if the wrapped function raises an exception. The exception still propagates because the `finally` block does not suppress it.

Use a timing decorator when you want to measure several functions consistently without duplicating timing code in each function.

## 5.6 A logging decorator

```python
from functools import wraps

def log_call(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        print(f"Running {function.__name__}")
        return function(*args, **kwargs)

    return wrapper

@log_call
def create_backup(filename):
    print(f"Creating backup for {filename}")
    return True

success = create_backup("report.txt")
print("Success:", success)
```

Use this pattern for simple diagnostics. For production applications, Python's `logging` module is usually better than scattering `print()` statements throughout your code.

**Security note:** Do not blindly log passwords, tokens, secret keys, or sensitive file contents. Logging function arguments can accidentally expose private data.

## 5.7 A decorator that accepts its own arguments

Sometimes you want to configure a decorator, such as setting a label or threshold.

That requires an additional outer function:

```python
from functools import wraps

def repeat(times):
    def decorator(function):
        @wraps(function)
        def wrapper(*args, **kwargs):
            result = None

            for _ in range(times):
                result = function(*args, **kwargs)

            return result

        return wrapper

    return decorator

@repeat(3)
def greet(name):
    print(f"Hello, {name}")

greet("Nzegge")
```

Output:

```text
Hello, Nzegge
Hello, Nzegge
Hello, Nzegge
```

There are three layers:

1. `repeat(3)` receives the decorator configuration.
2. `decorator(function)` receives the function.
3. `wrapper(*args, **kwargs)` runs when the decorated function is called.

Be careful when repeating a function that changes files, sends emails, charges a payment method, or performs another side effect. Repeating it may cause duplicate actions. Retry decorators need explicit error-handling and retry rules.

## 5.8 When should you use a function decorator?

Good use cases include:

- Logging calls.
- Measuring execution time.
- Checking permissions.
- Validating shared preconditions.
- Caching expensive function results.
- Retrying operations when retrying is safe.
- Registering plugin or event-handler functions.

Avoid decorators when they obscure a simple operation or make program behavior hard to follow. A normal helper function may be clearer.

---

# 6. Class Decorators

## 6.1 What is a class decorator?

A class decorator is a callable that receives a class and returns a class or another suitable replacement.

For example:

```python
def add_label(cls):
    cls.label = "Managed class"
    return cls

@add_label
class Backup:
    pass

print(Backup.label)
```

Output:

```text
Managed class
```

The syntax:

```python
@add_label
class Backup:
    pass
```

is essentially equivalent to:

```python
class Backup:
    pass

Backup = add_label(Backup)
```

In this example, the decorator adds an attribute to the class and returns the same class.

## 6.2 A class decorator that adds a method

```python
def add_description(cls):
    def describe(self):
        return f"This is a {type(self).__name__} object"

    cls.describe = describe
    return cls

@add_description
class FileBackup:
    def __init__(self, filename):
        self.filename = filename

backup = FileBackup("report.txt")

print(backup.describe())
```

Output:

```text
This is a FileBackup object
```

The decorator defines a function and attaches it to the class. When Python retrieves it through an instance, it behaves as an instance method.

Use this technique when you deliberately want to add consistent behavior to multiple classes. For ordinary application code, explicit inheritance, composition, or a normal method may be easier to maintain.

## 6.3 A class decorator that registers classes

Decorators can maintain a registry of classes, which is useful in plugin systems.

```python
PLUGINS = {}

def register_plugin(cls):
    PLUGINS[cls.__name__] = cls
    return cls

@register_plugin
class FileBackup:
    pass

@register_plugin
class SystemMonitor:
    pass

print(PLUGINS.keys())
```

Output resembles:

```text
dict_keys(['FileBackup', 'SystemMonitor'])
```

A registry lets a program discover registered implementations without manually maintaining every class reference.

For larger projects, choose stable, intentional registry keys and consider how duplicate names should be handled.

## 6.4 Class decorator versus function decorator

| Feature | Function decorator | Class decorator |
|---|---|---|
| Receives | Usually a function | A class |
| Common purpose | Wrap function behavior | Modify, register, or replace a class |
| Typical use | Logging or timing | Plugin registration or class configuration |
| `@` syntax | Above a function definition | Above a class definition |

A class decorator is not the same as a decorator that turns a method into a class method.

For example:

```python
class Example:
    @classmethod
    def make(cls):
        return cls()
```

Here, `@classmethod` is a built-in **method decorator**. It makes `make` receive the class as its first argument. It does not decorate the class itself.

Similarly:

```python
class Example:
    @staticmethod
    def helper():
        return "No automatic self or cls argument"
```

`@staticmethod` creates a method that does not automatically receive the instance or class as its first argument.

## 6.5 Built-in decorators you should recognize

### `@property`

Lets you access a method like an attribute:

```python
class FileInfo:
    def __init__(self, filename):
        self.filename = filename

    @property
    def extension(self):
        return self.filename.rsplit(".", 1)[-1]

file_info = FileInfo("report.txt")
print(file_info.extension)
```

Output:

```text
txt
```

Use `@property` when a value is derived from an object or when attribute-style access is a natural interface. Avoid using it for expensive operations that users would reasonably expect to be cheap.

### `@classmethod`

Receives the class as `cls`. Commonly used for alternative constructors:

```python
class User:
    def __init__(self, username):
        self.username = username

    @classmethod
    def from_email(cls, email):
        username = email.split("@")[0]
        return cls(username)

user = User.from_email("nzegge@example.com")
print(user.username)
```

### `@staticmethod`

Groups a helper function logically inside a class without automatic `self` or `cls`:

```python
class FileTools:
    @staticmethod
    def is_text_file(filename):
        return filename.endswith(".txt")

print(FileTools.is_text_file("notes.txt"))
```

Use a module-level function instead if the helper is not meaningfully related to the class.

### `@dataclass`

The `dataclasses` module can generate common methods for data-focused classes:

```python
from dataclasses import dataclass

@dataclass
class BackupRecord:
    source: str
    destination: str
    success: bool = True

record = BackupRecord("report.txt", "backups/report.txt")

print(record)
```

A dataclass can automatically provide methods such as `__init__()` and `__repr__()`, and usually equality comparison when configured with the default options. This avoids writing repetitive boilerplate.

---

# 7. Choosing the Right Tool

## 7.1 `sys`, `time`, and process tools

| Need | Tool |
|---|---|
| Read command-line arguments | `sys.argv` or `argparse` |
| Stop a CLI program with a status code | `sys.exit()` |
| Inspect an exception | `sys.exception()` / `sys.exc_info()` |
| Pause the current thread | `time.sleep()` |
| Measure elapsed execution time | `time.perf_counter()` |
| Implement a timeout | `time.monotonic()` |
| Launch an external program | `subprocess.run()` or `subprocess.Popen()` |
| Run Python work in a separate process | `multiprocessing` |
| Get current process ID | `os.getpid()` |

## 7.2 Magic methods

Use magic methods when you want your custom objects to behave naturally with Python operations.

- Want `print(obj)` to show useful information? Implement `__str__()`.
- Want `len(obj)` to work? Implement `__len__()`.
- Want `obj1 == obj2` to compare meaningful values? Implement `__eq__()`.
- Want `obj[key]` to work? Consider `__getitem__()`.
- Want an object to be usable in a `with` statement? Implement the context-manager protocol or use `contextlib`.
- Want `obj()` to call an object? Implement `__call__()`.

Do not implement all magic methods automatically. Implement only those that match your class's purpose.

## 7.3 Function decorators

Use a function decorator when you want to apply reusable behavior around function calls.

Examples:
- Timing.
- Logging.
- Authorization checks.
- Caching.
- Validation.
- Safe retry policies.

## 7.4 Class decorators

Use a class decorator when you want to modify, configure, or register classes at definition time.

Examples:
- Plugin registration.
- Adding class metadata.
- Applying class-level configuration.
- Integrating with frameworks that expect class decoration.

Before writing a custom class decorator, consider whether inheritance, composition, a metaclass, or a normal helper would be clearer.

---

# 8. Practice Projects

## Project 1: Command-line file information tool

**Goal:** Practise `sys.argv`, `sys.exit()`, `pathlib`, and exception handling.

Requirements:
1. Accept a filename from the command line.
2. Print a usage message if no filename is supplied.
3. Check whether the path exists.
4. Print its name, suffix, and size if it is a file.
5. Return an appropriate exit code for success or failure.

Hint:

```python
import sys
from pathlib import Path

if len(sys.argv) < 2:
    print("Usage: py file_info.py <path>")
    sys.exit(2)

path = Path(sys.argv[1])

if not path.is_file():
    print("Error: Not a file or file does not exist.")
    sys.exit(1)

print("Name:", path.name)
print("Extension:", path.suffix)
print("Size:", path.stat().st_size, "bytes")
```

## Project 2: Function execution timer

**Goal:** Practise `time.perf_counter()` and a function decorator.

Requirements:
1. Create a `measure_time` decorator.
2. Use `functools.wraps`.
3. Measure the duration of the decorated function.
4. Preserve its return value.
5. Ensure the timing is reported even if the function raises an exception.

Hint: Use `try/finally` in the wrapper.

## Project 3: Backup record class

**Goal:** Practise classes and magic methods.

Requirements:
1. Create a `BackupRecord` class.
2. Store the source path and destination path.
3. Implement `__str__()` for a readable message.
4. Implement `__repr__()` for debugging.
5. Optionally implement `__eq__()` if you can define clear equality rules.

## Project 4: Process launcher

**Goal:** Practise `subprocess`.

Requirements:
1. Launch a small child Python program using `sys.executable`.
2. Capture its output.
3. Display the return code.
4. Handle `subprocess.CalledProcessError` if you use `check=True`.
5. Avoid passing untrusted input into a shell command.

## Project 5: Plugin registry

**Goal:** Practise class decorators.

Requirements:
1. Create a `PLUGINS` dictionary.
2. Write a `register_plugin` class decorator.
3. Register two classes.
4. Print the registered names.
5. Instantiate a registered class by looking it up in the registry.

---

# Final Best-Practice Checklist

- Check `sys.argv` length before reading an argument.
- Prefer `argparse` for command-line interfaces with several options.
- Use `sys.exit()` mainly at the outer boundary of command-line scripts.
- Use `except SpecificError as error` for ordinary error handling.
- Use `time.perf_counter()` to measure durations.
- Use `time.monotonic()` for deadlines and timeouts.
- Use `subprocess.run()` with an argument list for most simple child-process tasks.
- Protect multiprocessing entry points with `if __name__ == "__main__":`.
- Use magic methods only when their behavior is natural and unsurprising.
- Use `functools.wraps` in function decorators that wrap another function.
- Make wrappers accept `*args` and `**kwargs` when the decorator should support general callables.
- Return the wrapped function's result unless intentionally changing its behavior.
- Do not accidentally suppress exceptions in a decorator or context manager.
- Avoid logging secrets and sensitive arguments.
- Prefer the simplest solution that makes the code understandable.

## Official Python documentation

- `sys`: https://docs.python.org/3/library/sys.html
- `time`: https://docs.python.org/3/library/time.html
- `subprocess`: https://docs.python.org/3/library/subprocess.html
- `multiprocessing`: https://docs.python.org/3/library/multiprocessing.html
- `threading`: https://docs.python.org/3/library/threading.html
- Data model and magic methods: https://docs.python.org/3/reference/datamodel.html
- `functools` and `wraps`: https://docs.python.org/3/library/functools.html
- `contextlib`: https://docs.python.org/3/library/contextlib.html
- `dataclasses`: https://docs.python.org/3/library/dataclasses.html
