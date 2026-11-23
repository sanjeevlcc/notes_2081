# Unit 3: Disk Quotas, Storage Management & Network Configuration

**Teaching hours:** 5 Theory + 5 Practical

## 1. Quick Notes

### 1.1 Disk quotas
Disk quotas limit how much disk space or how many files (inodes) a user/group can use.
- **Soft limit:** Can be exceeded temporarily during a grace period.
- **Hard limit:** Cannot be exceeded.
- **Block quota:** Limits disk space.
- **Inode quota:** Limits number of files.

Useful commands (Ubuntu/Debian; filesystem must support quotas):
```bash
sudo apt install quota
sudo mount | grep ' /home '
# Add usrquota,grpquota to the relevant filesystem's mount options in /etc/fstab
sudo mount -o remount /home
sudo quotacheck -cugm /home
sudo quotaon -vug /home
sudo edquota -u student
sudo quota -u student
sudo repquota -a
```

### 1.2 RAID
RAID combines disks for redundancy, performance, or both.
- **RAID 0:** Striping; fast, no redundancy.
- **RAID 1:** Mirroring; copies data to both disks; survives one disk failure in a 2-disk mirror.
- **RAID 5:** Striping with distributed parity; requires 3+ disks.
- **RAID 6:** Dual parity; requires 4+ disks.
- **RAID 10:** Mirroring + striping; requires 4+ disks.

RAID is not a backup. It protects against some disk failures, not accidental deletion or other disasters.

### 1.3 Logical Volume Management (LVM)
LVM allows flexible storage allocation and resizing.
- **PV (Physical Volume):** Disk/partition initialized for LVM.
- **VG (Volume Group):** Pool of PV storage.
- **LV (Logical Volume):** Virtual storage volume created from a VG.

Basic commands:
```bash
sudo pvcreate /dev/sdb1
sudo vgcreate vgdata /dev/sdb1
sudo lvcreate -L 5G -n lvfiles vgdata
sudo mkfs.ext4 /dev/vgdata/lvfiles
sudo mkdir -p /mnt/lvfiles
sudo mount /dev/vgdata/lvfiles /mnt/lvfiles
sudo pvs; sudo vgs; sudo lvs
```

### 1.4 Disk and inode limits
- **Disk space:** Capacity used by file contents and metadata.
- **Inode:** Filesystem record for a file; tracks metadata and pointers to data.
- A filesystem can run out of inodes even if free space remains.
- Check usage: `df -h` (space), `df -i` (inodes).

### 1.5 IPv4 and IPv6
- **IPv4:** 32-bit address, e.g. `192.168.1.10/24`.
- **IPv6:** 128-bit address, e.g. `2001:db8:1::10/64`.
- **Static IP:** Manually configured and stays fixed unless changed.
- **Dynamic IP:** Usually assigned automatically by DHCP.
- **Default gateway:** Router used to reach other networks.
- **DNS:** Resolves domain names to IP addresses.

Common Linux network commands:
```bash
ip addr                 # show addresses
ip route                # show routes
ping -c 4 8.8.8.8         # test IP connectivity
ping -c 4 example.com    # test DNS + connectivity
resolvectl status        # DNS status (systemd systems)	raceroute example.com   # trace path (install if needed)
```

## 2. Practical Lab Scenarios

> **Safety:** Identify the correct disks first. RAID/LVM setup can erase data. Do not use your system/boot disk. Use a VM with extra blank disks or a lab environment.

### Lab 1 — User disk quota
**Scenario:** Limit user `student` to 500 MB and 1000 files on `/home`.

1. Install quota tools: `sudo apt install quota`.
2. Configure `/home` with `usrquota` in `/etc/fstab` (for example, in its options column), then remount it.
3. Initialize and enable quota:
   ```bash
   sudo quotacheck -cugm /home
   sudo quotaon -vug /home
   ```
