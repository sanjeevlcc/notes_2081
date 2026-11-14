# UNIT 1: INTRODUCTION TO LINUX AND CLI COMMANDS

**Course:** System Administration Using Linux\
**Unit:** 1 --- Introduction to Linux and CLI Commands\
**Teaching hours:** 5 Theory + 5 Practical\
**Practical environment:** Ubuntu/Debian Linux, a Linux virtual machine,
WSL, or KillerCoda\
**Level:** Beginner

------------------------------------------------------------------------

## 1. Learning outcomes

By the end of this unit, students should be able to:

1.  Explain open-source software and the Linux ecosystem.
2.  Describe the basic architecture of a Unix/Linux operating system.
3.  Use a terminal and understand Linux command syntax.
4.  Navigate directories and create, copy, move, rename, and delete
    files.
5.  Use common shell commands to inspect and manage a Linux system.
6.  Search, filter, extract, and modify text using `grep`, `awk`, and
    `sed`.
7.  Inspect processes and control foreground/background jobs.
8.  Explain Linux filesystem hierarchy, mounts, and disk usage.
9.  Identify the package manager used by a Linux distribution and
    install/update software.

## 2. Suggested teaching and lab sequence

  ------------------------------------------------------------------------
  Session                 Theory focus            Practical focus
  ----------------------- ----------------------- ------------------------
  1                       Open source, Unix/Linux Log in, inspect system,
                          history, architecture,  use `man`, `pwd`, `ls`,
                          terminal and command    `whoami`
                          syntax                  

  2                       File and directory      Create a practice
                          management              directory tree; copy,
                                                  move, rename and remove
                                                  files

  3                       Shell commands and text Use pipes, redirection,
                          processing              `grep`, `awk`, `sed`,
                                                  `sort`, `uniq`, `wc`

  4                       Processes, jobs and     Use `ps`, `top`, `jobs`,
                          filesystem concepts     `bg`, `fg`, `kill`,
                                                  `df`, `du`, `find`

  5                       Package management and  Inspect/install/remove a
                          revision                package; complete the
                                                  integrated case study
  ------------------------------------------------------------------------

> **Safety rule:** Practise in a lab directory or disposable virtual
> machine. Do not experiment with `rm -rf`, disk formatting,
> partitioning, or package removal on a production server.

------------------------------------------------------------------------

# 1.1 Introduction to Open-Source Software

## What is software?

Software is a collection of programs and instructions that tells a
computer what to do. An operating system, web browser, database, and
text editor are all examples of software.

## What is open-source software?

Open-source software makes its source code available under a licence
that allows people to inspect, use, modify, and redistribute it, subject
to that licence's conditions.

**Example:** Linux kernel source code can be inspected and modified.
Different organizations build Linux distributions by combining the
kernel with system tools, libraries, package managers, and applications.

### Open source versus closed source

  ---------------------------------------------------------------------------
  Feature                 Open-source software    Closed-source/proprietary
                                                  software
  ----------------------- ----------------------- ---------------------------
  Source code             Available under an      Usually not publicly
                          open-source licence     available

  Modification            Permitted by licence    Usually restricted

  Redistribution          Permitted under licence Usually restricted
                          conditions              

  Support                 Community, vendors, or  Usually vendor or
                          internal teams          contracted support

  Examples                Linux, PostgreSQL,      Microsoft Windows, many
                          Nginx, Python           commercial applications
  ---------------------------------------------------------------------------

**Important:** Open source does not automatically mean free of cost,
free of bugs, or free of security risks. Licence conditions still apply,
and software must be maintained and updated.

## Why is open source important in system administration?

-   **Transparency:** administrators can inspect how software works.
-   **Flexibility:** software can be configured for different
    environments.
-   **Automation:** command-line tools and scripts support repeatable
    administration.
-   **Community:** documentation, bug reports, and improvements are
    shared.
-   **Cost and choice:** organizations can choose from multiple
    distributions and support providers.
-   **Infrastructure:** Linux is widely used in servers, cloud
    platforms, containers, networking devices, and embedded systems.

### Scenario: A college web server

A college wants to host its website and student project applications. It
chooses a Linux server, Nginx as the web server, and PostgreSQL as the
database.

The college can inspect documentation, automate updates, choose a
supported distribution, and avoid depending on one software vendor for
every component.

**Discussion:** Open source does not remove the need for backups,
security updates, monitoring, and trained administrators.

------------------------------------------------------------------------

# 1.2 Unix System Architecture, Linux Differences, Using a Linux System, Command Syntax

## What is Unix?

Unix is a family of operating systems developed in the 1970s. It
introduced ideas that strongly influenced modern operating systems,
including multiuser operation, multitasking, hierarchical filesystems,
small command-line tools, and pipes.

Examples of Unix or Unix-like systems include BSD systems, macOS, and
Linux.

## What is Linux?

Linux is technically the **kernel**: the core component that manages
hardware resources and provides services to programs. In everyday
conversation, "Linux" often means a complete Linux distribution.

A Linux distribution (distro) combines the Linux kernel with system
utilities, libraries, a package manager, installers, and applications.

Examples:

-   Ubuntu
-   Debian
-   Fedora
-   Red Hat Enterprise Linux (RHEL)
-   Rocky Linux
-   AlmaLinux
-   openSUSE

## Unix and Linux: key differences

  -----------------------------------------------------------------------
  Unix                                Linux
  ----------------------------------- -----------------------------------
  Historically developed as a family  Open-source kernel first released
  of operating systems                in 1991

  Some Unix systems are proprietary;  Linux kernel is open source
  others are open source              

  Often associated with               Available in many community and
  vendor-specific systems and         enterprise distributions
  standards                           

  Examples include AIX, HP-UX, and    Examples include Ubuntu, Debian,
  BSD variants                        Fedora, and RHEL
  -----------------------------------------------------------------------

