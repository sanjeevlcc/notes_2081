# Unit 4: Network Services and File Sharing

**Teaching hours:** 5 Theory + 5 Practical

## 1. Quick Notes

### 1.1 DNS (Domain Name System)
DNS translates domain names (e.g., `example.com`) into IP addresses.
- **DNS server:** Answers name-resolution queries.
- **A record:** Maps a hostname to an IPv4 address.
- **AAAA record:** Maps a hostname to an IPv6 address.
- **PTR record:** Reverse lookup (IP to name).
- **CNAME:** Alias for another hostname.


### 1.2 DHCP (Dynamic Host Configuration Protocol)
DHCP automatically assigns network settings to clients, including IP address, subnet mask, gateway, and DNS server.
- **DORA:** Discover → Offer → Request → Acknowledge.
- **Lease:** IP address assigned for a limited period.
- Prevents manually configuring each client.

### 1.3 Hostname Resolution and DNS Queries
Hostname resolution finds the IP address associated with a hostname.
Useful commands:
```bash
hostname
hostnamectl
getent hosts example.com
nslookup example.com
 dig example.com
 dig -x 8.8.8.8
```

### 1.4 FTP
FTP transfers files between a client and server.
- Uses separate control and data connections.
- Traditional FTP does not encrypt credentials or data; avoid it on untrusted networks.
- Prefer secure alternatives such as SFTP or FTPS when security matters.

### 1.5 NFS
NFS (Network File System) allows Linux/Unix systems to share directories over a network.
- Server exports directories.
- Clients mount exported directories.
- Commonly used for Linux-to-Linux file sharing.

### 1.6 Samba
Samba provides SMB/CIFS file sharing, commonly allowing Linux shares to be accessed by Windows and other SMB clients.
- Configure a share in `/etc/samba/smb.conf`.
- Control access with Linux permissions and Samba settings.

## 2. Useful Commands

| Task | Command |
|---|---|
| Check hostname | `hostnamectl` |
| Query DNS | `dig example.com` |
| Simple DNS lookup | `nslookup example.com` |
| Check name resolution | `getent hosts server1` |
| Restart DNS service (BIND) | `sudo systemctl restart bind9` |
| Check service status | `sudo systemctl status bind9` |
| Renew DHCP lease (client) | `sudo dhclient -r && sudo dhclient` |
| Check IP configuration | `ip addr` |
| Check routes | `ip route` |
| Check listening ports | `sudo ss -tulpn` |
| Check NFS exports | `sudo exportfs -v` |
| Mount NFS share | `sudo mount -t nfs server:/share /mnt` |
| Check Samba configuration | `testparm` |
| List Samba shares | `smbclient -L //server -U username` |

## 3. Practical Lab Scenarios

### Lab 1: Configure DNS Server (BIND9)
**Goal:** Configure a DNS server and resolve a local hostname.

**On the server (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install bind9 bind9-utils dnsutils
```
Edit DNS configuration and zone files (example domain: `lab.local`) under `/etc/bind/`, then validate and restart:
```bash
sudo named-checkconf
sudo named-checkzone lab.local /etc/bind/db.lab.local
sudo systemctl restart bind9
sudo systemctl status bind9
```
**Test from a client:**
```bash
dig @SERVER_IP lab.local
nslookup lab.local
```
**Expected result:** The DNS query returns the configured IP address for the domain/host.

### Lab 2: Set Up DHCP Server
**Goal:** Automatically provide IP settings to clients on a lab network.

**Install:**
```bash
sudo apt update
sudo apt install isc-dhcp-server
```
Edit `/etc/dhcp/dhcpd.conf` to define the lab subnet, address pool, gateway, and DNS server. Example configuration:
```conf
subnet 192.168.10.0 netmask 255.255.255.0 {
  range 192.168.10.100 192.168.10.150;
  option routers 192.168.10.1;
  option domain-name-servers 192.168.10.1, 8.8.8.8;
}
```
Restart and check:
```bash
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
```
**On a client:** Renew its lease, then check `ip addr` and `ip route`.
**Expected result:** Client receives an address from the pool, along with gateway and DNS settings.

> Use a dedicated lab network. DHCP servers should not be started on a network where you are not authorized to provide addresses.

### Lab 3: Configure Samba File Share
**Goal:** Share a directory with network users (including Windows clients).

**Install and create a share directory:**
```bash
sudo apt update
sudo apt install samba
sudo mkdir -p /srv/shared
sudo chmod 2770 /srv/shared
sudo chown root:users /srv/shared
```
Add to `/etc/samba/smb.conf`:
```conf
[Shared]
   path = /srv/shared
   browseable = yes
   read only = no
   valid users = @users
```
Create or add a user to the Linux `users` group as needed. Set a Samba password for an existing Linux user:
```bash
sudo smbpasswd -a username
sudo testparm
sudo systemctl restart smbd
```
**Client test:** Open `\\SERVER_IP\Shared` from Windows or connect using `smbclient`.
**Expected result:** Authorized users can access the share according to filesystem and Samba permissions.

### Lab 4: Configure NFS Share
**Goal:** Share a Linux directory with another Linux client.

**On the server:**
```bash
sudo apt update
sudo apt install nfs-kernel-server
sudo mkdir -p /srv/nfs/shared
sudo chown nobody:nogroup /srv/nfs/shared
sudo chmod 755 /srv/nfs/shared
```
Add an export in `/etc/exports` (replace the example client IP with the lab client's IP):
```conf
/srv/nfs/shared 192.168.10.20(rw,sync,no_subtree_check)
```
Apply and verify:
```bash
sudo exportfs -ra
sudo exportfs -v
sudo systemctl restart nfs-server
```
**On the client:**
```bash
sudo apt install nfs-common
sudo mkdir -p /mnt/nfs
sudo mount -t nfs SERVER_IP:/srv/nfs/shared /mnt/nfs
mount | grep nfs
```
**Expected result:** The client can access the exported directory, subject to export rules and permissions.

### Lab 5: Hostname and DNS Troubleshooting
**Scenario:** A client cannot access a server by its hostname.

**Check step-by-step:**
```bash
hostname
ip addr
ip route
cat /etc/resolv.conf
getent hosts server1.lab.local
dig server1.lab.local
ping SERVER_IP
ping server1.lab.local
```
- If IP ping works but hostname resolution fails, check DNS settings and records.
- If neither works, check IP configuration, routes, firewall, and physical/virtual network connection.
- Check server service status and logs if DNS is running but queries fail.

## 4. Quick Revision

- **DNS:** Resolves names to IP addresses (and supports reverse lookup).
- **DHCP:** Automatically assigns IP configuration to clients.
- **FTP:** Transfers files; traditional FTP is unencrypted.
- **NFS:** Network file sharing, especially for Linux/Unix.
- **Samba:** SMB file sharing, commonly between Linux and Windows.
- **DORA:** DHCP Discover, Offer, Request, Acknowledge.
- **Security:** Use least privilege, restrict share/export access, use strong credentials, and prefer encrypted transfer protocols.
