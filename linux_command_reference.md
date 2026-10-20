# Linux Command Reference: `awk`, `sort`, `uniq`, `wc`, `cut`, `find`, `locate`, `whereis`, `which`, and Cron

A practical, copyable reference for Ubuntu, Debian, WSL, and most
GNU/Linux systems.

> **Important:** Command options can vary between GNU/Linux, BSD, and
> macOS. This guide primarily documents GNU/Linux tools. For the exact
> options installed on your system, run `man COMMAND` or
> `COMMAND --help`.

------------------------------------------------------------------------

## Table of contents

1.  [General command-help habits](#1-general-command-help-habits)
2.  [`awk`](#2-awk)
3.  [`sort`](#3-sort)
4.  [`uniq`](#4-uniq)
5.  [`wc`](#5-wc)
6.  [`cut`](#6-cut)
7.  [`find`](#7-find)
8.  [`locate`](#8-locate)
9.  [`whereis`](#9-whereis)
10. [`which`](#10-which)
11. [Cron jobs and `crontab`](#11-cron-jobs-and-crontab)
12. [Combining commands into useful
    pipelines](#12-combining-commands-into-useful-pipelines)
13. [Practice exercises](#13-practice-exercises)

------------------------------------------------------------------------

## 1. General command-help habits

``` bash
command --help       # Quick summary of common options
man command          # Full manual page
info command         # Info documentation, if installed
type command         # Shows how the shell interprets a command
command -v command  # Finds the command using the shell's lookup rules
```

Examples:

``` bash
awk --help
man awk
find --help
man crontab
```

In a manual page: - Press `/word` to search for `word`. - Press `n` to
move to the next match. - Press `q` to quit.

Use `man 5 crontab` for the crontab file format on systems that provide
that manual section.

------------------------------------------------------------------------

## 2. `awk`

### 2.1 What `awk` does

`awk` reads input a record at a time (normally one line at a time),
splits each record into fields, and lets you select, calculate,
transform, and print data.

Basic form:

``` bash
awk 'pattern { action }' file
```

-   **pattern** decides which records match.
-   **action** says what to do with matching records.
-   If the pattern is omitted, the action runs for every record.
-   If the action is omitted, a matching record is printed.

Example file `people.txt`:

``` text
Ada 25 London
Ben 31 Douala
Chinwe 28 Lagos
David 42 Buea
```

``` bash
awk '{ print }' people.txt          # Print every line
awk '{ print $1 }' people.txt       # Print field 1
awk '{ print $1, $3 }' people.txt   # Print fields 1 and 3
awk '$2 > 30 { print $1, $2 }' people.txt
```

### 2.2 Fields and built-in variables

  Variable          Meaning
  ----------------- --------------------------------------------
  `$0`              Entire current record/line
  `$1`, `$2`, ...   First, second, and later fields
  `NF`              Number of fields in current record
  `NR`              Record/line number across all input files
  `FNR`             Record/line number within the current file
  `FS`              Input field separator
  `OFS`             Output field separator
  `RS`              Input record separator
  `ORS`             Output record separator
  `FILENAME`        Name of the current input file
  `ARGC`            Number of command-line arguments
  `ARGV`            Array of command-line arguments

Examples:

``` bash
awk '{ print NR, $0 }' people.txt       # Add line number
awk '{ print NF, $0 }' people.txt       # Show field count
awk '{ print $NF }' people.txt          # Print final field
awk 'NF >= 3 { print $1, $3 }' people.txt
awk 'END { print "Total records:", NR }' people.txt
```

### 2.3 `FS` and `OFS`

`FS` determines how input is divided into fields. `OFS` determines what
separates fields printed with commas in `print`.

``` bash
awk 'BEGIN { FS = ":"; OFS = " | " } { print $1, $3 }' /etc/passwd
```

This reads colon-separated records and prints field 1 and field 3
separated by `|`.

Set them with `-F` or `-v`:

``` bash
awk -F: '{ print $1, $3 }' /etc/passwd
awk -F, 'BEGIN { OFS = "\t" } { print $1, $2 }' data.csv
awk -v FS=: -v OFS=' -> ' '{ print $1, $3 }' /etc/passwd
```

**Whitespace detail:** The default `FS` is a single space, which in
`awk` means runs of whitespace (spaces and tabs) are treated as
separators. It does not behave exactly like setting `FS` to a literal
single space in every context.

### 2.4 `BEGIN` and `END`

-   `BEGIN { ... }` runs before input records are read.
-   `END { ... }` runs after all input records have been processed.

``` bash
awk 'BEGIN { print "Name Age" } { print $1, $2 } END { print "Finished" }' people.txt
```

A small report:

``` bash
awk '
BEGIN {
    print "----- AGE REPORT -----"
    total = 0
}
{
    total += $2
    count++
}
END {
    if (count > 0)
        printf "People: %d\nAverage age: %.2f\n", count, total / count
    else
        print "No records"
}' people.txt
```

### 2.5 Conditions and pattern matching

``` bash
awk '$2 >= 30 { print }' people.txt
awk '$1 == "Ada" { print }' people.txt
awk '$3 ~ /Douala|Buea/ { print }' people.txt
awk '$3 !~ /London/ { print }' people.txt
awk 'NR == 1 { print "First line:", $0 }' people.txt
awk 'NR >= 2 && NR <= 4 { print }' people.txt
awk 'NF == 3 { print }' people.txt
```

Operators: - `==`, `!=`: equal, not equal - `>`, `<`, `>=`, `<=`:
comparisons - `&&`: AND - `||`: OR - `!`: NOT - `~`: matches regular
expression - `!~`: does not match regular expression

### 2.6 Arithmetic and assignment

``` bash
awk '{ total += $2 } END { print total }' people.txt
awk '{ print $1, $2 + 5 }' people.txt
awk '{ print $1, $2 * 12 }' people.txt
```

Assignment operators include `=`, `+=`, `-=`, `*=`, `/=`, and `%=`.
Increment/decrement operators `++` and `--` are also available.

### 2.7 Formatting with `printf`

``` bash
awk '{ printf "%-10s %3d\n", $1, $2 }' people.txt
```

Useful format specifiers: - `%s`: string - `%d`: integer - `%f`:
floating-point number - `%.2f`: floating-point number to two decimal
places - `\n`: newline - `\t`: tab

### 2.8 String functions

``` bash
awk '{ print length($1) }' people.txt
awk '{ print toupper($1), tolower($3) }' people.txt
awk '{ print substr($1, 1, 2) }' people.txt
awk '{ print gsub(/a/, "A", $0), $0 }' people.txt
awk '{ print index($0, "ou") }' people.txt
awk '{ print split($0, parts, /[[:space:]]+/), parts[1] }' people.txt
```

Common functions: - `length(s)`: length of string `s`, or current record
if omitted - `tolower(s)`, `toupper(s)`: change case -
`substr(s, start, length)`: extract part of a string - `index(s, t)`:
position of substring `t`, or zero if absent -
`split(s, array, separator)`: split into array elements; returns count -
`sub(regex, replacement, target)`: replace first match -
`gsub(regex, replacement, target)`: replace all matches -
`match(s, regex)`: find a regex match - `sprintf(format, ...)`: return
formatted text

### 2.9 Arrays

`awk` arrays are associative: keys may be strings or numbers.

``` bash
awk '{ count[$3]++ } END { for (city in count) print city, count[city] }' people.txt
```

Array iteration order is not guaranteed. Sort output separately if order
matters.

### 2.10 Useful `awk` options

  Option            Purpose
  ----------------- ----------------------------------------------------
  `-F fs`           Set input field separator
  `-v name=value`   Set an awk variable before execution
  `-f script.awk`   Read program from a file
  `-e 'program'`    Supply program text; supported by GNU awk (`gawk`)
  `--version`       Show version for GNU awk
  `--help`          Show GNU awk help

Examples:

``` bash
awk -F: '{ print $1 }' /etc/passwd
awk -v limit=30 '$2 > limit { print $1, $2 }' people.txt
awk -f report.awk people.txt
```

**Safety tip:** Quote the awk program in single quotes when using a
shell, so the shell does not expand `$1`, `$2`, or other awk expressions
first.

------------------------------------------------------------------------

## 3. `sort`

### 3.1 What it does

`sort` orders lines from files or standard input.

``` bash
sort names.txt
sort < names.txt
printf '%s\n' pear apple mango | sort
sort names.txt -o sorted.txt
```

### 3.2 Common options

  -----------------------------------------------------------------------
  Option                              Meaning
  ----------------------------------- -----------------------------------
  `-b`                                Ignore leading blanks when sorting

  `-d`                                Dictionary order: consider blanks
                                      and alphanumeric characters

  `-f`                                Fold lowercase to uppercase for
                                      sorting

  `-g`                                General numeric sort; understands
                                      values such as scientific notation

  `-h`                                Human-readable numeric sort (`2K`,
                                      `1G`, etc.)

  `-i`                                Ignore nonprinting characters

  `-M`                                Sort by month names (`JAN` through
                                      `DEC`)

  `-n`                                Numeric sort

  `-r`                                Reverse the result

  `-R`                                Randomize order, grouping equal
                                      keys

  `-u`                                Output one line from each group of
                                      equal sort keys

  `-c`                                Check whether input is already
                                      sorted

  `-C`                                Check whether input is sorted,
                                      without reporting the first
                                      disorder

  `-m`                                Merge already-sorted files

  `-o FILE`                           Write output to `FILE`

  `-s`                                Stable sort: do not reorder lines
                                      equal under the sort keys

  `-t CHAR`                           Use `CHAR` as the field delimiter

  `-k KEYDEF`                         Sort using a key/field definition

  `-V`                                Version sort, e.g. `file2` before
                                      `file10`

  `--parallel=N`                      Use up to N parallel sort threads
                                      where supported

  `-T DIR`                            Use directory `DIR` for temporary
                                      files

  `-S SIZE`                           Set memory buffer size

  `--help`                            Display help

  `--version`                         Display version
  -----------------------------------------------------------------------

### 3.3 Alphabetical, reverse, and case-insensitive sorting

``` bash
sort names.txt
sort -r names.txt
sort -f names.txt
sort -u names.txt
```

`sort -u` removes lines that compare equal under the selected sort
settings. For a common deduplication workflow, `sort file | uniq` is
also used.

### 3.4 Numeric sorting

``` bash
printf '%s\n' 100 9 20 3 | sort
printf '%s\n' 100 9 20 3 | sort -n
```

Default text sorting and numeric sorting are different. Use `-n` for
ordinary numeric values.

For sizes with suffixes:

``` bash
printf '%s\n' 2K 500 1M 10K | sort -h
```

### 3.5 Sort using a field

Example `scores.txt`:

``` text
Ada 80
Ben 95
Chinwe 72
David 88
```

``` bash
sort -k2,2n scores.txt       # Numeric sort by field 2
sort -k2,2nr scores.txt      # Numeric descending by field 2
sort -k1,1 scores.txt        # Sort by field 1
```

A key such as `-k2,2n` means start at field 2, end at field 2, and
compare numerically.

For colon-separated data:

``` bash
sort -t: -k3,3n /etc/passwd
```

### 3.6 Check or merge sorted files

``` bash
sort -c names.txt
sort -C names.txt
sort -m part1.sorted part2.sorted
sort names.txt -o names.txt
```

`sort -o names.txt names.txt` is safe for in-place sorting with GNU
`sort`; avoid casually using shell redirection like
`sort names.txt > names.txt`, because the shell truncates the output
file before `sort` reads it.

------------------------------------------------------------------------

## 4. `uniq`

### 4.1 What it does

`uniq` removes or reports **adjacent** repeated lines. It does not
normally find duplicates that are separated by other lines.

``` bash
uniq names.txt
sort names.txt | uniq
sort -u names.txt
```

### 4.2 Options

  -----------------------------------------------------------------------
  Option                              Meaning
  ----------------------------------- -----------------------------------
  `-c`                                Prefix each output line with its
                                      occurrence count

  `-d`                                Print only repeated lines

  `-D`                                Print all lines in each repeated
                                      group (GNU extension)

  `-u`                                Print only lines that are not
                                      repeated

  `-i`                                Ignore case when comparing

  `-f N`                              Ignore the first N fields when
                                      comparing

  `-s N`                              Ignore the first N characters when
                                      comparing

  `-w N`                              Compare no more than N characters

  `--all-repeated[=METHOD]`           Print all lines from repeated
                                      groups; GNU option

  `--group[=METHOD]`                  Separate groups in output; GNU
                                      option

  `--help`                            Show help

  `--version`                         Show version
  -----------------------------------------------------------------------

### 4.3 Examples

``` bash
uniq names.txt
sort names.txt | uniq
sort names.txt | uniq -c
sort names.txt | uniq -d
sort names.txt | uniq -u
sort -f names.txt | uniq -i
```

Show the most frequent lines:

``` bash
sort access.log | uniq -c | sort -nr | head
```

**Important:** `uniq -c` counts each consecutive group. Sorting first
groups equal lines together.

------------------------------------------------------------------------

## 5. `wc`

### 5.1 What it does

`wc` counts lines, words, bytes, and characters.

``` bash
wc file.txt
```

Typical output has counts for lines, words, bytes, followed by the
filename.

### 5.2 Options

  Option        Meaning
  ------------- --------------------------------------------------
  `-l`          Count lines
  `-w`          Count words
  `-c`          Count bytes
  `-m`          Count characters according to the current locale
  `-L`          Print length of the longest line
  `--help`      Show help
  `--version`   Show version

Examples:

``` bash
wc file.txt
wc -l file.txt
wc -w file.txt
wc -c file.txt
wc -m file.txt
wc -L file.txt
wc -l *.txt
cat file.txt | wc -w
```

A line count counts newline characters; a final line without a
terminating newline may affect the result you expect. `-c` counts bytes,
while `-m` counts characters, which can differ for UTF-8 text.

------------------------------------------------------------------------

## 6. `cut`

### 6.1 What it does

`cut` extracts selected fields, characters, or bytes from each input
line. It is useful for simple delimited text; it is not a full CSV
parser.

### 6.2 Options

  -----------------------------------------------------------------------
  Option                              Meaning
  ----------------------------------- -----------------------------------
  `-f LIST`                           Select fields

  `-d DELIM`                          Set field delimiter; default is TAB

  `-c LIST`                           Select character positions

  `-b LIST`                           Select byte positions

  `--complement`                      Select everything except the
                                      specified positions/fields

  `-s`                                Suppress lines that contain no
                                      delimiter when using fields

  `--output-delimiter=STRING`         Use a custom delimiter between
                                      output fields

  `--help`                            Show help

  `--version`                         Show version
  -----------------------------------------------------------------------

### 6.3 Field selection

Example `students.tsv`:

``` text
Ada 80  London
Ben 95  Douala
Chinwe  72  Lagos
```

``` bash
cut -f1 students.tsv
cut -f1,3 students.tsv
cut -f2- students.tsv
cut -f1-2 students.tsv
```

### 6.4 Change the delimiter

``` bash
cut -d: -f1 /etc/passwd
cut -d: -f1,3,6 /etc/passwd
cut -d, -f1,3 data.csv
```

For a line such as `Ada:25:London`, `cut -d: -f1,3` prints `Ada:London`.

``` bash
cut -d: --output-delimiter=' | ' -f1,3 /etc/passwd
```

### 6.5 Character and byte positions

``` bash
printf 'abcdef\n' | cut -c1-3       # abc
printf 'abcdef\n' | cut -c2,5       # be
printf 'abcdef\n' | cut -c3-        # cdef
printf 'abcdef\n' | cut -c-4       # abcd
printf 'abcdef\n' | cut -c1-6 --complement  # empty for this six-character input
```

Position-list syntax: - `N`: position N - `N-M`: positions N through M -
`N-`: position N through end - `-M`: start through position M - Separate
multiple selections with commas, such as `1,3,5-7`.

Use `-b` for byte positions. For multibyte UTF-8 text, character
positions and byte positions can differ.

### 6.6 Limitations

-   `cut -d,` splits on every comma, including commas inside quoted CSV
    fields.
-   For complex CSV, use a CSV-aware tool or a suitable `awk`/Python
    program.
-   With `-f`, lines without the delimiter are normally printed
    unchanged; `-s` suppresses them.

------------------------------------------------------------------------

## 7. `find`

### 7.1 What it does

`find` searches directory trees based on file name, type, size,
timestamps, permissions, owner, and other tests.

General form:

``` bash
find STARTING_PATH EXPRESSION
```

Examples:

``` bash
find .
find /etc -type f
find /home -name '*.txt'
```

Quote patterns such as `'*.txt'` so the shell does not expand them
before `find` sees them.

### 7.2 Common tests

  -----------------------------------------------------------------------
  Test                                Meaning
  ----------------------------------- -----------------------------------
  `-name PATTERN`                     Match filename using case-sensitive
                                      shell-style pattern

  `-iname PATTERN`                    Case-insensitive filename match

  `-type f`                           Regular file

  `-type d`                           Directory

  `-type l`                           Symbolic link

  `-type b`                           Block device

  `-type c`                           Character device

  `-type p`                           Named pipe

  `-type s`                           Socket

  `-empty`                            Empty file or directory

  `-size N`                           Match by size

  `-user NAME`                        Owned by user

  `-group NAME`                       Owned by group

  `-uid N`                            Match numeric user ID

  `-gid N`                            Match numeric group ID

  `-perm MODE`                        Match permissions

  `-readable`                         Readable by the current user (GNU
                                      find)

  `-writable`                         Writable by the current user (GNU
                                      find)

  `-executable`                       Executable by the current user (GNU
                                      find)

  `-links N`                          Number of hard links

  `-inum N`                           Inode number

  `-samefile FILE`                    Same inode as the specified file
                                      (GNU find)
  -----------------------------------------------------------------------

### 7.3 Name, type, and path searches

``` bash
find . -name '*.py'
find . -iname 'readme*'
find /var/log -type f -name '*.log'
find . -type d -name node_modules
find . -path '*/.git/*'
find . -not -name '*.txt'
```

Common logical operators: - `-a` / `-and`: AND; usually implied between
adjacent tests - `-o` / `-or`: OR - `!` / `-not`: NOT - `\( ... \)`:
group tests; backslashes protect parentheses from the shell

``` bash
find . -type f \( -name '*.jpg' -o -name '*.png' \)
find . -type f ! -name '*.tmp'
```

### 7.4 Size searches

Size suffixes: - `c`: bytes - `k`: 1024-byte units - `M`: mebibytes -
`G`: gibibytes

``` bash
find . -type f -size +100M
find . -type f -size -1k
find . -type f -size 10M
find . -type f -size +0c
```

For `-size`, `+N` means greater than the rounded size threshold, and
`-N` means less than it. Check `man find` for exact rounding semantics.

### 7.5 Time searches

  Test              Meaning
  ----------------- ------------------------------------------------
  `-mtime N`        Modification time in 24-hour periods
  `-mmin N`         Modification time in minutes
  `-atime N`        Last access in 24-hour periods
  `-amin N`         Last access in minutes
  `-ctime N`        Metadata/status-change time in 24-hour periods
  `-cmin N`         Metadata/status-change time in minutes
  `-newer FILE`     Modified more recently than FILE
  `-newermt DATE`   Modified after date/time text (GNU find)

Examples:

``` bash
find . -type f -mtime -1       # Modified less than about 24 hours ago
find . -type f -mtime +7       # Older than seven 24-hour periods
find . -type f -mmin -30       # Modified within about 30 minutes
find . -newermt '2026-01-01'
```

`-mtime`/`-mmin` use age ranges; they are not exact calendar-day
filters.

### 7.6 Permissions

``` bash
find . -type f -perm 644
find . -type f -perm -644
find . -type f -perm /111
find . -type f -perm -u+x
find . -type f -perm -g+w
```

-   `-perm 644`: exact permission mode.
-   `-perm -644`: all permission bits in `644` must be set.
-   `-perm /111`: at least one execute bit is set (GNU find).
-   Symbolic modes such as `-u+x` can be used.

### 7.7 Depth, filesystem boundaries, and pruning

  -----------------------------------------------------------------------
  Option                              Meaning
  ----------------------------------- -----------------------------------
  `-maxdepth N`                       Do not descend more than N levels
                                      (GNU find)

  `-mindepth N`                       Do not test starting paths until at
                                      least N levels down (GNU find)

  `-xdev` / `-mount`                  Do not descend into other
                                      filesystems

  `-depth`                            Process directory contents before
                                      the directory

  `-prune`                            Do not descend into a directory

  `-follow`                           Follow symbolic links (legacy
                                      behavior; position matters)

  `-P`                                Never follow symbolic links
                                      (default)

  `-H`                                Follow command-line symbolic links
                                      only

  `-L`                                Follow symbolic links encountered
  -----------------------------------------------------------------------

Examples:

``` bash
find . -maxdepth 2 -type f
find . -mindepth 2 -type f
find . -type d -name .git -prune -o -type f -print
find / -xdev -type f -name '*.conf' 2>/dev/null
```

### 7.8 Actions: print, delete, and execute

  -----------------------------------------------------------------------
  Action                              Meaning
  ----------------------------------- -----------------------------------
  `-print`                            Print path, usually the default
                                      action

  `-print0`                           Print paths separated by NUL bytes;
                                      safe for unusual filenames

  `-delete`                           Delete matched entries

  `-exec COMMAND {} \;`               Run command once per match

  `-exec COMMAND {} +`                Pass multiple matches to each
                                      command invocation

  `-ok COMMAND {} \;`                 Ask before each command

  `-ls`                               Print a detailed listing
  -----------------------------------------------------------------------

Examples:

``` bash
find . -type f -name '*.log' -print
find . -type f -name '*.tmp' -print0
find . -type f -name '*.txt' -exec wc -l {} \;
find . -type f -name '*.jpg' -exec file {} +
```

**Delete carefully.** First preview the exact results:

``` bash
find . -type f -name '*.tmp' -print
```

Only if the results are correct, delete:

``` bash
find . -type f -name '*.tmp' -delete
```

`-delete` is powerful and irreversible. Test the search first,
especially when running as root.

Safe handling of filenames containing spaces or newlines:

``` bash
find . -type f -print0 | xargs -0 file
```

### 7.9 Common `find` recipes

``` bash
find . -type f -name '*.pdf'
find . -type f -empty
find . -type f -user "$USER"
find . -type f -size +50M
find . -type f -mmin -60
find . -type f -perm /111
find . -type f -name '*.bak' -exec rm -i {} \;
```

------------------------------------------------------------------------

## 8. `locate`

### 8.1 What it does

`locate` searches a prebuilt filename database rather than walking the
filesystem every time. It is usually much faster than `find`, but
results may be stale or missing until the database is updated.

``` bash
locate passwd
locate '*.pdf'
locate -i readme
```

Patterns may be interpreted as basic regular expressions by default on
some implementations. Use the options and manual page for the installed
implementation.

### 8.2 Common GNU `locate` options

  Option                  Meaning
  ----------------------- ----------------------------------------------------
  `-i`, `--ignore-case`   Ignore case
  `-c`, `--count`         Print number of matches instead of paths
  `-l N`, `--limit=N`     Stop after N results
  `-n N`                  Alias for a result limit on common implementations
  `-b`, `--basename`      Match only the final filename component
  `-r REGEXP`             Search using a regular expression
  `-e`, `--existing`      Print only entries that currently exist
  `-0`, `--null`          Separate results with NUL bytes
  `-S`, `--statistics`    Show database statistics
  `-w`, `--wholename`     Match the whole stored path
  `--help`                Show help
  `--version`             Show version

Option availability can vary because systems may install GNU `plocate`,
`mlocate`, or another implementation.

### 8.3 Update the database

Depending on the distribution and installed package, one of these may be
used:

``` bash
sudo updatedb
sudo systemctl status plocate-updatedb.timer
```

Do not assume the timer or service name is identical on every
distribution. If `locate` is missing, install a package such as
`plocate` on distributions that provide it.

### 8.4 `locate` versus `find`

-   Use `locate` when you want a fast search by stored path.
-   Use `find` when you need current filesystem results or filters such
    as size, modification time, owner, permissions, or actions.
-   `locate` may show deleted paths if its database has not been
    refreshed.

------------------------------------------------------------------------

## 9. `whereis`

### 9.1 What it does

`whereis` locates the binary, source, and manual-page files associated
with a command. It searches standard locations, not every directory on
the computer.

``` bash
whereis bash
whereis python3
whereis ls
```

Example output might include locations for the executable and manual
page.

### 9.2 Common options

  -----------------------------------------------------------------------
  Option                              Meaning
  ----------------------------------- -----------------------------------
  `-b`                                Search for binaries

  `-m`                                Search for manual pages

  `-s`                                Search for source files

  `-u`                                Search for unusual entries

  `-B DIRS`                           Limit binary search to specified
                                      directories

  `-M DIRS`                           Limit manual-page search to
                                      specified directories

  `-S DIRS`                           Limit source search to specified
                                      directories

  `-f`                                Terminate a directory-list option
                                      before the next name on
                                      implementations that support it

  `-l`                                Show the paths searched, where
                                      supported

  `--help`                            Show help

  `--version`                         Show version
  -----------------------------------------------------------------------

Examples:

``` bash
whereis -b bash
whereis -m bash
whereis -s bash
whereis -b -m python3
whereis -B /usr/bin -f bash
```

**Note:** Option behavior differs slightly among implementations. Check
`man whereis`.

------------------------------------------------------------------------

## 10. `which`

### 10.1 What it does

`which` prints the executable path that would be found through the
shell's `PATH` search, depending on the implementation.

``` bash
which python3
which ls
which -a python3
```

### 10.2 Common options

The exact option set depends on the installed `which` program. Common
implementations support:

  Option        Meaning
  ------------- -------------------------------------------------------
  `-a`          Print all matching executables in `PATH`
  `-s`          Silent mode; commonly returns status without printing
  `--version`   Show version on implementations that support it
  `--help`      Show help on implementations that support it

Examples:

``` bash
which bash
which -a python3
```

### 10.3 Prefer shell built-ins for reliable checks

`which` may be an external program and may not fully understand aliases,
functions, or all shell command-resolution rules. In Bash, use:

``` bash
command -v python3
type python3
type -a python3
```

-   `command -v NAME` reports how the shell resolves a name.
-   `type NAME` explains whether it is an alias, function, builtin, or
    executable.
-   `type -a NAME` lists all matching definitions/locations known to the
    shell.

### 10.4 Compare the tools

  Command             Main purpose
  ------------------- -------------------------------------------------
  `which command`     Find executable through `PATH`
  `whereis command`   Find binary/source/manual in standard locations
  `locate name`       Search a filename database
  `find path ...`     Search live directory trees with detailed tests
  `command -v name`   Ask the current shell how it resolves a command

------------------------------------------------------------------------

## 11. Cron jobs and `crontab`

### 11.1 What cron is

Cron is a scheduler that runs commands at specified times. A **cron
job** is a scheduled command or script.

User crontab entries generally contain five time fields followed by a
command:

``` text
MINUTE HOUR DAY_OF_MONTH MONTH DAY_OF_WEEK COMMAND
```

Field ranges:

  -----------------------------------------------------------------------
  Field                   Allowed values          Meaning
  ----------------------- ----------------------- -----------------------
  Minute                  `0-59`                  Minute of the hour

  Hour                    `0-23`                  Hour of the day

  Day of month            `1-31`                  Calendar day

  Month                   `1-12` or names         Month

  Day of week             `0-7` or names on       Sunday is often `0` or
                          common cron             `7`
                          implementations         
  -----------------------------------------------------------------------

Cron implementations differ in how day-of-month and day-of-week are
combined when both are restricted. Consult the local `crontab(5)`
manual.

### 11.2 Special scheduling characters

  Character   Meaning                Example
  ----------- ---------------------- ------------------------------------------
  `*`         Every possible value   `* * * * *` runs every minute
  `,`         List of values         `0,30 * * * *` at minutes 0 and 30
  `-`         Range                  `9-17 * * * *` during hours 9 through 17
  `/`         Step interval          `*/5 * * * *` every 5 minutes

Examples:

``` cron
* * * * * command             # Every minute
*/5 * * * * command           # Every five minutes
0 * * * * command             # At minute 0 every hour
0 8 * * * command             # Every day at 08:00
30 6 * * 1-5 command          # Weekdays at 06:30
0 0 1 * * command             # At midnight on the first day of each month
0 12 * * 0 command            # Sunday at noon on common cron implementations
15 9 1,15 * * command         # At 09:15 on the 1st and 15th of each month
0 9-17 * * 1-5 command        # At minute 0, every hour from 09:00 to 17:00, weekdays
```

**Do not confuse `*/5` with "five minutes after every hour."** `*/5` in
the minute field means every five minutes.

### 11.3 Common schedule examples

``` cron
# Every minute
* * * * * /usr/local/bin/task.sh

# Every 15 minutes
*/15 * * * * /usr/local/bin/task.sh

# At 02:30 every day
30 2 * * * /usr/local/bin/backup.sh

# At 07:00 on weekdays
0 7 * * 1-5 /home/USER/scripts/morning.sh

# Every Sunday at 04:00
0 4 * * 0 /home/USER/scripts/weekly.sh

# On the first of each month at 01:15
15 1 1 * * /home/USER/scripts/monthly.sh
```

Replace `USER` with the actual account name or use the full path to your
script.

### 11.4 Special strings

Common cron implementations support these shortcuts:

  Shortcut                   Meaning
  -------------------------- ---------------------
  `@reboot`                  Run at cron startup
  `@yearly` or `@annually`   Once a year
  `@monthly`                 Once a month
  `@weekly`                  Once a week
  `@daily` or `@midnight`    Once a day
  `@hourly`                  Once an hour

Examples:

``` cron
@reboot /home/USER/scripts/startup.sh
@daily /home/USER/scripts/daily.sh
@weekly /home/USER/scripts/weekly.sh
```

`@reboot` means when the cron daemon starts, not necessarily every time
an interactive user logs in.

### 11.5 `crontab` commands

  -----------------------------------------------------------------------
  Command                             Purpose
  ----------------------------------- -----------------------------------
  `crontab -e`                        Edit current user's crontab

  `crontab -l`                        List current user's cron entries

  `crontab -r`                        Remove current user's entire
                                      crontab

  `crontab -i -r`                     Prompt before removing the current
                                      user's crontab on implementations
                                      that support `-i`

  `crontab FILE`                      Install entries from a file

  `crontab -u USER -l`                List another user's crontab,
                                      usually requiring privilege

  `crontab -u USER -e`                Edit another user's crontab,
                                      usually requiring privilege

  `crontab -u USER -r`                Remove another user's crontab,
                                      usually requiring privilege
  -----------------------------------------------------------------------

**Danger:** `crontab -r` deletes the entire current crontab. Use
`crontab -l` first and make a backup before making major changes.

Backup and restore:

``` bash
crontab -l > "$HOME/crontab.backup"
crontab "$HOME/crontab.backup"
```

Edit your crontab:

``` bash
crontab -e
```

Choose an editor if prompted. Save and exit using that editor's normal
commands.

### 11.6 User crontabs versus system crontabs

A user's crontab normally has **five time fields plus a command**:

``` cron
30 2 * * * /home/USER/backup.sh
```

System crontab files such as `/etc/crontab` and files in `/etc/cron.d/`
commonly include an extra **username field**:

``` text
MIN HOUR DOM MON DOW USER COMMAND
```

Example system entry:

``` cron
30 2 * * * root /usr/local/sbin/backup.sh
```

Do not put the extra username field in a normal user crontab. The format
differs by file.

### 11.7 Make cron scripts reliable

Cron runs with a limited environment. Use absolute paths, set needed
variables, and log output.

Example script `/home/USER/scripts/backup.sh`:

``` bash
#!/usr/bin/env bash
set -eu

# Use absolute paths for commands and files in scheduled jobs.
BACKUP_DIR="/home/USER/backups"
SOURCE_DIR="/home/USER/Documents"
mkdir -p "$BACKUP_DIR"

# Create a timestamped archive.
tar -czf "$BACKUP_DIR/documents-$(date +%F-%H%M%S).tar.gz" "$SOURCE_DIR"
```

Make it executable:

``` bash
chmod +x /home/USER/scripts/backup.sh
```

Crontab entry with a log:

``` cron
30 2 * * * /home/USER/scripts/backup.sh >> /home/USER/backup-cron.log 2>&1
```

-   `>>` appends standard output to the log.
-   `2>&1` sends standard error to the same destination.
-   Use full paths for scripts, files, and commands when possible.
-   Ensure the cron user has permission to read source files and write
    the destination.
-   Test the script manually before scheduling it.

You can define environment variables at the top of a user crontab:

``` cron
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin
MAILTO=""
```

`MAILTO=""` commonly disables cron mail for that crontab. Without that
setting, cron may email job output if mail delivery is configured.

### 11.8 Diagnose cron jobs

``` bash
crontab -l
systemctl status cron
systemctl status crond
```

Service name depends on the distribution: Debian/Ubuntu commonly use
`cron`; other distributions often use `crond`.

Logs may be available through:

``` bash
journalctl -u cron
journalctl -u crond
grep CRON /var/log/syslog
```

Not every system uses the same log file or service name.

Checklist when a job does not run: 1. Confirm the entry with
`crontab -l`. 2. Check the cron service status. 3. Use absolute paths.
4. Check script permissions and its shebang. 5. Redirect output and
errors to a log. 6. Check the system clock and timezone. 7. Remember
cron does not automatically load your interactive shell's `.bashrc`. 8.
In WSL, cron may not run unless the service is started and the WSL
environment remains running. Service support depends on WSL
configuration.

### 11.9 Cron time-zone note

Cron usually uses the system or daemon's configured time zone unless a
supported per-crontab time-zone setting is used. Some implementations
support `CRON_TZ`; availability and behavior vary. Confirm the machine's
time zone before relying on a schedule.

### 11.10 Cron versus `at` and systemd timers

-   **cron:** recurring schedules, such as every day or every 10
    minutes.
-   **at:** one-time future execution, where installed and enabled.
-   **systemd timer:** scheduling integrated with systemd; often useful
    for services and persistent system jobs.

------------------------------------------------------------------------

## 12. Combining commands into useful pipelines

A pipeline (`|`) sends the output of one command to the next command's
standard input.

### Count repeated lines

``` bash
sort names.txt | uniq -c | sort -nr
```

### Show the 10 most common log entries

``` bash
sort access.log | uniq -c | sort -nr | head -10
```

### Count lines in all text files

``` bash
wc -l *.txt
```

### Extract usernames from `/etc/passwd`

``` bash
cut -d: -f1 /etc/passwd
```

### Show usernames and login shells

``` bash
awk -F: '{ print $1, $7 }' /etc/passwd
```

### Find large log files

``` bash
find /var/log -type f -size +50M -print 2>/dev/null
```

### Find files modified in the last day and list them

``` bash
find . -type f -mtime -1 -print
```

### Count `.py` files under a directory

``` bash
find . -type f -name '*.py' -print | wc -l
```

For unusual filenames, avoid newline-delimited pipelines where possible.
Use `-print0` with `xargs -0` for commands that support it.

------------------------------------------------------------------------

## 13. Practice exercises

Try each exercise first, then compare your command with the suggested
answer.

1.  Print the first field of `/etc/passwd`.

    ``` bash
    cut -d: -f1 /etc/passwd
    ```

2.  Print the first and seventh fields of `/etc/passwd`, separated by a
    space.

    ``` bash
    awk -F: '{ print $1, $7 }' /etc/passwd
    ```

3.  Sort a list of numbers numerically.

    ``` bash
    sort -n numbers.txt
    ```

4.  Count repeated lines in a file.

    ``` bash
    sort file.txt | uniq -c
    ```

5.  Find all PDF files below the current directory.

    ``` bash
    find . -type f -iname '*.pdf'
    ```

6.  Find files larger than 100 MiB.

    ``` bash
    find . -type f -size +100M
    ```

7.  Count the number of Python files below the current directory.

    ``` bash
    find . -type f -name '*.py' -print | wc -l
    ```

8.  Find the executable path for Python using the current shell's lookup
    rules.

    ``` bash
    command -v python3
    ```

9.  Find binary and manual locations for `bash`.

    ``` bash
    whereis bash
    ```

10. Schedule a script to run every day at 06:30.
    `cron     30 6 * * * /absolute/path/to/script.sh`

11. Schedule a script to run every five minutes and log both output and
    errors.
    `cron     */5 * * * * /absolute/path/to/script.sh >> /absolute/path/to/script.log 2>&1`

12. Preview temporary files before deleting them.
    `bash     find . -type f -name '*.tmp' -print     # If the results are correct, then run:     find . -type f -name '*.tmp' -delete`

------------------------------------------------------------------------

## Final reminders

-   Use `man COMMAND` to check details for your installed version.
-   Quote filename patterns such as `'*.txt'` so the shell does not
    expand them prematurely.
-   Use `sort` before `uniq` when you need to count duplicates that may
    not be adjacent.
-   `cut` is for simple delimiters, not complex quoted CSV.
-   `locate` is fast but relies on a database; `find` searches the
    filesystem directly.
-   `command -v` is often more reliable than `which` when asking Bash
    how it resolves a command.
-   Test `find -delete` searches before deleting anything.
-   Cron commands should use absolute paths and send output to a log
    while you are troubleshooting.
