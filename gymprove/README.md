# GYMprove

Trainingsplan & Trainingslogbuch — eigenständiger Dienst mit eigenem Login,
ohne Ingress. Der Zugriff erfolgt über die HTTPS-Subdomain am vorhandenen
Reverse-Proxy; das Add-on veröffentlicht Port **9250**, bietet aber bewusst
keinen HTTP-WebUI-Button an (die Cookies sind `Secure`).

## Erste Schritte

1. Add-on starten.
2. Über die HTTPS-Subdomain anmelden. Bei einer frischen Datenbank existiert
   noch kein Konto:
   ```bash
   docker exec -it <container> gymprove user create \
     -email name@example.com -name Name
   ```
   Für den Parallelbetrieb stattdessen die importierte Bestands-DB nach
   `/data/gymprove.db` legen (Add-on vorher stoppen, `-wal`/`-shm` entfernen)
   und mit dem Bestands-Passwort anmelden.
3. Passwort vergessen/ändern per CLI:
   ```bash
   docker exec -it <container> gymprove user passwd -email name@example.com
   ```

## Daten & Backups

- SQLite-Datenbank: `/data/gymprove.db` (WAL-Modus).
- HA-Snapshots sind `cold`; zusätzlich ist ein konsistentes Online-Backup
  möglich:
  ```bash
  docker exec -it <container> \
    gymprove backup -o /share/gymprove-$(date +%F-%H%M).db
  ```
- Restore: Add-on stoppen, `/data/gymprove.db` durch das Backup ersetzen,
  `gymprove.db-wal`/`-shm` entfernen, starten.

## Sicherheit

- Login nur über HTTPS; HSTS, CSP, `nosniff`, `frame-ancestors 'none'` und
  `noindex` werden von der App gesetzt.
- Kein `X-Forwarded-For`-Trust: Das Rate-Limit arbeitet mit der
  Proxy-Peer-Adresse und dem Konto.
- `backup: cold`, Request-Body-Limit 1 MiB, Read-/Write-/Idle-Timeouts.
