# Wartungsseite (Docker + Nginx)

## Zweck

Diese Docker-basierte Wartungsseite wird genutzt, um produktive Systeme (z. B. Moodle) während Wartungsfenstern zu ersetzen.

Sie zeigt Nutzern eine klare Wartungsinformation und verhindert Zugriff auf das Produktivsystem.


## VERHALTEN DER SEITE
- HTTP Status: 503 Service Temporarily Unavailable
- Retry-After Header: 3600 Sekunden
- Automatische Weiterleitung nach 120 Sekunden
- Manuelle Rückkehr zur Hauptdomain möglich

## HTTP HEADER (PHP)
- 503 Status signalisiert temporäre Nichtverfügbarkeit
- Retry-After informiert Clients über Wiederverfügbarkeit

## HTML VERHALTEN
- Zentrierte Wartungsseite
- Hinweis auf geplante Wartung
- E-Mail Kontakt für Admin erreichbar
- Optionaler Redirect zur Hauptseite

## AUTOMATISCHE WEITERLEITUNG
<meta http-equiv="refresh" content="120; URL=https://domain.de">

- Nach 120 Sekunden Redirect zur Hauptseite
- Nutzer können Seite offen lassen
## DOCKER SETUP
Service basiert auf Nginx Container:

	version: '3.9'
	
	volumes:
	  html:
	  config:
	
	services:
	  app:
	    image: nginx
	    container_name: nginx_wartungsseite
	
	    volumes:
	      - html:/usr/share/nginx/html:ro
	      - config:/etc/nginx
	    restart: unless-stopped
	
	    environment:
	      - VIRTUAL_HOST=wartung.domain.de
	      - VIRTUAL_PORT=8085
	      - LETSENCRYPT_HOST=wartung.domain.de
	      - LETSENCRYPT_EMAIL=admin@domain.de
	
	networks:
	  default:
	    external: true
	    name: nginx-proxy

## NETZWERK
- Reverse Proxy: nginx-proxy Netzwerk
- TLS: Let's Encrypt automatisch
- Routing über VIRTUAL_HOST

## VERWENDUNG
- Wird während Wartung aktiviert
- Ersetzt temporär produktive Anwendungen
- Keine Abhängigkeit vom Backend-System