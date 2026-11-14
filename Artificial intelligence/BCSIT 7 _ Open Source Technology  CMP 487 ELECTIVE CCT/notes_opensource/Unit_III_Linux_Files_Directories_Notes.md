# UNIT III: FILES, DIRECTORIES AND BASIC COMMANDS IN LINUX

**Total teaching hours:** 4\
**Unit focus:** Understanding the Linux file system, managing files and
directories, controlling permissions and ownership, and using essential
system and text-processing utilities.

------------------------------------------------------------------------

## 1. Learning objectives

By the end of this unit, students should be able to:

1.  Explain the Linux directory hierarchy and identify standard
    directories.
2.  Navigate directories and create, copy, view, move, and remove files
    and directories.
3.  Read Linux file permissions and change permissions and ownership
    safely.
4.  Use basic utility commands to inspect system information and process
    text.

------------------------------------------------------------------------

# 2. Linux file structure and hierarchy

Linux organizes files and directories in a tree-like structure. The top
of this structure is the **root directory**, represented by `/`.

Unlike Windows, which commonly uses drive letters such as `C:` and `D:`,
Linux places storage devices and filesystems into one directory tree.

### Example directory tree

``` text
/
├── bin
├── boot
├── dev
├── etc
├── home
│   ├── ram
│   │   ├── Documents
│   │   └── Downloads
│   └── sita
├── lib
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin
├── srv
├── sys
├── tmp
├── usr
└── var
```

### Important standard directories

  -----------------------------------------------------------------------
  Directory               Purpose                 Example
  ----------------------- ----------------------- -----------------------
  `/`                     Root of the entire      All directories begin
                          Linux filesystem        here

  `/home`                 Personal directories of `/home/ram`
                          ordinary users          

  `/root`                 Home directory of the   `/root`
                          root administrator      

  `/etc`                  System-wide             `/etc/hostname`
                          configuration files     

  `/bin`                  Essential user commands `ls`, `cp`
                          (often linked to        
                          `/usr/bin`)             

  `/sbin`                 Essential system        `fsck`
                          administration commands 
                          (often linked to        
                          `/usr/sbin`)            

  `/usr`                  Applications, libraries `/usr/bin`
                          and shared read-only    
                          data                    

  `/var`                  Frequently changing     `/var/log`
                          data such as logs and   
                          caches                  

  `/tmp`                  Temporary files         Temporary application
                                                  files

  `/boot`                 Files needed to start   Kernel and bootloader
                          the system              files

  `/dev`                  Device files            `/dev/sda`
                          representing hardware   
                          and virtual devices     

  `/proc`                 Virtual filesystem      `/proc/cpuinfo`
                          containing process and  
                          kernel information      

  `/sys`                  Kernel and device       `/sys/class`
                          information             

  `/lib`                  Essential shared        System libraries
                          libraries (often linked 
                          to `/usr/lib`)          

  `/media`                Mount points commonly   USB drive
                          used for removable      
                          media                   

  `/mnt`                  Temporary/manual mount  Mounted test partition
                          point                   

  `/opt`                  Optional third-party    `/opt/appname`
                          software                

  `/srv`                  Data served by services Web or FTP service data

  `/run`                  Runtime state data      Process and service
                          since boot              information
  -----------------------------------------------------------------------

**Important:** The exact contents and layout can differ between Linux
distributions. Some modern distributions merge directories such as
`/bin` into `/usr/bin`.

### Absolute path and relative path

-   **Absolute path:** Starts from `/` and gives the complete location.
    -   Example: `/home/ram/Documents/report.txt`
-   **Relative path:** Starts from the current working directory.
    -   If the current directory is `/home/ram`, then
        `Documents/report.txt` refers to
        `/home/ram/Documents/report.txt`.

Special path symbols:

  Symbol   Meaning
  -------- -----------------------------------
  `.`      Current directory
  `..`     Parent directory
  `~`      Current user's home directory
  `/`      Root directory and path separator

Example:

``` bash
cd /home/ram/Documents
cd ..
cd ~
```

------------------------------------------------------------------------

# 3. Commands for file and directory handling

Commands are entered in a terminal. Linux commands are generally
case-sensitive: `Report.txt` and `report.txt` are different filenames.

## 3.1 `pwd` --- Print Working Directory

Displays the full path of the directory in which the user is currently
working.

``` bash
pwd
```

Possible output:

``` text
/home/ram/Documents
```

**Use:** Check your current location before creating or deleting files.

