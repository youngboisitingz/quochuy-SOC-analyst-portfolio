# Incident Report: [Incident Title - e.g., Web Server Compromise via Brute Force]

## Metadata
| Parameter | Details |
| :--- | :--- |
| **Incident ID** | INC-[YYYYMMDD]-[01] |
| **Date / Time** | YYYY-MM-DD HH:MM UTC |
| **Severity** | Critical / High / Medium / Low |
| **Lead Analyst** | [Your Name / GitHub Handle] |
| **Status** | Closed / Resolved |

---

## 1) Executive Summary
* **Overview:** A concise summary stating what happened, the affected host/service, and the business impact (1-2 sentences).
* **Root Cause:** How the attack or failure occurred (e.g., exploitation of default administrative credentials, unauthenticated buffer overflow, volumetric SYN flood).
* **Final Outcome:** Current status of the asset (e.g., malicious scripts removed, credentials rotated, network availability restored).

---

## 2) Incident Timeline
| Timestamp (UTC) | Source / Actor | Destination / Target | Event Description & Artifact |
| :--- | :--- | :--- | :--- |
| `HH:MM:SS` | `[Source IP / User]` | `[Dest IP / Port]` | Initial probe or anomaly observed (e.g., Frame #X, Event ID 4625) |
| `HH:MM:SS` | `[Source IP / User]` | `[Dest IP / Port]` | Successful exploitation / intrusion execution |
| `HH:MM:SS` | `[Security Control]` | `[Compromised Asset]` | Detection triggered / alert generated |
| `HH:MM:SS` | `[SOC Team]` | `[Target Host]` | Containment action executed (e.g., IP blocked, service isolated) |

---

## 3) Detection & Initial Scoping
* **Detection Source:** Network Protocol Analyzer (`tcpdump`, Wireshark) / SIEM (Wazuh, Splunk) / Host Logs (Sysmon, Event Viewer).
* **Trigger / Signature:** Specific condition that raised the red flag (e.g., anomalous SYN-to-ACK ratio, repeated failed authentication spikes, ICMP Type 3 Code 3 burst).
* **Scope of Impact:** 
  * **Affected Systems:** `[Hostname / Target IP / Subnet]`
  * **Targeted Accounts:** `[Admin username / Service account]`
  * **Observed Threat Actor IPs:** `[Malicious external IP(s)]`

---

## 4) Technical Investigation & Evidence Analysis
### 4.1 Network / Protocol Analysis
<!-- Dành cho các bài phân tích gói tin, web traffic, firewall logs -->
* **Handshake / Flow Inspection:** Detail whether handshakes completed (`SYN` -> `SYN-ACK` -> `ACK`) or failed abnormally (`RST, ACK`, Half-open saturation).
* **Payload & Request Inspection:** Detail application-layer anomalies (e.g., HTTP method abuse, injected JavaScript functions, base64-encoded strings).

### 4.2 Endpoint & Authentication Inspection
<!-- Dành cho các bài phân tích Windows Event Logs, Sysmon, Linux Auth logs -->
* **Logon Events:** Event ID 4625 (Failed Logon) vs. Event ID 4624 (Successful Logon), Logon Type (`Type 3` Network, `Type 10` RemoteInteractive).
* **Execution & Process Lineage:** Anomalous parent-child process chains (e.g., `cmd.exe` or `powershell.exe` spawned by `w3wp.exe` / `apache2`).

---

## 5) MITRE ATT&CK Mapping
| Tactic | Technique ID | Technique Name | Observed Behavioral Context |
| :--- | :--- | :--- | :--- |
| **Initial Access** | `T1110.001` | Brute Force: Password Guessing | Repeated authentication attempts using default credential lists. |
| **Execution** | `T1059.007` | Command and Scripting: JavaScript | Malicious script injected into web pages to serve drive-by download files. |
| **Impact** | `T1498.001` | Network DoS: Direct Network Flood | Volumetric packet burst targeted at port 443 causing service unavailability. |

---

## 6) Impact Assessment
* **Confidentiality:** None / Low / Medium / High (Were sensitive credentials, databases, or user data exfiltrated?).
* **Integrity:** None / Low / Medium / High (Were web files, source code, or system binaries modified?).
* **Availability:** None / Low / Medium / High (Was service disrupted, degraded, or completely offline?).
* **Overall Severity Rating:** **[Critical / High / Medium / Low]** based on organizational operational risk.

---

## 7) Containment, Eradication & Remediation
- [ ] **Immediate Containment:** Isolate compromised host from the network / block malicious IP `[X.X.X.X]` at perimeter firewall.
- [ ] **Eradication:** Terminate malicious processes, wipe injected scripts, and restore verified clean source code from backups.
- [ ] **Remediation & Hardening:**
  - Enforce Account Lockout Policy (e.g., lock account after 5 consecutive failures).
  - Mandate Multi-Factor Authentication (MFA) across all administrative management portals.
  - Tune kernel/firewall parameters (e.g., enable TCP SYN Cookies `net.ipv4.tcp_syncookies = 1`).

---

## 8) Lessons Learned & Detection Engineering
* **Root Cause Deficiencies:** What policy, architecture, or configuration weakness allowed this incident to occur?
* **Detection Gaps:** Why was the intrusion not caught earlier (e.g., absence of rate-limiting thresholds, unmonitored default accounts)?
* **Actionable Next Steps:**
  * Implement a custom Wazuh / Suricata detection rule to trigger alerts when login failures exceed 10 attempts per minute.
  * Establish automated configuration auditing to ensure default credentials are eliminated prior to production deployment.
