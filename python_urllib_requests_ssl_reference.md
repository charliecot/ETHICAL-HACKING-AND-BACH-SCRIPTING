# Python Networking Reference: `urllib`, `requests`, `ssl`, and HTTPS

> Detailed, copyable study notes on Python's built-in URL tools, the third-party `requests` package, and TLS/SSL security.

## Table of contents

1. [Big picture](#1-big-picture)
2. [Python import syntax](#2-python-import-syntax)
3. [`urllib` overview](#3-urllib-overview)
4. [`urllib.request`](#4-urllibrequest)
5. [`urllib.parse`](#5-urllibparse)
6. [`urllib.error`](#6-urlliberror)
7. [`ssl` and TLS](#7-ssl-and-tls)
8. [`requests` library](#8-requests-library)
9. [HTTP methods](#9-http-methods)
10. [Query parameters, headers, and request bodies](#10-query-parameters-headers-and-request-bodies)
11. [Reading responses](#11-reading-responses)
12. [Errors, timeouts, and robust code](#12-errors-timeouts-and-robust-code)
13. [TLS certificates and verification](#13-tls-certificates-and-verification)
14. [Sessions, authentication, and downloads](#14-sessions-authentication-and-downloads)
15. [Complete examples](#15-complete-examples)
16. [Common mistakes](#16-common-mistakes)
17. [Cheat sheet](#17-cheat-sheet)
18. [Practice projects](#18-practice-projects)

---

## 1. Big picture

Python can communicate with web servers and APIs using HTTP and HTTPS.

- **HTTP** is a protocol for requesting and returning web resources.
- **HTTPS** is HTTP protected by TLS encryption and certificate-based authentication.
- **`urllib`** is part of Python's standard library. It does not normally need separate installation.
- **`ssl`** is another standard-library module. It provides TLS/SSL support for network connections.
- **`requests`** is a separate third-party package that offers a convenient API for HTTP requests. Install it in the active Python environment.

Install `requests`:

```bash
python -m pip install requests
```

Check the installation:

```bash
python -m pip show requests
python -c "import requests; print(requests.__version__)"
```

### The correct imports

```python
from urllib import request       # Correct: singular module name
import urllib.request            # Correct: fully qualified module
import ssl                       # Correct: separate standard-library module
import requests                  # Correct if third-party package is installed
```

These are generally incorrect:

```python
from urllib import requests      # ImportError: urllib has no standard requests module
from urllib import SSL            # ImportError: SSL is not a urllib submodule
```

Python is case-sensitive. The standard-library module is named `ssl`, in lowercase. The third-party library is named `requests`, in lowercase and plural.

---

## 2. Python import syntax

Imports make code defined in modules available to your program.

### Import a whole module

```python
import urllib.request

response = urllib.request.urlopen("https://example.com", timeout=10)
response.close()
```

Because the module is imported by its full name, call its function using `urllib.request.urlopen()`.

### Import a module from a package

```python
from urllib import request

response = request.urlopen("https://example.com", timeout=10)
response.close()
```

Now the module is available as `request`.

### Import a function directly

```python
from urllib.request import urlopen

with urlopen("https://example.com", timeout=10) as response:
    print(response.status)
```

Now call `urlopen()` directly.

### Import with an alias

```python
import urllib.request as url_request

with url_request.urlopen("https://example.com", timeout=10) as response:
    print(response.status)
```

An alias gives an imported module a different local name.

### Import multiple names

```python
from urllib.parse import urlencode, urlparse
from urllib.error import HTTPError, URLError
```

### Why avoid wildcard imports?

```python
from urllib.request import *
```

This can place many names into your current namespace and make it harder to see where each name came from. Prefer explicit imports.

---

## 3. `urllib` overview

`urllib` is a package in Python's standard library for working with URLs and URL-related operations.

Important modules include:

| Module | Main purpose |
|---|---|
| `urllib.request` | Opens URLs and makes HTTP requests |
| `urllib.parse` | Parses, constructs, and encodes URLs |
| `urllib.error` | Provides exceptions related to URL operations |
| `urllib.response` | Contains response-related classes used by URL handlers |
| `urllib.robotparser` | Reads and interprets `robots.txt` rules |

### Simple `urllib.request` example

```python
from urllib.request import urlopen

with urlopen("https://example.com", timeout=10) as response:
    print("Status:", response.status)
    body = response.read()
    print(body.decode("utf-8"))
```

Important details:

- `urlopen()` opens the URL and returns a response-like object.
- `response.status` is the HTTP status code, when available.
- `response.read()` returns bytes, not a Python string.
- `.decode("utf-8")` converts bytes into text using UTF-8.
- `with` ensures that the response is closed when the block exits.
- `timeout=10` sets a timeout for the connection/response operations; it is not necessarily a strict total-duration limit.

---

## 4. `urllib.request`

`urllib.request` provides functions and classes for opening URLs and sending requests.

### 4.1 `urlopen()`

Use `urlopen()` for a straightforward request.

```python
from urllib.request import urlopen

with urlopen("https://example.com", timeout=10) as response:
    print(response.status)
    print(response.headers)
    print(response.read().decode("utf-8"))
```

Useful response attributes and methods include:

- `status`: HTTP status code, such as `200`.
- `headers`: response headers.
- `read()`: reads response body bytes.
- `geturl()`: returns the final URL after redirects, if any.
- `getcode()`: returns the status code on compatible response objects.
- `close()`: closes the response.

### 4.2 Create a `Request`

`Request` lets you specify a method, headers, and body.

```python
from urllib.request import Request, urlopen

req = Request(
    "https://example.com",
    headers={"User-Agent": "PythonLearningExample/1.0"},
    method="GET",
)

with urlopen(req, timeout=10) as response:
    print(response.status)
    print(response.read().decode("utf-8"))
```

### 4.3 Send JSON using `urllib`

The body must be encoded to bytes. Use Python's `json` module to serialize a dictionary.

```python
import json
from urllib.request import Request, urlopen
from urllib.error import HTTPError, URLError

url = "https://jsonplaceholder.typicode.com/posts"
payload = {
    "title": "Learning Python",
    "body": "Practising HTTP requests",
    "userId": 1,
}

body = json.dumps(payload).encode("utf-8")

req = Request(
    url,
    data=body,
    headers={"Content-Type": "application/json"},
    method="POST",
)

try:
    with urlopen(req, timeout=10) as response:
        response_body = response.read().decode("utf-8")
        print("Status:", response.status)
        print(response_body)
except HTTPError as error:
    print("HTTP status:", error.code)
    print(error.read().decode("utf-8", errors="replace"))
except URLError as error:
    print("URL/connection problem:", error.reason)
```

JSONPlaceholder is a testing service. A successful test response does not necessarily mean data has been permanently stored.

### 4.4 What does `data=` mean in `urllib.request.Request`?

- If `data` is omitted, the request is usually a GET.
- If `data` is supplied, the request is usually a POST unless you specify a method.
- The body must be bytes or a compatible bytes-like object.
- Set `Content-Type` to describe the body format.

For form-encoded data:

```python
from urllib.parse import urlencode
from urllib.request import Request, urlopen

form_data = {"username": "charles", "topic": "python"}
body = urlencode(form_data).encode("utf-8")

req = Request(
    "https://httpbin.org/post",
    data=body,
    headers={"Content-Type": "application/x-www-form-urlencoded"},
    method="POST",
)

with urlopen(req, timeout=10) as response:
    print(response.read().decode("utf-8"))
```

---

## 5. `urllib.parse`

`urllib.parse` helps split URLs into components and encode or decode URL data.

### 5.1 Parse a URL with `urlparse()`

```python
from urllib.parse import urlparse

url = "https://example.com:443/products?id=25#details"
parts = urlparse(url)

print(parts.scheme)    # https
print(parts.netloc)    # example.com:443
print(parts.hostname)  # example.com
print(parts.port)      # 443
print(parts.path)      # /products
print(parts.query)     # id=25
print(parts.fragment)  # details
```

Common URL components:

- `scheme`: protocol, such as `https`.
- `netloc`: network location, often a host and optional port.
- `hostname`: host name.
- `port`: explicit port, if present.
- `path`: resource path.
- `query`: text after `?`.
- `fragment`: text after `#`; it is generally processed by the client and not sent to the server as part of an HTTP request.

### 5.2 Encode query parameters with `urlencode()`

```python
from urllib.parse import urlencode

params = {
    "q": "python requests",
    "page": 2,
    "language": "en",
}

query_string = urlencode(params)
print(query_string)
```

The result is an encoded query string. Spaces and special characters are encoded appropriately.

### 5.3 Build a URL with query parameters

```python
from urllib.parse import urlencode

base_url = "https://example.com/search"
params = {"q": "Python basics", "page": 1}

url = f"{base_url}?{urlencode(params)}"
print(url)
```

For a URL with an existing query string, take care not to append a second `?`. The `requests` library's `params=` argument is often easier.

### 5.4 Decode query parameters

```python
from urllib.parse import parse_qs, parse_qsl

query = "tag=python&tag=web&page=2"

print(parse_qs(query))
# {'tag': ['python', 'web'], 'page': ['2']}

print(parse_qsl(query))
# [('tag', 'python'), ('tag', 'web'), ('page', '2')]
```

- `parse_qs()` returns a dictionary whose values are lists.
- `parse_qsl()` returns a list of key-value pairs.

### 5.5 Quote individual values

```python
from urllib.parse import quote, unquote

value = "python & networking"
encoded = quote(value)
decoded = unquote(encoded)

print(encoded)
print(decoded)
```

Use URL-encoding tools instead of manually replacing spaces or special characters.

---

## 6. `urllib.error`

`urllib.error` contains exceptions used by `urllib.request`.

### `URLError`

Often represents a URL, connection, DNS, or other access problem.

### `HTTPError`

Represents an HTTP error status returned by a server, such as 404 or 500. It is also a kind of `URLError`, so catch `HTTPError` first if you want to handle it separately.

```python
from urllib.request import urlopen
from urllib.error import HTTPError, URLError

try:
    with urlopen("https://example.com/missing-page", timeout=10) as response:
        print(response.status)
except HTTPError as error:
    print("HTTP status:", error.code)
    print("Reason:", error.reason)
except URLError as error:
    print("URL/connection problem:", error.reason)
```

An error can arise from the server's HTTP status, DNS resolution, connectivity, TLS verification, or other network conditions. Handle expected errors specifically instead of hiding every error with a broad `except Exception`.

---

## 7. `ssl` and TLS

### 7.1 SSL versus TLS

SSL means **Secure Sockets Layer**. It is obsolete and has been replaced by TLS, or **Transport Layer Security**. Python's module retains the historical name `ssl`, but modern secure connections use TLS.

TLS provides:

1. **Confidentiality:** encrypts traffic in transit.
2. **Integrity:** helps detect tampering.
3. **Authentication:** verifies a server's identity using certificates.

HTTPS is HTTP carried over TLS.

### 7.2 Import `ssl`

```python
import ssl

print(ssl.OPENSSL_VERSION)
```

This prints the OpenSSL version used by the Python SSL implementation, when available.

### 7.3 Create a secure default context

```python
import ssl

context = ssl.create_default_context()
print(context.verify_mode)
print(context.check_hostname)
```

`ssl.create_default_context()` is the recommended starting point for ordinary client TLS connections. It enables certificate verification and hostname checking using the system/Python trust configuration.

### 7.4 Use a context with `urllib`

```python
import ssl
from urllib.request import urlopen

context = ssl.create_default_context()

with urlopen(
    "https://example.com",
    context=context,
    timeout=10,
) as response:
    print(response.status)
    print(response.read().decode("utf-8"))
```

The context controls TLS settings for the connection. For a normal HTTPS request, `urllib` already uses secure certificate verification by default, so supplying a default context explicitly is often unnecessary.

### 7.5 Certificates and certificate authorities (CAs)

A server presents a TLS certificate. A client checks whether the certificate chains to a trusted certificate authority and whether the hostname matches the certificate.

Certificate errors can occur because:

- the certificate has expired;
- the URL hostname does not match the certificate;
- the certificate chain is incomplete or untrusted;
- the computer's clock is wrong;
- a corporate proxy is intercepting TLS with a private CA that the client does not trust;
- CA certificates are missing or outdated.

The right fix is usually to correct the certificate, hostname, system time, CA trust, or proxy configuration.

### 7.6 Do not disable verification casually

Avoid this pattern:

```python
# INSECURE: demonstration only; do not use for normal production requests.
import requests

response = requests.get(
    "https://example.com",
    verify=False,
    timeout=10,
)
```

With `verify=False`, Requests does not verify the server's certificate. This can expose the connection to interception. It may also generate an `InsecureRequestWarning`.

Prefer:

```python
import requests

response = requests.get("https://example.com", timeout=10)
response.raise_for_status()
```

If your environment uses a custom trusted CA bundle, configure it explicitly:

```python
import requests

response = requests.get(
    "https://example.com",
    verify="/path/to/trusted-ca-bundle.pem",
    timeout=10,
)
response.raise_for_status()
```

Replace the example path with a real CA bundle path from a trusted source.

### 7.7 TLS in Requests

Requests verifies HTTPS certificates by default. You usually do not need to import `ssl` for standard `requests.get()` or `requests.post()` calls.

The `verify` argument accepts:
- `True` (default): verify using the configured trust store.
- `False`: skip verification; insecure.
- A string path: use a specific CA bundle.

Importing `ssl` does not automatically make Requests use a custom `SSLContext`. Requests manages TLS through its HTTP transport adapter.

---

## 8. `requests` library

Requests is a third-party Python library for making HTTP requests with readable, convenient syntax.

Install it:

```bash
python -m pip install requests
```

Import it:

```python
import requests
```

### 8.1 A basic GET request

```python
import requests

response = requests.get("https://example.com", timeout=10)

print(response.status_code)
print(response.headers)
print(response.text)
```

Important objects and attributes:

- `requests.get(url)`: sends a GET request.
- `response.status_code`: numeric HTTP status code.
- `response.text`: response body decoded as text.
- `response.content`: raw response body as bytes.
- `response.json()`: parses a JSON response into Python objects.
- `response.headers`: response headers.
- `response.url`: final request URL.
- `response.ok`: true for status codes below 400; it does not mean the status was exactly 200.
- `response.raise_for_status()`: raises `HTTPError` for unsuccessful 4xx/5xx status codes.

Always use a timeout in production code.

### 8.2 GET with status checking

```python
import requests

try:
    response = requests.get("https://example.com", timeout=10)
    response.raise_for_status()
    print(response.text)
except requests.exceptions.RequestException as error:
    print("Request failed:", error)
```

### 8.3 A JSON API request

```python
import requests

url = "https://jsonplaceholder.typicode.com/users"

response = requests.get(url, timeout=10)
response.raise_for_status()

users = response.json()

print(type(users))       # Usually list for this endpoint
print(users[0]["name"])
```

JSON values map into Python values:

| JSON | Python |
|---|---|
| object | `dict` |
| array | `list` |
| string | `str` |
| number | `int` or `float` |
| `true` / `false` | `True` / `False` |
| `null` | `None` |

A response may not contain valid JSON even if the request succeeds. `response.json()` can raise a JSON decoding error.

### 8.4 POST JSON data

```python
import requests

url = "https://jsonplaceholder.typicode.com/posts"
payload = {
    "title": "Learning Python",
    "body": "Practising APIs",
    "userId": 1,
}

response = requests.post(url, json=payload, timeout=10)
response.raise_for_status()

print(response.status_code)
print(response.json())
```

The `json=` parameter serializes the Python object to JSON and sets the JSON content type.

### 8.5 `json=` versus `data=`

```python
requests.post(url, json={"name": "Charles"})
```

Use `json=` when the server expects JSON.

```python
requests.post(url, data={"username": "charles"})
```

When `data=` receives a dictionary, Requests normally sends form-encoded data. Use the format the server's API expects.

To send raw bytes or a pre-encoded body, `data=` can also accept a bytes-like value.

---

## 9. HTTP methods

HTTP methods express the action a client wants to perform.

| Method | Typical purpose |
|---|---|
| `GET` | Retrieve a resource |
| `POST` | Submit data or create/process a resource |
| `PUT` | Replace a resource |
| `PATCH` | Partially update a resource |
| `DELETE` | Delete a resource |
| `HEAD` | Retrieve headers without the normal response body |
| `OPTIONS` | Discover communication options supported by a resource |

Examples:

```python
import requests

base_url = "https://jsonplaceholder.typicode.com/posts"

# GET: retrieve a resource
r = requests.get(f"{base_url}/1", timeout=10)

# POST: submit/create data
r = requests.post(
    base_url,
    json={"title": "Hello", "body": "Example", "userId": 1},
    timeout=10,
)

# PUT: replace a resource representation
r = requests.put(
    f"{base_url}/1",
    json={"id": 1, "title": "Updated", "body": "New body", "userId": 1},
    timeout=10,
)

# PATCH: partially update a resource
r = requests.patch(
    f"{base_url}/1",
    json={"title": "Partially updated"},
    timeout=10,
)

# DELETE: request deletion
r = requests.delete(f"{base_url}/1", timeout=10)
```

These examples illustrate syntax. A testing API may simulate changes instead of persisting them.

---

## 10. Query parameters, headers, and request bodies

### 10.1 Query parameters

Avoid building query strings manually when using Requests.

```python
import requests

response = requests.get(
    "https://httpbin.org/get",
    params={"q": "python networking", "page": 2},
    timeout=10,
)

print(response.url)
print(response.json())
```

Requests encodes the parameter values for the URL.

### 10.2 Request headers

Headers carry metadata about a request.

```python
import requests

headers = {
    "Accept": "application/json",
    "User-Agent": "PythonStudyClient/1.0",
}

response = requests.get(
    "https://httpbin.org/headers",
    headers=headers,
    timeout=10,
)

print(response.json())
```

Common headers include:
- `Accept`: response formats the client can accept.
- `Content-Type`: format of the request body.
- `Authorization`: credentials or token for authentication.
- `User-Agent`: identifies the client software.

Do not put passwords or API keys in URLs. URLs can be logged or retained in browser history and server logs.

### 10.3 Form data

```python
import requests

form_data = {
    "username": "charles",
    "topic": "python",
}

response = requests.post(
    "https://httpbin.org/post",
    data=form_data,
    timeout=10,
)

print(response.json())
```

### 10.4 JSON data

```python
response = requests.post(
    "https://httpbin.org/post",
    json={"username": "charles", "active": True},
    timeout=10,
)
```

Use `json=` for JSON APIs and `data=` for form data or a deliberately prepared raw body.

---

## 11. Reading responses

### Text

```python
print(response.text)
```

Use when the response is textual, such as HTML, plain text, or JSON represented as text.

### Bytes

```python
content = response.content
print(type(content))  # bytes
```

Use bytes for images, PDFs, archives, and other binary content.

### JSON

```python
data = response.json()
print(data)
```

This parses JSON into Python values. It raises an error if the response is not valid JSON.

### Headers

```python
print(response.headers)
print(response.headers.get("Content-Type"))
```

Header names are case-insensitive in HTTP.

### Status code

Common HTTP codes:

| Code | Meaning |
|---:|---|
| `200` | OK |
| `201` | Created |
| `202` | Accepted for processing |
| `204` | No Content |
| `301` / `302` | Redirect |
| `400` | Bad Request |
| `401` | Unauthorized: authentication is missing or invalid |
| `403` | Forbidden |
| `404` | Not Found |
| `405` | Method Not Allowed |
| `409` | Conflict |
| `422` | Unprocessable Content, often used for validation errors |
| `429` | Too Many Requests |
| `500` | Internal Server Error |
| `502` | Bad Gateway |
| `503` | Service Unavailable |
| `504` | Gateway Timeout |

The exact meaning and behavior can depend on the server/API implementation.

---

## 12. Errors, timeouts, and robust code

### 12.1 Why timeouts matter

Without a timeout, a request may wait much longer than your program can reasonably tolerate.

```python
response = requests.get("https://example.com", timeout=10)
```

A single number applies to the connect and read timeout settings. You can also specify them separately:

```python
response = requests.get(
    "https://example.com",
    timeout=(3.05, 10),
)
```

This uses a 3.05-second connection timeout and a 10-second read timeout. A read timeout generally limits waiting for data between received bytes; it is not necessarily a total wall-clock deadline.

### 12.2 Useful Requests exceptions

- `requests.exceptions.Timeout`: connection or read timeout.
- `requests.exceptions.ConnectionError`: connection-related problem.
- `requests.exceptions.HTTPError`: raised by `raise_for_status()` for 4xx/5xx responses.
- `requests.exceptions.TooManyRedirects`: redirect limit exceeded.
- `requests.exceptions.RequestException`: base class for most Requests-related errors.

### 12.3 Robust GET pattern

```python
import requests

url = "https://jsonplaceholder.typicode.com/users"

try:
    response = requests.get(url, timeout=(3.05, 10))
    response.raise_for_status()
    users = response.json()
    print(users)
except requests.exceptions.Timeout:
    print("The request timed out.")
except requests.exceptions.HTTPError as error:
    print("The server returned an HTTP error:", error)
except requests.exceptions.ConnectionError:
    print("Could not connect to the server.")
except requests.exceptions.RequestException as error:
    print("Request failed:", error)
except ValueError:
    print("The response was not valid JSON.")
```

For some Requests versions, JSON decoding errors may also be represented by a Requests-specific exception that inherits from a JSON decoding exception. The broad `RequestException` handler handles transport/request failures; keep JSON parsing error handling where it is appropriate.

### 12.4 Retry considerations

Retries can help with temporary failures, but do not blindly repeat operations such as payments or order creation. Retrying a POST can create duplicates unless the API supports idempotency keys or another safe retry mechanism.

For retry logic, use a deliberate policy with limited attempts, backoff, and attention to HTTP status codes and method safety.

---

## 13. TLS certificates and verification

### 13.1 Requests default

```python
import requests

response = requests.get("https://example.com", timeout=10)
response.raise_for_status()
```

HTTPS certificate verification is enabled by default.

### 13.2 Use a custom CA bundle

```python
import requests

response = requests.get(
    "https://internal.example",
    verify="/path/to/organization-ca-bundle.pem",
    timeout=10,
)
response.raise_for_status()
```

Use the actual CA bundle supplied by your organization's trusted administrator. Do not trust a certificate merely because a website or an error message suggests downloading it.

### 13.3 Client certificates

Some services require a client certificate for mutual TLS:

```python
import requests

response = requests.get(
    "https://mutual-tls.example",
    cert=("/path/to/client-cert.pem", "/path/to/client-key.pem"),
    timeout=10,
)
```

The paths are placeholders. Protect private keys and never commit them to source control.

### 13.4 Debugging TLS problems

If you see errors such as `SSLError` or `CERTIFICATE_VERIFY_FAILED`, check:

1. The URL uses the expected hostname.
2. The system date and time are correct.
3. Python and the certificate bundle are reasonably up to date.
4. The server sends a complete, valid certificate chain.
5. A proxy is not intercepting TLS with an untrusted CA.
6. If a private CA is required, configure that CA securely.

Do not make `verify=False` the permanent fix.

---

## 14. Sessions, authentication, and downloads

### 14.1 Reuse a `requests.Session`

A Session can reuse connections and retain shared settings such as cookies.

```python
import requests

with requests.Session() as session:
    session.headers.update({"Accept": "application/json"})

    first = session.get("https://httpbin.org/get", timeout=10)
    first.raise_for_status()

    second = session.get("https://httpbin.org/headers", timeout=10)
    second.raise_for_status()

    print(first.json())
    print(second.json())
```

### 14.2 Bearer-token authentication

```python
import requests

token = "REPLACE_WITH_YOUR_TOKEN"

response = requests.get(
    "https://api.example.com/profile",
    headers={"Authorization": f"Bearer {token}"},
    timeout=10,
)

response.raise_for_status()
print(response.json())
```

Use a real token from a trusted source. Do not publish tokens or commit them to a repository. In actual projects, load secrets from environment variables or a secret manager.

### 14.3 Basic authentication

```python
import requests

response = requests.get(
    "https://httpbin.org/basic-auth/user/pass",
    auth=("user", "pass"),
    timeout=10,
)

print(response.status_code)
```

This endpoint is a demonstration service. Real credentials should only be sent over HTTPS to a trusted server.

### 14.4 Download a file in chunks

```python
import requests

url = "https://example.com/large-file.zip"
destination = "large-file.zip"

try:
    with requests.get(url, stream=True, timeout=(5, 30)) as response:
        response.raise_for_status()

        with open(destination, "wb") as file:
            for chunk in response.iter_content(chunk_size=8192):
                if chunk:
                    file.write(chunk)

    print("Download complete.")
except requests.exceptions.RequestException as error:
    print("Download failed:", error)
```

Streaming avoids loading the entire file into memory. For untrusted downloads, also consider file-size limits, disk-space limits, content validation, and whether the destination path is safe.

---

## 15. Complete examples

### Example A: GET JSON and display selected fields

```python
import requests

url = "https://jsonplaceholder.typicode.com/users"

try:
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    users = response.json()

    for user in users:
        print(f"Name: {user['name']}")
        print(f"Email: {user['email']}")
        print("-" * 30)

except requests.exceptions.RequestException as error:
    print("HTTP/network problem:", error)
except (ValueError, KeyError, TypeError) as error:
    print("Could not process the returned data:", error)
```

### Example B: Add query parameters

```python
import requests

params = {"userId": 1}
url = "https://jsonplaceholder.typicode.com/posts"

try:
    response = requests.get(url, params=params, timeout=10)
    response.raise_for_status()

    for post in response.json():
        print(post["id"], post["title"])
except requests.exceptions.RequestException as error:
    print("Request failed:", error)
```

### Example C: POST JSON

```python
import requests

payload = {
    "title": "My first API post",
    "body": "I am learning Python networking.",
    "userId": 1,
}

try:
    response = requests.post(
        "https://jsonplaceholder.typicode.com/posts",
        json=payload,
        timeout=10,
    )
    response.raise_for_status()
    print("Status:", response.status_code)
    print("Response:", response.json())
except requests.exceptions.RequestException as error:
    print("POST request failed:", error)
```

### Example D: Use `urllib` with a custom header

```python
from urllib.request import Request, urlopen
from urllib.error import HTTPError, URLError

req = Request(
    "https://httpbin.org/headers",
    headers={"User-Agent": "PythonStudyClient/1.0"},
)

try:
    with urlopen(req, timeout=10) as response:
        print(response.status)
        print(response.read().decode("utf-8"))
except HTTPError as error:
    print("HTTP error:", error.code)
except URLError as error:
    print("Connection/URL error:", error.reason)
```

---

## 16. Common mistakes

### Mistake 1: Importing `requests` from `urllib`

Incorrect:

```python
from urllib import requests
```

Correct:

```python
import requests
```

Install the package first if necessary:

```bash
python -m pip install requests
```

### Mistake 2: Importing `SSL` from `urllib`

Incorrect:

```python
from urllib import SSL
```

Correct:

```python
import ssl
```

### Mistake 3: Confusing `request` and `requests`

```python
from urllib import request  # Standard library module
import requests             # Third-party package
```

These are different modules.

### Mistake 4: Treating bytes as text

Incorrect:

```python
body = response.read()
print(body.upper())  # This is bytes, not str
```

If you need text:

```python
body = response.read().decode("utf-8")
print(body)
```

The correct encoding depends on the content. For JSON, use a JSON parser where possible.

### Mistake 5: Forgetting to check HTTP status

A server can return an error response without a network failure.

```python
response = requests.get("https://example.com/missing", timeout=10)
response.raise_for_status()
```

### Mistake 6: Omitting a timeout

Prefer:

```python
requests.get("https://example.com", timeout=10)
```

over a request with no timeout.

### Mistake 7: Disabling TLS verification

Avoid `verify=False` as a general fix. Correct the trust or certificate problem instead.

### Mistake 8: Installing into the wrong Python environment

If `import requests` fails, verify which Python and pip are active:

```bash
python --version
python -m pip --version
python -m pip show requests
```

Activate the correct virtual environment before installing the package.

---

## 17. Cheat sheet

```python
# Standard-library imports
import urllib.request
import urllib.parse
import urllib.error
import ssl

# Direct imports from urllib
from urllib.request import urlopen, Request
from urllib.parse import urlencode, urlparse, parse_qs
from urllib.error import HTTPError, URLError

# Third-party package
import requests

# GET with urllib
with urlopen("https://example.com", timeout=10) as response:
    status = response.status
    body_text = response.read().decode("utf-8")

# GET with Requests
response = requests.get("https://example.com", timeout=10)
response.raise_for_status()
status = response.status_code
body_text = response.text

# JSON
data = response.json()

# Query parameters
response = requests.get(
    "https://httpbin.org/get",
    params={"q": "python", "page": 1},
    timeout=10,
)

# POST JSON
response = requests.post(
    "https://httpbin.org/post",
    json={"name": "Charles"},
    timeout=10,
)

# TLS context for urllib
context = ssl.create_default_context()

# Certificate verification is on by default in Requests
response = requests.get("https://example.com", verify=True, timeout=10)
```

### Quick memory trick

- **`urllib.request`**: built-in URL and HTTP tools.
- **`urllib.parse`**: URL components and encoding.
- **`urllib.error`**: URL-related exceptions.
- **`ssl`**: TLS/SSL functionality.
- **`requests`**: third-party HTTP library.

---

## 18. Practice projects

1. **Website status checker** — accept a URL and print the HTTP status code. Handle invalid URLs, timeouts, and connection failures.
2. **API user viewer** — fetch JSON users and display their names and email addresses.
3. **Query-string builder** — accept search terms and page numbers, then encode them with `urllib.parse.urlencode()`.
4. **HTTP header inspector** — fetch a webpage and display its `Content-Type`, server response code, and final URL.
5. **JSON POST client** — send a dictionary to a test API and inspect the returned JSON.
6. **File downloader** — stream a file to disk, use timeouts, and handle errors.
7. **TLS verification investigation** — inspect how a valid HTTPS site behaves with default verification. Do not disable verification on sensitive requests.
8. **Session-based client** — reuse a `requests.Session` and common headers across multiple calls.
9. **HTTP error reporter** — distinguish a timeout, connection failure, and HTTP 404/500 response.
10. **Mini API client** — combine query parameters, JSON, status checking, authentication headers, and clear error messages.

## Final summary

Your original statements were:

```python
from urllib import requests, SSL
import requests
```

The first line is incorrect because the standard `urllib` package does not export a `requests` module or an `SSL` module.

Use:

```python
from urllib import request
import ssl
import requests
```

Remember that `urllib` and `ssl` are included with Python, while `requests` is a separate package that you usually install. For secure HTTPS communication, keep certificate verification enabled and set reasonable timeouts.
