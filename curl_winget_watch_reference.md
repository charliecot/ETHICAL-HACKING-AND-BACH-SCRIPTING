# Command-Line Reference Guide: `curl`, `winget`, and `watch`

A practical, copyable reference for Windows PowerShell, Ubuntu, and WSL. Examples use placeholder URLs such as `https://example.com`; replace them with trusted, real URLs and endpoints.

> **Safety:** Inspect downloads before running them. Do not disable TLS verification just to silence errors. Do not paste access tokens or passwords into shared scripts. Use administrator privileges only when necessary.

---

# 1. `curl` — Network transfers, downloads, uploads, and HTTP testing

## 1.1 What is curl?

`curl` transfers data to and from servers using protocols supported by your build, including HTTP and HTTPS. It is useful for downloading files, uploading files, testing REST APIs, inspecting headers, following redirects, and troubleshooting connections.

Check the installed version:

```bash
curl --version
```

On Windows PowerShell, explicitly use `curl.exe` so that you invoke curl rather than any shell alias:

```powershell
curl.exe --version
```

General syntax:

```bash
curl [OPTIONS] URL
```

## 1.2 Common options

| Option | Meaning |
|---|---|
| `-o FILE` | Save the response to a filename you choose. |
| `-O` | Save using the filename from the URL path. |
| `-L` | Follow HTTP redirects. |
| `-I` | Request headers only, normally using HTTP HEAD. |
| `-i` | Include response headers in output. |
| `-v` | Verbose connection and protocol diagnostics. |
| `-s` | Silent mode; suppress progress and normal error messages. |
| `-S` | Show errors even when `-s` is used. |
| `-f` | Fail on HTTP response codes 400 and above. |
| `--fail-with-body` | Return an error on HTTP 400+ but retain the response body. |
| `-C -` | Attempt to resume a download from the existing file size. |
| `-#` | Show a progress bar. |
| `--connect-timeout SEC` | Limit time spent establishing a connection. |
| `--max-time SEC` | Limit total time for the transfer. |
| `-H "Name: value"` | Add or customize an HTTP header. |
| `-A TEXT` | Set the User-Agent header. |
| `-d DATA` | Send request data, commonly with POST. |
| `-F "name=value"` | Send multipart form data. |
| `-T FILE` | Upload a file; for HTTP(S), commonly uses PUT. |
| `-X METHOD` | Explicitly set the HTTP method. |
| `-u USER:PASS` | Supply credentials for supported authentication schemes. |
| `-x URL` | Use a proxy. |
| `-k` | Skip TLS certificate verification. **Avoid for normal use.** |
| `-w FORMAT` | Print selected transfer information, such as HTTP status. |
| `--help all` | Display the full option help. |
| `--manual` | Display the full manual. |

Check the exact behavior supported by your installed version using `curl --help all` and `curl --manual`.

## 1.3 Download a file

### Save with a filename you choose (`-o`)

```bash
# Save the response as report.pdf in the current directory.
curl -o report.pdf "https://example.com/files/report.pdf"
```

- `-o` means output to the named file.
- `report.pdf` is your local filename.
- The URL identifies the remote resource.

### Keep the filename from the URL (`-O`)

```bash
# Save using the final path component of the URL.
curl -O "https://example.com/files/report.pdf"
```

The resulting local filename will normally be `report.pdf`.

### Follow redirects (`-L`)

Some download URLs redirect to a different URL. Use `-L` to follow those redirects.

```bash
curl -L -O "https://example.com/download"
```

A practical download command:

```bash
# Follow redirects, fail on HTTP 400+, show errors and progress,
# and save to the filename you choose.
curl -L --fail --show-error --progress-bar \
     -o report.pdf \
     "https://example.com/files/report.pdf"
```

The backslash (`\`) continues a command on the next line in Bash. For PowerShell, use a backtick (`` ` ``) for line continuation, or write the command on one line.

### Download to a specific directory

Linux / Ubuntu / WSL:

```bash
# The Downloads directory must already exist.
curl -L -o "$HOME/Downloads/report.pdf" \
     "https://example.com/files/report.pdf"
```

Windows PowerShell:

```powershell
# Save to your Windows Downloads folder.
curl.exe -L -o "$HOME\Downloads\report.pdf" "https://example.com/files/report.pdf"
```

### Resume an interrupted download

```bash
# Continue from the existing file size if the server supports resuming.
curl -L -C - -o archive.zip "https://example.com/files/archive.zip"
```

`-C -` asks curl to continue from the existing local file size. The server must support the relevant range request. If the server does not support resuming, the command may fail or you may need to restart the download.