## 3.2 `cd` --- Change Directory

Moves from one directory to another.

``` bash
cd /home/ram/Documents
cd ..
cd ~
cd /
```

Examples:

-   `cd /etc` --- go to `/etc`.
-   `cd ..` --- move one level up.
-   `cd ~` --- go to your home directory.
-   `cd -` --- return to the previous working directory.

## 3.3 `ls` --- List Directory Contents

Displays files and directories.

``` bash
ls
ls -l
ls -a
ls -lh
ls -la
```

  -----------------------------------------------------------------------
  Option                              Meaning
  ----------------------------------- -----------------------------------
  `-l`                                Long listing, including
                                      permissions, owner, size and date

  `-a`                                Include hidden files (names
                                      beginning with `.`)

  `-h`                                Show file sizes in human-readable
                                      units when used with a size listing
  -----------------------------------------------------------------------

Example:

``` bash
ls -lh /home/ram/Documents
```

A long listing may look like:

``` text
-rw-r--r-- 1 ram ram 2.0K Nov 14 10:30 report.txt
drwxr-xr-x 2 ram ram 4.0K Nov 14 10:35 Projects
```

The first character indicates the type: `-` is a regular file and `d` is
a directory.

## 3.4 `mkdir` --- Make Directory

Creates a new directory.

``` bash
mkdir Projects
mkdir LinuxLab
mkdir -p Projects/Unit3/Notes
```

The `-p` option creates missing parent directories as needed and does
not report an error if the directory already exists.

## 3.5 `rmdir` --- Remove Empty Directory

Removes a directory only when it is empty.

``` bash
rmdir OldFolder
```

If the directory contains files, `rmdir` will not remove it.

## 3.6 `cp` --- Copy Files and Directories

Copies files or directories.

``` bash
cp report.txt report_backup.txt
cp report.txt /home/ram/Documents/
cp -r Projects Projects_backup
```

-   `cp source destination` copies a file.
-   `cp -r` copies a directory and its contents recursively.

**Example:** Make a backup before editing an important report.

``` bash
cp report.txt report.txt.bak
```

## 3.7 `rm` --- Remove Files and Directories

Deletes files.

``` bash
rm old_report.txt
rm -i old_report.txt
rm -r OldFolder
```

-   `rm` removes a file.
-   `rm -i` asks for confirmation before each removal.
-   `rm -r` removes a directory and its contents recursively.

**Safety warning:** `rm` can permanently delete data. Check the path
with `pwd` and `ls` before deleting. Avoid running destructive commands
with `sudo` unless absolutely necessary. Never experiment with commands
such as `rm -rf /`.

## 3.8 `file` --- Identify File Type

Examines a file and reports the type of data it contains. It does not
rely only on the filename extension.

``` bash
file report.txt
file photo.jpg
file /bin/ls
```

Example output:

``` text
report.txt: ASCII text
photo.jpg: JPEG image data
```

## 3.9 `cat` --- Display or Combine File Content

Displays the contents of a text file.

``` bash
cat notes.txt
cat file1.txt file2.txt
cat file1.txt file2.txt > combined.txt
```

The `>` operator redirects output into a file and replaces its existing
contents. Use `>>` to append instead:

``` bash
echo "New line" >> notes.txt
```

## 3.10 `more` and `less` --- View Long Text Files

These commands display text one screen at a time.

``` bash
more /var/log/syslog
less /var/log/syslog
```

Common navigation in `less`:

-   `Space` --- next page
-   `b` --- previous page
-   `/word` --- search for a word
-   `n` --- next search result
-   `q` --- quit

`less` supports convenient backward and forward navigation. `more` is a
simpler pager.

### Quick practice: create and manage a folder

Run these commands one by one:

``` bash
mkdir -p ~/LinuxLab/Unit3
cd ~/LinuxLab/Unit3
pwd
touch first.txt
ls -l
cp first.txt first_backup.txt
ls
rm first_backup.txt
ls
cd ..
rmdir Unit3
```

**Note:** The final `rmdir Unit3` will fail if `Unit3` is not empty. In
this example, `first.txt` remains inside it. Remove that file first if
you want to remove the directory:

``` bash
rm Unit3/first.txt
rmdir Unit3
```

------------------------------------------------------------------------

# 4. File permissions and ownership

Linux is a multi-user operating system. Permissions help control who can
read, modify or execute a file and who can access a directory.

## 4.1 Understanding permissions

Run:

``` bash
ls -l report.txt
```

Example:

``` text
-rwxr-xr-- 1 ram staff 1200 Nov 14 10:30 report.txt
```

The permission string is:

``` text
-  rwx  r-x  r--
|   |    |    |
|   |    |    └── Others
|   |    └────── Group
|   └─────────── Owner/user
└─────────────── File type
```

The three basic permissions are:

  -------------------------------------------------------------------------------
  Permission    Symbol            Numeric value For a file       For a directory
  ------------- ------------- ----------------- ---------------- ----------------
  Read          `r`                           4 View contents    List directory
                                                                 entries

  Write         `w`                           2 Modify contents  Create, delete
                                                                 or rename
                                                                 entries (subject
                                                                 to other
                                                                 controls)

  Execute       `x`                           1 Run as a         Enter/traverse
                                                program/script   the directory
  -------------------------------------------------------------------------------

A dash (`-`) means that permission is not granted.

### Example: `rwxr-xr--`

-   Owner: `rwx` = read, write and execute.
-   Group: `r-x` = read and execute.
-   Others: `r--` = read only.

## 4.2 `chmod` --- Change File Permissions

`chmod` changes the permissions of a file or directory.

### Method A: Numeric (octal) method

The values are added together:

-   Read = 4
-   Write = 2
-   Execute = 1

Examples:

  -------------------------------------------------------------------------------------
  Command                   Owner          Group          Others         Meaning
  ------------------------- -------------- -------------- -------------- --------------
  `chmod 755 script.sh`     7 = rwx        5 = r-x        5 = r-x        Owner has full
                                                                         access; others
                                                                         can read and
                                                                         execute

  `chmod 644 report.txt`    6 = rw-        4 = r--        4 = r--        Owner can
                                                                         edit; others
                                                                         can read

  `chmod 600 secret.txt`    6 = rw-        0 = ---        0 = ---        Only owner has
                                                                         read/write
                                                                         access

  `chmod 700 private_dir`   7 = rwx        0 = ---        0 = ---        Only owner can
                                                                         access the
                                                                         directory
  -------------------------------------------------------------------------------------

Example:

``` bash
touch report.txt
chmod 644 report.txt
ls -l report.txt
```

### Method B: Symbolic method

Symbolic permissions use:

-   `u` = owner/user
-   `g` = group
-   `o` = others
-   `a` = all users
-   `+` = add permission
-   `-` = remove permission
-   `=` = set exactly the specified permission

Examples:

``` bash
chmod u+x script.sh
chmod g-w report.txt
chmod o+r report.txt
chmod a+r public.txt
chmod u=rw,go=r report.txt
```

**Important:** Execute permission on a directory allows traversal
through it. A user generally needs execute permission on each parent
directory in a path to access an item inside it.

## 4.3 `chown` --- Change File Ownership

Changes the owner and optionally the group of a file or directory. It
normally requires administrator privileges.

``` bash
sudo chown sita report.txt
sudo chown sita:staff report.txt
sudo chown -R sita:staff Projects
```

-   `chown sita report.txt` --- change the owner to `sita`.
-   `chown sita:staff report.txt` --- change owner to `sita` and group
    to `staff`.
-   `chown -R` --- apply the change recursively to a directory and its
    contents.

Check ownership with:

``` bash
ls -l report.txt
```

**Practical example:** If an administrator creates a project folder but
the student account `ram` must manage it:

``` bash
sudo mkdir /srv/student-project
sudo chown ram:ram /srv/student-project
```

Only use `sudo` and ownership changes when you understand which account
should own the data.

------------------------------------------------------------------------

# 5. Basic Linux utility commands

## 5.1 `touch` --- Create an Empty File or Update Timestamp

``` bash
touch notes.txt
touch file1.txt file2.txt
```

If the file does not exist, `touch` creates an empty file. If it already
exists, it updates its modification/access timestamps (subject to system
behavior and permissions).

## 5.2 `date` --- Display Date and Time

``` bash
date
date "+%Y-%m-%d"
date "+%H:%M:%S"
```

Example output:

``` text
2026-11-14
```

Useful format symbols:

  Symbol   Meaning
  -------- -----------------------
  `%Y`     Four-digit year
  `%m`     Month
  `%d`     Day
  `%H`     Hour (24-hour format)
  `%M`     Minute
  `%S`     Second

## 5.3 `passwd` --- Change User Password

``` bash
passwd
```

Changes the current user's password. An administrator can set another
user's password:

``` bash
sudo passwd ram
```

