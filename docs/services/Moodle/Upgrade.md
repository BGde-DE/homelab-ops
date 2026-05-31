# Moodle Upgrade (LXC + Docker Wartungsfrontend + HAProxy)

Diese Anleitung beschreibt das Upgrade einer Moodle-Installation im LXC-Container,
während die Wartungsseite über Docker bereitgestellt wird und HAProxy den Traffic umleitet.

## ARCHITEKTUR

- Moodle läuft in einem LXC-Container
- Apache + PHP laufen im LXC
- Wartungsseite läuft in Docker (Portainer Stack)
- HAProxy leitet um:
  - Produktivsystem (LXC)
  - Wartungs-Docker (Frontend)


## 1. WARTUNGSSYSTEM (DOCKER / HAPROXY)


1. Portainer:
   - Stack "wartung-moodle" deployen/aktualisieren

2. OPNsense HAProxy:
   - Disable:  Moodle_send-proxy
   - Enable:   Moodle_WARTUNG_Docker


## 1.1. Backup läuft – Backup prüfen!



## 2. APACHE IM LXC STOPPEN


/etc/init.d/apache2 stop

Hinweis:
- Traffic ist bereits umgeleitet
- Keine aktiven User Requests mehr
- Sicherer Zustand für Dateisystemoperationen


## 3. TEMPORÄRES BACKUP DER INSTALLATION


cd /var/www/

rm -rf moodle.bak
mv moodle moodle.bak


## 4. NEUE MOODLE VERSION


wget https://download.moodle.org/download.php/direct/stable310/moodle-latest-310.tgz

tar -xvzf moodle-latest-310.tgz
rm moodle-latest-310.tgz


## 5. KONFIGURATION ÜBERNEHMEN


cp moodle.bak/public/config.php moodle/


## 6. PLUGINS (MOODLE 5 MIT NEUER PUBLIC STRUCTURE)


dirs=("mod" "blocks" "report" "lib" "local")

for dir in "${dirs[@]}"; do
    cp -pr moodle.bak/public/$dir/* moodle/public/$dir/
done

Optional gezielt durch neue Version ersetzen:

rm -rf moodle/public/mod/publication

unzip plugins/local_adminer_moodle51_2026021001.zip \
  -d moodle/public/local/


## 7. RECHTE


chown -R root:root moodle
find moodle -type d -exec chmod 755 {} \;
find moodle -type f -exec chmod 644 {} \;

chmod 755 moodle/admin/cli/cron.php


## 8. UPGRADE


sudo -u www-data php moodle/admin/cli/upgrade.php


## 9. APACHE START


/etc/init.d/apache2 start


## 10. WARTUNG BEENDEN


- HAProxy zurückschalten:
  Moodle_WARTUNG_Docker -> disable
  Moodle_send-proxy -> enable

- Test: Läuft alles mit Wartungsmodus?

- Portainer Wartungsstack stoppen

sudo -u www-data php /var/www/html/admin/cli/maintenance.php --disable


## 11. CHECKS


- Login testen
- Kurse öffnen
- Cron prüfen
- Logs prüfen:
  /var/log/apache2/
  Moodle logs
- Plugin-Kompatibilität prüfen


## NOTIZEN


- Moodle Wartungsmodus ist unabhängig vom HAProxy Setup
- Docker Wartung = nur UI / Traffic Layer
- LXC ist die einzige relevante Runtime für das Upgrade
- Upgrade niemals ohne DB + Filesystem Backup
