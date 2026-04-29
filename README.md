# local-ai-agent-linux
Dieses Projekt implementiert eine vollständig lokal ausgeführten KI Agenten mit n8n.<br>
Diese Anleitung ermöglicht es jedem, die Installation vollständig zu reproduzieren – inklusive potentiell aufgetretenen Probleme und deren Lösungen.

# tech stack
1. Linux Ubuntu
2. Docker Engine
3. n8n (Docker Image)
4. Docker Volumes 

# Dokumentation
Aktualisiere die Packetlisten, damit das System die neusten Infos darüber hat, welche Software verfügbar ist und in welchen Versionen <br>
```bash
sudo apt update
```

Installiere System tools, welche benötigt werden um Docker sicher zu installieren und dessen offizielles Repository hinzuzufügen <br>
```bash
sudo apt install -y ca-certificates curl gnupg lsb-release
```

ACHTUNG!
In manchen Anleitungen steht, dass man Docker-Desktop unter diesem Befehl installieren kann <br>s
```bash
sudo apt install ./docker-desktop-amd64.deb
```
Nachdem Ausführen sollte diese Fehlermeldung auftauchen: *E: Unsupported file ./docker-desktop-amd64.deb given on commandline*

Docker Desktop ist kein normales apt skript, weshalb es nicht auf diese Weise installiert werden kann.
Für dieses Projekt ist die Desktop Variante überflüssig.
Es macht mehr Sinn Docker Engine zu installieren

Falls man sich doch für Docker Desktop entscheidet
```bash
sudo dpkg -i docker-desktop-amd64.deb
sudo apt-get install -f
```

Ordner für Docker Sicherheitsschüssel anlegen
```bash
sudo mkdir -p /etc/apt/keyrings
```

Docker-GPG-Key hinzufügen. Das sorgt dafür, dass das System Docker Packete als vertrauenswürdig sieht
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
| sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Das Docker Repository hinzufügen
```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" \
| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Nach Installation des Docker Repos muss die Packetliste aktualisiert werden
```bash
sudo apt update
```

Docker Engine und folgende weitere Tools installieren<br>
**docker-ce** <br>
→ Die Docker Engine selbst (Startet, stoppt und verwaltet Container)
**docker-ce-cli** <br>
→ Das docker‑Kommando im Terminal (docker run, docker ps, docker stop)<br>
**containerd.io** <br>
→ Der technische Unterbau, der Container tatsächlich ausführt (Docker nutzt das intern) <br>
**docker-buildx-plugin** <br>
→ Werkzeug zum Bauen von Docker‑Images (z. B. eigene Images für später)<br>
**docker-compose-plugin** <br>
→ Ermöglicht das Starten mehrerer Container mit einer Datei (docker compose up für n8n + KI + DB) <br>

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Benutzer zur Docker-Gruppe hinzufügen
```bash
sudo usermod -aG docker $USER
```
Das sorgt dafür, das man beim Starten von Docker ohne sudo nutzen kann. <br>

Beim testen von Docker wird normalerweise eine Fehlermeldung auftreten *permission denied while trying to connect to the docker API*
Das passiert, weil die Benutzer-/Gruppen Berechtigungen bei Ab-/Anmeldung und Neustart aktualisiert werden. <br>
```bash
sudo reboot
```

Um zu testen ob Docker funktioniert
```bash
docker run hello-word
```

Wenn es funktioniert, sollte dieser oder ein ähnlicher Text erscheinen
```text
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete 
d5e71e642bf5: Download complete 
Digest: sha256:452a468a4bf985040037cb6d5392410206e47db9bf5b7278d281f94d1c2d0931
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

Jetzt wird ein Projekt-Ordner im home Verzeichnis angelegt. Dort werden später Konfigurationsdateien liegen.
```bash
mkdir -p agentwerk
cd agentwerk
```

Mit diesem Befehl öffnet sich ein leerer Editor indem die n8n Konfiguration eingetragen wird
```bash
nano docker-compose.yml
```

Der Inhalt dieser Datei
```bash
version: "3.8"

services:
  n8n:
    image: n8nio/n8n
    container_name: n8n
    ports:
      - "5678:5678"
    volumes:
      - ./n8n_data:/home/node/.n8n
    restart: unless-stopped
```
Speichern: *Ctrl* + *O* → Enter
Beenden: *Ctrl* + *X*

Wenn die Datei erfogreich ausgeführt wurde, dann müsste sowas ähnliches im Output stehen
```text
[+] Running 3/3
 ✔ Network agentwerk_default   Created
 ✔ Container agentwerk-n8n-1   Started
 ✔ Container agentwerk-db-1    Started
```

Mit folgendem Befehl kann man überprüfen ob Docker korrekt installiert ist und funktioniert
```bash
docker run hello-world
```

Falls keine Probleme auftreten, sollte das erscheinen
```text
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

Der Befehl um n8n zu starten
```bash
docker volume create n8n_data
docker run -it --rm --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
``
```

Falls n8n nicht installiert ist, wird das Image vom offiziellen Docker Regitry gezogen.
Nachdem das Image erfolgreich gezogen wurde, kommt folgende Meldung
```bash
docker run -it --rm --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
``
n8n_data
Unable to find image 'docker.n8n.io/n8nio/n8n:latest' locally
latest: Pulling from n8nio/n8n
319da0b8bec5: Pull complete 
a8342f1cbe78: Pull complete 
066d0200625d: Pull complete 
ef97b32b4277: Download complete 
Digest: sha256:ad20607cdd24bac004ec44804b6b8ded9a2fbf92ed46c4496bf007762c883af2
Status: Downloaded newer image for docker.n8n.io/n8nio/n8n:latest
docker: Error response from daemon: Conflict. The container name "/n8n" is already in use by container "c00be688f4d59c20a76ee3c5f3eb1962c5bcd0a732ce32fb7e4fc46007cb8519". You have to remove (or rename) that container to be able to reuse that name.

Run 'docker run --help' for more information

``
```
Jetzt muss n8n nochmal gestartet werden
```
docker volume create n8n_data
docker run -it --rm --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
``
```

Es kann passieren, dass folgende Fehlermeldung kommt
```Fehlermeldung
docker: Error response from daemon: Conflict. The container name "/n8n" is already in use by container "c00be688f4d59c20a76ee3c5f3eb1962c5bcd0a732ce32fb7e4fc46007cb8519". You have to remove (or rename) that container to be able to reuse that name.

Run 'docker run --help' for more information
```

Container stoppen
```bash
docker stop n8n
```

Container löschen
```bash
docker rm n8n
``
```

Docker n8n neustarten
```bash
docker run -it --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
``
```

Um zu überprüfen ob n8n läuft
```bash
CONTAINER ID   IMAGE                     COMMAND                  CREATED         STATUS         PORTS                                         NAMES
07af28b17949   docker.n8n.io/n8nio/n8n   "tini -- /docker-ent…"   4 minutes ago   Up 4 minutes   0.0.0.0:5678->5678/tcp, [::]:5678->5678/tcp   n8n
```

Im Browser folgenden link eingeben
```bash
http://localhost:5678
```
Jetzt sollte sich ein Fenster öffnen wo man sich mit Email-Addresse, Name/Vorname und Passwort registrieren kann.
Nach erfolgreicher Registrierung gibt es die Möglichkeit ein "*Onboarding*" für n8n durchzuführen. Dabei wird abgefragt, wofür man n8n nutzen möchte, in welchem Bereich es eingesetzt werden soll und wie groß das Unternehmen ist, für das man es verwenden will. <br>

*n8n läuft komplett kostenlos und lokal* <br>

Das Problem besteht derzeit darin, dass n8n nicht mehr läuft, sobald das Terminal geschlossen wird. Daher automatisieren wir diesen Prozess. <br>

Laufenden Container stoppen
```bash
docker stop n8n
``
```
Container entfernen
```bash
docker rm n8n
```

Container mit Auto Start neu starten
```bash
docker run -d \
--name n8n \
--restart unless stopped \
-p 5678:5678 \
-v n8n_data:/home/node/.n8n \
docker.n8n.io/n8nio/n8n
```
oder in einer Zeile
```bash
docker run -d --name n8n --restart unless-stopped -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
``
```

Erklärung
**docker**
→ Das Docker Programm
**run**
→ Startet einen Container
**-d**
→ Läuft im Hintergrund (Auch wenn Terminal geschlossen wird)
**--name n8n**
→ Container wird der name n8n zugewiesen
**restart unless stopped**
→ Startet n8n automatisch nach Neustart au0er es wird bewusst gestoppt
**-p 5678:5678**
→ Verbindet Laptop-Port 5678 mit n8n Container
**-v n8n_data:/home/node/.n8n**
→ Dauerhafter Speicher für Workflow und Zugangsdaten
**docker.n8n.io/n8nio/n8n**
→ Das offizielle n8n image

Um zu prüfen ob Der Container automatisch nach eine Hochfahren oder Neustart läuft
```bash
docker inspect -f '{{.HostConfig.RestartPolicy.Name}}' n8n
```
Der Output sollte wie folgt aussehen
```bash
unless-stopped
```

Browser Link
```bash
http://localhost:5678
```

