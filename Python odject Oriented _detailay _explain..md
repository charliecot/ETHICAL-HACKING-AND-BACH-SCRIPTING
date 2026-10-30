# Python Classes, Decorators, Dunder Methods, and Type Annotations

## Table of Contents

1. Variables and type annotations (`variable: str`)
2. Function parameters and return types (`->`)
3. Understanding `__init__()`
4. Understanding `__call__()`
5. What are dunder methods?
6. Understanding decorators
7. Creating a decorator using a function
8. Creating a decorator using a class
9. Understanding `@` syntax
10. `@staticmethod`, `@classmethod`, and `@property`
11. Understanding `self` and `cls`
12. Type aliases and advanced type annotations
13. Practical examples
14. Common mistakes
15. Best practices and cheat sheet

---

# 1. Variables and Type Annotations

You mentioned that you used to write something like:

```python
name: str
```

This is called a **type annotation**.

Type annotations communicate the kind of value a variable is expected to contain.

## 1.1 Normal variable declaration

```python
name = "Nzegge"
age = 25
price = 19.99
is_active = True
```

Python determines the types from the assigned values.

```python
print(type(name))       # <class 'str'>
print(type(age))        # <class 'int'>
print(type(price))      # <class 'float'>
print(type(is_active))  # <class 'bool'>
```

## 1.2 Declaring variables with type annotations

```python
name: str = "Nzegge"
age: int = 25
price: float = 19.99
is_active: bool = True
```

Here is what each annotation means:

| Annotation | Expected type | Example |
|---|---|---|
| `str` | String | `"Hello"` |
| `int` | Integer | `25` |
| `float` | Floating-point number | `19.99` |
| `bool` | Boolean | `True` |
| `list` | List | `[1, 2, 3]` |
| `tuple` | Tuple | `(1, 2, 3)` |
| `dict` | Dictionary | `{"name": "John"}` |
| `set` | Set | `{1, 2, 3}` |
| `None` | Absence of a value | `None` |

**Important:** Python generally does not enforce variable annotations at runtime.

For example:

```python
age: int = 25

age = "Hello"

print(age)  # Hello
```

Python permits this assignment even though it conflicts with the annotation.

Static type checkers, such as mypy and Pyright, can detect this kind of problem.

## 1.3 Declaring a variable without assigning a value

```python
name: str
```

This tells a type checker that `name` is expected to be a string.

However, it does not assign a string to `name`.

Consequently, this will fail if you have not assigned a value:

```python
name: str

print(name)  # NameError
```

If you want to initialize the variable with an empty string, write:

```python
name: str = ""
```

If the variable is allowed to contain no value, you can use:

```python
name: str | None = None
```

This means that the variable can contain either a string or `None`.

Example:

```python
name: str | None = None

name = "Nzegge"

print(name)
```

The `|` symbol in `str | None` means that either type is acceptable.

This syntax is supported in Python 3.10 and newer.

---

# 2. Function Parameters and Return Types

You mentioned functions that have arrows.

You are probably referring to this:

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"
```

The arrow `->` introduces a **return type annotation**.

Let's break this down.

```python
def greet(name: str) -> str:
```

- `def` defines a function.
- `greet` is the function's name.
- `name: str` indicates that the parameter is expected to be a string.
- `-> str` indicates that the function is expected to return a string.
- `return` sends the result back to the caller.

Example:

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"


message: str = greet("Nzegge")

print(message)
```

Output:

```text
Hello, Nzegge!
```

## 2.1 Function that returns an integer

```python
def add(a: int, b: int) -> int:
    return a + b


result: int = add(10, 20)

print(result)  # 30
```

The annotations communicate that `a`, `b`, and the returned result are expected to be integers.

Python does not automatically enforce these expectations.

## 2.2 Function that returns a Boolean

```python
def is_adult(age: int) -> bool:
    return age >= 18


print(is_adult(25))  # True
print(is_adult(15))  # False
```

Here, `-> bool` means the function is expected to return `True` or `False`.

## 2.3 Function that returns nothing meaningful

```python
def display_message(message: str) -> None:
    print(message)


display_message("Learning Python")
```

`-> None` indicates that the function is not intended to return a meaningful value.

Python functions without an explicit `return` statement return `None`.

For example:

```python
def say_hello() -> None:
    print("Hello!")


result = say_hello()

print(result)  # None
```

## 2.4 Function that returns a list

```python
def get_numbers() -> list[int]:
    return [10, 20, 30]


numbers: list[int] = get_numbers()

print(numbers)
```

`list[int]` means a list whose elements are expected to be integers.

Python 3.9 and newer support this syntax.

## 2.5 Function that returns a dictionary

