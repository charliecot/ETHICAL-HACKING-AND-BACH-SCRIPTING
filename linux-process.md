# Linux Process Management — Complete Reference

For Ubuntu, Debian, WSL, and other GNU/Linux systems.

## 1. What is a process?

A **process** is a running instance of a program. For example, running Python or opening a web server creates one or more processes.

Every process commonly has:
- **PID:** Process ID.
- **PPID:** Parent process ID.
- **USER:** Account running the process.
- **STATE:** Current process state.
- **CPU%:** Approximate CPU usage.
- **MEM%:** Approximate memory usage.

### Process states

| State | Meaning |
|---|---|
| `R` | Running or ready to run |
| `S` | Interruptible sleep |
| `D` | Uninterruptible sleep, often waiting for I/O |
| `T` | Stopped or traced |
| `Z` | Zombie: exited but not yet reaped by its parent |

A zombie has already finished executing. Sending `kill` to it generally does not remove it; its parent must collect its exit status.

### Useful process variables

```bash
echo $$       # PID of the current shell
echo $PPID    # PID of the shell's parent
echo $!       # PID of the most recent background process
```

## 2. `ps` — display processes

`ps` displays a snapshot of process information.

```bash
ps                    # Processes associated with this terminal
ps -e                 # All processes
ps -ef                # Full-format list of all processes
ps aux                # Detailed BSD-style listing on Linux
ps -u "$USER"         # Processes owned by the current user
ps -p 1234             # Show process with PID 1234
ps -ef --forest        # Display process hierarchy
```

### Common options

| Option | Meaning |
|---|---|
| `-e`, `-A` | Select all processes |
| `-f` | Full-format output |
| `-F` | Extra-full format on GNU procps |
| `-u USER` | Select by effective user |
| `-U USER` | Select by real user |
| `-p PID` | Select specified process IDs |
| `-C NAME` | Select by command name |
| `-t TTY` | Select by terminal |
| `-o FORMAT` | Choose output columns |
| `--sort=KEY` | Sort output by a field |
| `--forest` | Display parent-child hierarchy |
| `--ppid PID` | Select children of a parent PID |
| `-L` | Show threads in supported formats |
| `--help` | Show help |
| `--version` | Show version |

### Select specific columns

```bash
ps -eo pid,ppid,user,stat,%cpu,%mem,comm
```

Column meanings:
- `pid`: process ID
- `ppid`: parent process ID
- `user`: owner
- `stat`: process state
- `%cpu`: CPU usage
- `%mem`: memory usage
- `comm`: command name
- `args`: full command line

Examples:

```bash
# Show complete command lines
ps -eo pid,ppid,user,stat,%cpu,%mem,args

# Show one process with useful details
ps -p 1234 -o pid,ppid,user,stat,etime,%cpu,%mem,args

# Show processes using the most CPU
ps -eo pid,user,%cpu,%mem,comm --sort=-%cpu | head

# Show processes using the most memory
ps -eo pid,user,%mem,%cpu,comm --sort=-%mem | head
```

## 3. `top` — monitor processes live

Unlike `ps`, `top` repeatedly refreshes the process display.

```bash
top
top -u "$USER"
top -p 1234
top -b -n 1
top -b -n 5 -d 2
```

### Common options

| Option | Meaning |
|---|---|
| `-b` | Batch mode |
| `-n N` | Number of iterations |
| `-d N` | Delay between updates in seconds |
| `-p PID` | Monitor selected PIDs |
| `-u USER` | Show processes for a user |
| `-H` | Show threads |
| `-c` | Show or toggle command-line display, depending on version |
| `-o FIELD` | Select a sort field on supported versions |
| `-w N` | Set output width on supported versions |
| `-h` | Help |
| `-v` | Version |

### Useful keys inside `top`

| Key | Action |
|---|---|
| `q` | Quit |
| `h` | Help |
| `P` | Sort by CPU |
| `M` | Sort by memory |
| `N` | Sort by PID |
| `k` | Send a signal to a process |
| `r` | Change process niceness |
| `1` | Toggle per-CPU display on many versions |
| `c` | Toggle command display |
| `u` | Filter by user |

Use `h` inside `top` to see the keys supported by your installed version.

## 4. `htop` — interactive process monitor

`htop` provides an interactive display with convenient searching, filtering, and tree views.

```bash
htop
htop -u "$USER"
htop -p 1234
```

Install on Ubuntu or Debian if necessary:

```bash
sudo apt update
sudo apt install htop
```

Common keys:
- `F3`: search
- `F4`: filter
- `F5`: tree view
- `F6`: select sort column
- `F9`: send a signal to a selected process
- `F10`: quit

Keys can vary depending on terminal and version.

## 5. `pgrep` — find process IDs

`pgrep` searches for processes by name or pattern.

```bash
pgrep bash
pgrep -a ssh
pgrep -u "$USER" python
pgrep -x sshd
pgrep -af 'python.*app.py'
pgrep -c bash
```

### Common options

| Option | Meaning |
|---|---|
| `-a` | Print PID and full command line |
| `-f` | Match the full command line |
| `-x` | Require an exact process-name match |
| `-u USER` | Match effective user |
| `-U USER` | Match real user |
| `-n` | Newest matching process |
| `-o` | Oldest matching process |
| `-c` | Count matching processes |
| `-l` | List PID and process name |
| `-P PID` | Match children of a parent PID |
| `-t TTY` | Match by terminal |
| `-d DELIM` | Set delimiter between PIDs |
| `--help` | Display help |

Example:

```bash
# Find Python processes and show their command lines
pgrep -af python
```

## 6. `pidof` — find PIDs by program name

```bash
pidof bash
pidof sshd
```

`pidof` returns PIDs associated with a program name on systems that provide it. Its options vary by implementation, so check:

```bash
man pidof
```

## 7. `kill` — send signals to processes

Despite its name, `kill` sends a signal to a process. Its default signal is normally `SIGTERM`, which requests graceful termination.

```bash
kill 1234
kill -TERM 1234
kill -15 1234
kill -l
```

### Common signals

| Signal | Common Linux number | Purpose |
|---|---:|---|
| `SIGHUP` | 1 | Hangup; some programs reload configuration |
| `SIGINT` | 2 | Interrupt, similar to Ctrl+C |
| `SIGQUIT` | 3 | Quit; may generate a core dump |
| `SIGKILL` | 9 | Force termination |
| `SIGTERM` | 15 | Request graceful termination |
| `SIGSTOP` | 19 on common architectures | Stop a process |
| `SIGCONT` | 18 on common architectures | Continue a stopped process |
| `SIGTSTP` | 20 on common architectures | Terminal stop, commonly Ctrl+Z |

Signal numbers can vary by architecture. Prefer signal names for portability.

### Recommended termination procedure

```bash
# 1. Verify the process first
ps -p 1234 -o pid,user,stat,etime,args

# 2. Request graceful termination
kill -TERM 1234

# 3. Check whether it remains
sleep 2
ps -p 1234
```

If it is still present and you understand the consequences:

```bash
kill -KILL 1234
```

**Important:** `SIGKILL` cannot be caught or ignored. It prevents the process from performing cleanup and may cause data loss. Use it only as a last resort.

## 8. `pkill` — signal processes by pattern

`pkill` sends a signal to processes that match a name or pattern.

```bash
pkill -TERM -x myprogram
pkill -TERM -u "$USER" -f 'python.*test.py'
```

Common options include:
- `-TERM` or `-SIGNAL`: choose the signal.
- `-f`: match the full command line.
- `-x`: exact process-name match.
- `-u USER`: match effective user.
- `-U USER`: match real user.
- `-n`: newest match.
- `-o`: oldest match.
- `-P PID`: match children of a parent process.
- `-t TTY`: match by terminal.
- `-e`: report signaled processes.

### Safe usage

First preview the matches:

```bash
pgrep -af 'python.*test.py'
```

Then signal the intended processes:

```bash
pkill -TERM -f 'python.*test.py'
```

Be careful with `-f`: a broad pattern may match unrelated processes.

## 9. `killall` — signal processes by name

On many Linux distributions, `killall` signals processes by name.

```bash
killall -TERM myprogram
killall -i -TERM myprogram
```

Common options on the psmisc implementation include:

| Option | Meaning |
|---|---|
| `-e` | Require exact long-name matches |
| `-I` | Case-insensitive matching |
| `-i` | Ask before signaling each process |
| `-l` | List signal names |
| `-q` | Suppress warning if no process matches |
| `-r` | Treat the process name as a regular expression |
| `-s SIGNAL` | Select a signal |
| `-u USER` | Match processes owned by a user |
| `-v` | Report whether signaling succeeded |
| `-w` | Wait for processes to exit |
| `-V` | Show version |

Options differ across implementations. Check `man killall` before using it. Avoid broad names that might match important processes.

## 10. Background and foreground jobs

Shell job commands manage jobs started by the current shell. They do not list every process on the system.

### Start a background process

```bash
sleep 100 &
```

The `&` tells the shell to start the command asynchronously.

Capture its PID:

```bash
sleep 100 &
PID=$!
echo "$PID"
```

### `jobs` — list shell jobs

```bash
jobs
jobs -l
jobs -p
jobs -r
jobs -s
```

Common Bash options:
- `-l`: include PIDs.
- `-p`: show process IDs of job leaders.
- `-r`: show running jobs.
- `-s`: show stopped jobs.

### `bg` — resume a job in the background

Start a foreground command:

```bash
sleep 300
```

Press `Ctrl+Z` to suspend it, then:

```bash
jobs
bg %1
```

`%1` refers to job number 1.

### `fg` — bring a job to the foreground

```bash
fg %1
```

### `wait` — wait for a background process

```bash
sleep 2 &
PID=$!

wait "$PID"
echo "Exit status: $?"
```

### Keyboard shortcuts

- `Ctrl+C`: usually sends an interrupt to the foreground process group.
- `Ctrl+Z`: usually suspends the foreground job.
- `bg %1`: resume job 1 in the background.
- `fg %1`: bring job 1 to the foreground.

## 11. `nohup` — continue after terminal hangup

`nohup` runs a command while making it ignore hangup signals in the usual way.

```bash
nohup ./long-task.sh > task.log 2>&1 &
echo $!
```

Explanation:
- `nohup`: ignore hangup signals.
- `> task.log`: redirect standard output.
- `2>&1`: redirect standard error to the same destination.
- `&`: start in the background.

If output is not redirected, `nohup` may write to `nohup.out`.

`nohup` does not provide the full monitoring and restart capabilities of a service manager.

## 12. `disown` — detach jobs from Bash

```bash
long-command &
jobs -l
disown %1
```

In Bash, `disown` removes a job from the shell's job table or marks it not to receive `SIGHUP`, depending on the option.

Common Bash options:
- `-a`: all jobs.
- `-r`: running jobs only.
- `-h`: mark jobs so the shell does not send them `SIGHUP`.
- `-f`: remove a job from the job table where supported.

Example:

```bash
disown -h %1
```

## 13. `setsid` — start a new session

On systems that provide it:

```bash
setsid ./long-task.sh
```

`setsid` runs a command in a new session. Its options vary by implementation.

For reliable long-running applications, prefer a properly configured service when systemd is available.

## 14. `nice` — start with a scheduling priority

Linux nice values commonly range from `-20` to `19`:

- `-20`: higher scheduling priority.
- `0`: default niceness.
- `19`: lower scheduling priority.

Niceness influences CPU scheduling preference; it does not set a hard CPU usage limit.

```bash
nice -n 10 ./cpu-task.sh
nice -n 19 ./background-work.sh
```

Common GNU options:
- `-n N` or `--adjustment=N`: adjust niceness.
- `--help`: show help.
- `--version`: show version.

Ordinary users can generally make their processes less favored more easily than they can increase priority. Lower nice values often require elevated privilege.

## 15. `renice` — change an existing process's priority

```bash
renice 10 -p 1234
sudo renice -5 -p 1234
```

Common GNU procps options:
- `-n N`: set the new nice value.
- `-p`: target process IDs.
- `-g`: target process groups.
- `-u`: target users.
- `--help`: show help.
- `--version`: show version.

The exact behavior and permissions depend on the system. Raising priority generally requires elevated privilege.

## 16. `free` — view memory usage

```bash
free
free -h
free -m
free -s 2
```

Common options:

| Option | Meaning |
|---|---|
| `-b` | Bytes |
| `-k` | Kibibytes |
| `-m` | Mebibytes |
| `-g` | Gibibytes |
| `-h` | Human-readable units |
| `-w` | Separate memory columns on supported versions |
| `-s N` | Repeat every N seconds |
| `-c N` | Number of repetitions where supported |
| `-t` | Include a total line |
| `-l` | Show low/high memory details where supported |

Linux uses spare memory for caches that can often be reclaimed. The `available` column is generally more useful than looking only at `free`.

## 17. `uptime` — uptime and load average

```bash
uptime
```

Displays how long the system has been running, logged-in user count, and load averages for approximately 1, 5, and 15 minutes.

**Load average is not the same as CPU percentage.** On Linux, it also includes tasks in uninterruptible sleep.

