# Linux Users, FTP, and File Transfer Between PCs

Practical guide for Ubuntu/Linux, Windows, and WSL.

## 1. Understand Linux users and groups

-   **User**: an account that can log in and run processes.
-   **UID**: numeric user ID.
-   **Group**: a collection of users used to manage shared access.
-   **GID**: numeric group ID.
-   **Home directory**: usually `/home/username`.
-   **Root**: administrator account, UID `0`.

``` bash
whoami                 # Current effective username
id                     # UID, primary GID, supplementary groups
id john                # Inspect john's IDs and groups
groups                 # Your groups
groups john            # Groups belonging to john
who                    # Users currently logged in
w                      # Logged-in users and activity
users                  # Usernames currently logged in
hostname               # Computer name
hostname -I            # Local IP addresses on Linux
```

## 2. Create, inspect, modify, and remove users

``` bash
getent passwd john             # Look up account john
getent group developers        # Inspect a group
sudo adduser john              # Create a user interactively on Ubuntu
sudo passwd john               # Set/reset john's password
passwd                         # Change your own password
sudo usermod -aG developers john  # Add john to group; -a preserves other groups
sudo usermod -s /bin/bash john    # Set login shell
sudo chage -l john                # Show password-aging information
sudo deluser john                 # Remove account, normally leaving home
sudo deluser --remove-home john   # Remove account and home directory
```

**Important:** Use `usermod -aG group user` when adding a supplementary
group. Omitting `-a` can replace the user's existing supplementary
groups. New group membership usually takes effect in a new login
session.

`getent passwd` entries look approximately like:

``` text
john:x:1001:1001:John Example:/home/john:/bin/bash
```

Fields are username, password placeholder, UID, primary GID,
comment/name, home directory, and login shell. Password hashes are
normally stored in `/etc/shadow`.

## 3. Group commands and shared folders

``` bash
sudo groupadd developers            # Create a group
sudo gpasswd -a john developers     # Add john to group
sudo gpasswd -d john developers     # Remove john from group
getent group developers             # Inspect group
sudo chgrp developers report.txt    # Change file's group
sudo chmod 664 report.txt           # Owner/group read-write; others read
```

Create a collaborative directory:

``` bash
sudo mkdir -p /srv/shared
sudo chgrp developers /srv/shared
sudo chmod 2770 /srv/shared
```

`2770` means SGID plus owner/group read-write-execute, and no
permissions for others. SGID on a directory makes new entries generally
inherit the directory's group, but does not guarantee every new file is
group-writable.

Optional default ACLs:

``` bash
sudo setfacl -m g:developers:rwx /srv/shared
sudo setfacl -d -m g:developers:rwx /srv/shared
getfacl /srv/shared
```

## 4. Communicate with logged-in Linux users

These commands are most useful on multi-user Linux systems; they may be
limited or less meaningful on desktops, containers, and WSL.

``` bash
who                      # See logged-in sessions
w                        # Sessions and activity
write john               # Message a logged-in user, if permitted
mesg                     # Check terminal message permission
mesg y                   # Allow terminal messages
mesg n                   # Disallow terminal messages
wall "Maintenance soon"  # Broadcast message to logged-in terminals
last                     # Recent login history
loginctl list-sessions   # Sessions on systems using systemd-logind
```

With `write john`, type your message and press **Ctrl+D** to finish.
`wall` is a terminal broadcast, not an internet chat tool.

## 5. Choose a file-transfer method

  -----------------------------------------------------------------------
  Method                  Best use                Security / requirements
  ----------------------- ----------------------- -----------------------
  SCP                     Copy files over a       Encrypted SSH;
                          network                 destination needs SSH
                                                  server

  SFTP                    Interactive file        Encrypted SSH;
                          transfer                destination needs SSH
                                                  server

  rsync over SSH          Repeated folder sync    Encrypted SSH; both
                          and backups             ends need rsync/SSH

  SMB/Samba               Windows/Linux network   Configure credentials
                          shares                  and firewall carefully

  FTP                     Legacy systems or       Classic FTP is
                          isolated labs           unencrypted

  FTPS                    FTP secured with TLS    Requires correct TLS
                                                  configuration

  HTTP/HTTPS              Download from a web     HTTPS encrypts
                          server                  transport

  USB drive               Offline transfer        Physical access;
                                                  encrypt sensitive data

  Cloud storage           PCs on different        Account and sharing
                          networks                settings matter
  -----------------------------------------------------------------------

**Recommendation:** For command-line Linux transfers, use **SFTP, SCP,
or rsync over SSH** instead of plain FTP. Classic FTP can expose
usernames, passwords, and data to network observers. SFTP is a separate
protocol over SSH; it is not FTP with encryption.

## 6. Find the receiving computer's IP address

On Linux:

``` bash
hostname -I             # Show local IP addresses
ip addr                 # Inspect network interfaces
ip route                # Inspect routing and default gateway
ping -c 4 192.168.1.25  # Test reachability where ping is allowed
```

On Windows PowerShell:

``` powershell
ipconfig
ping 192.168.1.25
```

Replace `192.168.1.25` with the receiving PC's actual reachable IP. Both
PCs commonly need to be on the same LAN, or connected through a valid
route/VPN. A failed ping does not prove the host is offline; firewalls
may block ICMP.

## 7. Recommended method: SSH + SCP

### Set up the receiving Ubuntu PC

Run on the **receiving PC**:

``` bash
sudo apt update
sudo apt install openssh-server
sudo systemctl enable --now ssh
sudo systemctl status ssh
```

If UFW is enabled, allow SSH only as appropriate for your network:

``` bash
sudo ufw status
sudo ufw allow OpenSSH
```

### Copy a file from the sending PC

Run on the **sending PC**:

``` bash
scp ./report.pdf john@192.168.1.25:/home/john/
```

Explanation: - `scp`: secure copy using SSH. - `./report.pdf`: local
file to send. - `john@192.168.1.25`: destination username and IP. -
`:/home/john/`: destination directory.

Download a file from the receiving PC:

``` bash
scp john@192.168.1.25:/home/john/report.pdf .
```

Copy a directory:

``` bash
scp -r ./project john@192.168.1.25:/home/john/
```

Use a non-default SSH port:

``` bash
scp -P 2222 ./report.pdf john@192.168.1.25:/home/john/
```

`scp` uses uppercase `-P` for the port. The `ssh` command uses lowercase
`-p`.

## 8. Transfer interactively with SFTP

Connect:

``` bash
sftp john@192.168.1.25
```

Common commands at the `sftp>` prompt:

``` text
pwd                  # Remote current directory
lpwd                 # Local current directory
ls                   # List remote files
lls                  # List local files
cd /home/john        # Change remote directory
lcd ~/Documents      # Change local directory
put report.pdf       # Upload a file
get report.pdf       # Download a file
put -r project       # Upload a directory recursively
get -r project       # Download a directory recursively
mkdir uploads        # Create remote directory
exit                 # End session
```

## 9. SSH keys (optional, recommended for regular use)

On the sending PC:

``` bash
ssh-keygen -t ed25519              # Create a key pair
ssh-copy-id john@192.168.1.25      # Install public key, where supported
ssh john@192.168.1.25              # Test login
```

Keep `~/.ssh/id_ed25519` (the private key) secret. Only the `.pub`
public key should be installed on the destination. Use a passphrase
where practical. Do not send your private key to another computer.

## 10. Synchronize folders with rsync over SSH

``` bash
sudo apt install rsync
rsync -avh --progress ./project/ john@192.168.1.25:/home/john/project/
```

Options: - `-a`: archive mode; recursive copying and preservation of
common metadata. - `-v`: verbose output. - `-h`: human-readable sizes. -
`--progress`: show transfer progress. - `-n` / `--dry-run`: preview
without copying. - `--delete`: remove destination files absent from
source; use with great care.

Preview before deletion:

``` bash
rsync -avhn --delete ./project/ john@192.168.1.25:/home/john/project/
```

Check the preview before removing `-n`. A trailing slash matters:
`./project/` means copy the directory's contents; `./project` generally
copies the directory itself.

## 11. FTP: how it works and basic commands

FTP means **File Transfer Protocol**. An FTP server accepts connections;
an FTP client connects to it and uploads/downloads files.

Install an FTP client on Ubuntu:

``` bash
sudo apt update
sudo apt install ftp
```

Connect to a server:

``` bash
ftp 192.168.1.25
```

Common FTP prompt commands (support varies by client):

``` text
ls             # List remote files
pwd            # Remote working directory
cd uploads     # Change remote directory
get report.pdf # Download a file
put report.pdf # Upload a file
binary         # Binary transfer mode
bye            # Disconnect
```

Installing the client does **not** install an FTP server. Classic FTP is
unencrypted. Do not use it for passwords or sensitive files on untrusted
networks. Prefer SFTP or properly configured FTPS. If a class requires
FTP, use an isolated lab network, test files, a non-sensitive account,
and restrictive firewall rules. Avoid anonymous write access and do not
expose plain FTP to the internet.

## 12. Windows-to-Linux and Linux-to-Windows

If the receiving Linux PC has SSH enabled, Windows PowerShell can use
SCP:

``` powershell
scp "C:\Users\YourName\Documents\report.pdf" john@192.168.1.25:/home/john/
```

Download to Windows:

``` powershell
scp john@192.168.1.25:/home/john/report.pdf "C:\Users\YourName\Downloads\"
```

Windows 10/11 may have OpenSSH client installed. Check with:

``` powershell
ssh -V
scp
```

### SMB / Windows network sharing

SMB is commonly used for Windows shared folders. On a trusted LAN: 1.
Share a specific folder. 2. Grant access only to intended users. 3.
Connect from another PC with a path like `\\192.168.1.25\Shared`. 4.
Authenticate using an authorized account.

