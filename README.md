# Ops-Portfolio (Sysadmin · Netzwerk · Support)

Anonymisiertes Operations-Portfolio (homelab-/betriebsnah): Planung, Betrieb und Troubleshooting von Infrastruktur/Services mit Fokus auf **Security/Segmentierung**, **stabile Deployments**, **Backup/Restore (DR)** und **Runbooks**.

## Quick links (Start here)
- **Systemübersicht / Architektur:** `docs/systemuebersicht.md`
- **Netzwerk & Security (Segmentierung/Policy):** `docs/netzwerk_security.md`
- **Firewall-Regeln:** `docs/firewall.md`
- **Backup/Restore/DR (Restore-Checks):** `docs/backup_restore_dr.md`
- **Service-Beispiel (Moodle, Proxmox LXC):** `services/Moodle`

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