```python
def get_user() -> dict[str, str]:
    return {
        "name": "Nzegge",
        "country": "Cameroon"
    }


user: dict[str, str] = get_user()

print(user)
```

The annotation `dict[str, str]` means that the dictionary is expected to have string keys and string values.

For a dictionary with string keys and integer values:

```python
scores: dict[str, int] = {
    "Python": 90,
    "SQL": 85
}
```

## 2.6 Function that returns a tuple

```python
def get_coordinates() -> tuple[int, int]:
    return (10, 20)


coordinates: tuple[int, int] = get_coordinates()

print(coordinates)
```

The tuple is expected to contain two integers.

## 2.7 The difference between `:` and `->`

Consider:

```python
def multiply(a: int, b: int) -> int:
    result: int = a * b
    return result
```

There are three different annotations here:

```python
a: int
b: int
result: int
```

These annotate parameters and a variable.

Meanwhile:

```python
-> int
```

annotates the expected return type of the function.

**Remember:**

- `variable: str` — the variable is expected to be a string.
- `parameter: int` — the parameter is expected to be an integer.
- `-> bool` — the function is expected to return a Boolean.
- `-> None` — the function is not intended to return a meaningful value.

---

# 3. Understanding `__init__()`

Now let's move to classes and dunder methods.

A class is a blueprint for creating objects.

```python
class User:
    pass
```

We can create an object from this class:

```python
user = User()
```

But what if we want every user to have a name and an age?

This is where `__init__()` becomes useful.

## 3.1 Creating a class with `__init__()`

```python
class User:
    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = age


user = User("Nzegge", 25)

print(user.name)
print(user.age)
```

Output:

```text
Nzegge
25
```

Let's understand what happens.

### Step 1: Define the class

```python
class User:
```

This creates a class named `User`.

### Step 2: Define `__init__()`

```python
def __init__(self, name: str, age: int) -> None:
```

`__init__()` initializes an object's attributes.

The parameters are:

- `self`: refers to the particular instance.
- `name`: receives the name supplied when creating the object.
- `age`: receives the age supplied when creating the object.
- `-> None`: indicates that `__init__()` should not return a value.

### Step 3: Store the attributes

```python
self.name = name
self.age = age
```

The left-hand side creates or assigns instance attributes.

The right-hand side refers to the incoming parameter values.

For example:

```python
user = User("Nzegge", 25)
```

Conceptually, Python creates an instance and initializes it using the supplied arguments.

The resulting attributes are:

```python
user.name  # "Nzegge"
user.age   # 25
```

### Important distinction

`__init__()` is not the method that actually creates the instance.

Python normally creates the instance through `__new__()` and then initializes it through `__init__()`.

You rarely need to implement `__new__()` in everyday Python programming.

## 3.2 Why use `self`?

Consider:

```python
class User:
    def __init__(self, name: str) -> None:
        self.name = name
```

Now create two objects:

```python
user1 = User("Nzegge")
user2 = User("Charles")

print(user1.name)  # Nzegge
print(user2.name)  # Charles
```

Each object has its own `name` attribute.

`self` allows the method to work with the particular object on which it is called.

The name `self` is a convention, not a reserved keyword, but you should use it.

## 3.3 Adding a regular method

```python
class User:
    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = age

    def introduce(self) -> str:
        return f"My name is {self.name} and I am {self.age}."


user = User("Nzegge", 25)

print(user.introduce())
```

Output:

```text
My name is Nzegge and I am 25.
```

Notice that `introduce()` uses `self.name` and `self.age`.

These attributes were initialized by `__init__()`.

---

# 4. Understanding `__call__()`

This is one of the most important concepts for creating a decorator using a class.

The special method `__call__()` allows an instance of a class to be called like a function.

## 4.1 A normal class instance

```python
class Greeter:
    def greet(self, name: str) -> str:
        return f"Hello, {name}!"


greeter = Greeter()

print(greeter.greet("Nzegge"))
```

Output:

```text
Hello, Nzegge!
```

Here, we must call the `greet()` method.

What if we want to write this instead?

```python
greeter("Nzegge")
```

By default, a normal `Greeter` instance cannot be called this way.

We can change that by defining `__call__()`.

## 4.2 Making an instance callable

```python
class Greeter:
    def __call__(self, name: str) -> str:
        return f"Hello, {name}!"


greeter = Greeter()

print(greeter("Nzegge"))
```

Output:

```text
Hello, Nzegge!
```

The object behaves like a function because the class implements `__call__()`.

The following two ideas are different:

```python
greeter = Greeter()
```

This creates an instance.

```python
greeter("Nzegge")
```

This calls the instance, causing Python to invoke its `__call__()` method.

