# Runbooks (Support / Betrieb)

## VPN verbindet nicht
- Client: DNS/Internet/Uhrzeit/Config
- Firewall: WireGuard up? Logs? Regeln korrekt?
- Routing: AllowedIPs, Netzkollisionen
- Verifikation: Zugriff auf definiertes internes Ziel

## Reverse Proxy: 503 / falscher Dienst
- DNS zeigt korrekt?
- Proxy-Regel/ACL/vHost korrekt?
- Backend up? Health Check grün?
- Firewall: Proxy → Backend erlaubt?
- App-Logs prüfen

## WLAN langsam/instabil (APs)
- RSSI/Interferenz/Kanal
- VLAN-Tagging/Switchport Errors
- Vergleichstest: Client per Kabel

## Proxmox VM/LXC startet nicht
- Storage/Platz/I/O
- Locks/Task Logs
- Wenn schnell: Restore-Test aus Backup in Testnetz
