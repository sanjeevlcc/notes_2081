# UNIT 2: INSTALLATION, BOOT PROCESS AND USER ADMINISTRATION

**Course:** System Administration Using Linux\
**Teaching hours:** 5 Theory + 5 Practical\
**Lab environment:** Ubuntu Server/Desktop VM (recommended), or another
Linux VM\
**Level:** Beginner

## 1. Learning outcomes

After this unit, students should be able to:

-   Explain Linux installation methods and the Linux boot sequence.
-   Identify the roles of firmware, bootloader, kernel, and `systemd`.
-   Inspect kernel modules and basic boot settings.
-   Create, modify, lock, and delete users and groups.
-   Apply file permissions and understand ownership.
-   Configure and verify Access Control Lists (ACLs).

> **Safety:** Practise in a virtual machine with a snapshot. Bootloader,
> partition, password, and permission changes can make a system
> inaccessible if applied incorrectly. Commands below are examples;
> device names, bootloader configuration, and available tools vary by
> distribution.

------------------------------------------------------------------------

# 2.1 Linux Installation and Boot Process

## A. Linux installation

Linux can be installed in several ways:

  -----------------------------------------------------------------------
  Method                  Description             Example use
  ----------------------- ----------------------- -----------------------
  Interactive             Install using an ISO    Installing Ubuntu in
  installation            and guided installer    VirtualBox

  Network-based           Installer obtains       Installing many lab
  installation            packages or             computers
                          installation files over 
                          a network               

  Automated installation  Installation uses a     Repeatable server
                          predefined              deployment
                          configuration           

  Cloud image             A prepared image is     Creating a cloud server
                          launched as a virtual   
                          machine                 
  -----------------------------------------------------------------------

### Basic VM installation steps

1.  Download an ISO from the Linux distribution's official website.
2.  Create a new VM in VirtualBox, VMware, or another hypervisor.
3.  Assign suitable RAM, CPU, and virtual disk capacity.
4.  Attach the ISO as the virtual optical disk.
5.  Start the VM and follow the installer.
6.  Create a user and password; configure network and disk options.
7.  Complete installation, reboot, and detach the ISO if necessary.
8.  Log in and verify the system.

**Example VM for a basic Ubuntu Server lab:** 2 virtual CPUs, 2--4 GB
RAM, and a 20 GB virtual disk. Requirements depend on the distribution
and workload.

## B. Network-based installation

In a network installation, a computer starts an installer using network
services or downloads installation packages from a network repository.

A common large-lab arrangement may use:

-   **PXE:** allows a client to boot over the network.
-   **DHCP:** supplies network settings and may provide boot-server
    information.
-   **TFTP or HTTP:** supplies boot files, depending on the setup.
-   **HTTP/HTTPS repository:** supplies installation packages or images.
-   **Automated installation configuration:** provides answers such as
    disk layout, users, and packages.

### Network installation flow

``` mermaid
flowchart TD
    A[Client computer starts] --> B[PXE network boot]
    B --> C[DHCP provides network settings]
    C --> D[Client obtains boot files]
    D --> E[Installer starts]
    E --> F[Packages downloaded from repository]
    F --> G[Automated or interactive setup]
    G --> H[Linux installed and rebooted]
```

A network installation requires correctly configured network services
and a compatible installer. It is more complex than installing from a
local ISO.

## C. Linux boot sequence

The boot process is the sequence of steps from powering on a computer to
reaching the login screen or command prompt.

``` mermaid
flowchart TD
    A[Power on] --> B[BIOS or UEFI firmware]
    B --> C[Boot device selected]
    C --> D[GRUB bootloader]
    D --> E[Linux kernel and initramfs loaded]
    E --> F[Kernel initializes CPU, memory and devices]
    F --> G[Root filesystem mounted]
    G --> H[systemd starts services and targets]
    H --> I[Login prompt or graphical login]
```

### Main boot stages

1.  **BIOS/UEFI:** firmware initializes hardware and selects a boot
    device.
2.  **Bootloader:** commonly GRUB; presents boot choices and loads the
    kernel.