Conceptually, the second operation corresponds to:

```python
greeter.__call__("Nzegge")
```

For normal code, prefer calling the instance directly.

## 4.3 Using `__init__()` and `__call__()` together

Now let's use both methods.

```python
class Multiplier:
    def __init__(self, factor: int) -> None:
        self.factor = factor

    def __call__(self, number: int) -> int:
        return number * self.factor


double = Multiplier(2)
triple = Multiplier(3)

print(double(10))  # 20
print(triple(10))  # 30
```

Let's break down the process.

When we write:

```python
double = Multiplier(2)
```

Python creates an instance and initializes it with:

```python
self.factor = 2
```

When we write:

```python
double(10)
```

Python invokes:

```python
def __call__(self, number: int) -> int:
    return number * self.factor
```

The result is:

```python
10 * 2
```

Therefore, the output is `20`.

**Remember:**

- `__init__()` initializes an instance.
- `__call__()` makes an instance callable.

This combination is useful when creating class-based decorators.

---

# 5. What Are Dunder Methods?

Dunder means **double underscore**.

Dunder methods are special methods whose names begin and end with two underscores.

Examples:

```python
__init__
__call__
__str__
__repr__
__len__
__iter__
__next__
__enter__
__exit__
__eq__
__add__
```

Python uses these methods to implement special behavior for objects.

## 5.1 Common dunder methods

| Method | Purpose |
|---|---|
| `__init__()` | Initializes a new instance |
| `__new__()` | Creates an instance |
| `__call__()` | Allows an instance to be called |
| `__str__()` | Provides a user-friendly string representation |
| `__repr__()` | Provides a developer-oriented representation |
| `__len__()` | Defines the behavior of `len(object)` |
| `__iter__()` | Makes an object iterable or returns its iterator |
| `__next__()` | Retrieves the next item from an iterator |
| `__enter__()` | Enters a context manager |
| `__exit__()` | Exits a context manager |
| `__eq__()` | Defines equality comparisons |
| `__add__()` | Defines behavior for the `+` operator |
| `__getitem__()` | Defines indexed or keyed access |
| `__setitem__()` | Defines indexed or keyed assignment |
| `__contains__()` | Defines membership testing with `in` |

Let's examine a few.

## 5.2 `__str__()`

```python
class User:
    def __init__(self, name: str) -> None:
        self.name = name

    def __str__(self) -> str:
        return f"User: {self.name}"


user = User("Nzegge")

print(user)
```

Output:

```text
User: Nzegge
```

Python calls `__str__()` when `str(user)` or `print(user)` needs a string representation.

## 5.3 `__repr__()`

```python
class User:
    def __init__(self, name: str) -> None:
        self.name = name

    def __repr__(self) -> str:
        return f"User(name={self.name!r})"


user = User("Nzegge")

print(repr(user))
```

Output:

```text
User(name='Nzegge')
```

`__repr__()` is primarily intended to provide an informative representation useful to developers.

## 5.4 `__len__()`

```python
class Team:
    def __init__(self, members: list[str]) -> None:
        self.members = members

    def __len__(self) -> int:
        return len(self.members)


team = Team(["Nzegge", "Charles", "John"])

print(len(team))  # 3
```

When you call:

```python
len(team)
```

Python uses the class's `__len__()` implementation.

## 5.5 `__enter__()` and `__exit__()`

These methods are used to implement context managers.

A context manager works with the `with` statement.

For example:

```python
with open("notes.txt", "w", encoding="utf-8") as file:
    file.write("Learning Python")
```

The file object manages opening and closing the file.

You do not need to implement your own context manager for ordinary file operations, but understanding these methods will help you when creating more advanced reusable tools.

---

# 6. Understanding Decorators

A decorator is a callable that takes another callable and returns a callable, usually adding or changing behavior.

Functions are objects in Python, so they can be passed to other functions and returned from them.

## 6.1 A normal function

```python
def greet() -> None:
    print("Hello!")


greet()
```

Output:

```text
Hello!
```

Now imagine that we want to print a message before and after `greet()` runs.

We could rewrite the original function, but a decorator allows us to add behavior without changing its body.

## 6.2 The basic idea

A decorator can wrap another function.

```python
def greet() -> None:
    print("Hello!")


def decorator(func):
    def wrapper():
        print("Before the function")
        func()
        print("After the function")

    return wrapper


greet = decorator(greet)

greet()
```

Output:

```text
Before the function
Hello!
After the function
```

Let's understand the process.

### Step 1: Define the original function

```python
def greet() -> None:
    print("Hello!")
```

At this point, `greet` refers to the original function.

### Step 2: Pass the function into the decorator

```python
greet = decorator(greet)
```