Linux is Unix-like, but Linux is not simply another name for the
original Unix operating system.

## Basic Linux architecture

Think of the system as layers:

``` text
+----------------------------------------------+
|                 USER                         |
+----------------------------------------------+
| Applications: browser, editor, database      |
+----------------------------------------------+
| Shell and system utilities: bash, ls, grep   |
+----------------------------------------------+
| Libraries and system-call interface          |
+----------------------------------------------+
|              LINUX KERNEL                   |
| Process | Memory | Device | Filesystem | Net  |
+----------------------------------------------+
| Hardware: CPU, RAM, disk, NIC, USB            |
+----------------------------------------------+
```

### Main components

1.  **Hardware:** CPU, RAM, storage, network cards, and other devices.
2.  **Kernel:** manages CPU scheduling, memory, devices, filesystems,
    and networking.
3.  **System libraries:** provide standard functions that applications
    can use.
4.  **Shell:** accepts commands and starts programs. Bash is a common
    shell.
5.  **Utilities and applications:** tools such as `ls`, `grep`, editors,
    web servers, and databases.
6.  **User:** interacts with the system through a terminal or graphical
    interface.

### What happens when a command runs?

When a user enters `ls -l /etc`:

1.  The shell reads the command.
2.  It interprets the command name and arguments.
3.  The shell locates the `ls` executable using its command search path
    (or a supplied path).
4.  The program requests filesystem information through operating-system
    interfaces.
5.  The kernel performs the required operations.
6.  The command prints the result to the terminal.

## Using a Linux system

You may access Linux through:

-   A local terminal on a Linux desktop.
-   SSH to a remote Linux server.
-   A virtual machine (VirtualBox, VMware, or similar).
-   WSL on Windows.
-   A browser-based lab such as KillerCoda.

### First inspection commands

Run these one by one:

``` bash
whoami
```

**Purpose:** displays the current username.

Example output:

``` text
student
```

``` bash
hostname
```

**Purpose:** displays the computer's hostname.

Example output:

``` text
linux-lab
```

``` bash
uname -a
```

**Purpose:** displays kernel and system information. Exact output varies
by system.

``` bash
cat /etc/os-release
```

**Purpose:** displays Linux distribution information.

Example output (shortened):

``` text
NAME="Ubuntu"
VERSION="24.04 LTS"
ID=ubuntu
```

``` bash
date
uptime
```

`date` displays the system date/time. `uptime` displays how long the
system has been running and a load summary.

## Linux command syntax

A common command structure is:

``` text
command [options] [arguments]
```

Example:

``` bash
ls -l /etc
```

-   `ls` = command
-   `-l` = option (long listing)
-   `/etc` = argument (target directory)

More examples:

``` bash
pwd
ls -a
mkdir practice
touch notes.txt
```

### Important command-line concepts

**Option:** changes how a command behaves. For example, `-l` in `ls -l`.

**Argument:** tells a command what item to work on. For example, `/etc`
in `ls /etc`.

**Absolute path:** begins at the root directory `/`. Example:
`/home/student/notes.txt`.

**Relative path:** interpreted from the current working directory.
Example: `notes.txt` or `../shared/report.txt`.

**Command case sensitivity:** Linux filenames and commands are generally
case-sensitive. `Report.txt` and `report.txt` are different names.

**Spaces in filenames:** quote the name or escape spaces.

``` bash
touch "class notes.txt"
ls "class notes.txt"
```

### Getting help

``` bash
man ls
ls --help
whatis ls
```

-   `man` opens the manual page.
-   `--help` often prints a short usage guide.
-   `whatis` gives a brief description if its manual database is
    available.

Inside a manual page, press `q` to quit. Use `/word` to search.

------------------------------------------------------------------------

# 1.3 File Management

Linux stores files and directories in a hierarchical tree. A directory
is similar to a folder; it can contain files and other directories.

## Navigation commands

  Command      Meaning
  ------------ ---------------------------------------
  `pwd`        Print current working directory
  `ls`         List directory contents
  `ls -l`      Long listing
  `ls -a`      Include hidden entries
  `ls -lh`     Human-readable file sizes
  `cd /path`   Change to a directory
  `cd ..`      Move to parent directory
  `cd ~`       Move to current user's home directory
  `cd -`       Return to previous directory

### Example and expected result

``` bash
pwd
```

Possible output:

``` text
/home/student
```

``` bash
mkdir linux-unit1
cd linux-unit1
pwd
```

Possible output:

``` text
/home/student/linux-unit1
```

## Create files and directories

``` bash
mkdir documents
mkdir -p projects/linux/week1
touch notes.txt
```

-   `mkdir documents` creates one directory.
-   `mkdir -p` creates missing parent directories as needed.
-   `touch` creates an empty file if it does not exist. If it exists, it
    updates its timestamps.

Write content into a file:

``` bash
echo "Linux is useful for server administration." > notes.txt
cat notes.txt
```

Expected output:

``` text
Linux is useful for server administration.
```

## Copy, move, rename, and remove

``` bash
cp notes.txt notes-backup.txt
```

Copies a file.

``` bash
mv notes-backup.txt documents/
```

Moves a file into a directory.

``` bash
mv notes.txt linux-notes.txt
```

Renames a file.

``` bash
rmdir empty-directory
```

Removes an empty directory.

``` bash
rm old-file.txt
```

Removes a file. There is usually no recycle bin in a terminal deletion.

``` bash
rm -r old-directory
```

