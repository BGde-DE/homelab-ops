# Runbook: Restore-Test (PBS)

Ziel: Backups gelten erst als „gut“, wenn ein Restore (stichprobenartig) erfolgreich getestet wurde.

## 1) Auswahl
- 1 VM oder 1 LXC aus kritischer Kategorie auswählen (z. B. Nextcloud/Moodle/DNS/Firewall-abhängig).
- monatlich rotieren (nicht immer dieselbe Instanz).

## 2) Vorbereitung (Isolation)
- Restore in isoliertem Test-Segment oder temporär ohne WAN-Exposure.
- Falls nötig: Test-VM/LXC mit geänderter IP / ohne Default-Route starten, um Konflikte zu vermeiden.

## 3) Restore durchführen (Prinzip)
- Restore aus PBS Snapshot (letzter „known good“).
- Ziel: separater Test-Name/ID (nicht überschreiben).

## 4) Checks (Minimal)
- Boot ok, keine Filesystem-Errors
- Login ok (UI/SSH je nach Service)
- Kernfunktion ok (z. B. Web-Login, DB-Connect, 1–2 Kern-Workflows)
- Logs kurz prüfen (Fehler/Crashloops)

## 5) Ergebnis dokumentieren (kurz)
- Datum
- Objekt (VMID/CTID anonymisiert)
- Snapshot-Zeitpunkt
- Ergebnis: OK / Auffälligkeiten
- Falls Auffälligkeiten: Ticket/Notiz + Follow-up

## 6) Cleanup
- Test-Instanz wieder entfernen oder ausgeschaltet lassen (klar markieren).
- Wenn Restore „nur“ verifiziert werden sollte: sofort wieder löschen.

## Optional: PBS Verify-Status prüfen
- Verify-Jobs laufen regelmäßig und werden reviewed.
- Einträge wie „All OK (old)“ sind ein Reviewpunkt (Verify-Runs aktualisieren).