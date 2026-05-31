# Runbook: PVE Update (Kernel + Cleanup)

1. Update ausführen:
   - `apt update && apt full-upgrade`

2. Reboot (bei Kernel-Update):
   - `shutdown -r +1 "PVE kernel update reboot"`

3. Nach dem Boot prüfen:
   - `uname -r`
   - `pveversion -v`
   - `systemctl is-active pveproxy pvedaemon pvestatd`
   - `qm list && pct list`
   - `zpool status`

3.1 Optional: PBS lokal auf dem Host prüfen (falls installiert)
   - `systemctl is-active proxmox-backup proxmox-backup-proxy`
   - `journalctl -u proxmox-backup -u proxmox-backup-proxy -n 50 --no-pager`

4. Kernel-Cleanup (nach erfolgreichem Boot):
   - `apt autoremove --purge`
   - optional: alte Kernel gezielt purgen
   - `update-grub`
   - `proxmox-boot-tool kernel list`