Removes a directory and its contents recursively. Check the path
carefully before using it.

## Inspect files

``` bash
cat notes.txt
less notes.txt
head notes.txt
tail notes.txt
wc -l notes.txt
file notes.txt
stat notes.txt
```

-   `cat`: prints file content.
-   `less`: views long files page by page; press `q` to quit.
-   `head`: displays the first 10 lines by default.
-   `tail`: displays the last 10 lines by default.
-   `wc -l`: counts lines.
-   `file`: identifies file type.
-   `stat`: displays file metadata such as size, permissions, and
    timestamps.

### File management simulation

Run this block in your terminal:

``` bash
mkdir -p ~/unit1-lab/{reports,backup,logs}
cd ~/unit1-lab

echo "Linux lab started" > reports/day1.txt
echo "Student: Asha" >> reports/day1.txt
echo "Status: completed" >> reports/day1.txt

cp reports/day1.txt backup/day1-copy.txt
mv reports/day1.txt reports/linux-report.txt

echo "ERROR: disk warning" > logs/system.log
echo "INFO: service started" >> logs/system.log
echo "ERROR: login failed" >> logs/system.log

find . -type f
cat reports/linux-report.txt
ls -lhR .
```

**What should happen?**

-   Three directories are created.
-   `day1.txt` is created and receives three lines.
-   A copy is placed in `backup`.
-   The original report is renamed.
-   A log file is created with three entries.
-   `find` lists the files; `cat` displays the report; `ls -lhR` lists
    the tree recursively.

------------------------------------------------------------------------

# 1.4 Shell Commands

## What is a shell?

A shell is a command interpreter. It accepts commands from the user or a
script and starts programs. Bash (Bourne Again Shell) is widely used on
Linux.

Check your current shell:

``` bash
echo "$SHELL"
ps -p $$ -o comm=
```

The first command usually shows the user's configured login shell. The
second shows the shell process currently running.

## Useful everyday commands

  Command        Purpose                             Example
  -------------- ----------------------------------- ----------------------
  `clear`        Clear terminal display              `clear`
  `echo`         Print text or variable values       `echo "Hello"`
  `history`      Show command history                `history`
  `which`        Locate a command in PATH            `which grep`
  `type`         Explain how shell resolves a name   `type cd`
  `hostname`     Show hostname                       `hostname`
  `id`           Show user and group IDs             `id`
  `who`          Show logged-in users                `who`
  `free -h`      Show memory usage                   `free -h`
  `df -h`        Show filesystem space               `df -h`
  `du -sh DIR`   Show directory's total size         `du -sh ~/Downloads`

## Wildcards (globbing)

Wildcards help match filenames.

``` bash
ls *.txt
ls report?.txt
ls [ab]*.log
```

-   `*` matches zero or more characters.
-   `?` matches one character.
-   `[ab]` matches one character from the set `a` or `b`.

These patterns are normally expanded by the shell before the command
runs.

## Standard input, output, and errors

Linux commands commonly use three streams:

  Stream              Number Meaning
  ----------------- -------- -----------------------
  Standard input           0 Input to a command
  Standard output          1 Normal command output
  Standard error           2 Error messages

### Output redirection

``` bash
echo "first line" > sample.txt
echo "second line" >> sample.txt
cat sample.txt
```

-   `>` creates/overwrites a file.
-   `>>` appends to a file.

### Input redirection

``` bash
wc -l < sample.txt
```

The file is supplied as standard input.

### Redirect errors

``` bash
ls /directory-that-does-not-exist 2> error.txt
cat error.txt
```

### Redirect output and errors

``` bash
some-command > output.txt 2> errors.txt
```

To send both standard output and standard error to one file in Bash:

``` bash
some-command > all-output.txt 2>&1
```

## Pipes

A pipe `|` sends the standard output of one command to the standard
input of another.

``` bash
ls /etc | less
```

``` bash
ps aux | grep ssh
```

``` bash
cat /etc/passwd | wc -l
```

The last example counts lines in `/etc/passwd`. The file contains
account records; it does not contain users' login passwords.

**Remember:** `>` writes to a file; `|` sends output to another command.

------------------------------------------------------------------------

# 1.5 Text Processing

System administrators often inspect logs, configuration files, user
records, and command output. Text-processing tools help find patterns
and extract useful information.

## Prepare a sample log

``` bash
mkdir -p ~/unit1-lab/text
cd ~/unit1-lab/text

cat > server.log <<'EOF'
2026-11-14 08:00:01 INFO nginx started
2026-11-14 08:05:11 ERROR database connection failed
2026-11-14 08:07:25 WARNING disk usage is high
2026-11-14 08:10:00 INFO user student logged in
2026-11-14 08:15:42 ERROR nginx upstream timeout
2026-11-14 08:20:10 INFO backup completed
EOF
```

Check it:

``` bash
cat server.log
```

Expected output:

``` text
2026-11-14 08:00:01 INFO nginx started
2026-11-14 08:05:11 ERROR database connection failed
2026-11-14 08:07:25 WARNING disk usage is high
2026-11-14 08:10:00 INFO user student logged in
2026-11-14 08:15:42 ERROR nginx upstream timeout
2026-11-14 08:20:10 INFO backup completed
```

## A. `grep` --- search for matching lines

Syntax:

``` text
grep [options] "pattern" filename
```

Examples:

``` bash
grep "ERROR" server.log
```

Expected output:

``` text
2026-11-14 08:05:11 ERROR database connection failed
2026-11-14 08:15:42 ERROR nginx upstream timeout
```

Useful options:

``` bash
grep -i "error" server.log       # case-insensitive
grep -n "ERROR" server.log       # show line numbers
grep -v "INFO" server.log        # lines that do not contain INFO
grep -c "ERROR" server.log       # count matching lines
grep -E "ERROR|WARNING" server.log
```

Search recursively:

``` bash
grep -R "PermitRootLogin" /etc/ssh
```

Permission errors may appear when searching protected directories; use
elevated privileges only when you are authorized and need them.

## B. `awk` --- process columns and fields

`awk` is useful when text is arranged in fields separated by spaces or
another delimiter.

Example:

``` bash
awk '{print $1, $3}' server.log
```

Expected output:

``` text
2026-11-14 08:00:01
2026-11-14 08:05:11
2026-11-14 08:07:25
2026-11-14 08:10:00
2026-11-14 08:15:42
2026-11-14 08:20:10
```

Here `$1` means the first field and `$3` means the third field. Because
the timestamp contains a space, the fields are date, time, level, and
remaining message words.

Print only the log level:

``` bash
awk '{print $3}' server.log
```

Expected output:

``` text
INFO
ERROR
WARNING
INFO
ERROR
INFO
```

Print only error records:

``` bash
awk '$3 == "ERROR" {print}' server.log
```

Count error records:

``` bash
awk '$3 == "ERROR" {count++} END {print count}' server.log
```

Expected output:

``` text
2
```

Use a comma-separated file:

``` bash
cat > students.csv <<'EOF'
name,course,marks
Asha,ICT,85
Bikash,ICT,72
Mina,Networking,91
Ravi,ICT,64
EOF
```

Print names and marks (skip header):

``` bash
awk -F, 'NR > 1 {print $1, $3}' students.csv
```

Expected output:

``` text
Asha 85
Bikash 72
Mina 91
Ravi 64
```

`-F,` sets the field separator to a comma. `NR` is the current
record/line number.

## C. `sed` --- stream editor

`sed` can search and transform text as it is read.

Replace the first occurrence on each line:

``` bash
sed 's/INFO/NOTICE/' server.log
```

Replace every occurrence on each line:

``` bash
sed 's/INFO/NOTICE/g' server.log
```

Delete lines containing `WARNING` from the displayed output:

``` bash
sed '/WARNING/d' server.log
```

Show only lines 2 to 4:

``` bash
sed -n '2,4p' server.log
```

**Important:** These commands print modified output; they do not change
the original file unless you use in-place editing. Practise on a copy
before using `sed -i`.

## D. Other useful text tools

``` bash
sort students.csv
sort -n marks.txt
uniq names.txt
sort names.txt | uniq
wc -l server.log
cut -d, -f1 students.csv
tr 'a-z' 'A-Z' < names.txt
```

-   `sort`: sorts lines.
-   `sort -n`: sorts numerically.
-   `uniq`: removes adjacent duplicate lines; sort first if duplicates
    may be separated.
-   `wc`: counts lines, words, and bytes/characters.
-   `cut`: extracts fields or character positions.
-   `tr`: translates or deletes characters.

## Text-processing pipeline simulation

Count how many records exist for each log level:

``` bash
awk '{print $3}' server.log | sort | uniq -c
```

Possible output:

``` text
      2 ERROR
      3 INFO
      1 WARNING
```

Find error records and save them:

``` bash
grep "ERROR" server.log > errors.log
cat errors.log
```

Count them:

``` bash
grep -c "ERROR" server.log
```

Expected output:

``` text
2
```

**Concept:** `awk` extracts the level, `sort` groups equal values
together, and `uniq -c` counts each group.

------------------------------------------------------------------------

# 1.6 Job Control

## What is a process?

A process is a running instance of a program. Each process has a process
ID (PID). Linux uses processes for services, commands, shells, and
applications.

## Inspect processes

``` bash
ps
ps -ef
ps aux
```

Common meanings:

-   `ps`: processes associated with the current terminal.
-   `ps -ef`: a full-format process listing.
-   `ps aux`: a BSD-style listing of processes.

Find a process:

``` bash
ps aux | grep nginx
```

This may also show the `grep` command itself. A more focused alternative
on many Linux systems is:

``` bash
pgrep -a nginx
```

Monitor processes interactively:

``` bash
top
```

Press `q` to quit. If installed, `htop` provides an interactive
alternative.

## Foreground and background jobs

A foreground job occupies the terminal until it finishes or is
suspended.

Start a long-running command:

``` bash
sleep 300
```

Press `Ctrl+C` to interrupt it.

Start it in the background:

``` bash
sleep 300 &
```

The shell prints a job number and PID. The prompt returns immediately.

List jobs started by the current shell:

``` bash
jobs
```

Suspend a foreground job:

1.  Run `sleep 300`.
2.  Press `Ctrl+Z`.
3.  Run `jobs`.

Resume in the background:

``` bash
bg
```

Bring a job to the foreground:

``` bash
fg
```

If multiple jobs exist, specify a job number, for example:

``` bash
fg %1
```

## Stop a process

Find its PID:

``` bash
pgrep -a sleep
```

Send a normal termination request:

``` bash
kill PID
```

Replace `PID` with the actual process ID printed on your system.

Check whether it is still running:

``` bash
ps -p PID
```

If a process does not stop, investigate why before considering a
forceful signal:

``` bash
kill -TERM PID
```

`kill -9 PID` sends SIGKILL and should be a last resort because the
process cannot clean up.

## Practical job-control simulation

Open a terminal and run:

``` bash
sleep 600 &
sleep 700 &
jobs
ps -ef | grep '[s]leep'
```

Bring the first job to the foreground:

``` bash
fg %1
```

Press `Ctrl+Z` to suspend it, then:

``` bash
jobs
bg %1
jobs
```