The original function is passed to `decorator()` as the argument `func`.

Inside the decorator, `wrapper()` is defined.

### Step 3: Return the wrapper

```python
return wrapper
```

The decorator returns the wrapper function.

The name `greet` is then reassigned to refer to the wrapper.

### Step 4: Call the decorated function

```python
greet()
```

This now executes the wrapper.

The wrapper prints a message, calls the original function through `func()`, and prints another message.

**The important idea:** A decorator lets you add behavior around a function.

---

# 7. Using the `@` Decorator Syntax

Python provides convenient syntax for applying decorators.

Instead of writing:

```python
def greet() -> None:
    print("Hello!")


greet = decorator(greet)
```

You can write:

```python
@decorator
def greet() -> None:
    print("Hello!")
```

These forms are equivalent:

```python
@decorator
def greet():
    print("Hello!")
```

And:

```python
def greet():
    print("Hello!")


greet = decorator(greet)
```

Python evaluates the decorator and assigns the resulting callable to the function's name.

The `@` symbol is not unique to class-based decorators. It is also used with ordinary function decorators.

---

# 8. Creating a Decorator Using a Class

Now we arrive at the concept you originally wanted to remember.

A class-based decorator commonly uses:

- `__init__()` to store the original function and any configuration.
- `__call__()` to execute the decorated behavior.

Here is a complete example.

## 8.1 A simple class-based decorator

```python
from collections.abc import Callable
from typing import Any


class MyDecorator:
    def __init__(self, func: Callable[..., Any]) -> None:
        self.func = func

    def __call__(self, *args: Any, **kwargs: Any) -> Any:
        print("Before the function")

        result = self.func(*args, **kwargs)

        print("After the function")

        return result


@MyDecorator
def greet(name: str) -> str:
    print(f"Hello, {name}!")
    return f"Greeted {name}"


message = greet("Nzegge")

print(message)
```

Output:

```text
Before the function
Hello, Nzegge!
After the function
Greeted Nzegge
```

Let's understand every important part.

### Step 1: Define the decorator class

```python
class MyDecorator:
```

This defines the class that will be used as a decorator.

### Step 2: Define `__init__()`

```python
def __init__(self, func: Callable[..., Any]) -> None:
    self.func = func
```

When Python applies `@MyDecorator` to `greet`, it effectively performs:

```python
greet = MyDecorator(greet)
```

Therefore, the original function is passed into the decorator instance.

The line:

```python
self.func = func
```

stores the original function in the instance.

You can think of `self.func` as a reference to the original `greet()` function.

### Step 3: Define `__call__()`

```python
def __call__(self, *args: Any, **kwargs: Any) -> Any:
```

Because `greet` now refers to an instance of `MyDecorator`, calling:

```python
greet("Nzegge")
```

invokes the decorator instance's `__call__()` method.

The parameters:

```python
*args
```

collect positional arguments.

The parameters:

```python
**kwargs
```

collect keyword arguments.

This allows the decorator to work with different function signatures.

For example, all these calls can pass through the wrapper:

```python
greet("Nzegge")
```

```python
greet(name="Nzegge")
```

### Step 4: Execute the original function

```python
result = self.func(*args, **kwargs)
```

This invokes the original function with the arguments supplied by the caller.

### Step 5: Return the result

```python
return result
```

This is essential when the original function produces a value.

Without returning `result`, the decorated call would normally return `None` instead of the original function's result.

## 8.2 Understanding the type annotations

The example uses:

```python
from collections.abc import Callable
from typing import Any
```

Let's explain these types.

### `Callable`

`Callable` describes something that can be called, such as a function.

```python
func: Callable[..., Any]
```

This says that `func` is expected to be callable.

The ellipsis `...` indicates that the parameter types are not specified here.

`Any` indicates that the return value can be of any type.

### `Any`

```python
def __call__(self, *args: Any, **kwargs: Any) -> Any:
```

This tells a type checker that the arguments and returned value can have varying types.

It is flexible, but it provides less precise type checking.

For more advanced code, you can use `ParamSpec` and `TypeVar` to preserve the original function's parameter and return types.

We'll examine those later.

---

# 9. Class-Based Decorator With Configuration

Sometimes you want to configure the decorator.

For example:

```python
@Repeat(times=3)
def greet() -> None:
    print("Hello!")
```

Here, `times=3` tells the decorator how many times to execute the function.

Let's implement it.

```python
from collections.abc import Callable
from typing import Any


class Repeat:
    def __init__(self, times: int) -> None:
        self.times = times

    def __call__(
        self,
        func: Callable[..., Any]
    ) -> Callable[..., Any]:

        def wrapper(*args: Any, **kwargs: Any) -> Any:
            result = None

            for _ in range(self.times):
                result = func(*args, **kwargs)

            return result

        return wrapper


@Repeat(times=3)
def greet(name: str) -> None:
    print(f"Hello, {name}!")


greet("Nzegge")
```

