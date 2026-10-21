# Python Socket Programming Reference Guide

A practical, copyable reference for learning Python network programming
on Linux, Ubuntu, WSL, and other platforms.

> **Scope:** Core Python `socket` APIs, TCP and UDP, client/server
> examples, timeouts, message framing, multiple clients,
> troubleshooting, and safe practice.
>
> Official references: - https://docs.python.org/3/library/socket.html -
> https://docs.python.org/3/library/socketserver.html -
> https://docs.python.org/3/library/ssl.html -
> https://docs.python.org/3/library/selectors.html

------------------------------------------------------------------------

## 1. What is a socket?

A **socket** is a programming endpoint used to send or receive data. A
client and server can use sockets to communicate over a network.

Important terms:

  -----------------------------------------------------------------------
  Term                                Meaning
  ----------------------------------- -----------------------------------
  IP address                          Identifies a network interface or
                                      host address.

  Port                                Identifies a service endpoint on a
                                      host.

  Client                              Usually initiates a connection or
                                      sends a request.

  Server                              Waits for or receives requests and
                                      responds.

  Protocol                            Rules for exchanging data.

  TCP                                 Reliable, ordered byte-stream
                                      transport.

  UDP                                 Datagram transport without
                                      TCP-style delivery guarantees.

  `localhost`                         Name for the current computer.

  `127.0.0.1`                         IPv4 loopback address.

  `::1`                               IPv6 loopback address.

  `0.0.0.0`                           Wildcard IPv4 bind address; clients
                                      should connect to a real reachable
                                      address instead.
  -----------------------------------------------------------------------

## 2. Import the module and create sockets

The standard-library `socket` module is included with Python. Usually,
no third-party package is needed.

``` python
import socket

# IPv4 + TCP
tcp_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_socket.close()

# IPv4 + UDP
udp_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
udp_socket.close()
```

General form:

``` python
socket.socket(family=AF_INET, type=SOCK_STREAM, proto=0)
```

  -----------------------------------------------------------------------
  Argument                            Meaning
  ----------------------------------- -----------------------------------
  `family`                            Address family, such as `AF_INET`
                                      (IPv4) or `AF_INET6` (IPv6).

  `type`                              Socket type, such as `SOCK_STREAM`
                                      (usually TCP) or `SOCK_DGRAM`
                                      (usually UDP).

  `proto`                             Protocol number; `0` normally asks
                                      Python/the operating system to
                                      choose the appropriate protocol.
  -----------------------------------------------------------------------

Common combinations:

``` python
socket.socket(socket.AF_INET, socket.SOCK_STREAM)   # IPv4 TCP
socket.socket(socket.AF_INET, socket.SOCK_DGRAM)    # IPv4 UDP
socket.socket(socket.AF_INET6, socket.SOCK_STREAM)  # IPv6 TCP
socket.socket(socket.AF_INET6, socket.SOCK_DGRAM)   # IPv6 UDP
```

Use a context manager (`with`) when possible. It closes the socket
automatically when the block ends.

## 3. TCP compared with UDP

  -----------------------------------------------------------------------
  Feature                 TCP                     UDP
  ----------------------- ----------------------- -----------------------
  Socket type             `SOCK_STREAM`           `SOCK_DGRAM`

  Communication           Connection-oriented     Datagram-based, no
                                                  TCP-style connection
                                                  setup

  Delivery                Reliable, ordered byte  Delivery/order not
                          stream                  guaranteed

  Message boundaries      Not preserved           Each datagram has its
                                                  own boundary

  Typical methods         `connect`, `listen`,    `sendto`, `recvfrom`
                          `accept`, `sendall`,    
                          `recv`                  

  Example uses            Web traffic, file       DNS, real-time
                          transfer, remote shells applications, some
                                                  games
  -----------------------------------------------------------------------

**Important:** TCP preserves the order of bytes but not the boundaries
between individual calls to `send()` or `sendall()`. Your application
protocol must define how messages are separated.

