# Python `watchdog` — From Basics to Advanced

## 1. What is `watchdog`?

`watchdog` is a third-party Python library that monitors files and directories for file-system events, such as:

- Creating a file or directory
- Modifying a file
- Deleting a file or directory
- Moving or renaming a file or directory

It is useful for automatic backups, file-processing pipelines, development tools, and logging changes.

**Do not confuse these two:**
- Python `watchdog`: a library you import into Python.
- Linux `watch`: a command that repeatedly runs a shell command.

This document focuses on Python `watchdog`, then briefly covers the Linux command.

## 2. Install it

### Windows PowerShell

```powershell
py -m pip install watchdog
py -m pip show watchdog
```

### Linux / WSL Ubuntu

```bash
python3 -m pip install watchdog
python3 -m pip show watchdog
```

Install it inside the same virtual environment that runs your Python script.

Test the import:

```python
import watchdog

print("watchdog is installed")
```

## 3. The important components

| Component | Purpose |
|---|---|
| `Observer` | Monitors a path and manages the observer thread. |
| `FileSystemEventHandler` | Base class for your event-handling class. |
| `on_created()` | Responds to creation events. |
| `on_modified()` | Responds to modification events. |
| `on_deleted()` | Responds to deletion events. |
| `on_moved()` | Responds to move/rename events. |
| `schedule()` | Registers a handler to watch a path. |

Typical workflow:

```text
Choose folder -> create handler -> create Observer
-> schedule handler -> start observer -> receive events
-> stop observer -> join observer thread
```

## 4. Your first folder watcher

Create `watch_folder.py`:

```python
from pathlib import Path
from time import sleep

from watchdog.events import FileSystemEventHandler
from watchdog.observers import Observer

# Resolve the watched folder relative to this Python script.
BASE_DIR = Path(__file__).resolve().parent
WATCH_DIR = BASE_DIR / "watched_folder"

# Create the folder if it does not exist.
WATCH_DIR.mkdir(parents=True, exist_ok=True)


class MyHandler(FileSystemEventHandler):
    """Print file-system events."""

    def on_created(self, event):
        print(f"CREATED: {event.src_path}")

    def on_modified(self, event):
        print(f"MODIFIED: {event.src_path}")

    def on_deleted(self, event):
        print(f"DELETED: {event.src_path}")

    def on_moved(self, event):
        print(f"MOVED: {event.src_path} -> {event.dest_path}")


handler = MyHandler()
observer = Observer()

# recursive=True means watch nested directories too.
observer.schedule(handler, str(WATCH_DIR), recursive=True)
observer.start()

print(f"Watching: {WATCH_DIR}")
print("Create, edit, rename, or delete a file in that folder.")
print("Press Ctrl+C to stop.")

try:
    while True:
        # Keep the main script alive while the observer runs.
        sleep(1)
except KeyboardInterrupt:
    print("\\nStopping watcher...")
    observer.stop()
finally:
    # Wait for the observer thread to finish.
    observer.join()
    print("Watcher stopped.")
```

Run it:

```powershell
py watch_folder.py
```

Open `watched_folder` in File Explorer and create or edit a file. Events should appear in the terminal.

### What each step does

1. `MyHandler` defines what the program should do for each event.
2. `Observer()` creates the watcher.
3. `schedule()` tells it which folder to monitor and which handler to call.
4. `start()` starts monitoring.
5. The loop keeps the main script running.
6. `stop()` requests shutdown.
7. `join()` waits for the observer thread to finish.

A single save can trigger multiple events. That is normal because editors may write temporary files or perform multiple operations.

## 5. Understanding event attributes

```python
def on_created(self, event):
    print("Path:", event.src_path)
    print("Is directory:", event.is_directory)
```

- `event.src_path`: path associated with the event.
- `event.is_directory`: `True` when the event concerns a directory.
- `event.dest_path`: destination path for move events.

Convert a path string to a `Path` object when you want to inspect its name or extension:

```python
from pathlib import Path

def on_created(self, event):
    path = Path(event.src_path)
    print("Name:", path.name)
    print("Extension:", path.suffix)
```

`path.name` is the final name, and `path.suffix` is the extension, such as `.txt`.

## 6. Handle only files or specific extensions

Ignore directory events:

```python
def on_created(self, event):
    if event.is_directory:
        return

    print("New file:", event.src_path)
```

Watch only Python files:

```python
from pathlib import Path
from watchdog.events import FileSystemEventHandler

class PythonFileHandler(FileSystemEventHandler):
    def on_modified(self, event):
        if event.is_directory:
            return

        path = Path(event.src_path)
        if path.suffix.lower() == ".py":
            print("Python file changed:", path)
```

Filter several file types:

```python
ALLOWED_EXTENSIONS = {".py", ".txt", ".json", ".md"}

path = Path(event.src_path)
if path.suffix.lower() in ALLOWED_EXTENSIONS:
    print("Supported file:", path)
```

Use `.lower()` to handle extensions such as `.PY` as well as `.py`.

## 7. Recursive versus non-recursive watching

```python
observer.schedule(handler, str(WATCH_DIR), recursive=False)
```

This watches only the selected directory.

```python
observer.schedule(handler, str(WATCH_DIR), recursive=True)
```

This also watches subdirectories. For example, with `recursive=True`, a change in `watched_folder/lessons/python/basics.md` can be detected.

Watch large trees carefully: they can generate many events.

## 8. Use `pathlib` for paths

Relative paths can behave differently depending on where the script is launched. A robust pattern is:

```python
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent
WATCH_DIR = BASE_DIR / "watched_folder"

if not WATCH_DIR.is_dir():
    raise FileNotFoundError(f"Folder not found: {WATCH_DIR}")
```

Then schedule the observer:

```python
observer.schedule(handler, str(WATCH_DIR), recursive=True)
```

`Path(__file__).resolve().parent` means “the directory containing this Python script.”

## 9. Log events with `logging`

For a long-running watcher, a log file is often more useful than only printing to the terminal.

```python
import logging
from pathlib import Path
from watchdog.events import FileSystemEventHandler

BASE_DIR = Path(__file__).resolve().parent
LOG_FILE = BASE_DIR / "watcher.log"

logging.basicConfig(
    filename=LOG_FILE,
    level=logging.INFO,
    format="%(asctime)s %(levelname)s: %(message)s",
    encoding="utf-8",
)

class LoggingHandler(FileSystemEventHandler):
    def on_created(self, event):
        logging.info("Created: %s", event.src_path)

    def on_modified(self, event):
        logging.info("Modified: %s", event.src_path)

    def on_deleted(self, event):
        logging.info("Deleted: %s", event.src_path)

    def on_moved(self, event):
        logging.info("Moved: %s -> %s", event.src_path, event.dest_path)
```

Avoid putting passwords, private file contents, or other sensitive information in logs.

## 10. Practical example: back up created or modified files

This example copies a file to a backup folder. It overwrites the backup copy on later changes; it does not preserve historical versions or mirror deletions.

Example layout:

```text
file_watcher_project/
├── backup_watcher.py
├── watched/
│   └── lessons/
│       └── python.md
└── backups/
    └── lessons/
        └── python.md
```

Code:

```python
import shutil
from pathlib import Path
from time import sleep

from watchdog.events import FileSystemEventHandler
from watchdog.observers import Observer

BASE_DIR = Path(__file__).resolve().parent
WATCH_DIR = BASE_DIR / "watched"
BACKUP_DIR = BASE_DIR / "backups"

WATCH_DIR.mkdir(parents=True, exist_ok=True)
BACKUP_DIR.mkdir(parents=True, exist_ok=True)


class BackupHandler(FileSystemEventHandler):
    def backup_file(self, source_path):
        source = Path(source_path)

        # Ignore directories and paths that no longer exist.
        if not source.is_file():
            return

        # Preserve the file's path relative to the watched root.
        relative_path = source.relative_to(WATCH_DIR)
        destination = BACKUP_DIR / relative_path
        destination.parent.mkdir(parents=True, exist_ok=True)

        try:
            shutil.copy2(source, destination)
            print(f"Backed up: {source} -> {destination}")
        except OSError as error:
            print(f"Could not back up {source}: {error}")

    def on_created(self, event):
        if not event.is_directory:
            self.backup_file(event.src_path)

    def on_modified(self, event):
        if not event.is_directory:
            self.backup_file(event.src_path)


observer = Observer()
observer.schedule(BackupHandler(), str(WATCH_DIR), recursive=True)
observer.start()

print("Watching:", WATCH_DIR)
print("Backing up to:", BACKUP_DIR)
print("Press Ctrl+C to stop.")

try:
    while True:
        sleep(1)
except KeyboardInterrupt:
    observer.stop()
finally:
    observer.join()
```

### How relative paths work

If the source file is:

```text
watched/lessons/python/basics.md
```

then:

```python
relative_path = source.relative_to(WATCH_DIR)
```

produces:

```text
lessons/python/basics.md
```

And:

```python
destination = BACKUP_DIR / relative_path
```

produces:

```text
backups/lessons/python/basics.md
```

Preserving the relative path avoids collisions when different subdirectories contain files with the same name.

### Important limitations

- A later save overwrites the previous backup. Use timestamped or versioned copies to retain history.
- A file may still be written when an event arrives. Real tools should handle temporary copy failures and retry.
- Editors can generate duplicate or repeated events. Debouncing can reduce unnecessary work.
- This example does not handle deletes, moves, or full restore workflows.
- Keep the backup directory outside the watched directory, or the watcher may respond to its own copies.
- Test with disposable files and verify that backups can actually be restored.

## 11. Debouncing repeated events

**Debouncing** means waiting briefly until a file has been quiet before processing it. This helps when one save triggers several events.

A robust approach is:

1. Receive an event.
2. Record the path in a dictionary or set.
3. Reset a short timer for that path when another event arrives.
4. Process the path once after the quiet interval.
5. Retry when a file is temporarily locked or incomplete.

Avoid long `sleep()` calls inside event-handler methods. Handlers should return quickly; move lengthy work to a queue or worker thread.

## 12. Common errors

### `ModuleNotFoundError: No module named 'watchdog'`

Install using the same interpreter that runs the script:

```powershell
py -m pip install watchdog
```

Activate your virtual environment first if you use one.

### The wrong directory is being watched

Inspect paths:

```python
from pathlib import Path

print("Current working directory:", Path.cwd())
print("Script directory:", Path(__file__).resolve().parent)
```

Prefer paths based on `Path(__file__).resolve().parent`.

### No events appear

Check that:
- The watched path is correct.
- You are changing a file inside it.
- `observer.start()` has been called.
- The script is still running.
- `recursive=True` is used if you are changing a nested folder.
- The file system supports event notifications reliably.

Some network or virtualized file systems may not report events reliably. Polling can be an alternative.

### Duplicate event output

This can be normal. Filter events, debounce, or compare metadata if your task needs to avoid repeated work.

### Permission errors

Ensure the process has permission to read the watched folder and write to the backup destination. Do not run with elevated privileges unless necessary.

## 13. Polling versus `watchdog`

Polling repeatedly checks a folder:

```python
from pathlib import Path
from time import sleep

folder = Path("watched_folder")

while True:
    for path in folder.iterdir():
        if path.is_file():
            print(path.name, path.stat().st_mtime)

    sleep(5)
```

This example prints metadata repeatedly; it does not by itself identify which files changed. A real polling solution saves a previous snapshot and compares it with the next one.

| Topic | `watchdog` | Polling |
|---|---|---|
| Method | Reacts to file-system events | Scans at intervals |
| Response time | Usually quick | Depends on interval |
| Repeated scanning | Usually unnecessary | Required |
| Unusual/network file systems | Events may be unreliable | Can work if metadata access works |
| Implementation | Handler and observer lifecycle | Snapshot and comparison logic |

For important backups, do not rely only on events. Periodically reconcile the source and backup, verify copies, and test recovery.

## 14. Linux `watch` command

The Linux `watch` command is separate from Python's `watchdog` library. It reruns a shell command periodically.

```bash
watch -n 2 date
watch -n 5 'free -h'
watch -n 3 'df -h'
watch -n 2 'ls -lah'
```

- `-n 2`: repeat every two seconds.
- `free -h`: display memory usage.
- `df -h`: display disk space.
- `ls -lah`: list files, including hidden files, in readable form.

Stop with `Ctrl+C`. To read the options available on your system:

```bash
man watch
watch --help
```

## 15. Best practices

1. Use `pathlib` for predictable paths.
2. Confirm the watched folder exists.
3. Use recursive monitoring only when needed.
4. Ignore directory events when processing files only.
5. Filter by extension when appropriate.
6. Keep event handlers short.
7. Expect repeated or combined events.
8. Handle temporary file and permission errors.
9. Keep backup output outside the watched folder.
10. Log important events.
11. Test with temporary files.
12. Verify backups and test restoring them.
13. For important data, use periodic reconciliation as well as event monitoring.

## 16. Practice projects

Work through these in order:

1. Print all create, modify, move, and delete events.
2. Report changes only to `.py` files.
3. Notify when a new file arrives in an `incoming/` folder.
4. Write events and timestamps to `watcher.log`.
5. Count files after a directory changes.
6. Process a new JSON file after it is fully written.
7. Copy changed files to a backup folder.
8. Keep timestamped versions instead of overwriting backups.
9. Debounce repeated events.
10. Build a more reliable backup tool with retries, rename/delete handling, logging, and restore testing.

For each project, ask:
- Which event should trigger the program?
- What information is available on the event object?
- What action should run?
- What errors could occur?
- How will I test the result?

## 17. Quick reference

```python
from watchdog.events import FileSystemEventHandler
from watchdog.observers import Observer

class Handler(FileSystemEventHandler):
    def on_created(self, event):
        print("Created:", event.src_path)

    def on_modified(self, event):
        print("Modified:", event.src_path)

    def on_deleted(self, event):
        print("Deleted:", event.src_path)

    def on_moved(self, event):
        print("Moved:", event.src_path, "->", event.dest_path)

observer = Observer()
observer.schedule(Handler(), "watched_folder", recursive=True)
observer.start()
```

Keep the program running and stop/join the observer during shutdown.

### Terms to remember

- **Event:** A reported file-system change.
- **Handler:** Code that responds to events.
- **Observer:** Component that watches paths and dispatches events.
- **Recursive:** Include nested directories.
- **Polling:** Check for changes at intervals.
- **Debouncing:** Combine repeated events into one later action.
- **Backup:** A separate copy of data that should be verified and recoverable.

## Official documentation

- `watchdog`: https://python-watchdog.readthedocs.io/
- PyPI package: https://pypi.org/project/watchdog/
- Python `pathlib`: https://docs.python.org/3/library/pathlib.html
- Python `logging`: https://docs.python.org/3/library/logging.html
- Python `shutil`: https://docs.python.org/3/library/shutil.html
