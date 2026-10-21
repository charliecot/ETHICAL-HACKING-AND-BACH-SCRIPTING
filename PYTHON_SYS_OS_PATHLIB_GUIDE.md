# Python `sys`, `os`, and `pathlib` --- Complete Practical Guide

A beginner-friendly reference explaining what each module does, when to
use it, how they work together, and how to avoid confusion.

------------------------------------------------------------------------

# 1. The Big Picture

  -----------------------------------------------------------------------
  Module                  Main responsibility     Think of it as
  ----------------------- ----------------------- -----------------------
  `sys`                   Python interpreter,     Python itself
                          command-line arguments, 
                          import paths, standard  
                          streams, exit status    

  `os`                    Environment variables,  The operating system
                          current working         
                          directory,              
                          operating-system        
                          operations              

  `pathlib`               Building, inspecting,   Working with paths
                          creating, reading, and  
                          writing filesystem      
                          paths                   
  -----------------------------------------------------------------------

## The simplest rule

-   Need command-line arguments or Python interpreter details? Use
    `sys`.
-   Need environment variables or current working directory? Use `os`.
-   Need to build, inspect, or manipulate file paths? Prefer `pathlib`.

They are not competitors. They can work together.

``` python
import sys
import os
from pathlib import Path
```

------------------------------------------------------------------------

# 2. `sys` --- Work with the Python Interpreter

The `sys` module provides access to the running Python interpreter and
its environment.

## 2.1 `sys.version`

``` python
import sys

print(sys.version)  # Show the Python version and build information
```

Useful when debugging version-dependent behavior.

## 2.2 `sys.executable`

``` python
import sys

print(sys.executable)  # Path to the Python executable running this program
```

This is useful when multiple Python installations or virtual
environments exist.

## 2.3 `sys.argv` --- Command-line arguments

`sys.argv` is a list containing the script name and the arguments passed
to it.

Create `greet.py`:

``` python
import sys

print(sys.argv)
```

Run:

``` bash
python greet.py Charles 25
```

Possible output:

``` text
['greet.py', 'Charles', '25']
```

Remember: - `sys.argv[0]` is usually the script name or path. -
`sys.argv[1]` is the first argument. - `sys.argv[2]` is the second
argument. - Arguments are strings, even when they look like numbers.

### Practical example

``` python
# greet.py
import sys

if len(sys.argv) < 2:
    print("Usage: python greet.py NAME")
    sys.exit(1)

name = sys.argv[1]
print(f"Hello, {name}!")
```

Run:

``` bash
python greet.py Charles
```

Output:

``` text
Hello, Charles!
```

For more complex command-line applications, `argparse` is usually
clearer than manually parsing a long `sys.argv` list.

## 2.4 `sys.exit()`

``` python
import sys

age = 15

if age < 18:
    print("You must be at least 18.")
    sys.exit(1)

print("Access allowed.")
```

Common conventions: - `sys.exit(0)`: successful completion. -
`sys.exit(1)`: general failure. - Other nonzero values may indicate
other failure types.

`sys.exit()` raises `SystemExit`. It normally terminates the program
unless that exception is caught.

## 2.5 `sys.path`

`sys.path` is a list of locations Python searches when importing
modules.

``` python
import sys

for location in sys.path:
    print(location)
```

You can temporarily add an import directory:

``` python
import sys
from pathlib import Path

project_root = Path(__file__).resolve().parent
sys.path.insert(0, str(project_root / "helpers"))
```

Do not use `sys.path` modifications as the automatic fix for every
import issue. Prefer a well-structured package and the correct project
environment.

## 2.6 Standard input, output, and error

-   `sys.stdin`: standard input.
-   `sys.stdout`: normal output.
-   `sys.stderr`: error or diagnostic output.

``` python
import sys

sys.stdout.write("Normal message\n")
sys.stderr.write("Diagnostic message\n")
```

Use `print()` for ordinary beginner-level output. Use `sys.stderr` when
errors should be separated from normal output.

## 2.7 `sys.platform`

``` python
import sys

print(sys.platform)
```

Typical values: - `"win32"` on Windows. - `"linux"` on Linux. -
`"darwin"` on macOS.

``` python
import sys

if sys.platform == "win32":
    print("Running on Windows")
elif sys.platform.startswith("linux"):
    print("Running on Linux")
elif sys.platform == "darwin":
    print("Running on macOS")
else:
    print("Running on another platform")
```

