# UNIT 5: WEB, EMAIL, DATABASE SERVICES, AND NETWORK SECURITY

**Teaching Hours:** 4 Theory + 4 Practical

## Learning Objectives

After studying this unit, students should be able to:

-   Explain how web, email, and database services operate on a network.
-   Configure a basic Apache web server and host a sample website.
-   Describe and configure the roles of Postfix and Dovecot in email
    delivery.
-   Perform basic MySQL administration and troubleshoot common
    connection problems.
-   Explain cryptography, SSL/TLS, SSH, firewalls, and Access Control
    Lists (ACLs).
-   Apply basic server-hardening practices and diagnose common service
    failures.

------------------------------------------------------------------------

## 5.1 Apache, HTTPD, and Email Operations

### 5.1.1 Web server and HTTP

A **web server** is software that receives requests from clients,
usually web browsers, and returns web content such as HTML pages,
images, CSS, JavaScript, or application responses.

A web server commonly uses:

-   **HTTP (Hypertext Transfer Protocol):** transfers web requests and
    responses.
-   **HTTPS:** HTTP protected by TLS encryption.
-   **TCP port 80:** the conventional HTTP port.
-   **TCP port 443:** the conventional HTTPS port.

A client enters a URL, the browser resolves the server's name through
DNS, establishes a connection, sends an HTTP request, and receives a
response.

#### Example: opening a website

When a user visits `http://example.test/`, the browser may send a
request similar to:

``` http
GET / HTTP/1.1
Host: example.test
```

The server may respond:

``` http
HTTP/1.1 200 OK
Content-Type: text/html

<h1>Welcome</h1>
```

The status code `200` indicates success. Other common status codes
include:

  -----------------------------------------------------------------------
  Code                    Meaning                 Example
  ----------------------- ----------------------- -----------------------
  200                     OK                      Page returned
                                                  successfully

  301                     Moved Permanently       Site redirects to a new
                                                  URL

  403                     Forbidden               Client is not permitted
                                                  to access a resource

  404                     Not Found               Requested page does not
                                                  exist

  500                     Internal Server Error   Server-side application
                                                  error

  503                     Service Unavailable     Service is temporarily
                                                  unable to handle
                                                  requests
  -----------------------------------------------------------------------

### 5.1.2 Apache HTTP Server

**Apache HTTP Server**, often called Apache or `httpd`, is open-source
web-server software. It can serve static content and work with
application technologies such as PHP.

#### Important Apache concepts

-   **Document root:** directory from which a website's files are
    served.
-   **Virtual host:** configuration that allows one server to host
    multiple websites.
-   **Configuration file:** contains server settings, modules,
    directories, and virtual hosts.
-   **Access and error logs:** record requests and operational problems.
-   **Module:** extends Apache functionality, such as TLS support or URL
    rewriting.

On Ubuntu/Debian, common paths include:

-   Main configuration: `/etc/apache2/apache2.conf`
-   Site configurations: `/etc/apache2/sites-available/`
-   Enabled sites: `/etc/apache2/sites-enabled/`
-   Default website directory: `/var/www/html/`
-   Logs: `/var/log/apache2/`

On RHEL-based systems, Apache is commonly called `httpd`; configuration
and log paths differ. Always verify paths for the Linux distribution
being used.

### 5.1.3 Install and start Apache on Ubuntu

``` bash
sudo apt update
sudo apt install apache2
sudo systemctl enable --now apache2
sudo systemctl status apache2
```

-   `apt update` refreshes package information.
-   `apt install` installs Apache.
-   `enable --now` configures automatic startup and starts the service.
-   `status` displays the service state.

Check whether Apache is listening on port 80:

``` bash
sudo ss -ltnp | grep ':80'
```

Test the local server:

``` bash
curl -I http://localhost
```

A successful response commonly includes `HTTP/1.1 200 OK` or another
successful HTTP status.

### 5.1.4 Host a sample website

Create a simple web page:

``` bash
sudo mkdir -p /var/www/html
sudo nano /var/www/html/index.html
```

Add the following HTML:

``` html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Networking Lab</title>
</head>
<body>
  <h1>Welcome to the Networking Lab</h1>
  <p>This page is served by Apache on Ubuntu.</p>
</body>
</html>
```

Save the file and test:

``` bash
curl http://localhost
```

Open `http://localhost/` in a browser on the same machine. From another
computer, use the server's reachable IP address, for example
`http://192.168.1.20/`, if routing and firewall rules permit access.

### 5.1.5 Apache virtual host example

A virtual host lets one Apache server serve different websites. For
example, `lab.example.test` can use its own document root.

Create the directory:

``` bash
sudo mkdir -p /var/www/lab.example.test
```

Create `/etc/apache2/sites-available/lab.example.test.conf`:

``` apache
<VirtualHost *:80>
    ServerName lab.example.test
    DocumentRoot /var/www/lab.example.test

    <Directory /var/www/lab.example.test>
        Require all granted
        AllowOverride None
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/lab-error.log
    CustomLog ${APACHE_LOG_DIR}/lab-access.log combined
</VirtualHost>
```

Enable and validate the site:

``` bash
sudo a2ensite lab.example.test.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

For a local classroom test, the client machine must resolve
`lab.example.test` to the server's IP address. A DNS record is the
normal solution; a hosts-file entry can be used in a controlled lab.

**Security note:** `Require all granted` allows access to the specified
directory. Do not use broad permissions such as `chmod -R 777` to fix
access problems. Set suitable ownership and permissions instead.

### 5.1.6 HTTPS and TLS

HTTPS protects HTTP communication using **Transport Layer Security
(TLS)**. TLS provides:

1.  **Confidentiality:** encryption helps prevent unauthorized reading.
2.  **Integrity:** detects unauthorized changes to data in transit.
3.  **Authentication:** certificates help the client verify the server's
    identity.

A server certificate is issued by a trusted Certificate Authority (CA),
or by an internal CA for a managed environment. A self-signed
certificate is useful for testing but is not automatically trusted by
browsers.

On Ubuntu, Apache TLS support can be enabled with:

``` bash
sudo a2enmod ssl
sudo apache2ctl configtest
sudo systemctl reload apache2
```

A certificate and private key must be configured in the HTTPS virtual
host. For public websites, use a trusted certificate and keep its
private key readable only by authorized system processes.

### 5.1.7 Email system overview

Email delivery involves several components:

-   **MUA (Mail User Agent):** application used by a person to read or
    compose email, such as a desktop or webmail client.
-   **MTA (Mail Transfer Agent):** transfers mail between mail servers.
-   **MDA (Mail Delivery Agent):** places received messages into a
    user's mailbox.
-   **Mailbox:** storage location for a user's messages.

#### Common email protocols and ports

  -----------------------------------------------------------------------
  Protocol                Purpose                 Common port
  ----------------------- ----------------------- -----------------------
  SMTP                    Transfers outgoing      25
                          email between mail      
                          systems                 

  SMTP submission         Client submission,      587
                          usually with            
                          authentication and      
                          encryption              

  SMTPS                   SMTP submission with    465
                          implicit TLS            

  IMAP                    Accesses and            143
                          synchronizes messages   
                          on a server             

  IMAPS                   IMAP with implicit TLS  993

  POP3                    Downloads messages from 110
                          a mailbox               

  POP3S                   POP3 with implicit TLS  995
  -----------------------------------------------------------------------

SMTP is used to send and relay messages. IMAP and POP3 are used by
clients to access received messages. Port numbers are conventional
defaults; the actual service configuration determines which ports are
active.

### 5.1.8 Postfix

**Postfix** is an MTA. It accepts messages from authorized clients,
queues them, and transfers them to the destination mail server. It can
also deliver messages locally through a configured delivery agent.

Typical message flow:

``` text
Mail client
   |
   | SMTP submission
   v
Postfix (outgoing mail server)
   |
   | SMTP transfer
   v
