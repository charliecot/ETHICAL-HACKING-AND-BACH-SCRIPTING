# Linux Important File Paths and Configuration Reference

A practical reference for Ubuntu, Debian, Linux administration, networking, security, systemd, cron, storage, logs, web servers, and WSL.

---

## 1. Basic Linux Directories

| Path | Purpose |
|---|---|
| `/` | Root of the entire filesystem |
| `/etc/` | System-wide configuration files |
| `/home/` | Regular users' home directories |
| `/root/` | Root user's home directory |
| `/root/.config/` | Root user's application configuration, when used |
| `/usr/` | Installed programs, libraries, and shared resources |
| `/usr/bin/` | Common executable programs |
| `/usr/sbin/` | System administration programs on many distributions |
| `/usr/local/` | Locally installed software |
| `/usr/local/bin/` | Locally installed commands |
| `/usr/local/sbin/` | Locally installed administration commands |
| `/usr/lib/` | Libraries and application support files |
| `/usr/share/` | Architecture-independent data and documentation |
| `/opt/` | Optional or third-party software |
| `/var/` | Variable data such as logs, caches, and databases |
| `/tmp/` | Temporary files |
| `/var/tmp/` | Temporary files that may survive reboots |
| `/run/` | Runtime state, sockets, and PID files |
| `/mnt/` | Common manual mount location |
| `/media/` | Common removable-media mount location |
| `/dev/` | Device files |
| `/proc/` | Virtual filesystem containing process and kernel information |
| `/sys/` | Virtual filesystem exposing kernel devices and subsystems |
| `/srv/` | Data provided by system services |
| `/lost+found/` | Filesystem recovery directory on applicable filesystems |

Commands:

```bash
pwd
ls -lah /
ls -lah /etc/
ls -lah /var/
df -h
findmnt
```

## 2. Important System Configuration Files

| File | Purpose |
|---|---|
| `/etc/hostname` | System hostname |
| `/etc/hosts` | Local hostname-to-IP mappings |
| `/etc/os-release` | Linux distribution information |
| `/etc/fstab` | Persistent filesystem mount definitions |
| `/etc/environment` | System-wide environment variables |
| `/etc/profile` | System-wide login-shell configuration |
| `/etc/shells` | Recognized login shells |
| `/etc/issue` | Text displayed before some login prompts |
| `/etc/motd` | Message displayed after login on configured systems |
| `/etc/nsswitch.conf` | Name service lookup configuration |
| `/etc/locale.conf` | Locale configuration on systems that use this file |
| `/etc/timezone` | Timezone name on some Debian/Ubuntu installations |
| `/etc/localtime` | System timezone data, often a symlink |
| `/etc/machine-id` | Unique machine identifier |
| `/etc/sysctl.conf` | Kernel parameter configuration |
| `/etc/sysctl.d/` | Additional kernel parameter configuration |
| `/etc/modules-load.d/` | Configuration for modules loaded at boot |
| `/etc/modprobe.d/` | Kernel module configuration |

Useful commands:

```bash
cat /etc/hostname
cat /etc/os-release
cat /etc/hosts
cat /etc/fstab
cat /etc/nsswitch.conf
timedatectl
hostnamectl
```

Not every distribution uses every file listed above. Some settings are generated or managed by other services.

## 3. Users, Groups, and Permissions

| File or directory | Purpose |
|---|---|
| `/etc/passwd` | User account information |
| `/etc/shadow` | Password hashes and password-aging information |
| `/etc/group` | Group information |
| `/etc/gshadow` | Restricted group security information |
| `/etc/sudoers` | Sudo permissions and policy |
| `/etc/sudoers.d/` | Additional sudo policy files |
| `/etc/login.defs` | Defaults for user account management tools |
| `/etc/skel/` | Default files copied when creating users |
| `/etc/pam.d/` | PAM authentication configuration |
| `/etc/security/` | Additional security-related configuration |

Commands:

```bash
whoami
id
groups
getent passwd
getent group
sudo visudo -c
sudo passwd username
```

Important notes:

- `/etc/passwd` is not where ordinary plaintext passwords are stored.
- `/etc/shadow` is restricted because it contains password hashes and related information.
- Use `sudo visudo` to edit sudo policy safely.
- Use `chmod` to change permissions and `chown` to change ownership.

Examples:

```bash
ls -l file.txt
chmod 640 file.txt
chown username:groupname file.txt
```

## 4. Networking Configuration

| Path | Purpose |
|---|---|
| `/etc/hosts` | Local hostname resolution |
| `/etc/resolv.conf` | DNS resolver configuration; often automatically managed |
| `/etc/hostname` | Machine hostname |
| `/etc/netplan/` | Netplan YAML network configuration on applicable Ubuntu systems |
| `/etc/NetworkManager/` | NetworkManager configuration |
| `/etc/systemd/resolved.conf` | Configuration for systemd-resolved |
| `/etc/nsswitch.conf` | Lookup order for hosts, users, and other databases |
| `/etc/ssh/sshd_config` | SSH server configuration |
| `/etc/ssh/ssh_config` | SSH client configuration |
| `/etc/ufw/` | UFW firewall configuration |
| `/proc/net/` | Kernel-provided network information |

Commands:

```bash
ip address
ip route
ip link
hostname -I
cat /etc/hosts
cat /etc/resolv.conf
ls -lah /etc/netplan/
ls -lah /etc/NetworkManager/
```

Validate SSH configuration before restarting the SSH service:

```bash
sudo sshd -t
```

Inspect DNS configuration management:

```bash
ls -l /etc/resolv.conf
resolvectl status
```

If `resolvectl` is unavailable, your system may not use systemd-resolved.

## 5. SSH and Remote Access

| Path | Purpose |
|---|---|
| `/etc/ssh/` | System SSH configuration and host keys |
| `/etc/ssh/sshd_config` | SSH server settings |
| `/etc/ssh/ssh_config` | Default SSH client settings |
| `/etc/ssh/sshd_config.d/` | Additional server configuration files on systems configured to use them |
| `~/.ssh/` | Per-user SSH keys and configuration |
| `~/.ssh/config` | Per-user SSH client settings |
| `~/.ssh/authorized_keys` | Public keys allowed to log in to that user's account |
| `~/.ssh/known_hosts` | Previously recorded remote host keys |
| `/etc/ssh/ssh_host_*` | System SSH host keys |

Commands:

```bash
ssh username@server-ip
ssh-keygen
ssh-keygen -lf ~/.ssh/id_ed25519.pub
ls -lah ~/.ssh/
sudo sshd -t
```

Security:

- Never share private keys such as `~/.ssh/id_ed25519`.
- Protect private keys with suitable permissions.
- A common private-key permission is `600`.
- A common `.ssh` directory permission is `700`.

Example:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

Only run permission changes on files you own and understand.

## 6. systemd Services and Timers

| Path | Purpose |
|---|---|
| `/etc/systemd/system/` | Locally created or administrator-managed system units |
| `/usr/lib/systemd/system/` | Vendor-provided units on many distributions |
| `/lib/systemd/system/` | Vendor unit location on some filesystem layouts |
| `/run/systemd/system/` | Runtime system units |
| `~/.config/systemd/user/` | Per-user service and timer units |
| `/etc/systemd/journald.conf` | Journal logging configuration |
| `/etc/systemd/journald.conf.d/` | Additional journal configuration |
| `/etc/systemd/logind.conf` | Login manager configuration |
| `/etc/systemd/system.conf` | System service manager configuration |

Examples of unit files:

```text
/etc/systemd/system/mytask.service
/etc/systemd/system/mytask.timer
~/.config/systemd/user/mytask.service
~/.config/systemd/user/mytask.timer
```

Useful commands:

```bash
systemctl list-units --type=service
systemctl list-timers --all
systemctl status ssh.service
systemctl cat mytask.timer
systemctl show mytask.timer -p FragmentPath
```

After changing a system unit:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now mytask.timer
systemctl status mytask.timer
```

For a system service, common commands are:

```bash
sudo systemctl start service-name
sudo systemctl stop service-name
sudo systemctl restart service-name
sudo systemctl enable service-name
sudo systemctl disable service-name
sudo journalctl -u service-name -n 50 --no-pager
```

### Example systemd service

File: `/etc/systemd/system/mytask.service`

```ini
[Unit]
Description=Run my Bash script

[Service]
Type=oneshot
ExecStart=/home/YOUR_USERNAME/scripts/mytask.sh
```

Replace `YOUR_USERNAME` with your actual Linux username.

### Example systemd timer

File: `/etc/systemd/system/mytask.timer`

```ini
[Unit]
Description=Run my task every five minutes

[Timer]
OnBootSec=1min
OnUnitActiveSec=5min
AccuracySec=1s
Unit=mytask.service

[Install]
WantedBy=timers.target
```

Activate it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now mytask.timer
systemctl list-timers --all
```

`OnUnitActiveSec=5min` schedules the next activation relative to the associated unit's last activation. `AccuracySec=1s` controls the timer's scheduling accuracy window; it is not a guarantee that the script will finish or even start at an exact wall-clock instant.

## 7. Cron and Scheduled Tasks

| Path | Purpose |
|---|---|
| `/etc/crontab` | System-wide cron schedule with a user field |
| `/etc/cron.d/` | Additional system cron schedules |
| `/etc/cron.hourly/` | Scripts intended for hourly execution |
| `/etc/cron.daily/` | Scripts intended for daily execution |
| `/etc/cron.weekly/` | Scripts intended for weekly execution |
| `/etc/cron.monthly/` | Scripts intended for monthly execution |
| `/var/spool/cron/crontabs/` | Per-user crontabs on many Debian/Ubuntu systems |
| `/etc/anacrontab` | Anacron schedules on systems where Anacron is installed |

Commands:

```bash
crontab -l
crontab -e
sudo crontab -l
```

Example: run a script every five minutes.

```cron
*/5 * * * * /home/YOUR_USERNAME/scripts/backup.sh
```

Cron fields:

```text
minute hour day-of-month month day-of-week command
```

Example: run daily at 06:30.

```cron
30 6 * * * /home/YOUR_USERNAME/scripts/backup.sh
```

Cron and systemd timers are different scheduling systems. Choose one based on your needs and what is available in your environment.

## 8. Logs and Troubleshooting

| Path | Purpose |
|---|---|
| `/var/log/` | Main log directory |
| `/var/log/syslog` | General system logs on systems using traditional syslog |
| `/var/log/auth.log` | Authentication logs on many Ubuntu/Debian systems |
| `/var/log/kern.log` | Kernel log file on some installations |
| `/var/log/dpkg.log` | Package installation and removal history |
| `/var/log/apt/` | APT logs |
| `/var/log/nginx/` | Nginx access and error logs when configured with standard paths |
| `/var/log/apache2/` | Apache logs on Ubuntu/Debian with standard configuration |
| `/var/log/journal/` | Persistent systemd journal data, if enabled |
| `/run/log/journal/` | Runtime journal data on some systems |

Commands:

```bash
journalctl -n 50 --no-pager
journalctl -f
journalctl -b
journalctl -p err
sudo journalctl -u mytask.service -n 50 --no-pager
sudo less /var/log/auth.log
```

Log files vary by distribution and logging configuration. `journalctl` is especially useful on systems using systemd.

## 9. Bash and User Configuration

