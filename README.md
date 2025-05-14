# Flask + Redis mit Docker Compose

Dieses Projekt zeigt, wie man eine einfache Flask-Anwendung mit einem Redis-Backend mithilfe von Docker Compose erstellt und betreibt.

---

## 📋 Voraussetzungen

- **Docker Desktop** ist installiert (Docker Compose ist bereits enthalten!)
- Grundverständnis von Docker und Containern

---

## ❓ Muss man Docker Compose separat installieren?

**Nein!**

Wenn du Docker Desktop installiert hast, ist **Docker Compose v2 bereits integriert**.

```bash
docker compose version
```

Beispielausgabe:
```
Docker Compose version v2.27.0
```

Seit Docker Compose v2 nutzt man `docker compose` (mit Leerzeichen) statt `docker-compose` (mit Bindestrich).

---

## 🛠 Projektaufbau

### 1. Projektverzeichnis erstellen

```bash
mkdir composetest
cd composetest
```

### 2. `app.py` erstellen

```python
import time
import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return f'Hello World! I have been seen {count} times.\n'
```

### 3. `requirements.txt`

```txt
flask
redis
```

### 4. `Dockerfile`

```Dockerfile
# syntax=docker/dockerfile:1
FROM python:3.10-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run", "--debug"]
```

> ⚠️ Achte darauf, dass die Datei **keine `.txt`-Endung** hat.

---

## 🧩 5. `compose.yaml` erstellen

```yaml
services:
  web:
    build: .
    ports:
      - "8000:5000"
  redis:
    image: "redis:alpine"
```

Dieses Compose-Setup erstellt:
- **web**: wird aus `Dockerfile` gebaut, leitet Port 5000 auf Host-Port 8000 weiter
- **redis**: verwendet das Redis-Image von Docker Hub

---

## ▶️ Anwendung starten

```bash
docker compose up
```

Du solltest in der Ausgabe u. a. Folgendes sehen:

```
web_1    | * Running on http://0.0.0.0:5000/
redis_1  | * Ready to accept connections
```

