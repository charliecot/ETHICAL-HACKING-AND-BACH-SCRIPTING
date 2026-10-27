# Python Practical Guide: Logging, Custom Logging, `datetime`, Time Zones, and Formatting

> A simple, practical reference for everyday Python work, file monitoring, backups, debugging, and system tools.
>
> **Recommended approach:** Use Python's `logging` module for application logs and `datetime` with `zoneinfo` for human-readable dates and time zones. Use `time.perf_counter()` for measuring durations, not `datetime`.

---

# Part 1 — Python Logging

## 1. What is logging?

Logging lets your program record what is happening while it runs.

For example, a file-backup program may need to record:

- The program started.
- A file was detected.
- A backup was created.
- A file could not be read.
- An unexpected error occurred.

You could use `print()`, but logging is more flexible. It supports severity levels, timestamps, output to files or terminals, formatting, filtering, and exception tracebacks.

```python
import logging

logging.warning("This is a warning")
```

By default, the root logger normally displays messages at `WARNING` level and above.

## 2. The five standard logging levels

From least severe to most severe:

| Level | Numeric value | Use it for |
|---|---:|---|
| `DEBUG` | 10 | Detailed diagnostic information useful while developing |
| `INFO` | 20 | Normal events worth recording |
| `WARNING` | 30 | Something unexpected or potentially problematic, but the program can continue |
| `ERROR` | 40 | An operation failed |
| `CRITICAL` | 50 | A serious problem that may prevent the program from continuing |

Example:

```python
import logging

logging.basicConfig(level=logging.DEBUG)

logging.debug("Debug details")
logging.info("Program started")
logging.warning("Disk space is getting low")
logging.error("Could not copy a file")
logging.critical("The application cannot continue")
```

Because the configured level is `DEBUG`, all five messages can appear.

If the level is set to `INFO`, `DEBUG` messages are filtered out:

```python
logging.basicConfig(level=logging.INFO)
```

The level is a threshold: messages at that level and above are eligible to be emitted.

> `logging.basicConfig()` generally configures the root logger only if it has not already been configured with handlers. In notebooks or larger applications, existing configuration can affect the result.

## 3. `logging.basicConfig()` — simple logging setup

`basicConfig()` is the easiest way to configure basic logging.

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)

logging.info("Application started")
logging.warning("This is a warning")
```

Example output:

```text
2026-10-27 16:30:00 | INFO | Application started
2026-10-27 16:30:00 | WARNING | This is a warning
```

The exact timestamp depends on when you run the program and your system's local time.

Arguments to know:

- `level`: minimum severity to display.
- `format`: layout of each log record.
- `datefmt`: format used for the timestamp in `asctime`.
- `filename`: write logs to a file instead of the default console handler.
- `filemode`: file opening mode, such as `"a"` for append or `"w"` for overwrite.
- `stream`: stream used by the default console handler.
- `handlers`: provide pre-created handlers.
- `force=True`: replace existing root handlers; use cautiously because it changes existing logging configuration.

Example that logs to a file:

```python
import logging

logging.basicConfig(
    filename="application.log",
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
    encoding="utf-8",
)

logging.info("Application started")
logging.error("An operation failed")
```

`encoding` is available on supported modern Python versions. It is useful for writing non-ASCII text correctly.

## 4. The most useful logging methods

The module-level functions are convenient for small scripts:

```python
import logging

logging.debug("Detailed diagnostic")
logging.info("Normal event")
logging.warning("Unexpected condition")
logging.error("Operation failed")
logging.critical("Severe failure")
```

When you need exception details, use:

### `logging.exception()`

Call it from inside an `except` block. It logs at `ERROR` level and includes exception information.

```python
import logging

logging.basicConfig(level=logging.INFO)

try:
    result = 10 / 0
except ZeroDivisionError:
    logging.exception("Calculation failed")
```

The output includes the message and traceback.

You can also use:

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    logging.error("Calculation failed", exc_info=True)
```

Use `logging.exception()` when you want a traceback for the exception currently being handled. Do not call it outside an exception handler expecting it to invent an exception traceback.

## 5. Use a logger object in real projects

For reusable modules and applications, prefer a named logger over calling module-level functions everywhere.

```python
import logging

logger = logging.getLogger(__name__)

logger.debug("Debug information")
logger.info("Module started")
logger.warning("Unexpected state")
logger.error("Operation failed")
```

`__name__` is the current module's name. If the code is in `backup.py`, it will normally be `"__main__"` when run directly and `"backup"` when imported as a module.

Why use `logging.getLogger(__name__)`?

- Log messages show which module generated them when the format includes `%(name)s`.
- Logging can be configured centrally.
- Different modules can be filtered separately.
- Libraries can emit logs without deciding the application's final output configuration.

A reusable module should usually create a logger but leave final logging configuration to the application.

---

# Part 2 — Understanding Log Format Fields

The `format` argument to `basicConfig()` uses **logging-specific fields** written as `%(fieldname)s`.

This is different from the `%Y`, `%m`, and `%d` directives used by date formatting.

Example:

```python
format="%(asctime)s | %(levelname)s | %(name)s | %(message)s"
```

| Logging field | Meaning |
|---|---|
| `%(asctime)s` | Human-readable time when the log record was created |
| `%(levelname)s` | Level name, such as `INFO` or `ERROR` |
| `%(levelno)s` | Numeric severity level |
| `%(name)s` | Logger name |
| `%(message)s` | Final log message after interpolation |
| `%(pathname)s` | Full path of the source file |
| `%(filename)s` | Source filename |
| `%(module)s` | Module name derived from the filename |
| `%(funcName)s` | Function that issued the logging call |
| `%(lineno)d` | Source line number |
| `%(process)d` | Process ID |
| `%(processName)s` | Process name, when available |
| `%(thread)d` | Thread ID |
| `%(threadName)s` | Thread name |
| `%(created)f` | Timestamp as seconds since the Unix epoch |
| `%(relativeCreated)d` | Milliseconds since logging was loaded |
| `%(levelname)s` | Severity label |

A useful development format:

```python
"%(asctime)s | %(levelname)s | %(name)s | %(filename)s:%(lineno)d | %(message)s"
```

This can tell you when the event happened, which module generated it, and the source line.

## Logging format vs date format

These are two separate format systems:

```python
# Logging record format:
"%(asctime)s | %(levelname)s | %(message)s"

# Date/time format:
"%Y-%m-%d %H:%M:%S"
```

The first controls the whole log line. The second controls how the timestamp is displayed.

Example:

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(message)s",
    datefmt="%d/%m/%Y %H:%M:%S",
)

logging.info("Backup completed")
```

Possible output:

```text
27/10/2026 16:30:00 | INFO | Backup completed
```

---

# Part 3 — Custom Logging: Logger, Handler, Formatter, Filter

Custom logging means deciding where logs go, which messages are accepted, and how each message looks.

The main parts are:

1. **Logger** — your code sends log records to it.
2. **Handler** — decides where accepted records are sent, such as a terminal or file.
3. **Formatter** — controls the displayed layout.
4. **Filter** — optionally accepts or rejects records according to custom rules.

The flow is:

```text
Your code
   |
   v
Logger
   |
   v
Handler -----> Formatter -----> Terminal or file
   |
   v
Optional filters control which records pass
```

## 1. A custom logger that writes to both the console and a file

This is a useful pattern for a backup or monitoring program.

```python
import logging
from pathlib import Path

LOG_DIR = Path("logs")
LOG_DIR.mkdir(parents=True, exist_ok=True)

logger = logging.getLogger("backup_tool")
logger.setLevel(logging.DEBUG)

# Avoid adding duplicate handlers if this setup code runs more than once.
if not logger.handlers:
    # Display INFO and above on the console.
    console_handler = logging.StreamHandler()
    console_handler.setLevel(logging.INFO)

    # Record DEBUG and above in the file.
    file_handler = logging.FileHandler(
        LOG_DIR / "backup.log",
        encoding="utf-8",
    )
    file_handler.setLevel(logging.DEBUG)

    formatter = logging.Formatter(
        fmt="%(asctime)s | %(levelname)s | %(name)s | %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S",
    )

    console_handler.setFormatter(formatter)
    file_handler.setFormatter(formatter)

    logger.addHandler(console_handler)
    logger.addHandler(file_handler)