### Limit connection and total time

```bash
curl -L \
     --connect-timeout 15 \
     --max-time 300 \
     -o archive.zip \
     "https://example.com/files/archive.zip"
```

- `--connect-timeout 15`: allow up to 15 seconds to establish the connection.
- `--max-time 300`: limit the complete transfer to 300 seconds.

### Download quietly, but still show errors

```bash
curl -sS -L -o archive.zip "https://example.com/files/archive.zip"
```

### Check the HTTP status code

Linux / Ubuntu / WSL:

```bash
# Discard the response body and print only the HTTP status code.
curl -sS -o /dev/null -w "%{http_code}\n" "https://example.com"
```

Windows PowerShell:

```powershell
# NUL is the Windows equivalent of a discard destination.
curl.exe -sS -o NUL -w "%{http_code}`n" "https://example.com"
```

Common HTTP codes:

| Code | Meaning |
|---|---|
| `200` | Request succeeded. |
| `301` / `302` | Redirect. |
| `400` | Bad request. |
| `401` | Authentication required or failed. |
| `403` | Access forbidden. |
| `404` | Resource not found. |
| `429` | Too many requests. |
| `500` | Server error. |
| `503` | Service unavailable. |

### Inspect response headers

```bash
# Request headers only (HEAD, if supported by the server).
curl -I "https://example.com"

# Include headers and the response body.
curl -i "https://example.com"

# Print detailed connection diagnostics.
curl -v "https://example.com"
```

Verbose output can include sensitive headers and connection information. Redact it before sharing.

## 1.4 Upload a file

The upload method depends on what the server expects.

### HTTP PUT with `-T`

```bash
# Upload report.pdf using HTTP PUT to an endpoint that accepts PUT.
curl -T report.pdf "https://example.com/uploads/report.pdf"
```

`-T` is shorthand for `--upload-file`. The server must allow this upload method and destination. Do not assume an arbitrary website accepts PUT uploads.

### Multipart form upload with `-F`

Many web forms and Django REST Framework upload endpoints expect multipart form data:

```bash
# Send a local file in the form field named "cv".
curl -X POST \
     -F "cv=@resume.pdf" \
     "http://127.0.0.1:8000/api/upload-cv/"
```

- `-X POST`: explicitly select POST.
- `-F`: create a multipart form field.
- `cv`: field name expected by the server.
- `@resume.pdf`: read the local file and attach it.

For a Django serializer with `cv = serializers.FileField()`, the form field name must match `cv`.

Send a file and another field:

```bash
curl -F "cv=@resume.pdf" \
     -F "description=My CV" \
     "http://127.0.0.1:8000/api/upload-cv/"
```

Do not manually set the multipart `Content-Type` header for this request. Curl generates the correct boundary.

### Upload a file with JWT bearer authentication

```bash
# Replace YOUR_ACCESS_TOKEN with a valid access token.
curl -X POST \
     -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
     -F "cv=@resume.pdf" \
     "http://127.0.0.1:8000/api/upload-cv/"
```

Do not commit real tokens to Git, place them in public scripts, or share terminal logs containing them.

## 1.5 Send JSON to a REST API

```bash
curl -X POST \
     -H "Content-Type: application/json" \
     -d '{"name":"Keyboard","price":25000}' \
     "http://127.0.0.1:8000/api/products/"
```

- `-H` sets the JSON content type.
- `-d` sends the request body.
- The API must accept the specified method and JSON fields.

For authenticated requests:

```bash
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
     "http://127.0.0.1:8000/api/orders/"
```

## 1.6 Download safety and verification

- Prefer HTTPS.
- Do not use `-k` to bypass certificate errors in normal use; it disables certificate verification.
- Check the HTTP status and confirm the downloaded file is the expected size and type.
- Compare a checksum with a value published by a trusted software provider.
- Avoid `curl URL | bash` for unknown scripts because it immediately executes downloaded content.

Linux / Ubuntu / WSL:

```bash
sha256sum archive.zip
```

Windows PowerShell:

```powershell
Get-FileHash .\archive.zip -Algorithm SHA256
```

A calculated hash is only useful for integrity verification when compared against a trusted expected hash.

---

# 2. `winget` — Windows Package Manager

## 2.1 What is winget?

`winget` is Microsoft's Windows Package Manager CLI. It can search for, install, upgrade, uninstall, and download application packages on Windows. It is commonly provided by the App Installer component.

Open PowerShell and check:

```powershell
winget --version
winget --help
winget --info
```

General syntax:

```powershell
winget COMMAND [OPTIONS]
```

## 2.2 Search and inspect packages

```powershell
# Search for Python packages.
winget search Python

