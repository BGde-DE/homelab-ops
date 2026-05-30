# Backup / Restore / DR (High Level)

## Ziele (RPO/RTO, grob)
- Ziel: Ausfälle ohne Datenverlust-Katastrophe überstehen
- **RPO (typisch):** 24h (kritische Daten ggf. kürzer)
- **RTO (typisch):** 2–4h für Kernservices, 24h für „nice to have“

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