logger.debug("Detailed diagnostic saved to the log file")
logger.info("Backup process started")
logger.warning("A file was skipped")
logger.error("Could not back up a file")
```

What happens?

- The logger's level allows `DEBUG` and above.
- The console handler emits `INFO` and above.
- The file handler emits `DEBUG` and above.
- Both handlers use the same formatter.
- The log file is created inside the `logs` directory.

**Important:** A record must pass the logger's level and the handler's level to be emitted. Handler levels can be different, which lets you keep detailed diagnostics in a file without cluttering the console.

## 2. Useful `Logger` methods and attributes

| Method/attribute | Purpose |
|---|---|
| `logging.getLogger(name)` | Get or create a named logger |
| `logger.setLevel(level)` | Set the logger's minimum severity |
| `logger.debug(msg, *args)` | Log a debug message |
| `logger.info(msg, *args)` | Log an informational event |
| `logger.warning(msg, *args)` | Log a warning |
| `logger.error(msg, *args)` | Log an error |
| `logger.critical(msg, *args)` | Log a critical error |
| `logger.exception(msg, *args)` | Log an error with current exception information |
| `logger.log(level, msg, *args)` | Log using a numeric or named level value |
| `logger.isEnabledFor(level)` | Check whether the logger would process that level |
| `logger.addHandler(handler)` | Attach an output handler |
| `logger.removeHandler(handler)` | Remove an attached handler |
| `logger.addFilter(filter)` | Attach a filter |
| `logger.removeFilter(filter)` | Remove a filter |
| `logger.propagate` | Controls whether records are passed to ancestor loggers |
| `logger.handlers` | List of handlers attached directly to this logger |
| `logger.name` | Logger's name |
| `logger.parent` | Parent logger in the hierarchy, when applicable |

Prefer standard methods rather than manually changing internal attributes.

## 3. Useful handlers

| Handler | Purpose |
|---|---|
| `logging.StreamHandler()` | Sends logs to a stream, usually the console |
| `logging.FileHandler("app.log")` | Writes logs to a file |
| `logging.handlers.RotatingFileHandler` | Rotates a file after it reaches a size limit |
| `logging.handlers.TimedRotatingFileHandler` | Rotates a file on a time schedule |
| `logging.handlers.QueueHandler` | Sends records to a queue, useful in some multi-thread/process designs |
| `logging.handlers.QueueListener` | Reads queued records and sends them to handlers |
| `logging.NullHandler()` | Discards records; useful for library authors who do not want to configure output |

Import handlers from `logging.handlers` when needed:

```python
from logging.handlers import RotatingFileHandler
```

## 4. Rotate a log file by size

A log file can grow indefinitely. A rotating handler keeps a limited number of old log files.

```python
import logging
from logging.handlers import RotatingFileHandler

logger = logging.getLogger("monitor")
logger.setLevel(logging.INFO)

handler = RotatingFileHandler(
    "monitor.log",
    maxBytes=1_000_000,  # About 1 MB
    backupCount=3,       # Keep three rotated files
    encoding="utf-8",
)

handler.setFormatter(logging.Formatter(
    "%(asctime)s | %(levelname)s | %(message)s"
))

logger.addHandler(handler)

logger.info("Monitoring started")
```

When the file reaches the configured size, it rotates and retains a limited number of backups. If this setup can run repeatedly in one process, guard against adding the same handler more than once.

## 5. Use `logger.exception()` for errors

```python
import logging

logger = logging.getLogger(__name__)

try:
    with open("missing.txt", "r", encoding="utf-8") as file:
        data = file.read()
except OSError:
    logger.exception("Could not read the file")
```

This records the error message and traceback. Avoid logging the same exception repeatedly at multiple layers unless each log adds useful context.

## 6. Custom filters

A filter can decide whether a record should pass through.

```python
import logging

class OnlyBackupMessages(logging.Filter):
    def filter(self, record):
        return "backup" in record.getMessage().lower()

logger = logging.getLogger("my_app")
logger.setLevel(logging.DEBUG)

handler = logging.StreamHandler()
handler.addFilter(OnlyBackupMessages())
handler.setFormatter(logging.Formatter("%(levelname)s: %(message)s"))

logger.addHandler(handler)

logger.info("Backup started")     # Displayed
logger.info("Network connected")  # Filtered out
```

The `filter()` method returns a truthy value to allow a record through, or a false value to reject it. Filters are useful when logs need more selective handling than levels alone provide.

## 7. Avoid duplicate logs

If the same message appears twice, common causes include:
- The same handler was added more than once.
- A named logger has a handler and also propagates to a root logger with a similar handler.

For a standalone script, a guarded setup can help:

```python
if not logger.handlers:
    logger.addHandler(handler)