Terminate the background jobs:

``` bash
kill %1
kill %2
jobs
```

Job numbers are assigned by your shell and may differ. If the jobs have
already finished, `kill` may report that the job does not exist.

### Process versus job

-   **Process:** a running program identified by a PID.
-   **Job:** a command or pipeline managed by the current shell,
    identified by a job number such as `%1`.
-   One job can contain more than one process.

------------------------------------------------------------------------

# 1.7 Filesystem Concepts

## What is a filesystem?

A filesystem is the method and data structure used by an operating
system to organize files and directories on storage devices.

Examples include ext4, XFS, Btrfs, and FAT32.

## Linux directory hierarchy

Linux uses one directory tree beginning at `/`, called the root
directory.

  -----------------------------------------------------------------------
  Directory                           Typical purpose
  ----------------------------------- -----------------------------------
  `/`                                 Root of the entire directory tree

  `/home`                             Regular users' home directories

  `/root`                             Root user's home directory

  `/etc`                              System-wide configuration files

  `/var`                              Variable data such as logs, caches,
                                      and spool files

  `/var/log`                          System and application logs

  `/tmp`                              Temporary files

  `/usr`                              Most user-space applications,
                                      libraries, and shared data

  `/bin`                              Essential user commands; often
                                      linked to `/usr/bin` on modern
                                      systems

  `/sbin`                             System administration commands;
                                      often linked to `/usr/sbin`

  `/dev`                              Device files

  `/proc`                             Virtual filesystem exposing process
                                      and kernel information

  `/sys`                              Virtual filesystem exposing device
                                      and kernel information

  `/boot`                             Bootloader and kernel-related files

  `/mnt`                              Common temporary mount point

  `/media`                            Common mount location for removable
                                      media
  -----------------------------------------------------------------------

## Absolute and relative paths

``` bash
pwd
cd /var/log
cd ..
cd ~/Documents
```

-   `/var/log` is an absolute path.
-   `../` means the parent directory.
-   `.` means the current directory.
-   `~` represents the current user's home directory.

## Disk and directory usage

``` bash
df -h
```

Shows used and available space on mounted filesystems.

``` bash
df -h /
```

Shows the filesystem containing `/`.

``` bash
du -sh /var/log
```

Shows the total size of `/var/log` (permissions may restrict access).

``` bash
du -h --max-depth=1 ~
```

Shows the sizes of immediate subdirectories in the home directory on GNU
`du`.

## Identify disks and mounts

``` bash
lsblk
lsblk -f
findmnt
mount
```

-   `lsblk`: lists block devices.
-   `lsblk -f`: also shows filesystem information.
-   `findmnt`: displays mounted filesystems.
-   `mount`: displays or manages mounts depending on arguments and
    privileges.

**Important distinction:**

-   A **disk** is a storage device.
-   A **partition** is a defined region of a disk.
-   A **filesystem** organizes data within a storage area.
-   A **mount point** is a directory through which a filesystem is
    accessed.

Do not format or mount a real disk in a shared or production machine as
part of this introductory exercise. Use a disposable VM and
instructor-provided virtual disks for storage labs.

## Find files

``` bash
find ~ -name "*.txt"
find /var/log -type f
find ~ -type f -size +1M
```

-   `-name` searches by name.
-   `-type f` searches regular files.
-   `-size +1M` searches files larger than approximately 1 MiB.
-   Search results depend on permissions and the files present.

------------------------------------------------------------------------

# 1.8 Package Management

## What is a package?

A package is a software bundle containing program files, metadata, and
sometimes configuration or dependency information. A package manager
helps install, update, query, and remove software.

Package management saves administrators from manually locating every
required file and dependency.

## Common package management systems

  Distribution family              Package format/tool   Common commands
  -------------------------------- --------------------- -----------------
  Debian, Ubuntu                   `.deb`, APT           `apt`, `dpkg`
  Fedora, RHEL, Rocky, AlmaLinux   `.rpm`, DNF           `dnf`, `rpm`
  openSUSE                         `.rpm`, Zypper        `zypper`
  Arch Linux                       `pacman`              `pacman`

Use the package manager appropriate for your distribution.

## Identify your distribution

``` bash
cat /etc/os-release
```

## Ubuntu/Debian examples

Refresh package metadata:

``` bash
sudo apt update
```

Upgrade installed packages:

``` bash
sudo apt upgrade
```

Search for a package:

``` bash
apt search tree
```

Show package information:

``` bash
apt show tree
```

Install:

``` bash
sudo apt install tree
```

Check the command:

``` bash
tree --version
```

Remove a package:

``` bash
sudo apt remove tree
```

`remove` generally leaves package configuration files; `purge` removes
package configuration too. Do not use purge on an important system
package without understanding the consequences.

## Fedora/RHEL-family examples

Refresh metadata and upgrade:

``` bash
sudo dnf check-update
sudo dnf upgrade
```

Search and inspect:

``` bash
dnf search tree
dnf info tree
```

Install and remove:

``` bash
sudo dnf install tree
sudo dnf remove tree
```

## Package-management precautions

1.  Check which distribution you are using.
2.  Read the package name and transaction summary before confirming.
3.  Install software from trusted repositories.
4.  Do not interrupt package operations unnecessarily.
5.  Keep backups and plan updates on production servers.
6.  Remember that installing a package may start or enable a service
    only if its package scripts or your configuration do so; verify
    service state separately.

------------------------------------------------------------------------

# 3. Integrated Practical Lab: College Linux Server Preparation

## Scenario

A college is preparing a Linux server for an introductory ICT class. The
administrator must create a directory structure for student reports,
generate a sample service log, find errors, inspect running processes,
check disk space, and install a small utility.