# Search for Visual Studio Code.
winget search "Visual Studio Code"

# Inspect a package by exact ID.
winget show --id Microsoft.VisualStudioCode --exact

# List installed applications recognized by winget.
winget list
```

Package IDs are generally more reliable than names when targeting a specific application. Inspect the package name, publisher, version, and source before installing.

## 2.3 Install applications

```powershell
# Install Visual Studio Code.
winget install --id Microsoft.VisualStudioCode --exact

# Install Git.
winget install --id Git.Git --exact

# Install 7-Zip.
winget install --id 7zip.7zip --exact
```

These are example package IDs. Availability, installer options, and behavior can vary.

### Common installation options

| Option | Purpose |
|---|---|
| `--id ID` | Select a package by identifier. |
| `--exact` | Require an exact match. |
| `--source NAME` | Choose a configured package source. |
| `--scope user` | Request a per-user installation when supported. |
| `--scope machine` | Request a machine-wide installation when supported. |
| `--version VERSION` | Request a specific available version. |
| `--interactive` | Allow interactive installer behavior. |
| `--silent` | Request silent installation when supported. |
| `--accept-source-agreements` | Accept source agreements. |
| `--accept-package-agreements` | Accept package agreements. |
| `--disable-interactivity` | Disable interactive prompts where supported. |
| `--location PATH` | Request an install location when supported by the package. |
| `--log PATH` | Specify a log file where supported. |

Example:

```powershell
# Request a per-user Git installation.
winget install --id Git.Git --exact --scope user
```

Automated installation example:

```powershell
winget install --id Git.Git --exact `
    --silent `
    --accept-source-agreements `
    --accept-package-agreements
```

PowerShell uses a backtick for line continuation. Do not put spaces after the backtick. You can also place the command on one line.

Not all installers support every option. A package may ignore or reject a requested scope, version, or installation location.

## 2.4 Upgrade packages

```powershell
# List packages for which upgrades are available.
winget upgrade

# Upgrade a specific package.
winget upgrade --id Git.Git --exact

# Upgrade Visual Studio Code.
winget upgrade --id Microsoft.VisualStudioCode --exact

# Upgrade all eligible packages.
winget upgrade --all
```

Common upgrade options include:

| Option | Purpose |
|---|---|
| `--all` | Upgrade all eligible packages. |
| `--id ID` | Target a specific package. |
| `--exact` | Match the package ID exactly. |
| `--include-unknown` | Include packages whose installed version is unknown. |
| `--include-pinned` | Include pinned packages where supported. |
| `--silent` | Request silent upgrade behavior. |
| `--uninstall-previous` | Request removal of the previous version where supported. |

Review the proposed upgrades before running `winget upgrade --all`. Upgrades can change application behavior or cause compatibility issues.

## 2.5 Uninstall and repair

```powershell
# Uninstall Git.
winget uninstall --id Git.Git --exact

# Inspect uninstall options.
winget uninstall --help

# Inspect repair options.
winget repair --help
```

If repair is supported by the installed winget version and package:

```powershell
winget repair --id Microsoft.VisualStudioCode --exact
```

Not every application supports repair.

## 2.6 Download an installer without installing it

First check your installed winget version's options:

```powershell
winget download --help
```

Example for versions that support the shown arguments:

```powershell
winget download --id Git.Git --exact --download-directory "$HOME\Downloads"
```

This requests a download to your Downloads directory; exact syntax and package behavior may vary by winget version and package manifest. Check the local help before using the command in automation.

List downloaded files:

```powershell
Get-ChildItem "$HOME\Downloads" -Recurse -File
```

Downloading an installer does not itself install the application. Verify the source and digital signature or checksum where available before running the installer.

## 2.7 Export and import package lists

Export a list of applications to JSON:

```powershell
winget export --output "$HOME\Downloads\my-apps.json"
```

Read the JSON:

```powershell
Get-Content "$HOME\Downloads\my-apps.json"
```

On another machine, review the file and then import it:

```powershell
winget import --import-file "$HOME\Downloads\my-apps.json"
```

This can help recreate a development environment, but it does not back up all program settings, databases, credentials, projects, or user data. Some applications may not be exportable or importable through winget.

## 2.8 Manage sources and settings

```powershell
# List configured sources.
winget source list

# Update source metadata.
winget source update

# Show source-management help.
winget source --help

# Inspect settings, features, pins, and configuration commands.
winget settings
winget features
winget pin --help
winget configure --help
```