```

This is a simple safeguard, not a universal logging-configuration strategy. In larger applications, configure logging once at the application entry point and let imported modules use `logging.getLogger(__name__)`.

## 8. Logging best practices

- Use `DEBUG` for details useful during troubleshooting.
- Use `INFO` for important normal events.
- Use `WARNING` for unusual conditions that do not necessarily stop the task.
- Use `ERROR` when an operation fails.
- Use `logger.exception()` inside an exception handler when a traceback is useful.
- Prefer named loggers in modules.
- Configure logging once near the program's entry point.
- Use UTF-8 for log files that may contain non-ASCII text.
- Consider rotation for long-running applications.
- Do not log passwords, authentication tokens, private keys, or sensitive file contents.
- Avoid building log messages eagerly when expensive formatting is unnecessary. Prefer `logger.debug("Processed %d files", count)` over `logger.debug(f"Processed {count} files")` when the message may be filtered out.
- In multi-process applications, avoid having many processes independently rotate the same log file. Consider a queue-based logging design or another centralized logging approach.

---

# Part 4 — `datetime`: The Straightforward Approach

## 1. What is `datetime`?

The `datetime` module handles dates and times.

```python
from datetime import date, time, datetime, timedelta, timezone
```

Think of the main types this way:

| Type | Represents |
|---|---|
| `date` | Calendar date: year, month, day |
| `time` | Time of day: hour, minute, second, microsecond |
| `datetime` | Date and time together |
| `timedelta` | A duration or difference between dates/times |
| `timezone` | A fixed UTC offset, such as UTC or UTC+02:00 |
| `zoneinfo.ZoneInfo` | A named time zone with historical and daylight-saving rules |

## 2. Get today's date

```python
from datetime import date

today = date.today()

print(today)
print(today.year)
print(today.month)
print(today.day)
```

The printed date normally uses the ISO form `YYYY-MM-DD`.

## 3. Get the current local date and time

```python
from datetime import datetime

now = datetime.now()

print(now)
print("Year:", now.year)
print("Month:", now.month)
print("Day:", now.day)
print("Hour:", now.hour)
print("Minute:", now.minute)
print("Second:", now.second)
```

`datetime.now()` without a time-zone argument returns a **naive datetime**: it does not carry explicit time-zone information. It reflects the system's local clock.

For new applications where time-zone correctness matters, prefer an aware datetime, as explained below.

## 4. Get the current UTC time — recommended for many applications

```python
from datetime import datetime, timezone

now_utc = datetime.now(timezone.utc)

print(now_utc)
```

This returns an **aware datetime** with UTC time-zone information.

Prefer `datetime.now(timezone.utc)` over the older `datetime.utcnow()`, which returns a naive datetime and is deprecated in modern Python.

Why UTC is useful:
- Consistent timestamps across systems.
- Easier comparison of events from different time zones.
- Good for logs, databases, and event records when stored with time-zone awareness.

Convert to local time for display when needed.

## 5. Create a specific date or datetime

```python
from datetime import date, datetime

birthday = date(2000, 5, 12)
meeting = datetime(2026, 10, 27, 14, 30, 0)

print(birthday)
print(meeting)
```

Constructor argument order:

```python
date(year, month, day)

datetime(year, month, day, hour, minute, second, microsecond)
```

Months are numbered from `1` to `12`; hours use the 24-hour clock from `0` to `23`.

## 6. Add or subtract time with `timedelta`

```python
from datetime import datetime, timedelta, timezone

now = datetime.now(timezone.utc)

tomorrow = now + timedelta(days=1)
in_two_hours = now + timedelta(hours=2)
last_week = now - timedelta(days=7)

print("Now:", now)
print("Tomorrow:", tomorrow)
print("In two hours:", in_two_hours)
print("Last week:", last_week)
```

Supported common arguments include:
- `days`
- `seconds`
- `microseconds`
- `milliseconds`
- `minutes`
- `hours`
- `weeks`

`timedelta` represents a duration, not a calendar rule such as “the same local time next month.” Calendar-month calculations need separate logic because months have different lengths.

## 7. Calculate the difference between two dates/times

```python
from datetime import datetime, timezone

start = datetime(2026, 10, 20, tzinfo=timezone.utc)
end = datetime(2026, 10, 27, tzinfo=timezone.utc)

difference = end - start