Jetzt im Browser aufrufen:
- [http://localhost:8000](http://localhost:8000)

Nach jedem Neuladen sollte sich der Zähler erhöhen:
```
Hello World! I have been seen 2 times.
```

---

## 🔄 Optional: Live Code Reload mit Compose Watch

### Compose-Datei anpassen:

```yaml
services:
  web:
    build: .
    ports:
      - "8000:5000"
    develop:
      watch:
        - action: sync
          path: .
          target: /code
  redis:
    image: "redis:alpine"
```

Starte mit Watch-Modus:
```bash
docker compose up --watch
```

Änderst du z. B. in `app.py`:
```python
return f'Hello from Docker! I have been seen {count} times.\n'
```

→ Speichern → Browser neu laden → Änderung sichtbar.

---

## 🧱 Services aufteilen mit mehreren Compose-Dateien

`infra.yaml` (neue Datei für Redis):

```yaml
services:
  redis:
    image: "redis:alpine"
```

`compose.yaml` anpassen:

```yaml
include:
  - infra.yaml

services:
  web:
    build: .
    ports:
      - "8000:5000"
    develop:
      watch:
        - action: sync
          path: .
          target: /code
```

Jetzt wieder starten:
```bash
docker compose up
```

---

## ⚙️ Nützliche Befehle

| Befehl                             | Beschreibung                          |
|------------------------------------|---------------------------------------|
| `docker compose up`                | Startet alle Services                 |
| `docker compose up -d`             | Startet im Hintergrund (detached)    |
| `docker compose ps`                | Zeigt laufende Services               |
| `docker compose stop`              | Stoppt, aber behält Container         |
| `docker compose down`              | Stoppt und entfernt Container/Netz    |
| `docker image ls`                  | Zeigt lokale Images                   |

---

## 📦 Beispielhafte Ausgabe für `docker image ls`

```bash
REPOSITORY        TAG           IMAGE ID      CREATED        SIZE
composetest_web   latest        e2c21aa48cc1  4 minutes ago  93.8MB
python            3.10-alpine   84e6077c7ab6  7 days ago     82.5MB
redis             alpine        9d8fa9aa0e5b  3 weeks ago    27.5MB
```

---

## ✅ Fazit

- Du brauchst **kein Docker Compose separat installieren**, wenn du **Docker Desktop** nutzt.
- Mit `docker compose` kannst du **komplexe Anwendungen einfach starten und verwalten**.
- Funktionen wie **Compose Watch** und **Multi-File-Unterstützung** machen das Arbeiten effizienter.

---

📊 Diagramm der Containerstruktur
pgsql
Kopieren
Bearbeiten
               +-------------------+            +--------------------+
               |     Web-App       |            |       Redis        |
               |-------------------|            |--------------------|
               | Container: web    | <--------> | Container: redis   |
               | Image: composetest_web         | Image: redis:alpine|
               | Port: 5000 (intern)            | Port: 6379 (intern)|
               | ↧ Port Mapping ↧               |                    |
               | Host: localhost:8000           | Netzwerk: default  |
               +-------------------+            +--------------------+
Alle Container befinden sich im gleichen virtuellen Docker-Netzwerk (composetest_default), das automatisch von Docker Compose erstellt wird.

🧩 Beschreibung der verwendeten Container
1. Web-Container (Flask-App)
Funktion: Stellt eine kleine Webanwendung bereit.

Image: Wird aus dem Dockerfile im Projektordner gebaut.

Technologien: Python, Flask, Redis-Client

Ports:

Intern (im Container): 5000

Extern (am Host): 8000 (weitergeleitet mit ports: "8000:5000")

Kommunikation: Verbindet sich mit dem Redis-Container über den Hostnamen redis.

2. Redis-Container
Funktion: Key-Value-Datenbank zur Zählung der Seitenaufrufe.

Image: Offizielles Image redis:alpine von Docker Hub.

Port: 6379 (Standardport für Redis)

Besonderheit: Läuft „standalone“ ohne eigene Persistenz (Speicherung bei Neustart verloren, außer Volumes werden verwendet).

❓ Was ist Redis?
Redis steht für Remote Dictionary Server und ist eine In-Memory-Datenbank, die Key-Value-Paare speichert.

Eigenschaften:
Extrem schnell (Daten im RAM)

Unterstützt Datentypen wie Strings, Listen, Hashes, Sets

Wird oft verwendet für:

Caching

Session-Speicherung

Zähler (wie in deiner App)

Beispiel aus deiner Anwendung:
python
Kopieren
Bearbeiten
cache = redis.Redis(host='redis', port=6379)
count = cache.incr('hits')
incr('hits') erhöht den Zähler hits bei jedem Aufruf

Redis speichert den aktuellen Zählerwert im RAM

Ideal für einfache, schnelle Zugriffe

🌐 Welche Ports werden genutzt?
Komponente	Port im Container	Port am Host	Zweck
Flask-App	5000	8000	Zugriff auf die Web-App
Redis	6379	nicht freigegeben	Interner Zugriff durch Flask-App

Beispiel aus compose.yaml:
yaml
Kopieren
Bearbeiten
  web:
    ports:
      - "8000:5000"
Bedeutet: localhost:8000 öffnet den Port 5000 im web-Container.

Der Redis-Container braucht keinen veröffentlichten Port, da nur intern genutzt.

⚙️ Was ist die Bedeutung von ENV im Dockerfile?
ENV im Dockerfile definiert Umgebungsvariablen, die beim Bauen des Images und zur Laufzeit des Containers verfügbar sind.

In deinem Dockerfile:
dockerfile
Kopieren
Bearbeiten
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
Erklärung:
Variable	Bedeutung
FLASK_APP=app.py	Gibt Flask an, welches Python-File die App enthält. Ohne das müsste man z. B. flask run -app app.py tippen.
FLASK_RUN_HOST=0.0.0.0	Macht den Flask-Server von außen zugänglich, da 127.0.0.1 nur internen Zugriff erlaubt.

Vorteil:
Diese Variablen machen das Verhalten flexibel und konfigurierbar, ohne den Code zu ändern.