**Objective:** Practise the unit's commands in one realistic workflow.

**Assumption:** You are logged in as a normal lab user. Use `sudo` only
for package installation if your lab account is authorized.

## Task 1 --- Identify the system

``` bash
whoami
hostname
uname -r
cat /etc/os-release
uptime
```

**Expected result:** You should see your username, hostname, kernel
release, distribution details, and uptime. Exact values depend on the
lab machine.

## Task 2 --- Build the college directory structure

``` bash
mkdir -p ~/college-server/{students,reports,logs,backup}
cd ~/college-server
pwd
find . -type d
```

**Expected directory tree:**

``` text
college-server/
├── backup/
├── logs/
├── reports/
└── students/
```

## Task 3 --- Create student records

``` bash
cat > students/students.csv <<'EOF'
name,department,marks
Asha,ICT,85
Bikash,ICT,72
Mina,Networking,91
Ravi,ICT,64
Sita,Networking,78
EOF

cat students/students.csv
```

Expected content:

``` text
name,department,marks
Asha,ICT,85
Bikash,ICT,72
Mina,Networking,91
Ravi,ICT,64
Sita,Networking,78
```

Find ICT students:

``` bash
awk -F, 'NR > 1 && $2 == "ICT" {print $1, $3}' students/students.csv
```

Expected output:

``` text
Asha 85
Bikash 72
Ravi 64
```

Find students with marks at least 80:

``` bash
awk -F, 'NR > 1 && $3 >= 80 {print $1, $3}' students/students.csv
```

Expected output:

``` text
Asha 85
Mina 91
```

## Task 4 --- Generate and investigate a service log

``` bash
cat > logs/college-service.log <<'EOF'
08:00 INFO Web service started
08:05 INFO Student portal available
08:10 ERROR Database connection failed
08:12 WARNING Disk usage reached 80 percent
08:15 ERROR Login service timeout
08:20 INFO Backup completed
EOF
```

Find errors:

``` bash
grep "ERROR" logs/college-service.log
```

Expected output:

``` text
08:10 ERROR Database connection failed
08:15 ERROR Login service timeout
```

Count errors:

``` bash
grep -c "ERROR" logs/college-service.log
```

Expected output:

``` text
2
```

Extract log levels and count them:

``` bash
awk '{print $2}' logs/college-service.log | sort | uniq -c
```

Expected output:

``` text
      2 ERROR
      1 WARNING
      3 INFO
```

Save error lines to a separate report:

``` bash
grep "ERROR" logs/college-service.log > reports/error-report.txt
cat reports/error-report.txt
```

## Task 5 --- Back up the records

``` bash
cp students/students.csv backup/
cp logs/college-service.log backup/
ls -lh backup/
```

Expected result: the backup directory contains copies of the CSV file
and service log. File sizes and timestamps vary.

## Task 6 --- Inspect processes

``` bash
ps -ef | head
top
```

Press `q` to exit `top`.

Run a test background job:

``` bash
sleep 300 &
jobs
```

Find it:

``` bash
pgrep -a sleep
```

Terminate it using its actual PID:

``` bash
kill PID
```

Replace `PID` with the value returned by `pgrep`. Confirm with:

``` bash
pgrep -a sleep
```

No output normally means no matching `sleep` process remains.

## Task 7 --- Check filesystem usage

``` bash
df -h
du -sh ~/college-server
lsblk
findmnt
```

**Expected result:** commands display the actual lab machine's mounted
filesystems, directory size, block devices, and mount information. These
values will differ across machines.

## Task 8 --- Install a utility

On Ubuntu/Debian:

``` bash
sudo apt update
sudo apt install tree
tree ~/college-server
```

On Fedora/RHEL-family systems:

``` bash
sudo dnf install tree
tree ~/college-server
```

Expected directory view (file details may vary):

``` text
college-server
├── backup
│   ├── college-service.log
│   └── students.csv
├── logs
│   └── college-service.log
├── reports
│   └── error-report.txt
└── students
    └── students.csv
```

If `tree` is unavailable or you cannot install packages, use:

``` bash
find ~/college-server -print
```

## Lab completion checklist

-   [ ] Identified Linux distribution and kernel.
-   [ ] Created the college directory structure.
-   [ ] Created and inspected a CSV file.
-   [ ] Used `awk` to filter student records.
-   [ ] Used `grep` to find errors.
-   [ ] Used a pipeline with `sort` and `uniq`.
-   [ ] Redirected error records into a report.
-   [ ] Backed up files using `cp`.
-   [ ] Inspected processes with `ps` and `top`.
-   [ ] Started and stopped a background job.
-   [ ] Checked disk and directory usage.
-   [ ] Installed or investigated a package using the correct package
    manager.

------------------------------------------------------------------------

# 4. Troubleshooting Guide

  ---------------------------------------------------------------------------------
  Problem                       Possible reason         What to check
  ----------------------------- ----------------------- ---------------------------
  `command not found`           Command is not          `command -v commandname`,
                                installed or not in     then check package manager
                                `PATH`                  

  `Permission denied`           User lacks permission   `ls -l path`, `id`, and
                                                        directory permissions

  `No such file or directory`   Wrong path or filename  `pwd`, `ls`, and
                                                        spelling/case

  `Is a directory`              A command expected a    Check the command syntax
                                file or a different     and target type
                                target                  

  File is empty                 Used `>` and overwrote  `cat file`, inspect
                                it, or command produced previous command
                                no output               

  `apt: command not found`      Not a Debian/Ubuntu     Read `/etc/os-release`; use
                                system                  the correct package manager

  Process is not found          It finished or PID/job  `jobs`, `pgrep -a name`,
                                number is wrong         `ps -ef`

  `grep` finds nothing          Pattern/case/path does  `grep -i`, inspect file
                                not match               with `cat`, verify path

  Package install fails         No network, repository, Read the full error; check
                                permission, or lock     network and package manager
                                issue                   state
  ---------------------------------------------------------------------------------

