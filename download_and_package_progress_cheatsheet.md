# Download and Package Progress Cheatsheet
## `wget`, `curl`, `apt`, and `apt-get` — copyable reference

This guide covers progress displays, downloading, logging, resuming, package installation, troubleshooting, and common options on Linux, Ubuntu, and WSL.

> Replace example URLs and filenames with real values. Only access resources you are authorized to use.

---

## 1. Quick overview

| Tool | Purpose | Progress option |
|---|---|---|
| `wget` | Download files from URLs | `--progress=bar:force` |
| `curl` | Transfer data to/from URLs and APIs | `--progress-bar` or `-#` |
| `apt` | Manage Ubuntu/Debian packages | `--show-progress` where supported |
| `apt-get` | Package management, often used in scripts | `--show-progress` where supported |

These options are not interchangeable: Wget and Curl download web resources; APT and APT-GET manage software packages.

## 2. Wget progress options

```bash
# Default progress display
wget https://example.com/image.png

# Force a progress bar
wget --progress=bar:force https://example.com/image.png

# Force a bar with reduced scrolling
wget --progress=bar:force:noscroll https://example.com/image.png

# Dot-style progress
wget --progress=dot https://example.com/image.png

# Dot-style progress with a larger reporting interval
wget --progress=dot:mega https://example.com/large-file.zip

# Save with a custom filename and show progress
wget --progress=bar:force -O image.png https://example.com/image.png
```

| Option | Meaning |
|---|---|
| `--progress=bar` | Use a progress bar |
| `--progress=bar:force` | Force the bar even when output is not detected as a terminal |
| `--progress=bar:force:noscroll` | Force a bar and reduce scrolling |
| `--progress=dot` | Use dot-style progress |
| `--progress=dot:mega` | Dot-style progress with a larger reporting interval |
| `--progress=dot:giga` | Dot-style progress with a gigabyte reporting interval |

Progress details can vary by Wget version. Check `wget --help` or the installed manual.

### Wget downloads and filenames

```bash
# Download using the URL/server-provided filename
wget https://example.com/file.zip

# Save using a custom filename
wget -O archive.zip https://example.com/file.zip

# Save in a directory
wget -P ~/Downloads https://example.com/file.zip

# Download multiple URLs
wget https://example.com/file1.zip https://example.com/file2.zip

# Read URLs from a text file
wget -i urls.txt
```

Example `urls.txt`:

```text
https://example.com/file1.zip
https://example.com/file2.zip
https://example.com/image.png
```

### Wget resume, logs, quiet mode

```bash
# Resume a partial download when supported by the server
wget -c https://example.com/large-file.zip

# Write messages to a log file
wget -o wget.log https://example.com/file.zip

# Append messages to an existing log
wget -a wget.log https://example.com/file.zip

# Quiet mode (hides normal output and progress)
wget -q https://example.com/file.zip

# Read the log and follow it as it grows
cat wget.log
tail -f wget.log
```

**Remember:** Wget `-o` (lowercase) sets the log filename; `-O` (uppercase) sets the downloaded document filename.

### Wget speed limits, retries, timeouts

```bash
# Limit speed to about 500 KB/s
wget --limit-rate=500k https://example.com/large-file.zip

# Limit speed to about 2 MB/s
wget --limit-rate=2m https://example.com/large-file.zip

# Retry up to five times
wget -t 5 https://example.com/file.zip

# Set a network timeout of 20 seconds
wget -T 20 https://example.com/file.zip

# Wait two seconds between retrievals
wget -w 2 https://example.com/file.zip

# Retry indefinitely (use cautiously)
wget -t 0 https://example.com/file.zip
```

### Wget inspect URLs and help

```bash
# Check a URL without downloading its body
wget --spider https://example.com/

# Print response headers while checking
wget -S --spider https://example.com/

# Show headers during a download
wget -S https://example.com/file.zip

# Help and version
wget --help
wget --version
```

A successful `--spider` check does not guarantee that a later download will work. Some servers handle HEAD requests differently from GET requests.