4. Edit quota for the user: `sudo edquota -u student`.
5. Set the block soft/hard limits to about **500 MB** (use the editor's KB units) and inode limits to **800/1000** (soft/hard).
6. Verify: `sudo quota -u student`; system-wide: `sudo repquota -a`.

**Expected result:** User receives warnings at the soft limit and cannot exceed the hard limit.

### Lab 2 — RAID 1 mirror with mdadm
**Scenario:** Create a mirrored array using two spare disks (example `/dev/sdb` and `/dev/sdc`).

1. Install: `sudo apt install mdadm`.
2. Confirm disks are blank and correct: `lsblk`.
3. Create mirror (replace example disk names with your spare disks):
   ```bash
   sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
   ```
4. Monitor: `cat /proc/mdstat`; details: `sudo mdadm --detail /dev/md0`.
5. Create filesystem and mount:
   ```bash
   sudo mkfs.ext4 /dev/md0
   sudo mkdir -p /mnt/raid1
   sudo mount /dev/md0 /mnt/raid1
   df -h /mnt/raid1
   ```
6. Optional lab demonstration: simulate a failed member **only in a disposable VM**, then observe array status and recovery.

**Expected result:** RAID 1 array is active; usable capacity is approximately one disk's capacity.

### Lab 3 — Create an LVM logical volume
**Scenario:** Use a spare partition `/dev/sdb1` to create a 2 GB logical volume.

```bash
sudo pvcreate /dev/sdb1
sudo vgcreate vg_lab /dev/sdb1
sudo lvcreate -L 2G -n lv_data vg_lab
sudo mkfs.ext4 /dev/vg_lab/lv_data
sudo mkdir -p /mnt/lv_data
sudo mount /dev/vg_lab/lv_data /mnt/lv_data
sudo pvs; sudo vgs; sudo lvs
```

**Expected result:** A formatted 2 GB LV is mounted and ready for files.

### Lab 4 — Static and dynamic IP configuration
**Scenario:** Configure a Linux host with a static address, then switch it back to DHCP.

**Inspect first:** `ip addr` and `ip route`.

**Temporary static configuration (replace interface and addresses):**
```bash
ip link                         # find interface, e.g. enp0s3
sudo ip addr flush dev enp0s3
sudo ip addr add 192.168.1.50/24 dev enp0s3
sudo ip link set enp0s3 up
sudo ip route replace default via 192.168.1.1
```
This is temporary and may disconnect your session. For persistent configuration, use your distribution's network manager (NetworkManager or Netplan).

**Return to DHCP (example with NetworkManager):**
```bash
nmcli connection show
sudo nmcli connection modify "Wired connection 1" ipv4.method auto
sudo nmcli connection up "Wired connection 1"
```

**Expected result:** Host receives an address, gateway, and DNS settings from DHCP.

### Lab 5 — Network troubleshooting
**Scenario:** A workstation cannot access websites.

1. Check link and address: `ip link`; `ip addr`.
2. Check route: `ip route`.
3. Ping loopback: `ping -c 4 127.0.0.1`.
4. Ping own gateway: `ping -c 4 192.168.1.1` (replace with actual gateway).
5. Test external IP: `ping -c 4 8.8.8.8`.
6. Test DNS: `ping -c 4 example.com`; inspect DNS with `resolvectl status`.
7. Trace route if needed: `traceroute 8.8.8.8`.

**Quick diagnosis:**
- No IP address → check interface, DHCP, or static configuration.
- Cannot reach gateway → check cable/Wi-Fi, subnet, interface, or gateway.
- Can reach IP but not domain → likely DNS issue.
- Can reach gateway but not outside → check router, route, or upstream connection.

## 3. Quick Revision

- Quota controls per-user/group disk space and file counts.
- RAID 1 mirrors data; RAID is not a backup.
- LVM = PV → VG → LV; flexible volume management.
- `df -h` checks space; `df -i` checks inodes.
- IPv4 is 32-bit; IPv6 is 128-bit.
- `ip addr` shows addresses; `ip route` shows routing.
- Troubleshoot layer by layer: link → address → gateway → external IP → DNS.