3.  **Kernel:** initializes memory management, process scheduling,
    device support, and other core functions.
4.  **initramfs:** temporary early userspace environment that helps the
    kernel access the real root filesystem.
5.  **systemd (often PID 1):** starts and manages system services and
    reaches the configured target.
6.  **Login:** users can sign in through a terminal or graphical display
    manager.

### Inspect boot information

``` bash
uname -r
```

Example output (your kernel version will differ):

``` text
6.x.x-generic
```

``` bash
systemctl get-default
```

Possible output:

``` text
graphical.target
```

A server may instead use:

``` text
multi-user.target
```

View boot messages:

``` bash
journalctl -b
journalctl -b -p err
```

Show the previous boot's logs, if retained:

``` bash
journalctl -b -1
```

## D. Bootloader: GRUB

GRUB is a commonly used Linux bootloader. It can present a boot menu and
load a selected kernel with its boot parameters.

Inspect the generated GRUB configuration location (do not edit generated
files directly):

``` bash
ls -l /boot/grub/
```

On many Ubuntu/Debian systems, the main configuration source is:

``` bash
sudo nano /etc/default/grub
```

For example, the menu timeout may be set with:

``` text
GRUB_TIMEOUT=5
```

After a deliberate configuration change, regenerate the configuration on
Ubuntu/Debian:

``` bash
sudo update-grub
```

On many Fedora/RHEL-family systems, GRUB configuration management
differs; follow the distribution's documentation and tools.

**Important:** Do not copy a GRUB configuration from another computer or
blindly overwrite files under `/boot/grub`. Incorrect bootloader
settings can prevent the system from booting. Take a VM snapshot first.

## E. Kernel modules

A kernel module is code that can be loaded into or removed from the
running Linux kernel. Modules commonly provide device drivers and
filesystem or network features.

Useful commands:

``` bash
lsmod
```

Lists currently loaded modules.

``` bash
modinfo MODULE_NAME
```

Displays information about a module. Replace `MODULE_NAME` with a real
module shown by `lsmod` or found on the system.

``` bash
sudo modprobe MODULE_NAME
```

Requests loading a module and its dependencies.

``` bash
sudo modprobe -r MODULE_NAME
```

Requests removal of a module if it is not in use.

Example inspection:

``` bash
lsmod | head
```

Possible output (module names and values vary):

``` text
Module                  Size  Used by
overlay                204800  0
nf_conntrack           180224  1
```

Do not unload a storage, network, or other essential module on a working
system just to experiment. Use a disposable VM and understand the
module's purpose first.

------------------------------------------------------------------------

# 2.2 User and Group Administration

## A. Users and groups

A **user account** identifies a person or service that can access a
Linux system. A **group** is a collection of users used to manage shared
access to files and resources.

Important account files:

  File             Purpose
  ---------------- ---------------------------------------------------
  `/etc/passwd`    Basic user account information
  `/etc/shadow`    Protected password hashes and password-aging data
  `/etc/group`     Group information
  `/etc/gshadow`   Protected group administration information

Inspect account information:

``` bash
whoami
id
getent passwd
getent group
```

Do not display or share `/etc/shadow`; it contains sensitive
authentication data.

## B. Create and manage users

Create a user:

``` bash
sudo useradd -m -s /bin/bash ram
```

-   `-m` creates a home directory.
-   `-s /bin/bash` sets the login shell.

Set the user's password interactively:

``` bash
sudo passwd ram
```

Inspect the account:

``` bash
id ram
getent passwd ram
ls -ld /home/ram
```

Modify a user's shell:

``` bash
sudo usermod -s /bin/bash ram
```

Add a user to an additional group:

``` bash
sudo usermod -aG developers ram
```

The `-a` is important with `-G`: it appends the supplementary group
rather than replacing the existing supplementary group list.

Lock and unlock an account:

``` bash
sudo passwd -l ram
sudo passwd -u ram
```

Delete a user but retain the home directory:

``` bash
sudo userdel ram
```

Delete a user and their home directory:

``` bash
sudo userdel -r ram
```

Use `userdel -r` only when deletion is intended and required files have
been backed up.

