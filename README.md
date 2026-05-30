# Ops-Portfolio (Sysadmin · Netzwerk · Support)

Anonymisiertes Operations-Portfolio (homelab-/betriebsnah): Planung, Betrieb und Troubleshooting von Infrastruktur/Services mit Fokus auf **Security/Segmentierung**, **stabile Deployments**, **Backup/Restore (DR)** und **Runbooks**.

## Highlights (Recruiter-Quickscan)
- Netzwerksegmentierung (VLAN/Policy) + Firewall-Konzept
- Reverse-Proxy-Pattern für interne Services (ohne öffentliche Exposure)
- Backup-Kette inkl. Restore-Checks (DR-Denke)
- Runbooks für wiederholbare Fehleranalyse / Support-Fälle
- Beispiel-Service: Moodle (Self-hosted auf Proxmox LXC)

## Quick links (Start here)
- **Systemübersicht / Architektur:** `docs/10_systemuebersicht.md`
- **Netzwerk & Security (Segmentierung/Policy):** `docs/02_netzwerk_security.md`
- **Backup/Restore/DR (Restore-Checks):** `docs/04_backup_restore_dr.md`
- **Runbooks / Troubleshooting:** `docs/06_runbooks.md`
- **Service-Beispiel (Moodle, Proxmox LXC):** `docs/services/moodle.md`

## Stack (Auszug)
- Proxmox VE · Proxmox Backup Server
- OPNsense (Firewall, VLANs, Unbound DNS, HAProxy)
- Docker (Compose)
- TrueNAS SCALE (Storage / Backup-Stufen)
- OpenWRT (APs)
- Home Assistant + KNX

## Scope / Kontext
- Betrieb interner Web-/Collab-/Utility-Services (privat/familiär)
- Dokumentation ist absichtlich "high level" und ohne Identifikatoren

## Was absichtlich NICHT enthalten ist
- keine realen IPs/Subnetze, Domains oder Hostnamen
- keine Portlisten / Rule-Exports
- keine Keys/Tokens, vollständigen Config-Backups oder Screenshots mit sensiblen Daten
