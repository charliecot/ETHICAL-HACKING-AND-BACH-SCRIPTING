# BASH `read` & FILE PROCESSING CHEATSHEET
## Beginner to Advanced — Ubuntu / WSL

---

# 1. UNDERSTANDING THE `read` COMMAND

The Bash `read` built-in reads input from standard input and stores it in one or more variables.

General syntax:

```bash
read [OPTIONS] [VARIABLE...]
```

Basic example:

```bash
read name
echo "$name"
```

Enter `Nzegge` when prompted. The variable `name` receives the input.

A more user-friendly example:

```bash
read -r -p "Enter your name: " name
printf 'Hello, %s!\n' "$name"
```

What happens:

1. `read` waits for input.
2. `-p` displays a prompt.
3. `-r` prevents backslashes from being treated as escape characters.
4. `name` stores the input.
5. `printf` displays the result.

**Best practice:** Use `read -r` for ordinary text input unless you specifically need Bash's backslash-escape behavior.

# 2. IMPORTANT `read` OPTIONS

| Option | Meaning | Example |
|---|---|---|
| `-r` | Read backslashes literally | `read -r name` |
| `-p` | Display a prompt | `read -p "Name: " name` |
| `-s` | Do not echo typed input | `read -s password` |
| `-t` | Timeout in seconds | `read -t 5 answer` |
| `-n N` | Read at most N characters | `read -n 1 choice` |
| `-N N` | Read exactly N characters unless EOF or timeout intervenes | `read -N 4 code` |
| `-a ARRAY` | Store words in an indexed array | `read -a names` |
| `-d DELIM` | Stop at a delimiter instead of newline | `read -d ':' value` |
| `-e` | Enable Readline editing | `read -e name` |
| `-i TEXT` | Set initial text when used with `-e` | `read -e -i "hello" text` |
| `-u FD` | Read from a file descriptor | `read -u 3 line` |

Availability and exact behavior depend on the Bash version and input source. Check `help read` on your system.

## 2.1 `-r`: preserve backslashes

```bash
read -r path
```

Input:

```text
C:\Users\Nzegge
```

With `-r`, the backslashes are preserved.

Without `-r`, Bash may interpret backslashes as escape characters.

## 2.2 `-p`: display a prompt

```bash
read -r -p "Enter your username: " username
printf 'Username: %s\n' "$username"
```

## 2.3 `-s`: hidden input

```bash
read -r -s -p "Enter a password: " password
printf '\nPassword received.\n'
```

The password is not displayed while typing.

Avoid printing the password itself or saving it in shell history. For real applications, use an appropriate secret-management mechanism.

## 2.4 `-t`: timeout

```bash
if read -r -t 5 -p "Enter your name within 5 seconds: " name; then
    printf 'Hello, %s\n' "$name"
else
    printf '\nInput was not received in time.\n'
fi
```

The timeout is useful when a script should not wait indefinitely.

## 2.5 `-n`: read a limited number of characters

```bash
read -r -n 1 -p "Continue? (y/n): " choice
printf '\nYou selected: %s\n' "$choice"
```

This reads at most one character, usually useful for interactive menus.

## 2.6 `-N`: read an exact number of characters

```bash
read -r -N 4 -p "Enter a four-character code: " code
printf '\nCode received: %s\n' "$code"
```

Unlike `-n`, `-N` does not stop at a newline. It attempts to read the requested number of characters unless EOF or a timeout intervenes.

## 2.7 `-a`: read into an array

```bash
read -r -a users -p "Enter names separated by spaces: "

for user in "${users[@]}"; do
    printf '%s\n' "$user"
done
```

Input:

```text
Alice Bob Charlie
```

The array contains three elements.

**Important:** This method splits input on `IFS` characters. It does not preserve multiword names as individual elements.

For a predefined array:

```bash
users=("Alice Smith" "Bob" "Charlie")

for user in "${users[@]}"; do
    printf '%s\n' "$user"
done
```

Use `"${users[@]}"` to preserve each array element, including spaces.

## 2.8 `-d`: choose a delimiter

```bash
read -r -d ':' value
```

This reads until a colon is encountered.

Test it interactively:

```bash
printf 'Alice:Admin\n' | {
    IFS= read -r -d ':' name
    printf 'Name: %s\n' "$name"
}
```

The `read` command stops at the colon. Because the delimiter is consumed, the variable contains `Alice`.

Note that `read -d ''` uses a NUL byte as the delimiter in Bash.

## 2.9 `-e` and `-i`: editable input

