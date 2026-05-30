# Ops-Portfolio (Sysadmin · Netzwerk · Support)

Dieses Repository ist ein anonymisiertes Portfolio aus meinem Homelab-/Betriebsalltag.
Fokus: stabile Services, Netzwerksegmentierung, Backup/Restore, Troubleshooting.

## Stack (Auszug)
- Proxmox VE, Proxmox Backup Server
- OPNsense (Firewall, VLANs, Unbound DNS, HAProxy)
- Docker (Compose)
- TrueNAS SCALE (Storage/Backup-Stufen)
- OpenWRT (mehrere APs)
- Home Assistant + KNX
- Betrieb von Web-/Collab-/Utility-Services sowie ein paar Ports für Gameserver (privat/familiär)

## Was absichtlich NICHT enthalten ist
- keine realen IPs/Subnetze, Domains, Hostnamen
- keine Portlisten / Rule-Exports
- keine Keys, Tokens, Config-Backups oder Screenshots mit sensiblen Daten

## Inhalte
- docs/02_netzwerk_security.md – Segmentierung/Policy (high level)
- docs/03_services_reverse_proxy.md – Reverse Proxy Pattern (ohne echte vHosts)
- docs/04_backup_restore_dr.md – Backup-Kette + Restore-Checks
- docs/06_runbooks.md – Support-Runbooks & Troubleshooting
