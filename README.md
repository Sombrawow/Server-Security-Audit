# shadowserver — Security Audit

Self-conducted security audit of a 24/7 automated trading server running MetaTrader 5 on Ubuntu 24.04 LTS.

## Purpose

This repository documents a host-level security audit performed on a personal trading server as part of a cybersecurity portfolio. The audit covers network exposure, access control, running services, firewall configuration, scheduled tasks, and credential handling.

## System Overview

- **OS:** Ubuntu 24.04.4 LTS
- **Purpose:** Automated XAUUSD trading (MetaTrader 5 / Wine)
- **Access:** WireGuard VPN + SSH + xrdp
- **Exposure:** LAN-only with WireGuard for remote access

## Repository Structure

```
.
├── README.md
├── Security_Audit_Report.md
└── Remediation_Checklist.md
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

## Methodology

Manual audit using standard Linux enumeration tools:
- `ss -tlnp` / `ss -ulnp` — port enumeration
- `systemctl list-units` — service inventory
- `ufw status verbose` + `iptables -L` — firewall analysis
- `wg show` / `wg0.conf` — VPN configuration review
- `/etc/ssh/sshd_config` — SSH hardening review
- `find / -perm -4000` — SUID binary audit
- `crontab -l` / `systemctl list-timers` — scheduled task review
- `last` / `lastb` — authentication log review

## Disclaimer

This audit was performed on infrastructure owned and operated by the auditor. All findings and remediations are documented for educational and portfolio purposes. No sensitive credentials, private keys, or personally identifiable information are included in this repository.