## 18. `vmstat` — memory, processes, and system activity

```bash
vmstat
vmstat 2
vmstat 2 5
```

`vmstat` reports process queue, memory, swap, I/O, system activity, and CPU statistics.

In `vmstat 2 5`, the interval is 2 seconds and the number of reports is 5. The first report may summarize activity since boot.

## 19. `iostat` and `sar` — extended system statistics

These utilities are often provided by the `sysstat` package.

```bash
iostat -xz 2
sar -u 2 5
sar -r 2 5
```

- `iostat`: CPU and storage I/O statistics.
- `sar`: system activity statistics, including interval reports and historical data if collection is configured.

Install on Ubuntu/Debian if needed:

```bash
sudo apt update
sudo apt install sysstat
```

## 20. `pstree` — view process hierarchy

```bash
pstree
pstree -p
pstree -u
pstree -a
pstree -s 1234
pstree -p 1234
```

Common options on the psmisc implementation:
- `-p`: show PIDs.
- `-u`: show UID changes.
- `-a`: show command-line arguments.
- `-s`: show ancestors of a process.
- `-n`: sort siblings by PID.
- `-A`: use ASCII tree characters.
- `-l`: long lines.
- `-V`: show version.

Check `man pstree` for your installed version.

## 21. `/proc` — inspect process information

Linux exposes process information through the virtual `/proc` filesystem.

```bash
cat /proc/1234/status
cat /proc/1234/cmdline
readlink -f /proc/1234/exe
readlink -f /proc/1234/cwd
ls -l /proc/1234/fd/
cat /proc/meminfo
```

Replace `1234` with the process ID.

Useful paths:
- `/proc/PID/status`: IDs, state, and memory information.
- `/proc/PID/cmdline`: command-line arguments, separated by NUL bytes.
- `/proc/PID/exe`: link to the executable.
- `/proc/PID/cwd`: link to the working directory.
- `/proc/PID/fd/`: open file descriptors.
- `/proc/meminfo`: system memory information.
- `/proc/cpuinfo`: processor information.

Permissions or kernel settings may restrict access.

## 22. `lsof` — list open files

If installed:

```bash
lsof -p 1234
lsof -u "$USER"
lsof /path/to/file
sudo lsof -i :8000
```

Common options:
- `-p PID`: files opened by a process.
- `-u USER`: files opened by a user.
- `-i`: network sockets.
- `-i :PORT`: network activity on a port.
- `+D DIR`: recursively scan a directory; this can be expensive.
- `-n`: do not resolve network names.
- `-P`: do not resolve port names.

## 23. `systemctl` — manage system services

Services are often long-running processes managed by systemd. Use `systemctl` for a managed service rather than killing its process directly.

```bash
systemctl status cron
systemctl list-units --type=service --all
systemctl list-units --type=service --state=running
systemctl --failed --type=service
```

### Common service commands

```bash
sudo systemctl start SERVICE
sudo systemctl stop SERVICE
sudo systemctl restart SERVICE
sudo systemctl reload SERVICE
sudo systemctl enable SERVICE
sudo systemctl disable SERVICE
systemctl is-active SERVICE
systemctl is-enabled SERVICE
```

Meanings:
- `start`: start the service now.
- `stop`: stop it now.
- `restart`: stop and start it again.
- `reload`: ask it to reload configuration, if supported.
- `enable`: configure automatic startup at boot; does not necessarily start it now.
- `disable`: remove automatic-start configuration; does not necessarily stop it now.
- `is-active`: check whether it is active.
- `is-enabled`: check whether it is enabled at boot.

### View service logs

```bash
journalctl -u SERVICE
journalctl -u SERVICE -f
journalctl -u SERVICE --since today
```

Replace `SERVICE` with the actual service unit name.

On WSL, systemd may not be enabled, so these commands may not work.

## 24. `ss` — identify processes listening on ports

```bash
sudo ss -ltnp
sudo ss -ltnp 'sport = :8000'
```

Common options:
- `-l`: listening sockets.
- `-t`: TCP sockets.
- `-u`: UDP sockets.
- `-n`: numeric addresses and ports.
- `-p`: process information, when permitted.
- `-a`: all sockets.

Inspect a PID shown in the output:

```bash
ps -p 1234 -o pid,user,stat,args
```

## 25. `watch` — repeatedly monitor processes

```bash
watch -n 2 'ps -eo pid,stat,%cpu,%mem,comm --sort=-%cpu | head -15'
watch -n 2 -d 'free -h'
```

Common `watch` options:
- `-n SECONDS`: interval between updates.
- `-d`: highlight differences.
- `-t`: hide the header.
- `-b`: beep if the command exits unsuccessfully.
- `-e`: freeze on command errors until a key is pressed.
- `-g`: exit when output changes.
- `-p`: attempt precise intervals.
- `-x`: execute directly instead of through a shell.
- `-h`: show help.

Options vary by implementation. Check `watch --help`.

**Important:** `watch` reruns the command. Use it for monitoring, not for repeatedly executing tasks that modify files, send emails, or change databases unless repetition is intended.

## 26. Practical recipes

### Find a process by name

```bash
pgrep -a firefox
```

### Show the top CPU consumers

```bash
ps -eo pid,user,%cpu,%mem,comm --sort=-%cpu | head -15
```

### Show the top memory consumers

```bash
ps -eo pid,user,%mem,%cpu,comm --sort=-%mem | head -15
```

### Start a background task and log output

```bash
./long-task.sh > long-task.log 2>&1 &
PID=$!
echo "Started with PID $PID"
```

### Count processes

```bash
ps -e --no-headers | wc -l
```

### Show processes owned by the current user

```bash
ps -u "$USER" -f
```

### Follow a log file

```bash
tail -f application.log
```

Press `Ctrl+C` to stop following the log.

### Find the process using a port

```bash
sudo ss -ltnp 'sport = :8000'
```

## 27. Safety and troubleshooting

1. Verify the PID, user, and full command before sending a signal.
2. Prefer `SIGTERM`; use `SIGKILL` only as a last resort.
3. Avoid broad `pkill -f` patterns. Preview matches with `pgrep -af`.
4. Do not terminate essential system processes casually.
5. Use `systemctl` for systemd-managed services.
6. A zombie has already exited; its parent must reap it.
7. High load average is not the same as high CPU percentage.
8. Permissions may restrict viewing or signaling another user's process.
9. WSL, containers, and service managers can change how background processes behave.
10. Consult `man COMMAND` because options vary by distribution and version.

## 28. Practice exercises

1. List all processes:
   ```bash
   ps -ef
   ```

2. Show the top 10 CPU users:
   ```bash
   ps -eo pid,comm,%cpu --sort=-%cpu | head -10
   ```

3. Find Bash processes:
   ```bash
   pgrep -a bash
   ```

4. Start a background process and print its PID:
   ```bash
   sleep 120 &
   echo "$!"
   ```

5. List shell jobs:
   ```bash
   jobs -l
   ```

6. Gracefully stop a verified process (replace `PID` with its actual number):
   ```bash
   kill -TERM PID
   ```

7. Run a command with lower scheduling priority:
   ```bash
   nice -n 10 sha256sum large-file.iso
   ```

8. Check memory:
   ```bash
   free -h
   ```

9. Display the process tree:
   ```bash
   pstree -p
   ```

10. Monitor processes live:
    ```bash
    top
    ```

11. List listening ports:
    ```bash
    sudo ss -ltnp
    ```

12. Follow service logs:
    ```bash
    journalctl -u SERVICE -f
    ```

## 29. Final cheat sheet

| Goal | Command |
|---|---|
| List all processes | `ps -ef` |
| Live process monitor | `top` |
| Interactive process monitor | `htop` |
| Find a process | `pgrep -a NAME` |
| Find program PIDs | `pidof NAME` |
| Request termination | `kill -TERM PID` |
| Force termination (last resort) | `kill -KILL PID` |
| Signal by exact process name | `pkill -TERM -x NAME` |
| Start in background | `COMMAND &` |
| List current shell jobs | `jobs -l` |
| Resume a background job | `bg %1` |
| Bring a job to foreground | `fg %1` |
| Continue after terminal hangup | `nohup COMMAND > task.log 2>&1 &` |
| Lower scheduling priority | `nice -n 10 COMMAND` |
| Change existing niceness | `renice 10 -p PID` |
| Memory summary | `free -h` |
| Uptime and load | `uptime` |
| Process hierarchy | `pstree -p` |
| Listening ports | `sudo ss -ltnp` |
| Service status | `systemctl status SERVICE` |
| Follow service logs | `journalctl -u SERVICE -f` |

**Remember:** `ps` takes a snapshot, `top` monitors live activity, `pgrep` finds processes, `kill` sends signals, `jobs`/`bg`/`fg` manage jobs from your current shell, and `systemctl` manages systemd services.
