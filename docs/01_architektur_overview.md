# Architektur-Overview

## Ziele
- Segmentiertes Netzwerk nach Schutzbedarf (Clients/IoT/Gast/Mgmt/Server/Automation/AP-Mgmt)
- Zentrale "Frontdoor" für Webservices über Reverse Proxy
- Backup/Restore in mehreren Stufen inkl. Offsite-Kopie
- Betrieb mit Runbooks, Logging und regelmäßigen Checks

## Bausteine (abstrahiert)
- Firewall/Router: OPNsense
  - VLAN Routing + Policies
  - Unbound DNS
  - HAProxy als Reverse Proxy für HTTPS
  - IDS/IPS + Threat-Intel Blocklists
- Virtualisierung: Proxmox VE, Docker
  - VMs/LXC/Docker für Services
- Storage/Backup:
  - Proxmox Backup Server
  - NAS-Stufen + Offsite über VPN (WireGuard)
- WLAN:
  - OpenWRT APs, VLAN-Tagging, getrennte SSIDs

## Leitprinzipien
- Default deny zwischen Segmenten
- Admin-Zugriffe nur aus Management
- Öffentlich erreichbare Services: bewusst, kontrolliert, logging-/update-getrieben
