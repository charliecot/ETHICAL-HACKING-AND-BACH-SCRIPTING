# Python Cryptography: From Zero to Advanced

## Bytes, `encode()` / `decode()`, data representation, file modes, AES, RSA, and best practices

This guide builds a mental model first, then shows practical Python
examples. Use the examples for learning and for data you own or are
authorized to protect.

## Table of contents

1.  Big picture
2.  Bits, bytes, strings, binary, hex and Base64
3.  `encode()` and `decode()`
4.  Encoding vs hashing vs encryption vs signatures
5.  `open()` and file modes
6.  Keys, IVs, nonces, tags and padding
7.  AES and its modes
8.  RSA, PEM, DER, PKCS #1 and PKCS #8
9.  Hybrid encryption
10. Password-derived keys
11. Complete AES-GCM example
12. Common errors
13. Best-practice checklist
14. Exercises and official references

------------------------------------------------------------------------

# 1. The big picture

Cryptographic programs move data between representations:

``` text
Text (str) --encode("utf-8")--> bytes --encryption--> ciphertext bytes
ciphertext bytes --Base64/hex representation, if needed--> printable text
```

To recover text:

``` text
Base64/hex text --decode representation--> ciphertext bytes
ciphertext bytes --decryption + authentication check--> plaintext bytes
plaintext bytes --decode("utf-8")--> text (str)
```

These operations are different: - **Encoding** changes representation. -
**Encryption** protects confidentiality using a key. - **Hashing**
creates a one-way digest. - **Decryption** recovers plaintext using the
correct key and parameters. - **Authentication** detects tampering or
verifies who produced data.

## A useful rule

-   Text: `str`, `.encode()`, `.decode()`
-   Raw binary: `bytes`, `bytearray`, file modes `"rb"` and `"wb"`
-   Printable representation: `.hex()`, `bytes.fromhex()`, Base64
-   Encryption: AES, RSA-OAEP, authenticated encryption
-   Integrity: authenticated encryption or a MAC/signature, depending on
    the task

------------------------------------------------------------------------

# 2. Bits, bytes, strings, binary, hex and Base64

## 2.1 Bits and bytes

A bit is `0` or `1`. A byte is 8 bits. A byte can hold values from 0 to
255.

``` python
n = 65
print(n)                 # 65: integer
print(bytes([n]))        # b'A': one byte with value 65
print(bytes([n]).hex())  # 41: hex representation
```

Do not confuse `65`, `"A"`, and `b"A"`: an integer, text, and bytes are
different Python values.

## 2.2 Python types

``` python
text = "Hello"              # str: text
raw = b"Hello"              # bytes: immutable bytes
mutable = bytearray(b"Hi")  # bytearray: mutable bytes
```

``` python
print(type(text))     # str
print(type(raw))      # bytes
print(type(mutable))  # bytearray
```

Bytes can represent text, an image, encrypted content, compressed data,
or any other binary format.

## 2.3 Decimal, binary and hexadecimal

  Representation          Value 65
  ----------------------- ------------
  Decimal (base 10)       `65`
  Binary (base 2)         `01000001`
  Hexadecimal (base 16)   `41`
  ASCII character         `A`

``` python
print(bin(65))           # 0b1000001
print(hex(65))           # 0x41
print(int("01000001", 2))  # 65
print(int("41", 16))       # 65
```

One hex digit represents 4 bits, so **two hex digits represent one
byte**. For example, `FF` is binary `11111111`, decimal `255`.

## 2.4 Bytes to hex and back

``` python
data = b"ABC"
hex_text = data.hex()

print(hex_text)                 # 414243
print(bytes.fromhex(hex_text))  # b'ABC'
```

Hex is a representation, not encryption. Anyone can convert it back.

## 2.5 Base64

Base64 turns arbitrary bytes into printable ASCII characters, useful
when a system expects text. It does not hide data.

``` python
import base64

raw = b"\x00\xff\x10secret"
encoded_bytes = base64.b64encode(raw)
encoded_text = encoded_bytes.decode("ascii")

recovered = base64.b64decode(encoded_text)
print(recovered == raw)  # True
```

Why `.decode("ascii")`? Base64 output uses ASCII characters, so this
converts the Base64 bytes into a Python string. Base64 decoding is not
decryption.

------------------------------------------------------------------------

# 3. `encode()` and `decode()`

## 3.1 `encode()` converts text to bytes

``` python
message = "Hello"
data = message.encode("utf-8")

print(data)        # b'Hello'
print(type(data))  # <class 'bytes'>
```

Model:

``` text
str --encode("utf-8")--> bytes
```

## 3.2 `decode()` converts bytes to text

``` python
data = b"Hello"
message = data.decode("utf-8")

print(message)        # Hello
print(type(message))  # <class 'str'>
```

Model:

``` text
bytes --decode("utf-8")--> str
```

Use an encoding that matches how the bytes represent text. Arbitrary
binary data is not necessarily valid UTF-8.

## 3.3 Why `.decode("ascii")` worked for your RSA key

``` python
private_key = keys.export_key().decode("ascii")
```

PyCryptodome exports a PEM-formatted key as bytes by default. PEM is
ASCII-compatible text, so decoding it as ASCII works.

These normally work for PEM output:

``` python
pem_text = keys.export_key().decode("ascii")
pem_text = keys.export_key().decode("utf-8")
pem_text = keys.export_key().decode()  # default encoding is UTF-8
```

But if you only want to save the key, you can avoid decoding:

``` python
private_key_bytes = keys.export_key()
with open("private_key.pem", "wb") as f:
    f.write(private_key_bytes)
```

## 3.4 Do not decode arbitrary ciphertext as text

Avoid assuming ciphertext is UTF-8:

``` python
# May raise UnicodeDecodeError:
ciphertext_text = ciphertext.decode("utf-8")
```

Keep ciphertext as bytes, or convert it to hex/Base64 for a text-only
transport:

``` python
hex_text = ciphertext.hex()
same_ciphertext = bytes.fromhex(hex_text)

b64_text = base64.b64encode(ciphertext).decode("ascii")
same_ciphertext = base64.b64decode(b64_text)
```

Neither hex nor Base64 encrypts anything.

## 3.5 Encoding comparison

  -----------------------------------------------------------------------
  Encoding                            Meaning
  ----------------------------------- -----------------------------------
  ASCII                               Basic English letters, digits,
                                      common symbols; 128 code points

  UTF-8                               Common Unicode encoding; 1--4 bytes
                                      per code point

  UTF-16                              Unicode encoding using 16-bit code
                                      units
  -----------------------------------------------------------------------

``` python
print("Hello".encode("ascii"))  # b'Hello'
print("Hello".encode("utf-8"))  # b'Hello'
print("Café".encode("utf-8"))   # b'Caf\xc3\xa9'
```

`"Café".encode("ascii")` raises `UnicodeEncodeError` because ASCII
cannot represent `é`.

------------------------------------------------------------------------

# 4. Encoding vs hashing vs encryption vs signatures

## Encoding

Changes representation. No secret key is required, and anyone can
reverse it.

## Hashing

Produces a digest. It is not encryption and does not have a decryption
function.

``` python
from Crypto.Hash import SHA256
print(SHA256.new(b"hello").hexdigest())
```

For security-sensitive general-purpose hashing, use modern choices such
as SHA-256, SHA-512, SHA-3 or BLAKE2. Avoid MD5 and SHA-1 where
collision resistance is needed.

## Password hashing and key derivation

Passwords are often guessable. Do not store plain passwords or use a
fast hash such as SHA-256 alone to store them. Use a password-hashing
scheme such as Argon2id, scrypt, or properly configured PBKDF2 with a
unique random salt. Prefer a well-maintained library and current
recommendations.

A password-derived encryption key is a different use from storing
password-verification hashes.

## Encryption

Protects confidentiality. Decryption requires the correct key and
parameters.

## MAC and digital signatures

-   A **MAC** (such as HMAC) checks integrity/authenticity between
    parties sharing a secret.
-   A **digital signature** uses a private signing key and public
    verification key.
-   A hash alone does not authenticate a message if an attacker can
    replace both the message and digest.

------------------------------------------------------------------------

# 5. Python `open()` and file modes

The `open()` mode controls reading, writing, creating, truncating, and
text/binary handling.

  -----------------------------------------------------------------------
  Mode              Meaning           Missing file?     Important
                                                        behavior
  ----------------- ----------------- ----------------- -----------------
  `"r"` / `"rt"`    Read text         Error             Default mode

  `"rb"`            Read bytes        Error             No automatic
                                                        decoding

  `"w"` / `"wt"`    Write text        Creates           Truncates
                                                        existing file

  `"wb"`            Write bytes       Creates           Truncates
                                                        existing file

  `"a"` / `"at"`    Append text       Creates           Writes at end

  `"ab"`            Append bytes      Creates           Writes at end

  `"x"` / `"xt"`    Create text file  Creates           Fails if file
                                                        already exists

  `"xb"`            Create binary     Creates           Fails if file
                    file                                already exists

  `"r+"`            Read/write text   Error             No initial
                                                        truncation

  `"r+b"` / `"rb+"` Read/write bytes  Error             No initial
                                                        truncation

  `"w+"`            Read/write text   Creates           Truncates
                                                        existing file

  `"w+b"` / `"wb+"` Read/write bytes  Creates           Truncates
                                                        existing file

  `"a+"`            Read/append text  Creates           Writes at end

  `"a+b"` / `"ab+"` Read/append bytes Creates           Writes at end
  -----------------------------------------------------------------------

`b` means binary, `t` means text, and `+` means reading and writing.
Text mode is the default.

## Text mode

``` python
with open("message.txt", "wt", encoding="utf-8") as f:
    f.write("Hello 🌍")

with open("message.txt", "rt", encoding="utf-8") as f:
    message = f.read()
```

Text mode converts between file bytes and strings using an encoding and
may normalize newline conventions.

## Binary mode

``` python
raw = b"\x00\xff\x10\x80"

with open("data.bin", "wb") as f:
    f.write(raw)

with open("data.bin", "rb") as f:
    recovered = f.read()

print(recovered == raw)  # True
```

Use `"rb"`/`"wb"` for ciphertext, images, audio, raw key bytes and other
binary data. Binary mode does not itself make a file secure.

### Important differences

-   `"r+b"`: file must exist; reads/writes bytes; does not truncate on
    opening.
-   `"w+b"`: creates or opens the file and truncates existing content
    immediately.
-   `"a+b"`: creates if missing; writes go to the end.

Use `with open(...)` so files close automatically. Create parent
directories before saving files:

``` python
from pathlib import Path
Path("RSA_KEYS").mkdir(parents=True, exist_ok=True)
```

------------------------------------------------------------------------

# 6. Keys, IVs, nonces, tags and padding

## Key

A cryptographic key is secret for symmetric encryption, or private for
asymmetric cryptography.

AES key sizes: - 16 bytes = 128 bits - 24 bytes = 192 bits - 32 bytes =
256 bits

``` python
from Crypto.Random import get_random_bytes
aes_key = get_random_bytes(32)
```

Use a cryptographically secure random generator. Do not use Python's
`random` module for keys, tokens, salts or nonces.

## IV and nonce

An IV (initialization vector) or nonce is a mode parameter, not a key.
It often does not need to be secret and may be stored beside ciphertext.
Requirements differ by mode. In modes such as GCM and CTR, never reuse a
nonce/counter stream with the same key.

## Authentication tag

AEAD modes such as GCM and EAX produce a tag. The receiver verifies it
to detect tampering or wrong parameters. **Do not trust decrypted
plaintext until tag verification succeeds.**

## Padding

AES has a 16-byte block size. CBC normally needs padded plaintext whose
length is a multiple of 16 bytes, often using PKCS#7 padding. GCM and
EAX handle arbitrary-length data and do not need PKCS#7 padding.

Padding does not encrypt or authenticate data.

------------------------------------------------------------------------

# 7. AES and its modes

AES is a symmetric block cipher: the same secret key is used for
encryption and decryption. AES has a 16-byte block size and supports
128-, 192-, and 256-bit keys. The mode determines how AES processes
data.

  ------------------------------------------------------------------------------------
  Mode            Authentication built     Padding normally Guidance
                                   in?              needed? 
  --------------- -------------------- -------------------- --------------------------
  ECB                               No    Yes for arbitrary Avoid for ordinary data;
                                                    lengths leaks repeated-block
                                                            patterns

  CBC                               No              Usually Legacy/interoperability
                                                            use; needs separate
                                                            authentication and a fresh
                                                            IV

  CFB                               No                   No Legacy/interoperability;
                                                            needs integrity protection

  OFB                               No                   No Legacy/interoperability;
                                                            needs integrity protection

  CTR                               No                   No Nonce/counter uniqueness
                                                            is critical; add
                                                            authentication

  GCM                              Yes                   No Strong general-purpose
                                                            choice when used correctly

  EAX                              Yes                   No Convenient
                                                            authenticated-encryption
                                                            mode in PyCryptodome

  CCM                              Yes                   No AEAD with
                                                            nonce/message-size
                                                            constraints

  OCB                              Yes                   No Check policy and
                                                            interoperability needs

  SIV                              Yes                   No Special key/nonce
                                                            behavior; consult
                                                            documentation
  ------------------------------------------------------------------------------------

For new application code, prefer authenticated encryption such as
AES-GCM or AES-EAX rather than unauthenticated modes. Never use ECB for
ordinary message/file encryption.

## GCM example

``` python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes

key = get_random_bytes(32)
plaintext = "Hello cryptography 🌍".encode("utf-8")

cipher = AES.new(key, AES.MODE_GCM)
ciphertext, tag = cipher.encrypt_and_digest(plaintext)
nonce = cipher.nonce

# The receiver must have the same key, nonce, ciphertext and tag.
decryptor = AES.new(key, AES.MODE_GCM, nonce=nonce)
candidate_plaintext = decryptor.decrypt(ciphertext)

try:
    decryptor.verify(tag)
except ValueError as exc:
    raise ValueError("Authentication failed") from exc

# Only after verification:
print(candidate_plaintext.decode("utf-8"))
```

Keep `candidate_plaintext` unused until `verify(tag)` succeeds. GCM
requires nonce uniqueness for each encryption with a given key; never
reuse a nonce with that key.

## EAX example

``` python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes

key = get_random_bytes(32)
cipher = AES.new(key, AES.MODE_EAX)
ciphertext, tag = cipher.encrypt_and_digest(b"secret message")
nonce = cipher.nonce

receiver = AES.new(key, AES.MODE_EAX, nonce=nonce)
plaintext = receiver.decrypt(ciphertext)
receiver.verify(tag)  # raises ValueError if verification fails
```

## What to store with ciphertext

Usually store/transmit a version/algorithm identifier, nonce,
ciphertext, and tag. Keep the secret key separate and protected. Define
an unambiguous format; do not concatenate fields without boundaries or
lengths.

------------------------------------------------------------------------

# 8. RSA, PEM, DER, PKCS #1 and PKCS #8

## RSA basics

RSA is asymmetric: a public key can be shared, while the private key
must be protected. RSA is slower than AES and should not encrypt large
files directly. Use RSA-OAEP to encrypt a randomly generated AES session
key, and AES to encrypt the actual data.

For new deployments, 3072-bit RSA is commonly recommended for roughly
128-bit classical security; follow current standards and your
application's security lifetime. A 2048-bit key is useful for learning
and may be accepted in existing systems, but choose sizes based on
current guidance.

## Generate and export keys

``` python
from pathlib import Path
from Crypto.PublicKey import RSA

keys_dir = Path("RSA_KEYS")
keys_dir.mkdir(parents=True, exist_ok=True)

key = RSA.generate(3072)
private_pem = key.export_key()
public_pem = key.public_key().export_key()

with open(keys_dir / "private_key.pem", "wb") as f:
    f.write(private_pem)

with open(keys_dir / "public_key.pem", "wb") as f:
    f.write(public_pem)
```

