# Netzwerk & Security (Policy-Level)

## Segmentierung (VLANs / Zonen)
Typische Segmente:
- AUTOMATION: Home Assistant / KNX / automations-nahe Systeme
- IOT: IoT-Geräte (untrusted)
- GUEST: Gäste (nur Internet)
- CLIENTS: Endgeräte (PC/Laptop/Smartphones)
- MGMT: Administration (WebUIs/SSH/Hypervisor/Storage)
- AP_MGMT: Management der Access Points
- SERVER: Server/Services (VM/LXC/Container)

## Default-Regeln (High Level)
- Default: VLANs voneinander getrennt (Block).
- GUEST → interne Netze: block.
- IOT → interne Netze: block, nur definierte Ausnahmen (z. B. zu HA, DNS/NTP).
- CLIENTS → SERVER: nur benötigte Services.
- MGMT → Infrastruktur: erlaubt (Admin Ports); Zugriff auf MGMT aus anderen VLANs: block.
- AP_MGMT: strikt, nur Management/Updates nach Bedarf.

## DNS (Unbound)
- Zentraler DNS Resolver (Unbound) pro Segment nutzbar.
- Optional: DNS-Filter/Blocklisten; Policies je Segment (z. B. Gäste freier).

## VPN
- WireGuard wird für Remote Access zu ausgewählten internen Services nutzbar.
- **Adminzugänge (WebUIs/SSH/Management)** sind **ausschließlich aus dem MGMT/AP‑MGMT Segment** erlaubt.
- Das MGMT/AP‑MGMT Segment ist **nicht über VPN erreichbar** (bewusste Trennung: Administration nur lokal/physisch im Admin-Netz).

## Defense in Depth
- Segmentierung + minimale Freigaben
- Reverse Proxy als zentraler Entry Point (für Webservices)
- per-Service Auth (App-eigen, Fallback: vorgeschaltete Auth falls nötig)
- IDS/IPS + Threat-Intel Blocklists (WAN)
- Regelmäßige Updates + Backups + Restore-Tests