## C. Group administration

Create a group:

``` bash
sudo groupadd developers
```

Add a user to the group:

``` bash
sudo usermod -aG developers ram
```

Check group membership:

``` bash
id ram
getent group developers
```

Remove a user from a supplementary group (on systems supporting
`gpasswd`):

``` bash
sudo gpasswd -d ram developers
```

Delete a group:

``` bash
sudo groupdel developers
```

A group cannot normally be deleted while it is a user's primary group.

## D. Password policies

Password policy controls how passwords are created, changed, and aged.

Set password aging for a user:

``` bash
sudo chage -l ram
```

Set a maximum password age of 90 days:

``` bash
sudo chage -M 90 ram
```

Set a minimum password age of 1 day:

``` bash
sudo chage -m 1 ram
```

Set a warning period of 7 days:

``` bash
sudo chage -W 7 ram
```

Require a password change at next login:

``` bash
sudo passwd -e ram
```

Password complexity and authentication policy are commonly managed
through PAM (Pluggable Authentication Modules) and distribution-specific
tools. Configuration differs across Ubuntu, Debian, Fedora, and
RHEL-family systems. Do not replace PAM files with copied examples: a
mistake can prevent users, including administrators, from logging in.

------------------------------------------------------------------------

# 2.3 File Permissions and Access Control Lists

## A. Linux ownership and permissions

Each filesystem object has an owner user, an owner group, and permission
bits.

View permissions:

``` bash
ls -l
```

Example:

``` text
-rw-r----- 1 ram developers 120 Nov 14 10:00 project.txt
```

Breakdown:

``` text
-   rw-   r--   ---
|    |     |     |
|   User  Group Others
|
Regular file
```

The first character indicates the file type. The next three groups
represent owner, group, and others.

### Permission meanings

  -----------------------------------------------------------------------
  Permission              File meaning            Directory meaning
  ----------------------- ----------------------- -----------------------
  `r` (4)                 Read file contents      List directory entries

  `w` (2)                 Modify file contents    Create, delete, or
                                                  rename entries (subject
                                                  to other controls)

  `x` (1)                 Execute file            Traverse/access entries
                                                  by name
  -----------------------------------------------------------------------

### Numeric permissions

Permissions can be represented by adding values:

-   `r = 4`
-   `w = 2`
-   `x = 1`

Examples:

    Number Permission
  -------- ------------
       `7` `rwx`
       `6` `rw-`
       `5` `r-x`
       `4` `r--`
       `0` `---`

Common examples:

``` bash
chmod 755 script.sh
chmod 644 notes.txt
chmod 700 private-folder
```

-   `755`: owner has full access; group and others can read and
    traverse/execute.
-   `644`: owner can read/write; group and others can read.
-   `700`: only the owner has permissions.

Symbolic permissions:

``` bash
chmod u+x script.sh
chmod g-w shared.txt
chmod o-r private.txt
```

## B. Change ownership

Change the owner:

``` bash
sudo chown ram project.txt
```

Change owner and group:

``` bash
sudo chown ram:developers project.txt
```

Change group only:

``` bash
sudo chgrp developers project.txt
```

Change ownership recursively (use carefully):

``` bash
sudo chown -R ram:developers project-folder
```

Always verify the target path before using recursive ownership or
permission changes.

## C. Special permission concepts

-   **SUID:** for an executable, runs with the file owner's effective
    user ID.
-   **SGID:** for an executable, uses the file group's effective group
    ID; on directories, newly created entries commonly inherit the
    directory's group.
-   **Sticky bit:** on a shared directory, restricts deletion/renaming
    of entries to the file owner, directory owner, or privileged user.

Example:

``` bash
sudo mkdir -p /shared
sudo chmod 1777 /shared
ls -ld /shared
```

Typical permission display begins with:

``` text
drwxrwxrwt
```

This is commonly used for shared temporary directories such as `/tmp`.

## D. Access Control Lists (ACLs)

Traditional permissions provide access rules for owner, group, and
others. An ACL can provide additional, named-user or named-group
permissions.

Check ACLs:

``` bash
getfacl project.txt
```

