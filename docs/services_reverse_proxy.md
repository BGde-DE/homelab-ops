# Reverse Proxy & Public Services (Pattern)

## Grundidee
- Webservices werden zentral über einen Reverse Proxy (HAProxy) veröffentlicht.
- TLS-Termination + Routing (vHost/SNI) an einem Punkt.
- Backends sind intern; WAN spricht nie direkt mit den Backends.

## Authentifizierung (Praxis)
- Dienste mit eigener Auth nutzen ihre eingebaute Benutzerverwaltung.
- Wenn ein Dienst keine sinnvolle Auth mitbringt, wird eine vorgeschaltete Auth-Schicht genutzt
  (separater Container/Service), nur für diese Fälle.

## Logging & Betrieb
- Reverse Proxy Logs sind der erste Einstieg für Troubleshooting.
- Änderungen erfolgen nach Checkliste (DNS, Zertifikat, Backend Health, Firewall).

## Public Exposure (bewusst)
- Es existieren bewusst öffentlich erreichbare Services.
- Zusätzlich werden Schutzschichten genutzt:
  - Threat-Intel Blocklists / Reputation-Listen
  - IDS/IPS
  - Allowlisting (z. B. “trusted only”) dort, wo sinnvoll
  - Rate-Limit/Block im Incident-Fall (Runbook)
 
## TLS-Termination & interne Verbindung (bewusste Entscheidung)
- TLS wird am Reverse Proxy terminiert (zentraler Entry Point).
- Die Verbindung **Reverse Proxy → Backend** läuft im internen Netz **ohne zusätzliche TLS-Verschlüsselung**.

Begründung / Tradeoff:
- Der interne Traffic kann dadurch durch das IDS/IPS besser inspiziert werden (Visibility/Detection).
- Die Maßnahme wird durch weitere Controls abgesichert:
  - Backend-Systeme sind nicht direkt aus dem WAN erreichbar (nur über den Proxy)
  - Segmentierung (SERVER vs. CLIENTS/IOT/GUEST vs. MGMT)
  - restriktive Firewall-Regeln (Proxy darf nur notwendige Ziele/Ports)
  - regelmäßige Updates, Logging und Incident-Runbooks

Hinweis:
- Für besonders sensitive Backends kann optional trotzdem End-to-End-TLS eingesetzt werden.