| Path | Purpose |
|---|---|
| `~/.bashrc` | Interactive Bash configuration |
| `~/.bash_profile` | Bash login configuration on systems using this file |
| `~/.profile` | User login environment configuration |
| `~/.bash_history` | Bash command history, when enabled |
| `~/.ssh/` | User SSH configuration and keys |
| `~/.config/` | Per-user application settings |
| `~/.local/bin/` | Common location for personal executable scripts |
| `~/.local/share/` | Per-user application data |
| `/etc/profile` | System-wide login-shell configuration |
| `/etc/profile.d/` | Additional shell initialization scripts |

Commands:

```bash
nano ~/.bashrc
source ~/.bashrc
history
echo "$PATH"
```

Example alias in `~/.bashrc`:

```bash
# List files with details, including hidden files
alias ll='ls -lah'

# Display the current directory
alias croot='cd /'
```

Apply changes:

```bash
source ~/.bashrc
```

## 10. Packages, Programs, and Libraries

| Path | Purpose |
|---|---|
| `/usr/bin/` | Common executables |
| `/usr/sbin/` | System administration executables on many systems |
| `/usr/local/bin/` | Locally installed executables |
| `/usr/local/sbin/` | Locally installed administration tools |
| `/usr/lib/` | Libraries and program support files |
| `/usr/share/` | Shared application data and documentation |
| `/lib/` | Essential libraries and modules on layouts that use it |
| `/opt/` | Optional third-party applications |
| `/etc/ld.so.conf` | Dynamic linker configuration entry point |
| `/etc/ld.so.conf.d/` | Additional dynamic linker configuration |
| `/var/lib/dpkg/` | Debian package database |
| `/var/cache/apt/` | APT cache |

Commands:

```bash
command -v python3
which python3
whereis python3
dpkg -L bash
dpkg -S /usr/bin/bash
apt list --installed
```

Use `command -v` as a good first choice for checking which executable or shell command will be used.

## 11. Storage and Mounting

| Path | Purpose |
|---|---|
| `/etc/fstab` | Persistent filesystem mount definitions |
| `/mnt/` | Common manual mount point |
| `/media/` | Common removable-media mount point |
| `/dev/` | Device files |
| `/proc/` | Process and kernel information |
| `/sys/` | Device and subsystem information |
| `/run/` | Runtime files and state |
| `/tmp/` | Temporary files |
| `/var/tmp/` | Temporary files that may survive reboots |

Commands:

```bash
lsblk -f
df -h
findmnt
sudo blkid
cat /etc/fstab
```

Warning: incorrect `/etc/fstab` entries can cause boot or mount problems. Back up the file before editing and test changes with `sudo mount -a` before rebooting.

## 12. Security and Firewall

| Path | Purpose |
|---|---|
| `/etc/sudoers` | Sudo policy |
| `/etc/sudoers.d/` | Additional sudo rules |
| `/etc/ssh/` | SSH configuration |
| `/etc/pam.d/` | Authentication configuration |
| `/etc/security/` | Security-related configuration |
| `/etc/fail2ban/` | Fail2ban configuration, if installed |
| `/etc/ufw/` | UFW firewall configuration |
| `/etc/apparmor.d/` | AppArmor profiles on applicable systems |
| `/etc/audit/` | Audit configuration, if installed |

Commands:

```bash
sudo visudo -c
sudo sshd -t
sudo ufw status verbose
sudo aa-status
```

Some commands require installing the relevant package. Always understand a security change before applying it.

## 13. Web Servers and Databases

| Path | Purpose |
|---|---|
| `/etc/nginx/` | Nginx configuration |
| `/etc/nginx/sites-available/` | Available Nginx site configurations |
| `/etc/nginx/sites-enabled/` | Enabled Nginx sites on common Debian/Ubuntu setups |
| `/etc/apache2/` | Apache configuration on Debian/Ubuntu |
| `/etc/apache2/sites-available/` | Available Apache virtual hosts |
| `/etc/apache2/sites-enabled/` | Enabled Apache virtual hosts |
| `/var/www/html/` | Common default web root |
| `/etc/mysql/` | MySQL/MariaDB configuration |
| `/etc/postgresql/` | PostgreSQL configuration |
| `/var/lib/mysql/` | Common MySQL/MariaDB data directory |
| `/var/lib/postgresql/` | Common PostgreSQL data directory |