### A safe troubleshooting sequence

When a command fails:

1.  Read the complete error message.
2.  Check the current directory using `pwd`.
3.  Check the filename/path using `ls`.
4.  Check command spelling and syntax using `man` or `--help`.
5.  Check permissions using `ls -l` and `id`.
6.  Check whether the required package exists.
7.  Make one correction at a time and rerun the command.

------------------------------------------------------------------------

# 5. End-of-Unit Case Study: The College Portal Incident

## Situation

A college runs a small student portal on a Linux server. At 10:00 AM,
students report that the portal is slow. The administrator receives this
sample information:

-   The service log contains `ERROR` and `WARNING` records.
-   The portal's report directory contains student CSV files.
-   A backup must be created before changes are made.
-   The administrator must inspect processes and filesystem usage.
-   A text report must be prepared for the instructor.

You are the junior Linux administrator. Use the commands learned in Unit
1 to investigate. This is a **simulation**: the files and process are
generated for practice; they are not a real web server incident.

## Part A --- Create the simulated environment

``` bash
mkdir -p ~/portal-incident/{logs,reports,data,backup}
cd ~/portal-incident

cat > logs/portal.log <<'EOF'
10:00:01 INFO Portal service started
10:01:10 INFO User login successful
10:02:15 WARNING Memory usage is high
10:03:20 ERROR Database connection timeout
10:04:05 INFO Retry initiated
10:05:40 ERROR Database connection failed
10:06:10 WARNING Disk usage is high
10:07:00 INFO Administrator notified
EOF

cat > data/students.csv <<'EOF'
name,department,marks
Asha,ICT,85
Bikash,ICT,72
Mina,Networking,91
Ravi,ICT,64
Sita,Networking,78
EOF

cat > reports/notes.txt <<'EOF'
Incident: College portal slow
Reported: 10:00 AM
Investigation: Pending
EOF
```

## Part B --- Investigate the logs

1.  Display the complete log.
2.  Find all errors.
3.  Count the errors.
4.  Find warnings.
5.  Display only the time and severity fields.
6.  Save errors into `reports/errors.txt`.

Commands to try:

``` bash
cat logs/portal.log
grep "ERROR" logs/portal.log
grep -c "ERROR" logs/portal.log
grep "WARNING" logs/portal.log
awk '{print $1, $2}' logs/portal.log
grep "ERROR" logs/portal.log > reports/errors.txt
```

**Check your result:** there should be two `ERROR` records and two
`WARNING` records.

## Part C --- Analyze student data

Display ICT students:

``` bash
awk -F, 'NR > 1 && $2 == "ICT" {print $1, $3}' data/students.csv
```

Display students scoring 80 or above:

``` bash
awk -F, 'NR > 1 && $3 >= 80 {print $1, $3}' data/students.csv
```

Count data rows, excluding the header:

``` bash
awk 'END {print NR-1}' data/students.csv
```

Expected output:

``` text
5
```

## Part D --- Back up before making changes

``` bash
cp -a logs reports data backup/
find backup -type f
```

Explain why a backup should be created before editing or deleting
incident-related records.

## Part E --- Simulate a slow process

Start a harmless test process:

``` bash
sleep 900 &
jobs
ps -ef | grep '[s]leep'
```

Inspect it with:

``` bash
pgrep -a sleep
```

Stop it using the actual PID:

``` bash
kill PID
```

**Do not kill real production services during this exercise.** The
`sleep` process is only a safe stand-in for a long-running process.

## Part F --- Check storage and system information

``` bash
df -h
du -sh ~/portal-incident
free -h
uptime
```

Write down what each command tells you. Do not assume that high disk
usage or high memory use is proven by the simulated log; verify actual
system measurements.

## Part G --- Prepare the incident summary

Create a report:

``` bash
cat > reports/incident-summary.txt <<'EOF'
COLLEGE PORTAL INCIDENT SUMMARY
================================
Issue: Portal reported as slow
Log errors: 2
Log warnings: 2
Backup: Created under backup/
Process check: Completed using ps and pgrep
Filesystem check: Completed using df and du
Conclusion: The log indicates database connection problems.
Further investigation of database and application services is required.
EOF

cat reports/incident-summary.txt
```

## Part H --- Final questions

Answer these in your lab notebook:

1.  What is the difference between Linux and a Linux distribution?
2.  What does the kernel do?
3.  What is the difference between an absolute and relative path?
4.  What is the difference between `>` and `>>`?
5.  How does a pipe differ from output redirection?
6.  When would you use `grep`, `awk`, and `sed`?
7.  What is the difference between a process ID and a shell job number?
8.  Why should `kill -9` not be the first choice?
9.  What is the difference between `df -h` and `du -sh`?
10. Why must an administrator use the correct package manager for the
    distribution?
11. Which evidence in the simulated incident suggests a database
    problem?
12. What additional checks would be needed before concluding that the
    database is the root cause?

## Case-study expected learning

A good administrator does not guess based on one warning. The
administrator:

-   Collects evidence from logs.
-   Filters and counts relevant messages.
-   Checks processes and resource usage.
-   Protects data with a backup.
-   Records findings in a report.
-   Identifies what is known and what still needs investigation.

The case study does **not** prove that the database is actually down. It
only provides log evidence that database connections failed. A real
investigation would also inspect service status, database logs, network
connectivity, disk and memory pressure, and application configuration.