Grant user `sita` read access:

``` bash
sudo setfacl -m u:sita:r project.txt
```

Grant group `developers` read/write access:

``` bash
sudo setfacl -m g:developers:rw project.txt
```

Remove a named user's ACL entry:

``` bash
sudo setfacl -x u:sita project.txt
```

Remove all extended ACL entries:

``` bash
sudo setfacl -b project.txt
```

Set a default ACL on a directory so new items can inherit an access
rule:

``` bash
sudo setfacl -m d:u:sita:rwx shared-project
```

Inspect it:

``` bash
getfacl shared-project
```

Default ACLs apply to newly created items under the directory; they do
not automatically change permissions on existing files. The filesystem
and installed ACL utilities must support ACLs.

------------------------------------------------------------------------

# 3. Practical Lab: College Project Team Access

## Scenario

A college ICT department has three students:

-   `ram` --- project leader
-   `sita` --- documentation team
-   `hari` --- developer

They are working on a shared project. The administrator must create
their accounts, create a team group, and protect project files.

**Goal:** Ram and Hari should be members of the `developers` group. Sita
should be able to read the project brief without being added to that
group. Only the project team should be able to modify the shared project
directory.

> Run this lab in your own VM with an account that has `sudo` access. If
> any of these usernames or groups already exist, choose different lab
> names.

## Step 1 --- Create the group and users

``` bash
sudo groupadd developers

sudo useradd -m -s /bin/bash ram
sudo useradd -m -s /bin/bash sita
sudo useradd -m -s /bin/bash hari

sudo passwd ram
sudo passwd sita
sudo passwd hari
```

Set passwords interactively when prompted.

## Step 2 --- Add team members to the group

``` bash
sudo usermod -aG developers ram
sudo usermod -aG developers hari
```

Verify:

``` bash
id ram
id sita
id hari
getent group developers
```

Expected concept:

-   `ram` belongs to `developers`.
-   `hari` belongs to `developers`.
-   `sita` is not a member of `developers`.

The exact numeric user and group IDs depend on the VM.

## Step 3 --- Create the shared project directory

``` bash
sudo mkdir -p /college/project
sudo chown root:developers /college/project
sudo chmod 2770 /college/project
```

Explanation:

-   Owner is `root`.
-   Group is `developers`.
-   `2770` gives owner and group full access, no access to others, and
    sets SGID on the directory.
-   SGID helps new items inherit the directory's group.

Check:

``` bash
ls -ld /college/project
```

Expected permission pattern:

``` text
drwxrws--- ... root developers ... /college/project
```

## Step 4 --- Create a project brief

``` bash
echo "College ICT Project - Linux Administration" | sudo tee /college/project/brief.txt
sudo chown root:developers /college/project/brief.txt
sudo chmod 640 /college/project/brief.txt
```

Expected permission pattern:

``` text
-rw-r----- ... root developers ... brief.txt
```

The group can read the file but cannot modify it with these traditional
permission bits.

## Step 5 --- Give Sita read-only access using ACL

``` bash
sudo setfacl -m u:sita:r-- /college/project/brief.txt
getfacl /college/project/brief.txt
```

The ACL output should include an entry similar to:

``` text
user:sita:r--
```

The ACL mask may also appear. The effective permission is limited by the
mask, so check the `getfacl` output rather than assuming an ACL entry
always grants its full listed permissions.

## Step 6 --- Test access

If `sudo` is available, test as each user:

``` bash
sudo -u ram cat /college/project/brief.txt
sudo -u hari cat /college/project/brief.txt
sudo -u sita cat /college/project/brief.txt
```

All three should be able to read the brief under the intended
configuration.

Try writing as Sita:

``` bash
sudo -u sita sh -c 'echo "Sita update" >> /college/project/brief.txt'
```

Expected result: permission denied, because Sita has read-only access.

Try creating a file as Ram:

``` bash
sudo -u ram touch /college/project/ram-file.txt
ls -l /college/project/
```

**Note:** Newly added group membership may require a fresh login session
for the user. `sudo -u` starts a command with the target user's identity
and is useful for this lab.

