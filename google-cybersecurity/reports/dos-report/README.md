# Incident Report: Denial of Service (DoS) via TCP SYN Flood Attack

> **Ticket ID:** INC-DOS-01 | **Time:** 3.14s - 51.82s (Relative Capture) | **Severity:** High | **Status:** Resolved

---

## 1) Executive Summary
- The enterprise booking web server (`192.0.2.1`) experienced a total service disruption, resulting in connection timeout errors for employees and prospective customers.
- Deep packet inspection identified a volumetric TCP SYN flood originating from an external untrusted IP address (`203.0.113.0`) targeting HTTPS port 443.
- The attack overwhelmed the TCP connection backlog table, rendering the server unable to accept valid connections. Temporary blocking rules were implemented on the edge firewall, and the adoption of SYN attack protection mechanisms was recommended.

---

## 2) Incident Timeline
| Time (s) | Source IP:Port | Destination IP:Port | Event Description & Technical Evidence |
| :--- | :--- | :--- | :--- |
| `3.144` | `198.51.100.23:42584` | `192.0.2.1:443` | Legitimate baseline session: 3-way handshake established; `GET /sales.html` returns `200 OK` (Frames 47-51). |
| `3.390` | `203.0.113.0:54770` | `192.0.2.1:443` | Threat actor initiates flood: Rapid TCP `[SYN]` packets targeting port 443 begin (Frame 52). |
| `3.441` | `192.0.2.1:443` | `203.0.113.0:54770` | Server allocates backlog buffer and responds with `[SYN, ACK]`; attacker fails to respond with `[ACK]` (Frame 53). |
| `6.230` | `192.0.2.1:443` | `198.51.100.16:32641` | Backlog queue saturation begins: Server drops valid user and emits `[RST, ACK]` (Frame 73). |
| `7.330` | `192.0.2.1:443` | `198.51.100.5` | Legitimate user request encounters HTTP `504 Gateway Time-out` (Frame 77). |
| `7.380+` | `192.0.2.1:443` | Multiple Clients | Consecutive `[RST, ACK]` packets sent to legitimate IPs (`198.51.100.7`, `198.51.100.22`, `198.51.100.9`). |

---

## 3) Detection & Initial Scoping
* **Detection Source:** Automated network monitoring alert signaling web server unresponsiveness, followed by internal user tickets reporting connection timeouts.
* **Trigger / Signature:** Anomalous spike in inbound TCP `SYN` packets without corresponding `ACK` handshakes; connection timeout threshold exceeded.
* **Scope of Impact:** 
  * **Affected Systems:** Production Travel Booking Web Server (`192.0.2.1:443`)
  * **Targeted Accounts:** N/A (Infrastructure availability attack, no credential targeting observed)
  * **Observed Threat Actor IPs:** `203.0.113.0` (Source Port: `54770`)

---

## 4) Technical Investigation & Evidence Analysis
### 4.1 Network / Protocol Analysis
* **Handshake / Flow Inspection:** 
  * Baseline traffic from valid clients (`198.51.100.23`, `198.51.100.14`) completed normal 3-way handshakes (`SYN` -> `SYN-ACK` -> `ACK`) and received HTTP `200 OK` responses.
  * Threat actor traffic (`203.0.113.0`) executed a continuous flood of `SYN` requests (`Seq=0 Win=5792 Len=0`). The attacker intentionally abandoned all sessions at the half-open stage (`SYN_RECV`), never returning final `ACK` packets.
* **Failure State:** As connection queues saturated, subsequent valid user requests were terminated by the server with `[RST, ACK]` flags or stalled until timing out with HTTP `504 Gateway Time-out` (Frame 77).

### 4.2 Endpoint & Authentication Inspection
* **Logon Events:** N/A. No administrative authentication or credential brute force attempts occurred during this window.
* **Host Resource Status:** Memory exhaustion in the kernel TCP connection backlog buffer, preventing new socket allocations for incoming network requests.

---

## 5) MITRE ATT&CK Mapping
| Tactic | Technique ID | Technique Name | Observed Behavioral Context |
| :--- | :--- | :--- | :--- |
| **Impact** | `T1498.001` | Network Denial of Service: Direct Network Flood | High-volume TCP SYN packet burst targeted at port 443 to saturate network bandwidth. |
| **Impact** | `T1499.002` | Endpoint Denial of Service: Service Exhaustion Flood | Flooding half-open TCP connections to exhaust the web server's backlog queue. |

---

## 6) Impact Assessment
* **Confidentiality:** None 
* **Integrity:** None 
* **Availability:** **High** 
* **Overall Severity Rating:** **High** 

---

## 7) Containment, Eradication & Remediation
- [x] **Immediate Containment:** Temporarily took the web server offline to flush half-open TCP state tables, then applied an edge firewall drop rule for `203.0.113.0`.
- [ ] **Kernel Hardening:** Enable TCP SYN Cookies to process connection requests without allocating queue memory until handshake completion:
