# Blogger

## Summary
The Blogger machine demonstrates a full exploitation chain beginning with REST API enumeration and abuse, leading to arbitrary file write via path traversal. Initial access was achieved by injecting an SSH key through a vulnerable file upload endpoint. Privilege escalation was then obtained through a misconfigured cron job that leveraged wildcard injection in a tar command.

This machine highlights the importance of API security, input validation, and secure scripting practices.

---

## Platform
- Platform: OffSec Proving Grounds
- Difficulty: Medium
- Focus Areas: Web/API Exploitation, Linux Privilege Escalation, File Upload Abuse
- Status: Completed

---

## Skills Demonstrated
- Enumeration
- Initial Access
- Privilege Escalation
- Detection / Defensive Thinking

---

## Tools Used
- Nmap
- curl
- ffuf
- SSH
- tar

---

## Reconnaissance
Initial scanning revealed multiple services, including FTP (anonymous access), SSH on a non-standard port, and two HTTP services.

The service running on port 33414 was identified as a Werkzeug-based Python application, indicating a potential REST API. Fuzzing and manual enumeration revealed API endpoints including `/help`, `/info`, `/file-list`, and `/file-upload`.

The `/help` endpoint provided documentation of available functionality, confirming the presence of a file management API.

---

## Initial Access
The `/file-list` endpoint allowed directory enumeration across the filesystem, revealing sensitive paths such as `/home/alfredo/.ssh`.

The `/file-upload` endpoint required both `file` and `filename` parameters. By manipulating the `filename` parameter, path traversal was achieved, allowing arbitrary file writes.

An SSH public key was uploaded to:
