# Coordinated Reconnaissance Scan Against WireGuard (51820/TCP)

**Incident ID:** INC-2026-001
**Detection Date:** 2026-06-18
**Severity:** Low
**Status:** Closed — No Action Required
**Analyst:** sombra_0

---

## Description

UFW recorded 60 blocked TCP connection attempts targeting port 51820 from four external IP addresses distributed across two autonomous systems (ASNs).

The pattern observed from three IP addresses within the same /24 subnet, each generating an identical number of connection attempts (16 each), suggests coordinated scanning activity designed to evade per-IP threshold-based detection mechanisms.

---

## Source Attribution

### IPXO LLC (United States)

* 216.180.246.233
* 216.180.246.212
* 216.180.246.211

Observed characteristics:

* IP leasing provider
* Historical AbuseIPDB reports: 740–777
* Current confidence score: 0% (reports fall outside the recent evaluation window)

### Scaleway (France)

* 51.159.125.200

Observed characteristics:

* 94 AbuseIPDB reports
* Current confidence score: 100%
* Actively categorized for port scanning and web application attack activity

---

## Technical Analysis

* Protocol: TCP targeting a WireGuard service (UDP-only)
* Successful interaction with the VPN service was not possible due to protocol mismatch
* All connection attempts were blocked by the UFW default-deny policy before reaching the application layer
* Apache access logs were reviewed and showed no requests originating from 51.159.125.200, confirming effective firewall enforcement
* No authentication attempts, exploitation activity, or lateral movement were observed

---

## MITRE ATT&CK Mapping

### Reconnaissance

* **T1595.001 — Active Scanning: Scanning IP Blocks**

### Discovery

* **T1046 — Network Service Discovery**

---

## Evidence

### Firewall Logs

```
/var/log/kern.log

[UFW BLOCK] SRC=216.180.246.x DPT=51820 PROTO=TCP
[UFW BLOCK] SRC=51.159.125.200 DPT=51820 PROTO=TCP
```

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
