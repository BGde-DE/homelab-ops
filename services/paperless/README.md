# Paperless-ngx (intern / privat)

Dieses Compose-Setup ist **für internen Gebrauch** gedacht und dient hier als **Template** (anonymisiert).

## Erreichbarkeit (wichtig)
- Die Domain `paperless.example.internal` ist als Beispiel gedacht und sollte bei dir ein **lokaler DNS-Override** sein (nur intern auflösbar).
- Der Dienst ist **nicht** über einen öffentlichen Reverse Proxy veröffentlicht und **nicht** für WAN-Exposure vorgesehen.
- SMB (Scanner-Ingest) ist **LAN-only** und darf **niemals** aus dem Internet erreichbar sein (keine Portweiterleitungen, keine WAN-Regeln).

## Empfehlung: SMB nur intern binden
Wenn du SMB überhaupt nutzt, binde die Ports **nicht** auf `0.0.0.0`, sondern auf eine interne Server-IP (Beispiel):

- `"10.50.70.10:445:445/tcp"`
- `"10.50.70.10:137:137/udp"` usw.

(Die IP ist nur ein Platzhalter. Nimm eine Adresse aus deinem internen Server-Netz.)

## Benutzer anlegen
- In den Paperless-Container wechseln
- `python3 manage.py createsuperuser`

## Secrets
- Nutze `.env` 
