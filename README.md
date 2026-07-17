# Pi-hole + Unbound

Lokaler DNS-Server für das Heimnetz mit:

- **Pi-hole** für DNS-basierte Werbe- und Tracking-Blockierung
- **Unbound** als rekursivem DNS-Resolver
- **Docker Compose** für einfachen Betrieb und Updates

## Architektur

```text
Clients im Heimnetz
        │
        ▼
Pi-hole :53
        │
        ▼
Unbound :5335
        │
        ▼
DNS Root-/TLD-/autoritative Nameserver
```

Pi-hole verwendet ausschließlich den internen Unbound-Container als Upstream. Unbound fragt DNS-Antworten rekursiv ab und validiert DNSSEC.

## Voraussetzungen

- Linux-Server oder kleine VM mit fester LAN-IP
- Docker Engine
- Docker Compose Plugin
- Port `53/tcp` und `53/udp` dürfen auf dem Host nicht bereits belegt sein

Für Proxmox empfiehlt sich eine kleine separate Debian- oder Ubuntu-VM, damit DNS unabhängig von größeren Docker-Stacks wie Plex, Jellyfin oder Immich bleibt.

## Dateien

```text
.
├── compose.yml
├── .env.example
├── .gitignore
└── unbound/
    └── unbound.conf
```

Der persistente Pi-hole-Ordner `pihole/` wird beim ersten Start automatisch angelegt und bewusst nicht versioniert.

## Installation

Repository klonen:

```bash
git clone git@github.com:encor1/pihole-unbound.git
cd pihole-unbound
```

Umgebungsdatei anlegen:

```bash
cp .env.example .env
```

Danach in `.env` ein sicheres Passwort setzen:

```dotenv
PIHOLE_PASSWORD=hier-ein-sicheres-passwort-eintragen
```

Dateiberechtigungen einschränken:

```bash
chmod 600 .env
```

Container starten:

```bash
docker compose up -d
```

Status prüfen:

```bash
docker compose ps
```

Logs anzeigen:

```bash
docker compose logs -f
```

## Weboberfläche

Die Pi-hole-Oberfläche ist standardmäßig erreichbar unter:

```text
http://SERVER-IP:8080/admin/
```

`SERVER-IP` ist die feste LAN-IP der VM beziehungsweise des Docker-Hosts.

## Funktion testen

DNS-Anfrage direkt über Pi-hole:

```bash
dig @SERVER-IP example.com
```

Unbound aus dem Pi-hole-Container testen:

```bash
docker exec pihole dig @172.30.0.2 -p 5335 example.com
```

DNSSEC-Validierung testen:

```bash
dig @SERVER-IP dnssec.works
```

Diese Anfrage sollte erfolgreich sein:

```bash
dig @SERVER-IP dnssec-failed.org
```

Diese Anfrage sollte mit `SERVFAIL` scheitern.

## Router konfigurieren

Trage die feste IP dieses Servers als lokalen DNS-Server im DHCP-Bereich deines Routers ein. Danach erhalten Geräte im Heimnetz Pi-hole automatisch als DNS-Server.

Als zweiten DNS-Server solltest du nicht `1.1.1.1`, `8.8.8.8` oder einen anderen öffentlichen Resolver eintragen. Viele Clients verwenden beide Einträge parallel und könnten Pi-hole dadurch umgehen.

Für echte Ausfallsicherheit sind zwei getrennte Pi-hole-Instanzen sinnvoll, beispielsweise:

```text
192.168.178.2  Pi-hole 1
192.168.178.3  Pi-hole 2
```

## Updates

Images aktualisieren und Container neu erstellen:

```bash
docker compose pull
docker compose up -d
```

Nicht mehr verwendete Images entfernen:

```bash
docker image prune
```

## Backup

Gesichert werden sollten:

- `compose.yml`
- `.env`
- `unbound/unbound.conf`
- der lokale Ordner `pihole/`

Vor einem konsistenten Backup kann der Stack kurz gestoppt werden:

```bash
docker compose down
```

Nach dem Backup:

```bash
docker compose up -d
```

## Hinweise

- Unbound veröffentlicht keinen Port auf dem Host und ist nur im internen Docker-Netz erreichbar.
- Pi-hole veröffentlicht DNS auf Port 53 sowie das Webinterface auf Port 8080.
- DNSSEC wird von Unbound validiert; die zusätzliche DNSSEC-Option in Pi-hole bleibt deshalb deaktiviert.
- Der Compose-Stack verwendet feste interne IP-Adressen, damit Pi-hole seinen Resolver zuverlässig erreicht.