For most path operations, `pathlib` handles platform-specific path
conventions for you.

## 2.8 `sys.stdin`

``` python
import sys

text = sys.stdin.readline()
print("You entered:", text.strip())
```

Useful when input comes from a pipe or redirected file.

``` bash
python read_input.py < input.txt
```

## 2.9 Useful `sys` names

  -----------------------------------------------------------------------
  Name                                Purpose
  ----------------------------------- -----------------------------------
  `sys.argv`                          Command-line arguments

  `sys.version`                       Python version information

  `sys.version_info`                  Structured version information

  `sys.executable`                    Python executable path

  `sys.path`                          Module import search path

  `sys.platform`                      Platform identifier

  `sys.exit()`                        Request program termination

  `sys.stdin`                         Standard input

  `sys.stdout`                        Standard output

  `sys.stderr`                        Standard error

  `sys.modules`                       Modules loaded in this interpreter

  `sys.getsizeof(obj)`                Approximate size of an object; not
                                      all objects it references

  `sys.maxsize`                       Platform-dependent size-related
                                      limit

  `sys.implementation`                Python implementation information

  `sys.prefix`                        Prefix of the current Python
                                      installation or environment

  `sys.base_prefix`                   Prefix of the base Python
                                      installation
  -----------------------------------------------------------------------

### Check for a virtual environment

``` python
import sys

if sys.prefix != sys.base_prefix:
    print("Likely running inside a virtual environment")
else:
    print("Likely running outside a virtual environment")
```

This is a common check, though unusual Python setups may affect its
interpretation.

------------------------------------------------------------------------

# 3. `os` --- Interact with the Operating System

The `os` module provides interfaces to operating-system features, such
as environment variables, working directories, and filesystem
operations.

## 3.1 `os.getcwd()` --- Current working directory

``` python
import os

print(os.getcwd())
```

The current working directory (CWD) is the directory from which relative
paths are interpreted.

It is not necessarily the directory containing your script.

For example:

``` bash
cd /home/user
python projects/demo.py
```

The current working directory may be `/home/user`, while the script is
in `/home/user/projects`.

## 3.2 `os.chdir()` --- Change the working directory

``` python
import os

os.chdir("/tmp")
print(os.getcwd())
```

On Windows:

``` python
import os

os.chdir(r"C:\Projects")
```

Use `chdir()` only when you intentionally want to change the working
directory of the running process. It affects subsequent relative-path
operations.

Often it is safer to use explicit `pathlib` paths instead of changing
the working directory.

## 3.3 `os.environ` and `os.getenv()` --- Environment variables

Environment variables store configuration outside your source code.

``` python
import os

print(os.getenv("HOME"))  # Common on Linux and macOS
print(os.getenv("USERPROFILE"))  # Common on Windows
```

Read a setting:

``` python
import os

database_url = os.environ.get("DATABASE_URL")

if database_url is None:
    print("DATABASE_URL is not configured")
else:
    print("Database URL is configured")
```

Supply a fallback:

``` python
import os

mode = os.getenv("APP_MODE", "development")
print(mode)
```

Set an environment variable for the current process:

``` python
import os

os.environ["APP_MODE"] = "testing"
print(os.environ["APP_MODE"])
```

This affects the current Python process and its subsequently launched
child processes. It does not permanently change your operating system's
environment settings.

Avoid printing secrets such as passwords, API keys, and database
credentials in logs.

## 3.4 `os.path` --- Path utilities

`os.path` provides functions for working with path strings. It remains
valid and useful, especially in existing code.

``` python
import os

folder = "projects"
filename = "main.py"

path = os.path.join(folder, filename)
print(path)
```

More examples:

``` python
import os

path = "projects/main.py"

print(os.path.basename(path))  # Final path component
print(os.path.dirname(path))   # Parent path text
print(os.path.splitext(path))  # Split into root and final extension
print(os.path.exists(path))    # Does the path exist?
print(os.path.isfile(path))    # Is it a file?
print(os.path.isdir(path))     # Is it a directory?
print(os.path.abspath(path))   # Absolute path
```

For new path-oriented code, `pathlib` often reads more clearly.

## 3.5 Create directories

Create one directory:

``` python
import os

os.mkdir("reports")
```

Create nested directories:

``` python
import os

os.makedirs("data/reports/2026", exist_ok=True)
```

`exist_ok=True` prevents an error just because the target directory
already exists.

The `pathlib` equivalent is:

``` python
from pathlib import Path

Path("data/reports/2026").mkdir(parents=True, exist_ok=True)
```

## 3.6 `os.listdir()` --- List directory entries

``` python
import os

for name in os.listdir("."):
    print(name)
```

The `"."` represents the current working directory. `os.listdir()`
returns names, not full paths.

## 3.7 `os.walk()` --- Traverse directory trees

``` python
import os

for root, directories, files in os.walk("."):
    print("Current directory:", root)

    for directory in directories:
        print("  Directory:", directory)

    for filename in files:
        print("  File:", filename)
```

`os.walk()` recursively traverses a directory tree. It is useful for
indexing files and controlling which directories to visit.
`Path.rglob()` can be simpler for straightforward recursive searches.

## 3.8 Rename and remove files or directories

``` python
import os

os.rename("old.txt", "new.txt")  # Rename or move a path
os.remove("new.txt")             # Delete a file
os.rmdir("empty_folder")         # Remove an empty directory
```

These operations change the filesystem. Verify paths before destructive
operations.

`shutil.rmtree()` can recursively delete a directory and its contents.
Use it only when you have verified the exact target and intend to remove
everything inside it.

## 3.9 `os.name` and `os.cpu_count()`

``` python
import os

print(os.name)       # Commonly "nt" on Windows, "posix" on Linux/macOS
print(os.cpu_count())  # Logical CPU count, or None if unknown
```

`os.name` is a broad OS-interface identifier. `sys.platform` is usually
more specific.

## 3.10 Useful `os` names

  ------------------------------------------------------------------------
  Name                                 Purpose
  ------------------------------------ -----------------------------------
  `os.getcwd()`                        Get current working directory

  `os.chdir(path)`                     Change current working directory

  `os.environ`                         Environment-variable mapping

  `os.getenv(name)`                    Read an environment variable

  `os.name`                            General OS-interface identifier

  `os.listdir(path)`                   List directory entry names

  `os.mkdir(path)`                     Create one directory

  `os.makedirs(path, exist_ok=True)`   Create nested directories

  `os.remove(path)`                    Delete a file

  `os.rename(src, dst)`                Rename or move a path, subject to
                                       platform/filesystem rules

  `os.rmdir(path)`                     Remove an empty directory

  `os.walk(path)`                      Traverse a directory tree

  `os.cpu_count()`                     Report logical CPU count

  `os.path.join()`                     Join path strings

  `os.path.exists()`                   Check path existence

  `os.path.isfile()`                   Check for a file

  `os.path.isdir()`                    Check for a directory

  `os.path.abspath()`                  Convert to an absolute path

  `os.path.basename()`                 Get final path component

  `os.path.dirname()`                  Get parent path text

  `os.path.splitext()`                 Split root and final extension
  ------------------------------------------------------------------------

------------------------------------------------------------------------

# 4. `pathlib` --- Work with Paths Clearly

`pathlib` is the recommended modern interface for many filesystem path
tasks. It represents paths as objects instead of requiring you to
manipulate strings manually.

``` python
from pathlib import Path

path = Path("data") / "reports" / "summary.txt"
print(path)
```

The `/` operator joins path components here; it is not performing
arithmetic division.

## 4.1 Create a `Path`

``` python
from pathlib import Path

path = Path("notes.txt")
folder = Path("documents")
```

These statements represent paths. They do not create a file or directory
automatically.

## 4.2 Join paths

``` python
from pathlib import Path

project = Path("my_project")
file_path = project / "src" / "main.py"

print(file_path)
```

Avoid manually building paths with separators:

``` python
# Fragile: hardcodes a separator.
path = "my_project" + "/" + "src" + "/" + "main.py"
```

Prefer:

``` python
from pathlib import Path

path = Path("my_project") / "src" / "main.py"
```

## 4.3 Absolute and resolved paths

``` python
from pathlib import Path

path = Path("notes.txt")

print(path.absolute())
print(path.resolve())
```

-   `absolute()` makes the path absolute.
-   `resolve()` resolves it to an absolute path, normally eliminating
    `..` components and resolving symbolic links where possible.

An absolute path does not guarantee that the file exists.

## 4.4 Script directory versus current working directory

``` python
from pathlib import Path

print(Path.cwd())  # Current working directory
```

In a normal Python script:

``` python
from pathlib import Path

script_directory = Path(__file__).resolve().parent
print(script_directory)
```