Linux normally asks for the current password when a user changes their
own password. Password characters may not appear while typing; this is
normal.

## 5.4 `finger` --- Display User Information

Where installed, `finger` displays information about local or remote
user accounts.

``` bash
finger
finger ram
```

**Note:** `finger` is not installed by default on many modern Linux
distributions and may be disabled for security reasons. If the command
is unavailable, an administrator may need to install a suitable package.
For basic account information, use:

``` bash
whoami
id
```

## 5.5 `head` --- Display the Beginning of a File

By default, displays the first 10 lines.

``` bash
head report.txt
head -n 5 report.txt
```

Example: Display the first five records of a CSV file.

``` bash
head -n 5 students.csv
```

## 5.6 `tail` --- Display the End of a File

By default, displays the last 10 lines.

``` bash
tail report.txt
tail -n 5 report.txt
```

To monitor a log file as new lines are added:

``` bash
tail -f /var/log/syslog
```

Press `Ctrl+C` to stop following the file.

## 5.7 `cut` --- Extract Columns or Fields

Extracts selected parts of each line. It is useful for simple,
consistently delimited text.

Example file `students.csv`:

``` text
101,Ram,Butwal
102,Sita,Palpa
103,Hari,Nawalparasi
```

Commands:

``` bash
cut -d',' -f1 students.csv
cut -d',' -f2 students.csv
cut -d',' -f1,3 students.csv
```

-   `-d','` sets the delimiter to a comma.
-   `-f2` selects the second field.
-   `-f1,3` selects the first and third fields.

**Caution:** `cut` is not a full CSV parser. It may not correctly handle
CSV fields containing quoted commas.

## 5.8 `sort` --- Sort Lines

Sorts text lines alphabetically by default.

``` bash
sort names.txt
sort -r names.txt
sort -n marks.txt
sort -u names.txt
```

  Option   Meaning
  -------- -----------------------------------------------
  `-r`     Reverse the sort order
  `-n`     Numeric sort
  `-u`     Remove duplicate lines from the sorted output

Example:

``` bash
printf "45\n9\n100\n" > marks.txt
sort -n marks.txt
```

Output:

``` text
9
45
100
```

## 5.9 `grep` --- Search Text Using a Pattern

Searches lines that match a word or pattern.

``` bash
grep "Linux" notes.txt
grep -i "linux" notes.txt
grep -n "error" system.log
grep -r "password" ~/LinuxLab
```

  Option   Meaning
  -------- ----------------------------------------
  `-i`     Ignore uppercase/lowercase differences
  `-n`     Show line numbers
  `-r`     Search recursively in directories

Example:

``` bash
grep -i "failed" login.log
```

This displays lines containing `failed`, `FAILED`, `Failed`, and similar
case variations.

**Security note:** Avoid searching for or sharing real passwords,
private keys, tokens or other secrets. Use sample files for practice.

------------------------------------------------------------------------

# 6. Combining commands: practical examples

Linux commands can be combined using pipes (`|`) and redirection.

## Example 1: Find errors in a log

``` bash
grep -i "error" application.log
```

Searches for lines containing the word `error`, regardless of case.

## Example 2: Count matching log entries

``` bash
grep -i "error" application.log | wc -l
```

The pipe sends the output of `grep` to `wc -l`, which counts lines.

## Example 3: Extract and sort student names

Given `students.csv`:

``` text
101,Ram,Butwal
102,Sita,Palpa
103,Hari,Nawalparasi
```

Run:

``` bash
cut -d',' -f2 students.csv | sort
```

Output:

``` text
Hari
Ram
Sita
```

## Example 4: Save a directory listing to a file

``` bash
ls -lh ~/Documents > document_list.txt
```

The `>` operator replaces the destination file if it already exists. Use
`>>` to append output.

------------------------------------------------------------------------

# 7. Command summary

  Command    Main purpose                              Example
  ---------- ----------------------------------------- ----------------------------
  `pwd`      Show current directory                    `pwd`
  `cd`       Change directory                          `cd /etc`
  `ls`       List files                                `ls -la`
  `mkdir`    Create directory                          `mkdir Projects`
  `rmdir`    Remove empty directory                    `rmdir OldFolder`
  `cp`       Copy files/directories                    `cp a.txt b.txt`
  `rm`       Remove files/directories                  `rm a.txt`
  `file`     Identify file type                        `file report.pdf`
  `cat`      Display/combine text                      `cat notes.txt`
  `more`     Page through text                         `more log.txt`
  `less`     Navigate through text                     `less log.txt`
  `chmod`    Change permissions                        `chmod 644 report.txt`
  `chown`    Change ownership                          `sudo chown ram:ram a.txt`
  `touch`    Create empty file/update timestamp        `touch notes.txt`
  `finger`   Display user information (if installed)   `finger ram`
  `passwd`   Change password                           `passwd`
  `date`     Display date/time                         `date`
  `head`     Show first lines                          `head -n 5 log.txt`
  `tail`     Show last lines                           `tail -n 5 log.txt`
  `cut`      Extract fields/columns                    `cut -d',' -f2 data.csv`
  `sort`     Sort lines                                `sort names.txt`
  `grep`     Search matching lines                     `grep "error" log.txt`

------------------------------------------------------------------------

# 8. Hands-on laboratory exercise

**Scenario:** You are a Linux lab assistant. Create a folder for a
class, prepare a text file, make a backup, set permissions and inspect
its contents.

Run:

``` bash
mkdir -p ~/LinuxLab/ClassNotes
cd ~/LinuxLab/ClassNotes
pwd
touch linux_unit3.txt
echo "Linux manages files using a hierarchical directory structure." > linux_unit3.txt
echo "Permissions control access to files and directories." >> linux_unit3.txt
cat linux_unit3.txt
cp linux_unit3.txt linux_unit3_backup.txt
ls -lh
file linux_unit3.txt
chmod 644 linux_unit3.txt
ls -l linux_unit3.txt
head -n 1 linux_unit3.txt
tail -n 1 linux_unit3.txt
grep -i "permissions" linux_unit3.txt
```

### Student tasks

1.  Create a directory named `Assignment`.
2.  Inside it, create `student.txt`.
3.  Add your name and class to the file.
4.  Make a copy named `student_backup.txt`.
5.  Display both files using `ls -l`.
6.  Set `student.txt` permission to `600`.
7.  Identify its file type using `file`.
8.  Display the first line using `head`.
9.  Search for your class name using `grep`.
10. Explain why `student.txt` and `student_backup.txt` should not be
    confused with one another.

------------------------------------------------------------------------

# 9. Real-world case study: College server and student records

## Case title: Protecting student records and organizing departmental files on a Linux server

### Background

A college in Butwal uses a Linux server to store student records,
examination reports, class notes and departmental documents. The server
is maintained by a system administrator. Three staff members have
different responsibilities:

-   **Ram:** System administrator who manages the server.
-   **Sita:** Examination officer who prepares student result files.
-   **Hari:** Lecturer who needs to read class notes but must not change
    examination records.

The college stores files in the following locations:

``` text
/srv/college/
├── examination/
│   ├── semester1.csv
│   └── semester2.csv
├── notes/
│   ├── linux_unit3.txt
│   └── database_unit2.txt
└── backup/
```

Recently, the college experienced several problems:

1.  Staff members could not remember where examination files were
    stored.
2.  A lecturer accidentally modified an examination result file.
3.  There was no separate backup copy of the examination records.
4.  The administrator needed to find failed student records from a CSV
    file.
5.  The college wanted to identify when a report was last updated and
    check the server log for errors.

### Problems to solve

**Problem 1: Organize the directory structure.**\
Create the required `examination`, `notes` and `backup` directories
under `/srv/college`.

**Problem 2: Protect examination records.**\
Only the examination officer and authorized administrators should be
able to modify examination files. Other users should not be able to edit
them.

**Problem 3: Create a backup.**\
Copy `semester1.csv` into the backup directory.

**Problem 4: Find failed records.**\
Search the examination file for records containing the word `Failed`.

**Problem 5: Extract student names.**\
Assume the CSV file has the following format:

``` text
Roll,Name,Status
101,Ram,Pass
102,Sita,Failed
103,Hari,Pass
104,Gita,Failed
```

Extract the student names and sort them alphabetically.

**Problem 6: Inspect system logs.**\
Display the last five lines of the system log and search for lines
containing `error`.

**Problem 7: Verify the result.**\
Display the directory structure and permissions to check whether the
files are organized and protected.

------------------------------------------------------------------------

## 10. Case study solution: Commands and explanations

> These commands are an example solution. Run them on a practice system
> or test directory first. Creating folders under `/srv` and changing
> ownership normally requires administrator privileges.

### Solution 1: Create the directory structure

``` bash
sudo mkdir -p /srv/college/examination
sudo mkdir -p /srv/college/notes
sudo mkdir -p /srv/college/backup
```