```bash
read -r -e -i "example.com" host
printf 'Host: %s\n' "$host"
```

This pre-fills the prompt with `example.com`, allowing you to edit it interactively.

## 2.10 `-u`: read from a file descriptor

```bash
exec 3< users.txt

while IFS= read -r -u 3 line; do
    printf '%s\n' "$line"
done

exec 3<&-
```

Here:

- `3< users.txt` opens the file for reading on file descriptor 3.
- `read -u 3` reads from that descriptor.
- `exec 3<&-` closes it.

For simpler scripts, ordinary input redirection is usually enough.

---

# 3. `IFS` AND HOW BASH SPLITS INPUT

`IFS` stands for **Internal Field Separator**. It controls how Bash splits words in many contexts, including `read`.

By default, it contains spaces, tabs, and newlines.

## 3.1 Read a line without trimming leading or trailing whitespace

```bash
while IFS= read -r line; do
    printf '<%s>\n' "$line"
done < users.txt
```

- `IFS=` prevents the default trimming of leading and trailing IFS whitespace.
- `-r` preserves backslashes.
- `line` stores each line.

## 3.2 Read comma-separated values

File:

```text
Alice,25,Developer
Bob,30,Designer
Charlie,22,Student
```

Script:

```bash
while IFS=, read -r name age occupation; do
    printf 'Name: %s | Age: %s | Job: %s\n' \
        "$name" "$age" "$occupation"
done < people.csv
```

This is suitable for simple comma-separated records without embedded commas or quoted fields.

**Important:** CSV supports quoting and embedded commas. For real CSV files, use a CSV-aware parser, such as Python's `csv` module, rather than assuming `IFS=,` implements the full CSV format.

---

# 4. READING FILES LINE BY LINE

## 4.1 The standard pattern

```bash
while IFS= read -r line || [[ -n "$line" ]]; do
    printf '%s\n' "$line"
done < users.txt
```

This handles a final line that does not end with a newline.

## 4.2 Count the lines

```bash
count=0

while IFS= read -r line || [[ -n "$line" ]]; do
    ((count += 1))
done < users.txt

printf 'Total lines: %d\n' "$count"
```

## 4.3 Print line numbers

```bash
line_number=0

while IFS= read -r line || [[ -n "$line" ]]; do
    ((line_number += 1))
    printf '%d: %s\n' "$line_number" "$line"
done < users.txt
```

## 4.4 Skip empty lines

```bash
while IFS= read -r line || [[ -n "$line" ]]; do
    [[ -z "$line" ]] && continue
    printf '%s\n' "$line"
done < users.txt
```

## 4.5 Stop at a particular line

```bash
while IFS= read -r line; do
    [[ "$line" == "STOP" ]] && break
    printf '%s\n' "$line"
done < users.txt
```

- `break` exits the loop.
- `continue` skips to the next iteration.

---

# 5. WHY `< file` IS IMPORTANT

Consider:

```bash
while IFS= read -r line; do
    echo "$line"
done < users.txt
```

The file is connected to the loop's standard input.

Avoid this pattern for general line-by-line processing:

```bash
for line in $(cat users.txt); do
    echo "$line"
done
```

It breaks or changes data when lines contain spaces, tabs, wildcard characters, or other special characters.

Also understand this difference:

```bash
while IFS= read -r line; do
    count=$((count + 1))
done < users.txt
```

The loop runs in the current shell in the usual Bash implementation, so the final value of `count` remains available afterward.

But:

```bash
cat users.txt | while IFS= read -r line; do
    count=$((count + 1))
done
```

In ordinary Bash configurations, a pipeline loop executes in a subshell, so changes to `count` may not survive after the loop.

If you want to use a pipeline while preserving the variable, Bash supports `shopt -s lastpipe` under certain conditions, but input redirection is simpler for this task.

---

# 6. FILE PROCESSING COMMANDS

## 6.1 Viewing files

```bash
cat file.txt
less file.txt
head file.txt
head -n 10 file.txt
tail file.txt
tail -n 20 file.txt
tail -f application.log
```

| Command | Purpose |
|---|---|
| `cat` | Display or concatenate files |
| `less` | Browse a file interactively |
| `head` | Show the beginning |
| `tail` | Show the end |
| `tail -f` | Follow appended content |

## 6.2 Counting content

```bash
wc file.txt
wc -l file.txt
wc -w file.txt
wc -c file.txt
```

- `-l`: count newline characters, commonly used for line counts.
- `-w`: count words.
- `-c`: count bytes.

A file containing a final line without a newline may have fewer newline characters than logical lines.