A source is a repository or service used to retrieve package information. Do not reset or modify sources unless you understand the effect.

## 2.9 Troubleshoot winget

If `winget` is not recognized:

```powershell
Get-Command winget
winget --version
```

Check whether App Installer is installed and updated, then reopen PowerShell.

If multiple packages match, use an exact ID:

```powershell
winget search Git
winget show --id Git.Git --exact
```

If an installation fails, check the package ID, network or proxy access, installer requirements, existing installation state, and winget logs.

Avoid running every terminal as administrator. Elevation grants extra permissions and should be used only when required.

---

# 3. `watch` — Repeated command monitoring in Linux

## 3.1 What is watch?

`watch` repeatedly runs a command and refreshes its output in the terminal. It is commonly available in Ubuntu via the `procps` package.

Check it:

```bash
watch --version
```

Install it on Ubuntu/WSL if needed:

```bash
sudo apt update
sudo apt install procps
```

General syntax:

```bash
watch [OPTIONS] COMMAND
```

Example:

```bash
watch date
```

It refreshes the displayed date and time. Press `Ctrl+C` to stop.

## 3.2 Common options

| Option | Purpose |
|---|---|
| `-n SECONDS` | Set the refresh interval. |
| `-d` | Highlight differences between successive outputs. |
| `-d=permanent` | Keep differences highlighted against the initial output, where supported. |
| `-t` | Hide the header and timestamp. |
| `-p` | Attempt to keep execution on a regular interval. |
| `-g` | Exit when the displayed output changes. |
| `-b` | Beep if the command returns a non-zero status. |
| `-e` | Stop refreshing on command error until a key is pressed. |
| `-c` | Interpret ANSI color sequences. |
| `-x` | Execute the command directly rather than through a shell. |
| `-w` | Disable line wrapping where supported. |
| `-h` | Show help. |

Options vary between versions; check `watch --help` on your machine.

## 3.3 Set the refresh interval

```bash
# Default refresh interval is commonly 2 seconds.
watch date

# Refresh every 5 seconds.
watch -n 5 date

# Refresh every 10 seconds.
watch -n 10 uptime

# Use a half-second interval if supported.
watch -n 0.5 "date +%T"
```

Very short intervals can create unnecessary load, especially when the command is expensive.

## 3.4 Monitor processes and memory

```bash
# Display the first 15 lines of the process list.
watch -n 2 "ps aux | head -15"

# Monitor uptime and load averages.
watch -n 2 uptime

# Monitor memory.
watch -n 2 free -h

# Count processes.
watch -n 2 "ps -e --no-headers | wc -l"
```

For interactive process monitoring, use:

```bash
top
```

On common procps versions, press `P` to sort by CPU, `M` to sort by memory, and `q` to quit.

## 3.5 Monitor disk space and directories

```bash
# Show filesystem usage in readable units.
watch -n 5 df -h

# Monitor the root filesystem.
watch -n 5 "df -h /"

# Monitor files in the current directory.
watch -n 2 "ls -lah"

# Highlight differences in directory output.
watch -d -n 2 "ls -lah"

# Estimate the size of Downloads.
watch -n 10 "du -sh ~/Downloads"
```

Repeated `du` scans may be expensive for very large directories.

## 3.6 Monitor network interfaces, sockets, and routes

```bash
# Network interface counters.
watch -n 2 "ip -s link"

# TCP connections.
watch -n 2 "ss -tn"

# Listening TCP and UDP sockets.
watch -n 3 "ss -tuln"

# Routing table.
watch -n 5 "ip route"

# DNS resolution.
watch -n 5 "getent hosts example.com"
```

These commands help you observe network state. They do not by themselves provide full traffic capture or intrusion detection.

## 3.7 Highlight changes or exit on a change

```bash
# Highlight changes between refreshes.
watch -d -n 2 "free -h"

# Keep changes highlighted on supported versions.
watch -d=permanent -n 2 "ip -s link"

# Exit when the displayed output changes.
watch -g -n 2 "cat /tmp/status.txt"

# Hide the header.
watch -t -n 2 uptime
```

`watch -g` compares output at refresh times; it can miss a change that occurs and disappears between refreshes.

## 3.8 Run multiple commands

```bash
# Display time, uptime, and memory.
watch -n 3 "date; uptime; free -h"

# Show the top ten processes by CPU percentage.
watch -n 2 "ps aux | sort -nrk 3 | head -10"

# Separate disk and memory reports with a blank line.
watch -n 5 "df -h; echo; free -h"
```

