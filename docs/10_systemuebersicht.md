# Setup (anonymisierte Daten)

Dieses Kapitel enthält konkrete technische Daten meines Setups – anonymisiert:
- keine realen Domains/Subdomains
- keine Provider-/Standortdaten
- keine Portlisten oder vollständigen Firewall-Exports
- keine Secrets / Keys / Tokens

## 1) Netzwerksegmentierung (VLANs + DHCP)

| VLAN | Name        | Subnetz (sanitisiert) | DHCP-Range | Zweck |
|-----:|-------------|------------------------|------------|------|
| 10   | AUTOMATION  | 10.50.10.0/24          | .11–.200   | Home Assistant, KNX, automations-nahe Systeme |
| 20   | IOT         | 10.50.20.0/24          | .11–.200   | IoT-Geräte (untrusted) |
| 30   | GUEST       | 10.50.30.0/24          | .11–.200   | Gäste (nur Internet) |
| 40   | CLIENTS     | 10.50.40.0/24          | .11–.200   | Endgeräte (PC/Laptop/Smartphones) |
| 50   | MGMT        | 10.50.50.0/24          | .11–.200   | Administration (WebUIs/SSH/Hypervisor/Storage) |
| 60   | AP_MGMT     | 10.50.60.0/24          | .11–.200   | Management der Access Points (OpenWRT) |
| 70   | SERVER      | 10.50.70.0/24          | .11–.200   | Server/Services (VM/LXC/Container) |

Gateway/Firewall: OPNsense (Inter-VLAN Routing + Policies)

### Policy (Kurzfassung)
- Default: VLANs voneinander getrennt (deny).
- GUEST: nur Internet, kein Zugriff auf interne Netze.
- IOT: nur definierte Ausnahmen (z. B. DNS/NTP, optional ausgewählte Automation-Services).
- MGMT/AP_MGMT: Adminzugänge ausschließlich aus diesen Segmenten; andere VLANs dürfen nicht in MGMT hinein.

## 2) DNS / Resolver
- DNS wird zentral über **Unbound** bereitgestellt.
- DNS-Filter/Blocklisten können pro Segment unterschiedlich sein.
  - Beispiel: GUEST weniger aggressiv gefiltert (Kompatibilität), ohne Aufweichung der Inter-VLAN-Regeln.

## 3) VPN / Remote Access
- WireGuard wird für Remote Access zu ausgewählten internen Services genutzt.
- MGMT/AP_MGMT ist **nicht** über VPN erreichbar (Administration nur lokal/physisch im Admin-Netz).

## 4) Reverse Proxy / Public Services (ohne echte vHosts)
- Veröffentlichung von Webservices zentral über **HAProxy** auf OPNsense:
  - TLS-Termination und vHost/SNI-Routing am Entry Point
  - Backends bleiben intern (kein direkter WAN→Backend Zugriff)

### Auth-Modell (Praxis)
- Wenn ein Dienst eine sinnvolle Auth mitbringt: Nutzung der **App-eigenen Auth**.
- Wenn ein Dienst keine geeignete Auth besitzt: **vorgeschaltete Auth** (separater Container) nur für diese Fälle.

### Interne Proxy→Backend Verbindung (Tradeoff)
- Proxy→Backend läuft im internen Netz ohne zusätzliche TLS-Verschlüsselung.
- Begründung: höhere Sichtbarkeit für IDS/IPS (Traffic-Inspection).
- Absicherung: Segmentierung, restriktive Proxy→Backend Regeln, Logging, Patch-Disziplin.

## 5) IDS/IPS + Threat-Intel Blocklists (Edge Hardening)
- OPNsense mit aktivem IDS/IPS (regelbasiert, “Oink rules”/ähnliche Feeds).
- Ergänzend: Threat-Intel/Blocklists (Firewall-Aliases), z. B. Kategorien:
  - Botnet/C2 Indicators
  - Abuse-/Scanner-Aggregationen
  - Reputation-Listen (FireHOL o. ä.)
  - dynamische Decisions (z. B. CrowdSec)
- Betriebsprinzip:
  - zuerst “Known Bad” blocken, dann gezielte Allowlisting-Policies, danach definierte Public Entry Points
  - False Positives werden mit enger Allowlist behandelt (nicht “Listen abschalten”)

## 6) Backup / Restore / DR (mehrstufig, inkl. Offsite)
Backup-Flow (abstrahiert):
- Proxmox VE → Proxmox Backup Server (lokal, schnelle Restores)
- zusätzliche Backup-Stufe → NAS (lokal)
- NAS → NAS (zweite lokale Stufe)
- Offsite-Kopie via VPN (WireGuard) zu externem NAS/Storage

Restore-Checks:
- regelmäßige Restore-Tests in isoliertem Testnetz (klein monatlich, größer quartalsweise)
- Dauer/Probleme werden kurz dokumentiert
