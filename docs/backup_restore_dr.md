# Backup / Restore / DR (High Level)

## Ziele (RPO/RTO, grob)
Kontext: privat/familiär + ehrenamtlicher Betrieb (kein SLA), aber mit dem Anspruch, dass wichtige Services im Fehlerfall zeitnah wieder verfügbar sind.

- **RPO (typisch):** 24h (tägliche Wiederherstellbarkeit)
- **RTO (typisch):** gleicher Tag (einige Stunden) für wichtige Services, sonst 24h+

## Scope (abstrahiert)
- **Tier 1:** Identity/DNS/Reverse Proxy/Backup-Infrastruktur (Basis)
- **Tier 2:** Produktiv genutzte Services (Collab/Utility)
- **Tier 3:** Komfort/Games/Experimente

## Was wird gesichert (Prinzip)
- Image-/Snapshot-Backups für VMs/LXC (schneller Restore)
- Zusätzlich app-nahe Daten (z. B. DB-Dumps + Dateien), wo sinnvoll

## Restore-Check: Definition „bestanden“
- Service startet im isolierten Testnetz
- Login/Grundfunktion ok (z. B. UI erreichbar + Auth funktioniert)
- Daten vorhanden (stichprobenartig)
- Logs ohne wiederkehrende Fatal Errors
- Dauer/Probleme dokumentiert (Runbook/Notizen)
