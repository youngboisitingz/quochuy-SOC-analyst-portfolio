# Incident Report: Web Server Compromise via Brute Force & Drive-By Malware Redirection

## 1) Executive Summary
- The production web server hosting `yummyrecipesforme.com` was compromised by a disgruntled former employee using a brute force credential attack against default administrator accounts.
- The adversary successfully authenticated, hijacked administrative privileges by rotating credentials, and injected malicious JavaScript into the web source code.
- Visitors were prompted to download a trojanized browser update, which redirected outbound client sessions to a spoofed malicious domain, `greatrecipesforme.com`.

## 2) Timeline
| Time (UTC) | Source / Actor | Destination / Target | Event Description & Evidence |
| :--- | :--- | :--- | :--- |
| Pre-incident | Former employee | `yummyrecipesforme.com` (Admin) | Brute force password guessing using default credential lists; admin password modified. |
| Pre-incident | Adversary | Web Codebase | Malicious JavaScript function injected to serve drive-by download prompts. |
| 14:18:32 | Client (`52444`) | `dns.google` (UDP 53) | DNS query A for `yummyrecipesforme.com` -> resolved to `203.0.113.22`. |
| 14:18:36 | Client (`36086`) | `203.0.113.22` (TCP 80) | 3-way handshake completed (`[S]` -> `[S.]` -> `[.]`); HTTP GET request sent; malware dropper delivered. |
| 14:20:32 | Client (`52444`) | `dns.google` (UDP 53) | Secondary DNS query A for spoofed domain `greatrecipesforme.com` -> resolved to `192.0.2.17`. |
| 14:25:29 | Client (`56378`) | `192.0.2.17` (TCP 80) | Outbound HTTP connection established to rogue server hosting secondary malware payloads. |

## 3) Detection
- **Detection Method:** Customer incident reports (complaints regarding forced downloads and system degradation) and hosting provider escalation by the business owner.
- **Analysis Environment:** Isolated dynamic sandbox using `tcpdump` protocol analyzer.
- **Trigger:** Unauthorized codebase modification and anomalous outbound redirection observed during sandbox execution.

## 4) Investigation
### 4.1 Application & Credential Review
- Source code analysis confirmed unauthorized modification: an embedded JavaScript routine prompting site visitors to download and execute a fake browser update.
- Code inspection of the downloaded file confirmed an automated script designed to hijack client browser sessions and redirect them to `greatrecipesforme.com`.
- Administrative auditing verified that the web management portal lacked lockout thresholds, allowing unlimited automated login attempts against default passwords.

### 4.2 Network Traffic Analysis (Sandbox Artifacts)
- **Primary Session:** Workstation resolved `yummyrecipesforme.com` via Google Public DNS and completed a full TCP handshake (`[S]`, `[S.]`, `[.]`) on port 80[cite: 6]. The server delivered application data via cờ `[P.]` (PSH-ACK) with payload length 73 bytes.
- **Malware Redirection:** Following file execution, the endpoint initiated a new DNS query for `greatrecipesforme.com` and established a secondary TCP stream to `192.0.2.17:80`, validating malicious traffic redirection.

## 5) MITRE ATT&CK Mapping
- **Initial Access:** Brute Force: Password Guessing (`T1110.001`)
- **Persistence / Impact:** Defacement: External Defacement (`T1491.002`)
- **Execution:** User Execution: Malicious File (`T1204.002`) & Command and Scripting Interpreter: JavaScript (`T1059.007`)

## 6) Impact Assessment
- **Impact:** High (Total loss of administrative access, compromised website integrity, and client malware infections)
- **Likelihood:** High (No account lockout controls; exposed default credentials)
- **Severity:** High

## 7) Recommendations
- Enforce an Account Lockout Policy (e.g., lock accounts for 30 minutes after 3-5 consecutive failed attempts).
- Mandate changing default administrative credentials and implement Multi-Factor Authentication (MFA) on all management portals.
- Revert web server files to a verified, clean off-site backup to remove malicious JavaScript.
- Implement network perimeter blocks for malicious IP `192.0.2.17` and sinkhole domain `greatrecipesforme.com`.

## 8) Lessons Learned
- **Offboarding Deficiencies:** Failure to revoke former employee access and rotate shared administrative secrets created an immediate insider threat vector.
- **Monitoring Gaps:** Implement File Integrity Monitoring (FIM) and Web Application Firewall (WAF) inspection to detect unauthorized code changes and script injection attempts automatically.
