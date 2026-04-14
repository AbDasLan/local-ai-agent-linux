# local-ai-agent-linux
Dieses Projekt implementiert eine vollständig lokal ausgeführten KI Agenten mit n8n.<br>
Diese Anleitung ermöglicht es jedem, die Installation vollständig zu reproduzieren – inklusive potentiell aufgetretenen Probleme und deren Lösungen.

# tech stack
1. Linux Ubuntu
2. Docker Engine

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
In manchen Anleitungen steht, dass man Docker-Desktop unter diesem Befehl installieren kann <br>
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
Das

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

Inhalt der Datei
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

Speichern: *Ctrl* + *O* → Enter <br>
Beenden: *Ctrl* + *X*

Command um zu prüfen ob die Datei korrekt ist
```bash
docker compose up -d
```
Es wird geprüft ob das n8n image lokal vorhanden ist und die Datei docker-compose.yml wird ausgeführt. 

Nach erfolgreichem Ausführen der Datei sieht der Output wie folgt aus
```bash
WARN[0000] /home/abdul/agentwerk/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
[+] up 15/15
 ✔ Image n8nio/n8n           Pulled                                                                                                                                                     641.7s
 ✔ Network agentwerk_default Created                                                                                                                                                      0.1s
 ✔ Container n8n             Started  
 ```