Commands:

```bash
sudo nginx -t
sudo apache2ctl configtest
systemctl status nginx
systemctl status apache2
```

Database and web-server paths can be customized. Verify the active configuration before moving or deleting files.

## 14. WSL and Windows Integration

| Path | Purpose |
|---|---|
| `/mnt/c/` | Usually the Windows C: drive |
| `/mnt/d/` | Usually the Windows D: drive |
| `/home/YOUR_USERNAME/` | Your Linux home directory |
| `/etc/wsl.conf` | Optional WSL-specific Linux configuration |
| `/etc/hosts` | May be generated or managed by WSL |
| `/etc/resolv.conf` | May be generated or managed by WSL |

Example:

```bash
cd /mnt/c/Projects
```

This often corresponds to `C:\Projects` in Windows.

Check whether systemd is running in WSL:

```bash
ps -p 1 -o comm=
```

If the output is `systemd`, systemd commands are generally available. Otherwise, service-management behavior may differ.

## 15. Find, Inspect, and Identify Configuration Files

```bash
# Check whether a path exists and show its details
ls -ld /etc/systemd/system/

# List files, including hidden files
ls -lah /etc/

# Identify the type of a file
file /etc/hostname

# Read a text configuration file
less /etc/hostname

# Search for a filename beneath /etc
find /etc -name 'sshd_config' 2>/dev/null

# Locate a command
command -v ssh

# Show where a command and related files may be located
whereis ssh

# Show symbolic links
ls -l /etc/resolv.conf
```

## 16. Backup Before Editing Configuration

Before modifying a configuration file, make a backup.

Example for `/etc/hosts`:

```bash
sudo cp -a /etc/hosts /etc/hosts.backup
sudo nano /etc/hosts
```

Restore the backup if needed:

```bash
sudo cp -a /etc/hosts.backup /etc/hosts
```

For files that control boot, networking, authentication, SSH, or firewall behavior, ensure you have a recovery method before making changes.

## 17. Ten Paths to Memorize First

1. `/etc/` — System configuration
2. `/var/log/` — Logs and troubleshooting
3. `/etc/ssh/sshd_config` — SSH server settings
4. `/etc/fstab` — Filesystem mounts
5. `/etc/hosts` — Local hostname resolution
6. `/etc/netplan/` — Ubuntu networking on Netplan-based systems
7. `/etc/systemd/system/` — Custom systemd services and timers
8. `/etc/crontab` — System cron schedule
9. `~/.bashrc` — Bash configuration
10. `/etc/sudoers.d/` — Additional sudo policy

## 18. General Linux Configuration Workflow

Use this workflow when configuring Linux:

1. **Inspect:** Read the current configuration.
2. **Back up:** Save a copy of the file before editing.
3. **Edit:** Change only the setting you need.
4. **Validate:** Run the relevant syntax or configuration checker.
5. **Apply:** Reload or restart the relevant service if necessary.
6. **Verify:** Check the service status and logs.
7. **Recover:** Restore the backup if the change causes problems.

Example for a systemd timer:

```bash
sudo cp -a /etc/systemd/system/mytask.timer \
  /etc/systemd/system/mytask.timer.backup

sudo nano /etc/systemd/system/mytask.timer

sudo systemd-analyze verify /etc/systemd/system/mytask.timer

sudo systemctl daemon-reload
sudo systemctl restart mytask.timer
systemctl status mytask.timer
systemctl list-timers --all
sudo journalctl -u mytask.service -n 30 --no-pager
```

**Remember:** Not every Linux distribution uses exactly the same file layout. Some paths are optional, some files are generated automatically, and some directories are symbolic links. Always inspect your own system before changing configuration.