Do not expose SMB (especially TCP port 445) directly to the public
internet. Use a VPN for remote access rather than publishing SMB ports.

Ubuntu client tools:

``` bash
sudo apt install smbclient
smbclient -L //192.168.1.25 -U your_windows_username
smbclient //192.168.1.25/Shared -U your_windows_username
```

Inside `smbclient`, commands commonly include `ls`, `cd`, `get file`,
`put file`, and `exit`. Share configuration and permissions must allow
access.

### WSL note

WSL's Windows drives are commonly mounted under `/mnt`. This copies
between Windows and Linux **on the same PC**, not to another computer:

``` bash
cp /mnt/c/Users/YourName/Downloads/report.pdf ~/report.pdf
```

## 13. Quick temporary HTTP transfer (trusted LAN only)

Python can temporarily serve files for downloading:

``` bash
cd ~/share
python3 -m http.server 8000 --bind 0.0.0.0
```

On the other PC, open `http://SENDER_IP:8000/` in a browser. Stop the
server with **Ctrl+C** when finished.

**Safety:** This simple server has no authentication and no HTTPS.
Anyone who can reach it may be able to download files in the served
directory. Serve only a dedicated folder on a trusted LAN. Do not expose
it to the internet or use it for private files.

## 14. USB and cloud transfer

**USB drive** 1. Copy the file to the drive. 2. Safely eject it. 3.
Connect it to the other PC. 4. Copy the file and safely eject again. 5.
Encrypt the drive if it will hold sensitive data.

**Cloud storage** 1. Upload from the first PC. 2. Download from the
second PC. 3. Check sharing permissions; avoid public links unless
intended. 4. Use a strong unique password and multi-factor
authentication.

## 15. Verify a transfer

Generate a SHA-256 checksum on the sending PC:

``` bash
sha256sum report.pdf
```

Run the same command on the receiver:

``` bash
sha256sum report.pdf
```

If the hashes match, the file contents match with extremely high
confidence.

## 16. Troubleshooting

### Connection refused

The SSH server may not be running, or the IP/port may be wrong. On the
receiver:

``` bash
sudo systemctl status ssh
sudo ss -lntp | grep ':22'
```

### Connection timed out

Check the destination IP, routes, firewall, network isolation, and
whether the computers can reach each other. Ping alone is not
conclusive.

### Permission denied

Check the username, authentication method, destination directory
permissions, and ACLs:

``` bash
id john
ls -ld /home/john
namei -l /home/john
```

Check firewall status:

``` bash
sudo ufw status verbose
```

Do not disable the firewall entirely as a shortcut.

## 17. Safe practice lab

Use two Linux PCs or virtual machines on a private lab network.

1.  On the receiving PC, install and start SSH:

    ``` bash
    sudo apt update
    sudo apt install openssh-server
    sudo systemctl enable --now ssh
    ```

2.  Find its IP:

    ``` bash
    hostname -I
    ```

3.  On the sending PC, create a test file:

    ``` bash
    printf 'Hello from the sending PC\\n' > hello.txt
    ```

4.  Copy it:

    ``` bash
    scp hello.txt john@192.168.1.25:/home/john/
    ```

5.  On the receiver, verify:

    ``` bash
    cat /home/john/hello.txt
    sha256sum /home/john/hello.txt
    ```

6.  Try `sftp john@192.168.1.25`.

7.  Try `rsync` with `--dry-run` before attempting any operation
    involving `--delete`.

Replace the sample username and IP with the destination's actual values.

## 18. Quick reference

``` bash
# Users
whoami
id
who
w
sudo adduser john
sudo passwd john
sudo usermod -aG developers john
sudo deluser john

# Groups
sudo groupadd developers
sudo gpasswd -a john developers
getent group developers

# Terminal messages
write john
mesg y
wall "Maintenance starts soon"

# Network diagnostics
hostname -I
ip addr
ip route
ping -c 4 192.168.1.25
sudo ss -lntp

# SSH server on receiving Ubuntu PC
sudo apt install openssh-server
sudo systemctl enable --now ssh
sudo systemctl status ssh
sudo ufw allow OpenSSH

# Transfers
scp report.pdf john@192.168.1.25:/home/john/
scp -r project john@192.168.1.25:/home/john/
sftp john@192.168.1.25
rsync -avh --progress project/ john@192.168.1.25:/home/john/project/

# Verify
sha256sum report.pdf
```

## Final recommendations

-   Learn SSH, SCP, SFTP, and rsync first for Linux-to-Linux transfers.
-   Use SMB for carefully configured Windows network shares.
-   Treat classic FTP as unencrypted legacy technology; prefer SFTP or
    properly configured FTPS.
-   Confirm destination paths and permissions before copying.
-   Back up data before deleting accounts, home directories, or
    destination files.
