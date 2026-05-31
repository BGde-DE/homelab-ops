# Backup / Restore / DR (anonymisiert)

Dieses Dokument beschreibt die Backup- und DR-Strategie als **Pattern**.
Konkrete IPs, Hostnames, Datastore-Namen, Schlüsselmaterial und vollständige Export-Konfigurationen werden absichtlich nicht veröffentlicht.

---

## 1) Ziele / Threat Model (kurz)

Schutz vor:
- Ausfall einzelner Services/VMs/LXCs (schneller Restore)
- Hypervisor-Ausfall (Compute-Host weg)
- NAS-Ausfall (Storage-Domäne weg)
- Standort-Ausfall (Offsite notwendig)
- Bedienfehler (Delete/Overwrite), logische Schäden

---

## 2) Datenfluss (abstrahiert)

```text
PVE
 ├─(Backups)→ PBS0 (lokal, gleiche Host-Domäne)        [schneller Restore]
 └─(Backups)→ PBS1 (NAS-0, TrueNAS SCALE)              [zweite Domäne]

NAS-0
 ├─(Replikation)→ NAS-1                                [zweite lokale Kopie]
 └─(WireGuard)→ NAS-2 (offsite)                        [Standort-Trennung]
```

---

## 3) PBS: Betrieb (Backups + Verify)

### 3.1 Backup-Gruppen (Prinzip)
- Workloads (VMs und LXCs) werden als PBS-Backup-Gruppen gesichert.
- Es existieren regelmäßige Verify-Jobs; der Verify-Status wird geprüft (Ziel: “All OK”).
- Owner/Identitäten werden in dieser öffentlichen Doku anonymisiert (z. B. `<PVE_USER>@<REALM>`).

### 3.2 Zeitplan
- **Backup-Rhythmus: täglich** (nachts / außerhalb der Hauptnutzungszeit).

### 3.3 Verschlüsselung (Threat-Model sauber getrennt)
- **PBS Backup-Verschlüsselung (clientseitig):** aktuell nicht aktiviert
- **Verschlüsselung at rest:** ja – die PBS-Datastores liegen auf TrueNAS und sind dort vollständig verschlüsselt (Storage-/Dataset-Ebene).

**Einordnung**
- At-rest Verschlüsselung schützt v. a. bei Verlust/Diebstahl von Datenträgern bzw. NAS-Hardware.
- Gegen Szenarien wie „Backup-Repo wird logisch kompromittiert“ helfen zusätzlich: restriktive Rechte, getrennte Credentials, immutability/append-only, Offsite-Kopie.

### 3.4 Verify / Integrität
- Verify ist Teil des Betriebs (nicht nur “Backup vorhanden”, sondern “Backup konsistent”).
- Einträge wie “All OK (old)” sind ein operativer Reviewpunkt (Verify-Runs regelmäßig aktualisieren).

---

## 4) NAS-Replikation & Offsite (täglich)

- **NAS-0 → NAS-1:** tägliche Replikation als zweite lokale Kopie
- **NAS-0 → NAS-2 (offsite):** tägliche Sicherung über WireGuard (separate Location)

---

## 5) Restore-Checks (Betriebsprinzip)

- Restore-Tests als Stichprobe:
  - isolierter Restore (separates Netz / Test-VM/LXC)
  - Smoke-Checks: Boot, Login, Kernfunktion, relevante Logs
  - kurze Dokumentation („was wurde getestet / Ergebnis / Auffälligkeiten“)

---

## 6) Ransomware-/Manipulations-Resilienz (Roadmap)

Zusätzliche Schutzmaßnahmen sind geplant, z. B.:
- immutability/WORM/append-only (wo sinnvoll)
- getrennte Credentials/Scopes (Least Privilege)
- Offline-/Airgap-Überlegungen für besonders kritische Daten
- regelmäßiger DR-Test (z. B. quartalsweise „Worst Case“-Restore-Übung)

---