# HTB Starting Point - Appointment

## Overview

Appointment is a Tier 1 Hack The Box Starting Point machine focused on web application security and SQL Injection. The objective was to identify a vulnerable login portal, understand how SQL queries are used for authentication, and exploit improper input validation to bypass the login mechanism.

Through this room, I learned how SQL Injection can be used to manipulate authentication queries and gain unauthorized access to an application.

## Skills Learned

- Nmap service enumeration
- Web application reconnaissance
- SQL Injection fundamentals
- Authentication bypass techniques
- Understanding web application attack surfaces
- Identifying insecure database queries

## Tools Used

- Nmap
- Web Browser (Firefox on Pwnbox)

## Enumeration

### Service Discovery

I began by scanning the target host to identify running services.

```bash
nmap -sV <target-ip>
```

### Results

```text
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
```

### Findings

- One open port was discovered.
- Port 80 hosted a web application.
- The web server was Apache HTTP Server 2.4.38 running on Debian Linux.
- The web application became the primary attack surface.

## Web Application Analysis

After navigating to the target IP address, I was presented with a login portal.

### Observations

- Username field
- Password field
- Login button
- No registration functionality
- No valid credentials provided

Since authentication was required and no credentials were known, I investigated potential weaknesses in the login mechanism.

## Exploitation

### SQL Injection Authentication Bypass

The login form was vulnerable to SQL Injection.

#### Payload Used

**Username**

```sql
admin'#
```

**Password**

```text
abc123
```

### Explanation

The application failed to properly sanitise user input before inserting it into an SQL query.

The payload:

```sql
admin'#
```

- Closes the username string.
- Uses `#` to comment out the remainder of the SQL statement.
- Prevents password validation from occurring.

As a result, the application only checked whether the user `admin` existed, allowing authentication without knowing the real password.

## Flag Obtained

Successful exploitation resulted in authentication bypass and access to the flag.

## Key Concepts

### SQL Injection (SQLi)

SQL Injection is a vulnerability that occurs when untrusted user input is included directly within SQL queries.

Potential impacts include:

- Authentication bypass
- Database enumeration
- Data disclosure
- Data modification
- Remote compromise of application functionality

### HTTP

Hypertext Transfer Protocol (HTTP) is the communication protocol used between web browsers and web servers.

### Apache HTTP Server

Apache is one of the most widely used web servers and is responsible for serving web content to clients.

## Security Lessons

### Vulnerability

- SQL Injection Authentication Bypass

### Root Cause

- Unsanitised user input
- Lack of parameterised queries
- Improper authentication controls

### Recommended Mitigations

- Use parameterised queries
- Use prepared statements
- Validate and sanitise user input
- Implement secure coding practices
- Deploy Web Application Firewalls (WAFs)

## Cybersecurity Relevance

This room is relevant to:

- Web Application Security
- SOC Analyst Roles
- Security Analyst Roles
- Penetration Testing
- Threat Hunting

Understanding SQL Injection helps security professionals identify vulnerable applications and recognise exploitation attempts within web server and database logs.

## Key Takeaways

- Enumeration should always come before exploitation.
- Web applications are common attack surfaces.
- SQL Injection remains one of the most dangerous web vulnerabilities.
- Improper input handling can completely bypass authentication mechanisms.
- Parameterised queries are essential for secure database interactions.

## Screenshots

Add screenshots of:

1. Nmap service scan
2. Login page
3. Successful login bypass
4. Flag page

## Reflection

Appointment provided a practical introduction to SQL Injection and demonstrated how a simple input validation failure can compromise an application's authentication system. This room reinforced the importance of thorough enumeration, understanding how web applications interact with databases, and recognising common web vulnerabilities that are frequently encountered in real-world environments.