print(difference)          # 7 days, 0:00:00
print(difference.days)     # 7
print(difference.seconds)  # Remaining seconds after whole days
print(difference.total_seconds())
```

For a duration that may include days, use `total_seconds()` if you need the total number of seconds. The `.seconds` attribute is only the remainder after whole days; it is not the total duration in seconds.

## 8. Compare dates/times

```python
from datetime import datetime, timezone

first = datetime(2026, 10, 27, 9, 0, tzinfo=timezone.utc)
second = datetime(2026, 10, 27, 10, 0, tzinfo=timezone.utc)

print(first < second)  # True
print(first == second) # False
```

Do not compare a naive datetime with an aware datetime; ordering comparisons raise `TypeError`. Make sure the values use an intentional, consistent time-zone approach.

---

# Part 5 — Date/Time Formatting: `%Y`, `%m`, `%d`, and More

The main formatting methods are:

- `strftime(format)`: datetime/date/time object → formatted string.
- `strptime(text, format)`: formatted string → parsed datetime.
- `isoformat()`: produce an ISO 8601-style representation.
- `fromisoformat(text)`: parse supported ISO-formatted strings.

## 1. `strftime()` — display a date in the format you want

```python
from datetime import datetime

now = datetime.now()

print(now.strftime("%Y-%m-%d"))
print(now.strftime("%d/%m/%Y"))
print(now.strftime("%H:%M:%S"))
print(now.strftime("%d/%m/%Y %H:%M:%S"))
```

Possible output:

```text
2026-10-27
27/10/2026
16:30:00
27/10/2026 16:30:00
```

The exact values depend on the current date/time.

## 2. The most useful date/time directives

| Directive | Meaning | Example |
|---|---|---|
| `%Y` | Four-digit year | `2026` |
| `%y` | Two-digit year | `26` |
| `%m` | Month, `01`–`12` | `10` |
| `%B` | Full month name | `October` |
| `%b` | Abbreviated month | `Oct` |
| `%d` | Day of month, zero-padded | `07` |
| `%-d` | Day without leading zero on many Unix systems | `7` |
| `%A` | Full weekday name | `Tuesday` |
| `%a` | Abbreviated weekday | `Tue` |
| `%w` | Weekday number, Sunday `0` | `2` |
| `%H` | Hour, 24-hour clock | `16` |
| `%I` | Hour, 12-hour clock | `04` |
| `%p` | Locale-specific AM/PM | `PM` |
| `%M` | Minute | `30` |
| `%S` | Second | `45` |
| `%f` | Microsecond, six digits | `123456` |
| `%j` | Day of year, `001`–`366` | `300` |
| `%z` | UTC offset, e.g. `+0100` | `+0100` |
| `%Z` | Time-zone name/abbreviation when available | `UTC` |
| `%c` | Locale's date and time representation | Varies |
| `%x` | Locale's date representation | Varies |
| `%X` | Locale's time representation | Varies |
| `%%` | A literal percent sign | `%` |

**Portability note:** `%-d` and similar no-leading-zero directives are not supported consistently on every platform, particularly Windows. For portable code, `%d` is safer; remove the leading zero separately if you need that presentation.

## 3. Understand the format by reading it

```python
format_string = "%d/%m/%Y %H:%M:%S"
```

Read it as:

```text
%d  → day
/   → literal slash
%m  → month
/   → literal slash
%Y  → four-digit year
(space)
%H  → hour
:   → literal colon
%M  → minute
:   → literal colon
%S  → second
```

The characters such as `/`, `-`, spaces, and `:` are ordinary text. The directives beginning with `%` are special formatting instructions.

## 4. Common formats to copy

```python
from datetime import datetime

now = datetime.now()

print(now.strftime("%Y-%m-%d"))              # 2026-10-27
print(now.strftime("%d/%m/%Y"))              # 27/10/2026
print(now.strftime("%H:%M:%S"))              # 16:30:45
print(now.strftime("%I:%M:%S %p"))           # 04:30:45 PM
print(now.strftime("%A, %d %B %Y"))          # Tuesday, 27 October 2026
print(now.strftime("%Y%m%d_%H%M%S"))         # 20261027_163045
print(now.strftime("%Y-%m-%dT%H:%M:%S"))     # 2026-10-27T16:30:45
```

The output shown is illustrative.

For filenames, avoid characters that are invalid or awkward in file paths. A format such as `%Y%m%d_%H%M%S` works well across common operating systems.

## 5. `strptime()` — turn text into a datetime

The format must match the input text.

```python
from datetime import datetime

text = "27/10/2026 16:30"

parsed = datetime.strptime(text, "%d/%m/%Y %H:%M")

print(parsed)
print(parsed.year)
print(parsed.month)
print(parsed.day)
```

Examples:

```python
datetime.strptime("2026-10-27", "%Y-%m-%d")
datetime.strptime("27/10/2026", "%d/%m/%Y")
datetime.strptime("04:30 PM", "%I:%M %p")
```

If the text doesn't match the format, Python raises `ValueError`. Handle that if the text comes from a user or external file.

## 6. ISO 8601 — a good format for data exchange

```python
from datetime import datetime, timezone

now = datetime.now(timezone.utc)

text = now.isoformat()
print(text)
```

Example:

```text
2026-10-27T15:30:00+00:00
```

An ISO-style timestamp makes the date, time, and UTC offset explicit. The exact representation can include fractional seconds depending on the datetime value.

To parse a compatible ISO string:

```python
from datetime import datetime

text = "2026-10-27T15:30:00+00:00"
value = datetime.fromisoformat(text)

print(value)
```

For interchange, logs, or stored events, ISO 8601 is usually preferable to ambiguous dates such as `10/11/2026`, which could mean different dates in different locales.

---

# Part 6 — Time Zones: The Best Approach

## 1. What is a time zone?

A time zone describes the local clock rules for a region. Some regions change their UTC offset during the year due to daylight-saving time; others do not.

UTC is a stable reference. Local times are useful for display and for tasks tied to a person's region.

A timestamp such as:

```text
2026-10-27 15:30:00+00:00
```

means 15:30 at UTC offset `+00:00`.

A timestamp such as:

```text
2026-10-27 16:30:00+01:00
```

represents the same instant, expressed at UTC offset `+01:00`.

## 2. Naive versus aware datetimes

A **naive datetime** has no attached time-zone information:

```python
from datetime import datetime

local_value = datetime.now()
print(local_value)
print(local_value.tzinfo)  # None
```

An **aware datetime** includes time-zone information:

```python
from datetime import datetime, timezone

utc_value = datetime.now(timezone.utc)

print(utc_value)
print(utc_value.tzinfo)  # UTC
```

For systems that record events, exchange timestamps, or compare events across time zones, prefer aware datetimes.

A naive datetime is not automatically UTC. It may represent local wall-clock time or some other convention agreed upon by your application.

## 3. Fixed offsets versus named time zones

### Fixed UTC offset

```python
from datetime import datetime, timezone, timedelta

fixed_plus_one = timezone(timedelta(hours=1))

value = datetime(2026, 10, 27, 16, 30, tzinfo=fixed_plus_one)

print(value)
```

A fixed offset is useful when the offset itself is what matters. It does not include a region's historical or daylight-saving rules.

### Named time zone using `zoneinfo`

For regional time zones, use `zoneinfo.ZoneInfo`:

```python
from datetime import datetime
from zoneinfo import ZoneInfo

london_time = datetime.now(ZoneInfo("Europe/London"))

print(london_time)
```

`ZoneInfo` uses the IANA time-zone database, including regional rules such as daylight-saving transitions. On systems without an available time-zone database, installing the `tzdata` package may be necessary.

Use a region name rather than hardcoding an offset when the local civil-time rules matter.

## 4. Work with the Africa/Douala time zone

For Cameroon, the IANA time-zone name is `Africa/Douala`.

```python
from datetime import datetime
from zoneinfo import ZoneInfo

douala = datetime.now(ZoneInfo("Africa/Douala"))

print(douala)
print(douala.strftime("%Y-%m-%d %H:%M:%S %Z %z"))
```

This uses the named regional time zone rather than assuming that a manually chosen UTC offset will always describe the region correctly.

## 5. Convert a datetime from one time zone to another

Use `.astimezone()` to represent the same instant in another time zone.

```python
from datetime import datetime, timezone
from zoneinfo import ZoneInfo

utc_now = datetime.now(timezone.utc)

douala_time = utc_now.astimezone(ZoneInfo("Africa/Douala"))
london_time = utc_now.astimezone(ZoneInfo("Europe/London"))

print("UTC:", utc_now)
print("Douala:", douala_time)
print("London:", london_time)
```

These values refer to the same instant, displayed using different time-zone rules.

Do not confuse conversion with replacement:

- `value.astimezone(zone)` converts the same instant to another zone.
- `value.replace(tzinfo=zone)` attaches or replaces time-zone metadata without converting the clock fields. Use `replace(tzinfo=...)` only when you deliberately know that the existing clock fields should be interpreted in that zone.

## 6. Convert a UTC datetime to local time

```python
from datetime import datetime, timezone
from zoneinfo import ZoneInfo

utc_value = datetime(2026, 10, 27, 15, 30, tzinfo=timezone.utc)

local_value = utc_value.astimezone(ZoneInfo("Africa/Douala"))

print(local_value)
```

This is the recommended pattern when you have an unambiguous UTC timestamp and need to display it locally.

## 7. Parse an ISO timestamp and convert it

```python
from datetime import datetime
from zoneinfo import ZoneInfo

text = "2026-10-27T15:30:00+00:00"

parsed = datetime.fromisoformat(text)
douala_time = parsed.astimezone(ZoneInfo("Africa/Douala"))

print("Parsed:", parsed)
print("Douala:", douala_time)
```

Because the input includes `+00:00`, it identifies an offset-aware datetime.

If input text has no offset, you must know what time zone it is intended to represent before interpreting it. Do not silently assume UTC unless that is your documented data convention.

## 8. Best practices for time zones

1. Use aware datetimes for event timestamps.
2. Use `datetime.now(timezone.utc)` when you need the current UTC time.
3. Store or exchange timestamps in a consistent, unambiguous form, commonly UTC with an offset.
4. Use `ZoneInfo("Region/City")` for regional local-time rules.
5. Convert to a user's local time for display.
6. Do not assume a fixed UTC offset represents a region correctly for all dates.
7. Avoid mixing naive and aware datetimes.
8. Do not use `replace(tzinfo=...)` as a time-zone conversion.
9. For scheduling future local events, keep the intended regional time zone because future UTC offsets may change under local rules.
10. Be aware that some local clock times can be ambiguous or nonexistent around daylight-saving transitions.

---

# Part 7 — Logging with UTC or a Specific Time Zone

Python's standard logging formatter normally formats `asctime` using local time. You can customize the formatter to use UTC or a named time zone.

## 1. Simple UTC log timestamps

For a simple UTC setup, use `logging.Formatter.converter = time.gmtime` on the formatter instance:

```python
import logging
import time

handler = logging.StreamHandler()

formatter = logging.Formatter(
    "%(asctime)s | %(levelname)s | %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S UTC",
)

formatter.converter = time.gmtime
handler.setFormatter(formatter)

logger = logging.getLogger("utc_example")
logger.setLevel(logging.INFO)
logger.addHandler(handler)
logger.propagate = False

logger.info("This timestamp is formatted in UTC")
```

Setting `converter` on the formatter instance avoids changing the behavior of every formatter in the application.

For a standalone script, ensure this logger setup is performed only once, or clear/replace handlers deliberately if rerunning setup.

## 2. Custom formatter using `zoneinfo`

For a named regional time zone, you can subclass `logging.Formatter` and convert the log record's timestamp.

```python
import logging
from datetime import datetime
from zoneinfo import ZoneInfo

class TimezoneFormatter(logging.Formatter):
    def __init__(self, fmt=None, datefmt=None, tz_name="Africa/Douala"):
        super().__init__(fmt=fmt, datefmt=datefmt)
        self.tz = ZoneInfo(tz_name)

    def formatTime(self, record, datefmt=None):
        value = datetime.fromtimestamp(record.created, self.tz)

        if datefmt:
            return value.strftime(datefmt)

        return value.isoformat(timespec="seconds")

logger = logging.getLogger("regional_time")
logger.setLevel(logging.INFO)

handler = logging.StreamHandler()
handler.setFormatter(TimezoneFormatter(
    "%(asctime)s | %(levelname)s | %(message)s",
    "%Y-%m-%d %H:%M:%S %z",
    tz_name="Africa/Douala",
))

logger.addHandler(handler)
logger.propagate = False

logger.info("Backup completed")
```

This is useful when you specifically want log timestamps in a named region's time zone. For multi-region systems, UTC logs are often simpler for correlation, with local conversion handled in reports or interfaces.

---

# Part 8 — A Practical Example: File Backup with Logging and Timestamps

This example ties together `pathlib`, `shutil`, `datetime`, and logging. It creates one timestamped backup when run.

```python
import logging
import shutil
from datetime import datetime, timezone
from pathlib import Path