Output:

```text
Hello, Nzegge!
Hello, Nzegge!
Hello, Nzegge!
```

## 9.1 Why is this different?

Compare the earlier decorator:

```python
@MyDecorator
def greet():
    ...
```

With the configured decorator:

```python
@Repeat(times=3)
def greet():
    ...
```

The first form passes the function directly to the decorator class.

The second form first creates a configured decorator instance.

The process is approximately:

```python
repeat_decorator = Repeat(times=3)

greet = repeat_decorator(greet)
```

The instance's `__call__()` method receives the original function and returns a wrapper.

**Important distinction:** In this configured example, `__init__()` receives `times`, not the original function. The original function is received later by `__call__()`.

This is a common source of confusion when learning class-based decorators.

## 9.2 The two common patterns

### Pattern A: The class directly decorates the function

```python
@MyDecorator
def greet():
    ...
```

Typical structure:

```python
class MyDecorator:
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        ...
```

### Pattern B: The decorator has configuration

```python
@Repeat(times=3)
def greet():
    ...
```

Typical structure:

```python
class Repeat:
    def __init__(self, times):
        self.times = times

    def __call__(self, func):
        def wrapper(*args, **kwargs):
            ...
        return wrapper
```

These patterns serve different purposes.

---

# 10. Understanding `@staticmethod`, `@classmethod`, and `@property`

You may have seen these decorators when reading Python classes.

They are built-in decorators, and each has a different purpose.

## 10.1 `@staticmethod`

A static method does not automatically receive an instance or class as its first argument.

```python
class Calculator:
    @staticmethod
    def add(a: int, b: int) -> int:
        return a + b


print(Calculator.add(10, 20))
```

Output:

```text
30
```

Use a static method when an operation logically belongs to the class but does not need to access instance or class state.

You can also call it through an instance:

```python
calculator = Calculator()

print(calculator.add(5, 5))
```

## 10.2 `@classmethod`

A class method receives the class as its first argument, conventionally named `cls`.

```python
class User:
    user_count: int = 0

    def __init__(self, name: str) -> None:
        self.name = name
        type(self).user_count += 1

    @classmethod
    def get_user_count(cls) -> int:
        return cls.user_count


user1 = User("Nzegge")
user2 = User("Charles")

print(User.get_user_count())  # 2
```

Use `@classmethod` when the method needs to work with class-level information or provide an alternative way to construct instances.

A common use is an alternative constructor:

```python
class User:
    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = age

    @classmethod
    def from_string(cls, data: str) -> "User":
        name, age = data.split(",")

        return cls(name, int(age))


user = User.from_string("Nzegge,25")

print(user.name)
print(user.age)
```

Output:

```text
Nzegge
25
```

Here, `cls(...)` creates an instance of the class.

## 10.3 `@property`

A property lets you access a method using attribute syntax.

```python
class Rectangle:
    def __init__(self, width: float, height: float) -> None:
        self.width = width
        self.height = height

    @property
    def area(self) -> float:
        return self.width * self.height


rectangle = Rectangle(10, 5)

print(rectangle.area)
```

Output:

```text
50
```

Notice:

```python
rectangle.area
```

There are no parentheses.

The `@property` decorator makes the method accessible like a calculated attribute.

Without `@property`, you would write:

```python
rectangle.get_area()
```

or:

```python
rectangle.area()
```

depending on the method's name.

### A property with a setter

```python
class Temperature:
    def __init__(self, celsius: float) -> None:
        self._celsius = celsius

    @property
    def celsius(self) -> float:
        return self._celsius

    @celsius.setter
    def celsius(self, value: float) -> None:
        if value < -273.15:
            raise ValueError("Temperature cannot be below absolute zero.")

        self._celsius = value


temperature = Temperature(25)

print(temperature.celsius)

temperature.celsius = 30

print(temperature.celsius)
```

Output:

```text
25
30
```

The getter retrieves the value, while the setter validates and updates it.

The underscore in `_celsius` is a naming convention indicating that the attribute is intended for internal use.

---

# 11. Understanding `self` and `cls`

These names are especially important when working with classes and decorators.

## 11.1 `self`

`self` refers to the current instance.

```python
class User:
    def __init__(self, name: str) -> None:
        self.name = name

    def greet(self) -> str:
        return f"Hello, {self.name}"


user = User("Nzegge")

print(user.greet())
```

When you write:

```python
user.greet()
```

Python automatically supplies `user` as the first argument to the bound method.