`__file__` may not be available in interactive consoles or some
notebooks.

Remember:

-   `Path.cwd()` means the process's current working directory.
-   `Path(__file__).resolve().parent` means the directory containing the
    current script.

## 4.5 Inspect a path

``` python
from pathlib import Path

path = Path("notes.txt")

print(path.exists())   # Does it exist?
print(path.is_file())  # Is it a file?
print(path.is_dir())   # Is it a directory?
print(path.name)       # Final component, e.g. notes.txt
print(path.stem)       # Name without final suffix, e.g. notes
print(path.suffix)     # Final suffix, e.g. .txt
print(path.parent)     # Parent path
```

For `archive.tar.gz`, `path.suffix` is `.gz`, and `path.stem` is
`archive.tar`.

``` python
from pathlib import Path

path = Path("archive.tar.gz")
print(path.suffixes)  # ['.tar', '.gz']
```

## 4.6 Create directories

``` python
from pathlib import Path

Path("reports").mkdir(exist_ok=True)
```

Create nested folders:

``` python
from pathlib import Path

folder = Path("data") / "reports" / "2026"
folder.mkdir(parents=True, exist_ok=True)
```

-   `parents=True`: create missing parent directories.
-   `exist_ok=True`: do not fail just because the target directory
    already exists.

## 4.7 Write text to a file

``` python
from pathlib import Path

path = Path("notes.txt")

path.write_text(
    "Learning Python sys, os, and pathlib.\n",
    encoding="utf-8",
)
```

`write_text()` creates the file if needed and replaces existing content.

**Warning:** This overwrites existing file contents. Use it only when
replacement is intended.

## 4.8 Read text from a file

``` python
from pathlib import Path

path = Path("notes.txt")

if path.is_file():
    content = path.read_text(encoding="utf-8")
    print(content)
else:
    print("File not found")
```

For large files, reading everything at once may consume a lot of memory.
Read line by line when appropriate:

``` python
from pathlib import Path

path = Path("notes.txt")

with path.open("r", encoding="utf-8") as file:
    for line in file:
        print(line.rstrip())
```

## 4.9 Append to a file

``` python
from pathlib import Path

path = Path("notes.txt")

with path.open("a", encoding="utf-8") as file:
    file.write("Another line.\n")
```

Mode `"a"` appends rather than replacing existing content.

## 4.10 Delete a file or empty directory

``` python
from pathlib import Path

file_path = Path("temporary.txt")

if file_path.is_file():
    file_path.unlink()
```

`unlink()` removes a file or symbolic link. It does not remove a
non-empty directory.

``` python
from pathlib import Path

folder = Path("empty_folder")

if folder.is_dir():
    folder.rmdir()
```

For a directory containing files, recursive deletion requires another
approach. Be especially careful with recursive deletion.

## 4.11 List directory contents

``` python
from pathlib import Path

for item in Path(".").iterdir():
    print(item)
```

`iterdir()` lists immediate children only.

## 4.12 Find files recursively

``` python
from pathlib import Path

for file_path in Path(".").rglob("*.py"):
    print(file_path)
```

This finds `.py` files beneath the starting directory.

## 4.13 File metadata

``` python
from pathlib import Path

path = Path("notes.txt")

if path.is_file():
    info = path.stat()
    print("Size:", info.st_size, "bytes")
    print("Modified timestamp:", info.st_mtime)
```

`st_mtime` is a timestamp. Use `datetime` to format it as a readable
date and time.

## 4.14 Useful `pathlib` names

  Name                     Purpose
  ------------------------ ----------------------------------------
  `Path(path)`             Create a path object
  `Path.cwd()`             Current working directory
  `Path.home()`            User home directory
  `path / "child"`         Join a path component
  `path.resolve()`         Resolve to an absolute path
  `path.absolute()`        Make a path absolute
  `path.exists()`          Check existence
  `path.is_file()`         Check for a file
  `path.is_dir()`          Check for a directory
  `path.iterdir()`         List immediate children
  `path.glob(pattern)`     Find paths matching a pattern
  `path.rglob(pattern)`    Search recursively
  `path.mkdir()`           Create a directory
  `path.rename(target)`    Rename or move a path
  `path.replace(target)`   Replace a target path where supported
  `path.unlink()`          Remove a file or symbolic link
  `path.rmdir()`           Remove an empty directory
  `path.read_text()`       Read text
  `path.write_text()`      Write text, replacing existing content
  `path.open()`            Open a file
  `path.stat()`            Retrieve filesystem metadata
  `path.name`              Final component
  `path.stem`              Name without final suffix
  `path.suffix`            Final suffix
  `path.suffixes`          List of suffixes
  `path.parent`            Parent path
  `path.parents`           Parent paths in order

