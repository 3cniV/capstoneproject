---
description: >-
  A home lab setup for intensive testing of different tools used in offensive
  security.
---

# Intensive 14-Day Penetration Testing Tools Lab

## Network Enumeration

* Target: [scanme.nmap.org](http://scanme.nmap.org).
* Tool: Nmap (Aggressive Scan -A).
* Technical Challenges and Troubleshooting:
  * **Issue Encountered:** During initial enumeration, the standard Nmap SYN scan (-sS) failed to return results, terminating with the error:

> &#x20;                            Warning: 45.33.32.156 giving up on port because retransmission cap hit (10)

* **Root Cause Analysis:** The target system or an intermediate firewall (ISP level) was dropping unsolicited SYN packets. The scanner attempted to resend packets 10 times with no response, leading to a timeout. This behaviour indicates the presence of a stateless packet filter or an Intrusion Prevention System (IPS) blocking "half-open" connection attempts
* **Troubleshooting Strategy:** I switched from the default SYN scan to a TCP Connect Scan (-sT).
  * **Why it worked:** Unlike the stealthy SYN scan, the connect scan completes the full TCP 3-Way Handshake (SYN → SYN-ACK → ACK). This mimics the behaviour of a legitimate web browser or client, allowing the traffic to pass through the firewall inspection rules.
  * **Outcome:** The -sT scan successfully bypassed the filter, revealing open ports (22, 80, 9929) and allowing for subsequent Service Version Enumeration (-sV).
* **Key Findings:**
  * OS Identification: High probability of a Virtualized Linux environment (QEMU/VirtualBox).
  * Web Server: Apache 2.4.7 (Outdated).
  * SSH: OpenSSH 6.6.1p1 (Outdated).
  * Network Layout: High latency (1300ms+), requiring TCP Connect scans to maintain stability.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

## Passive Reconnaissance

* Target: Ford ([ford.com](http://ford.com)) .
* Tool: Google Advanced Search Operators (Dork), PaGoDo.
* Query 1: site:[ford.com](http://ford.com) filetype:pdf “supplier”
  * Findings: I found public “Code of Conduct” documents and technical specifications for OFTP data transfer.
* Query 2: site:[ford.com](http://ford.com) filetype:pdf “confidential”
  * Findings: I found documents that have “confidential” information.
* Query 3: site:[ford.com](http://ford.com) filetype:pdf “internal use only”
  * Findings: I found documents that were classified as“internal use only” and may not be resold by vendors.
* Query 4: site:[ford.com](http://ford.com) filetype:xlsx
  * Findings:&#x20;
  * Exposed Staging Environments: I found several files on wwwqa (Quality Assurance) subdomains, showing that non-production environments are publicly indexed by search engines, which could expose test data or unpatched code.
  * Internal Process Documentation: I identified a “New Hire On-Boarding: spreadsheet publicly accessible. This poses a risk of Social Engineering attacks, as the file likely contains internal workflows and tool locations.
  * Business Intelligence Leak: I discovered a “Supplier Cost Breakdown: spreadsheet, which can potentially expose sensitive pricing strategies to competitors.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

## **Network Traffic Analysis.**

* Target: [testphp.vulnweb.com](http://testphp.vulnweb.com) (Authorized testing environment).
* Tool: Wireshark.
* Objective: To capture and inspect unencrypted HTTP POST requests to identify clear-text credentials.
* Technical Challenges and Troubleshooting:
  * Issue Encountered: Network traffic was not recorded when the Wireshark capture was initiated, even though I was browsing the web using firefox on Kali.
  * **Root cause analysis:** I selected a disconnected ethernet adapter (eth0) rather than the active adapter with real time traffic (eth1).
  * **Troubleshooting Stage:** I terminated the current capture session, then I ran the “Ip a” command in my kali terminal to determine which adapter was the active one; I then noticed eth0 and eth1 both had inet connections but when I pinged google with each adapter specifically (using “ping -I eth0/eth1 [google.com](http://google.com)”), I was able to get the active adapter. I then restarted the capture session on the eth1 network adapter showing live traffic.
* Process:
  * Initiated packet capture on eth,
  * Generated login traffic via Firefox browser,
  * Applied display filter: {http.request.method == “POST”} to isolate form submissions.
* Key Findings:
  * Successfully intercepted the HTTP body.
  * Credential Exposure: Identification of fields “uname” and “pass” containing plaintext credentials.
  * Security Analysis: This shows the critical risk of using HTTP. It is mandatory to use SSL/TLS (HTTPS) encryption to prevent Man-in-the-Middle (MitM) attacks.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

## Web Directory Enumeration.

* Target: [testphp.vulnweb.com](http://testphp.vulnweb.com) (Authorized testing environment).
* Tool: Gobuster.
* Objective: To identify hidden web directories and unlinked administrative paths.
* Process:
  * I utilized the dir mode of Gobuster with the common.txt wordlist using the following command “gobuster dir -u http://testphp.vulnweb.com -w /usr/share/wordlists/dirb/common.txt”.
  * Targeted the root URL to enumerate subdirectories
* Key Findings:
  * /admin (Status: 301): I discovered a hidden administrative panel login page. The existence of an accessible /admin panel without IP restriction or VPN requirements increases the attack surface for Brute Force attacks&#x20;
  * /CVS (Status: 200): I found a Concurrent Versions System repository (older version control), which may leak source code structure.
  * /images (Status: 200): I then validated an open directory for assets.
* Advanced Enumeration (File Extensions):
  * Command: gobuster dir ... -x php,txt
  * Objective: Identify executable scripts and text files that standard directory scans miss.
  * Findings:
  * login.php (Status: 200) - Exposed authentication endpoint.
  * search.php (Status: 200) - Potential vector for input-based attacks (XSS/SQLi).

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
