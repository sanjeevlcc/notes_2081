# UNIT IV: SYSTEM ADMINISTRATION BASICS

**Total Hours:** 9

## Specific Objectives

1. Understand the Linux boot process and kernel basics
2. Manage processes and background tasks
3. Edit files using the Vi editor
4. Manage users, groups, and superuser privileges
5. Monitor disk usage and manage partitions
6. Install and remove software packages efficiently

> **Note:** Commands marked `#` are run as root (or with `sudo`); commands marked `$` are run as a normal user.

---

## 4.1 Understanding the Boot Process and Related System Files

### Boot sequence (step by step)

| Stage | What happens |
|-------|--------------|
| 1. **BIOS / UEFI** | Firmware runs POST (Power-On Self Test), finds the boot device |
| 2. **MBR / GPT** | Boot sector is read; it loads the bootloader |
| 3. **Bootloader (GRUB2)** | Shows the boot menu, loads the kernel and initramfs into memory |
| 4. **Kernel** | Initializes hardware, mounts the root filesystem (read-only first) |
| 5. **initramfs** | Temporary root filesystem with drivers needed to mount the real root |
| 6. **init / systemd (PID 1)** | First process; starts all services up to the default target |
| 7. **Login prompt** | Text (getty) or graphical (display manager) login appears |

### Important boot-related files and directories

| Path | Purpose |
|------|---------|
| `/boot/` | Kernel (`vmlinuz-*`), initramfs (`initramfs-*.img`), GRUB files |
| `/boot/grub2/grub.cfg` (RHEL/CentOS) or `/boot/grub/grub.cfg` (Debian/Ubuntu) | Generated GRUB configuration (do not edit by hand) |
| `/etc/default/grub` | Editable GRUB settings (timeout, kernel parameters) |
| `/etc/fstab` | Filesystems mounted at boot |
| `/etc/inittab` | Legacy SysV init default runlevel (older systems) |
| `/etc/systemd/system/` | systemd unit files and default target link |
| `/etc/os-release` | OS name and version |
| `/var/log/boot.log`, `/var/log/messages`, `/var/log/syslog` | Boot and system logs |

### Runlevels vs systemd targets

| SysV Runlevel | systemd Target | Meaning |
|---------------|----------------|---------|
| 0 | `poweroff.target` | Halt / shutdown |
| 1 | `rescue.target` | Single-user / rescue mode |
| 3 | `multi-user.target` | Multi-user, text mode (no GUI) |
| 5 | `graphical.target` | Multi-user with GUI |
| 6 | `reboot.target` | Reboot |

### Command examples

```bash
# Show the current default target
systemctl get-default

# Change default boot to text mode / graphical mode
sudo systemctl set-default multi-user.target
sudo systemctl set-default graphical.target

# Switch to rescue mode right now
sudo systemctl isolate rescue.target

# Show current runlevel (legacy)
runlevel

# View kernel boot messages
dmesg | less
dmesg | grep -i error

# View logs of the current boot / previous boot
journalctl -b
journalctl -b -1

# Time taken by each boot stage
systemd-analyze
systemd-analyze blame

# Regenerate the GRUB configuration after editing /etc/default/grub
sudo grub2-mkconfig -o /boot/grub2/grub.cfg      # RHEL / CentOS / Fedora
sudo update-grub                                  # Debian / Ubuntu

# Reboot / shutdown
sudo reboot
sudo shutdown -h now
sudo shutdown -r +10 "Rebooting in 10 minutes"
```

---

## 4.2 Linux Kernel Fundamentals

The **kernel** is the core of the operating system. It sits between hardware and applications.

### Main functions of the kernel

- **Process management** – creating, scheduling, and terminating processes
- **Memory management** – RAM allocation, virtual memory, swapping
- **Device management** – drivers for disks, network cards, USB, etc.
- **File system management** – ext4, xfs, etc.
- **System calls and security** – interface between user programs and hardware

### Kernel types

- **Monolithic** (Linux): whole OS runs in kernel space, supports loadable modules
- **Microkernel**: only minimal services in kernel space

### Kernel-related locations

| Path | Purpose |
|------|---------|
| `/boot/vmlinuz-<version>` | Compressed kernel image |
| `/lib/modules/<version>/` | Loadable kernel modules (drivers) |
| `/proc/` | Virtual filesystem exposing kernel and process info |
| `/sys/` | Virtual filesystem for devices and drivers |
| `/etc/sysctl.conf`, `/etc/sysctl.d/` | Persistent kernel parameters |

### Command examples

```bash
# Kernel version and system info
uname -r          # kernel release
uname -a          # all information

# CPU and memory info from /proc
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/version

# List loaded kernel modules
lsmod

# Information about a module
modinfo ext4

# Load / remove a module
sudo modprobe usb_storage
sudo modprobe -r usb_storage
sudo rmmod usb_storage

# View / change kernel parameters at runtime
sysctl -a | grep ip_forward
sudo sysctl -w net.ipv4.ip_forward=1

# Make the change permanent
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Hardware information
lspci
lsusb
```

---

## 4.3 Background Processing

A **foreground** process holds the terminal. A **background** process runs while you keep using the shell.

### Key operators and commands

| Command / Key | Description |
|---------------|-------------|
| `command &` | Start a command in the background |
| `Ctrl + Z` | Suspend (stop) the foreground process |
| `Ctrl + C` | Terminate the foreground process |
| `jobs` | List background and stopped jobs |
| `bg %n` | Resume job *n* in the background |
| `fg %n` | Bring job *n* to the foreground |
| `nohup` | Keep a process running after logout |
| `disown` | Detach a job from the shell |

### Command examples

```bash
# Run in background
sleep 300 &

# List jobs
jobs
# [1]+  Running    sleep 300 &

# Start a foreground job, suspend it, resume in background
sleep 500
# press Ctrl + Z
bg %1

# Bring it back to the foreground
fg %1

# Keep running after logout; output goes to nohup.out
nohup ./backup.sh &

# Redirect output to a custom log file
nohup ./backup.sh > backup.log 2>&1 &

# Detach an already running job from the shell
disown %1

# Run at a scheduled time (one-time job)
echo "tar -czf /tmp/home.tar.gz /home" | at 23:00
atq              # list scheduled jobs
atrm 1           # remove job number 1

# Terminal multiplexers (sessions survive disconnects)
tmux new -s mysession
screen -S mysession
```

---

## 4.4 Process Commands

A **process** is a running instance of a program. Each has a unique **PID** (Process ID) and a parent **PPID**.

### a. `kill` — send signals to processes

| Signal | Number | Meaning |
|--------|--------|---------|
| `SIGHUP` | 1 | Hang up / reload configuration |
| `SIGINT` | 2 | Interrupt (same as Ctrl + C) |
| `SIGKILL` | 9 | Force kill (cannot be ignored) |
| `SIGTERM` | 15 | Graceful terminate (default) |
| `SIGSTOP` | 19 | Pause process |
| `SIGCONT` | 18 | Continue a paused process |

```bash
# List all signals
kill -l

# Graceful termination (default SIGTERM)
kill 1234

# Force kill
kill -9 1234
kill -SIGKILL 1234

# Reload configuration without stopping
kill -HUP 1234

# Kill by name
pkill firefox
killall nginx

# Kill all processes of a user
sudo pkill -u student

# Kill a background job by job number
kill %1
```

### b. `ps` — snapshot of current processes

```bash
# Processes in the current shell
ps

# All processes, full format (BSD style)
ps aux

# All processes, full format (standard style)
ps -ef

# Search for a process
ps aux | grep nginx

# Show process tree
ps -ef --forest

# Show specific columns
ps -o pid,ppid,user,%cpu,%mem,cmd -u root

# Sort by memory usage (top 5)
ps aux --sort=-%mem | head -5

# Find PID of a program
pidof sshd
pgrep -l sshd
```

**Columns in `ps aux`:** `USER`, `PID`, `%CPU`, `%MEM`, `VSZ`, `RSS`, `TTY`, `STAT`, `START`, `TIME`, `COMMAND`

**Common STAT codes:** `R` running, `S` sleeping, `D` uninterruptible sleep, `T` stopped, `Z` zombie

### c. `who` — who is logged in

```bash
who                 # logged-in users
who -a              # all information
who -b              # last system boot time
who am i            # your own login session
whoami              # current username
w                   # users and what they are doing
last                # login history
last -n 5           # last 5 logins
lastlog             # last login of every user
id                  # UID, GID and groups of current user
uptime              # system uptime and load average
```

