---
layout: page
title: LF7 Docker
image: '/images/pages/docker.png'
---

Bildquelle: Von dotCloud, Inc. - <https://github.com/docker/docker/blob/master/docs/static_files/docker-logo-compressed.png>, Apache License 2.0, <https://commons.wikimedia.org/w/index.php?curid=28456942>

# Installation Webserver für Entwicklung

## Installation

```bash
docker pull ubuntu
```

Wir überprüfen das herunterladen des Images:

```bash
root@proxmox:~# docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
ubuntu                  latest              dd6f76d9cc90        2 days ago          122MB

```
Wir erstellen aus dem Image einen Container, auf dem wir angemeldet sind, um die benötigte Software manuell zu installieren:

```bash
docker run -i -t ubuntu /bin/bash
```

Wir spielen die aktuellen Updates ein:

```bash
apt update
apt -y dist-upgrade
```
Installation der benötigten Programme:

```bash
apt install -y sudo apache2 apache2-utils php php-common openssh-server vim-nox git
```

Di beiden Server openssh und apache2 sollen beim start des Systems automatisch starten. Der einfachste Weg ist der Eintrag in die Datei .bashrc. Dazu müsen wir zuerst den Installationspfad von  service herausfinden:

```bash
root@63fb5dc1ac6b:/# which service                                             
/usr/sbin/service
```

Wir merken uns dn Pfad und wechseln in das Home-Verzeichnis und öffnen dort die Datei .bashrc:

```bash
cd ~
vim .bashrc 
```

