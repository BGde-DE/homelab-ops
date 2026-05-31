# Runbook: Incident – First 10 Minutes (Checkliste)

Ziel: Schnell Stabilität herstellen, dann Ursachenanalyse.

## 0) Safety / Scope
- Was ist betroffen? (ein Dienst / mehrere Dienste / gesamtes Netz)
- Ist es ein Security-Vorfall-Verdacht? (ungewöhnliche Logins, neue Firewall-Allows, Crypto-miner, plötzliches Exfil)

## 1) Nutzerwirkung / Priorität
- Welche Services sind down?
- Gibt es Workarounds? (Wartungsseite, Read-only, temporär deaktivieren)

## 2) Quick Triage (Infra)
- Hypervisor: CPU/RAM/Disk voll?
- Storage: ZFS/SMART ok?
- Netzwerk: DNS ok? Gateway ok? WAN ok?

Kommandos (Linux):
- `uptime`
- `df -h`
- `free -h`
- `journalctl -p err -n 50 --no-pager`

## 3) Logs – Einstiegspunkte
- Reverse Proxy Logs
- Service Logs (App/DB)
- Firewall Logs (Blocks/IDS)

## 4) Stabilisieren (Stop bleeding)
- Wenn ein Dienst eskaliert (Crashloop/CPU 100%): kontrolliert stoppen/restarten
- Wenn ein Update/Change kurz davor war: Rollback-Pfad prüfen (Snapshot/Restore/Config revert)

## 5) Verifizieren
- Dienst wieder erreichbar? (intern/external je nach Ziel)
- Monitoring/Checks grün?
- Kurznotiz: „was war’s / was wurde getan / Ergebnis“

## 6) Nacharbeit
- Root cause, dauerhafter Fix
- Doku/Runbook ergänzen
- Preventive checks (Alerting/Capacity/Access)