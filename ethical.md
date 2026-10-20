# BASH, LINUX NETWORKING, SYSTEM ADMINISTRATION & ETHICAL HACKING FIELD GUIDE

**Environment:** Ubuntu Linux / WSL on Windows  
**Level:** Beginner → Intermediate → Advanced  
**Purpose:** Linux fundamentals, networking, troubleshooting, defensive security, and authorized penetration-testing labs.

---

# TABLE OF CONTENTS

1. Linux terminal fundamentals
2. Bash syntax, variables, and quoting
3. Input, output, redirection, and pipes
4. Bash loops, conditions, functions, and scripts
5. Files and directories
6. Search, text processing, and regular expressions
7. Users, permissions, and privileges
8. Processes, jobs, and system monitoring
9. Package management and system information
10. Networking fundamentals
11. Network interfaces, IP addresses, and routing
12. DNS and domain investigation
13. Ports, sockets, and connections
14. HTTP, HTTPS, and web troubleshooting
15. Network scanning and diagnostics
16. Packet capture and analysis
17. SSH and remote administration
18. Hashing, encoding, encryption, and certificates
19. Logs and security investigations
20. Bash automation scripts
21. Cybersecurity workflow and essential tools
22. Practical exercises
23. Learning roadmap and command reference

---

# 1. LINUX TERMINAL FUNDAMENTALS

## 1.1 What is the terminal?

The terminal is the interface where you type commands to interact with Linux.

The shell interprets those commands. Bash means **Bourne Again SHell**.

Example prompt:

```bash
nzegge_charles@DESKTOP-6MJ4LAO:~$
```

Breakdown:

- `nzegge_charles`: current username.
- `DESKTOP-6MJ4LAO`: hostname.
- `~`: current user's home directory.
- `$`: ordinary user prompt.
- `#`: commonly used for the root user's prompt.

Check your environment:

```bash
whoami           # Current username
hostname         # Computer name
pwd              # Current directory
echo "$SHELL"    # Configured login shell
bash --version   # Bash version
uname -a         # Kernel and system information
```

## 1.2 Essential navigation commands

```bash
pwd              # Print working directory
ls               # List files
ls -l            # Detailed listing
ls -la           # Include hidden files
ls -lh           # Human-readable file sizes
cd /etc          # Enter /etc
cd ..            # Parent directory
cd ~             # Home directory
cd -             # Previous directory
clear            # Clear terminal display
history          # Show command history
```

Useful shortcuts:

- `Tab`: autocomplete commands and paths.
- `Ctrl+C`: interrupt a foreground command.
- `Ctrl+Z`: suspend a foreground job.
- `Ctrl+D`: send end-of-input in many terminal situations.
- `Ctrl+L`: clear the terminal display.
- Up/down arrows: navigate command history.

**Important:** `Ctrl+Z` suspends a process; it does not terminate it.

## 1.3 Understanding command syntax

General pattern:

```bash
command [options] [arguments]
```

Example:

```bash
ls -lh /var/log
```

- `ls`: command.
- `-l`: long listing.
- `-h`: human-readable sizes.
- `/var/log`: argument specifying the directory.

Options can have short or long forms:

```bash
ls -a
ls --all
```

They are equivalent for this command.

Options are command-specific. Never assume that `-r`, `-f`, or `-v` means the same thing for every command.

Get help:

```bash
command --help
man command
help read
type cd
command -v curl
```

Use the actual command name instead of the word `command` in the first two examples.

For Bash built-ins:

```bash
help read
help printf
help test
```

For external programs:

```bash
man grep
man sed
man ip
man nmap
```

Inside a manual page, `/pattern` searches for a term, `n` moves to the next match, and `q` exits.

---

# 2. BASH SYNTAX, VARIABLES, AND QUOTING

## 2.1 Variables

```bash
name="Nzegge"
age=25

printf 'Name: %s\n' "$name"
printf 'Age: %d\n' "$age"
```

Do not put spaces around the assignment operator:

```bash
name="Nzegge"     # Correct
name = "Nzegge"   # Incorrect: Bash treats this as a command
```

Variables are referenced with `$`:

```bash
echo "$name"
echo "${name}"
```

Braces make boundaries explicit:

```bash
file="report"
echo "${file}.txt"
```

## 2.2 Environment variables

```bash
echo "$HOME"       # Home directory
echo "$USER"       # Current user
echo "$PATH"       # Executable search paths
echo "$PWD"        # Current directory
echo "$SHELL"      # Configured shell
env                # Display environment
printenv HOME      # Display one variable
```

Export a variable so child processes inherit it:

```bash
export APP_ENV="development"
```

Remove a shell variable:

```bash
unset APP_ENV
```

Do not store passwords, API keys, or tokens directly in scripts or shell history.

## 2.3 Single quotes, double quotes, and unquoted text

```bash
name="Alice"

echo '$name'       # Prints literal $name
echo "$name"       # Prints Alice
echo $name         # Expansion followed by word splitting
```

Prefer quoting variable expansions:

```bash
cp "$source_file" "$destination_file"
```

This protects paths containing spaces.

## 2.4 Command substitution

Run a command and capture its output:

```bash
current_user=$(whoami)
current_date=$(date)

echo "$current_user"
echo "$current_date"
```

Prefer `$(...)` over the older backtick form.

## 2.5 Arithmetic

```bash
a=10
b=3

echo $((a + b))
echo $((a - b))
echo $((a * b))
echo $((a / b))
echo $((a % b))
```

Bash integer arithmetic does not provide ordinary decimal floating-point calculations.

## 2.6 Exit status

Commands return an exit status.

- `0`: success, by convention.
- Nonzero: failure or another condition.

```bash
grep -q "root" /etc/passwd

echo "$?"
```

The special variable `$?` contains the exit status of the most recently executed command.

Combine commands based on status:

```bash
mkdir test_dir && cd test_dir
```

`&&` runs the second command only if the first succeeds.

```bash
mkdir test_dir || echo "Directory creation failed"
```

`||` runs the second command if the first fails.

```bash
command1 ; command2
```

`;` runs the next command regardless of the first command's status.

---

# 3. INPUT, OUTPUT, REDIRECTION, AND PIPES

These concepts are essential for Bash automation and log analysis.

## 3.1 Standard streams

| Stream | Number | Meaning |
|---|---:|---|
| Standard input | 0 | Input received by a command |
| Standard output | 1 | Normal output |
| Standard error | 2 | Error messages |

## 3.2 Output redirection

```bash
echo "Hello" > output.txt
```

Creates the file or replaces its existing contents.

```bash
echo "Another line" >> output.txt
```

Appends instead of replacing.

```bash
ls /missing_directory 2> errors.txt
```

Redirects error messages.

```bash
command > output.txt 2>&1
```

Redirects standard output and standard error to the same destination.

## 3.3 Input redirection

```bash
cat < output.txt
```

Reads the file through standard input.

Example with a `while` loop:

```bash
while IFS= read -r line || [[ -n "$line" ]]; do
    printf 'Line: %s\n' "$line"
done < output.txt
```

- `IFS=` prevents default whitespace trimming by `read`.
- `read -r` preserves backslashes.
- `line` receives each line.
- `< output.txt` redirects the file into the loop.

## 3.4 Pipes

A pipe sends one command's standard output into another command's standard input.

```bash
ls -l | less
```

```bash
ps aux | grep firefox
```

```bash
ip addr | grep inet
```

```bash
journalctl -n 100 | grep -i error
```

A pipe does not automatically pass the first command's standard error to the second command.

## 3.5 Here-documents

Supply multiple lines of input:

```bash
cat <<'EOF'
First line
Second line
Third line
EOF
```

This is useful for creating configuration files and scripts.

## 3.6 Useful combinations

```bash
ip addr > interfaces.txt
grep -i "failed" auth.log > failed.txt
sort names.txt | uniq -c
cat access.log | grep "404"
```

The last example works, but a simpler form is:

```bash
grep "404" access.log
```

Avoid unnecessary `cat` commands when a program accepts a filename directly.

---

# 4. BASH LOOPS, CONDITIONS, FUNCTIONS, AND SCRIPTS

## 4.1 Read user input

```bash
read -r -p "Enter your name: " name
printf 'Hello, %s\n' "$name"
```

## 4.2 Read whitespace-separated input into an array

```bash
read -r -a users -p "Enter usernames separated by spaces: "

for user in "${users[@]}"; do
    printf 'User: %s\n' "$user"
done
```

Here, `read -a` splits input into array elements using the current `IFS`, which is whitespace by default.

## 4.3 Arrays and `[@]`

```bash
users=("Alice Smith" "Bob" "Charlie")

for user in "${users[@]}"; do
    printf '%s\n' "$user"
done
```

`"${users[@]}"` expands to separate array elements while preserving spaces inside each element.

By contrast, `"${users[*]}"` expands to one combined word.

## 4.4 For loop with numbers

```bash
for number in {1..5}; do
    echo "$number"
done
```

Python	Bash
len(items)	${#items[@]}
len(text)	${#text}
items[0]	${items[0]}
items.append("x")	items+=("x")
for item in items:	for item in "${items[@]}"; do ...; done
range(len(items))	((i=0; i<${#items[@]}; i++))
print(value)	echo "$value" or printf '%s\n' "$value"



C-style loop:

```bash
for ((i = 1; i <= 5; i++)); do
    echo "$i"
done
```

## 4.5 While loop reading a file

```bash
while IFS= read -r line || [[ -n "$line" ]]; do
    printf '%s\n' "$line"
done < users.txt
```

This is usually the best approach for processing a text file line by line.

Avoid this for general file processing:

```bash
for line in $(cat users.txt); do
    echo "$line"
done
```

Word splitting and filename expansion can corrupt lines containing spaces or special characters.

## 4.6 Conditions

```bash
if [[ -f "/etc/passwd" ]]; then
    echo "The file exists"
else
    echo "The file is missing"
fi
```

Common tests:

| Test | Meaning |
|---|---|
| `-e FILE` | Path exists |
| `-f FILE` | Regular file |
| `-d DIR` | Directory |
| `-r FILE` | Readable |
| `-w FILE` | Writable |
| `-x FILE` | Executable |
| `-z STRING` | Empty string |
| `-n STRING` | Nonempty string |
| `STRING == PATTERN` | Pattern comparison in `[[ ]]` |
| `A -eq B` | Integer equality in numeric test syntax |

Example:

```bash
read -r -p "Enter your age: " age

if [[ "$age" =~ ^[0-9]+$ ]] && (( age >= 18 )); then
    echo "Age is at least 18"
else
    echo "Invalid input or age below 18"
fi
```

## 4.7 Case statements

```bash
read -r -p "Choose start, stop, or status: " action

case "$action" in
    start)
        echo "Starting service"
        ;;
    stop)
        echo "Stopping service"
        ;;
    status)
        echo "Checking status"
        ;;
    *)
        echo "Unknown action"
        ;;
esac
```

## 4.8 Functions

```bash
greet() {
    local name="$1"
    printf 'Hello, %s\n' "$name"
}

greet "Nzegge"
```

`$1` is the first argument passed to the function. `local` restricts the variable's scope to the function.

## 4.9 Create and run a script

Create a file:

```bash
nano system_info.sh
```

Write:

```bash
#!/usr/bin/env bash

# Stop on most errors, unset variables, and failed pipeline commands.
set -Eeuo pipefail

printf 'User: %s\n' "$(whoami)"
printf 'Host: %s\n' "$(hostname)"
printf 'Directory: %s\n' "$PWD"

echo "Disk usage:"
df -h /

echo "Memory:"
free -h
```

Save the file, then run:

```bash
chmod +x system_info.sh
./system_info.sh
```

`set -Eeuo pipefail` is useful, but it is not a substitute for checking errors. Some commands intentionally return nonzero statuses, so scripts must account for those cases.

---

# 5. FILES AND DIRECTORIES

## 5.1 Create, copy, move, and delete

```bash
mkdir projects
mkdir -p projects/bash/lessons

touch notes.txt
cp notes.txt notes_backup.txt
mv notes_backup.txt backup.txt

rm backup.txt
rmdir empty_directory
```

Be cautious with `rm`. Deletion may be irreversible.

Important options:

- `mkdir -p`: create parent directories as needed.
- `cp -r`: copy directories recursively.
- `cp -i`: ask before overwriting.
- `mv -i`: ask before overwriting.
- `rm -i`: prompt before each removal.
- `rm -r`: remove directories recursively.

Never run unfamiliar destructive commands with `sudo` or broad wildcards without understanding their effects.

## 5.2 View files

```bash
cat notes.txt
less notes.txt
head notes.txt
head -n 20 notes.txt
tail notes.txt
tail -n 50 notes.txt
tail -f application.log
```

`tail -f` follows new log entries as they appear. Press `Ctrl+C` to stop.

## 5.3 Find files

```bash
find "$HOME" -type f -name "*.txt"
find /var/log -type f -name "*.log"
find . -type d -name "backup"
find . -type f -mtime -7
```

Important `find` options:

- `-type f`: regular files.
- `-type d`: directories.
- `-name`: filename pattern.
- `-iname`: case-insensitive filename matching.
- `-mtime -7`: modified within approximately the last seven 24-hour periods.
- `-size +100M`: files larger than 100 MiB.

Example:

```bash
find "$HOME" -type f -name "*.log" -print
```

## 5.4 File metadata

```bash
file notes.txt
stat notes.txt
du -sh "$HOME"
df -h
ls -lh
```

- `file`: identifies file types.
- `stat`: displays metadata.
- `du`: reports disk usage.
- `df`: reports filesystem capacity.

---

# 6. SEARCH, TEXT PROCESSING, AND REGULAR EXPRESSIONS

These tools are essential for log analysis, source-code review, and incident response.

## 6.1 `grep`: search text

```bash
grep "error" application.log
grep -i "error" application.log
grep -n "error" application.log
grep -v "debug" application.log
grep -c "failed" application.log
grep -r "TODO" ./src
```

Options:

| Option | Meaning |
|---|---|
| `-i` | Ignore case |
| `-n` | Show line numbers |
| `-v` | Invert match |
| `-c` | Count matching lines |
| `-l` | Show matching filenames |
| `-r` | Search directories recursively |
| `-E` | Extended regular expressions |
| `-F` | Fixed strings, not regular expressions |
| `-w` | Match whole words |
| `-A 3` | Show three lines after a match |
| `-B 3` | Show three lines before a match |
| `-C 3` | Show three lines around a match |
| `-q` | Suppress output; use exit status |

Examples:

```bash
grep -in "failed" auth.log
grep -E "error|warning|critical" application.log
grep -F "192.168.1.10" access.log
grep -n -C 2 "permission denied" system.log
```

## 6.2 `sed`: edit or transform text

Print selected lines:

```bash
sed -n '1,10p' notes.txt
```

Replace text in output:

```bash
sed 's/old/new/' notes.txt
```

Replace all occurrences on each line:

```bash
sed 's/old/new/g' notes.txt
```

Modify a file in place:

```bash
sed -i 's/old/new/g' notes.txt
```

**Warning:** `-i` modifies the file. Make a backup first if the original matters.

With GNU `sed`, a backup can be created using:

```bash
sed -i.bak 's/old/new/g' notes.txt
```

## 6.3 `awk`: process structured text

```bash
awk '{print $1}' access.log
awk '{print $1, $2}' access.log
awk '{print NR, $0}' notes.txt
```

`awk` treats whitespace as the default field separator.

Example access log:

```text
192.168.1.10 GET 200
192.168.1.20 POST 403
192.168.1.30 GET 404
```

Print the IP address and status:

```bash
awk '{print $1, $3}' access.log
```

Find unsuccessful HTTP responses in this sample:

```bash
awk '$3 >= 400 {print $1, $3}' access.log
```

This assumes the third field is the HTTP status code.

## 6.4 Other text utilities

```bash
sort names.txt
sort -n numbers.txt
uniq names.txt
sort names.txt | uniq
sort names.txt | uniq -c
wc -l access.log
wc -w notes.txt
cut -d: -f1 /etc/passwd
tr '[:lower:]' '[:upper:]' < notes.txt
paste file1.txt file2.txt
diff old.conf new.conf
comm sorted_a.txt sorted_b.txt
xargs
```

Important details:

- `uniq` only removes adjacent duplicate lines; sorting first is often necessary.
- `cut -d: -f1` splits fields using `:`.
- `diff` compares files.
- `xargs` constructs command arguments from input; be careful with filenames containing whitespace or newlines.

## 6.5 Regular expression basics

| Pattern | Meaning |
|---|---|
| `.` | Any single character |
| `*` | Zero or more of the preceding expression |
| `^` | Start of line |
| `$` | End of line |
| `[abc]` | One character from the set |
| `[0-9]` | One digit |
| `[^0-9]` | One nondigit character |
| `+` | One or more in extended regex |
| `?` | Zero or one in extended regex |
| `|` | Alternative in extended regex |
| `()` | Grouping in extended regex |
| `{3}` | Exactly three repetitions in extended regex |

Examples:

```bash
grep -E '^[0-9]+$' numbers.txt
grep -E 'error|warning' application.log
grep -E '^[[:space:]]*$' notes.txt
```

In ordinary `grep`, `+`, `?`, `|`, and parentheses have different rules unless you use `-E`.

---

# 7. USERS, PERMISSIONS, AND PRIVILEGES

## 7.1 User and group information

```bash
whoami
id
groups
who
w
getent passwd
getent group
```

## 7.2 Permissions

```bash
ls -l
chmod 644 notes.txt
chmod 600 secret.txt
chmod 755 script.sh
chmod u+x script.sh
chown user:group notes.txt
```

Typical permission meanings:

- `r`: read.
- `w`: write.
- `x`: execute or directory traversal.

Numeric permissions:

| Number | Permission |
|---|---|
| `0` | `---` |
| `1` | `--x` |
| `2` | `-w-` |
| `3` | `-wx` |
| `4` | `r--` |
| `5` | `r-x` |
| `6` | `rw-` |
| `7` | `rwx` |

Example:

```bash
chmod 750 private_script.sh
```

Owner gets `rwx`, group gets `r-x`, and others get no permissions.

## 7.3 Privilege escalation awareness

```bash
sudo command
sudo -l
```

`sudo -l` shows the commands your account is permitted to run through `sudo`.

Use elevated privileges only when necessary. Do not run unknown scripts with root privileges.

---

# 8. PROCESSES, JOBS, AND SYSTEM MONITORING

## 8.1 View running processes

```bash
ps
ps aux
ps -ef
top
```

`ps aux` is a common BSD-style format for listing processes.

`top` displays changing process and resource information. Press `q` to quit.

If available, `htop` provides an interactive alternative:

```bash
sudo apt install htop
htop
```

## 8.2 Find processes

```bash
pgrep bash
pgrep -a python
pidof ssh
ps aux | grep '[p]ython'
```

A PID is a process identifier.

## 8.3 Stop processes

```bash
kill PID
kill -TERM PID
kill -KILL PID
pkill -x process_name
```

Replace `PID` with the actual numeric process ID.

- `SIGTERM` asks a process to terminate cleanly.
- `SIGKILL` forces termination and should generally be a last resort.
- `pkill` selects processes by name or other criteria; verify the target carefully.

Do not use `kill -9` indiscriminately.

## 8.4 Jobs in the current shell

```bash
sleep 300 &
jobs
```

The `&` starts the command in the background.

```bash
fg %1
```

Bring job 1 to the foreground.

```bash
bg %1
```

Resume a suspended job in the background.

```bash
jobs -l
```

Show jobs with process IDs.

`jobs`, `fg`, and `bg` refer to jobs associated with the current shell. They are not general system-wide process managers.

## 8.5 CPU, memory, and disk

```bash
uptime
free -h
df -h
du -sh "$HOME"
vmstat 1
iostat
```

`iostat` may require the `sysstat` package.

Install useful system monitors:

```bash
sudo apt install sysstat procps psmisc
```

Other commands:

```bash
lscpu
lsmem
lsblk
uname -r
```

---

# 9. PACKAGE MANAGEMENT AND SYSTEM INFORMATION

## 9.1 APT commands

```bash
sudo apt update
sudo apt upgrade
sudo apt install curl
sudo apt remove curl
sudo apt autoremove
apt search network
apt show nmap
dpkg -l
dpkg -L package_name
```

Meanings:

- `apt update`: refresh package metadata.
- `apt upgrade`: install available upgrades.
- `apt install`: install packages.
- `apt remove`: uninstall a package.
- `apt autoremove`: remove packages no longer needed.
- `apt show`: display package information.
- `dpkg -l`: list installed packages.
- `dpkg -L`: list files installed by a package.

`apt update` does not itself upgrade installed software.

## 9.2 System information

```bash
date
timedatectl
hostnamectl
uname -a
cat /etc/os-release
uptime
free -h
df -h
lsblk
```

## 9.3 Services and logs

On systems using systemd:

```bash
systemctl status SERVICE
systemctl list-units --type=service
journalctl -n 100
journalctl -p warning
journalctl -u SERVICE
```

Replace `SERVICE` with an actual service name.

WSL may not have systemd enabled, depending on its configuration. Some service-management commands may therefore be unavailable or behave differently.

---

# 10. NETWORKING FUNDAMENTALS

## 10.1 What is a network?

A network allows devices to exchange information.

Important concepts:

- **IP address:** identifies a network interface logically.
- **MAC address:** link-layer address used on local network links.
- **Subnet:** defines an IP address range.
- **Gateway:** forwards traffic to other networks.
- **DNS:** translates names into resource records, often including IP addresses.
- **Port:** identifies a transport-layer endpoint associated with an application.
- **Protocol:** defines communication rules.
- **TCP:** reliable, connection-oriented transport.
- **UDP:** datagram-based transport without TCP's built-in delivery guarantees.
- **HTTP:** web communication protocol.
- **HTTPS:** HTTP protected by TLS.

## 10.2 Private IPv4 ranges

| Range | Purpose |
|---|---|
| `10.0.0.0/8` | Private network |
| `172.16.0.0/12` | Private network |
| `192.168.0.0/16` | Private network |
| `127.0.0.0/8` | IPv4 loopback |

The address `127.0.0.1` refers to the local host.

`0.0.0.0` has context-dependent meanings. For example, a server bound to `0.0.0.0` usually listens on all its available IPv4 interfaces.

## 10.3 CIDR notation

Example:

```text
192.168.1.10/24
```

`/24` means the first 24 bits are the network prefix.

A conventional IPv4 `/24` subnet has 256 addresses, commonly 254 usable host addresses in an ordinary broadcast LAN.

Calculate subnet details:

```bash
ipcalc 192.168.1.10/24
```

---

# 11. NETWORK INTERFACES, IP ADDRESSES, AND ROUTING

## 11.1 The `ip` command

Modern Linux generally uses `ip` rather than legacy `ifconfig`.

```bash
ip addr
ip -br addr
ip link
ip route
ip neigh
```

Common options and subcommands:

- `ip addr`: display interface addresses.
- `ip -br addr`: concise address listing.
- `ip link`: display link-layer interfaces.
- `ip route`: show routing information.
- `ip neigh`: show neighbor-cache entries.

Example:

```bash
ip route get 1.1.1.1
```

This shows how Linux would route traffic to the destination.

## 11.2 Legacy tools

```bash
ifconfig
route -n
arp -n
netstat -rn
```

These are older utilities from `net-tools`. Prefer `ip` and `ss` for modern Linux administration.

## 11.3 Test connectivity

```bash
ping -c 4 1.1.1.1
ping -c 4 example.com
```

- `-c 4`: send four requests.
- A successful response indicates reachability for those packets, not that every service is working.
- A failed response does not always mean the destination is offline; filtering may block ICMP.

Check the route:

```bash
tracepath example.com
traceroute example.com
mtr example.com
```

Some probes may be filtered, and WSL networking can affect the results.

---

# 12. DNS AND DOMAIN INVESTIGATION

## 12.1 `dig`

```bash
dig example.com
dig example.com A
dig example.com AAAA
dig example.com MX
dig example.com NS
dig example.com TXT
dig example.com SOA
```

Record types:

- `A`: IPv4 address.
- `AAAA`: IPv6 address.
- `MX`: mail servers.
- `NS`: authoritative name servers.
- `TXT`: text records, often used for verification and email security.
- `SOA`: start-of-authority information.

Useful options:

```bash
dig +short example.com
dig +noall +answer example.com
dig @1.1.1.1 example.com
dig -x 1.1.1.1
```

`@1.1.1.1` asks that DNS server directly. `-x` performs a reverse DNS lookup.

## 12.2 Other DNS tools

```bash
host example.com
nslookup example.com
getent hosts example.com
```

`getent hosts` can use the system's configured name-resolution mechanisms, which may include sources other than DNS.

## 12.3 WHOIS

```bash
whois example.com
whois 1.1.1.1
```

WHOIS data availability varies by registry and privacy policy. Some registration information is now more readily available through RDAP services.

## 12.4 DNS troubleshooting workflow

```bash
# 1. Check configured name resolution
cat /etc/resolv.conf

# 2. Query DNS
dig example.com

# 3. Query IPv4 records
dig example.com A

# 4. Check system name resolution
getent hosts example.com

# 5. Test HTTPS separately
curl -I https://example.com
```

This helps distinguish name-resolution problems from HTTP or general connectivity problems.

---

# 13. PORTS, SOCKETS, AND CONNECTIONS

## 13.1 The `ss` command

`ss` is the modern tool for inspecting sockets.

```bash
ss -tuln
ss -tunap
ss -ltn
ss -lun
```

Options:

- `-t`: TCP.
- `-u`: UDP.
- `-l`: listening sockets.
- `-n`: numeric addresses and ports.
- `-a`: all sockets.
- `-p`: show process information where permitted.

Example:

```bash
sudo ss -tulpen
```

This is useful for inspecting local listening services and associated process details. Root privileges may be needed to see all process information.

## 13.2 Legacy `netstat`

```bash
netstat -tuln
netstat -an
netstat -rn
```

Prefer `ss` for most current Linux workflows.

## 13.3 Test a TCP connection

```bash
nc -vz example.com 443
```

- `-v`: verbose output.
- `-z`: connection check without sending application data.

A successful TCP connection does not prove that authentication works or that the application is healthy.

Check a local service:

```bash
nc -vz 127.0.0.1 8000
```

Only test systems and ports you are authorized to access.

## 13.4 Common ports

| Port | Common service |
|---:|---|
| 20/21 | FTP data/control in traditional active FTP |
| 22 | SSH |
| 25 | SMTP |
| 53 | DNS over TCP or UDP |
| 67/68 | DHCP for IPv4 |
| 80 | HTTP |
| 110 | POP3 |
| 123 | NTP |
| 143 | IMAP |
| 443 | HTTPS |
| 445 | SMB |
| 587 | Message submission |
| 993 | IMAPS |
| 995 | POP3S |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 8000 | Common development-server port |
| 8080 | Common alternate HTTP/development port |

A port number does not guarantee which application is running there.

---

# 14. HTTP, HTTPS, AND WEB TROUBLESHOOTING

## 14.1 `curl`

```bash
curl https://example.com
curl -I https://example.com
curl -L https://example.com
curl -o page.html https://example.com
curl -w '\nHTTP status: %{http_code}\n' -o /dev/null -s https://example.com
```

Important options:

- `-I`: request headers only, typically with a HEAD request.
- `-L`: follow redirects.
- `-o FILE`: write response body to a file.
- `-O`: save using the remote filename.
- `-s`: silent progress and error display.
- `-v`: verbose connection details.
- `-H`: add a request header.
- `-d`: send request data.
- `-X`: specify the HTTP method explicitly.

Example API request:

```bash
curl -H "Accept: application/json" https://example.com/api/
```

Example POST with JSON:

```bash
curl -X POST https://example.com/api/ \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice"}'
```

Only send requests to APIs you are permitted to use.

## 14.2 `wget`

```bash
wget https://example.com/file.zip
wget -O saved_file.zip https://example.com/file.zip
wget --spider https://example.com
```

`wget` is commonly used for downloading files and checking resource availability.

## 14.3 TLS inspection with OpenSSL

```bash
openssl s_client -connect example.com:443 -servername example.com
```

This connects to a TLS service and displays handshake and certificate information.

For a compact certificate summary:

```bash
echo | openssl s_client \
  -connect example.com:443 \
  -servername example.com 2>/dev/null |
  openssl x509 -noout -subject -issuer -dates
```

The `-servername` option supplies SNI, which is important when servers host multiple domains.

---

# 15. NETWORK SCANNING AND DIAGNOSTICS

Use scanning tools only on your own lab, your own systems, or systems for which you have explicit authorization.

## 15.1 Nmap basics

Check the installed version:

```bash
nmap --version
```

Scan your own lab machine:

```bash
nmap 127.0.0.1
```

Scan a specific authorized host:

```bash
nmap -sT -p 22,80,443 192.168.1.10
```

Options:

- `-sT`: TCP connect scan.
- `-p`: select ports.
- `-sV`: attempt service/version detection.
- `-O`: attempt operating-system detection.
- `-Pn`: skip host-discovery probes and treat the target as online.
- `-oN FILE`: save output in normal text format.

For a service inventory of your own lab host:

```bash
nmap -sT -sV -p 22,80,443 127.0.0.1
```

Service/version detection is an estimate, not definitive proof of the software or version.

## 15.2 Understand port states

- `open`: a service accepted the probe.
- `closed`: the host responded, but no service was listening on the probed port.
- `filtered`: Nmap could not determine whether the port was open because filtering or another network condition interfered.
- `open|filtered`: the available probe did not distinguish between those possibilities.

## 15.3 Other network diagnostic tools

```bash
mtr example.com
tracepath example.com
iperf3 --version
ipcalc 192.168.1.10/24
```

`iperf3` requires a server on one endpoint and a client on the other to measure throughput.

---

# 16. PACKET CAPTURE AND ANALYSIS

## 16.1 `tcpdump`

List available capture interfaces:

```bash
tcpdump -D
```

Capture packets on a supported interface:

```bash
sudo tcpdump -i any
```

Capture a limited number:

```bash
sudo tcpdump -i any -c 20
```

Capture DNS traffic:

```bash
sudo tcpdump -i any port 53
```

Capture TCP traffic involving a particular host:

```bash
sudo tcpdump -i any host 192.168.1.10 and tcp
```

Save packets for later analysis:

```bash
sudo tcpdump -i any -c 100 -w capture.pcap
```

Read a saved capture:

```bash
tcpdump -r capture.pcap
```

Options:

- `-i`: interface.
- `-D`: list interfaces.
- `-c`: stop after a number of packets.
- `-w`: write packet capture to a file.
- `-r`: read a capture file.
- `-n`: avoid name resolution.
- `-nn`: avoid resolving both hostnames and service names.

**Privacy:** Packet captures can contain sensitive information. Capture only traffic you are authorized to inspect. WSL's virtual networking may limit which packets you can see.

## 16.2 Wireshark and TShark

Wireshark provides a graphical packet analyzer. TShark is its command-line counterpart.

```bash
tshark --version
tshark -r capture.pcap
```

A basic display filter for DNS packets:

```bash
tshark -r capture.pcap -Y dns
```

Packet capture and display filters use different syntax. `tcpdump` capture expressions are not interchangeable with Wireshark display filters.

---

# 17. SSH AND REMOTE ADMINISTRATION

## 17.1 SSH client

```bash
ssh username@server
ssh -p 2222 username@server
```

- `-p`: select the SSH port.
- `-i`: specify a private-key file.

Example:

```bash
ssh -i ~/.ssh/id_ed25519 username@server
```

Never share your private key.

## 17.2 Generate an SSH key

```bash
ssh-keygen -t ed25519
```

The private key stays on your machine. The corresponding public key can be installed on an authorized server.

## 17.3 Copy files

```bash
scp file.txt username@server:/path/to/destination/
scp username@server:/path/to/file.txt .
```

## 17.4 Check the SSH client

```bash
ssh -V
```

If the client is missing:

```bash
sudo apt install openssh-client
```

An SSH client is different from an SSH server. Installing `openssh-client` does not automatically make your machine accept incoming SSH connections.

---

# 18. HASHING, ENCODING, ENCRYPTION, AND CERTIFICATES

These concepts are related but not interchangeable.

## 18.1 Base64 encoding

```bash
printf 'Hello' | base64
printf 'SGVsbG8=' | base64 -d
```

Base64 changes the representation of data. It does not encrypt or protect it.

## 18.2 SHA-256 hashing

```bash
sha256sum file.txt
```

Hash a string:

```bash
printf 'Hello' | sha256sum
```

Verify a saved checksum:

```bash
sha256sum file.txt > checksums.txt
sha256sum -c checksums.txt
```

A cryptographic hash is designed to produce a fixed-size digest. It is not intended to be reversible.

## 18.3 Other hash tools

```bash
sha1sum file.txt
sha256sum file.txt
sha512sum file.txt
md5sum file.txt
```

MD5 and SHA-1 are not recommended for modern collision-resistant security uses. Prefer SHA-256 or stronger algorithms for integrity verification.

## 18.4 Random bytes and hexadecimal

```bash
od -An -tx1 -N16 /dev/urandom
```

Generate 16 random bytes and display them as hexadecimal.

Another useful tool:

```bash
xxd file.bin
xxd -r file.hex output.bin
```

`xxd` creates and reverses hexadecimal dumps. Ensure the input format is appropriate when reversing a dump.

## 18.5 Encryption

Encryption uses a key to transform data so it can be recovered by someone with the appropriate key.

Learn the difference between:

- Symmetric encryption: one secret key is used for encryption and decryption.
- Asymmetric cryptography: related public and private keys serve different roles.
- Digital signatures: help verify authenticity and integrity.
- Hashes: one-way digests, not encryption.
- Encoding: representation changes, not secrecy.

Do not invent your own cryptographic algorithms for real security systems.

---

# 19. LOGS AND SECURITY INVESTIGATIONS

Logs help you reconstruct events and diagnose failures.

## 19.1 Common log locations

```bash
ls -lah /var/log
```

Examples:

- `/var/log/syslog`: general system events on some distributions.
- `/var/log/auth.log`: authentication-related events on some Debian/Ubuntu configurations.
- `/var/log/kern.log`: kernel messages on some configurations.
- Application-specific log files: location depends on the application.

Not every WSL installation has the same logs or logging services.

## 19.2 Search logs

```bash
grep -i "error" /var/log/syslog
grep -i "failed" /var/log/auth.log
grep -in "denied" application.log
```

Use `sudo` only when necessary to read protected logs.

## 19.3 Inspect recent system events

```bash
journalctl -n 100
journalctl -p err
journalctl --since "1 hour ago"
```

If systemd is enabled and the relevant service logs to the journal:

```bash
journalctl -u ssh
```

The exact service name may differ.

## 19.4 Count repeated events

```bash
grep "LOGIN_FAILED" access.log | wc -l
```

Count occurrences by first field:

```bash
awk '/LOGIN_FAILED/ {print $1}' access.log | sort | uniq -c | sort -nr
```

This example assumes the first field contains an identifier such as an IP address.

## 19.5 Investigate carefully

A log entry can be misleading without context. Check timestamps, time zones, source addresses, authentication results, and surrounding events before drawing conclusions.

Do not assume an IP address identifies a particular person.

---

# 20. BASH AUTOMATION SCRIPTS

## 20.1 Check whether a host responds

Create `check_host.sh`:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

host="${1:-}"

if [[ -z "$host" ]]; then
    echo "Usage: $0 HOSTNAME_OR_IP" >&2
    exit 2
fi

if ping -c 1 -W 2 "$host" >/dev/null 2>&1; then
    printf '%s responded to ping\n' "$host"
else
    printf '%s did not respond to ping\n' "$host"
fi
```

Run:

```bash
chmod +x check_host.sh
./check_host.sh 127.0.0.1
```

A failed ping does not prove a host is offline; ICMP may be blocked.

## 20.2 Summarize errors from a log

Create `count_errors.sh`:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

log_file="${1:-}"

if [[ -z "$log_file" || ! -r "$log_file" ]]; then
    echo "Usage: $0 READABLE_LOG_FILE" >&2
    exit 2
fi

count=$(grep -icE 'error|critical|failed' "$log_file" || true)

printf 'Matching lines: %s\n' "$count"
```

Run:

```bash
chmod +x count_errors.sh
./count_errors.sh application.log
```

`grep` returns status 1 when there are no matches. The `|| true` prevents that expected condition from stopping this particular script under `set -e`.

## 20.3 Inventory local listening sockets

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

echo "Listening TCP and UDP sockets:"
ss -tuln
```

This gives a basic view of local listening endpoints. It does not scan external hosts.

## 20.4 Script best practices

- Quote variables: `"$file"`.
- Use `read -r` for ordinary line input.
- Validate user-supplied arguments.
- Avoid running untrusted scripts as root.
- Use meaningful variable names.
- Print errors to standard error when appropriate.
- Handle expected nonzero exit codes.
- Do not hardcode passwords or API tokens.
- Test scripts against sample data before using them on important systems.

---

# 21. CYBERSECURITY WORKFLOW AND ESSENTIAL TOOLS

Ethical hacking requires more than memorizing commands. You need networking knowledge, operating-system fundamentals, programming, security concepts, and disciplined testing.

## 21.1 Reconnaissance

Goal: understand the authorized target and its publicly available information.

Useful tools:

```bash
dig example.com
whois example.com
curl -I https://example.com
```

Keep reconnaissance within the scope you have permission to investigate.

## 21.2 Asset and service discovery

For your own lab:

```bash
ip addr
ip route
ss -tuln
nmap -sT -sV 127.0.0.1
```

Document discovered services and why each one is required.

## 21.3 Web security fundamentals

Learn:

- HTTP methods and status codes.
- Headers, cookies, and sessions.
- Authentication and authorization.
- Input validation and output encoding.
- SQL injection and its prevention.
- Cross-site scripting and its prevention.
- Cross-site request forgery.
- Access-control failures.
- Secure password storage.
- TLS certificate validation.
- Rate limiting and logging.

Practice using deliberately vulnerable applications in isolated labs, not real third-party systems.

## 21.4 Defensive monitoring

Useful commands:

```bash
ps aux
ss -tuln
journalctl -p warning
grep -i "failed" application.log
sha256sum important_file
```

Combine them with sound investigation methods: establish a baseline, collect evidence, check timestamps, and preserve relevant files.

## 21.5 Recommended learning platforms and references

- **OverTheWire Bandit:** beginner Linux and shell exercises.
- **PortSwigger Web Security Academy:** web security lessons and legal practice labs.
- **OWASP Web Security Testing Guide:** web application testing methodology.
- **Nmap Reference Guide:** scanning options and interpretation.
- **GNU Bash manual:** shell syntax and built-ins.
- **Ubuntu documentation:** package management and system administration.
- **Wireshark documentation:** packet capture and analysis.

Always follow the rules and scope of each training environment.

---

# 22. PRACTICAL EXERCISES

Work through these exercises in order. Create a dedicated practice directory so you do not accidentally modify important files.

## Exercise 1: Basic Bash

```bash
mkdir -p ~/linux-lab
cd ~/linux-lab

whoami
pwd
date
uname -a
```

Goal: understand your user, directory, date, and kernel.

## Exercise 2: File processing

```bash
printf 'Alice\nBob\nCharlie\n' > users.txt

while IFS= read -r user; do
    printf 'User: %s\n' "$user"
done < users.txt
```

Goal: understand input redirection and loops.

## Exercise 3: Text analysis

```bash
printf 'INFO started\nERROR failed\nINFO complete\n' > app.log

grep -i error app.log
grep -c INFO app.log
awk '{print $1}' app.log
```

Goal: learn searching, counting, and field extraction.

## Exercise 4: Local networking

```bash
ip addr
ip route
ping -c 3 127.0.0.1
ss -tuln
```

Goal: inspect local interfaces, routing, connectivity, and listening sockets.

## Exercise 5: DNS and HTTPS

```bash
dig example.com
curl -I https://example.com
```

Goal: distinguish DNS lookup from HTTP connectivity.

## Exercise 6: Local port testing

If a suitable local development server is available, inspect its listening socket:

```bash
ss -ltn
```

Then test a port that you know is open on your own machine:

```bash
nc -vz 127.0.0.1 8000
```

Replace `8000` with the actual port. An unsuccessful connection may simply mean no service is listening.

## Exercise 7: Check file integrity

```bash
printf 'Important practice data\n' > sample.txt

sha256sum sample.txt
sha256sum sample.txt > sample.sha256
sha256sum -c sample.sha256
```

Goal: understand hashing and checksum verification.

## Exercise 8: Build a system report

Create a script that records:

- Current user.
- Hostname.
- Date.
- Kernel version.
- Memory usage.
- Disk usage.
- IP addresses.
- Listening sockets.

Save the output to a report file and ensure the script handles failures clearly.

---

# 23. LEARNING ROADMAP

Follow this order instead of trying to learn every tool at once.

## Stage 1 — Linux and Bash fundamentals

Master:

- Navigation and file management.
- Variables and quoting.
- Pipes and redirection.
- `read`, `for`, and `while`.
- Conditions and functions.
- Permissions and users.
- `grep`, `sed`, `awk`, `find`, and `sort`.

**Goal:** Write small scripts that process files safely.

## Stage 2 — Networking fundamentals

Master:

- IPv4 and IPv6.
- Subnets and CIDR.
- DNS.
- TCP and UDP.
- Ports and sockets.
- Routing and gateways.
- HTTP and HTTPS.
- TLS certificates.

**Goal:** Explain what happens when you open a website in a browser.

## Stage 3 — Linux administration

Master:

- Processes and signals.
- Package management.
- Services and logs.
- Memory and disk usage.
- Users and permissions.
- SSH and remote administration.

**Goal:** Diagnose common Linux problems independently.

## Stage 4 — Network troubleshooting and analysis

Master:

- `ip`, `ping`, `dig`, `ss`, and `traceroute`.
- `curl` and OpenSSL.
- `tcpdump` and Wireshark.
- Nmap in authorized labs.
- Log analysis and evidence preservation.

**Goal:** Identify whether a failure comes from DNS, routing, a blocked port, TLS, or the application itself.

## Stage 5 — Cybersecurity

Study:

- Linux security and permissions.
- Networking protocols and common attacks.
- Secure coding.
- Web application security.
- Authentication and access control.
- Cryptography fundamentals.
- Vulnerability management.
- Incident response.
- Threat modeling.
- Legal and ethical testing practices.

**Goal:** Understand vulnerabilities, reproduce them in controlled labs, and learn how to prevent or detect them.

---

# FINAL QUICK-REFERENCE TABLE

| Task | Command |
|---|---|
| Current user | `whoami` |
| Current directory | `pwd` |
| List files | `ls -lah` |
| Read a file | `less file.txt` |
| Find a file | `find . -name "*.txt"` |
| Search text | `grep -in "error" file.log` |
| Transform text | `sed 's/old/new/g' file.txt` |
| Process columns | `awk '{print $1}' file.txt` |
| Count lines | `wc -l file.txt` |
| Show processes | `ps aux` |
| Monitor processes | `top` |
| Terminate a process | `kill -TERM PID` |
| Show background jobs | `jobs -l` |
| Show IP addresses | `ip addr` |
| Show routes | `ip route` |
| Test reachability | `ping -c 4 HOST` |
| Query DNS | `dig DOMAIN` |
| Show listening ports | `ss -tuln` |
| Test a TCP port | `nc -vz HOST PORT` |
| Inspect HTTP headers | `curl -I URL` |
| Trace a route | `traceroute HOST` |
| Scan an authorized host | `nmap -sT HOST` |
| Capture packets | `sudo tcpdump -i any` |
| Calculate subnet | `ipcalc IP/CIDR` |
| Hash a file | `sha256sum FILE` |
| Decode Base64 | `base64 -d FILE` |
| Inspect disk capacity | `df -h` |
| Inspect memory | `free -h` |
| Inspect system logs | `journalctl -n 100` |
| Install a package | `sudo apt install PACKAGE` |
| Get command help | `COMMAND --help` |
| Read a manual | `man COMMAND` |
| Get Bash built-in help | `help COMMAND` |

**Final advice:** Learn what a command changes, what its output means, what permissions it requires, and how to verify its result. A strong ethical hacker is not someone who runs the most commands; it is someone who understands systems well enough to identify problems, test hypotheses carefully, interpret evidence, and improve security without causing unnecessary harm.