Am Ende der Datei (hinter dem #fi) tragen wir ein:

```bash
# Dienste die gestartet werden sollen
/usr/sbin/service apache2 start
/usr/sbin/service ssh start
```
Wir kopieren uns die Container-ID aus der Konsole und verlassen dn Container mit exit.

Als nächstes legen wir ein neues Image an:

```bash
docker commit 63fb5dc1ac6b ubuntu:web
```

Wir schauen uns das neue Image an, dass die installierten Programme enthält:

```bash
root@proxmox:~# docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
ubuntu                  web                 571c92391ecf        47 seconds ago      405MB
ubuntu                  latest              dd6f76d9cc90        2 days ago          122MB
```

Wir stellen nun sicher, dass die Dinste beim start eines Containers starten:

```bash
docker run -i -t ubuntu:web /bin/bash
* Starting Apache httpd web server apache2                                     
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.4. Set the 'ServerName' directive globally to suppress this message
 * 
 * Starting OpenBSD Secure Shell server sshd 
```
Mit einem exit verlassen wir den Container. Wir brauchen jetzt noch einen Inhalt für unsere Seite. Wir gehen dazu auf <http://www.oswd.org/> und suchen uns ein Design aus.

Wir laden das Design herunter und gehen im Browser auf die Downloads um die URL zum Download zu bekommen. Wir legen auf unserem Server ein Verzeichnis an und laden das Design in das Verzeichnis.

```bash
cd ~
mkdir web
wget http://static.oswd.org/designs/3699/Red.zip
```

In dem Verzeichnis web legen wir den statschen Anteil unserer Webseite ab. Also entpacken wir unser Template:

```bash
unzip Red.zip
```

Wir kopieren die entpackten Dateien eine Ebene höher und löschen das Verzeichnis:

```bash
mv Red/* .
rmdir Red
```

## Test des Images mit statischem Webinhalt

Wir startn jetzt einen neuen Container:

```bash
docker run --name=webtest -p 80:80 -v /root/web:/var/www/html -i -t ubuntu:web /bin/bash
```

Mit dem Befehl binden wir das lokale Verzeichnis /root/web  in den Container unter /var/www/html ein und öffnen den Port 80.
Der Server sollte jetzt unter http://IP-Adresse die entsprechende Seite ausliefern.

## Git-Repository

Wir können jetzt ein Git-Repository erzeugen. Dazu verlassen wir den Container. Im Verzeichnis /root/web geben wir ein:

```bash
git init .
```

und überprüfen den Erfolg:

```bash
git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	RedTie.zip
	images/
	index.html

nothing added to commit but untracked files present (use "git add" to track)
```
Wir bekommen den Hinweis, dass wir ein git add ausführen sollten:

```bash
git add *
```

und überprüfen den Erfolg wieder mit einem git status:

```bash
root@proxmox:~/web# git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   RedTie.zip
	new file:   images/bg_menu.gif
	new file:   images/bg_submenu.gif
	new file:   images/style.css
	new file:   images/tie_logo.gif
	new file:   index.html
```

Wir führen ein git commit durch:

```bash
root@proxmox:~/web# git commit -m "Erste Seite"
[master (root-commit) 7e7e763] Erste Seite
 Committer: root <root@proxmox.fritz.box>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly. Run the
following command and follow the instructions in your editor to edit
your configuration file:

    git config --global --edit

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

 6 files changed, 166 insertions(+)
 create mode 100644 RedTie.zip
 create mode 100644 images/bg_menu.gif
 create mode 100644 images/bg_submenu.gif
 create mode 100644 images/style.css
 create mode 100644 images/tie_logo.gif
 create mode 100644 index.html
```

und überprüfen den Erfolg:

```bash
root@proxmox:~/web# git status
On branch master
nothing to commit, working tree clean
```

Jetzt wäre der Zeitpunkt für einen Account bei github und der Angabe von Name und E-Mail. Wir überspringen den Schritt.

Wir könnten jetzt auch über den oben installierten git-Client das Repository direkt in das Filesystem des Container laden ohne das wir das lokale Filesystem brauchen!

Wir starten unseren Container:

```bash
docker start webtest
```

Wir melden uns auf den Container an:

```bash
docker attach webtest
```

Wir können über das Git-Repository die Wbsite in jedem Container oder auf dem Muttrtsystem ändern und diese durch das Tolkit von git nachvollziehbar ändern und auch evntuelle Änderungen rückgängig machen. Aussrdem haben wir ein externes Backup der Inhalte.

Wir wechseln in das Verzeichnis /var/www/html:

```bash
cd /var/www/html
git status
```

Und sehen mit dem zwiten Befehl, dass das Repository "sauber" ist.

## git clone

Wir bereiten unser System auf einen zweiten Container vor. Dafür kopieren wir das Repository um die Websites der Produktion von der Entwicklung zu trennen.

```bash
mkdir dockergit
git clone root@localhost:/root/web/ dockergit
```

Wir haben jetzt eine Kopie des Verzeichnisses web in dockergit. Beides sind getrennte Git-Repositories.

Wir starten einen neuen Container:

```bash
docker run -d -i -t --name=webdev -p 8081:80 -v /root/dockergit:/var/www/html ubuntu:web /bin/bash
```
Wir überprüfen, dass der Container läuft:

```bash
root@proxmox:~/dockergit# docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED              STATUS                PORTS                                                NAMES
5f3964eaabf6        ubuntu:web              "/bin/bash"              About a minute ago   Up About a minute     0.0.0.0:8081->80/tcp                                 webdev
```
Wir starten noch einen witeren Container auf Port 8083 und dem Namen webdev2:

```bash
docker run -d -i -t --name=webdev2 -p 8082:80 -v /root/dockergit:/var/www/html ubuntu:web /bin/bash
```

Und überprüfen, dass beide Container laufen:

```bash
root@proxmox:~/dockergit# docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS                PORTS                                                NAMES
e93677ab35a8        ubuntu:web              "/bin/bash"              2 minutes ago       Up 2 minutes          0.0.0.0:8082->80/tcp                                 webdev2
5f3964eaabf6        ubuntu:web              "/bin/bash"              6 minutes ago       Up 6 minutes          0.0.0.0:8081->80/tcp                                 webdev

```
Überprüfe auch über den Browser, dass beide Maschinn erreichbar sind (http://192.168.1.50:8082 und http://192.168.1.50:8081).

# Nginx

Wir möchten jetzt gerne beide Container unter Port 80 zur Verfügung stellen. Dafür starten wir einen weitern Container mit nginx:

```bash
docker run -i -t --name nginx -p 80:80 nginx /bin/bash
```

Wir starten Nginx:

```bash
service nginx start 
```

und überprüfen im Browser, ob Nginx auf Port 80 erreichbar ist.

Wir wechseln in das Konfigurationsverzeichnis:

```bash
cd /etc/nginx/conf.d
```
Wir spielen die Updates ein und installieren unseren Lieblings-Editor:

```bash
apt update
apt dist-upgrade
apt install vim-nox
```
Und öffnen die Datei "default.conf":

```bash
vim default.conf
```
Wir richten nun einen sogenannten Round Robin Proxy ein (siehe: <https://de.wikipedia.org/wiki/Round_Robin_(Informatik)>).

Wir ändern die Konfiguration wie folgt ab:

```bash
upstream container{
	server 192.168.1.50:8081;
	server 192.168.1.50:8082;
}
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
	proxy_pass http://container;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}

```
Hinzugefügt wurden die ersten 4 Zeilen. Sie sagen aus, das die angegenen Ports des Servers über Port 80 ausgeliefert werden sollen. Und der Abschnitt location wurd angepasst, um die Vorbereitungen zu aktivieren.

Dann starten wir nginx neu:

```bash
service nginx restart
```
Wenn wir jetzt im Browser http://192.168.1.50 eingeben, dann sollte einer der beidn Container die Anfrage beantworten und unsere vertraute Webseite erscheinen.

