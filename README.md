# shadowserver — Security Audit

Self-conducted security audit and active incident tracking for a 24/7 automated trading server running MetaTrader 5 on Ubuntu 24.04 LTS.

> **Analyst:** Javier García — [LinkedIn](https://linkedin.com/in/jvrgs) · 
> [GitHub](https://github.com/Sombrawow)  
> **Audit date:** June 2026  
> **Status:** Active — ongoing monitoring and incident documentation

---
## Purpose

This repository documents a host-level security audit performed on personal infrastructure as part of a cybersecurity portfolio. It covers network exposure, access control, running services, firewall configuration, scheduled tasks, and credential handling.
Beyond the initial audit, the repository serves as an active incident log; documenting real security events detected on the server with full analysis, threat intelligence, and MITRE ATT&CK mapping.

---
## System Overview

- **OS:** Ubuntu 24.04.4 LTS
- **Purpose:** Automated XAUUSD trading (MetaTrader 5 / Wine)
- **Access:** WireGuard VPN + SSH + xrdp
- **Exposure:** Public IP with UFW default-deny; WireGuard port exposed

## Repository Structure

```
.
├── README.md
├── Security_Audit_Report.md     # Full audit findings with evidence
├── Remediation_Checklist.md     # Tracked remediation status
└── findings/
└── INC-2026-001_recon_scan_wg0.md   # Incident report: WireGuard recon scan
```

## Audit Scope

| Area | Covered |
|------|---------|
| Network exposure (open ports) | ✅ |
| Firewall rules (UFW) | ✅ |
| Running services | ✅ |
| SSH configuration | ✅ |
| VPN configuration (WireGuard) | ✅ |
| User accounts and sudo | ✅ |
| SUID/SGID binaries | ✅ |
| Scheduled tasks (cron / systemd timers) | ✅ |
| Authentication logs | ✅ |
| OS patch status | ✅ |

## Findings Overview

| Severity | Count |
|----------|-------|
| 🔴 Critical | 2 |
| 🟠 High | 5 |
| 🟡 Medium | 6 |
| 🔵 Low | 4 |

See [Security_Audit_Report.md](./Security_Audit_Report.md) for full details, evidence, and remediations.

## Incident Reports

Real security events detected post-audit, documented following a structured incident response format: description, source attribution, technical analysis, MITRE ATT&CK mapping, evidence, and disposition.

| ID | Date | Summary | Severity | Status |
|----|------|---------|----------|--------|
| [INC-2026-001](./findings/INC-2026-001_recon_scan_wg0.md) | 2026-06-18 | Coordinated recon scan targeting WireGuard (51820/TCP) from 4 IPs across 2 ASNs | Low | Closed |

---
## Methodology

Manual audit using standard Linux enumeration tools:
**Enumeration**
- `ss -tlnp` / `ss -ulnp` — port enumeration
- `systemctl list-units` — service inventory
- `ufw status verbose` + `iptables -L` — firewall analysis
- `wg show` / `wg0.conf` — VPN configuration review
- `/etc/ssh/sshd_config` — SSH hardening review
- `find / -perm -4000` — SUID binary audit
- `crontab -l` / `systemctl list-timers` — scheduled task review
- `last` / `lastb` — authentication log review
  
**Threat Intelligence**
- AbuseIPDB — IP reputation scoring and historical abuse reports

---
## Disclaimer

This audit was performed on infrastructure owned and operated by the auditor. All findings and remediations are documented for educational and portfolio purposes. No sensitive credentials, private keys, or personally identifiable information are included in this repository.