------------------------------------------------------------------------

# 6. Quick Revision Cheat Sheet

``` bash
# SYSTEM INFORMATION
whoami                 # Current user
hostname               # Hostname
uname -a               # Kernel/system information
cat /etc/os-release    # Distribution information
uptime                 # System uptime/load

# NAVIGATION
pwd                    # Current directory
ls -lah                # List files, including hidden files
cd /path               # Change directory
cd ..                  # Parent directory
cd ~                   # Home directory

# FILES AND DIRECTORIES
mkdir test             # Create directory
mkdir -p a/b/c         # Create nested directories
touch file.txt         # Create empty file/update timestamp
cp a.txt b.txt         # Copy file
mv a.txt b.txt         # Move or rename
rm file.txt            # Remove file
rmdir emptydir         # Remove empty directory
rm -r directory        # Remove directory recursively
cat file.txt           # Display file

# SEARCH AND TEXT PROCESSING
grep "ERROR" file      # Search matching lines
grep -in "error" file  # Case-insensitive search with line numbers
awk '{print $1}' file  # Print first field
sed 's/old/new/g' file # Replace text in output
sort file              # Sort lines
sort file | uniq -c    # Count repeated adjacent lines
cut -d, -f1 file       # Extract first comma-separated field
wc -l file             # Count lines
head file              # First lines
tail file              # Last lines
tail -f file           # Follow a growing log; Ctrl+C to stop

# REDIRECTION AND PIPELINES
command > out.txt      # Overwrite output file
command >> out.txt     # Append output
command 2> err.txt     # Redirect errors
command1 | command2    # Pipe output to next command

# PROCESS AND JOB CONTROL
ps -ef                 # Process listing
ps aux                 # Process listing
top                    # Interactive process monitor
jobs                   # Shell jobs
command &              # Start command in background
Ctrl+Z                 # Suspend foreground job
bg                     # Resume job in background
fg                     # Bring job to foreground
kill PID               # Request process termination
pgrep -a name          # Find matching process

# FILESYSTEM
df -h                  # Mounted filesystem capacity
du -sh directory       # Directory size
lsblk                  # Block devices
lsblk -f               # Devices and filesystem types
findmnt                # Mounted filesystems
find /path -name '*.txt' # Find matching files

# PACKAGE MANAGEMENT
sudo apt update        # Refresh APT metadata
sudo apt install tree  # Install on Debian/Ubuntu
sudo dnf install tree  # Install on Fedora/RHEL family
```

------------------------------------------------------------------------

# 7. Short Assessment

## Multiple-choice questions

**1. Which component manages memory and CPU scheduling?**

A. Shell\
B. Kernel\
C. Text editor\
D. Package manager

**Answer:** B --- Kernel.

**2. Which command prints the current working directory?**

A. `ls`\
B. `cd`\
C. `pwd`\
D. `who`

**Answer:** C --- `pwd`.

**3. Which symbol appends output to a file?**

A. `>`\
B. `>>`\
C. `|`\
D. `<`

**Answer:** B --- `>>`.

**4. Which command searches for lines containing `ERROR`?**

A. `grep`\
B. `mkdir`\
C. `ps`\
D. `df`

**Answer:** A --- `grep`.

**5. Which command displays running processes interactively?**

A. `touch`\
B. `top`\
C. `mv`\
D. `pwd`

**Answer:** B --- `top`.

**6. Which command shows mounted filesystem capacity?**

A. `du`\
B. `df`\
C. `awk`\
D. `jobs`

**Answer:** B --- `df`.

**7. What does `command1 | command2` do?**

A. Deletes command1\
B. Sends command1's standard output to command2\
C. Runs command2 next week\
D. Saves output automatically

**Answer:** B.

**8. Which package manager is commonly used on Ubuntu?**

A. DNF\
B. APT\
C. Pacman\
D. Zypper

**Answer:** B --- APT.

## Practical assessment tasks

1.  Create `~/assessment/unit1` and subdirectories `files`, `logs`, and
    `backup`.
2.  Create a five-line log containing at least two `ERROR` entries.
3.  Use `grep` to save only error lines into a new file.
4.  Use `awk` to print selected columns from a CSV file.
5.  Use `sed` to replace one word in displayed output without changing
    the original file.
6.  Start `sleep 120` in the background, identify it, and terminate it.
7.  Display filesystem capacity and the size of your assessment
    directory.
8.  Identify the distribution and state the correct package manager.

## Suggested marking rubric (20 marks)

  Area                                            Marks
  -------------------------------------------- --------
  Linux/system identification and navigation          2
  File and directory operations                       4
  Text processing (`grep`, `awk`, `sed`)              5
  Process and job control                             3
  Filesystem inspection                               2
  Package-management knowledge                        2
  Case-study report and explanation                   2
  **Total**                                      **20**

------------------------------------------------------------------------

## Further reading

-   Linux manual pages: `man` command and locally installed manual
    pages.
-   GNU Coreutils manual: https://www.gnu.org/software/coreutils/manual/
-   GNU Grep manual: https://www.gnu.org/software/grep/manual/
-   GNU Awk manual: https://www.gnu.org/software/gawk/manual/
-   GNU Sed manual: https://www.gnu.org/software/sed/manual/
-   Ubuntu documentation: https://help.ubuntu.com/
-   Debian documentation: https://www.debian.org/doc/

**Instructor note:** Command output, process IDs, device names, package
versions, disk sizes, and service state depend on the student's
environment. The outputs in this guide are illustrative where explicitly
stated. Students should record their own lab output rather than copy
assumed system values.
