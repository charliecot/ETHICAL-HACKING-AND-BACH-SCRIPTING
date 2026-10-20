# Linux System Services, systemctl, SSH, Firewall, Apache & Networking Cheatsheet

A practical Ubuntu/Linux administration reference with commands, common options, explanations, and safe exercises.

> **Safety note:** Run administrative commands only on systems you own or are authorized to manage. Firewall and SSH changes can interrupt remote access. Keep an existing session open when changing remote firewall or SSH settings, and test a new connection before closing it.

## Contents

1. Understand services and `systemd`
2. `systemctl` reference
3. Read service logs with `journalctl`
4. SSH client and server
5. SSH keys and secure configuration
6. Firewall basics with UFW
7. Apache HTTP Server
8. Other useful services
9. Ports and network diagnostics
10. Processes and resource monitoring
11. Troubleshooting workflow
12. Ubuntu vs WSL notes
13. Safe practice lab
14. Quick command reference

---

## 1. Understand services and systemd

A **service** is a background program that provides a function, such as remote login (SSH), web hosting (Apache), time synchronization, or scheduled jobs.

On many modern Linux distributions, `systemd` is the service manager. Its command-line interface is `systemctl`.

Examples of service unit names:

- `ssh.service` — OpenSSH server on Ubuntu
- `apache2.service` — Apache web server on Ubuntu
- `cron.service` — scheduled jobs
- `systemd-timesyncd.service` — network time synchronization (if enabled)
- `NetworkManager.service` — network management on systems using NetworkManager
- `ufw.service` — UFW firewall integration may appear as a unit, but UFW is primarily managed with the `ufw` command

A service may be **active**, **inactive**, **failed**, or **not found**. Installed does not necessarily mean enabled or running.

Check whether systemd is PID 1:

```bash
ps -p 1 -o comm=
```

If it prints `systemd`, systemd is running as the init system. In WSL, systemd support depends on the WSL configuration and distribution setup.

## 2. `systemctl` command reference


### a. in order to write your own serviecs

[Unit]
# Description of the service
Description=My Bash Application Service

# Start after the basic network target
After=network.target

[Service]
# Run the service as a one-time task
Type=oneshot

# Command to execute
ExecStart=/bin/bash /home/YOUR_USERNAME/myapp/myapp.sh

[Install]
# Allow the service to be enabled for normal startup
WantedBy=multi-user.target


### Check status

```bash
systemctl status ssh
systemctl status apache2
systemctl status cron
```

`status` displays whether the unit is active, recent log messages, and the main process information.

### Start, stop, restart, reload

```bash
sudo systemctl start ssh
sudo systemctl stop ssh
sudo systemctl restart ssh
sudo systemctl reload apache2
```

| Command | Meaning | Typical use |
|---|---|---|
| `start UNIT` | Start now | Start SSH |
| `stop UNIT` | Stop now | Stop a service temporarily |
| `restart UNIT` | Stop then start | Apply changes when reload is unsupported |
| `reload UNIT` | Ask service to reload configuration | Reload Apache config without a full restart, if supported |
| `try-restart UNIT` | Restart only if already running | Avoid starting an inactive service |
| `reload-or-restart UNIT` | Reload if possible, otherwise restart | Apply a change when supported |
| `kill UNIT` | Send a signal to a unit's processes | Advanced troubleshooting; use carefully |

### Enable and disable at boot

```bash
sudo systemctl enable ssh
sudo systemctl disable ssh
sudo systemctl enable --now ssh
sudo systemctl disable --now ssh
```

- `enable` configures the service to start at boot; it does not necessarily start it now.
- `disable` removes the boot-start link; it does not necessarily stop a currently running service.
- `--now` also starts or stops the service immediately.

### Check enabled/running state

```bash
systemctl is-active ssh
systemctl is-enabled ssh
systemctl is-failed ssh
```

These are useful in scripts because they return a status code as well as output.

### List services

```bash
systemctl list-units --type=service
systemctl list-units --type=service --state=running
systemctl list-units --type=service --state=failed
systemctl list-unit-files --type=service
```

