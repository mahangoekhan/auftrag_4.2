
# 🐳 Docker Cheat Sheet

## 🔹 Docker Grundlagen

```bash
docker --version       # Zeigt installierte Docker-Version
docker info            # Systeminformationen anzeigen
docker help            # Hilfe anzeigen
```

---

## 🔹 Images

```bash
docker pull <image>    # Image vom Docker Hub laden
docker images          # Liste lokaler Images
docker rmi <image>     # Image löschen
```

---

## 🔹 Container

```bash
docker run <image>                 # Container starten (Standard)
docker run -it <image> /bin/bash  # Interaktiv starten
docker run -d <image>             # Im Hintergrund starten (detached)
docker run -p 8080:80 <image>     # Portweiterleitung (Host:Container)
docker ps                         # Laufende Container anzeigen
docker ps -a                      # Alle Container anzeigen
docker start <id>                 # Gestoppten Container starten
docker stop <id>                  # Container stoppen
docker restart <id>              # Container neu starten
docker rm <id>                   # Container löschen
```

---

## 🔹 Volumen und Daten

```bash
docker volume create <name>      # Volume erstellen
docker volume ls                 # Liste der Volumes
docker run -v <volume>:/path     # Volume einbinden
docker volume rm <name>          # Volume löschen
```

---

## 🔹 Netzwerke

```bash
docker network ls                # Netzwerke anzeigen
docker network create <name>    # Netzwerk erstellen
docker network rm <name>        # Netzwerk löschen
docker run --network=<name>     # Container mit Netzwerk verbinden
```

---

## 🔹 Dockerfile & Build

```bash
docker build -t <name> .                        # Image aus Dockerfile bauen
docker build -f <file> -t <name> .              # Mit bestimmtem Dockerfile
```

---

## 🔹 Docker Compose

```bash
docker-compose up              # Services starten
docker-compose up -d          # Im Hintergrund starten
docker-compose down           # Alle Services stoppen
docker-compose build          # Services neu bauen
```

---

## 🔹 Aufräumen

```bash
docker system prune           # Alles Ungenutzte löschen
docker image prune            # Ungenutzte Images löschen
docker container prune        # Gestoppte Container löschen
docker volume prune           # Ungenutzte Volumes löschen
```

---

## 📘 Glossar

| Variable         | Bedeutung |
|------------------|-----------|
| `<image>`        | Name oder ID eines Docker-Images, z. B. `nginx`, `ubuntu` |
| `<id>`           | Container-ID oder kurzer Hash eines Containers |
| `<volume>`       | Name eines Docker-Volumes zum Speichern von Daten |
| `<name>`         | Beliebiger Name für ein Image, Volume oder Netzwerk |
| `<file>`         | Pfad zu einer Dockerfile-Datei (z. B. `Dockerfile.dev`) |
| `/path`          | Pfad im Container, z. B. `/usr/share/nginx/html` |
| `8080:80`        | Port-Weiterleitung: Host-Port 8080 leitet auf Container-Port 80 weiter |
