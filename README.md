# GYMprove Home Assistant Add-on Repository

[GYMprove](https://github.com/schlangeweile/gymprove-addon) — Trainingsplan &
Trainingslogbuch für Bodybuilding und weitere Sportarten. Eigenständiger
Dienst mit eigenem Login: **kein Ingress**, keine HA-Header-Abhängigkeit,
Erreichbarkeit über eine eigene Subdomain am vorhandenen Reverse-Proxy.

Privates Projekt von [schlangeweile](https://github.com/schlangeweile) —
Source-Repo ist geschlossen, die Container-Images liegen öffentlich auf GHCR
(`ghcr.io/schlangeweile/{arch}-addon-gymprove`).

## Installation

1. Einstellungen → Add-ons → Add-on Store → ⋮ → **Repositories**
2. `https://github.com/schlangeweile/gymprove-addon` eintragen und bestätigen
3. **GYMprove** installieren und starten

## Betrieb

- Web-UI auf Port **9250** (Container und Host identisch); im Normalfall
  hinter dem eigenen Reverse-Proxy (z. B. Caddy → `localhost:9250`).
- **Login nur über HTTPS.** Die Session-Cookies sind `Secure`; ein Aufruf
  über `http://<LAN-IP>:9250` kann keinen Login abschließen. Das Add-on hat
  deshalb bewusst keinen `webui`-Button.
- Alle Daten (SQLite im WAL-Modus) liegen auf dem Add-on-Volume `/data`.
- **Backups:** HA-Snapshots sind auf `cold` gestellt (Add-on wird vor dem
  Archivieren gestoppt), weil ein Hot-Tar einer laufenden SQLite-WAL-Datei
  nicht konsistent ist. Zusätzlich ist ein Online-Backup über die CLI möglich:
  `gymprove backup -o /share/gymprove-<Datum>.db` (das Add-on mountet
  `/share`). Restore: Add-on stoppen, `/data/gymprove.db` ersetzen,
  `-wal`/`-shm` entfernen, starten.
- Nutzerkonten und der Bestandsimport laufen über die CLI des Containers:
  `docker exec -it <container> gymprove user create …` bzw.
  `gymprove user passwd …`.
- Das Login-Rate-Limit arbeitet ohne `X-Forwarded-For`-Trust; die
  konto-basierte Sperre bleibt auch hinter dem Proxy wirksam.

## Architekturen

`aarch64` (getestet auf dem Zielsystem) und `amd64`.