Difference:

- `list-units` shows units systemd currently knows about, usually loaded units.
- `list-unit-files` shows installed unit files and their enablement state.

### Find a unit and its configuration

```bash
systemctl cat ssh
systemctl show ssh
systemctl show ssh --property=ActiveState --property=SubState
systemctl list-dependencies apache2
```

`cat` shows the unit file and drop-in configuration. `show` exposes systemd properties. `list-dependencies` displays dependency relationships.

### Validate unit files and reload systemd

```bash
sudo systemd-analyze verify /etc/systemd/system/example.service
sudo systemctl daemon-reload
```

Run `daemon-reload` after creating or changing systemd unit files. It tells systemd to reread unit definitions; it does **not** reload an application's own configuration.

### Mask and unmask (advanced)

```bash
sudo systemctl mask example.service
sudo systemctl unmask example.service
```

Masking prevents a unit from being started, including as a dependency, by linking it to `/dev/null`. Do not mask critical services unless you understand the consequences.

### Common `systemctl` options

| Option | Purpose | Example |
|---|---|---|
| `--type=service` | Filter to service units | `systemctl list-units --type=service` |
| `--state=running` | Filter by state | `systemctl list-units --state=running` |
| `--all` | Include inactive units in many listing commands | `systemctl list-units --all` |
| `--no-pager` | Print directly instead of opening a pager | `systemctl status ssh --no-pager` |
| `--full` | Avoid truncating unit names/descriptions in supported output | `systemctl --full status ssh` |
| `--user` | Manage the current user's systemd units | `systemctl --user status` |
| `--failed` | Show failed units (supported shortcut) | `systemctl --failed` |

Check your local help:

```bash
systemctl --help
man systemctl
```

---

## 3. Read service logs with journalctl

`journalctl` reads logs collected by the systemd journal.

```bash
# Show recent system journal entries
journalctl -n 50

# Follow new messages live
journalctl -f

# Logs for SSH
sudo journalctl -u ssh

# Recent SSH logs
sudo journalctl -u ssh -n 100

# Follow Apache logs
sudo journalctl -u apache2 -f

# Logs from the current boot
journalctl -b

# Logs since a time
journalctl --since "1 hour ago"

# Logs for a priority and above (err, warning, info, debug)
journalctl -p warning
```

| Option | Meaning |
|---|---|
| `-u UNIT` | Filter by unit |
| `-f` | Follow new entries |
| `-n N` | Show the last N entries |
| `-b` | Current boot; `-b -1` means previous boot if retained |
| `--since TIME` | Start time filter |
| `--until TIME` | End time filter |
| `-p PRIORITY` | Filter by priority |
| `--no-pager` | Do not use a pager |
| `-xe` | Common troubleshooting view: recent entries with explanations |

Example:

```bash
sudo journalctl -u ssh --since today --no-pager
```

Some application logs are written to their own files rather than the journal. Apache on Ubuntu commonly writes to `/var/log/apache2/`.

---

## 4. SSH: Secure Shell

SSH allows secure remote command execution and file transfer.

Two parts:

- **SSH client:** connects to another machine (`ssh` command).
- **SSH server:** accepts connections (`sshd`, usually managed as `ssh.service` on Ubuntu).

### Install the client/server

```bash
# Install the client
sudo apt update
sudo apt install openssh-client

# Install the server, if you want this machine to accept SSH connections
sudo apt install openssh-server
```

Check the server:

```bash
systemctl status ssh
sudo ss -ltnp | grep ':22'
```

Port `22` is the default SSH port, but a server can be configured to use another port.

### Connect to another machine

```bash
ssh username@server.example.com
ssh username@192.0.2.10
ssh -p 2222 username@server.example.com
```

| Option | Purpose | Example |
|---|---|---|
| `-p PORT` | Use a non-default server port (client) | `ssh -p 2222 user@host` |
| `-i FILE` | Use a specified private key | `ssh -i ~/.ssh/id_ed25519 user@host` |
| `-v` | Verbose diagnostics; repeat for more detail | `ssh -vv user@host` |
| `-N` | Do not run a remote command; useful for forwarding | `ssh -N user@host` |
| `-T` | Disable pseudo-terminal allocation | `ssh -T user@host` |
| `-L` | Local port forwarding | `ssh -L 8080:127.0.0.1:80 user@host` |
| `-J` | Connect through a jump host | `ssh -J user@bastion host` |
| `-o OPTION=VALUE` | Set a configuration option | `ssh -o ConnectTimeout=10 user@host` |

Use real hostnames and usernames. Documentation-only IPs such as `192.0.2.10` are examples, not real servers to connect to.

### Run one remote command

```bash
ssh username@server.example.com 'whoami && hostname'
```

### Copy files with SCP

```bash
# Copy local file to remote machine
scp ./report.txt username@server.example.com:/home/username/

# Copy remote file to local directory
scp username@server.example.com:/home/username/report.txt .

# Use a non-default SSH port (SCP's -P is uppercase)
scp -P 2222 ./report.txt username@server.example.com:/home/username/
```

### Transfer files with SFTP

```bash
sftp username@server.example.com
```

Inside the interactive SFTP prompt, common commands include `ls`, `pwd`, `cd`, `lcd`, `get`, `put`, and `bye`.

### Generate an SSH key

```bash
ssh-keygen -t ed25519 -C "your-device-key"
```

Accept the default path if appropriate and consider protecting the private key with a passphrase.

- Private key: `~/.ssh/id_ed25519` — keep secret.
- Public key: `~/.ssh/id_ed25519.pub` — can be copied to the server.

Install the public key on a server you administer:

```bash
ssh-copy-id username@server.example.com
```

Then test:

```bash
ssh username@server.example.com
```

Never share your private key or paste it into chats or websites.

### SSH server configuration

Ubuntu's main configuration is typically:

```text
/etc/ssh/sshd_config
```

Check the effective configuration:

```bash
sudo sshd -t
sudo sshd -T | less
```

- `sshd -t` tests configuration syntax.
- `sshd -T` prints effective settings.

Edit only when needed:

```bash
sudoedit /etc/ssh/sshd_config
```

After a configuration change:

```bash
sudo sshd -t && sudo systemctl reload ssh
```

If reload is unsupported or fails, inspect the service status and logs before deciding whether to restart.

Security guidance:

- Prefer SSH keys over passwords where practical.
- Use strong account passwords and keep the system updated.
- Avoid permitting direct root login over SSH.
- Do not disable password authentication until key-based login has been tested in a separate session.
- Do not change SSH ports or firewall rules on a remote server without a fallback access path.

Relevant configuration directives can include `PermitRootLogin`, `PasswordAuthentication`, `PubkeyAuthentication`, `AllowUsers`, and `Port`. Their defaults and included configuration files vary by distribution and version. Understand each setting before changing it.

---

## 5. Firewall basics with UFW

A firewall controls network traffic. Ubuntu commonly uses **UFW** (Uncomplicated Firewall) as a simpler frontend to Linux firewalling.

Check status first:

```bash
sudo ufw status
sudo ufw status verbose
sudo ufw status numbered
```

### Basic UFW commands

```bash
# Allow SSH before enabling UFW on a machine you access remotely
sudo ufw allow OpenSSH

# Allow web traffic
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable firewall
sudo ufw enable

# Disable firewall (use only when you understand the impact)
sudo ufw disable

# Delete a rule
sudo ufw delete allow 80/tcp

# Show numbered rules and delete by number
sudo ufw status numbered
sudo ufw delete 2

# Reload firewall rules
sudo ufw reload

# Reset all UFW rules (destructive to current rule configuration)
sudo ufw reset
```

**Important:** Before enabling UFW over SSH, allow the actual SSH service/port first, or you may lock yourself out. Verify the port your server uses.

### Restrict access to a source address

Example: allow SSH only from a trusted IP address:

```bash
sudo ufw allow from 203.0.113.25 to any port 22 proto tcp
```

`203.0.113.25` is a documentation-only example address; replace it with the trusted source IP. If your IP changes, the rule may no longer work.

Allow traffic from a subnet:

```bash
sudo ufw allow from 192.168.1.0/24 to any port 22 proto tcp
```

Use the subnet that matches your real network, not this example blindly.

### Common UFW syntax

```bash
sudo ufw allow PORT
sudo ufw allow PORT/tcp
sudo ufw deny PORT/tcp
sudo ufw allow from SOURCE_IP
sudo ufw allow from SOURCE_IP to any port PORT proto tcp
sudo ufw delete RULE
```

Useful commands:

```bash
ufw --help
man ufw
```

Do not mix UFW with manual firewall rules from other tools without understanding how the rules interact.

---

## 6. Apache HTTP Server

Apache serves websites over HTTP/HTTPS. On Ubuntu, the service and package are commonly named `apache2`.

### Install and check status

```bash
sudo apt update
sudo apt install apache2

systemctl status apache2
```

Start now and enable at boot:

```bash
sudo systemctl enable --now apache2
```

### Validate and reload configuration

```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
```

`configtest` should report `Syntax OK` before you reload after editing configuration.

Restart when necessary:

```bash
sudo systemctl restart apache2
```

A reload generally applies configuration changes with less disruption than a restart, when supported.

### Common Apache commands

```bash
# Show build and module/config information
apache2ctl -V

# List virtual hosts (if supported by installed version)
sudo apache2ctl -S

# List enabled modules
apache2ctl -M

# Enable a site
sudo a2ensite example.conf

# Disable a site
sudo a2dissite example.conf

# Enable a module
sudo a2enmod rewrite

# Disable a module
sudo a2dismod rewrite
```

After enabling/disabling a site or module, test and reload:

```bash
sudo apache2ctl configtest && sudo systemctl reload apache2
```

### Common locations on Ubuntu

| Path | Purpose |
|---|---|
| `/etc/apache2/apache2.conf` | Main configuration |
| `/etc/apache2/ports.conf` | Listen ports |
| `/etc/apache2/sites-available/` | Available virtual host configurations |
| `/etc/apache2/sites-enabled/` | Enabled site configurations, usually symlinks |
| `/etc/apache2/mods-available/` | Available modules |
| `/etc/apache2/mods-enabled/` | Enabled modules |
| `/var/www/html/` | Default document root |
| `/var/log/apache2/access.log` | HTTP request log |
| `/var/log/apache2/error.log` | Error log |

### Test the local web server

```bash
curl -I http://127.0.0.1/
```

The `-I` option asks for response headers. A successful local response means Apache responded locally; it does not automatically prove the site is reachable from another machine.

### Apache and firewall

If hosting a public HTTP site, allow only the services you intend to expose:

```bash
sudo ufw allow 'Apache'
```

Or allow specific ports:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Use HTTPS for real sites that transmit credentials or private information. TLS certificate setup is a separate task.

---

## 7. Other useful services

### Cron: scheduled jobs

```bash
systemctl status cron
sudo systemctl enable --now cron
```

Edit the current user's cron schedule:

```bash
crontab -e
```

List scheduled jobs:

```bash
crontab -l
```

A cron line has five time/date fields followed by a command:

```text
minute hour day-of-month month day-of-week command
```

Example: run a script at 02:30 every day:

```cron
30 2 * * * /home/youruser/scripts/backup.sh
```

Use absolute paths in cron jobs and redirect output to a log if you need to troubleshoot.

### Time synchronization

```bash
timedatectl status
systemctl status systemd-timesyncd
```

The time-sync service may vary by system. Do not enable competing time services without understanding which one your distribution uses.

### NetworkManager

On systems using NetworkManager:

```bash
systemctl status NetworkManager
nmcli device status
nmcli connection show
```

Do not restart network services on a remote machine unless you have console or other fallback access.

### Fail2ban (optional intrusion-prevention helper)

Install only if you need it and understand its configuration:

```bash
sudo apt install fail2ban
systemctl status fail2ban
```

Fail2ban can monitor logs and temporarily block addresses after repeated failures. It is not a replacement for secure SSH settings, updates, or a firewall.

### Docker (if installed)

```bash
systemctl status docker
sudo systemctl enable --now docker
docker ps
```

Docker may create firewall rules and expose container ports. Review the security implications before exposing containers publicly.

### PostgreSQL and MySQL/MariaDB (if installed)

Possible unit names vary:

```bash
systemctl status postgresql
systemctl status mysql
systemctl status mariadb
```

Do not expose database ports to the internet by default. Prefer local-only access or tightly restricted network access.

---

## 8. Ports and network diagnostics

### Show listening ports and owning processes

```bash
sudo ss -lntup
```

Options:

- `-l` — listening sockets
- `-n` — numeric addresses and ports
- `-t` — TCP
- `-u` — UDP
- `-p` — process information (may require `sudo`)

Examples:

```bash
sudo ss -ltnp
sudo ss -lunp
sudo ss -ltnp | grep ':22'
sudo ss -ltnp | grep -E ':(80|443)\b'
```

### Check connectivity

```bash
ping -c 4 example.com
getent hosts example.com
curl -I --max-time 10 https://example.com
```

- `ping` tests ICMP reachability; some networks block it.
- `getent hosts` tests name resolution using system name-service configuration.
- `curl -I` checks an HTTP response's headers.
- `--max-time 10` limits the total curl operation to 10 seconds.

### Check routes and addresses

```bash
ip address
ip route
ip link
```

### Check a port from the local machine

If `nc` (netcat) is installed:

```bash
nc -vz 127.0.0.1 22
nc -vz 127.0.0.1 80
```

This tests whether a TCP connection can be established to that address and port. It does not test every part of the service's functionality.

---

## 9. Processes and resource monitoring

### `ps`

```bash
ps aux
ps aux | grep apache2
ps -p 1 -o pid,comm,args
```

### `top`

```bash
top
```

Interactive process monitor. Press `q` to quit.

### `free`

```bash
free -h
```

Shows memory and swap usage in human-readable units.

### `df` and `du`

```bash
df -h
du -sh ~/some_directory
```

- `df -h` — filesystem space.
- `du -sh PATH` — approximate disk usage of a path.

### `kill`

```bash
kill PID
kill -TERM PID
```

`TERM` asks a process to exit gracefully. Use the PID from a trustworthy source such as `ps` or `systemctl status`. Avoid `kill -9` unless a normal shutdown fails and you understand the consequences; it prevents cleanup.

---

## 10. Troubleshooting workflow

When a service does not work, inspect it in this order.

### Step 1: Is it installed?

```bash
dpkg -l | grep -E 'openssh-server|apache2|ufw'
```

Package names depend on the service.

### Step 2: Does the unit exist and what is its state?

```bash
systemctl status ssh --no-pager
systemctl status apache2 --no-pager
```

### Step 3: Read logs

```bash
sudo journalctl -u ssh -n 100 --no-pager
sudo journalctl -u apache2 -n 100 --no-pager
```

For Apache, also inspect:

```bash
sudo tail -n 50 /var/log/apache2/error.log
```

### Step 4: Is the application listening?

```bash
sudo ss -lntup
```

A service can be running but listening only on `127.0.0.1`, making it unavailable from other machines.

### Step 5: Is the configuration valid?

```bash
sudo sshd -t
sudo apache2ctl configtest
```

Use the command that matches the service.

### Step 6: Is the firewall permitting the intended traffic?

```bash
sudo ufw status verbose
```

Also consider cloud-provider firewalls, router rules, Windows firewall, and network routing where relevant.

### Step 7: Test locally before testing remotely

```bash
curl -I http://127.0.0.1/
ssh -v username@server.example.com
```

Use the appropriate local test for the service and a real server address for remote SSH.

---

## 11. Ubuntu vs WSL notes

WSL is a Linux environment hosted by Windows. Behavior depends on WSL version, configuration, and whether systemd is enabled.

Check WSL/Linux details:

```bash
uname -a
cat /etc/os-release
ps -p 1 -o comm=
```

If systemd is not running as PID 1, `systemctl` may return a message saying the system was not booted with systemd.

On supported WSL versions, systemd can be enabled through `/etc/wsl.conf`:

```ini
[boot]
systemd=true
```

After changing WSL configuration, from Windows PowerShell run:

```powershell
wsl --shutdown
```

Then start the distribution again. Check current Microsoft WSL documentation for your version before changing configuration.

Additional notes:

- A service running inside WSL may not be exposed to other computers automatically.
- Windows firewall and WSL networking settings can affect inbound access.
- Windows-mounted paths under `/mnt/c` can have different permission behavior from the Linux filesystem.
- A web server inside WSL is useful for local development, but production hosting needs appropriate network, security, and availability planning.

---

## 12. Safe practice lab

This lab inspects service management and a local HTTP endpoint without changing firewall rules.

```bash
# 1. Identify init process
ps -p 1 -o comm=

# 2. Inspect service state (some services may not be installed)
systemctl status ssh --no-pager
systemctl status apache2 --no-pager

# 3. List failed services
systemctl --failed

# 4. Read recent SSH logs if SSH is installed
sudo journalctl -u ssh -n 20 --no-pager

# 5. Inspect listening TCP ports
sudo ss -ltnp

# 6. If Apache is installed, validate its configuration
sudo apache2ctl configtest

# 7. If Apache is running, test it locally
curl -I --max-time 5 http://127.0.0.1/
```

Optional Apache practice, if you want a local web server:

```bash
sudo apt update
sudo apt install apache2
sudo apache2ctl configtest
sudo systemctl enable --now apache2
curl -I http://127.0.0.1/
```

Do not enable the firewall or modify remote SSH settings as a first exercise. Learn to inspect current state before making changes.

---

## 13. Quick command reference

```bash
# systemd / services
systemctl status SERVICE
sudo systemctl start SERVICE
sudo systemctl stop SERVICE
sudo systemctl restart SERVICE
sudo systemctl reload SERVICE
sudo systemctl enable --now SERVICE
sudo systemctl disable --now SERVICE
systemctl is-active SERVICE
systemctl is-enabled SERVICE
systemctl --failed
systemctl list-units --type=service --state=running

# Logs
journalctl -u SERVICE -n 50 --no-pager
sudo journalctl -u SERVICE -f
journalctl -b
journalctl --since "1 hour ago"

# SSH
ssh user@host
ssh -p 2222 user@host
ssh -i ~/.ssh/id_ed25519 user@host
ssh -v user@host
ssh-keygen -t ed25519
ssh-copy-id user@host
scp ./file.txt user@host:/path/

# UFW
sudo ufw status verbose
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw delete allow 80/tcp

# Apache on Ubuntu
sudo apache2ctl configtest
sudo apache2ctl -S
sudo systemctl reload apache2
sudo systemctl restart apache2
curl -I http://127.0.0.1/

# Network diagnostics
ip address
ip route
sudo ss -lntup
getent hosts example.com
ping -c 4 example.com
curl -I --max-time 10 https://example.com

# Resources
top
free -h
df -h
ps aux
```

Replace placeholders such as `SERVICE`, `user`, `host`, and `/path/` with real values. Do not paste every command as a batch; select the command appropriate to the service and task.

## 14. Further reading

```bash
man systemctl
man journalctl
man ssh
man sshd_config
man ufw
man apache2
man ss
```

Useful documentation sites:

- systemd: https://www.freedesktop.org/software/systemd/man/
- Ubuntu Server documentation: https://ubuntu.com/server/docs
- OpenSSH manual pages: https://man.openbsd.org/ssh
- Apache HTTP Server documentation: https://httpd.apache.org/docs/
- Microsoft WSL documentation: https://learn.microsoft.com/windows/wsl/
