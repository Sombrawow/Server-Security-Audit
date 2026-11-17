# Persistent Reconnaissance Campaign Against WireGuard Port (51820/TCP)

**Incident ID:** INC-2026-001
**Initial Detection:** 2026-06-18
**Last Activity Observed:** 2026-06-22
**Severity:** Low
**Status:** Closed — Monitoring
**Analyst:** Javier García

---

## Description

UFW recorded multiple waves of blocked TCP connection attempts targeting port 51820
across at least four days. The primary source is the 216.180.246.0/24 address block,
which rotates source IPs between scanning sessions — a technique used to evade
per-IP threshold-based detection and fail2ban thresholds.

A secondary source (Scaleway, FR / 51.159.125.200) contributed 12 attempts on
2026-06-18. A third actor (Tamatiya EOOD, Bulgaria / 79.124.56.210), categorized as
active Brute-Force infrastructure with 100% AbuseIPDB confidence, generated a single
probe on 2026-06-21/22.

All attempts were blocked before reaching the WireGuard process. No impact occurred.

---

## Source Attribution

### 216.180.246.0/24 — Rotating Scanner Infrastructure

**IP ownership chain (whois):**
```
NETWORK TRANSIT HOLDINGS LLC (NTHL, US)
  └─ IPXO LLC (IP leasing, US)
       └─ Internet Utilities NA LLC
            └─ Private Customer — Massy, France 91300 (registered 2025-08-18)
```

The /24 is ultimately assigned to an anonymous private customer. Two IPs in this block
present reverse DNS (PTR) records pointing to `deepfield.net` (Nokia network
intelligence service).

> **⚠️ PTR records are not authentication.** Reverse DNS entries can be configured
> arbitrarily by the IP block owner. The PTR values `crawler038.deepfield.net` and
> `crawler216.deepfield.net` are not verified as belonging to Nokia/DeepField's
> legitimate scanning infrastructure. The anonymous Private Customer ownership chain
> is inconsistent with legitimate corporate research infrastructure. These PTR records
> are likely set to evade automated blocks.

**IPs observed — 2026-06-18 (kern.log.1):**

| IP | Events | AbuseIPDB Reports | Confidence | Last Report |
|----|--------|------------------|------------|-------------|
| 216.180.246.233 | 16 | 777 | 0% | Outside window |
| 216.180.246.212 | 16 | 748 | 0% | Outside window |
| 216.180.246.211 | 16 | 740 | 0% | Outside window |

**IPs observed — 2026-06-21/22 (kern.log):**

| IP | Events | AbuseIPDB Reports | Confidence | PTR | Last Report |
|----|--------|------------------|------------|-----|-------------|
| 216.180.246.38 | 16 | **1,197** (307 sources) | 0% | crawler038.deepfield.net | 3h ago |
| 216.180.246.216 | 16 | **963** (239 sources) | 0% | crawler216.deepfield.net | 14h ago |

> **Note on 0% confidence with high report count:** IPXO leases blocks to rotating
> tenants. Historical reports may reflect previous occupants. 0% indicates reports
> fall outside the recent evaluation window, not absence of malicious activity.
> Report volume (963–1,197 from hundreds of sources) indicates sustained historical
> abuse from this /24.

---

### Scaleway / Online.net (France)

| IP | Events | AbuseIPDB Reports | Confidence | Categories | Last Report |
|----|--------|------------------|------------|------------|-------------|
| 51.159.125.200 | 12 | 94 | 100% | Port Scan, Web App Attack | Verified 2026-06-18 |

Actively flagged at time of analysis. Scaleway VPS infrastructure is frequently
abused for automated scanning due to low provisioning cost.

---

### Tamatiya EOOD (Bulgaria) — ⚠️ Elevated Concern

| IP | Events | AbuseIPDB Reports | Confidence | Categories | Last Report |
|----|--------|------------------|------------|------------|-------------|
| 79.124.56.210 | 1 | **3,473** (236 sources) | **100%** | **Brute-Force** | 51 min ago |

**Whois:**
```
inetnum:   79.124.56.0 - 79.124.56.255
netname:   Tamatiya-EOOD
country:   BG (Bulgaria)
ASN:       AS50360
Hostname:  ip-56-210.4vendeta.com
```

This IP represents the highest-confidence threat in this incident. Key indicators:

- **100% AbuseIPDB confidence** — actively malicious within the recent evaluation window
- **Brute-Force categorization** — not reconnaissance; this actor targets authentication services
- **3,473 reports from 236 distinct sources** — widespread, persistent, multi-target activity
- **Active 51 minutes before analysis** — operational at time of investigation
- **Single probe against 51820** — consistent with service discovery, although one
  SYN alone is not enough to infer intent against this host

The Brute-Force categorization describes this IP's reported behaviour elsewhere; here
only one blocked SYN was observed. It is treated as the highest-priority source for
monitoring, not as evidence of a credential attack against this server.

---

## Technical Analysis

- **Protocol mismatch:** All attempts used TCP against port 51820. WireGuard operates
  exclusively over UDP. Successful VPN interaction was impossible regardless of firewall
  state.

- **Scanner fingerprint — masscan:** All blocked packets from the 216.180.246.0/24 block
  show `WINDOW=1025`. Standard operating systems do not generate this TCP window size.
  This value is a documented fingerprint of **masscan** and similar high-speed internet
  scanning tools, confirming fully automated origin.

- **Evasion technique:** Use of 5 distinct IPs within the same /24 across multiple days,
  each generating ≤16 events, is consistent with deliberate distribution below fail2ban
  default thresholds (typically 5–10 failures per IP per window).

- **UFW enforcement:** All events blocked by default-deny policy before reaching the
  application layer. No packets were delivered to the WireGuard or any other process.

- **Apache logs verified:** No requests from 51.159.125.200 in
  `/var/log/apache2/access.log`. UFW enforcement confirmed effective despite this IP's
  Web App Attack categorization.

- **No post-compromise indicators:** No authentication attempts, exploitation, privilege
  escalation, or lateral movement observed in any log source.

---

## Event Timeline

| Date | Source IPs | Events | Log Source |
|------|-----------|--------|------------|
| 2026-06-18 | 216.180.246.233/212/211, 51.159.125.200 | 60 | kern.log.1 (rotated) |
| 2026-06-21 | 216.180.246.38, 79.124.56.210 | 17 | kern.log |
| 2026-06-22 | 216.180.246.216 | 16 | kern.log |

**Total blocked events across campaign:** 93
**Campaign window:** 2026-06-18 — 2026-06-22T08:27:33 UTC

---

## MITRE ATT&CK Mapping

### Reconnaissance

- **T1595.001 — Active Scanning: Scanning IP Blocks**
  Evidence: Systematic TCP probes from rotating IPs within 216.180.246.0/24 across
  multiple days, plus a single blocked probe from 79.124.56.210.

---

## Evidence

### Firewall Logs — Representative Samples (kern.log)

**First observed event in current log (2026-06-21):**
```
2026-06-21T04:37:13.371195+00:00 <HOSTNAME> kernel: [UFW BLOCK] IN=eno1 OUT= \
MAC=<MAC_HOST>:<MAC_GATEWAY>:08:00 SRC=216.180.246.38 DST=[REDACTED] \
LEN=44 TOS=0x00 PREC=0x00 TTL=57 ID=16828 PROTO=TCP SPT=21635 DPT=51820 \
WINDOW=1025 RES=0x00 SYN URGP=0
```

**Last observed event (2026-06-22):**
```
2026-06-22T08:27:33.885358+00:00 <HOSTNAME> kernel: [UFW BLOCK] IN=eno1 OUT= \
MAC=<MAC_HOST>:<MAC_GATEWAY>:08:00 SRC=216.180.246.216 DST=[REDACTED] \
LEN=44 TOS=0x00 PREC=0x00 TTL=57 ID=6705 PROTO=TCP SPT=21403 DPT=51820 \
WINDOW=1025 RES=0x00 SYN URGP=0
```

> Events from 2026-06-18 (initial detection wave) are located in `/var/log/kern.log.1`.
> WINDOW=1025 fingerprint is consistent across all observed events from both log files.

---

### Threat Intelligence Summary

| IP | Reports | Confidence | Category | ASN/Org | Verified |
|----|---------|------------|----------|---------|---------|
| 216.180.246.233 | 777 | 0% | Port Scan | IPXO/NTHL (US) | ✅ 2026-06-18 |
| 216.180.246.212 | 748 | 0% | Port Scan | IPXO/NTHL (US) | ✅ 2026-06-18 |
| 216.180.246.211 | 740 | 0% | Port Scan | IPXO/NTHL (US) | ✅ 2026-06-18 |
| 51.159.125.200 | 94 | 100% | Port Scan, Web App | Scaleway (FR) | ✅ 2026-06-18 |
| 216.180.246.38 | 1,197 | 0% | Port Scan | NTHL/Private (FR) | ✅ 2026-06-22 |
| 216.180.246.216 | 963 | 0% | Port Scan | NTHL/Private (FR) | ✅ 2026-06-22 |
| 79.124.56.210 | **3,473** | **100%** | **Brute-Force** | Tamatiya EOOD (BG) | ✅ 2026-06-22 |

---

## Conclusion

This incident documents persistent, automated reconnaissance against the port
forwarded to this host from the internet (51820). Five distinct IPs from the
216.180.246.0/24 block confirm an ongoing scanning operation using IP rotation
to evade threshold-based controls.

The most significant actor is **79.124.56.210** (Tamatiya EOOD, Bulgaria): 100%
AbuseIPDB confidence, 3,473 reports, Brute-Force category, active at time of
analysis. Its single probe against 51820 was blocked at the firewall layer before
reaching any service.

No indicators of compromise were identified. The protocol mismatch (TCP vs UDP)
and UFW default-deny policy prevented any interaction with protected services.
External TCP scan (nmap -sS, 2026-06-18) showed no open TCP ports from the internet.
UDP exposure was not verified by that scan (`-sS` is TCP-only); a `nmap -sU -p 51820`
check from an external host is pending.

**Side finding:** the blocked TCP packets to 51820 reached `eno1`, which means the
router's port forward for 51820 also covers TCP. WireGuard only needs UDP, so the
forward should be restricted to UDP.

---

## Response Actions

**Action Taken:** Documented as baseline external reconnaissance activity.

**Escalation:** Not required.

**Completed:**
- [x] whois verified for all 7 IPs
- [x] AbuseIPDB scores verified for all 7 IPs
- [x] Apache logs checked for 51.159.125.200 — no hits confirmed
- [x] PTR record anomaly for deepfield.net hostnames noted and documented

**Monitoring:**
- [ ] Re-run `grep "UFW BLOCK" kern.log | grep "DPT=51820"` weekly
- [ ] Flag any new IPs not in the 216.180.246.0/24 block for individual investigation
- [ ] Flag any future event from 79.124.56.210 regardless of port — Brute-Force actor
- [ ] Restrict the router port forward for 51820 to UDP only
- [ ] Verify UDP exposure from an external host (`nmap -sU -p 51820`)
