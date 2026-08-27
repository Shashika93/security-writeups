# vsftpd 2.3.4 Backdoor — Metasploitable 2

## Target

The target is Metasploitable 2, a deliberately vulnerable machine, running at 192.168.56.101. I hosted it in an isolated virtual environment for this exercise.

## Reconnaissance

First, I ran a host discovery scan with `nmap -sn 192.168.56.0/24` to find live hosts on the network. This identified the target at 192.168.56.101.

Next, I ran a service scan with `nmap -sV 192.168.56.101` to enumerate open ports and their versions. 
This revealed 23 open ports, including FTP (vsftpd 2.3.4) on port 21.

## Vulnerability

vsftpd 2.3.4 has a known backdoor vulnerability. Malicious code was intentionally added to the source code of this version. 
If a username ending with a smiley `:)` is submitted, the server opens a backdoor shell on port 6200, giving root access without any valid password. 
Later versions removed this backdoor.

## Exploitation

First, I logged into the FTP server with `ftp 192.168.56.101`. I entered a username ending with a smiley, `hacker:)`, and any password. 
(The login itself fails, but the smiley triggers the backdoor.)

Then I opened a second terminal and connected to the backdoor port with `nc 192.168.56.101 6200`. This opened a root shell with no further authentication.

I confirmed root access by running `whoami`, which returned `root`, and `id`, which returned `uid=0(root)`.

## Root Cause and Remediation

This vulnerability exists because the FTP server runs vsftpd 2.3.4, a version whose source code was deliberately compromised. An attacker inserted a backdoor into the official source distribution — an example of a supply chain attack.

This can be remediated by upgrading vsftpd to a current, supported version. More broadly, production systems should not run outdated or unsupported software, and package integrity should be verified against official checksums before installation.
