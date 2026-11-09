# HTB Starting Point - Crocodile

## Overview

Crocodile is a Tier 1 Hack The Box Starting Point machine focused on enumeration, exposed services, credential discovery, and web application authentication. The objective was to identify accessible services, investigate an exposed FTP server, recover credentials, discover hidden web content, and gain access to a protected dashboard.

This room demonstrated how seemingly small misconfigurations can be chained together to compromise a system. Anonymous FTP access exposed sensitive files containing usernames and passwords, which were then leveraged to access a hidden web application.

---

## Skills Learned

- Service enumeration
- FTP enumeration
- Anonymous FTP access
- File retrieval via FTP
- Credential discovery
- Credential reuse attacks
- Web directory enumeration
- Authentication testing
- Gobuster usage

---

## Tools Used

- Nmap
- FTP Client
- Gobuster
- Firefox Browser
- Pwnbox

---

## Enumeration

### Service Discovery

I began by scanning the target machine to identify exposed services.

```bash
nmap -sV <target-ip>
```

### Results

```text
21/tcp open  ftp    vsftpd 3.0.3
80/tcp open  http   Apache httpd 2.4.41 ((Ubuntu))
```

### Findings

- FTP service available on Port 21.
- Apache web server available on Port 80.
- Two potential attack surfaces were identified.

---

## Investigating FTP

### Anonymous Login

Since FTP was available, I tested whether anonymous access was enabled.

```bash
ftp <target-ip>
```

Credentials:

```text
Username: anonymous
Password: anonymous
```

### Result

```text
230 Login successful.
```

### Findings

Anonymous access was enabled, allowing anyone to connect to the FTP server without valid credentials.

---

## Enumerating FTP Files

### Listing Contents

```bash
ls
```

### Results

```text
allowed.userlist
allowed.userlist.passwd
```

The file names suggested they contained usernames and passwords.

### Downloading Files

```bash
get allowed.userlist
```

```bash
get allowed.userlist.passwd
```

Both files were successfully downloaded.

---

## Credential Discovery

### Usernames

```bash
cat allowed.userlist
```

Output:

```text
aron
pwnmeow
egotisticalsw
admin
```

### Passwords

```bash
cat allowed.userlist.passwd
```

Output:

```text
root
Supersecretpassword1
@BASD$9032123sADS
rKXM59ESxesUFHAd
```

### Findings

The FTP server exposed valid usernames and passwords that could potentially be reused elsewhere on the target.

---

## Web Application Investigation

### Initial Observation

After browsing to the website on Port 80, I was presented with a Smash Bootstrap webpage.

At first glance, there was no visible login page or obvious functionality that could be accessed using the discovered credentials.

Because no authentication page was visible, further enumeration was required.

---

## Directory Enumeration

### Command

```bash
gobuster dir -u http://<target-ip> -w /usr/share/wordlists/dirb/common.txt
```

### Results

```text
/assets
/css
/dashboard
/fonts
/js
```

### Key Discovery

```text
/ dashboard
```

This hidden directory appeared to contain a restricted section of the application.

---

## Authentication Attempts

After discovering the dashboard, I attempted to authenticate using the usernames and passwords obtained from FTP.

### Challenge Encountered

Initially, I did not know which username matched which password.

Rather than immediately finding the correct combination, I systematically tested different username and password combinations using the credentials discovered on the FTP server.

Discovered usernames:

```text
aron
pwnmeow
egotisticalsw
admin
```

Discovered passwords:

```text
root
Supersecretpassword1
@BASD$9032123sADS
rKXM59ESxesUFHAd
```

I worked through the available combinations until I successfully identified valid credentials and gained access to the application.

### Lesson Learned

Finding credentials is only part of the attack process. Attackers often need to validate which credentials are active and determine where they can be used.

---

## Accessing the Dashboard

Successful authentication provided access to the Server Manager Dashboard.

### Result

- Dashboard access obtained.
- Restricted content exposed.
- Flag successfully retrieved.

---

## Challenges Encountered

### Challenge 1: No Login Page on the Homepage

**Problem**

The website only displayed a Smash Bootstrap page.

**Solution**

Performed directory enumeration using Gobuster.

**Lesson**

Important application functionality is often hidden and requires additional enumeration.

---

### Challenge 2: Determining Valid Credentials

**Problem**

Several usernames and passwords were discovered, but no indication was provided regarding which combinations were valid.

**Solution**

Tested multiple username and password combinations until successful authentication was achieved.

**Lesson**

Credential discovery does not always immediately lead to access. Testing and validation are often required.

---

## Key Concepts

### FTP

File Transfer Protocol used to transfer files between systems.

### Anonymous FTP

A feature that allows users to access an FTP server without a dedicated account.

### Credential Exposure

Occurs when usernames, passwords, or authentication data are publicly accessible.

### Credential Reuse

The practice of using the same credentials across multiple systems or services.

### Gobuster

A directory enumeration tool used to discover hidden web content.

### Apache HTTP Server

A web server responsible for hosting websites and applications.

---

## Security Lessons

### Vulnerabilities Identified

- Anonymous FTP Access
- Credential Exposure
- Credential Reuse
- Hidden Administrative Functionality

### Risks

- Unauthorized access
- Credential compromise
- Information disclosure
- Privilege escalation opportunities

### Recommended Mitigations

- Disable anonymous FTP access.
- Restrict access to sensitive files.
- Enforce strong authentication controls.
- Avoid credential reuse.
- Regularly audit exposed services and web content.

---

## Cybersecurity Relevance

This room is relevant to:

- Security Analyst
- SOC Analyst
- Penetration Testing
- Web Security
- Identity and Access Management

The techniques used in this room are commonly encountered during real-world assessments where exposed services, leaked credentials, and poor authentication practices can lead to system compromise.

---

## Key Takeaways

- Enumeration should always come first.
- Anonymous FTP can expose sensitive information.
- Credentials found on one service may work on another.
- Directory enumeration is critical when functionality is not visible.
- Hidden application components can often be discovered through systematic testing.
- Multiple weaknesses chained together can result in full compromise.

---

## Screenshots

1. Nmap Scan (`nmap-scan.png`)
2. Anonymous FTP Login (`ftp-anonymous-login.png`)
3. FTP File Listing (`ftp-file-listing.png`)
4. Username Discovery (`usernames-discovered.png`)
5. Password Discovery (`passwords-discovered.png`)
6. Gobuster Enumeration (`gobuster-scan.png`)
7. Dashboard Access (`dashboard-access.png`)
8. Flag (`flag.png`)

---

## Reflection

Crocodile highlighted the importance of thorough enumeration and demonstrated how multiple low-severity weaknesses can be combined into a successful attack path. Anonymous FTP access exposed credential files, and those credentials were then used to gain access to a hidden administrative dashboard. The room reinforced the value of methodical investigation, credential testing, and directory enumeration when assessing web applications and exposed services.