Conceptually, the call is equivalent to:

```python
User.greet(user)
```

You normally write:

```python
user.greet()
```

because Python handles the binding.

## 11.2 `cls`

`cls` refers to the class itself.

```python
class User:
    role: str = "member"

    @classmethod
    def get_role(cls) -> str:
        return cls.role


print(User.get_role())
```

Output:

```text
member
```

Here:

```python
cls.role
```

accesses the class attribute.

**Remember:**

| Name | Refers to | Common use |
|---|---|---|
| `self` | An instance | Instance methods |
| `cls` | A class | Class methods |

Both are conventions, but using these names makes your code easier to understand.

---

# 12. Advanced Type Annotations

Now let's explore a few more annotations you will encounter in modern Python libraries.

## 12.1 `list[str]`

```python
names: list[str] = ["Nzegge", "Charles", "John"]
```

A list expected to contain strings.

## 12.2 `dict[str, int]`

```python
scores: dict[str, int] = {
    "Python": 90,
    "SQL": 85
}
```

A dictionary expected to have string keys and integer values.

## 12.3 `str | int`

```python
user_id: str | int = 100

user_id = "USR-100"
```

The value may be a string or an integer.

## 12.4 `list[str] | None`

```python
names: list[str] | None = None

names = ["Nzegge", "Charles"]
```

The variable may contain a list of strings or `None`.

## 12.5 `Callable`

```python
from collections.abc import Callable


def execute_action(action: Callable[[], str]) -> str:
    return action()


def say_hello() -> str:
    return "Hello!"


print(execute_action(say_hello))
```

The annotation:

```python
Callable[[], str]
```

means a callable that accepts no arguments and is expected to return a string.

Another example:

```python
Callable[[int, int], int]
```

This describes a callable expected to accept two integers and return an integer.

This is useful when a function receives another function as an argument.

## 12.6 `TypeVar` and `ParamSpec`

These are useful for advanced decorators.

`TypeVar` represents a type that can be determined or preserved by a type checker.

`ParamSpec` helps preserve a callable's parameter signature.

For example:

```python
from collections.abc import Callable
from functools import wraps
from typing import ParamSpec, TypeVar

P = ParamSpec("P")
R = TypeVar("R")


def log_calls(func: Callable[P, R]) -> Callable[P, R]:

    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        print(f"Calling {func.__name__}")

        result = func(*args, **kwargs)

        print(f"Finished {func.__name__}")

        return result

    return wrapper


@log_calls
def add(a: int, b: int) -> int:
    return a + b


print(add(10, 20))
```

Output:

```text
Calling add
Finished add
30
```

Let's understand the important pieces.

```python
P = ParamSpec("P")
```

`P` represents the original callable's parameter signature.

```python
R = TypeVar("R")
```

`R` represents its return type.

```python
Callable[P, R]
```

This indicates a callable with parameter types represented by `P` and a return type represented by `R`.

The wrapper uses:

```python
*args: P.args
**kwargs: P.kwargs
```

to preserve the parameter signature for static type checking.

And:

```python
-> R
```

indicates that the wrapper returns the same type expected from the original function.

This approach is particularly useful for reusable decorators.

**Important:** Type annotations help static analysis, but they do not automatically validate arguments or return values at runtime.

---

# 13. Why Use `functools.wraps`?

When writing function-based decorators, you should normally use `functools.wraps`.

Consider:

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

After decorating a function, metadata such as its name and docstring may refer to the wrapper instead of the original function.

Let's improve it:

```python
from functools import wraps


def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

Now the wrapper preserves important metadata from the original function.

Example:

```python
from functools import wraps


def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")

        return func(*args, **kwargs)

    return wrapper


@log_calls
def greet(name: str) -> str:
    """Return a greeting."""
    return f"Hello, {name}!"


print(greet.__name__)
print(greet.__doc__)
```

Output:

```text
greet
Return a greeting.
```

Without `@wraps(func)`, the wrapper's name and documentation would generally be exposed instead.

For production code, preserving metadata is good practice.

---

# 14. A Practical Class-Based Decorator: Measuring Execution Time

Let's create a decorator that measures how long a function takes to execute.

This is useful when building monitoring tools or profiling programs.

```python
from collections.abc import Callable
from functools import wraps
from time import perf_counter
from typing import Any


class Timer:
    def __init__(self, func: Callable[..., Any]) -> None:
        self.func = func
        wraps(func)(self)

    def __call__(self, *args: Any, **kwargs: Any) -> Any:
        start = perf_counter()

        try:
            return self.func(*args, **kwargs)
        finally:
            elapsed = perf_counter() - start

            print(
                f"{self.func.__name__} "
                f"took {elapsed:.6f} seconds"
            )


