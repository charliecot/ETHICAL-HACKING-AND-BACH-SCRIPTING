# Wget Cheatsheet — Beginner to Advanced

A copyable reference for Linux, Ubuntu, WSL, and Bash.

## 1. Basic syntax

```bash
wget [OPTIONS] URL
```

- `wget`: command-line downloader for HTTP, HTTPS, and FTP.
- `OPTIONS`: controls how the download behaves.
- `URL`: the address of the resource to download.

```bash
# Download using the server-provided filename
wget https://example.com/file.zip

# Save using a custom filename (overwrites an existing file)
wget -O myfile.zip https://example.com/file.zip

# Save into a directory
wget -P ~/Downloads https://example.com/file.zip
```

Replace example URLs with real URLs you are authorized to access.

## 2. Essential options

| Option | Meaning | Example |
|---|---|---|
| `-O FILE` | Save document to a specified filename | `wget -O app.zip URL` |
| `-o FILE` | Write messages to a log file | `wget -o wget.log URL` |
| `-a FILE` | Append messages to a log file | `wget -a wget.log URL` |
| `-c` | Continue a partial download when supported | `wget -c URL` |
| `-P DIR` | Save into a directory | `wget -P downloads URL` |
| `-q` | Quiet mode | `wget -q URL` |
| `-v` | Verbose output | `wget -v URL` |
| `-nv` | Less verbose output | `wget -nv URL` |
| `-i FILE` | Read URLs from a file | `wget -i urls.txt` |
| `--spider` | Check a URL without saving its content | `wget --spider URL` |
| `-S` | Print server response headers | `wget -S URL` |
| `--limit-rate=RATE` | Limit download speed | `wget --limit-rate=500k URL` |
| `-t N` | Retry N times | `wget -t 5 URL` |
| `-T SEC` | Set network timeout | `wget -T 20 URL` |
| `-w SEC` | Wait between retrievals | `wget -w 2 URL` |
| `--user-agent=STRING` | Set the User-Agent header | `wget --user-agent="MyDownloader/1.0" URL` |
| `--help` | Display help | `wget --help` |
| `--version` | Display version | `wget --version` |

`URL` is a placeholder, not a literal value to type.

## 3. Download files

```bash
# Download a PDF
wget https://example.com/document.pdf

# Choose the output filename
wget -O report.pdf https://example.com/document.pdf

# Save inside a directory
wget -P ~/Downloads https://example.com/document.pdf

# Download multiple files
wget https://example.com/file1.zip https://example.com/file2.zip

# Download URLs listed in a text file
wget -i urls.txt
```

Example `urls.txt`:

```text
https://example.com/file1.zip
https://example.com/file2.zip
https://example.com/document.pdf
```

Run:

```bash
wget -i urls.txt
```

## 4. Resume interrupted downloads

```bash
# Continue a partial download, if the server supports it
wget -c https://example.com/large-file.zip
```

If you specify a custom output filename, be careful when combining `-c` and `-O`. Output-file handling can vary by Wget version; verify the resulting file and its size.

## 5. Logs and quiet mode

```bash
# Write messages to a log file
wget -o wget.log https://example.com/file.zip

# Append messages to an existing log
wget -a wget.log https://example.com/file.zip

# Quiet download
wget -q https://example.com/file.zip

# Quiet download while writing a log
wget -q -o wget.log https://example.com/file.zip

# Read the log
cat wget.log

# Follow a log as it grows
tail -f wget.log
```

Remember the case difference:

- `-o` = output log.
- `-O` = output document.

Linux options are case-sensitive.

## 6. Check a URL and inspect response headers

```bash
# Check whether a URL appears accessible without downloading the body
wget --spider https://example.com/

# Print response headers while checking
wget -S --spider https://example.com/

# Show server response headers during a normal download
wget -S https://example.com/file.zip
```

A successful `--spider` check does not guarantee every later download will work. Some servers treat HEAD requests differently from normal GET requests.

## 7. Speed limits, retries, and timeouts

```bash
# Limit download speed to about 500 KB/s
wget --limit-rate=500k https://example.com/large.zip

# Limit download speed to about 2 MB/s
wget --limit-rate=2m https://example.com/large.zip

# Retry up to five times
wget -t 5 https://example.com/file.zip

# Set a network timeout of 20 seconds
wget -T 20 https://example.com/file.zip

# Wait two seconds between retrievals
wget -w 2 https://example.com/file.zip

# Retry indefinitely (use cautiously)
wget -t 0 https://example.com/file.zip
```

The `k` and `m` suffixes are commonly used for kilobytes and megabytes per second in Wget's rate-limit option.

## 8. HTTP headers and authentication

