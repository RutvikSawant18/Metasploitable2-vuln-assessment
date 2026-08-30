Vulnerability Assessment Report — Metasploitable2
Author: Rutvik Sawant
Type: Internal network penetration test (lab environment)
Target: Metasploitable2 (intentionally vulnerable training VM)
Tools: Nmap, Metasploit Framework, VirtualBox (isolated internal network)
Assessment window: August 2026
> This is a sanitized summary of a full vulnerability assessment report. Exact exploit commands and raw shell output have been omitted; methodology, findings, risk ratings, and remediation guidance are included in full. The full report is available on request.
Overview
This assessment followed a standard four-phase vulnerability management workflow — reconnaissance, enumeration, validation, and reporting — against a single host in an isolated lab network with no connectivity to any production system or the internet.
A full 65,535-port Nmap scan (`-sV -sC -O -p-`) identified 30 open services. Each was cross-referenced against public CVE records. Two Critical findings were actively validated using Metasploit; both resulted in complete unauthenticated remote root access. Ten additional findings were identified via service fingerprinting and CVE research but not exploited, in the interest of scope.
Summary of Findings
#	Finding	Port/Service	CVSS v3.1	Severity	Verified
1	vsftpd 2.3.4 Backdoor Command Execution (CVE-2011-2523)	21/tcp	10.0	Critical	✅ Exploited
2	Samba "username map script" Command Injection (CVE-2007-2447)	139,445/tcp	10.0	Critical	✅ Exploited
3	UnrealIRCd Backdoor (CVE-2010-2075)	6667,6697/tcp	10.0	Critical	Identified only
4	distcc Remote Code Execution (CVE-2004-2687)	3632/tcp	9.8	Critical	Identified only
5	Pre-existing root shell backdoor	1524/tcp	10.0	Critical	Identified only
6	Anonymous FTP login permitted	21/tcp	7.5	High	Identified only
7	Legacy r-services with weak trust authentication	512,513,514/tcp	8.1	High	Identified only
8	SMB message signing disabled	139,445/tcp	5.9	Medium	Identified only
9	Deprecated SSLv2 with export-grade ciphers	25/tcp	5.3	Medium	Identified only
10	Outdated database services (MySQL 5.0, PostgreSQL 8.3)	3306,5432/tcp	5.0	Medium	Identified only
11	Unencrypted Telnet service	23/tcp	5.9	Medium	Identified only
12	End-of-life Linux kernel (2.6.x)	Host-level	N/A	Informational	Identified only
Highlighted Findings
1. vsftpd 2.3.4 Backdoor Command Execution — CVE-2011-2523
Description: The publicly distributed vsftpd 2.3.4 source archive was compromised in 2011 to include a hidden backdoor — a specific string in the FTP username field triggers a listener that hands back an unauthenticated root shell.
Validation approach: Confirmed via Metasploit's `vsftpd_234_backdoor` module against the target. The exploit successfully returned an interactive shell, confirmed to be running as `uid=0(root)`.
Impact: Complete, unauthenticated remote compromise of the host — full read/write access to the filesystem and all local user accounts.
Remediation: Upgrade vsftpd to a version from an official, verified source; validate package integrity via checksums for any future third-party software installs.
2. Samba "username map script" Command Injection — CVE-2007-2447
Description: Samba's `username map script` configuration option, when enabled, passes login-supplied usernames directly to a shell command without sanitization, allowing arbitrary command injection.
Validation approach: Confirmed via Metasploit's `usermap_script` module. A reverse shell payload initially failed due to a loopback-address misconfiguration; explicitly setting the listener address to the attacker host resolved this, and the exploit returned a root shell on the second attempt.
Impact: Identical to Finding 1 — full unauthenticated root compromise, this time via an entirely independent service, indicating a systemic pattern of unmaintained/misconfigured services rather than an isolated flaw.
Remediation: Disable the `username map script` option unless explicitly required; keep Samba patched to a current release; avoid running services with root-equivalent privileges where avoidable.
Methodology Notes
All testing was performed against an isolated VirtualBox internal network with no route to the internet or any host system network.
Findings not actively exploited were still validated against public CVE/NVD records for the exact service version identified by Nmap, rather than relying on version banners alone.
CVSS v3.1 base scores were used for risk rating; scores marked "estimated" reflect a rating derived from the closest matching published CVE for that vulnerability class where an exact score for the specific instance wasn't published.
Conclusion
The assessed host demonstrates nearly every major category of network-service vulnerability in a single system — hardcoded backdoors, unauthenticated services, disabled integrity protections, and unencrypted protocols. Two independent Critical findings were confirmed to grant full root access with zero authentication required. This result reinforces a core vulnerability management principle: consistent patch cycles and minimal service exposure are not optional hardening steps but baseline requirements, since a single outdated package was sufficient for total compromise in both validated cases.
---
Full technical report — including exact command sequences, screenshots, and detailed remediation timelines — available on request.