### d. `top` — real-time process monitor

```bash
top                 # start interactive monitor
top -u student      # only one user's processes
top -p 1234         # monitor a specific PID
top -b -n 1         # batch mode, one iteration (good for scripts)
htop                # friendlier alternative (may need installation)
```

**Interactive keys inside `top`:**

| Key | Action |
|-----|--------|
| `q` | Quit |
| `k` | Kill a process (asks for PID) |
| `r` | Renice (change priority) |
| `M` | Sort by memory |
| `P` | Sort by CPU |
| `h` | Help |
| `1` | Show each CPU core |

### Process priority

```bash
nice -n 10 ./script.sh          # start with lower priority (-20 highest, 19 lowest)
renice -n 5 -p 1234             # change priority of a running process
```

---

## 4.5 Creating and Editing Files with the Vi Editor

Vi (or **Vim**) is a modal editor available on virtually every Linux system.

### Modes

| Mode | How to enter | Purpose |
|------|--------------|---------|
| **Command (Normal)** | Press `Esc` | Navigate, delete, copy, paste |
| **Insert** | `i`, `a`, `o` | Type text |
| **Last-line (Ex)** | `:` | Save, quit, search, replace |

### Open / create a file

```bash
vi notes.txt          # open or create a file
vi +10 notes.txt      # open at line 10
vi -R notes.txt       # open read-only
vimtutor              # built-in interactive tutorial
```

### Entering insert mode

| Key | Action |
|-----|--------|
| `i` | Insert before the cursor |
| `a` | Append after the cursor |
| `I` | Insert at beginning of line |
| `A` | Append at end of line |
| `o` | Open a new line below |
| `O` | Open a new line above |

### Navigation (Command mode)

| Key | Action |
|-----|--------|
| `h j k l` | Left, down, up, right |
| `w` / `b` | Next / previous word |
| `0` / `$` | Start / end of line |
| `gg` / `G` | First / last line of file |
| `:n` or `nG` | Go to line *n* |
| `Ctrl+f` / `Ctrl+b` | Page forward / back |

### Editing (Command mode)

| Key | Action |
|-----|--------|
| `x` | Delete character |
| `dd` | Delete (cut) current line |
| `5dd` | Delete 5 lines |
| `dw` | Delete word |
| `yy` | Copy (yank) current line |
| `p` / `P` | Paste after / before cursor |
| `u` | Undo |
| `Ctrl+r` | Redo |
| `.` | Repeat last command |
| `r` | Replace one character |

### Search and replace

```
/word          search forward        n  next match     N  previous match
?word          search backward
:%s/old/new/g          replace all occurrences in the file
:%s/old/new/gc         replace all with confirmation
:1,10s/old/new/g       replace only in lines 1 to 10
```

### Save and quit (Last-line mode)

| Command | Action |
|---------|--------|
| `:w` | Save |
| `:q` | Quit |
| `:wq` or `ZZ` | Save and quit |
| `:q!` | Quit without saving |
| `:w newfile` | Save as a new file |
| `:set nu` | Show line numbers |
| `:set nonu` | Hide line numbers |

### Quick practice

```text
1. vi hello.txt
2. Press i            -> type: Hello Linux
3. Press Esc
4. Type :wq and press Enter
```

---

## 4.6 Managing User Accounts

### Important files

| File | Content |
|------|---------|
| `/etc/passwd` | User account info (`username:x:UID:GID:comment:home:shell`) |
| `/etc/shadow` | Encrypted passwords and aging info |
| `/etc/group` | Group information |
| `/etc/gshadow` | Group passwords |
| `/etc/skel/` | Template files copied to every new home directory |
| `/etc/login.defs` | Defaults for UID ranges, password aging |

### a. Adding users

```bash
# Simple user (home directory creation depends on distro defaults)
sudo useradd ram

# With home directory, shell and comment
sudo useradd -m -s /bin/bash -c "Ram Sharma" ram

# Specify UID, primary group and secondary groups
sudo useradd -m -u 1500 -g developers -G sudo,docker ram

# Custom home directory
sudo useradd -m -d /home/projects/ram ram

# Account that expires on a date
sudo useradd -m -e 2026-12-31 intern

# Set the password
sudo passwd ram

# Debian/Ubuntu friendly interactive tool
sudo adduser ram
```