Protect private-key files and backups. Never commit private keys or
passwords to Git or print them in logs.

## Import keys

``` python
from Crypto.PublicKey import RSA

with open("RSA_KEYS/private_key.pem", "rb") as f:
    private_key = RSA.import_key(f.read())

with open("RSA_KEYS/public_key.pem", "rb") as f:
    public_key = RSA.import_key(f.read())

print(private_key.has_private())  # True
print(public_key.has_private())   # False
```

## PEM versus DER

-   **PEM** is a text representation with header/footer lines and
    Base64-like content.
-   **DER** is a binary encoding of the underlying ASN.1 structure.
-   PEM does not automatically mean the key is encrypted.

Typical labels:

``` text
-----BEGIN RSA PRIVATE KEY-----
```

Traditional RSA-specific private-key structure.

``` text
-----BEGIN PRIVATE KEY-----
```

Typically an unencrypted PKCS #8 private key.

``` text
-----BEGIN ENCRYPTED PRIVATE KEY-----
```

An encrypted PKCS #8 private key.

## PKCS #1 versus PKCS #8

-   **PKCS #1** describes RSA-specific structures.
-   **PKCS #8** is a general container for private keys from algorithms
    such as RSA and ECC.
-   `pkcs=8` selects the PKCS #8 private-key format in PyCryptodome.
-   `passphrase` provides the passphrase for encryption.
-   `protection` selects the password-based protection scheme.

## Password-protect a private key

``` python
from Crypto.PublicKey import RSA
from getpass import getpass

key = RSA.generate(3072)
passphrase = getpass("Private-key passphrase: ")

encrypted_pem = key.export_key(
    format="PEM",
    passphrase=passphrase,
    pkcs=8,
    protection="PBKDF2WithHMAC-SHA512AndAES256-CBC",
    prot_params={"iteration_count": 131072},
)

with open("private_key_encrypted.pem", "wb") as f:
    f.write(encrypted_pem)
```

Import it later:

``` python
from Crypto.PublicKey import RSA
from getpass import getpass

passphrase = getpass("Private-key passphrase: ")
with open("private_key_encrypted.pem", "rb") as f:
    data = f.read()

private_key = RSA.import_key(data, passphrase=passphrase)
print(private_key.has_private())
```

The iteration count is an example, not a universal recommendation.
Choose parameters using current library guidance and your
performance/threat requirements. Avoid hardcoding passphrases.

## RSA-OAEP encryption

``` python
from Crypto.Cipher import PKCS1_OAEP
from Crypto.Hash import SHA256
from Crypto.PublicKey import RSA

key = RSA.generate(3072)
public_key = key.public_key()

cipher = PKCS1_OAEP.new(public_key, hashAlgo=SHA256)
encrypted = cipher.encrypt(b"short secret")

decryptor = PKCS1_OAEP.new(key, hashAlgo=SHA256)
decrypted = decryptor.decrypt(encrypted)
print(decrypted)
```

RSA-OAEP can only encrypt a short message limited by the RSA modulus and
padding. Use it for a session key, not an arbitrary large file.

## Digital signatures are different from encryption

Use a signature scheme such as RSA-PSS to sign data; verify with the
public key.

``` python
from Crypto.Hash import SHA256
from Crypto.PublicKey import RSA
from Crypto.Signature import pss

key = RSA.generate(3072)
message = b"Document that needs a signature"

signature = pss.new(key).sign(SHA256.new(message))
pss.new(key.public_key()).verify(SHA256.new(message), signature)
print("Signature is valid")
```

Changing the message should cause verification to fail. Signatures do
not hide the message.

------------------------------------------------------------------------

# 9. Hybrid encryption: RSA + AES

A common design for large messages/files:

1.  Generate a random AES session key.
2.  Encrypt the data using AES-GCM.
3.  Encrypt the AES key using the recipient's RSA public key with OAEP.
4.  Send/store the encrypted AES key, nonce, ciphertext and tag.
5.  The recipient uses their RSA private key to recover the AES key.
6.  The recipient decrypts and verifies the data with AES-GCM.