SOURCE = Path("report.txt")
BACKUP_DIR = Path("backups")
LOG_DIR = Path("logs")

LOG_DIR.mkdir(parents=True, exist_ok=True)
BACKUP_DIR.mkdir(parents=True, exist_ok=True)

logging.basicConfig(
    filename=LOG_DIR / "backup.log",
    level=logging.DEBUG,
    format="%(asctime)s | %(levelname)s | %(name)s | %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
    encoding="utf-8",
)

logger = logging.getLogger("backup")

def create_backup(source: Path) -> Path:
    if not source.is_file():
        raise FileNotFoundError(f"Source file does not exist: {source}")

    # Use UTC for a clear, consistent timestamp.
    stamp = datetime.now(timezone.utc).strftime("%Y%m%dT%H%M%SZ")

    destination = BACKUP_DIR / f"{source.stem}_{stamp}{source.suffix}"

    shutil.copy2(source, destination)
    logger.info("Copied %s to %s", source, destination)

    return destination

def main():
    try:
        backup_path = create_backup(SOURCE)
    except (OSError, shutil.Error):
        logger.exception("Backup failed")
        raise SystemExit(1)

    print(f"Backup created: {backup_path}")

if __name__ == "__main__":
    logger.info("Backup program started")
    main()
```

Notes:
- Create `report.txt` before running the example.
- The log file records events in the system's local time because `basicConfig()` uses the default formatter. The filename's timestamp is UTC, marked by `Z`.
- If you need the log timestamp itself to be UTC, use the UTC formatter example above or configure a custom formatter.
- This script creates a backup when invoked; it does not monitor for changes continuously.
- If multiple backups are created within the same second, a filename collision is possible. Add microseconds or another unique identifier if that matters.

---

# Part 9 — Quick Reference Cheat Sheet

## Logging

```python
logger.debug("Details")
logger.info("Normal event")
logger.warning("Unexpected situation")
logger.error("Operation failed")
logger.critical("Severe failure")

try:
    ...
except Exception:
    logger.exception("Operation raised an exception")
```

## Datetime

```python
from datetime import date, datetime, timedelta, timezone

today = date.today()
local_now = datetime.now()
utc_now = datetime.now(timezone.utc)
tomorrow = utc_now + timedelta(days=1)

formatted = utc_now.strftime("%Y-%m-%d %H:%M:%S %z")
parsed = datetime.strptime("27/10/2026", "%d/%m/%Y")
```

## Time zones

```python
from datetime import datetime, timezone
from zoneinfo import ZoneInfo

utc_now = datetime.now(timezone.utc)
douala_now = utc_now.astimezone(ZoneInfo("Africa/Douala"))
london_now = utc_now.astimezone(ZoneInfo("Europe/London"))
```

## Formatting

```python
"%Y-%m-%d"             # Year-month-day
"%d/%m/%Y"             # Day/month/year
"%H:%M:%S"             # 24-hour time
"%I:%M %p"             # 12-hour time
"%A, %d %B %Y"         # Tuesday, 27 October 2026
"%Y%m%d_%H%M%S"        # Filename-friendly timestamp
"%Y-%m-%dT%H:%M:%S%z"  # Date, time, and numeric UTC offset
```

## The key rules to remember

- `logging` records events; `print()` is mainly for direct output and simple debugging.
- Logging fields use `%(name)s` syntax.
- Date/time directives use `%Y`, `%m`, `%d`, `%H`, `%M`, `%S`, and similar codes.
- `strftime()` formats date/time objects into strings.
- `strptime()` parses strings into date/time objects.
- Use `datetime.now(timezone.utc)` for an aware current UTC datetime.
- Use `ZoneInfo("Africa/Douala")` for Cameroon local-time rules.
- Use `.astimezone()` to convert the same instant between time zones.
- Use `time.perf_counter()` to measure durations.
- Use log rotation for long-running applications.
- Never log secrets.

---

# Official Python Documentation

- Logging: https://docs.python.org/3/library/logging.html
- Logging handlers: https://docs.python.org/3/library/logging.handlers.html
- Logging cookbook: https://docs.python.org/3/howto/logging-cookbook.html
- `datetime`: https://docs.python.org/3/library/datetime.html
- `zoneinfo`: https://docs.python.org/3/library/zoneinfo.html
- `time`: https://docs.python.org/3/library/time.html
