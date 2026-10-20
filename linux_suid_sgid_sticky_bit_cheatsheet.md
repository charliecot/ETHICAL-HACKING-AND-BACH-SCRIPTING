# Linux Special Permissions: SUID, SGID, and Sticky Bit

A practical Ubuntu/Linux cheatsheet explaining **Set User ID (SUID)**, **Set Group ID (SGID)**, and the **sticky bit**, with `chmod`, `find`, inspection commands, security notes, and safe exercises.

> **Security warning:** Special permission bits can change how programs run or how directories are shared. Do not set SUID/SGID on arbitrary executables, and do not remove special bits from system files unless you know why they are present. Practise only in a directory you own.

## 1. Ordinary Linux permissions recap

Normal permissions have three classes:

- **Owner** — the file's owner.
- **Group** — the file's group.
- **Others** — everyone else.

Each class can have:

- `r` — read
- `w` — write
- `x` — execute for files; search/traverse for directories

Example:

```bash
ls -l example.sh
chmod 755 example.sh
```

A permission display such as `-rwxr-xr-x` means the owner can read/write/execute, while group and others can read/execute.

Linux also has special permission bits: **SUID**, **SGID**, and **sticky bit**.

## 2. SUID — Set User ID

### What it means

When SUID is set on an executable file, the program generally runs with the effective user ID of the file's owner, rather than simply the ID of the user who launched it. This is commonly used by carefully designed system programs that need limited elevated privileges.

SUID is primarily meaningful on executable files. Behavior can vary by filesystem, mount options, kernel, and execution context. Linux ignores SUID on scripts in typical configurations, and `nosuid` mounts prevent it from taking effect.

### Identify SUID in `ls -l`

Example:

```text
-rwsr-xr-x 1 root root 12345 Oct 20  example
```

Look at the owner's execute position:

- `s` lowercase: SUID is set and owner execute is set.
- `S` uppercase: SUID is set but owner execute is not set.

### Set SUID

Symbolic form:

```bash
chmod u+s program
```

Numeric form:

```bash
chmod 4755 program
```

The leading `4` sets SUID; `755` specifies ordinary permissions.

### Remove SUID

```bash
chmod u-s program
```

Or explicitly set ordinary permissions without SUID:

```bash
chmod 0755 program
```

Use the numeric form only if `0755` is actually the intended ordinary permission mode. It overwrites the other permission bits as well.

### Inspect SUID files

Search a selected filesystem tree for SUID files:

```bash
find /usr/bin -type f -perm -4000 -print 2>/dev/null
```

Search your own home directory:

```bash
find "$HOME" -type f -perm -4000 -print 2>/dev/null
```

`-perm -4000` means the file has the SUID bit set, even if other permission bits are also set.

Do not assume every SUID file is malicious. Some legitimate system utilities require special permissions. Investigate unexpected files by checking ownership, package provenance, path, and permissions.

## 3. SGID — Set Group ID

SGID has two common behaviors depending on the object.

### On an executable file

The program generally runs with the effective group ID of the file's group.

In `ls -l`, SGID appears in the group's execute position:

```text
-rwxr-sr-x 1 root developers 12345 Oct 20  example
```

- `s` lowercase: SGID and group execute are set.
- `S` uppercase: SGID is set but group execute is not set.

Set it:

```bash
chmod g+s program
```

Numeric form:

```bash
chmod 2755 program
```

Remove it:

```bash
chmod g-s program
```

### On a directory

A directory with SGID set normally causes new files and subdirectories created inside it to inherit the directory's group, rather than the creator's usual primary group. New subdirectories commonly inherit SGID too, helping teams share a group-owned project directory.

Example:

```bash
sudo groupadd projectteam
sudo mkdir -p /srv/project
sudo chown root:projectteam /srv/project
sudo chmod 2770 /srv/project
```

Here:

- `2` sets SGID.
- `770` gives owner and group full access, and gives others no access.
- `root:projectteam` sets the owner and group.

To allow members of `projectteam` to collaborate, add the relevant existing accounts to the group, then have them start a new login session so group membership refreshes:

```bash
sudo usermod -aG projectteam username
id username
```

Replace `username` with the intended account. Group changes should be planned carefully, especially on shared systems.

Remove SGID from a directory:

```bash
chmod g-s /srv/project
```

### Find SGID files and directories

```bash
find /usr -type f -perm -2000 -print 2>/dev/null
find /srv -type d -perm -2000 -print 2>/dev/null
```

`-perm -2000` matches objects that have the SGID bit set.

## 4. Sticky bit

### What it means

The sticky bit is most often used on shared directories. In a sticky directory, users generally may remove or rename only files they own, files owned by the directory owner, or files for which they have the relevant privilege. This helps protect users' files in shared writable locations.

The classic example is `/tmp`.

### Recognize it

Example:

```text
drwxrwxrwt 10 root root 4096 Oct 20 /tmp
```

Look at the **others' execute position**:

- `t` lowercase: sticky bit and others execute/search are set.
- `T` uppercase: sticky bit is set but others execute/search is not set.

### Set the sticky bit

Symbolic form:

```bash
chmod +t shared_directory
```

Numeric form:

```bash
chmod 1777 shared_directory
```

The leading `1` sets the sticky bit; `777` grants read/write/execute to owner, group, and others. Only use `1777` when a world-writable directory is actually intended.

### Remove the sticky bit

```bash
chmod -t shared_directory
```

Inspect `/tmp`:

```bash
ls -ld /tmp
```

Do not remove the sticky bit from `/tmp` without a strong reason.

## 5. Special bits: numeric and symbolic reference

The special permission digit is placed before the three ordinary permission digits.

| Special bit | Numeric value | Symbolic command | Typical use |
|---|---:|---|---|
| SUID | `4` | `chmod u+s FILE` | Executable runs with file owner's effective UID |
| SGID | `2` | `chmod g+s FILE` | Executable uses file group's effective GID; directory group inheritance |
| Sticky bit | `1` | `chmod +t DIR` | Restricts deletion/renaming in shared writable directories |

Examples:

```bash
chmod 4755 program   # SUID + ordinary 755 permissions
chmod 2755 directory # SGID + ordinary 755 permissions
chmod 1777 directory # Sticky + ordinary 777 permissions
chmod 6755 program   # SUID + SGID + ordinary 755 permissions
```

Do not apply examples blindly to system programs or shared directories.

## 6. How to read special bits in `ls -l`

| Display example | Meaning |
|---|---|
| `-rwsr-xr-x` | SUID set; owner execute set |
| `-rwSr-xr-x` | SUID set; owner execute not set |
| `-rwxr-sr-x` | SGID set; group execute set |
| `-rwxr-Sr-x` | SGID set; group execute not set |
| `drwxrwxrwt` | Sticky set; others can search/write (subject to directory rules) |
| `drwxrwxrwT` | Sticky set; others execute/search not set |

For a file or directory:

```bash
ls -l path
stat path
stat -c '%A %a %U:%G %n' path
```

`%A` prints a human-readable permission string, `%a` prints the octal mode, `%U` the owner name, `%G` the group name, and `%n` the path.

## 7. `find` options for auditing special permissions

### Find SUID files

```bash
find / -type f -perm -4000 -print 2>/dev/null
```

### Find SGID files

```bash
find / -type f -perm -2000 -print 2>/dev/null
```

### Find sticky directories

```bash
find / -type d -perm -1000 -print 2>/dev/null
```

### Search within a smaller, safer scope

```bash
find "$HOME" -type f -perm /6000 -print
```

`-perm /6000` matches files with either SUID or SGID set (on GNU `find`). Option syntax varies across `find` implementations; see `man find`.

### Understand `-perm` forms (GNU find)

- `-perm 4755` — exact permission mode match.
- `-perm -4000` — all bits in `4000` must be set; other bits may also be set.
- `-perm /6000` — any bit in `6000` may be set.

Run a broad filesystem scan only when you understand the output volume and permission boundaries. `2>/dev/null` hides permission-denied messages; it does not mean every location was successfully inspected.

## 8. Safe practice lab

This lab creates files in your home directory. It does not require `sudo`.

```bash
# Create a practice directory and file
mkdir -p "$HOME/special-bits-lab"
cd "$HOME/special-bits-lab"
printf '#!/usr/bin/env bash\necho "Hello from the practice script"\n' > demo.sh
chmod 755 demo.sh

# Inspect ordinary permissions
ls -l demo.sh
stat -c '%A %a %n' demo.sh

# Set SUID for learning, then inspect it.
# Note: Linux generally ignores SUID on scripts; this is only a permission-bit exercise.
chmod u+s demo.sh
ls -l demo.sh
stat -c '%A %a %n' demo.sh

# Remove SUID
chmod u-s demo.sh

# Create a directory and set SGID
mkdir team
chmod 2770 team
ls -ld team
stat -c '%A %a %n' team

# Create a directory and set sticky bit
mkdir shared
chmod 1777 shared
ls -ld shared
stat -c '%A %a %n' shared

# Inspect special bits in this lab
find "$HOME/special-bits-lab" -maxdepth 2 \( -perm -4000 -o -perm -2000 -o -perm -1000 \) -print

# Clean up the lab when finished
cd "$HOME"
rm -r "$HOME/special-bits-lab"
```

This lab demonstrates how permission bits appear. It does not demonstrate the privilege behavior of a real SUID executable, and SUID scripts generally do not gain the file owner's effective identity on Linux.

## 9. Security best practices

- Do not set SUID on a program just to make it easier to run as root.
- Never test privilege escalation by changing permissions on system executables.
- Investigate unexpected SUID/SGID files using ownership, package records, file paths, and system logs.
- Keep shared writable directories sticky when that is the intended security model.
- Use SGID directories for group collaboration only after planning group membership and permissions.
- Review special bits after restoring backups or copying files between filesystems.
- Understand that filesystem mount options such as `nosuid` can prevent SUID/SGID behavior.
- Prefer least privilege: grant only the access needed.

## 10. Quick copy reference

```bash
# Inspect permissions
ls -l FILE
ls -ld DIRECTORY
stat -c '%A %a %U:%G %n' PATH

# SUID
chmod u+s FILE
chmod u-s FILE
find /usr/bin -type f -perm -4000 -print 2>/dev/null

# SGID
chmod g+s FILE_OR_DIRECTORY
chmod g-s FILE_OR_DIRECTORY
find /usr -type f -perm -2000 -print 2>/dev/null

# Sticky bit
chmod +t DIRECTORY
chmod -t DIRECTORY
find /tmp -type d -perm -1000 -print 2>/dev/null

# Numeric modes
chmod 4755 FILE       # SUID + 755
chmod 2755 DIRECTORY  # SGID + 755
chmod 1777 DIRECTORY  # Sticky + 777

# GNU find: any SUID or SGID bit
find "$HOME" -type f -perm /6000 -print
```

## 11. Further reading

```bash
man chmod
man chown
man find
man stat
man 7 inode
```

Also consult the documentation for your Linux distribution and filesystem. Special-bit behavior depends on file type, mount options, kernel rules, and security policy.