Recipient's mail server
   |
   v
Recipient mailbox
```

Postfix configuration is commonly stored in:

-   `/etc/postfix/main.cf`
-   `/etc/postfix/master.cf`

Useful commands:

``` bash
sudo apt install postfix
sudo systemctl status postfix
sudo postfix check
sudo postconf -n
sudo journalctl -u postfix
```

The installer may ask for a mail configuration type and system mail
name. Select settings appropriate to the lab. A real public mail server
also needs correct DNS records, a stable public IP, reverse DNS, TLS,
authentication, and spam-abuse controls.

### 5.1.9 Dovecot

**Dovecot** is commonly used to provide IMAP and POP3 access to
mailboxes. It is not a replacement for Postfix's role as an MTA.

A typical incoming-mail flow is:

``` text
Sender's mail server
       |
       | SMTP
       v
Postfix on recipient server
       |
       | local delivery / mailbox storage
       v
User's mailbox
       |
       | IMAP or POP3
       v
User's mail client
```

Useful checks on Ubuntu:

``` bash
sudo apt install dovecot-imapd
sudo systemctl status dovecot
sudo ss -ltnp | grep -E ':(143|993)\b'
```

A complete email setup requires compatible mailbox storage and
permissions, valid user accounts, secure authentication, and TLS. Avoid
exposing an unconfigured mail server to the public internet.

### 5.1.10 Email security and anti-spam measures

-   Require authentication for client submission.
-   Use TLS for client connections and protect private keys.
-   Prevent unauthorized open relaying.
-   Apply sensible connection and message-rate limits.
-   Use spam filtering and malware scanning where appropriate.
-   Publish SPF, DKIM, and DMARC records for a public sending domain.
-   Monitor mail queues and logs.
-   Keep mail software updated.

**SPF** identifies which sending hosts are authorized by a domain.
**DKIM** adds a cryptographic signature to messages. **DMARC** tells
receiving systems how to handle messages that fail domain-alignment
checks and provides reporting. These controls help reduce spoofing but
do not guarantee that every message is legitimate.

------------------------------------------------------------------------

## 5.2 MySQL Administration

### 5.2.1 What is a database server?

A database server stores, organizes, and provides controlled access to
data. **MySQL** is a relational database management system (RDBMS). It
stores data in tables made up of rows and columns and uses SQL to
define, retrieve, and modify data.

Example table:

  student_id   student_name   program
  ------------ -------------- ----------
  101          Asha           BSc CSIT
  102          Milan          BCA
  103          Ritu           BSc CSIT

A relational database can connect tables using keys. A **primary key**
uniquely identifies a row. A **foreign key** links a row to a related
row in another table.

### 5.2.2 Install and start MySQL on Ubuntu

``` bash
sudo apt update
sudo apt install mysql-server
sudo systemctl enable --now mysql
sudo systemctl status mysql
```

Connect locally using the system's configured authentication:

``` bash
sudo mysql
```

Check the server version:

``` sql
SELECT VERSION();
```

Exit the MySQL prompt:

``` sql
EXIT;
```

### 5.2.3 Basic SQL administration

Create a database and a table:

``` sql
CREATE DATABASE college;
USE college;

CREATE TABLE student (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(60) NOT NULL,
    program VARCHAR(40)
);
```

Insert sample data:

``` sql
INSERT INTO student (student_id, student_name, program)
VALUES
    (101, 'Asha', 'BSc CSIT'),
    (102, 'Milan', 'BCA'),
    (103, 'Ritu', 'BSc CSIT');
```

Retrieve records:

``` sql
SELECT * FROM student;
```

Retrieve only students in BSc CSIT:

``` sql
SELECT student_name
FROM student
WHERE program = 'BSc CSIT';
```

Update a record:

``` sql
UPDATE student
SET program = 'BSc Computer Science'
WHERE student_id = 101;
```

Delete a record:

``` sql
DELETE FROM student
WHERE student_id = 103;
```

Always use a suitable `WHERE` condition with `UPDATE` and `DELETE`.
Without it, the operation may affect every row.

### 5.2.4 Database users and privileges

Applications should connect using a dedicated database account with only
the privileges they need. Avoid using the database administrator account
for routine website operations.

Example:

``` sql
CREATE USER 'webapp'@'localhost'
IDENTIFIED BY 'Use-A-Strong-Unique-Password';

GRANT SELECT, INSERT, UPDATE, DELETE
ON college.*
TO 'webapp'@'localhost';

SHOW GRANTS FOR 'webapp'@'localhost';
```

The password shown is an example placeholder; replace it with a strong,
unique secret. Do not publish passwords in notes, source code
repositories, or screenshots.

To remove access when it is no longer required:

``` sql
REVOKE SELECT, INSERT, UPDATE, DELETE
ON college.*
FROM 'webapp'@'localhost';
```

### 5.2.5 MySQL backup and restore

A logical backup can be created with `mysqldump`:

``` bash
mysqldump -u webapp -p college > college_backup.sql
```

Restore into a database that already exists:

``` bash
mysql -u webapp -p college < college_backup.sql
```

For production systems, design a tested backup plan that considers
database size, consistency, retention, access controls, and recovery
time. A backup is useful only if it can be restored successfully.

### 5.2.6 MySQL network access

MySQL commonly uses TCP port **3306**. A database should not be exposed
publicly without a strong operational need and appropriate security
controls.

Useful commands:

``` bash
sudo ss -ltnp | grep ':3306'
sudo systemctl status mysql
sudo journalctl -u mysql
```

If an application and database run on the same machine, use a local
connection where possible. If remote access is required:

-   Bind MySQL to an appropriate private interface.
-   Permit access only from trusted application hosts.
-   Use dedicated accounts with limited privileges.
-   Use encrypted connections where appropriate.
-   Avoid opening port 3306 to the whole internet.

### 5.2.7 Common MySQL problems

  ---------------------------------------------------------------------------
  Symptom                 Possible cause          First checks
  ----------------------- ----------------------- ---------------------------
  Service will not start  Configuration error,    `systemctl status mysql`;
                          disk issue, or          journal logs
                          data-directory problem  

  Access denied           Wrong credentials or    Username, host component,
                          missing privileges      grants

  Connection refused      Service stopped, wrong  Service status and `ss`
                          address, or port not    
                          listening               

  Unknown database        Database name is        `SHOW DATABASES;`
                          incorrect or database   
                          was not created         

  Website cannot connect  Incorrect DB host,      Application configuration
                          port, credentials, or   and server logs
                          network rule            
  ---------------------------------------------------------------------------

------------------------------------------------------------------------

## 5.3 Cryptography, SSH, Firewall Configuration, and ACLs

### 5.3.1 Cryptography

**Cryptography** uses mathematical techniques to protect information and
communications.

Important terms:

-   **Plaintext:** original readable data.
-   **Ciphertext:** encrypted data.
-   **Encryption:** transforms plaintext into ciphertext using a key.
-   **Decryption:** transforms ciphertext back into plaintext using a
    key.
-   **Key:** secret or public/private value used by a cryptographic
    algorithm.
-   **Hash:** fixed-length digest generated from data; a secure hash is
    designed to make it difficult to recover the original input or find
    a different input with the same digest.
-   **Digital signature:** uses a private key to sign data; verification
    uses the corresponding public key.

#### Symmetric and asymmetric encryption

  -----------------------------------------------------------------------
  Type                    How it works            Example use
  ----------------------- ----------------------- -----------------------
  Symmetric               Same secret key is used Efficient encryption of
                          to encrypt and decrypt  large amounts of data

  Asymmetric              Uses a related          Key exchange, digital
                          public/private key pair signatures, identity
                                                  verification
  -----------------------------------------------------------------------

Modern secure protocols often combine both approaches: asymmetric
cryptography helps establish trust or agree on keys, while symmetric
encryption protects the session data efficiently.

A hash is not encryption. It is not intended to be decrypted. Passwords
should be stored using a dedicated password-hashing method with a salt,
not as plain text or a simple fast hash.

### 5.3.2 SSH (Secure Shell)

**SSH** provides encrypted remote access to a computer, commonly through
TCP port **22**. Administrators use it to manage Linux servers securely.

Basic connection:

``` bash
ssh username@192.168.1.20
```

Specify a different port:

``` bash
ssh -p 2222 username@192.168.1.20
```

Install and start the OpenSSH server on Ubuntu:

``` bash
sudo apt install openssh-server
sudo systemctl enable --now ssh
sudo systemctl status ssh
```

#### SSH key authentication

SSH key authentication uses a key pair:

-   **Private key:** stays on the client and must be protected.
-   **Public key:** can be placed on the server in the user's
    `~/.ssh/authorized_keys` file.

Generate a key pair:

``` bash
ssh-keygen -t ed25519
```

Copy the public key to the server:

``` bash
ssh-copy-id username@192.168.1.20
```

Then connect:

``` bash
ssh username@192.168.1.20
```

Protect the private key with an appropriate passphrase and file
permissions. Never send the private key to another person or place it on
a public server.

#### SSH hardening

Recommended measures include:

-   Prefer key-based authentication for administrative access.
-   Use strong account passwords where password login remains enabled.
-   Disable direct root login when operationally appropriate.
-   Limit SSH access to trusted users and networks.
-   Keep OpenSSH updated.
-   Use firewall rules to restrict access.
-   Keep a tested recovery path before changing SSH settings.

Configuration is commonly stored in `/etc/ssh/sshd_config`. After
editing, validate the configuration and keep an existing session open
until a new connection has been tested:

``` bash
sudo sshd -t
sudo systemctl reload ssh
```

Do not disable password authentication until key-based access has been
verified.

### 5.3.3 Firewalls

A **firewall** controls network traffic according to rules. Rules may
consider source and destination addresses, ports, protocols, connection
state, and network interfaces.

A firewall can:

-   Allow required traffic.
-   Block unwanted traffic.
-   Limit access to specific trusted hosts.
-   Reduce the exposed network services of a server.

A firewall does not replace software updates, authentication, or secure
application configuration.

#### Common Linux firewall tools

-   **iptables:** a command-line interface for configuring
    packet-filtering rules on Linux.
-   **firewalld:** a firewall management service that supports zones and
    runtime/permanent rules.
-   **UFW:** a simplified firewall management interface commonly used on
    Ubuntu.

Do not manage the same firewall rules through multiple tools without
understanding how they interact.

#### Example: UFW on Ubuntu

Check status:

``` bash
sudo ufw status verbose
```

Allow SSH before enabling the firewall on a remote server:

``` bash
sudo ufw allow OpenSSH
```

Allow HTTP and HTTPS:

``` bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Enable the firewall:

``` bash
sudo ufw enable
sudo ufw status numbered
```

If SSH uses a custom port, allow that port before enabling the firewall.
Otherwise, remote access may be lost.

#### Example: firewalld

Check status:

``` bash
sudo firewall-cmd --state
```

Allow HTTP permanently:

``` bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

List active rules:

``` bash
sudo firewall-cmd --list-all
```

The correct commands depend on the distribution and firewall manager in
use.

### 5.3.4 Access Control Lists (ACLs)

An **Access Control List** provides permissions for specific users or
groups beyond the traditional Linux owner/group/other permission model.

Traditional permissions example:

``` bash
chmod 640 report.txt
```

This gives the owner read/write permission, the group read permission,
and others no permission.

ACLs can grant additional access to a named user:

``` bash
setfacl -m u:student1:r report.txt
```

View ACLs:

``` bash
getfacl report.txt
```

Remove the named user's ACL entry:

``` bash
setfacl -x u:student1 report.txt
```

Set a default ACL on a directory so new items can inherit an access
rule:

``` bash
setfacl -m d:u:student1:rx /srv/class-materials
```

Default ACLs apply to newly created items within the directory; they do
not automatically change existing files. Actual access also depends on
directory traversal permissions, ACL masks, and the permissions
requested when a file is created.

#### ACL case example

A course folder is owned by the instructor. The instructor needs full
access, the lab group needs read access, and one teaching assistant
needs read/write access.

``` bash
sudo setfacl -m g:labgroup:rx /srv/course
sudo setfacl -m u:assistant:rwX /srv/course
getfacl /srv/course
```

The `X` permission adds execute/search permission to directories and to
files that already have an execute bit. Confirm that parent directories
allow users to traverse the path.

### 5.3.5 Least privilege

**Least privilege** means giving each user, application, or service only
the permissions needed to perform its task.

Examples:

-   A website database account should not have database-administrator
    privileges.
-   A student should not be able to modify another student's private
    files.
-   A web server should not run as the root user.
-   SSH access should be limited to authorized administrators.
-   A firewall should expose only services that are required.

Least privilege reduces the impact of mistakes, compromised accounts,
and vulnerable applications.

------------------------------------------------------------------------

## 5.4 Practical Works

### Practical 1: Set up Apache and host a sample website

**Aim:** Install Apache, serve a local HTML page, and verify the
service.

**Procedure:**

1.  Install Apache using the package manager.
2.  Start and enable the Apache service.
3.  Create an `index.html` file in the document root.
4.  Check that Apache is listening on port 80.
5.  Test using `curl` and a web browser.
6.  Review the access and error logs.

**Verification commands:**

``` bash
sudo systemctl status apache2
sudo ss -ltnp | grep ':80'
curl -I http://localhost
sudo tail -n 20 /var/log/apache2/access.log
sudo tail -n 20 /var/log/apache2/error.log
```

**Expected result:** The browser displays the sample page and the access
log records the request.

### Practical 2: Configure an email server using Postfix and Dovecot

**Aim:** Understand how Postfix handles SMTP and Dovecot provides
mailbox access.

**Suggested controlled lab tasks:**

1.  Install Postfix and Dovecot IMAP components.
2.  Inspect service status and configuration files.
3.  Identify which service listens on SMTP and IMAP ports.
4.  Create or identify a test mailbox using the lab's configured
    delivery method.
5.  Test only within the instructor-approved lab environment.
6.  Review service logs and identify failed delivery or authentication
    messages.

**Verification commands:**

``` bash
sudo systemctl status postfix
sudo systemctl status dovecot
sudo ss -ltnp | grep -E ':(25|143|587|993)\b'
sudo journalctl -u postfix --since "10 minutes ago"
sudo journalctl -u dovecot --since "10 minutes ago"
```

**Important:** A working Postfix service alone does not prove that
end-to-end email delivery is configured. Mailbox delivery, DNS,
authentication, TLS, and relay restrictions must also be correct. Do not
configure an open relay or expose an experimental mail server to the
public internet.

### Practical 3: Secure SSH and configure a firewall

**Aim:** Enable secure remote administration and permit only required
services.

**Procedure:**

1.  Install and start OpenSSH Server.
2.  Create an SSH key pair on the client.
3.  Install the public key for the intended server account.
4.  Test key-based login in a second terminal.
5.  Configure firewall rules for SSH and the required web ports.
6.  Verify access from an authorized client.
7.  Review SSH logs and firewall status.

**Verification commands:**

``` bash
sudo systemctl status ssh
sudo sshd -t
sudo ufw status verbose
sudo journalctl -u ssh --since "10 minutes ago"
```

**Safety reminder:** Keep an existing remote session open while testing
changes. Confirm that the new SSH connection works before closing the
original session.

------------------------------------------------------------------------

## 5.5 Case Scenarios and Troubleshooting

### Case 1: Website is not opening

**Scenario:** A student installs Apache. The service appears to be
running, but the website cannot be opened from another computer.

**Possible causes:**

-   Apache is not listening on the expected interface or port.
-   The firewall blocks port 80.
-   The client uses an incorrect IP address or hostname.
-   The web server and client are on different networks without a route.
-   The website file or virtual-host configuration is incorrect.

**Troubleshooting approach:**

``` bash
sudo systemctl status apache2
sudo ss -ltnp | grep ':80'
curl -I http://localhost
sudo ufw status
```

Test from the client with the server's IP address. Check logs and
network connectivity before changing configuration.

### Case 2: Email is sent but never reaches the recipient

**Scenario:** A user submits a message successfully, but the recipient
does not receive it.

**Possible causes:**

-   The message is still in the Postfix queue.
-   DNS or destination-server settings are incorrect.
-   The remote server rejects the message.
-   The message is classified as spam.
-   Mailbox delivery or mailbox permissions are incorrect.

**Troubleshooting approach:**

Check Postfix logs and queue status. Identify whether the message was
accepted, deferred, rejected, or delivered. Verify DNS and
recipient-server responses. Do not assume that a successful client
submission means final delivery.

### Case 3: Website cannot connect to MySQL

**Scenario:** Apache serves a page, but the application displays a
database connection error.

**Possible causes:**

-   MySQL is stopped.
-   The application has the wrong database name or credentials.
-   The database user is not permitted to connect from the application
    host.
-   MySQL is listening on a different address or port.
-   A firewall blocks the required connection.

**Troubleshooting approach:**

1.  Check the MySQL service.
2.  Verify the database and account settings.
3.  Test local database access.
4.  Inspect application and database logs.
5.  Confirm network and firewall rules if the database is remote.

Avoid solving the problem by granting global privileges or exposing port
3306 to all networks.

### Case 4: Administrator is locked out after firewall changes

**Scenario:** An administrator enables a firewall remotely and can no
longer connect through SSH.

**Cause:** The firewall may not allow the SSH port, or the configured
SSH port may differ from the allowed port.

**Prevention and recovery:**

-   Allow the correct SSH port before enabling the firewall.
-   Keep an existing administrative session open while testing.
-   Use a trusted console or out-of-band access method if remote access
    is lost.
-   Review the rules and restore only the access required.

### Case 5: A student can read a shared file but cannot edit it

**Scenario:** A student needs to edit one shared file, but the existing
group permissions allow only reading.

**Solution:** An administrator can add a targeted ACL entry for that
student, then verify the result with `getfacl` and a test performed as
the student. The administrator should not make the entire directory
world-writable.

------------------------------------------------------------------------

## 5.6 Quick Revision

-   Apache serves web content; HTTP commonly uses port 80 and HTTPS
    commonly uses port 443.
-   HTTPS protects HTTP traffic using TLS.
-   Postfix is an MTA that transfers and queues email.
-   Dovecot commonly provides IMAP/POP3 access to mailboxes.
-   SMTP transfers email; IMAP and POP3 provide mailbox access.
-   MySQL is a relational database server; SQL is used to define and
    manipulate data.
-   SSH provides encrypted remote administration, commonly on port 22.
-   A firewall controls network traffic according to rules.
-   ACLs provide more specific file and directory permissions.
-   Least privilege limits the permissions of users and services.
-   Logs, service status, listening ports, and configuration tests are
    key troubleshooting tools.
-   Secure configuration, updates, backups, strong authentication, and
    restricted network exposure are essential parts of server
    administration.

## Key Terms

  Term              Meaning
  ----------------- ------------------------------------------------------
  HTTP              Protocol for transferring web requests and responses
  HTTPS             HTTP protected by TLS
  TLS               Protocol for protecting communication in transit
  SMTP              Protocol used to transfer email
  IMAP              Protocol for accessing and synchronizing mailboxes
  MTA               Mail Transfer Agent
  RDBMS             Relational Database Management System
  SQL               Structured Query Language
  SSH               Secure Shell
  ACL               Access Control List
  Least privilege   Granting only the access required for a task
  Firewall          System that filters network traffic using rules
  Virtual host      Web-server configuration for hosting a site
  Mail queue        Messages waiting for processing or delivery
