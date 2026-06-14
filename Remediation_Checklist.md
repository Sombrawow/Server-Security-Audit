# Remediation Checklist — shadowserver

Use this file to track remediation progress. Update status as each item is completed.

## Phase 1 — Critical (apply immediately)

- [x] **CRIT-01** — Remove `SaveConfig = true` from `/etc/wireguard/wg0.conf`
- [x] **CRIT-02** — Add `ListenAddress 10.8.0.1` and `ListenAddress 192.168.0.16` to `/etc/ssh/sshd_config`
- [x] **CRIT-02** — Remove `22/tcp ALLOW IN Anywhere` and `2222/tcp ALLOW IN Anywhere` from UFW
- [x] **MED-05** — Set `X11Forwarding no` in `/etc/ssh/sshd_config`
- [x] **LOW-03** — `sudo ufw logging medium`
- [x] **LOW-01** — `sudo systemctl disable --now cups cups-browsed`
- [x] **MED-02** — `sudo systemctl disable --now avahi-daemon`
- [x] **LOW-02** — `sudo chmod u-s /usr/bin/pkexec`

## Phase 2 — High (next maintenance window / weekend)

- [x] **HIGH-02** — Determine if Apache/Nextcloud is still needed; disable if not
- [x] **HIGH-03** — Determine if MariaDB is still needed; disable if not
- [x] **HIGH-04** — `sudo systemctl disable --now postfix`
- [x] **HIGH-01** — Fix UFW rule for RDP: restrict to `wg0` only
- [x] **MED-04** — Full UFW ruleset reset and rebuild per report recommendations

## Phase 3 — Medium/Low (next weekend)

- [x] **MED-01** — Add `interfaces = lo wg0 eno1` and `bind interfaces only = yes` to `/etc/samba/smb.conf`
- [x] **MED-01** — `sudo systemctl disable --now winbind`
- [x] **MED-03** — Clean up duplicate crontab entries (root and user)
- [x] **MED-06** — Document or remove WireGuard peer `10.8.0.3` (84.125.103.70)
- [x] **LOW-04** — Disable `check-nextcloud.timer` if Nextcloud is not running
- [x] **LOW-04** — Remove `nextcloud-backup.sh` cron entry if Nextcloud is absent

## Notes

- Always test SSH connectivity from a second terminal before closing current session after sshd changes
- Apply UFW changes during market close (weekends) to avoid disrupting trading connectivity
- After rotating WireGuard key (CRIT-01): update all peer devices with new server public key