------------------------------------------------------------------------

# 5. SIDE-BY-SIDE EXAMPLES

## Example A: Which Python is running?

Use `sys`:

``` python
import sys

print(sys.executable)
print(sys.version)
```

## Example B: What directory is the process using?

Use `os` or `pathlib`:

``` python
import os
from pathlib import Path

print(os.getcwd())
print(Path.cwd())
```

Both report the current working directory.

## Example C: Build a path to a file

Prefer `pathlib`:

``` python
from pathlib import Path

path = Path("data") / "input.txt"
print(path)
```

## Example D: Read command-line arguments

Use `sys`:

``` python
import sys

print(sys.argv)
```

## Example E: Read configuration

Use `os`:

``` python
import os

mode = os.getenv("APP_MODE", "development")
print(mode)
```

## Example F: Check whether a file exists

Prefer `pathlib`:

``` python
from pathlib import Path

path = Path("data/input.txt")

if path.is_file():
    print("File exists")
else:
    print("File does not exist")
```

## Example G: Find every Python file

``` python
from pathlib import Path

for file_path in Path(".").rglob("*.py"):
    print(file_path)
```

Use `os.walk()` instead when you need detailed control over directory
traversal.

------------------------------------------------------------------------

# 6. MINI-PROJECT --- FILE INSPECTOR

Save as `file_inspector.py`:

``` python
import os
import sys
from pathlib import Path


def main():
    # sys: read command-line arguments.
    if len(sys.argv) < 2:
        print("Usage: python file_inspector.py PATH")
        sys.exit(1)

    # pathlib: represent and expand the supplied path.
    target = Path(sys.argv[1]).expanduser().resolve()

    # os: read an environment variable.
    app_mode = os.getenv("APP_MODE", "development")

    print("Python executable:", sys.executable)
    print("Python version:", sys.version.split()[0])
    print("Current working directory:", Path.cwd())
    print("Application mode:", app_mode)
    print("Target:", target)

    if not target.exists():
        print("Error: path does not exist")
        sys.exit(1)

    if target.is_file():
        print("Type: file")
        print("Name:", target.name)
        print("Extension:", target.suffix)
        print("Size:", target.stat().st_size, "bytes")

    elif target.is_dir():
        print("Type: directory")
        print("Immediate entries:")

        for item in target.iterdir():
            print(" -", item.name)

    else:
        print("The path exists but is not a regular file or directory.")


if __name__ == "__main__":
    main()
```

Run it:

``` bash
python file_inspector.py notes.txt
python file_inspector.py .
```

The `"."` means the current working directory.

What each module does here: - `sys.argv`: obtains the supplied path. -
`sys.exit()`: exits when input is missing or invalid. - `sys.executable`
and `sys.version`: report Python runtime details. - `os.getenv()`: reads
application configuration. - `Path`: resolves, checks, and inspects
filesystem paths. - `Path.cwd()`: reports the current working directory.

For production code, consider handling filesystem exceptions such as
`PermissionError` and `OSError` as well.

------------------------------------------------------------------------

# 7. COMMON CONFUSIONS

## Confusion 1: Script directory versus current working directory

These can be different.

``` python
from pathlib import Path

print(Path.cwd())  # Process working directory
print(Path(__file__).resolve().parent)  # Directory containing this script
```

The second example requires `__file__`, which is normally available in a
script but may be missing in interactive environments.

## Confusion 2: A `Path` object does not create a file

``` python
from pathlib import Path

path = Path("hello.txt")  # Represents a path; does not create the file
path.write_text("Hello!\n", encoding="utf-8")  # Creates or overwrites the file
```

## Confusion 3: `sys.path` is not a filesystem path

-   `sys.path` is a list of directories Python searches for imports.
-   `Path("notes.txt")` represents a filesystem path.

## Confusion 4: `os.getenv()` does not load `.env` files automatically

`os.getenv("DATABASE_URL")` reads an environment variable already
available to the process. It does not automatically parse a `.env` file.

## Confusion 5: Relative paths depend on the working directory

``` python
from pathlib import Path

path = Path("data.txt")  # Relative to the current working directory
```

