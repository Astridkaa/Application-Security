 # Übung 2: Linux Container - LXD/Incus
 
 ### Datum:

04.03.2025

### Gruppenmitglieder: 
Philip Magnus
Lorenzo Haidinger
Astrid Kuzma-Kuzniarski

## Aufgabenstellung

Diese Laborübung befasst sich mit der Einrichtung und Verwaltung von Linux-Containern mithilfe von **LXD** oder **Incus**. Ziel ist es, eine containerisierte Umgebung aufzusetzen, Netzwerke und Speicher zu verwalten und Container zu konfigurieren.
 
 ## 0. Aufsetzen und Konfiguration des Systems
 
 Wir haben uns für `LXD` via `snap` entschieden:
 
 ```bash 
# root@host
sudo snap install lxd
```
```bash
sudo snap refresh lxd 
```
 
Nun fügen wir einen neuen Benutzer mit einem Passwort ein und fügen ihn zur Gruppe `lxd` hinzu:

```bash
# root @ host
sudo useradd -m -s /usr/bin/bash lab 

sudo passwd lab
# hier haben wir 'passwort' als passwort benutzt

sudo usermod -aG lxd lab

su - lab
```

Nun kann man als user `lab` versuchen `lxd` zu starten:

```bash
# lab@host 

getent group lxd | grep lab


newgrp lxd 


lxd init --minimal
```

## 1. Informationen sammeln

Durch `lxc --help` und `lxc [command] --help` können wir folgende Fragen beantworten:

 
### 1.1 Image Quellen: 
- _Aus welchen Quellen können Sie Images beziehen? Welche Images sind dort vorhanden?_

```
# lab@host 
lxc remote list -f compact
```



