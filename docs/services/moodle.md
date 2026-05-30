# Moodle – Betrieb (anonymisiert)

## Kontext
Ich betreibe Moodle selbst als Service in einer Proxmox-Umgebung.

## Architektur (High Level)
- **Hypervisor:** Proxmox VE
- **Workload:** Debian LXC (unprivileged), `nesting=1`
- **Storage:**
  - Rootfs auf lokalem Storage (Performance/Resilience)
  - Separates Mount für **moodledata** / Uploads auf externem Storage (NAS/Storage)
- **Netz:** eigenes internes Segment (Firewall aktiv), Zugriff nur aus definierten Netzen/VPN

## Betrieb / Wartung
- Regelmäßige Updates (OS + Moodle)
- Wartungsfenster + kurzer Rollback-Plan (Snapshot/Backup vor Upgrade)
- Monitoring über Basis-Metriken (CPU/RAM/Disk), Logprüfung (Web/PHP)

## Backup / Restore (Prinzip)
- Backup umfasst:
  - Moodle Code / Webroot (falls nicht per Paket/Repo reproduzierbar)
  - **DB-Dump** (Moodle DB)
  - **moodledata** (separates Storage)
- Restore-Tests in isolierter Umgebung: “DB + moodledata konsistent” ist der Kerncheck

## Security / Hardening (Prinzip)
- Segmentierung: Service ist nicht “frei im LAN”
- Admin-Zugriff getrennt (MGMT-Konzept)
- Keine Secrets im Repo; Konfiguration/Passwörter nur lokal/Secret-Store

## Beispiel: Proxmox LXC Parameter (Beispiel)

```ini
arch: amd64
features: nesting=1 #optional (z.B. falls im CT weitere Container/Namespace-Features benötigt werden)
hostname: moodle
memory: 8192
onboot: 1
ostype: debian
swap: 0
unprivileged: 1

rootfs: local-zfs:subvol-XXX-disk-1,size=XXXG
startup: order=10,up=90 #Startreihenfolge/Delay (z.B. damit abhängige Storage-Mounts typischerweise vorher bereit sind)

mp0: /path/to/storage/moodledata,mp=/srv/moodledata,size=0T

net0: name=eth0,bridge=vmbrX,firewall=1,gw=10.50.50.1,hwaddr=XX:XX:XX:XX:XX:XX,ip=10.50.50.10/24,type=veth