## 4. Essential TCP methods

  ----------------------------------------------------------------------------
  Method                              Purpose / result
  ----------------------------------- ----------------------------------------
  `bind(address)`                     Assign a local address and port to a
                                      socket.

  `listen(backlog)`                   Mark a TCP socket as ready to accept
                                      connections. `backlog` is a requested
                                      queue size, not a limit on total
                                      clients.

  `accept()`                          Wait for a connection; returns
                                      `(connection_socket, client_address)`.

  `connect(address)`                  Connect a client socket to a server.

  `connect_ex(address)`               Similar to `connect()`, but returns an
                                      error indicator for connection errors
                                      instead of raising the usual connection
                                      exception.

  `send(data)`                        Send some bytes; it may send fewer bytes
                                      than supplied.

  `sendall(data)`                     Send all supplied bytes or raise an
                                      exception.

  `recv(size)`                        Receive up to `size` bytes. `b''` on a
                                      TCP connection means the peer has closed
                                      its sending side.

  `shutdown(how)`                     Disable sending, receiving, or both
                                      directions.

  `close()`                           Close the socket.
  ----------------------------------------------------------------------------

## 5. Essential UDP methods

  -----------------------------------------------------------------------
  Method                              Purpose / result
  ----------------------------------- -----------------------------------
  `sendto(data, address)`             Send a datagram to the given
                                      destination address.

  `recvfrom(size)`                    Receive a datagram; returns
                                      `(data, sender_address)`.

  `bind(address)`                     Bind a UDP socket to a local
                                      address and port.

  `settimeout(seconds)`               Configure a timeout for blocking
                                      operations.

  `close()`                           Close the socket.
  -----------------------------------------------------------------------

UDP does not guarantee that a datagram will arrive, arrive once, or
arrive in order. Applications needing those guarantees must implement
suitable logic or choose a transport/protocol that provides them.

## 6. Other useful socket methods and properties

  ------------------------------------------------------------------------
  Method/property                      Meaning
  ------------------------------------ -----------------------------------
  `gethostname()`                      Module function returning the
                                       machine hostname.

  `gethostbyname(host)`                Module function resolving a
                                       hostname to an IPv4 address.

  `getaddrinfo(host, port, ...)`       Module function for resolving
                                       addresses and service information;
                                       preferred for code that needs
                                       IPv4/IPv6 flexibility.

  `getfqdn(name='')`                   Module function returning a fully
                                       qualified domain name when
                                       available.

  `getsockname()`                      Return the socket's local address.

  `getpeername()`                      Return the connected peer's
                                       address.

  `settimeout(seconds)`                Set a timeout; `None` means
                                       blocking mode without a timeout.

  `gettimeout()`                       Get the configured timeout.

  `setblocking(flag)`                  Set blocking (`True`) or
                                       nonblocking (`False`) mode.

  `setsockopt(level, option, value)`   Set a socket option.

  `getsockopt(level, option)`          Read a socket option.

  `makefile(mode, ...)`                Create a file-like object around a
                                       socket.

  `fileno()`                           Return the underlying file
                                       descriptor.

  `family`                             Socket address family.

  `type`                               Socket type.

  `proto`                              Protocol number.
  ------------------------------------------------------------------------

## 7. Important constants

  -----------------------------------------------------------------------
  Constant                            Meaning
  ----------------------------------- -----------------------------------
  `socket.AF_INET`                    IPv4 address family

  `socket.AF_INET6`                   IPv6 address family

  `socket.AF_UNIX`                    Unix-domain sockets on supported
                                      systems

  `socket.SOCK_STREAM`                Stream socket, commonly TCP

  `socket.SOCK_DGRAM`                 Datagram socket, commonly UDP

  `socket.SOCK_RAW`                   Raw socket; support and privileges
                                      vary by platform

  `socket.SOL_SOCKET`                 Socket-level options

  `socket.SO_REUSEADDR`               Address reuse option; exact
                                      behavior is platform-dependent

  `socket.SO_KEEPALIVE`               Enable TCP keepalive behavior

  `socket.SO_BROADCAST`               Permit UDP broadcast sending where
                                      supported

  `socket.IPPROTO_TCP`                TCP protocol number

  `socket.IPPROTO_UDP`                UDP protocol number

  `socket.SHUT_RD`                    Disable further receiving

  `socket.SHUT_WR`                    Disable further sending

  `socket.SHUT_RDWR`                  Disable sending and receiving
  -----------------------------------------------------------------------

Example:

``` python
sock.setsockopt(socket.SOL_SOCKET, socket.SO_KEEPALIVE, 1)
```

Keepalive may help detect some dead connections, but it does not replace
application-level health checks.

## 8. Bytes and strings

Sockets transmit bytes, not Python text strings. Encode text before
sending and decode received text afterward.

``` python
message = "Hello from Python"

data = message.encode("utf-8")  # str -> bytes
print(data)                     # b'Hello from Python'

text = data.decode("utf-8")     # bytes -> str
print(text)                     # Hello from Python
```

Useful operations:

``` python
len(data)   # Number of bytes
data.hex()  # Hexadecimal representation of bytes
```

Use a consistent encoding, commonly UTF-8, for text protocols. Do not
decode arbitrary binary file contents as UTF-8.

## 9. Basic TCP server

Save as `server.py`:

``` python
import socket

HOST = "127.0.0.1"  # Local machine only
PORT = 5000

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
    server.bind((HOST, PORT))
    server.listen()

    print(f"Listening on {HOST}:{PORT}")

    # accept() returns a new socket for this client and its address.
    conn, address = server.accept()
    print("Client connected:", address)

    with conn:
        data = conn.recv(1024)

        if data:
            print("Client says:", data.decode("utf-8"))
            conn.sendall(b"Hello from the server!")

print("Server stopped.")
```

## 10. Basic TCP client

Save as `client.py`:

``` python
import socket

HOST = "127.0.0.1"
PORT = 5000

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client:
    client.connect((HOST, PORT))
    client.sendall("Hello from the client!".encode("utf-8"))

    response = client.recv(1024)
    if response:
        print("Server says:", response.decode("utf-8"))
```

Run the server first, then the client in a second terminal:

``` bash
python3 server.py
python3 client.py
```

Run those commands in separate terminals. The server above accepts one
connection and handles one short exchange.

### What happens?

1.  The server creates a socket.
2.  `bind()` chooses the local address and port.
3.  `listen()` makes the socket listen for TCP connections.
4.  The client calls `connect()`.
5.  The server's `accept()` returns a new connection socket.
6.  The programs exchange bytes.
7.  The context managers close their sockets.

## 11. Basic UDP server and client

Save as `udp_server.py`:

``` python
import socket

HOST = "127.0.0.1"
PORT = 5001

with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as server:
    server.bind((HOST, PORT))
    print(f"UDP server listening on {HOST}:{PORT}")

    data, address = server.recvfrom(1024)
    print("Received:", data.decode("utf-8"), "from", address)

    server.sendto(b"UDP message received!", address)
```

Save as `udp_client.py`:

``` python
import socket

HOST = "127.0.0.1"
PORT = 5001

with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as client:
    client.sendto(b"Hello through UDP!", (HOST, PORT))
    data, address = client.recvfrom(1024)
    print("Server replied:", data.decode("utf-8"))
```

Run `udp_server.py` first, then `udp_client.py` in another terminal.

## 12. Timeouts and blocking behavior

Blocking operations may wait indefinitely unless a timeout is
configured. A timeout is useful when a server is unavailable or a peer
stops responding.

``` python
import socket

HOST = "127.0.0.1"
PORT = 5000

try:
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client:
        client.settimeout(5)  # Seconds for blocking socket operations
        client.connect((HOST, PORT))
        client.sendall(b"Hello")
        response = client.recv(1024)
        print(response)
except socket.timeout:
    print("A socket operation timed out.")
except ConnectionRefusedError:
    print("No server accepted the connection.")
except OSError as error:
    print("Socket error:", error)
```

Modes:

``` python
sock.setblocking(True)   # Blocking mode
sock.setblocking(False)  # Nonblocking mode
sock.settimeout(5)       # Timeout mode
sock.gettimeout()        # Read timeout; None means no timeout
```

With nonblocking sockets, operations may raise `BlockingIOError` when
they cannot proceed immediately. Use `selectors` or another
readiness/event-loop mechanism to wait for activity.

## 13. Local and peer addresses

``` python
import socket

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client:
    client.connect(("example.com", 80))
    print("Local endpoint:", client.getsockname())
    print("Remote endpoint:", client.getpeername())
```

This example requires internet connectivity and a server accepting TCP
connections on port 80.

Other useful module functions:

``` python
import socket

print(socket.gethostname())
print(socket.gethostbyname("localhost"))
print(socket.getaddrinfo("localhost", 5000))
print(socket.getfqdn())
```

`gethostbyname()` resolves IPv4 only. For IPv4/IPv6-aware applications,
prefer `getaddrinfo()`.

## 14. Socket options and address reuse

During local development, a server may sometimes be unable to bind to a
port immediately after restarting. `SO_REUSEADDR` can help with some
address-reuse situations; behavior varies by operating system.

``` python
import socket

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server.bind(("127.0.0.1", 5000))
server.listen()

try:
    conn, address = server.accept()
    with conn:
        print("Connected:", address)
finally:
    server.close()
```

Set the option before calling `bind()`. It does not mean multiple
ordinary TCP servers can always share the same address and port.