``` text
data --AES-GCM + random AES key--> ciphertext + nonce + tag
AES key --RSA-OAEP with recipient public key--> encrypted AES key

Recipient:
encrypted AES key --RSA private key--> AES key
ciphertext + nonce + tag + AES key --AES-GCM verification--> plaintext
```

Example:

``` python
from Crypto.Cipher import AES, PKCS1_OAEP
from Crypto.Hash import SHA256
from Crypto.PublicKey import RSA
from Crypto.Random import get_random_bytes

# Demonstration only: generate a recipient key pair.
recipient_private = RSA.generate(3072)
recipient_public = recipient_private.public_key()
message = "A large message should be encrypted with AES.".encode("utf-8")

session_key = get_random_bytes(32)

aes = AES.new(session_key, AES.MODE_GCM)
ciphertext, tag = aes.encrypt_and_digest(message)
nonce = aes.nonce

rsa_encryptor = PKCS1_OAEP.new(recipient_public, hashAlgo=SHA256)
encrypted_session_key = rsa_encryptor.encrypt(session_key)

# Recipient side:
rsa_decryptor = PKCS1_OAEP.new(recipient_private, hashAlgo=SHA256)
recovered_key = rsa_decryptor.decrypt(encrypted_session_key)

aes_decryptor = AES.new(recovered_key, AES.MODE_GCM, nonce=nonce)
candidate = aes_decryptor.decrypt(ciphertext)
aes_decryptor.verify(tag)  # Never ignore failure.

print(candidate.decode("utf-8"))  # Only after successful verification.
```

For production systems, use reviewed protocols/formats where possible.
Authenticate the recipient public key, define a versioned data format,
protect private keys, and handle errors carefully.

------------------------------------------------------------------------

# 10. Password-derived keys

Do not use a human password directly as an AES key. Use a password-based
key derivation function with a random salt and appropriate work factors.

Example API demonstration with scrypt:

``` python
from Crypto.Protocol.KDF import scrypt
from Crypto.Random import get_random_bytes

password = b"example passphrase"
salt = get_random_bytes(16)

key = scrypt(password, salt, key_len=32, N=2**14, r=8, p=1)
```

This is an example configuration, not a universal production
recommendation. Select parameters using current guidance and test the
acceptable time/memory cost. To derive the same key later, retain the
password, salt and parameters. The salt normally need not be secret.

Different terms: - Password/passphrase: human-provided secret. - Salt:
random public value used in derivation. - Derived key: fixed-length
output from the KDF. - AES key: secret supplied to AES. - Nonce/IV: mode
parameter, not a key.

------------------------------------------------------------------------

# 11. Common errors

## `TypeError: a bytes-like object is required, not 'str'`

A function expects bytes but received text.

``` python
# Wrong for socket APIs expecting bytes:
# sock.send("GET / HTTP/1.0\r\n\r\n")

# Correct:
# sock.sendall("GET / HTTP/1.0\r\n\r\n".encode("ascii"))
```

## `UnicodeDecodeError`

You tried to interpret bytes as text using an incompatible encoding.
Keep arbitrary ciphertext as bytes, or represent it with hex/Base64.

## `UnicodeEncodeError`

The text contains characters not supported by the chosen encoding. ASCII
cannot encode `é`; UTF-8 can.

## Invalid AES key length

AES keys for normal modes must be 16, 24, or 32 bytes. A string's
character count is not necessarily its UTF-8 byte length. Do not solve
this by blindly truncating a password.

## Authentication failure / `MAC check failed` / `ValueError`

Possible causes: wrong key, nonce, ciphertext, tag, or modified data. Do
not ignore the error and do not use candidate plaintext after failed
authentication.

## `FileNotFoundError`

A file or parent directory is missing. Create parent directories with
`Path(...).mkdir(parents=True, exist_ok=True)`.

## Key export errors

Keep the complete traceback. Test imports and key export separately.
Avoid changing multiple packages and code paths simultaneously.

------------------------------------------------------------------------

# 12. Best-practice checklist

### Data

-   Know whether each value is `str`, `bytes`, `bytearray`, a key
    object, digest or ciphertext.
