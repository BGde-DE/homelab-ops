# Security Baseline (anonymisiert)

Dieses Dokument beschreibt **Security-Standards/Defaults** für das Homelab als Betriebs-Pattern.
Keine echten IPs, Domains, Hostnames, Userlisten, Keys oder vollständigen Config-Exports.

---

## 0) Ziele / Threat Model (kurz)

**Ziele**
- Reduzierte Angriffsfläche (minimale Exposition, Default deny, saubere Defaults)
- Schnell erkenn- und behebbarer Drift (Changes nachvollziehbar, Checks/Runbooks)
- Schutz vor typischen Homelab-Risiken: Fehlbedienung, Credential-Leaks, ungepatchte Dienste, „versehentlich ins WAN“

**Nicht Ziel / Grenzen**
- Kein „High Security“-Umfeld, sondern praktikables Defense-in-Depth
- Kein vollständiger Ersatz für professionelle SOC/EDR/Forensik

---

## 1) Identitäten & Authentifizierung

### 1.1 Prinzipien
- **Least Privilege**: Admin-Rechte nur dort, wo sie gebraucht werden.
- **Unique Credentials pro Zone/Service** (keine Wiederverwendung).
- Adminzugänge sind **nicht** gleich Nutzerzugänge (getrennte Konten/Rollen).

### 1.2 Hypervisor / Infra-Logins
- Adminzugriffe (WebUI/SSH) **nur aus MGMT/AP_MGMT** (kein Clients/IoT/Gast, kein VPN-Admin).
- Wo möglich: **2FA** für WebUIs (PVE/OPNsense/weitere zentrale Systeme).

### 1.3 SSH Baseline (Linux)
- Default: **Key-based Auth**, keine Secrets im Repo.
- Root-Login:
  - bevorzugt: `PermitRootLogin prohibit-password` oder `no`
  - Admin via normalem User + sudo
- Bruteforce-Schutz:
  - Rate-Limits / Fail2ban / SSH-Guard (je nach Host-Rolle)

*(Wenn ein System aus pragmatischen Gründen anders konfiguriert ist, wird das als Ausnahme dokumentiert.)*

---

## 2) Netzwerk-Security (Defense in Depth)

### 2.1 Segmentierung
- VLAN/Zonen nach Schutzbedarf (GUEST/IOT/CLIENTS/SERVER/MGMT/AP_MGMT/AUTOMATION/VPN).
- **Default deny** zwischen Segmenten.
- Ausnahmen sind ziel- und portbezogen (nur „was nötig ist“).

Siehe: `docs/netzwerk_security.md` und `docs/firewall.md`.

### 2.2 Public Exposure (WAN)
- **WAN → intern** nur über definierte Entry-Points (Reverse Proxy für Web).
- Backends sind intern; WAN spricht **nie direkt** mit Backend-Netzen.
- Für Non-HTTP Dienste: nur gezielte, begründete Ausnahmen.

Siehe: `docs/services_reverse_proxy.md` und `templates/checkliste_release_public_service.md`.

### 2.3 VPN (Remote Access)
- VPN ist für **Nutzer-/Servicezugriff** gedacht, nicht als Admin-Jump-Host.
- Adminzugänge bleiben **lokal im MGMT/AP_MGMT** (bewusste Trennung).

---

## 3) Host Hardening (Linux / VM / LXC)

### 3.1 Defaults
- Nur notwendige Services laufen (alles andere disabled).
- Firewall/Policy möglichst zentral (Edge), Host-Firewalls optional je nach Rolle.
- Zeitsync (NTP) aktiv; konsistente Logs/Zeiten für Troubleshooting.

### 3.2 Container/VM Isolation
- Default: **unprivileged LXC** (wenn LXC genutzt wird).
- Services ohne WAN-Need: nicht am WAN, sondern intern.
- „Management Interfaces“ (Admin-UIs) nur im Admin-Segment.

### 3.3 Updates / Patch-Disziplin
- Security-Updates regelmäßig (Wartungsfenster).
- Nach Kernel-Updates: Reboot + Health-Check.
- Änderungen werden nachvollziehbar durchgeführt (kurze Notiz + Rollback-Pfad).

Runbook: `docs/runbooks/pve_update.md`

---

## 4) Secrets & Konfiguration

### 4.1 Repo-Regeln (OPSEC)
- Keine Keys/Tokens/PSKs.
- Keine vollständigen Exports (Firewall, `/etc/pve`, VPN configs).
- Keine echten Domains/öffentliche IPs/Providerdetails.
- Beispielwerte nur als `.example` / Dummy.

Siehe: `SECURITY.md`

### 4.2 Compose / Service-Secrets
- Secrets nicht in `docker-compose.yml` committen.
- Nutzung von `.env` / Secret-Store (je nach Setup).
- Backups enthalten keine Klartext-Secrets, wenn vermeidbar.

---

## 5) Reverse Proxy & TLS

### 5.1 Prinzipien
- Zentraler Reverse Proxy ist die „Frontdoor“ für Webservices.
- TLS-Termination am Entry Point; Backends intern.
- Logs am Proxy sind erster Troubleshooting-Einstieg.

### 5.2 Zertifikate
- ACME/Let’s Encrypt (oder interner CA-Ansatz) mit Monitoring auf Ablauf.
- Zertifikatserneuerung/Änderungen: kurz testen (HTTP 200, Redirects, HSTS falls genutzt).

---

## 6) Logging / Monitoring / Alerts (minimal & wirksam)

### 6.1 Logging
- Firewall: WAN Blocks/IDS Events werden geloggt und stichprobenartig geprüft.
- Reverse Proxy: Access-/Error-Logs aktiv.
- Systeme: `journalctl` als Baseline; Errors werden bei Incidents geprüft.

### 6.2 Monitoring
- Baseline-Metriken: CPU/RAM/Disk/SMART/ZFS Health.
- Backup-Status: PBS Jobs + Verify-Status werden reviewed.

---

## 7) Backup/Restore als Security-Kontrolle

- Backups sind Teil des Security-Konzepts (Ransomware/Fehlbedienung/Recovery).
- Multi-Stufe + Offsite vorhanden.
- Restore-Tests sind Pflicht (Stichprobe).

Siehe: `docs/backup_restore_dr.md`  
Runbook (Restore-Tests): `docs/runbooks/restore_test_pbs.md`

---

## 8) Exceptions / Drift Handling

- Abweichungen von Defaults sind erlaubt, aber:
  - begründet (warum nötig),
  - dokumentiert (wo gilt die Ausnahme),
  - mit Review („kann das wieder weg?“)

---

## 9) Minimal-Checklist (Copy/Paste)

- [ ] Adminzugänge nur aus MGMT/AP_MGMT
- [ ] Default deny zwischen Segmenten, Ausnahmen eng
- [ ] Public Services nur via Reverse Proxy / definierte Entry Points
- [ ] Regelmäßige Updates + Reboot bei Kernel-Updates
- [ ] PBS Backups + Verify reviewed
- [ ] Restore-Test stichprobenartig durchgeführt und kurz dokumentiert
- [ ] Keine Secrets/Exports/Identifikatoren im Repo