```bash
# Set a custom User-Agent
wget --user-agent="Mozilla/5.0" https://example.com/

# Add an HTTP header
wget --header="Accept: application/json" \
     -O response.json \
     https://example.com/api/data

# Send a bearer token (replace the placeholder)
wget --header="Authorization: Bearer YOUR_TOKEN" \
     -O response.json \
     https://example.com/api/data

# HTTP Basic Authentication (replace placeholders)
wget --user="USERNAME" --password="PASSWORD" \
     https://example.com/private/file.zip
```

Security notes:
- Avoid putting real passwords or tokens directly in commands. They may remain in shell history or be visible in process listings.
- Use a protected configuration file or another authentication method supported by the service where possible.
- Only access accounts and resources you are authorized to use.

## 9. Recursive downloads and website mirroring

```bash
# Download a page and its linked resources
wget -p https://example.com/

# Convert links to help browse downloaded pages offline
wget -k -p https://example.com/

# Recursively download links (use cautiously)
wget -r https://example.com/

# Limit recursion to two levels
wget -r -l 2 https://example.com/

# Mirror a site
wget --mirror https://example.com/

# Mirror and convert links for offline browsing
wget --mirror --convert-links https://example.com/
```

Useful recursive options:

| Option | Meaning |
|---|---|
| `-r` | Recursive retrieval |
| `-l N` | Maximum recursion depth |
| `-p` | Download page requisites such as images and stylesheets |
| `-k` | Convert links for offline browsing |
| `-m` | Mirror mode |
| `-np` | Do not ascend to parent directory levels |
| `-nd` | Do not create directories |
| `-A EXT` | Accept matching file extensions |
| `-R EXT` | Reject matching file extensions |

Example: download PDF files from an authorized directory, to a depth of two levels:

```bash
wget -r -l 2 -A pdf https://example.com/documents/
```

Recursive downloads can create many requests and files. Respect the website's terms, permissions, and rate limits. Do not use Wget to bypass access controls.

## 10. HTTPS certificates

```bash
# Normal HTTPS download; certificate validation is enabled by default
wget https://example.com/file.zip

# Use a specific CA certificate file
wget --ca-certificate=/path/to/ca-certificates.crt \
     https://example.com/file.zip
```

Avoid `--no-check-certificate` except in controlled testing when you understand the risk. It disables certificate verification and can expose the connection to man-in-the-middle attacks.

## 11. Combine Wget with other Linux commands

```bash
# Download a file and inspect its detected type
wget -O download.bin https://example.com/file
file download.bin

# Download a page and search it for a word
wget -q -O page.html https://example.com/
grep -in "example" page.html

# Download an archive and calculate its SHA-256 hash
wget -O archive.zip https://example.com/archive.zip
sha256sum archive.zip

# Download a list of URLs and append output to a log
wget -i urls.txt -a wget.log
```

Compare a downloaded file's hash with a checksum from a trusted source to help verify integrity.

## 12. Troubleshooting

| Problem | What to check |
|---|---|
| `wget: command not found` | Install Wget |
| `404 Not Found` | Check the URL and filename |
| `403 Forbidden` | Check permissions and authentication |
| Connection timeout | Try `-T 30`; inspect network connectivity |
| Interrupted download | Try `-c` |
| Certificate error | Check system time, certificate store, and server certificate |
| Download is too fast for your connection | Use `--limit-rate=500k` to cap speed |
| Unsure whether a URL responds | Try `wget -S --spider URL` |

Install on Ubuntu or WSL:

```bash
sudo apt update
sudo apt install wget
wget --version
```

On Windows, check package availability first:

```powershell
winget search wget
```

Then install using the exact package ID shown in the results, for example:

```powershell
winget install --id PACKAGE_ID_FROM_SEARCH -e
```

## 13. Quick-reference commands

```bash
# Basic download
wget URL

# Save under a chosen filename
wget -O FILE URL

# Save in a directory
wget -P DIR URL

# Resume a partial download
wget -c URL

# Read URLs from a file
wget -i urls.txt

# Write output to a log
wget -o wget.log URL

# Append output to a log
wget -a wget.log URL

# Check a URL
wget --spider URL

# Print response headers
wget -S --spider URL

# Limit download speed
wget --limit-rate=500k URL

# Set retries and timeout
wget -t 5 -T 20 URL

# Download PDFs recursively to a limited depth
wget -r -l 2 -A pdf URL

# Display help
wget --help
```

## 14. Practice exercises

1. Download a PDF and save it as `guide.pdf`.
2. Download a file into `~/Downloads`.
3. Resume a partial download.
4. Read three URLs from `urls.txt`.
5. Write download messages to `wget.log`.
6. Check a URL using `--spider`.
7. Limit a download to `300k`.
8. Download PDFs recursively to a depth of one level from a site you are allowed to access.
9. Calculate the SHA-256 hash of a downloaded file.
10. Explain the difference between `-o` and `-O`.

## Official references

- GNU Wget manual: https://www.gnu.org/software/wget/manual/wget.html
- GNU Wget website: https://www.gnu.org/software/wget/
- Ubuntu package search: https://packages.ubuntu.com/
