# Amaterasu

## Summary
Amaterasu is a medium-difficulty Proving Grounds machine that demonstrates a full exploitation chain involving REST API enumeration, arbitrary file write via path traversal, SSH key injection, and Linux privilege escalation through cron wildcard injection.

The box emphasizes identifying non-standard web services, abusing insecure file upload functionality, and exploiting poorly written privileged scripts.

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
A full TCP port scan revealed multiple services:
- FTP (anonymous access enabled)
- SSH on a non-standard port (25022)
- HTTP services on ports 40080 and 33414

The web service on port 40080 served a static page and provided no attack surface.  
The service on port 33414 was identified as a Werkzeug-based Python application, suggesting a REST API.

Fuzzing revealed endpoints:
- `/help`
- `/info`

The `/help` endpoint disclosed additional functionality:
- `/file-list?dir=`
- `/file-upload`

---

## Initial Access
The `/file-list` endpoint allowed directory enumeration across the filesystem, revealing sensitive paths such as:
/home/alfredo/.ssh


The `/file-upload` endpoint required both `file` and `filename` parameters. By manipulating the `filename` parameter, path traversal was achieved, allowing arbitrary file writes.

An SSH public key was uploaded to:


/home/alfredo/.ssh/authorized_keys


A file extension filter was bypassed by renaming the key to a `.txt` file.

This allowed SSH access as user `alfredo`.

---

## Privilege Escalation
A cron job running as root was identified:


*/1 * * * * root /usr/local/bin/backup-flask.sh


The script contained:


tar czf /tmp/flask.tar.gz *


Because the script executed in a user-writable directory (`/home/alfredo/restapi`), wildcard injection was possible.

Malicious filenames were created:


--checkpoint=1
--checkpoint-action=exec=sh exploit.sh


A payload script was used to create a SUID shell:


cp /bin/sh /tmp/rootsh
chmod +s /tmp/rootsh


Once the cron job executed, a root shell was obtained via:


/tmp/rootsh -p


---

## Key Takeaways
- Non-standard web services (Werkzeug/Flask) often indicate hidden REST APIs
- File upload endpoints are high-risk and must be thoroughly tested
- Path traversal in upload parameters can lead to full system compromise
- Cron jobs using wildcards (`*`) are vulnerable to argument injection
- Timing (cron execution intervals) is critical during exploitation

---

## Detection / Hardening Notes

### Logs to Monitor
- API access logs for `/file-upload` and `/file-list`
- SSH logs for new or unusual key-based authentication
- Cron execution logs for unexpected behavior
- File integrity monitoring on `.ssh/authorized_keys`

### Detection Ideas
- Alert on file uploads containing path traversal sequences (`../`)
- Monitor for suspicious filenames (e.g., `--checkpoint-action`)
- Detect abnormal API usage patterns
- Alert on creation of SUID binaries in non-standard directories

### Hardening Recommendations
- Validate and sanitize all user input in API endpoints
- Restrict file upload paths and enforce strict directory boundaries
- Implement allowlists for file types and paths
- Avoid wildcard usage in privileged scripts
- Apply least privilege principles to service accounts
- Disable anonymous FTP access if not required