## Step 7 --- Set a password aging policy

``` bash
sudo chage -M 90 -W 7 ram
sudo chage -l ram
```

The output should show the configured maximum password age and warning
period. Exact dates depend on when the account was created and the
system clock.

## Step 8 --- Clean up (optional)

Only after completing and checking the lab:

``` bash
sudo rm -r /college/project
sudo userdel -r ram
sudo userdel -r sita
sudo userdel -r hari
sudo groupdel developers
```

Do not run cleanup commands on a real college server or on accounts that
contain actual user data.

------------------------------------------------------------------------

# 4. Mini Case Study: The Server Reboots and the Team Cannot Access Files

## Situation

A college server was restarted after a power interruption. The system
boots to a login prompt, but the project team reports that:

-   The shared project directory is not accessible to the expected
    users.
-   One student can read a report but cannot edit it.
-   Another student should have read-only access.
-   The administrator needs to confirm that the system booted
    successfully.

You are the junior Linux administrator. Investigate the situation in a
lab VM.

## Part A --- Check the booted system

``` bash
uptime
uname -r
systemctl get-default
systemctl --failed
journalctl -b -p err
```

Record:

-   Running kernel version.
-   Default systemd target.
-   Any failed services.
-   Boot-time error messages.

Do not assume every warning in the journal means the boot failed.

## Part B --- Inspect users and groups

``` bash
id ram
id sita
id hari
getent group developers
```

Questions:

1.  Are Ram and Hari members of `developers`?
2.  Is Sita a member of that group?
3.  Are the accounts present?

## Part C --- Inspect ownership and permissions

``` bash
ls -ld /college/project
ls -l /college/project
getfacl /college/project/brief.txt
```

Check:

-   Directory owner and group.
-   Directory permission bits.
-   File owner and group.
-   Named-user ACL entries.
-   ACL mask and effective permissions.

## Part D --- Repair the intended access

If the intended policy is still:

-   Ram and Hari: team access.
-   Sita: read-only access to the brief.
-   Other users: no access.

Reapply the lab configuration carefully:

``` bash
sudo chown root:developers /college/project
sudo chmod 2770 /college/project

sudo chown root:developers /college/project/brief.txt
sudo chmod 640 /college/project/brief.txt
sudo setfacl -m u:sita:r-- /college/project/brief.txt
```

Verify:

``` bash
ls -ld /college/project
getfacl /college/project/brief.txt
```

Test:

``` bash
sudo -u ram cat /college/project/brief.txt
sudo -u sita cat /college/project/brief.txt
sudo -u sita sh -c 'echo test >> /college/project/brief.txt'
```

The final write attempt should fail if the read-only policy is
effective.

## Part E --- Write a short incident report

``` bash
cat > ~/unit2-incident-report.txt <<'EOF'
COLLEGE LINUX SERVER INCIDENT REPORT
====================================
Boot status: Checked using systemd and journalctl
User accounts: Checked using id and getent
Group membership: Checked using getent group
Ownership: Checked using ls -l
ACL: Checked using getfacl
Access policy: Team access for developers; read-only access for Sita
Verification: Tested user access with sudo -u
EOF

cat ~/unit2-incident-report.txt
```

## Case-study questions

1.  What is the role of UEFI/BIOS during startup?
2.  What does GRUB do?
3.  Why does Linux use initramfs?
4.  What is the role of `systemd`?
5.  What is the difference between a user and a group?
6.  Why is `usermod -aG` safer than `usermod -G` when adding a
    supplementary group?
7.  What does permission `2770` mean on a directory?
8.  Why is an ACL useful when Sita needs access but should not join the
    developers group?
9.  What is the ACL mask?
10. Why should an administrator verify permissions after changing them?

**Expected learning:** Boot troubleshooting and access-control
troubleshooting are separate tasks. First establish whether the system
booted and services are healthy. Then inspect account identity, group
membership, ownership, permission bits, and ACLs. Change only the
setting that does not match the required policy.

------------------------------------------------------------------------

# 5. Quick Revision Cheat Sheet