Quotes keep pipes, semicolons, and other shell syntax together as one command for watch to run.

## 3.9 Monitor services and logs

```bash
# Check whether SSH is active.
watch -n 3 "systemctl is-active ssh"

# Check whether Nginx is active.
watch -n 3 "systemctl is-active nginx"
```

Service names differ by distribution; some systems use `sshd` instead of `ssh`.

For streaming logs, `journalctl -f` is often more suitable than watch:

```bash
# Show the latest 30 Nginx journal entries.
journalctl -u nginx -n 30

# Follow new Nginx log entries.
journalctl -u nginx -f
```

## 3.10 Cybersecurity lab monitoring

On systems you own or are authorized to administer:

```bash
# Observe listening ports.
watch -n 3 "ss -tuln"

# Observe established TCP connections.
watch -n 3 "ss -tn state established"

# Observe recent journal entries.
watch -n 5 "journalctl -n 10 --no-pager"

# Observe disk usage.
watch -n 10 "df -h"
```

These are basic monitoring examples, not a replacement for a dedicated monitoring or alerting system.

---

# 4. PowerShell alternatives to watch

Native Windows PowerShell does not normally include the Linux `watch` command. Use WSL or a PowerShell loop.

## 4.1 Monitor the top processes

```powershell
while ($true) {
    Clear-Host

    Get-Date
    Get-Process |
        Sort-Object CPU -Descending |
        Select-Object -First 10 Name, Id, CPU

    Start-Sleep -Seconds 2
}
```

Press `Ctrl+C` to stop the loop.

## 4.2 Monitor filesystem drives

```powershell
while ($true) {
    Clear-Host

    Get-PSDrive -PSProvider FileSystem

    Start-Sleep -Seconds 5
}
```

## 4.3 Monitor listening TCP ports

```powershell
while ($true) {
    Clear-Host

    Get-NetTCPConnection -State Listen |
        Select-Object LocalAddress, LocalPort, OwningProcess |
        Sort-Object LocalPort

    Start-Sleep -Seconds 3
}
```

These examples use native Windows commands.

---

# 5. Quick-reference command card

## curl

```bash
curl --version
curl -L -O "URL"
curl -L -o file.zip "URL"
curl -L -C - -o file.zip "URL"
curl -I "URL"
curl -v "URL"
curl -sS -o /dev/null -w "%{http_code}\n" "URL"
curl -T file.txt "UPLOAD_URL"
curl -F "file=@file.txt" "API_URL"
curl --help all
```

Windows PowerShell uses `curl.exe` and `NUL` rather than `/dev/null`.

## winget

```powershell
winget --version
winget search Python
winget show --id Git.Git --exact
winget install --id Git.Git --exact
winget list
winget upgrade
winget upgrade --all
winget uninstall --id Git.Git --exact
winget download --help
winget export --output apps.json
winget import --import-file apps.json
winget source list
winget --help
```

## watch

```bash
watch date
watch -n 5 uptime
watch -d -n 2 "free -h"
watch -n 3 "ps aux | head -15"
watch -n 5 df -h
watch -n 2 "ss -tuln"
watch -n 3 "systemctl is-active ssh"
watch --help
```

---

# 6. Practice exercises

1. **Download a file:** Use `curl -o` to save a trusted text file under a name you choose.
2. **Resume a download:** Test `curl -C -` with a large file from a server that supports resuming.
3. **Inspect HTTP:** Compare the output of `curl -I`, `curl -i`, and `curl -v`.
4. **Upload to Django:** Use `curl -F` with a multipart endpoint and the exact serializer field name.
5. **Install a Windows application:** Search for Git using winget, inspect its exact ID, and install it.
6. **Review upgrades:** Run `winget upgrade` before deciding whether to use `winget upgrade --all`.
7. **Export packages:** Export the application list to JSON and inspect the file before importing it elsewhere.
8. **Monitor memory:** Run `watch -n 2 free -h` and compare it with `top`.
9. **Monitor ports:** Run `watch -n 3 "ss -tuln"` in Ubuntu or WSL.
10. **Monitor Windows processes:** Run the PowerShell process-monitoring loop and stop it with `Ctrl+C`.

---

# 7. Official documentation

- curl manual: https://curl.se/docs/manpage.html
- Microsoft WinGet documentation: https://learn.microsoft.com/en-us/windows/package-manager/winget/
- Ubuntu watch manual: https://manpages.ubuntu.com/manpages/noble/man1/watch.1.html

**Remember:** `curl` transfers data, `winget` manages Windows packages, and `watch` repeatedly runs commands so you can observe changes.