## 6.3 Search using `grep`

```bash
grep "error" application.log
grep -i "error" application.log
grep -n "error" application.log
grep -v "debug" application.log
grep -c "failed" application.log
grep -r "TODO" ./src
```

| Option | Purpose |
|---|---|
| `-i` | Ignore case |
| `-n` | Display line numbers |
| `-v` | Invert matching |
| `-c` | Count matching lines |
| `-l` | Print matching filenames |
| `-r` | Recursive search |
| `-E` | Extended regular expressions |
| `-F` | Fixed-string matching |
| `-w` | Match whole words |
| `-A N` | Show N lines after a match |
| `-B N` | Show N lines before a match |
| `-C N` | Show N lines around a match |
| `-q` | Suppress normal output |

Examples:

```bash
grep -in "failed" auth.log
grep -E "error|warning|critical" application.log
grep -F "192.168.1.10" access.log
grep -n -C 2 "permission denied" system.log
```

## 6.4 Transform text using `sed`

```bash
sed -n '1,10p' file.txt
sed 's/old/new/' file.txt
sed 's/old/new/g' file.txt
sed -i.bak 's/old/new/g' file.txt
```

- `-n`: suppress automatic printing.
- `p`: print selected lines.
- `s/old/new/`: substitute the first match on each line.
- `g`: replace all matches on each line.
- `-i.bak`: edit in place while keeping a `.bak` backup with GNU `sed`.

## 6.5 Process columns using `awk`

```bash
awk '{print $1}' file.txt
awk '{print $1, $2}' file.txt
awk '{print NR, $0}' file.txt
```

- `$0`: entire current record.
- `$1`: first field.
- `$2`: second field.
- `NR`: current record number.
- `NF`: number of fields in the current record.

Example:

```bash
awk 'NF >= 3 {print $1, $3}' file.txt
```

## 6.6 Sort and remove duplicate lines

```bash
sort names.txt
sort -n numbers.txt
sort names.txt | uniq
sort names.txt | uniq -c
sort -nr counts.txt
```

- `sort`: sort lines.
- `-n`: numeric sort.
- `-r`: reverse order.
- `uniq`: remove adjacent duplicate lines.
- `uniq -c`: count adjacent duplicates.

## 6.7 Extract fields

```bash
cut -d: -f1 /etc/passwd
cut -d, -f1,3 people.txt
```

- `-d`: delimiter.
- `-f`: selected field numbers.

`cut` is suitable for simple delimited text; it does not parse quoted CSV fields.

## 6.8 Find files

```bash
find . -type f -name "*.txt"
find . -type d -name "backup"
find . -type f -iname "*.log"
find . -type f -mtime -7
find . -type f -size +10M
```

| Option | Meaning |
|---|---|
| `-type f` | Regular file |
| `-type d` | Directory |
| `-name` | Case-sensitive pattern |
| `-iname` | Case-insensitive pattern |
| `-mtime -7` | Modified within approximately seven 24-hour periods |
| `-size +10M` | Larger than 10 MiB |

## 6.9 Compare files

```bash
diff old.txt new.txt
cmp file1.bin file2.bin
```

- `diff`: show differences between text files.
- `cmp`: compare files byte by byte.

## 6.10 File metadata and disk usage

```bash
file document.pdf
stat document.pdf
du -sh .
df -h
ls -lh
```

- `file`: identify the likely file type.
- `stat`: show metadata.
- `du`: estimate file/directory disk usage.
- `df`: show filesystem capacity.
- `ls -lh`: list human-readable sizes.

---

# 7. FILE REDIRECTION AND FILE DESCRIPTORS

## 7.1 Overwrite and append

```bash
echo "First line" > file.txt
echo "Second line" >> file.txt
```

- `>` creates or overwrites.
- `>>` creates or appends.

## 7.2 Redirect errors

```bash
grep "error" missing.log 2> errors.txt
```

## 7.3 Redirect both output streams

```bash
command > output.log 2>&1
```

This sends standard output and standard error to the same file.

To discard output:

```bash
command > /dev/null 2>&1
```

Be careful: this hides diagnostic information.

## 7.4 Read and write through a file descriptor

```bash
exec 3< input.txt
read -r -u 3 first_line
exec 3<&-
```

Descriptor 3 is an additional input stream.

---

# 8. PRACTICAL EXAMPLE: PROCESS A USER LIST

Create the input file:

```bash
cat > users.txt <<'EOF'
alice
bob
charlie
david
EOF
```

Create the script:

```bash
cat > process_users.sh <<'EOF'
#!/usr/bin/env bash
set -Eeuo pipefail

file="${1:-users.txt}"

if [[ ! -r "$file" ]]; then
    printf 'Cannot read file: %s\n' "$file" >&2
    exit 1
fi

count=0

while IFS= read -r user || [[ -n "$user" ]]; do
    ((count += 1))
    printf '%d. User: %s\n' "$count" "$user"
done < "$file"

printf 'Total records: %d\n' "$count"
EOF

chmod +x process_users.sh
./process_users.sh
```

Expected output:

```text
1. User: alice
2. User: bob
3. User: charlie
4. User: david
Total records: 4
```

This script reads a list. It does not create, authenticate, or modify actual system accounts.

---

# 9. PRACTICAL EXAMPLE: ANALYZE A LOG FILE

Create a sample log:

```bash
cat > application.log <<'EOF'
INFO Application started
ERROR Database connection failed
INFO Retry started
WARNING High memory usage
ERROR Request failed
EOF
```

Count errors:

```bash
grep -ic "error" application.log
```

Display matching lines with line numbers:

```bash
grep -in "error" application.log
```

Show only the first field:

```bash
awk '{print $1}' application.log
```

Count the different log levels:

```bash
awk '{print $1}' application.log | sort | uniq -c
```

Extract errors and save them:

```bash
grep -i "error" application.log > errors.log
```

Read the extracted records:

```bash
cat errors.log
```

This workflow is useful for application debugging, Linux troubleshooting, and defensive security investigations.

---

# 10. IMPORTANT BASH VARIABLES FOR FILE PROCESSING

| Variable | Meaning |
|---|---|
| `$0` | Script name or invocation |
| `$1`, `$2` | First and second positional arguments |
| `$#` | Number of positional arguments |
| `"$@"` | All arguments as separate words |
| `"$*"` | All arguments combined into one word when quoted |
| `$?` | Exit status of the previous command |
| `$$` | PID of the current shell |
| `$!` | PID of the most recent background command |
| `IFS` | Internal Field Separator |
| `REPLY` | Default destination variable for `read` when no variable is specified |

Example:

```bash
printf 'Script: %s\n' "$0"
printf 'Argument count: %s\n' "$#"

for argument in "$@"; do
    printf 'Argument: %s\n' "$argument"
done
```

For `read`, this also works:

```bash
read -r -p "Enter a value: "
printf 'You entered: %s\n' "$REPLY"
```

---

# 11. QUICK COMMAND SELECTION GUIDE

| Goal | Recommended tool |
|---|---|
| Read input from a user | `read -r -p` |
| Read a file line by line | `while IFS= read -r` |
| Read into an array | `read -a` |
| Search text | `grep` |
| Replace text | `sed` |
| Extract columns | `awk` or `cut` |
| Count lines, words, bytes | `wc` |
| Sort records | `sort` |
| Count adjacent duplicates | `uniq -c` |
| Locate files | `find` |
| Compare text files | `diff` |
| View the end of a growing log | `tail -f` |
| Inspect file metadata | `stat` |
| Inspect disk usage | `du`, `df` |
| Combine commands | Pipes `|` |
| Save normal output | `>` |
| Append output | `>>` |
| Save error messages | `2>` |
| Get Bash built-in help | `help read` |

---

# 12. PRACTICE TASKS

Complete these in your `~/linux-lab` directory.

1. Ask for a name using `read -r -p` and greet the user.
2. Ask for a secret using `read -r -s -p` without displaying the secret.
3. Read `users.txt` line by line and print line numbers.
4. Count records without using `wc`.
5. Read a comma-separated file with `IFS=,`.
6. Search a log file using `grep -in`.
7. Replace a word using `sed`, preserving a backup.
8. Use `awk` to print the first and last fields of a file.
9. Sort a list and count repeated entries.
10. Write a script that checks whether a supplied file exists and is readable before processing it.

## Final principles

- Use `read -r` for ordinary text input.
- Use `IFS= read -r` for reliable line-by-line file processing.
- Use `"${array[@]}"` when iterating over array elements.
- Quote variable expansions to avoid unwanted splitting and glob expansion.
- Understand the difference between a command's output and its exit status.
- Prefer simple, composable tools over complicated one-liners.
- Test file modifications on copies before editing important data.
- Use `help read` to inspect the options supported by your installed Bash version.

**The key skill is understanding how data enters a command, how it is processed, and where the result goes.** Once you understand input, output, redirection, loops, and text-processing tools, you can build useful Linux automation scripts with confidence.