If the file should be beside the script:

``` python
from pathlib import Path

script_dir = Path(__file__).resolve().parent
path = script_dir / "data.txt"
```

## Confusion 6: Existence checks can become outdated

Another process could remove a file between `exists()` and
`read_text()`. For robust programs, handle relevant filesystem
exceptions.

## Confusion 7: Do not delete paths casually

Before calling `unlink()`, `rmdir()`, `os.remove()`, or a recursive
deletion function, verify the exact target.

## Confusion 8: Do not manually join paths

Avoid:

``` python
path = folder + "/" + filename
```

Prefer:

``` python
from pathlib import Path

path = Path(folder) / filename
```

------------------------------------------------------------------------

# 8. DECISION GUIDE

**Question 1: Am I working with the Python runtime or command-line
input?**

Use `sys` for: - Python version and executable - Command-line
arguments - Import search path - Standard streams - Program exit status

**Question 2: Am I interacting with environment variables or
operating-system settings?**

Use `os` for: - Environment variables - Current working directory -
Changing the working directory - OS-specific operations - `os.walk()`
when detailed traversal control is needed

**Question 3: Am I building, checking, reading, or writing a path or
file?**

Prefer `pathlib` for: - Joining paths - Checking files and directories -
Creating directories - Reading and writing text - Finding files
recursively

------------------------------------------------------------------------

# 9. PRACTICE EXERCISES

Try these before looking for solutions:

1.  Print the Python version and executable path.
2.  Print every entry in `sys.path`.
3.  Write a script that accepts a filename through `sys.argv`.
4.  Print the current working directory using `os` and `pathlib`.
5.  Read `APP_MODE`, defaulting to `"development"`.
6.  Create `practice/files` using `pathlib`.
7.  Create a text file inside it and write three lines.
8.  Read the file and print each line without its newline.
9.  Find all `.py` files beneath a directory.
10. Report whether a supplied path exists, is a file, or is a directory.
11. Display a file's name, suffix, parent, and size.
12. Build a file-inspection command-line tool with useful error
    messages.

## Challenge project

Build `project_report.py` that: - Uses `sys.argv` to accept a project
directory. - Uses `pathlib` to validate the directory and find `.py`
files. - Uses `os.getenv()` to read an optional `APP_MODE`. - Prints the
number and paths of Python files. - Exits with a nonzero status if the
directory is invalid.

------------------------------------------------------------------------

# 10. QUICK CHEAT SHEET

``` python
# sys: Python runtime and command-line features
import sys

print(sys.version)       # Python version
print(sys.executable)    # Python executable
print(sys.argv)          # Command-line arguments
print(sys.path)          # Import search paths
print(sys.platform)      # Platform identifier
sys.exit(0)              # Exit successfully


# os: operating-system features
import os

print(os.getcwd())       # Current working directory
print(os.getenv("HOME")) # Common on Linux/macOS
print(os.name)           # General OS interface name
print(os.listdir("."))   # List directory entries
os.makedirs("a/b", exist_ok=True)  # Create nested directories


# pathlib: path and file operations
from pathlib import Path

path = Path("data") / "notes.txt"  # Join paths
print(Path.cwd())                   # Current working directory
print(Path.home())                  # User home directory
print(path.exists())                # Does it exist?
print(path.is_file())               # Is it a file?
print(path.is_dir())                # Is it a directory?
print(path.name)                    # Final component
print(path.suffix)                  # Final extension
print(path.parent)                  # Parent path

path.parent.mkdir(parents=True, exist_ok=True)
path.write_text("Hello\n", encoding="utf-8")
print(path.read_text(encoding="utf-8"))
```

Note: `HOME` is common on Linux and macOS. On Windows, `USERPROFILE` is
often available instead.

------------------------------------------------------------------------

# 11. OFFICIAL DOCUMENTATION

-   `sys`: https://docs.python.org/3/library/sys.html
-   `os`: https://docs.python.org/3/library/os.html
-   `pathlib`: https://docs.python.org/3/library/pathlib.html
-   `argparse`: https://docs.python.org/3/library/argparse.html
-   `shutil`: https://docs.python.org/3/library/shutil.html

## Final rule

**`sys` = Python runtime. `os` = operating-system features. `pathlib` =
paths and filesystem objects.**

Use the module that matches the problem. Prefer `pathlib` for most new
path-handling code, `os` for environment and OS features, and `sys` for
interpreter and command-line details.
