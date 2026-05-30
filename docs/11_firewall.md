# Firewall-Regeln (Übersicht, anonymisiert)

Ziel dieser Seite ist eine **lesbare Policy-Übersicht** (kein Config-Export).
Alle Namen/Netze/Hosts sind Platzhalter. Keine realen Ports, Domains oder Providerdetails.

## 0) Grundprinzipien
- **Default deny** zwischen Segmenten (VLANs/Zonen).
- **Least privilege**: Freigaben sind ziel- und portbezogen (nur was nötig ist).
- **Administration nur aus MGMT/AP_MGMT** (nicht aus Clients/IoT/Gast, nicht via VPN).
- **WAN → interne Systeme** nur über definierte Entry Points (Reverse Proxy für Web; einzelne Ausnahmen wie Voice/Game/Remote nur gezielt).

---

## 1) Interface-Gruppen / Zonen
- `WAN`: Internet-Uplink(s)
- `SERVER`: Server/Services (VM/LXC/Container)
- `AUTOMATION`: Home Assistant/KNX/Automation
- `CLIENTS`: Endgeräte
- `IOT`: IoT (untrusted)
- `GUEST`: Gäste (untrusted)
- `MGMT`: Administration
- `AP_MGMT`: Management der Access Points
- `VPN`: WireGuard (Remote Access zu ausgewählten Services)

---

## 2) Regelreihenfolge (High Level)
Die Regeln sind “top-down” aufgebaut (first match, quick):

1. **Anti-Lockout / Notfallzugang** (falls vorhanden, sehr restriktiv)
2. **WAN: Block Known-Bad** (Threat-Intel / Reputation / dynamische Decisions)
3. **WAN: Allow Trusted** (enge Allowlists, wenn sinnvoll)
4. **WAN: Allow Public Services** (definierte veröffentlichte Dienste)
5. **Interne Segmente: Segment-Policy** (Default deny + gezielte Ausnahmen)
6. **Egress/Outbound** (Updates/NTP/DNS je Segment nach Bedarf)

---

## 3) WAN Inbound (Perimeter)

### 3.1 Notfall / Anti-Lockout (optional)
- **Allow**: sehr restriktiver Admin-/Notfallzugang zur Firewall (nur von *Trusted Admin Sources*).
- **Hinweis:** Ausgeschaltet!

### 3.2 Block: Threat-Intel / Known Bad (immer früh)
- **Block** inbound, Quelle ∈ `TI_KnownBad_*` (Aggregationen, Botnet/C2, Reputation)
- **Block** inbound, Quelle ∈ `TI_DynamicDecisions_*` (z. B. CrowdSec)
- **Block** inbound, Quelle ∈ `Manual_Blocklist`

Zweck:
- reduziert Massenscans/Noise
- verhindert Zugriff bekannter bösartiger Quellen auf Public Services

### 3.3 Allow: Trusted Sources (gezielt, nicht überall)
- **Allow** inbound, Quelle ∈ `Geo_Trusted` oder `Trusted_Admin_Sources`
- Nutzung nur für Services, die *bewusst* eingeschränkt sein sollen (z. B. Remote-Support, Admin-nahe Dienste)

Hinweis:
- GeoIP/Allowlisting ist nie perfekt (VPN/Cloud/Carrier NAT).
- False Positives werden über **enge** Allowlists gelöst (nicht durch Abschalten der Listen).

### 3.4 Allow: Public Services (definiert)
Es existieren bewusst öffentlich erreichbare Dienste. Veröffentlichung erfolgt nach Kategorie:

#### A) Webservices (HTTP/S)
- **Allow**: `WAN → HAProxy` (Entry Point für Web)
- **Deny**: `WAN → BackendNetze` direkt (keine direkten Backends am WAN)

#### B) Non-HTTP Dienste (gezielte Ausnahmen)
- **Allow**: einzelne Service-Kategorien (z. B. Voice / Game / Remote-Support)
  - typischerweise zusätzlich eingeschränkt über `Geo_Trusted` oder “min trusted”
- **Beispiel (anonym):**
  - “einige UDP-Ports für Gameserver (privat/familiär)”
  - “ein Voice-Service”
  - “ein Remote-Support Dienst”

**Wichtig:** In dieser Doku werden keine Portlisten veröffentlicht.

### 3.5 Default deny
- Alles andere inbound wird geblockt.

---

## 4) Interne Segment-Regeln (LAN/VLAN)

### 4.1 GUEST
- **Allow**: `GUEST → Internet` (DNS/NTP/HTTP(S))
- **Deny**: `GUEST → interne Netze` (alle)

### 4.2 IOT
- **Allow**: `IOT → DNS/NTP` (definierte Resolver/Time)
- **Allow (optional)**: `IOT → AUTOMATION` (nur benötigte Services, wenn Integrationen es erfordern)
- **Allow (optional)**: `IOT → Internet` eingeschränkt (Updates)
- **Deny**: `IOT → MGMT` und `IOT → SERVER` (default)

### 4.3 CLIENTS
- **Allow**: `CLIENTS → Internet`
- **Allow (optional)**: `CLIENTS → ausgewählte SERVER Dienste` (Nutzer-Services)
- **Deny**: `CLIENTS → MGMT` (Adminzugänge nur aus MGMT)

### 4.4 AUTOMATION
- **Allow**: `AUTOMATION → DNS/NTP`
- **Allow (optional)**: `AUTOMATION → Internet` eingeschränkt (Integrationen/Updates)
- **Deny**: `AUTOMATION → MGMT` (MGMT bleibt separiert)

### 4.5 SERVER
- **Allow**: `SERVER → Internet` nach Bedarf (Updates/Abhängigkeiten)
- **Allow**: `HAProxy → Backends` nur auf benötigte Ziele/Ports
- **Deny**: unnötige Ost-West-Kommunikation (segmentiert nach Servicebedarf)

### 4.6 MGMT und AP_MGMT
- **Allow**: `MGMT → Infrastruktur` (Firewall, Hypervisor, Storage, APs, Server-Management)
- **Allow**: `AP_MGMT → notwendige Targets` (z. B. Updates/Logging/Management-Services)
- **Deny**: Zugriff auf MGMT aus anderen Segmenten (one-way Administration)

---

## 5) VPN (WireGuard)
- **Allow**: `VPN → ausgewählte interne Services` (Nutzer-/Servicezugriff)
- **Deny**: `VPN → MGMT/AP_MGMT` (bewusste Trennung: Administration nur lokal)

---

## 6) Logging / Betrieb
- Security-relevante Blocks (WAN Threat-Intel / IDS Events) werden geloggt und stichprobenartig geprüft.
- Bei Vorfällen:
  - temporäres Rate-Limit / Block / enges Allowlisting
  - Review von Proxy-/App-Logs
  - Lessons Learned + Rule-Tuning

---

## 7) Änderungen / Review
- Regeländerungen erfolgen nachvollziehbar (kurze Notiz: Datum, Grund, Risiko, Rollback).
- Regelmäßiger Review: “Welche Ausnahme kann wieder entfernt werden?”