---

## 3. Curl progress options

```bash
# Default progress meter when output is connected to a terminal
curl -O https://example.com/image.png

# Show a progress bar
curl -# -O https://example.com/image.png

# Long form of -#
curl --progress-bar -O https://example.com/image.png

# Save under a custom name and show progress
curl --progress-bar -o image.png https://example.com/image.png

# Hide only the progress meter
curl --no-progress-meter -O https://example.com/image.png

# Silent mode: hide progress and normal messages
curl -s -O https://example.com/image.png

# Silent mode but continue showing errors
curl -sS -O https://example.com/image.png
```

| Option | Meaning |
|---|---|
| `-#` | Display a progress bar |
| `--progress-bar` | Long form of `-#` |
| `--no-progress-meter` | Hide the progress meter |
| `-s` / `--silent` | Suppress progress and normal messages |
| `-S` / `--show-error` | Show errors even in silent mode |

Curl normally shows its progress meter when connected to a terminal. `--progress-bar` changes its appearance; `--no-progress-meter` hides it.

### Curl download and resume options

```bash
# Save using the filename in the URL
curl -O https://example.com/image.png

# Save using a chosen filename
curl -o image.png https://example.com/image.png

# Follow redirects and show a progress bar
curl -L --progress-bar -o image.png https://example.com/image.png

# Resume an interrupted download when supported
curl -C - -O https://example.com/large-file.zip
```

- `-O` (uppercase O) uses the remote filename.
- `-o FILE` (lowercase o) uses the filename you provide.
- `-L` follows HTTP redirects.
- `-C -` tries to resume from the existing local file size.

### Curl headers and troubleshooting

```bash
# Include response headers with the response body
curl -i https://example.com/

# Request response headers only (HEAD request)
curl -I https://example.com/

# Show detailed connection diagnostics
curl -v https://example.com/

# Follow redirects
curl -L https://example.com/

# Print only the HTTP status code
curl -sS -o /dev/null -w '%{http_code}\n' https://example.com/

# Set a maximum operation time of 30 seconds
curl --max-time 30 -L -O https://example.com/large-file.zip

# Help and version
curl --help
curl --version
```

`-i` includes headers with the body. `-I` sends a HEAD request, which some servers handle differently from GET.

---

## 4. APT progress and package options

`apt` is the interactive package manager commonly used on Ubuntu and Debian.

```bash
# Refresh package lists
sudo apt update

# Install a package
sudo apt install wget

# Request progress display during supported operations
sudo apt install --show-progress wget

# Upgrade installed packages
sudo apt upgrade

# Upgrade with progress display where supported
sudo apt upgrade --show-progress

# Remove a package but normally retain its system configuration
sudo apt remove wget

# Remove a package and its system configuration
sudo apt purge wget

# Remove unused dependencies (review the proposed list)
sudo apt autoremove

# Clean downloaded package archives
sudo apt clean
```

`apt update` fetches package indexes, so its output mainly reports repository activity. `--show-progress` is most relevant to supported package operations that report installation progress. Output depends on APT version and terminal.

### Useful APT options

| Option | Meaning |
|---|---|
| `--show-progress` | Show progress during supported operations |
| `-y` | Automatically answer yes; use cautiously |
| `-s` | Simulate an operation without changing the system |
| `-qq` | Very quiet output |
| `--help` | Display help |

```bash
# Preview an installation without making changes
apt -s install wget

# Download a package file without installing it
apt download wget

# Display help and version
apt --help
apt --version
```

Use `-y` carefully because it skips confirmation prompts.

---

## 5. APT-GET progress and package options

`apt-get` is a lower-level package-management command often used in scripts.

```bash
# Refresh package lists
sudo apt-get update

# Install with progress display where supported
sudo apt-get --show-progress install wget

# Upgrade packages with progress display where supported
sudo apt-get --show-progress upgrade

# Simulate installation
sudo apt-get -s install wget

# Download packages without installing them
sudo apt-get --download-only install wget

# Reinstall a package
sudo apt-get install --reinstall wget

# Attempt to repair broken dependencies
sudo apt-get --fix-broken install

# Help and version
apt-get --help
apt-get --version
```

