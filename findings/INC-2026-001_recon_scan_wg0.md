# Coordinated Reconnaissance Scan Against WireGuard (51820/TCP)

**Incident ID:** INC-2026-001
**Detection Date:** 2026-06-18
**Severity:** Low
**Status:** Closed — Monitoring
**Analyst:** sombra_0

---

## Description

UFW recorded multiple waves of blocked TCP connection attempts targeting port 51820 across at least four days. All observed attempts originate from the 216.180.246.0/24 address block (IPXO LLC, US), which rotates source IPs between scanning sessions, a technique used to evade per-IP threshold-based detection and rate-limiting controls.

A secondary source (Scaleway, FR) contributed 12 attempts on 2026-06-18.

The persistent nature of this activity across multiple days and the consistent use of
rotating IPs within the same /24 block indicates an automated, large-scale internet scanning operation rather than a  targeted attack.
---

## Source Attribution

### IPXO LLC (United States) - 216.180.246.0/24

IP leasing provider. The same /24 block was observed across all scanning sessions, with different source IPs used in each wave. This is consistent with rotating infrastructure operated by a single scanning actor or service.

** IPs observed - 2026-06-18 (kern.log.1) :**

| IP | Events | AbuseIPDB Reports | Confidence |
|----|--------|-------------------|------------|
| 216.180.246.233 | 16 | 777 | 0% |
| 216.180.246.212 | 16 | 748 | 0% |
| 216.180.246.211 | 16 | 740 | 0% |

** IPs observed - 2026-06-21/22 (kern.log) :**

| IP | Events | Notes |
|----|--------|-------|
216.180.246.38 | 16 | Same /24 block, different host |
| 216.180.246.216 | 16 | Same /24 block, different host |

 ** Note on AbuseIPDB confidence scores :** 0% does not indicate benign activity.
 IPXO is an IP leasing company - blocks are rented to third parties, meaning historical reports may reflect previous tenants. The low confidence score indicates reports fall outside the recent evaluation window, not absence of malicious activity.


### Scaleway / Online.net (France)

| IP | Events | AbuseIPDB Reports | Confidence | Categories |
|----|--------|-------------------|------------|------------|
| 51.159.125.200 | 12 | 94 | 100% | Port Scan, Web App Attack |

Actively flagged as malicious at time of analysis. Scaleway VPS infrastructure is frequently abused for automated scanning due to low cost and ease of provisioning.

### Additional IP - Attribution Pending

| IP | Events | Notes |
|----|--------|-------|
| 79.124.56.210 | 1 | Single event, 2026-06-21/22. Attribution not yet verified. |

 ** Action required :** Run `whois 79.124.56.210' and check AbuseIPDB to determine if this is related to the IPXO campaign or an independent event.


---

## Technical Analysis

* Protocol: TCP targeting a WireGuard service (UDP-only)
* Successful interaction with the VPN service was not possible due to protocol mismatch
* All connection attempts were blocked by the UFW default-deny policy before reaching the application layer
* Apache access logs were reviewed and showed no requests originating from 51.159.125.200, confirming effective firewall enforcement
* No authentication attempts, exploitation activity, or lateral movement were observed
* TCP WINDOW=1025 observed in all blocked packets, characteristic of masscan or similar high-speed scanning tools

---

## MITRE ATT&CK Mapping

### Reconnaissance

* **T1595.001 — Active Scanning: Scanning IP Blocks**

---

## Evidence

### Firewall Logs

First observed event:
```
2026-06-21T04:37:13.371195+00:00 shadowserver kernel: [UFW BLOCK] IN=eno1 OUT= MAC=b8:ae:ed:71:64:f7:7c:13:1d:a0:63:d0:08:00 SRC=216.180.246.38 DST=[REDACTED] LEN=44 TOS=0x00 PREC=0x00 TTL=57 ID=16828 PROTO=TCP SPT=21635 DPT=51820 WINDOW=1025 RES=0x00 SYN URGP=0
```
Last observed event:
```
2026-06-22T08:27:33.885358+00:00 shadowserver kernel: [UFW BLOCK] IN=eno1 OUT= MAC=b8:ae:ed:71:64:f7:7c:13:1d:a0:63:d0:08:00 SRC=216.180.246.216 DST=[REDACTED] LEN=44 TOS=0x00 PREC=0x00 TTL=57 ID=6705 PROTO=TCP SPT=21403 DPT=51820 WINDOW=1025 RES=0x00 SYN URGP=0
```
**Attack window:** 2026-06-21 — 2026-06-22 UTC

**Total blocked events:** 60
- 216.180.246.233: 16 events
- 216.180.246.212: 16 events  
- 216.180.246.211: 16 events
- 51.159.125.200:  12 events

### Threat Intelligence

* AbuseIPDB reputation scores manually verified during investigation

---

## Conclusion

The activity is consistent with large-scale automated reconnaissance targeting publicly reachable infrastructure.

No indicators of compromise were identified, and no unauthorized access occurred.

The UFW firewall configuration and VPN-first architecture functioned as intended, successfully preventing interaction with protected services.

---

## Response Actions

**Action Taken:** None required. Event documented as baseline external reconnaissance activity.

**Escalation:** Not required.