@Timer
def calculate_total(limit: int) -> int:
    total = 0

    for number in range(limit):
        total += number

    return total


result = calculate_total(1_000_000)

print(result)
```

The exact execution time depends on your computer.

## Understanding the implementation

### Store the original function

```python
self.func = func
```

This preserves a reference to the original function.

### Preserve the function's metadata

```python
wraps(func)(self)
```

This copies important metadata to the decorator instance.

This works because `functools.update_wrapper`, which `wraps` uses, can update a wrapper object as well as a function.

### Measure execution time

```python
start = perf_counter()
```

This records a high-resolution timestamp suitable for measuring elapsed time.

### Run the original function

```python
return self.func(*args, **kwargs)
```

The original function is called with its arguments, and its result is returned.

### Use `finally`

```python
finally:
    elapsed = perf_counter() - start
```

The `finally` block executes whether the wrapped function succeeds or raises an exception.

This means the elapsed time can still be reported when an exception occurs.

**Important limitation:** This is a synchronous decorator. It is not sufficient for correctly measuring asynchronous functions defined with `async def`; those require an asynchronous-aware wrapper.

---

# 15. A Class-Based Decorator With Arguments

Here is another practical example: a decorator that checks whether a function's result is positive.

```python
from collections.abc import Callable
from typing import Any


class ValidatePositiveResult:
    def __init__(self, func: Callable[..., Any]) -> None:
        self.func = func

    def __call__(self, *args: Any, **kwargs: Any) -> Any:
        result = self.func(*args, **kwargs)

        if result <= 0:
            raise ValueError("The result must be positive.")

        return result


@ValidatePositiveResult
def subtract(a: int, b: int) -> int:
    return a - b


print(subtract(10, 5))  # 5
```

But:

```python
subtract(5, 10)
```

raises:

```text
ValueError: The result must be positive.
```

This example demonstrates how a decorator can enforce a runtime rule.

Unlike a type annotation, which generally communicates an expected type, this decorator actively checks a condition.

In real applications, consider whether such validation belongs in a decorator, the function itself, or a separate validation layer. Choose the simplest design that fits the requirement.

---

# 16. Common Mistakes

## Mistake 1: Forgetting to return the result

Incorrect:

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        func(*args, **kwargs)

    return wrapper
```

If the original function returns a value, the wrapper discards it.

Correct:

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

## Mistake 2: Forgetting to return the wrapper

Incorrect:

```python
def decorator(func):
    def wrapper():
        func()
```

Correct:

```python
def decorator(func):
    def wrapper():
        func()

    return wrapper
```

The decorator must return the wrapper for the decorated name to refer to it.

## Mistake 3: Confusing `__init__()` with `__call__()`

Remember the two patterns.

Without decorator configuration:

```python
@MyDecorator
def greet():
    ...
```

The constructor receives the original function.

With decorator configuration:

```python
@Repeat(times=3)
def greet():
    ...
```

The constructor receives `times=3`, and `__call__()` receives the original function.

## Mistake 4: Forgetting `self`

Incorrect:

```python
class Greeter:
    def __call__(name: str) -> str:
        return f"Hello, {name}"
```

Correct:

```python
class Greeter:
    def __call__(self, name: str) -> str:
        return f"Hello, {name}"
```

Instance methods require the instance parameter, conventionally named `self`.

## Mistake 5: Returning a value from `__init__()`

Incorrect:

```python
class User:
    def __init__(self, name: str) -> str:
        self.name = name
        return name
```

Correct:

```python
class User:
    def __init__(self, name: str) -> None:
        self.name = name
```

`__init__()` must return `None`. It should initialize the instance, not return the object or another value.

## Mistake 6: Assuming type annotations validate values

Incorrect assumption:

```python
age: int = "twenty"
```

The annotation does not automatically prevent this assignment at runtime.

Use static type checking or explicit runtime validation when enforcement is required.

## Mistake 7: Using a class decorator without understanding callable behavior

A class-based decorator without `__call__()` may still be possible in other designs, but the common pattern that directly decorates a function and later calls the decorated name requires the returned object to be callable.

---

# 17. Best Practices

1. **Use type annotations to communicate intent.** Annotate parameters and return types when doing so improves clarity.

2. **Use `-> None` for methods that are not intended to return a meaningful value.** This includes ordinary initialization methods.

3. **Use `self` and `cls` consistently.** These conventions make code easier to read.

4. **Use `functools.wraps` with function-based decorators.** It preserves important metadata about the original function.

5. **Preserve return values.** A wrapper should normally return the result of the wrapped function.

6. **Preserve exceptions unless you intentionally want to handle them.** A decorator should not silently hide errors.

