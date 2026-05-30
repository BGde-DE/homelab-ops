# Checkliste: Dienst öffentlich machen (ohne Details)

1) Risiko/Bedarf
- Muss der Dienst wirklich öffentlich sein?
- Gibt es eingebaute Auth / 2FA / Rollen?

2) Reverse Proxy
- vHost/SNI Routing
- TLS Zertifikat
- Logging aktiv

3) Firewall
- WAN nur Entry-Point (für Web: 80/443) oder gezielt einzelne Ports
- Threat-Intel Blocklists / Allowlisting Policy

4) Betrieb
- Updates eingeplant
- Backup/Restore klar
- Runbook vorhanden (mind. “503 / Login-Probleme / Downtime”)