![](https://i.imgur.com/eNgAmbB.png)


### 1.2 Netzwerke
- _Welche Netzwerke sind vorhanden? Wie sind diese konfiguriert?_

```bash
# lab@host 
$ lxc network list -f yaml
```

```yaml=

- name: lxdbr0
  description: ""
  type: bridge
  managed: true
  status: Created
  config:
    ipv4.address: 10.216.141.1/24
    ipv4.nat: "true"
    ipv6.address: fd42:20de:aea:4aeb::1/64
    ipv6.nat: "true"
  used_by:
  - /1.0/profiles/default
  locations:
  - none
- name: lo
  description: ""
  type: loopback
  managed: false
  status: ""
  config: {}
  used_by: []
  locations: []
- name: eno1
  description: ""
  type: physical
  managed: false
  status: ""
  config: {}
  used_by: []
  locations: []
- name: enp3s0f0
  description: ""
  type: physical
  managed: false
  status: ""
  config: {}
  used_by: []
  locations: []
- name: enp3s0f1
  description: ""
  type: physical
  managed: false
  status: ""
  config: {}
  used_by: []
  locations: []
- name: enp4s0f0
  description: ""
  type: physical
  managed: false
  status: ""
  config: {}
  used_by: []
  locations: []
- name: enp4s0f1
  description: ""
  type: physical
  managed: false
  status: ""
  config: {}
  used_by: []
  locations: []
- name: wlp6s0
  description: ""
  type: physical
  managed: false
  status: ""
  config: {}
  used_by: []
  locations: []
- name: virbr0
  description: ""
  type: bridge
  managed: false
  status: ""
  config: {}
  used_by: []
  locations: []

```

### 1.3 Storage Pools
- _Welche Storage Pools sind vorhanden? Wie sind diese konfiguriert_

```bash
# lab@host 
$ lxc storage list -f yaml
```

```yaml

- name: default
  description: ""
  driver: dir
  status: Created
  config:
    source: /var/snap/lxd/common/lxd/storage-pools/default
  used_by:
  - /1.0/profiles/default
  locations:
  - none

```

## 2. Container erstellen: 

Durch folgende Befehle wurden die Container erstellt:
```bash
# lab@host 
$ lxc launch ubuntu:24.04 container1
```
```bash
# container1@host (neues Terminal)
$ lxc exec container1 -- su
``` 

```bash
# lab@host
lxc launch ubuntu:24.04 container2
```

```bash
# container2@host (neues Terminal)
$ lxc exec container2 -- su
```


### 2.1 Memory usage

- _Wieviel Speicher benötigen die Container jeweils auf dem Host?_

Zunächst wird mit dem Befehl `lxc storage list` überprüft, wo sich die Speicherdateien befinden:

```bash
# lab@host
$ lxc storage list
```

![](https://i.imgur.com/dT6vLBG.png)

Nun wissen wir das die configs sich hier befinden: 

`/var/snap/lxd/common/lxd/storage-pools/default`

Mit dem Befehl `du` können wir die Größe des angegebenen Containers ermitteln:

![](https://i.imgur.com/pGHPlyw.png)


Der `Container1` nutzt ca. 1,3GB Speicherplatz.

### 2.2 PRIVILEGES

- _Sind die Container privileged ?_

[UID Mappings and Privileged Containers](https://documentation.ubuntu.com/server/how-to/containers/lxd-containers/index.html#uid-mappings-and-privileged-containers)

Per default sind die Container nicht als privileged eingestellt. 

Wenn wir uns die config ansehen, haben wir keinen Zugriff auf `security.privileged`, welches uns zeigt, dass die Container nicht priviligiert sind.

Zum Überprüfen verwenden wir den Befehl:

```bash 
# lab@host
lxc config get container1 security.privileged
```


### 2.3 Process UIDs

- _Unter welcher UID laufen Prozesse auf dem Host, die im Container von root gestartet werden?_

Zuerst starten wir in jedem Container einen Prozess als Root-Benutzer:

```bash 
# root@container1
# start a root process
$ less .bashrc
```

```bash 
# root@container2
# start a root process
$ nano .profile
```

Dann listen wir die Prozesse auf dem Host auf und suchen nach den Prozessen unserer Container:

```bash
# lab@host
$ ps aux | grep -e less -e nano
```

![](https://i.imgur.com/iyIU5xU.png)

### 2.4 Container als privileged konfigurieren

- _Wie können Sie konfigurieren, ob ein Container privileged ist? Was ändert sich an den UIDs dann?_

Dafür setzen wir das `security.priveleged`-Flag bei einem der Container und starten ihn neu.

```bash
$ lxc config set container0 security.privileged true
```

Und durch folgenden Befehl haben wir nachgeschaut ob der Prozess aus dem privilegierten Container als root auf dem Host läuft.


```bash
ps aux | grep -e nano -e less
```

![](https://i.imgur.com/on5Txbp.png)


## 3. Profil

> Mit der Laborbetreuung abgesprochen, da es sonst zu Problemen in der Konfiguration kommt.

Zuerst erstellen wir den neuen SSH-Schlüssel mit dem Befehl `ssh-keygen`. 

```bash
# lab@host
ssh-keygen
```

Dann konfigurieren wir das lxc-Profil mit den lxc-Profil-Unterbefehlen `create` und `edit`.

```bash
# lab@host
lxc profile create profile1
```

Wir editieren als nächstes das Profil:

```bash
# lab@host
EDITOR=nano lxc profile edit profile0
```

Jetzt richten wir die Profilkonfiguration wie folgt ein:

```bash
name: profile0
description: This is my profile
config:
  boot.autostart: "true"
  cloud-init.user-data: |
    #cloud-config
    users:
      - name: ubuntu
        ssh_authorized_keys:
          - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIK812XHiDlm7Q3tVWmf2kYaAyAPrb9H3LH5Un/0egmJ9 lab@host34
devices:
  eth0:
    name: eth0
    nictype: bridged
    parent: br0
    type: nic
  root:
    path: /
    pool: default
    type: disk
```



## 4. Netzwerk konfigurieren 

Zunächst erstellen wir eine Bridge auf dem Host. Dazu bearbeiten wir die netplan-Konfiguration mit dem Befehl:

```bash
# lab@host
sudo nano /etc/netplan/01-eno1.yaml
```

Dort schreiben wir folgendes rein:

```bash
network:
  version: 2
  renderer: networkd
  ethernets:
    eno1:
      nameservers:
        addresses:
          - 8.8.8.8
      dhcp4: yes

  bridges:
    br0:
      dhcp4: yes
      macaddress: 2a:c5:3c:bf:86:33
      nameservers:
        addresses:
          - 8.8.8.8
      interfaces:
        - eno1
```

Mit dem Aufruf von `ip a` sieht man nun einen neuen Eintrag: `br0`


![](https://i.imgur.com/Meyjpxq.png)

Mit dem folgenden Befehl können wir einen neuen Container mit diesem Profil zu starten:

```bash
# lab@host
lxc launch ubuntu:24.04 --profile profile0
```

Anschließend können wir mit `lxc list` die IP-Adresse unseres Containers der mit dem Profil angelegt wurde.

```bash
lab@host34:~$ lxc list
+----------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
|      NAME      |  STATE  |         IPV4          |                     IPV6                      |   TYPE    | SNAPSHOTS |
+----------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
| container1     | RUNNING | 10.64.248.150 (eth0)  | fd42:d300:d268:f6cb:216:3eff:fec6:b615 (eth0) | CONTAINER | 0         |
+----------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
| container2     | RUNNING | 10.64.248.109 (eth0)  | fd42:d300:d268:f6cb:216:3eff:fe87:778d (eth0) | CONTAINER | 0         |
+----------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
| funky-akita    | RUNNING | 192.168.10.171 (eth0) |                                               | CONTAINER | 0         |
+----------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
| good-monitor   | STOPPED |                       |                                               | CONTAINER | 0         |
+----------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
```

Wir haben diese Adresse verwendet, um uns mit unserem Schlüssel über SSH mit dem Container zu verbinden.

```bash
$ ssh -i .ssh/lxd_lab ubuntu@192.168.10.183

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

$ 

```

> ACHTUNG: der korrekte ssh key muss bei der Verbindung genau angegeben werden.

## 5. Shared Directory

Um ein shared directory zu erstellen, erstellen wir zunächst das lokale directory und hängen es dann in den Container ein:

```bash
# lab@host
mkdir shared_dir
```

```bash
# lab@host
lxc config device add container2 share1 disk source=/home/lab/shared_dir path=/mnt/shared_dir
```

Damit können wir den Inhalt des Ordners im Container lesen, aber damit unser Container ihn bearbeiten kann, müssen wir die UID des Container-Benutzers auf den entsprechenden Host-Benutzer umstellen. Das haben wir mit dem folgenden Befehl getan:

```bash
# lab@host
lxc config set container2 raw.idmap "both 1001 1001"
```

Nach dem Neustart des Containers konnten wir das gemeinsam genutzte Verzeichnis sowohl vom Container als auch vom Host aus bearbeiten.


```bash
lxc restart container2
```

```bash
lab@localhost:~$ lxc shell container2
root@container2:~# touch /mnt/shared_dir/test2.md
root@container2:~# 
```

## 6. btrfs


Dafür verwenden wir den Befehl `lxc storage create`, um einen Speicherpool mit einer maximalen Größe von 20 GB zu erstellen, da die Standardgröße 2 GB beträgt.

```bash
# lab@host
lxc storage create lab_pool btrfs size=20GB
```

Dann haben wir den Abschnitt `devices` im Profil mit `lxc profile edit` bearbeitet, wo wir den Speicherpool festgelegt und die Größenbeschränkung für das Root-Dateisystem durchgesetzt haben:

```bash
...
devices:
  root:
    path: /
    pool: lab_pool
    size: 10GB
    type: disk
...
```

Nun starten wir einen neuen Container mit diesem Profil und verwenden darin den Befehl `fallocate`, um zu überprüfen, dass die Größe des Root-Dateisystems begrenzt ist. 

Wie in der Kommandozeilen-Ausgabe zu sehen ist, ist die verfügbare Festplatte begrenzt, obwohl der Befehl `df` die gesamte Poolgröße anzeigt.

```bash
lab@host34:~$ lxc launch ubuntu:24.04 --profile profile0
Launching the instance
Instance name is: flying-pheasant
lab@host34:~$ lxc shell flying-pheasant
root@flying-pheasant:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/loop4       19G  1.1G   18G   6% /
none            492K  4.0K  488K   1% /dev
tmpfs           100K     0  100K   0% /dev/lxd
tmpfs           100K     0  100K   0% /dev/.lxd-mounts
tmpfs           7.8G     0  7.8G   0% /dev/shm
tmpfs           3.1G  180K  3.1G   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           1.6G   12K  1.6G   1% /run/user/0
```

Wie in der Kommandozeilen-Ausgabe zu sehen ist, ist die verfügbare Festplatte begrenzt, obwohl der Befehl `df` die gesamte Poolgröße anzeigt.

```bash
root@flying-pheasant:~# fallocate -l 11G test
fallocate: fallocate failed: Disk quota exceeded
root@flying-pheasant:~# fallocate -l 5G test
root@flying-pheasant:~# ls -la
total 5242888
drwx------ 1 root root         46 Mar 18 18:09 .
drwxr-xr-x 1 root root        244 Mar 13 08:46 ..
-rw-r--r-- 1 root root       3106 Apr 22  2024 .bashrc
-rw-r--r-- 1 root root        161 Apr 22  2024 .profile
drwx------ 1 root root         30 Mar 18 18:05 .ssh
-rw-r--r-- 1 root root 5368709120 Mar 18 18:10 test
```