7. **Use `Callable`, `ParamSpec`, and `TypeVar` when they make decorator types clearer.** For simple learning examples, `Any` may be easier to understand initially.

8. **Keep decorators focused.** A decorator should ideally handle one concern, such as logging, timing, authorization, or validation.

9. **Use class-based decorators when instance state is useful.** For simple wrappers, a function-based decorator is often shorter and easier to maintain.

10. **Remember that annotations do not perform runtime validation.** Use explicit checks or suitable validation libraries when you need guarantees about incoming data.

---

# 18. Final Cheat Sheet

## Variable annotations

```python
name: str = "Nzegge"
age: int = 25
price: float = 19.99
active: bool = True
```

## Function annotations

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"
```

## Function returning nothing meaningful

```python
def display(message: str) -> None:
    print(message)
```

## Class and constructor

```python
class User:
    def __init__(self, name: str) -> None:
        self.name = name
```

## Callable instance

```python
class Greeter:
    def __call__(self, name: str) -> str:
        return f"Hello, {name}!"


greeter = Greeter()

print(greeter("Nzegge"))
```

## Function-based decorator

```python
from functools import wraps


def my_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print("After")

        return result

    return wrapper


@my_decorator
def greet(name: str) -> str:
    return f"Hello, {name}!"


print(greet("Nzegge"))
```

## Class-based decorator

```python
from collections.abc import Callable
from functools import wraps
from typing import Any


class MyDecorator:
    def __init__(self, func: Callable[..., Any]) -> None:
        self.func = func
        wraps(func)(self)

    def __call__(self, *args: Any, **kwargs: Any) -> Any:
        print("Before")

        result = self.func(*args, **kwargs)

        print("After")

        return result


@MyDecorator
def greet(name: str) -> str:
    return f"Hello, {name}!"


print(greet("Nzegge"))
```

## Class-based decorator with configuration

```python
from collections.abc import Callable
from typing import Any


class Repeat:
    def __init__(self, times: int) -> None:
        self.times = times

    def __call__(
        self,
        func: Callable[..., Any]
    ) -> Callable[..., Any]:

        def wrapper(*args: Any, **kwargs: Any) -> Any:
            result = None

            for _ in range(self.times):
                result = func(*args, **kwargs)

            return result

        return wrapper


@Repeat(times=3)
def greet() -> None:
    print("Hello!")


greet()
```

## Built-in decorators

```python
class Example:
    @staticmethod
    def static_method() -> str:
        return "Static method"

    @classmethod
    def class_method(cls) -> str:
        return cls.__name__

    @property
    def value(self) -> str:
        return "Property"
```

## The most important distinctions

| Concept | Meaning |
|---|---|
| `variable: str` | The variable is expected to contain a string. |
| `parameter: int` | The parameter is expected to be an integer. |
| `-> str` | The function is expected to return a string. |
| `-> None` | The function is not intended to return a meaningful value. |
| `__init__()` | Initializes an instance. |
| `__call__()` | Makes an instance callable. |
| `self` | Refers to the current instance. |
| `cls` | Refers to the class. |
| `@decorator` | Applies a decorator to a function or method. |
| `@staticmethod` | Defines a method without automatic instance or class binding. |
| `@classmethod` | Defines a method that receives the class. |
| `@property` | Allows a method to be accessed using attribute syntax. |
| `Callable` | Describes a callable object. |
| `ParamSpec` | Helps preserve callable parameter types. |
| `TypeVar` | Represents a type variable for static type checking. |
| `@wraps(func)` | Preserves important metadata from a wrapped function. |

---

# 19. What You Should Practise First

Do not try to memorize everything at once. Learn these concepts in order:

1. Create a class with `__init__()`.
2. Understand `self` and instance attributes.
3. Create a class with `__call__()`.
4. Write a simple function-based decorator.
5. Convert that decorator into a class-based decorator.
6. Add `Callable` and `Any` annotations.
7. Learn `@wraps`.
8. Build a configurable decorator.
9. Practise `@staticmethod`, `@classmethod`, and `@property`.
10. Learn `ParamSpec` and `TypeVar` when you are comfortable with the fundamentals.

**The single most important class-based decorator pattern to remember is this:**

```python
class Decorator:
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        # Do something before the function.
        result = self.func(*args, **kwargs)
        # Do something after the function.

        return result
```

When you write:

```python
@Decorator
def my_function():
    ...
```

Python effectively creates a `Decorator` instance around `my_function`.

The decorated name then refers to that instance, and calling it invokes `__call__()`.

Once you understand that relationship between `__init__()`, `__call__()`, and `@Decorator`, you have the foundation needed to understand many class-based decorators in Python libraries.