### b. Deleting users

```bash
sudo userdel ram            # delete user, keep home directory
sudo userdel -r ram         # delete user and home directory + mail spool
sudo deluser ram            # Debian/Ubuntu
sudo deluser --remove-home ram
```

### c. Modifying users

```bash
sudo usermod -l newname oldname              # rename login
sudo usermod -c "Ram B. Sharma" ram          # change comment
sudo usermod -s /bin/zsh ram                 # change shell
sudo usermod -d /home/newhome -m ram         # change and move home directory
sudo usermod -aG docker,sudo ram             # APPEND to supplementary groups
sudo usermod -g developers ram               # change primary group
sudo usermod -L ram                          # lock account
sudo usermod -U ram                          # unlock account
sudo usermod -e 2027-01-01 ram               # set expiry date

# Password management
sudo passwd -l ram          # lock password
sudo passwd -u ram          # unlock password
sudo passwd -e ram          # force password change at next login
sudo passwd -d ram          # delete password

# Password aging
sudo chage -l ram                       # show aging info
sudo chage -M 90 -W 7 ram               # max 90 days, warn 7 days before
sudo chage -d 0 ram                     # force change at next login

# Verify
id ram
grep ram /etc/passwd
```

> **Warning:** Use `usermod -aG` (with **-a**). Using `-G` alone *replaces* all supplementary groups.

---

## 4.7 Becoming Superuser

The **root** user (UID 0) has unrestricted access. Avoid logging in directly as root; use `su` or `sudo`.

```bash
# Switch to root (needs root password)
su
su -                  # login shell for root (loads root's environment)

# Switch to another user
su - ram
su -c "systemctl restart nginx" -     # run one command as root

# Run a single command with root privileges (needs sudo rights)
sudo apt update
sudo -i                # interactive root shell
sudo -u ram whoami     # run as a different user
sudo -l                # list what you are allowed to run

# Exit back to normal user
exit
```

### Granting sudo rights

```bash
# Always edit the sudoers file with visudo (it checks syntax)
sudo visudo

# Example entries in /etc/sudoers
ram     ALL=(ALL)       ALL                    # full sudo access
%admins ALL=(ALL)       ALL                    # whole group
sita    ALL=(ALL)       NOPASSWD: /usr/bin/systemctl   # only one command, no password

# Easier way: add the user to the admin group
sudo usermod -aG wheel ram        # RHEL / CentOS (group: wheel)
sudo usermod -aG sudo ram         # Debian / Ubuntu (group: sudo)
```

**Logs:** sudo activity is recorded in `/var/log/secure` (RHEL) or `/var/log/auth.log` (Debian/Ubuntu).

---

## 4.8 Creating and Managing Groups

Groups let you give the same permissions to many users at once.

```bash
# Create a group
sudo groupadd developers
sudo groupadd -g 2000 testers          # with specific GID

# Modify a group
sudo groupmod -n devteam developers    # rename
sudo groupmod -g 2500 devteam          # change GID

# Delete a group
sudo groupdel devteam

# Add / remove users in a group
sudo usermod -aG developers ram        # add user
sudo gpasswd -a sita developers        # add user
sudo gpasswd -d sita developers        # remove user
sudo gpasswd -A ram developers         # make ram the group administrator

# Set group password (rarely used)
sudo gpasswd developers

# View group membership
groups ram
id ram
getent group developers
cat /etc/group | grep developers

# Temporarily switch active group
newgrp developers

# Shared directory example
sudo mkdir /projects
sudo chgrp developers /projects
sudo chmod 2775 /projects        # setgid: new files inherit the group
```

---

## 4.9 Disk Partitions and Sizes

### a. `df` — disk free space (per filesystem)

```bash
df                    # in 1K blocks
df -h                 # human-readable (K, M, G)
df -T                 # show filesystem type
df -i                 # inode usage
df -h /home           # only the filesystem containing /home
df -hT --exclude-type=tmpfs
```

### b. `du` — disk usage (per file/directory)

```bash
du -h /var/log                    # every subdirectory
du -sh /home/ram                  # summary (total) only
du -sh *                          # size of everything in current directory
du -h --max-depth=1 /var          # one level deep
du -ah /etc | sort -rh | head -10 # 10 biggest items
du -sh --exclude="*.log" /var
```

### c. `dd` — convert and copy data (low-level)

```bash
# Create a 100 MB file filled with zeros
dd if=/dev/zero of=testfile.img bs=1M count=100

# Backup a whole disk to an image
sudo dd if=/dev/sdb of=/backup/sdb.img bs=4M status=progress

# Restore the image to a disk (DANGEROUS: overwrites the target)
sudo dd if=/backup/sdb.img of=/dev/sdb bs=4M status=progress

# Backup the Master Boot Record (first 512 bytes)
sudo dd if=/dev/sda of=mbr.bak bs=512 count=1

# Write an ISO to a USB drive
sudo dd if=ubuntu.iso of=/dev/sdc bs=4M status=progress conv=fsync

# Test disk write speed
dd if=/dev/zero of=speedtest bs=1G count=1 oflag=dsync
```

**Options:** `if` = input file, `of` = output file, `bs` = block size, `count` = number of blocks.

> **Warning:** `dd` does exactly what you tell it. A wrong `of=` can wipe a disk permanently. Double-check device names with `lsblk` first.

### Partition management (related commands)

```bash
# List disks and partitions
lsblk
lsblk -f                     # with filesystem and UUID
sudo fdisk -l
sudo blkid

# Partition a disk (interactive)
sudo fdisk /dev/sdb
#   n  -> new partition      d -> delete      p -> print
#   t  -> change type        w -> write changes and exit    q -> quit without saving

# GPT disks / disks larger than 2 TB
sudo parted /dev/sdb print
sudo gdisk /dev/sdb

# Create a filesystem
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.xfs /dev/sdb1

# Mount and unmount
sudo mkdir /mnt/data
sudo mount /dev/sdb1 /mnt/data
sudo umount /mnt/data

# Mount automatically at boot: add a line to /etc/fstab
# UUID=<uuid-from-blkid>   /mnt/data   ext4   defaults   0  2
sudo mount -a                # test /etc/fstab for errors

# Swap space
sudo mkswap /dev/sdb2
sudo swapon /dev/sdb2
swapon --show
free -h                      # memory and swap usage
```

---

## 4.10 Installing and Removing Packages

A **package** bundles software, metadata, and dependency information.

| Family | Distros | Low-level tool | High-level tool |
|--------|---------|----------------|-----------------|
| Red Hat | RHEL, CentOS, Rocky, Alma, Fedora | `rpm` | `yum` / `dnf` |
| Debian | Debian, Ubuntu, Mint | `dpkg` | `apt-get` / `apt` |

**Low-level tools** (`rpm`, `dpkg`) install a single file and **do not** resolve dependencies. **High-level tools** (`yum`, `apt-get`) download from repositories and resolve dependencies automatically.

### a. RPM

```bash
# Install
sudo rpm -ivh package.rpm            # i=install v=verbose h=hash progress

# Upgrade / freshen
sudo rpm -Uvh package.rpm            # upgrade (or install if absent)
sudo rpm -Fvh package.rpm            # upgrade only if already installed

# Remove
sudo rpm -e package-name

# Query
rpm -qa                              # all installed packages
rpm -qa | grep httpd                 # search installed packages
rpm -q httpd                         # is it installed?
rpm -qi httpd                        # package information
rpm -ql httpd                        # files installed by the package
rpm -qc httpd                        # configuration files
rpm -qf /usr/sbin/httpd              # which package owns this file
rpm -qpi package.rpm                 # info about an uninstalled .rpm file
rpm -qpl package.rpm                 # files inside an uninstalled .rpm

# Verify and check
rpm -V httpd                         # verify installed files
sudo rpm --import RPM-GPG-KEY        # import a GPG key
rpm -K package.rpm                   # check signature

# Install ignoring dependencies (not recommended)
sudo rpm -ivh --nodeps package.rpm
```

### b. apt-get (and dpkg)

```bash
# Refresh the package list (always do this first)
sudo apt-get update

# Install
sudo apt-get install nginx
sudo apt-get install -y nginx curl git       # -y = auto-confirm
sudo apt-get install nginx=1.18.0-0ubuntu1   # specific version

# Upgrade
sudo apt-get upgrade                         # upgrade installed packages
sudo apt-get dist-upgrade                    # upgrade, handling changed dependencies
sudo apt-get install --only-upgrade nginx    # upgrade one package

# Remove
sudo apt-get remove nginx                    # remove, keep config files
sudo apt-get purge nginx                     # remove including config files
sudo apt-get autoremove                      # remove unused dependencies
sudo apt-get clean                           # clear the downloaded package cache

# Search and info (apt-cache / apt)
apt-cache search web server
apt-cache show nginx
apt-cache policy nginx
apt list --installed

# Using dpkg for a local .deb file
sudo dpkg -i package.deb                     # install
sudo dpkg -r package-name                    # remove
sudo dpkg -P package-name                    # purge
dpkg -l                                      # list installed
dpkg -L nginx                                # files of a package
dpkg -S /usr/sbin/nginx                      # which package owns a file
sudo apt-get install -f                      # fix broken dependencies

# Repository configuration
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/
```

### c. yum

```bash
# Install
sudo yum install httpd
sudo yum -y install httpd php mariadb-server     # multiple packages, auto-confirm
sudo yum localinstall package.rpm                # local file with dependency resolution

# Update
sudo yum check-update                            # list available updates
sudo yum update                                  # update everything
sudo yum update httpd                            # update one package

# Remove
sudo yum remove httpd
sudo yum autoremove

# Search and information
yum search apache
yum info httpd
yum list installed
yum list available | grep php
yum provides /usr/bin/python3                    # which package provides a file

# Package groups
yum grouplist
sudo yum groupinstall "Development Tools"
sudo yum groupremove "Development Tools"

# Repositories
yum repolist
yum repolist all
ls /etc/yum.repos.d/
sudo yum --enablerepo=epel install htop
sudo yum install epel-release                    # enable EPEL repository

# History and cache
yum history
sudo yum history undo 5                          # undo transaction 5
sudo yum clean all
```

> **Note:** On modern RHEL/CentOS 8+, `dnf` replaces `yum` with the same syntax (e.g. `sudo dnf install httpd`).

### Sample repository file `/etc/yum.repos.d/custom.repo`

```ini
[custom-repo]
name=My Custom Repository
baseurl=http://repo.example.com/centos/$releasever/$basearch/
enabled=1
gpgcheck=1
gpgkey=http://repo.example.com/RPM-GPG-KEY
```

---

## Quick Revision Cheat Sheet

| Task | Command |
|------|---------|
| Show kernel version | `uname -r` |
| Default boot target | `systemctl get-default` |
| Run in background | `command &` |
| List jobs | `jobs` |
| Show all processes | `ps aux` |
| Kill a process | `kill -9 PID` |
| Who is logged in | `who` / `w` |
| Live process monitor | `top` |
| Save and quit Vi | `:wq` |
| Add user | `useradd -m username` |
| Delete user + home | `userdel -r username` |
| Modify user | `usermod -aG group username` |
| Become root | `su -` / `sudo -i` |
| Create group | `groupadd groupname` |
| Free disk space | `df -h` |
| Directory size | `du -sh dirname` |
| Copy raw data | `dd if=src of=dest bs=4M` |
| RPM install / remove | `rpm -ivh pkg.rpm` / `rpm -e pkg` |
| APT install / remove | `apt-get install pkg` / `apt-get remove pkg` |
| YUM install / remove | `yum install pkg` / `yum remove pkg` |

## Practice Exercises

1. Find your kernel version and the default systemd target of your system.
2. Start `sleep 600 &`, list it with `jobs`, then kill it using `kill %1`.
3. Use `ps aux --sort=-%mem | head` to find the top 5 memory-consuming processes.
4. Create a file `notes.txt` with Vi, write three lines, save it, then replace a word using `:%s/old/new/g`.
5. Create the user `student1` with a home directory, add it to a new group `lab`, then delete both.
6. Grant `student1` sudo access using the `sudo`/`wheel` group.
7. Find which directory in `/var` uses the most disk space using `du`.
8. Create a 50 MB file using `dd` and delete it afterwards.
9. Install `htop` with `apt-get` or `yum`, check its details, then remove it.
10. Use `rpm -qa` or `dpkg -l` to count how many packages are installed.

---
*End of Unit IV notes*