## 15. TCP message framing

TCP is a **byte stream**. One call to `sendall()` does not correspond to
one call to `recv()`. A receiver may get only part of a message or
multiple messages together.

Common framing approaches:

1.  **Delimiter:** end each text message with a newline.
2.  **Length prefix:** send a fixed-size header containing the message
    length, followed by the body.
3.  **Fixed-size messages:** use only when the protocol defines a fixed
    length.

A simple delimiter example for a small teaching program:

``` python
def receive_line(sock, max_bytes=4096):
    """Read one UTF-8 line ending in newline, with a size limit."""
    data = bytearray()

    while True:
        chunk = sock.recv(1)

        if not chunk:
            raise ConnectionError("Peer closed before newline")

        if chunk == b"\n":
            return data.decode("utf-8")

        data.extend(chunk)

        if len(data) > max_bytes:
            raise ValueError("Message too large")
```

This one-byte-at-a-time implementation prioritizes clarity over
performance. A production implementation should buffer reads, handle
multiple messages, enforce limits, and manage timeouts. Never assume one
`recv(1024)` always returns the full application message.

## 16. Multiple clients with threading

This example starts one thread for each accepted TCP connection. It is a
teaching pattern, not a fully hardened server.

Save as `multi_server.py`:

``` python
import socket
import threading

HOST = "127.0.0.1"
PORT = 5002


def handle_client(conn, address):
    print("Connected:", address)

    with conn:
        try:
            while True:
                data = conn.recv(1024)
                if not data:
                    break

                print(f"{address}: {data!r}")
                conn.sendall(data)  # Echo the bytes back
        except OSError as error:
            print(f"Connection problem with {address}: {error}")

    print("Disconnected:", address)


with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
    server.bind((HOST, PORT))
    server.listen()
    print(f"Listening on {HOST}:{PORT}")

    try:
        while True:
            conn, address = server.accept()
            thread = threading.Thread(
                target=handle_client,
                args=(conn, address),
                daemon=True,
            )
            thread.start()
    except KeyboardInterrupt:
        print("\nStopping server.")
```

Save as `multi_client.py`:

``` python
import socket

HOST = "127.0.0.1"
PORT = 5002

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client:
    client.connect((HOST, PORT))
    print("Connected. Type quit to exit.")

    while True:
        message = input("You: ")
        if message.lower() == "quit":
            break

        client.sendall(message.encode("utf-8"))
        response = client.recv(1024)

        if not response:
            print("Server disconnected.")
            break

        print("Echo:", response.decode("utf-8"))
```

Run the server, then start one or more clients in separate terminals. A
production service should limit threads and concurrent connections and
handle shutdown carefully.

## 17. Higher-level server support: `socketserver`

Python's `socketserver` module can simplify basic server construction.

``` python
import socketserver


class EchoHandler(socketserver.BaseRequestHandler):
    def handle(self):
        data = self.request.recv(1024)
        if data:
            self.request.sendall(data)


with socketserver.ThreadingTCPServer(
    ("127.0.0.1", 5003),
    EchoHandler,
) as server:
    print("Echo server listening on port 5003")
    try:
        server.serve_forever()
    except KeyboardInterrupt:
        print("\nStopping server.")
```

`ThreadingTCPServer` uses threads to handle clients. For public-facing
applications, add appropriate timeouts, resource limits, input
validation, and error handling.

## 18. Common socket errors

  -----------------------------------------------------------------------------------
  Error                               Typical meaning         What to check
  ----------------------------------- ----------------------- -----------------------
  `ConnectionRefusedError`            No service accepted the Is the server running?
                                      connection.             Is the address and port
                                                              correct?

  `TimeoutError` / `socket.timeout`   An operation took too   Network connectivity,
                                      long.                   server response,
                                                              timeout.

  `OSError: Address already in use`   The address/port cannot Existing process, port,
                                      be bound as requested.  address-reuse behavior.

  `ConnectionResetError`              Peer reset the          Peer process and
                                      connection.             network interruptions.

  `BrokenPipeError`                   A send was attempted    Check peer state and
                                      after the connection    connection handling.
                                      was closed.             

  `socket.gaierror`                   Name/address resolution Hostname, DNS, system
                                      failed.                 name service.

  `BlockingIOError`                   A nonblocking operation Wait for readiness
                                      cannot proceed now.     before retrying.

  `UnicodeDecodeError`                Bytes do not match the  Encoding and protocol;
                                      chosen text encoding.   data may be binary.
  -----------------------------------------------------------------------------------