**Explanation:** `mkdir -p` creates the directories and any missing
parent directories.

### Solution 2: Assign ownership and permissions

Assume the Linux account `sita` is the examination officer and the group
`exams` exists.

``` bash
sudo chown -R sita:exams /srv/college/examination
sudo chmod 750 /srv/college/examination
```

For the result file:

``` bash
sudo chown sita:exams /srv/college/examination/semester1.csv
sudo chmod 640 /srv/college/examination/semester1.csv
```

**Explanation:**

-   `sita` owns the examination file.
-   Members of the `exams` group can read it.
-   Other users have no permission on the file.
-   `750` on the directory gives the owner full access, the group
    read/traverse access, and others no access.
-   `640` on the file gives the owner read/write, the group read-only,
    and others no access.

**Important:** The group must exist, and authorized staff must be
members of it. For example, an administrator can add a user to the group
with:

``` bash
sudo usermod -aG exams sita
```

A user may need to log out and back in for new group membership to
apply. Do not add users to privileged groups without authorization.

### Solution 3: Back up the examination file

``` bash
sudo cp /srv/college/examination/semester1.csv /srv/college/backup/semester1_backup.csv
```

Verify:

``` bash
ls -lh /srv/college/backup
```

**Explanation:** `cp` creates a separate copy. A production backup plan
should also include off-server or otherwise protected backups; a second
copy on the same disk is not sufficient protection against disk failure.

### Solution 4: Search for failed records

``` bash
grep -i "Failed" /srv/college/examination/semester1.csv
```

**Explanation:** `grep` displays the lines containing the search term.

### Solution 5: Extract and sort student names

``` bash
cut -d',' -f2 /srv/college/examination/semester1.csv | tail -n +2 | sort
```

Expected output:

``` text
Gita
Hari
Ram
Sita
```

**Explanation:**

-   `cut -d',' -f2` extracts the second comma-separated field.
-   `tail -n +2` skips the CSV header.
-   `sort` arranges the names alphabetically.

### Solution 6: Inspect system logs

On many systemd-based Linux distributions:

``` bash
sudo tail -n 5 /var/log/syslog
sudo grep -i "error" /var/log/syslog
```

Some distributions use `/var/log/messages`, or store logs in the systemd
journal instead. On those systems, use:

``` bash
sudo journalctl -n 5
sudo journalctl | grep -i "error"
```

**Explanation:** `tail` shows recent log lines, while `grep` searches
for matching error messages. Access to some logs may require
administrator privileges.

### Solution 7: Verify directory permissions and files

``` bash
ls -l /srv/college/examination
ls -l /srv/college/backup
ls -ld /srv/college/examination
```

-   `ls -l` displays file permissions, ownership, size and modification
    time.
-   `ls -ld` displays information about the directory itself rather than
    listing its contents.

### Expected outcome

After applying the solution:

-   College files are organized into separate directories.
-   Examination records have restricted access.
-   A backup copy of the result file exists.
-   Failed records can be searched quickly.
-   Student names can be extracted and sorted.
-   Recent system logs can be inspected for errors.
-   The administrator can verify file ownership and permissions.

### Discussion questions

1.  Why is `/srv/college` a suitable example location for college
    service data?
2.  What is the difference between `/home/sita` and `/srv/college`?
3.  Why is `chmod 640` more appropriate for a restricted result file
    than `chmod 777`?
4.  What is the difference between `cp` and `mv`?
5.  Why should the college maintain backups outside the same physical
    disk?
6.  How does `grep` help an administrator investigate system problems?
7.  What could happen if every staff member had write permission on
    examination records?
8.  Why should `sudo` be used carefully?

------------------------------------------------------------------------

# 11. Important revision points

-   Linux uses a hierarchical filesystem beginning at `/`.
-   `/home` stores ordinary users' home directories; `/root` is the root
    administrator's home directory.
-   `pwd` shows the current location; `cd` changes location; `ls` lists
    contents.
-   `mkdir` creates directories; `rmdir` removes empty directories.
-   `cp` copies; `rm` deletes; `cat`, `more` and `less` display text.
-   `chmod` changes permissions; `chown` changes ownership.
-   Permission values are `r = 4`, `w = 2` and `x = 1`.
-   `head` displays the beginning of a file; `tail` displays the end.
-   `cut` extracts fields; `sort` orders lines; `grep` searches text.
-   Always verify paths and permissions before performing administrative
    or destructive operations.

**End of Unit III**
