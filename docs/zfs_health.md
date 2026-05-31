# ZFS Health / Pool-Status (anonymisiert)

Ziel dieses Dokuments: ZFS-Layouts und Health-Checks nachvollziehbar dokumentieren, ohne Disk-Seriennummern/Modelle zu veröffentlichen.

---

## 1) Pools (Pattern)

- `local_RAID10`
  - Zweck: Workload-Storage (VM/LXC)
  - Layout: mehrere Mirrors (RAID10-Pattern)

- `pool_backup`
  - Zweck: lokale Backup-Stufe
  - Layout: Mirror

---

## 2) Health & Scrub

- Status: Pools sind **ONLINE**
- Scrub: erfolgreich, **0 errors**, keine Reparaturen nötig (Stand siehe Output unten)
- Betrieb: Scrub-Status wird regelmäßig geprüft (Monitoring/Review)

---

## 3) Feature-Flags / `zpool upgrade`

`zpool status` weist darauf hin, dass nicht alle angefragten Pool-Features aktiv sind.

**Bewertung**
- Pro: neue Features/Verbesserungen nach Upgrade
- Contra: mögliches Recovery-/Kompatibilitätsrisiko mit älterer Software

**Betriebsentscheidung**
- `zpool upgrade` nur als geplante Änderung (Wartungsfenster + Rollback-/Recovery-Überlegung).

---

## 4) Sanitisiertes Beispiel: `zpool status`

```text
  pool: local_RAID10
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
        The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
        the pool may no longer be accessible by software that does not support
        the features. See zpool-features(7) for details.
  scan: scrub repaired 0B in 02:02:51 with 0 errors on <DATE_REDACTED>
config:

        NAME                                            STATE     READ WRITE CKSUM
        local_RAID10                                    ONLINE       0     0     0
          mirror-0                                      ONLINE       0     0     0
            <DISK>                                      ONLINE       0     0     0
            <DISK>                                      ONLINE       0     0     0
          mirror-1                                      ONLINE       0     0     0
            <DISK>                                      ONLINE       0     0     0
            <DISK>                                      ONLINE       0     0     0
          mirror-2                                      ONLINE       0     0     0
            <DISK>                                      ONLINE       0     0     0
            <DISK>                                      ONLINE       0     0     0

errors: No known data errors

  pool: pool_backup
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
        The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
        the pool may no longer be accessible by software that does not support
        the features. See zpool-features(7) for details.
  scan: scrub repaired 0B in 14:33:58 with 0 errors on <DATE_REDACTED>
config:

        NAME                                 STATE     READ WRITE CKSUM
        pool_backup                          ONLINE       0     0     0
          mirror-0                           ONLINE       0     0     0
            <DISK>                           ONLINE       0     0     0
            <DISK>                           ONLINE       0     0     0

errors: No known data errors
```

---

## 5) Sanitizing-Kommandos (für Wiederholung)

```bash
# Disk IDs/Modelle/Serials entfernen:
zpool status | sed -E 's/(ata-[^ ]+)/<DISK>/g'

# Optional: Datum entschärfen:
zpool status | sed -E 's/(ata-[^ ]+)/<DISK>/g' | sed -E 's/on .*/on <DATE_REDACTED>/'
```