## 19. Linux network troubleshooting commands

``` bash
hostname
hostname -I
ip address
ip route
ss -lnt
ss -lnu
ss -lntp
nc -vz 127.0.0.1 5000
getent hosts example.com
sudo lsof -i :5000
```

  -----------------------------------------------------------------------
  Command                             Purpose
  ----------------------------------- -----------------------------------
  `hostname`                          Print the hostname.

  `hostname -I`                       Show detected IP addresses.

  `ip address`                        Show network interfaces and
                                      addresses.

  `ip route`                          Show routes.

  `ss -lnt`                           Show listening TCP sockets.

  `ss -lnu`                           Show UDP sockets.

  `ss -lntp`                          Show listening TCP sockets and
                                      process details where permitted.

  `nc -vz HOST PORT`                  Test TCP connectivity to a host and
                                      port.

  `getent hosts NAME`                 Query system host-name resolution.

  `sudo lsof -i :PORT`                Show processes using a port if
                                      `lsof` is installed.
  -----------------------------------------------------------------------

To stop a foreground Python server, press `Ctrl+C`.

## 20. Security practices

-   Bind to `127.0.0.1` when testing locally.
-   To accept connections from other machines, bind to a suitable
    interface or `0.0.0.0`, and configure firewall rules carefully.
-   A client must connect to a reachable server address, not `0.0.0.0`.
-   Treat all received data as untrusted.
-   Set timeouts and impose maximum message sizes.
-   Limit concurrent connections and resource usage.
-   Do not transmit passwords or private data over an unencrypted
    socket.
-   Use TLS with appropriate certificate validation when encryption and
    server authentication are required; see Python's `ssl`
    documentation.
-   Test network tools only on systems you own or are authorized to
    assess.
-   A basic TCP socket is not automatically encrypted or authenticated.

## 21. Ten projects to practise

1.  **TCP echo server:** return the same bytes the client sends.
2.  **UDP messaging:** practise `sendto()` and `recvfrom()`.
3.  **Multi-client chat:** manage several clients and broadcast
    messages.
4.  **File transfer:** transfer a file with explicit length framing and
    integrity checks.
5.  **Port connectivity checker:** test explicitly specified, authorized
    hosts and ports with timeouts.
6.  **LAN messenger:** communicate between devices on your own local
    network.
7.  **TLS client/server:** use certificates and verify server identity.
8.  **Service health checker:** test known services and log response
    times.
9.  **Network diagnostic tool:** combine DNS resolution and connection
    tests.
10. **Command protocol:** implement `HELP`, `TIME`, and `QUIT`, with
    authentication and strict access controls where needed.

Recommended order: TCP echo → UDP → multi-client server → framing and
file transfer → TLS.

## 22. Quick-reference cheat sheet

``` python
import socket

# Create sockets
tcp = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
udp = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# TCP server operations (use on the server socket)
tcp.bind(("127.0.0.1", 5000))
tcp.listen(5)
# conn, address = tcp.accept()

# TCP client operation (use on a client socket)
# tcp.connect(("127.0.0.1", 5000))

# Send/receive bytes on a connected TCP socket
# tcp.sendall("Hello".encode("utf-8"))
# data = tcp.recv(1024)

# UDP send/receive
udp.sendto(b"Hello", ("127.0.0.1", 5001))
# data, address = udp.recvfrom(1024)

# Timeout and endpoint information
tcp.settimeout(5)
# print(tcp.getsockname())
# print(tcp.getpeername())

# Name resolution
print(socket.gethostname())
print(socket.gethostbyname("localhost"))
print(socket.getaddrinfo("localhost", 5000))

# Release resources
tcp.close()
udp.close()
```

**Do not run the cheat sheet as one complete application.** Server and
client operations belong in their respective programs, and methods such
as `accept()` and `recv()` require the correct connection state.

## 23. Official documentation

-   Socket module: https://docs.python.org/3/library/socket.html
-   Socket server framework:
    https://docs.python.org/3/library/socketserver.html
-   TLS/SSL: https://docs.python.org/3/library/ssl.html
-   I/O multiplexing with `selectors`:
    https://docs.python.org/3/library/selectors.html
-   Asynchronous I/O with `asyncio`:
    https://docs.python.org/3/library/asyncio.html

------------------------------------------------------------------------

**Learning advice:** Do not only memorize method names. For each method,
learn what it expects, what it returns, whether it can block, what
errors it can raise, and which socket state it requires.
