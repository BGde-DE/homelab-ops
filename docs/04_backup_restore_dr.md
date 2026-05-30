# Backup / Restore / DR (High Level)

## Ziel
- Mehrstufige Backups inkl. Offsite-Kopie
- Restore-Fähigkeit ist nachweisbar (Restore-Checks)

## Backup-Kette (abstrahiert)
- Proxmox VE → Proxmox Backup Server (lokal)
- zusätzliche Backup-Stufe → NAS (lokal)
- NAS → NAS (zweite lokale Stufe)
- Offsite-Kopie über VPN (WireGuard) zu externem Ziel

## Restore-Checks (Beispiel)
- monatlich: Restore-Test einer kleinen VM/LXC in isoliertem Testnetz
- quartalsweise: Restore eines größeren Services (inkl. abhängiger Daten)

Checkliste:
1. Restore in isoliertem Netz
2. Service-Health prüfen (WebUI/Ports/Logs)
3. Dauer + Probleme dokumentieren
4. Testsystem entfernen
