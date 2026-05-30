# Paperless-ngx (lokal / privat)

Dieses Compose-Setup ist **für internen Gebrauch** gedacht.

## Erreichbarkeit (wichtig)
- Die Domain `paperless.example.internal` sollte ein **lokaler DNS Override** (nur intern auflösbar) sein.
- Der Dienst ist **nicht** über den öffentlichen Reverse Proxy veröffentlicht.
- SMB (Scanner-Ingest) ist **LAN-only** und darf **niemals** aus dem Internet erreichbar sein.

## Empfehlung: SMB-Ports auf LAN-IP binden
Statt `0.0.0.0:445` besser an eine interne IP binden, z. B.:

- `"10.50.70.10:445:445/tcp"`
- `"10.50.70.10:137:137/udp"` usw.

(Adresse ist ein Beispiel; benutze eine IP aus deinem internen Server-Netz.)

## Benutzer anlegen (Beispiel)
- Container-Konsole öffnen (paperless)
- `python3 manage.py createsuperuser`

## Secrets
- Keine Secrets ins Repo committen.
- Nutze `.env` (nicht committen) und committe nur `.env.example`.