``` bash
# BOOT AND KERNEL
uname -r                       # Kernel version
systemctl get-default          # Default systemd target
systemctl --failed             # Failed services/units
journalctl -b                  # Current boot logs
journalctl -b -p err           # Current boot errors
lsmod                          # Loaded kernel modules
modinfo MODULE_NAME            # Module details

# USERS
sudo useradd -m -s /bin/bash USER
sudo passwd USER
id USER
getent passwd USER
sudo usermod -aG GROUP USER
sudo usermod -s /bin/bash USER
sudo passwd -l USER            # Lock account
sudo passwd -u USER            # Unlock account
sudo userdel USER
sudo userdel -r USER           # Also remove home directory

# GROUPS
sudo groupadd GROUP
getent group GROUP
sudo gpasswd -d USER GROUP
sudo groupdel GROUP

# PASSWORD AGING
sudo chage -l USER
sudo chage -M 90 USER
sudo chage -m 1 USER
sudo chage -W 7 USER
sudo passwd -e USER

# PERMISSIONS AND OWNERSHIP
ls -l
chmod 755 script.sh
chmod 640 report.txt
chmod u+x script.sh
sudo chown USER:GROUP file
sudo chgrp GROUP file

# ACL
getfacl file
sudo setfacl -m u:USER:r file
sudo setfacl -m g:GROUP:rw file
sudo setfacl -x u:USER file
sudo setfacl -b file
sudo setfacl -m d:u:USER:rwx directory
```

------------------------------------------------------------------------

# 6. Short Assessment

## Multiple-choice questions

**1. Which component loads the Linux kernel during a typical GRUB-based
boot?**

A. `grep`\
B. GRUB\
C. `chmod`\
D. `passwd`

**Answer:** B --- GRUB.

**2. Which command shows the running kernel version?**

A. `uname -r`\
B. `id`\
C. `ls -l`\
D. `chage`

**Answer:** A --- `uname -r`.

**3. Which file stores basic local user account information?**

A. `/etc/group`\
B. `/etc/passwd`\
C. `/boot/grub/grub.cfg`\
D. `/var/log`

**Answer:** B --- `/etc/passwd`.

**4. Which command adds a user to a supplementary group without
replacing existing supplementary groups?**

A. `userdel`\
B. `usermod -aG`\
C. `groupdel`\
D. `passwd -l`

**Answer:** B --- `usermod -aG`.

**5. What does permission `755` mean?**

A. Owner `rwx`, group `r-x`, others `r-x`\
B. Owner `rw-`, group `rw-`, others `---`\
C. Everyone has full permission\
D. Nobody has permission

**Answer:** A.

**6. Which command displays ACL entries?**

A. `getfacl`\
B. `lsmod`\
C. `uptime`\
D. `uname`

**Answer:** A --- `getfacl`.

## Practical assessment

1.  Identify the kernel version and default systemd target.
2.  Create users `student1` and `student2`.
3.  Create a group named `labteam`.
4.  Add both users to `labteam`.
5.  Create `/lab/shared` owned by `root:labteam` with permission `2770`.
6.  Create a file and set its permission to `640`.
7.  Grant `student2` read-only access using an ACL.
8.  Verify the result using `id`, `ls -l`, and `getfacl`.

**Suggested marking:** Installation/boot concepts 4 marks; users and
groups 5 marks; permissions and ownership 5 marks; ACL configuration and
verification 6 marks. Total: 20 marks.

------------------------------------------------------------------------

## 7. Further reading

-   Ubuntu Server documentation:
    https://documentation.ubuntu.com/server/
-   systemd manual: https://www.freedesktop.org/software/systemd/man/
-   GNU GRUB manual: https://www.gnu.org/software/grub/manual/
-   Linux manual pages: `man useradd`, `man usermod`, `man chmod`,
    `man chown`, `man setfacl`, `man getfacl`
-   Linux kernel documentation: https://docs.kernel.org/

**Instructor note:** Bootloader commands, GRUB configuration paths, PAM
policy, ACL support, and package availability differ by distribution.
Demonstrate risky boot changes only in a disposable VM. Students should
record their own outputs; sample permission patterns are illustrative.
