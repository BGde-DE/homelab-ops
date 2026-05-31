# Proxmox VE (PVE) – Betrieb & Security-Standards (anonymisiert)

Dieses Dokument beschreibt mein Proxmox-Setup als **Betriebs-/Security-Pattern** (kein Config-Dump).
Ziel: Entscheidungen und Standards nachvollziehbar machen, ohne identifizierende Details (IPs, Fingerprints, Hostnames, Ports) zu veröffentlichen.

---

## 1) Plattform / Versionen (Auszug)

- Proxmox VE: **9.2.x**
- Kernel (aktiv): **7.0.x** (PVE Kernel)
- Virtualisierung: **QEMU/KVM** + **LXC**
- Storage: **ZFS** + **LVM-thin**
- Backup: **Proxmox Backup Server (PBS)** (lokal) + **PBS auf NAS (TrueNAS SCALE)**

---

## 2) Node-Profil (abstrahiert)

- Single Node (kein Cluster)
- CPU: Intel Core i3 (4 Cores)
- RAM: ~64 GiB
- Storage-Pattern:
  - NVMe (~2 TB) als System-/lokales Volume-Backend (LVM-thin)
  - ZFS-Pool für Workloads: RAID10-Pattern (mehrere Mirrors) auf HDDs (2 TB Klasse)
  - Separater ZFS-Pool für Backups: Mirror auf HDDs (4 TB Klasse)

**Security-/Betriebsbegründung**
- Trennung „Workload-Storage“ vs. „Backup-Storage“ reduziert Blast Radius (Fehlbedienung/Defekt/Incident).
- ZFS liefert robuste Integritätseigenschaften + gute Restore-Workflows.

---

## 3) Storage (PVE Storages – Pattern)

Konzeptuell existieren folgende Storage-Typen:
- `local` (dir): ISO/Templates/Import, minimal
- `local-lvm` (lvmthin): VM Images + LXC rootfs (lokal, thin-provisioned)
- `zfspool` (Workload-Pool): VM Images + LXC rootfs (primärer Workload-Storage)
- `dir` (Backup-Pool): lokale Backup-Stufe auf separatem Pool
- `pbs` (PBS Datastore): dedizierte Backup-Stufe (Backup/Restore über PBS)

---

## 4) ZFS Health (anonymisiert)

Hinweis: Details/Outputs separat in `zfs_health.md`.

### 4.1 Pools / Layout (Pattern)
- `local_RAID10`: ONLINE, RAID10-Pattern (mehrere Mirrors) für Workloads
- `pool_backup`: ONLINE, Mirror für lokale Backup-Stufe

### 4.2 Integrität / Scrubs
- Scrubs laufen erfolgreich durch (keine Reparaturen nötig, keine bekannten Datenfehler).
- Scrub-Ergebnisse werden als Teil des Betriebs regelmäßig geprüft.

### 4.3 Feature-Flags / `zpool upgrade`
- `zpool status` meldet, dass nicht alle angefragten Features aktiv sind.
- Ein `zpool upgrade` wird bewusst als Change behandelt:
  - Pro: neue Features/Verbesserungen
  - Contra: mögliche Inkompatibilität mit älterer Software/Recovery-Tools
  - Entscheidung: Upgrade nur nach Review und geplantem Wartungsfenster

---

## 5) Netzwerk (abstrahiert, Security-Sicht)

### 5.1 Grundaufbau
- 2 physische NICs → 2 Linux Bridges (`vmbr*`)
  - Bridge A: internes Server-/Management-Segment
  - Bridge B: Uplink-/Transit-/extern-nahes Segment

### 5.2 Prinzipien
- Segmentierung/Policy wird primär auf der Firewall/Edge umgesetzt (Hypervisor bleibt schlank).
- Statische Routen am Hypervisor nur, wenn bewusst erforderlich (und dokumentiert).

---

## 6) Virtualisierung (Standards)

### 6.1 LXC (Default)
- **Default: unprivileged LXC** (Least-Privilege-Ansatz)
- Services intern, keine direkte WAN-Exposition
- Lifecycle: ungenutzte/pausierte Container bewusst gestoppt (Attack Surface reduzieren)

### 6.2 VMs (QEMU/KVM)
- Für Workloads, die von voller VM-Isolation oder speziellen Anforderungen profitieren (Treiber/Kernel/Kompatibilität).

---

## 7) Snapshots (aktueller Stand)

- ZFS Snapshots werden aktuell nicht als Standardmechanismus genutzt (keine Snapshots im Pool sichtbar).
- Snapshot-Nutzung ist als Ausbauoption vorgesehen (z. B. vor Changes oder zeitgesteuert), wird aber bewusst gegen Betriebsaufwand/Komplexität abgewogen.

---

## 8) Backup / Restore / DR (Kurzfassung)

### 8.1 Backup-Domänen
- **PVE → PBS0 (lokal auf dem Hypervisor):** schnelle Restores, kurze Wege
- **PVE → PBS1 (NAS-0 / TrueNAS SCALE):** zweite Storage-Domäne (Resilienz bei Host-Ausfall)

### 8.2 NAS-Replikation + Offsite
- **NAS-0 → NAS-1:** tägliche Replikation (zweite lokale Kopie)
- **NAS-0 → NAS-2 (offsite):** tägliche Sicherung über **WireGuard** (Standort-Trennung)

### 8.3 Restore-Checks
- Backups gelten erst als „gut“, wenn Restore geprüft ist:
  - stichprobenartige Restores in isolierter Umgebung
  - Smoke-Checks (Boot/Login/Kernfunktion/Logs)
  - kurze Dokumentation „was getestet wurde / Ergebnis“

---

## 9) Betrieb / Changes

- Changes in Wartungsfenstern, mit Rollback-Pfad:
  - vor größeren Updates: Backup (und ggf. Snapshot)
  - danach: Smoke-Check + Log-Check
- Störungen: Hypothese → Messpunkt → Fix → Verifikation → kurze Notiz

---

## 10) Absichtlich nicht enthalten (Scope)

- echte IPs/Subnetze, Hostnames/Domains
- MACs, Seriennummern, Fingerprints, Providerdetails
- komplette Exports (`/etc/pve`, Firewall-Exports, Portlisten)
- Secrets/Tokens/Keys