| Option | Meaning |
|---|---|
| `--show-progress` | Show progress during supported operations |
| `-s` / `--simulate` | Simulate an operation |
| `-y` / `--yes` | Automatically answer yes |
| `-qq` | Very quiet output |
| `-d` / `--download-only` | Download packages without installing |
| `--no-download` | Do not download packages |
| `--reinstall` | Reinstall a package |
| `--fix-broken` | Attempt to repair broken dependencies |
| `--help` | Display help |

---

## 6. DPKG fancy progress

APT uses `dpkg` underneath. You can set the fancy progress display for a single command:

```bash
# Enable fancy dpkg progress for this installation
sudo apt-get -o Dpkg::Progress-Fancy=1 install wget

# Disable fancy dpkg progress for this installation
sudo apt-get -o Dpkg::Progress-Fancy=0 install wget
```

The display depends on the terminal and installed APT/dpkg versions.

---

## 7. Install Wget and Curl on Ubuntu/WSL

```bash
# Refresh package lists
sudo apt update

# Install both tools
sudo apt install wget curl

# Check installed versions
wget --version
curl --version
```

---

## 8. Quick comparison

| Goal | Wget | Curl | APT / APT-GET |
|---|---|---|---|
| Show progress bar | `--progress=bar:force` | `--progress-bar` or `-#` | `--show-progress` where supported |
| Hide progress | `-q` | `--no-progress-meter` or `-s` | `-qq` for very quiet output |
| Save as a custom filename | `-O FILE` | `-o FILE` | Not a general download-file option |
| Resume a download | `-c` | `-C -` | Different package-download behavior |
| Show help | `wget --help` | `curl --help` | `apt --help` / `apt-get --help` |
| Show version | `wget --version` | `curl --version` | `apt --version` / `apt-get --version` |

Filename option reminder:
- Wget: `-O` sets the downloaded filename; `-o` sets the log filename.
- Curl: `-O` uses the remote filename; `-o` sets a custom filename.

---

## 9. Copy-and-practice block

Replace `https://example.com/image.png` with a real URL.

```bash
# WGET: force a progress bar
wget --progress=bar:force https://example.com/image.png

# WGET: custom filename and progress bar
wget --progress=bar:force -O image.png https://example.com/image.png

# WGET: limit speed to 500 KB/s
wget --progress=bar:force --limit-rate=500k https://example.com/large-file.zip

# CURL: progress bar, remote filename
curl --progress-bar -O https://example.com/image.png

# CURL: progress bar, custom filename, follow redirects
curl -L --progress-bar -o image.png https://example.com/image.png

# CURL: hide progress but show errors
curl -sS -O https://example.com/image.png

# APT: install with progress display where supported
sudo apt install --show-progress wget

# APT-GET: install with progress display where supported
sudo apt-get --show-progress install wget

# APT: simulate an installation
apt -s install wget

# Show help for all tools
wget --help
curl --help
apt --help
apt-get --help
```

## 10. Practice exercises

1. Download an image with Wget and force the progress bar.
2. Save it as `my-image.png`.
3. Download a file with Curl using `--progress-bar`.
4. Hide Curl's progress meter while keeping error messages.
5. Resume a partial download with Wget.
6. Resume a partial download with Curl.
7. Simulate installing `wget` with `apt -s`.
8. Install `curl` and observe the package output.
9. Explain the difference between Wget `-o` and `-O`.
10. Explain the difference between Curl `-o` and `-O`.

## Official references

- GNU Wget manual: https://www.gnu.org/software/wget/manual/wget.html
- GNU Wget website: https://www.gnu.org/software/wget/
- Curl manual: https://curl.se/docs/manpage.html
- APT manual: https://manpages.ubuntu.com/manpages/latest/en/man8/apt.8.html
- APT-GET manual: https://manpages.ubuntu.com/manpages/latest/en/man8/apt-get.8.html