-   Encode text at clear boundaries, commonly as UTF-8.
-   Decode only bytes known to represent text in the selected encoding.
-   Do not mistake hex or Base64 for encryption.

### Files

-   Use `"rb"`/`"wb"` for raw binary data.
-   Use text mode with an explicit encoding for text.
-   Use `with open(...)`.
-   Remember `"w"` truncates an existing file.
-   Protect private keys, passwords, logs and backups.
-   Keep secrets out of source control.

### Cryptography

-   Prefer AES-GCM or AES-EAX for new symmetric encryption.
-   Avoid ECB for ordinary data.
-   Never reuse a nonce with the same key where uniqueness is required.
-   Verify authentication before trusting plaintext.
-   Use secure randomness for keys and nonces.
-   Do not use a password directly as an AES key.
-   Use RSA-OAEP for RSA encryption and RSA-PSS for signatures.
-   Use hybrid encryption for large data.
-   Follow current library guidance and use reviewed protocols.

### Conceptual reminders

-   Encoding changes representation; it does not provide secrecy.
-   Hashing is not encryption and cannot be reversed to recover the
    original input.
-   Encryption provides confidentiality; authentication detects
    tampering.
-   A public key can be shared, but verify that it belongs to the
    intended recipient.
-   PEM does not automatically mean encrypted.
-   PKCS #8 is a private-key container format; `protection` describes
    the password-based protection scheme.
-   `"rb"` = read bytes; `"wb"` = write bytes; `"r+b"` = read/write
    bytes without initial truncation.

------------------------------------------------------------------------

# 13. Exercises

1.  Create `"Hello"` and print its type.
2.  Encode it as UTF-8 and inspect the resulting bytes.
3.  Decode the bytes back to text.
4.  Convert `b"ABC"` to hex and back.
5.  Convert `b"ABC"` to Base64 and back.
6.  Write `b"\x00\xff\x10"` to a file using `"wb"` and read it using
    `"rb"`.
7.  Explain `"w+b"` versus `"r+b"`.
8.  Generate a 32-byte random key. Why is that 256 bits?
9.  Encrypt/decrypt with AES-GCM and verify the tag.
10. Change one ciphertext byte and observe verification failure.
11. Export an RSA private key using encrypted PKCS #8.
12. Explain why RSA-OAEP should encrypt a session key rather than a
    large file.
13. Explain the difference between `.decode("ascii")`,
    `base64.b64decode()`, and AES decryption.
14. Explain why a nonce may be stored with ciphertext while the key must
    be protected.
15. Explain why a hash digest cannot reconstruct the original message.

------------------------------------------------------------------------

# 14. Official references

-   Python file input/output and modes:
    https://docs.python.org/3/tutorial/inputoutput.html
-   Python built-in `open()`:
    https://docs.python.org/3/library/functions.html#open
-   PyCryptodome introduction:
    https://www.pycryptodome.org/src/introduction
-   AES modes and parameters:
    https://www.pycryptodome.org/src/cipher/aes
-   RSA, OAEP, key export:
    https://www.pycryptodome.org/src/public_key/rsa
-   PKCS #8: https://www.pycryptodome.org/src/io/pkcs8
-   PyCryptodome examples including hybrid encryption:
    https://www.pycryptodome.org/src/examples
-   PyCryptodome features: https://www.pycryptodome.org/src/features

------------------------------------------------------------------------

## Final mental model: ask these questions every time

1.  **What do I have?** Text, bytes, a key, a digest, or ciphertext?
2.  **Which representation do I need?** UTF-8, raw bytes, hex, Base64,
    PEM or DER?
3.  **Which operation is intended?** Encoding, hashing, encryption,
    decryption, signing or verification?
4.  **Which parameters are required?** Key, nonce/IV, tag, salt,
    passphrase or padding?
5.  **How should I store it?** Text mode or binary mode?
6.  **How do I validate it?** Verify authentication before using
    plaintext.
7.  **Where are the secrets?** Keep keys and passphrases out of source
    code, logs and unprotected storage.

Answering these questions makes cryptographic code much easier to
understand and debug.
