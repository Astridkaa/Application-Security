# Übung 3 - Ansible

Datum: 26.03.2025

## Gruppenmitglieder:

- Lorenzo Haidinge
- Astrid Kuzma-Kuzniarski
- Philip Magnus

## Aufgabenstellung

Dieser Laborübung behandelt die praktische Anwendung von Ansible. Das Ziel ist, containerisierte Systeme automatisiert zu konfigurieren, Sicherheitsrichtlinien umzusetzen, standardisierte Rollen einzusetzen und sensible Daten sicher mittels Ansible Vault zu verwalten.

# 0. Installation

Die Übung wurde auf einem Ubuntu 24.04LTS System mit dem Patchstand vom 26.03.2025 durchgefürht.

Um Ansible für die Übung nutzen zu können muss dies erst mit folgendem Command installiert werden:

```bash
it-security@host35:~$ sudo apt install ansible

[sudo] Passwort für it-security:
Das hat nicht funktioniert, bitte nochmal probieren.
[sudo] Passwort für it-security:
Paketlisten werden gelesen… Fertig
Abhängigkeitsbaum wird aufgebaut… Fertig
Statusinformationen werden eingelesen… Fertig
Die folgenden Pakete wurden automatisch installiert und werden nicht mehr benötigt:
  libatk-bridge2.0-0t64:i386 libatk1.0-0t64:i386 libatspi2.0-0t64:i386 libavahi-client3:i386 libavahi-common-data:i386 libavahi-common3:i386 libcolord2:i386 libcups2t64:i386
  libdecor-0-0:i386 libdecor-0-plugin-1-gtk:i386 libepoxy0:i386 libgssapi-krb5-2:i386 libgtk-3-0t64:i386 libk5crypto3:i386 libkeyutils1:i386 libkrb5-3:i386 libkrb5support0:i386
  liblcms2-2:i386 libsdl2-2.0-0:i386 libusb-1.0-0:i386 libwayland-cursor0:i386 libwayland-egl1:i386 libxcomposite1:i386 libxcursor1:i386 libxdamage1:i386 libxi6:i386 libxrandr2:i386
  steam-libs:i386
Verwenden Sie »sudo apt autoremove«, um sie zu entfernen.
Die folgenden zusätzlichen Pakete werden installiert:
  ansible-core python3-jmespath python3-kerberos python3-libcloud python3-lockfile python3-ntlm-auth python3-passlib python3-requests-ntlm python3-resolvelib python3-selinux
  python3-winrm python3-xmltodict
Vorgeschlagene Pakete:
  cowsay sshpass python-lockfile-doc
Die folgenden NEUEN Pakete werden installiert:
  ansible ansible-core python3-jmespath python3-kerberos python3-libcloud python3-lockfile python3-ntlm-auth python3-passlib python3-requests-ntlm python3-resolvelib python3-selinux
  python3-winrm python3-xmltodict
0 aktualisiert, 13 neu installiert, 0 zu entfernen und 1 nicht aktualisiert.
Es müssen 19,3 MB an Archiven heruntergeladen werden.
Nach dieser Operation werden 314 MB Plattenplatz zusätzlich benutzt.
Möchten Sie fortfahren? [J/n]
Holen:1 http://at.archive.ubuntu.com/ubuntu noble/universe amd64 python3-resolvelib all 1.0.1-1 [25,7 kB]
Holen:2 http://at.archive.ubuntu.com/ubuntu noble/universe amd64 ansible-core all 2.16.3-0ubuntu2 [1.280 kB]
Holen:3 http://at.archive.ubuntu.com/ubuntu noble/universe amd64 ansible all 9.2.0+dfsg-0ubuntu5 [16,4 MB]
Holen:4 http://at.archive.ubuntu.com/ubuntu noble/main amd64 python3-jmespath all 1.0.1-1 [21,3 kB]
Holen:5 http://at.archive.ubuntu.com/ubuntu noble/universe amd64 python3-kerberos amd64 1.1.14-3.1build9 [21,2 kB]
Holen:6 http://at.archive.ubuntu.com/ubuntu noble/universe amd64 python3-lockfile all 1:0.12.2-3 [13,7 kB]
Holen:7 http://at.archive.ubuntu.com/ubuntu noble/universe amd64 python3-libcloud all 3.4.1-5 [751 kB]
Holen:8 http://at.archive.ubuntu.com/ubuntu noble/universe amd64 python3-ntlm-auth all 1.5.0-1 [21,3 kB]
Holen:9 http://at.archive.ubuntu.com/ubuntu noble/universe amd64 python3-requests-ntlm all 1.1.0-3 [6.308 B]
Holen:10 http://at.archive.ubuntu.com/ubuntu noble-updates/universe amd64 python3-selinux amd64 3.5-2ubuntu2.1 [165 kB]
Holen:11 http://at.archive.ubuntu.com/ubuntu noble/main amd64 python3-xmltodict all 0.13.0-1 [13,4 kB]
Holen:12 http://at.archive.ubuntu.com/ubuntu noble/universe amd64 python3-winrm all 0.4.3-2 [31,9 kB]
Holen:13 http://at.archive.ubuntu.com/ubuntu noble/main amd64 python3-passlib all 1.7.4-4 [476 kB]
Es wurden 19,3 MB in 7 s geholt (2.907 kB/s).
Vormals nicht ausgewähltes Paket python3-resolvelib wird gewählt.
(Lese Datenbank ... 404469 Dateien und Verzeichnisse sind derzeit installiert.)
Vorbereitung zum Entpacken von .../00-python3-resolvelib_1.0.1-1_all.deb ...
Entpacken von python3-resolvelib (1.0.1-1) ...
Vormals nicht ausgewähltes Paket ansible-core wird gewählt.
Vorbereitung zum Entpacken von .../01-ansible-core_2.16.3-0ubuntu2_all.deb ...
Entpacken von ansible-core (2.16.3-0ubuntu2) ...
Vormals nicht ausgewähltes Paket ansible wird gewählt.
Vorbereitung zum Entpacken von .../02-ansible_9.2.0+dfsg-0ubuntu5_all.deb ...
Entpacken von ansible (9.2.0+dfsg-0ubuntu5) ...
Vormals nicht ausgewähltes Paket python3-jmespath wird gewählt.
Vorbereitung zum Entpacken von .../03-python3-jmespath_1.0.1-1_all.deb ...
Entpacken von python3-jmespath (1.0.1-1) ...
Vormals nicht ausgewähltes Paket python3-kerberos wird gewählt.
Vorbereitung zum Entpacken von .../04-python3-kerberos_1.1.14-3.1build9_amd64.deb ...
Entpacken von python3-kerberos (1.1.14-3.1build9) ...
Vormals nicht ausgewähltes Paket python3-lockfile wird gewählt.
Vorbereitung zum Entpacken von .../05-python3-lockfile_1%3a0.12.2-3_all.deb ...
Entpacken von python3-lockfile (1:0.12.2-3) ...
Vormals nicht ausgewähltes Paket python3-libcloud wird gewählt.
Vorbereitung zum Entpacken von .../06-python3-libcloud_3.4.1-5_all.deb ...
Entpacken von python3-libcloud (3.4.1-5) ...
Vormals nicht ausgewähltes Paket python3-ntlm-auth wird gewählt.
Vorbereitung zum Entpacken von .../07-python3-ntlm-auth_1.5.0-1_all.deb ...
Entpacken von python3-ntlm-auth (1.5.0-1) ...
Vormals nicht ausgewähltes Paket python3-requests-ntlm wird gewählt.
Vorbereitung zum Entpacken von .../08-python3-requests-ntlm_1.1.0-3_all.deb ...
Entpacken von python3-requests-ntlm (1.1.0-3) ...
Vormals nicht ausgewähltes Paket python3-selinux wird gewählt.
Vorbereitung zum Entpacken von .../09-python3-selinux_3.5-2ubuntu2.1_amd64.deb ...
Entpacken von python3-selinux (3.5-2ubuntu2.1) ...
Vormals nicht ausgewähltes Paket python3-xmltodict wird gewählt.
Vorbereitung zum Entpacken von .../10-python3-xmltodict_0.13.0-1_all.deb ...
Entpacken von python3-xmltodict (0.13.0-1) ...
Vormals nicht ausgewähltes Paket python3-winrm wird gewählt.
Vorbereitung zum Entpacken von .../11-python3-winrm_0.4.3-2_all.deb ...
Entpacken von python3-winrm (0.4.3-2) ...
Vormals nicht ausgewähltes Paket python3-passlib wird gewählt.
Vorbereitung zum Entpacken von .../12-python3-passlib_1.7.4-4_all.deb ...
Entpacken von python3-passlib (1.7.4-4) ...
python3-lockfile (1:0.12.2-3) wird eingerichtet ...
python3-passlib (1.7.4-4) wird eingerichtet ...
python3-ntlm-auth (1.5.0-1) wird eingerichtet ...
python3-resolvelib (1.0.1-1) wird eingerichtet ...
python3-kerberos (1.1.14-3.1build9) wird eingerichtet ...
ansible-core (2.16.3-0ubuntu2) wird eingerichtet ...
python3-libcloud (3.4.1-5) wird eingerichtet ...
python3-xmltodict (0.13.0-1) wird eingerichtet ...
python3-jmespath (1.0.1-1) wird eingerichtet ...
python3-selinux (3.5-2ubuntu2.1) wird eingerichtet ...
ansible (9.2.0+dfsg-0ubuntu5) wird eingerichtet ...
python3-requests-ntlm (1.1.0-3) wird eingerichtet ...
python3-winrm (0.4.3-2) wird eingerichtet ...
Trigger für man-db (2.12.0-4build2) werden verarbeitet ...
```

Anschließend wurde für die Übung ein eigener Ordner angelegt und in diesen für weitere Arbeit  gewechselt um dort Projekt-Dateien abzulegen:

```bash
it-security@host35:~$ mkdir ansible && cd ansible
```

Um die von der Übungsanforderung benötigten Container zu erstellen mussten noch LXC und LXD installiert werden:

```bash
it-security@host35:~$ snap install lxd
```

Für die Übung sollten die folgenden 3 Linux Container erstellt werden:

- web1
- web2
- mail1

Die Container wurden mit einem Ubuntu 24.04 wie folgt erstellt:

```bash
it-security@host35:~$ lxc launch ubuntu:24.04 web1

Launching web1
```

```bash
it-security@host35:~$ lxc launch ubuntu:24.04 web2

Launching web2
```

```bash
it-security@host35:~$ lxc launch ubuntu:24.04 mail1

Launching mail1
```

Ansible benötigt einen SSH-Key auf den Maschinen, die über Ansible gemanaged werden sollen, daher wird wie folgt ein SSH-Key erstellt:

```bash
it-security@host35:~$ ssh-keygen

Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/it-security/.ssh/id_ed25519): /home/it-security/.ssh/ansible
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/it-security/.ssh/ansible
Your public key has been saved in /home/it-security/.ssh/ansible.pub
The key fingerprint is:
SHA256:dViv37ZacnC+w72BdjnNC+jAc7bUQ4s8etHt/BwqzHg it-security@host35
The key's randomart image is:
+--[ED25519 256]--+
|            .    |
|           o .   |
|          o . .  |
|         . . .   |
|        S   oo.. |
|        . ..=o*+o|
|         ++O.O=X*|
|         .OE+ OBO|
|         .oo.o.=*|
+----[SHA256]-----+
```

Daraus resultiert der folgende Public- und Private-Key:

```bash
it-security@host35:~$ cat ~/.ssh/ansible.pub

ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBF3e+vS/Qq04DvhvO5Zup3zsy3mR4KKvFeiRTLjPNNe it-security@host35

it-security@host35:~$ cat ~/.ssh/ansible

-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACARd3vr0v0KtOA74bzuWbqd87Mt5keCirxXokUy4zzTXgAAAJgck/9pHJP/
aQAAAAtzc2gtZWQyNTUxOQAAACARd3vr0v0KtOA74bzuWbqd87Mt5keCirxXokUy4zzTXg
AAAECSM4Q9NroY1F/3jOAtAKDzNNV9Uqwb8A0WDVmxHWBKiRF3e+vS/Qq04DvhvO5Zup3z
sy3mR4KKvFeiRTLjPNNeAAAAEHBoaWxpcEBmcmFtZXdvcmsBAgMEBQ==
-----END OPENSSH PRIVATE KEY-----
```

Der Public-Key wurde in der Config jedes Containers mit `lxc config edit <container>` hinterlegt:

```bash
it-security@host35:~$ lxc config edit web1
```

```yaml
### This is a YAML representation of the configuration.
### Any line starting with a '# will be ignored.
###
### A sample configuration looks like:
### name: instance1
### profiles:
### - default
### config:
###   volatile.eth0.hwaddr: 00:16:3e:e9:f8:7f
### devices:
###   homedir:
###     path: /extra
###     source: /home/user
###     type: disk
### ephemeral: false
###
### Note that the name is shown but cannot be changed

name: web1
description: ""
status: Running
status_code: 103
created_at: 2025-03-19T18:29:51.935988371Z
last_used_at: 2025-03-19T21:03:36.468308668Z
location: none
type: container
project: default
architecture: x86_64
ephemeral: false
stateful: false
profiles:
- default
config:
  image.architecture: amd64
  image.description: ubuntu 24.04 LTS amd64 (release) (20250313)
  image.label: release
  image.os: ubuntu
  image.release: noble
  image.serial: "20250313"
  image.type: squashfs
  image.version: "24.04"
  user.user-data: |
    #cloud-config
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBF3e+vS/Qq04DvhvO5Zup3zsy3mR4KKvFeiRTLjPNNe it-security@host35
  volatile.base_image: 9203c53127ae6b11280d9b94b1579ed71b9964a428bc724eed96212f9ca699fc
  volatile.cloud-init.instance-id: 9d4d4644-7211-491b-8657-b7d17d3e5b59
  volatile.eth0.host_name: veth205246e5
  volatile.eth0.hwaddr: 00:16:3e:e2:33:58
  volatile.idmap.base: "0"
  volatile.idmap.current: '[{"Isuid":true,"Isgid":false,"Hostid":1000000,"Nsid":0,"Maprange":1000000000},{"Isuid":false,"Isgid":true,"Hostid":1000000,"Nsid":0,"Maprange":1000000000}]'
  volatile.idmap.next: '[{"Isuid":true,"Isgid":false,"Hostid":1000000,"Nsid":0,"Maprange":1000000000},{"Isuid":false,"Isgid":true,"Hostid":1000000,"Nsid":0,"Maprange":1000000000}]'
  volatile.last_state.idmap: '[]'
  volatile.last_state.power: RUNNING
  volatile.last_state.ready: "false
  volatile.uuid: f12c6580-1346-438e-b60e-d904ed017072
  volatile.uuid.generation: f12c6580-1346-438e-b60e-d904ed017072
devices: {}
  ```
  
  ```bash
it-security@host35:~$ lxc config edit web2
```

```yaml
### This is a YAML representation of the configuration.
### Any line starting with a '# will be ignored.
###
### A sample configuration looks like:
### name: instance1
### profiles:
### - default
### config:
###   volatile.eth0.hwaddr: 00:16:3e:e9:f8:7f
### devices:
###   homedir:
###     path: /extra
###     source: /home/user
###     type: disk
### ephemeral: false
###
### Note that the name is shown but cannot be changed

name: web2
description: ""
status: Running
status_code: 103
created_at: 2025-03-19T18:30:02.219875498Z
last_used_at: 2025-03-19T21:07:55.585624317Z
location: none
type: container
project: default
architecture: x86_64
ephemeral: false
stateful: false
profiles:
- default
config:
  image.architecture: amd64
  image.description: ubuntu 24.04 LTS amd64 (release) (20250313)
  image.label: release
  image.os: ubuntu
  image.release: noble
  image.serial: "20250313"
  image.type: squashfs
  image.version: "24.04"
  user.user-data: |
    #cloud-config
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBF3e+vS/Qq04DvhvO5Zup3zsy3mR4KKvFeiRTLjPNNe it-security@host
  volatile.base_image: 9203c53127ae6b11280d9b94b1579ed71b9964a428bc724eed96212f9ca699fc
  volatile.cloud-init.instance-id: 4321b9ab-d3ce-49bd-b927-c179b938e045
  volatile.eth0.host_name: vethb6626093
  volatile.eth0.hwaddr: 00:16:3e:d8:94:a2
  volatile.idmap.base: "0"
  volatile.idmap.current: '[{"Isuid":true,"Isgid":false,"Hostid":1000000,"Nsid":0,"Maprange":1000000000},{"Isuid":false,"Isgid":true,"Hostid":1000000,"Nsid":0,"Maprange":1000000000}]'
  volatile.idmap.next: '[{"Isuid":true,"Isgid":false,"Hostid":1000000,"Nsid":0,"Maprange":1000000000},{"Isuid":false,"Isgid":true,"Hostid":1000000,"Nsid":0,"Maprange":1000000000}]'
  volatile.last_state.idmap: '[]'
  volatile.last_state.power: RUNNING
  volatile.last_state.ready: "false"
  volatile.uuid: 53848eb1-5f5d-4d05-9d08-4cdc88de9655
  volatile.uuid.generation: 53848eb1-5f5d-4d05-9d08-4cdc88de9655
devices: {}
  ```

```bash
it-security@host35:~$ lxc config edit mail1
```

```yaml
### This is a YAML representation of the configuration.
### Any line starting with a '# will be ignored.
###
### A sample configuration looks like:
### name: instance1
### profiles:
### - default
### config:
###   volatile.eth0.hwaddr: 00:16:3e:e9:f8:7f
### devices:
###   homedir:
###     path: /extra
###     source: /home/user
###     type: disk
### ephemeral: false
###
### Note that the name is shown but cannot be changed

name: mail1
description: ""
status: Running
status_code: 103
created_at: 2025-03-19T18:30:07.989885373Z
last_used_at: 2025-03-19T21:07:55.591883772Z
location: none
type: container
project: default
architecture: x86_64
ephemeral: false
stateful: false
profiles:
- default
config:
  image.architecture: amd64
  image.description: ubuntu 24.04 LTS amd64 (release) (20250313)
  image.label: release
  image.os: ubuntu
  image.release: noble
  image.serial: "20250313"
  image.type: squashfs
  image.version: "24.04"
  user.user-data: |
    #cloud-config
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBF3e+vS/Qq04DvhvO5Zup3zsy3mR4KKvFeiRTLjPNNe it-security@host35
  volatile.base_image: 9203c53127ae6b11280d9b94b1579ed71b9964a428bc724eed96212f9ca699fc
  volatile.cloud-init.instance-id: a4025f9c-627b-4701-9ffc-e477f6a8c3aa
  volatile.eth0.host_name: vethb8c2163c
  volatile.eth0.hwaddr: 00:16:3e:d7:19:86
  volatile.idmap.base: "0"
  volatile.idmap.current: '[{"Isuid":true,"Isgid":false,"Hostid":1000000,"Nsid":0,"Maprange":1000000000},{"Isuid":false,"Isgid":true,"Hostid":1000000,"Nsid":0,"Maprange":1000000000}]'
  volatile.idmap.next: '[{"Isuid":true,"Isgid":false,"Hostid":1000000,"Nsid":0,"Maprange":1000000000},{"Isuid":false,"Isgid":true,"Hostid":1000000,"Nsid":0,"Maprange":1000000000}]'
  volatile.last_state.idmap: '[]'
  volatile.last_state.power: RUNNING
  volatile.last_state.ready: "false
  volatile.uuid: ece3e171-36b0-4e2a-a76c-a0757f93db29
  volatile.uuid.generation: ece3e171-36b0-4e2a-a76c-a0757f93db29
devices: {}
  ```
  
# 1. Ad-Hoc Commandos

## 1.1 Einrichten eines Inventory

Als ersten Schritt wurde in dem bereits eingerichteten Verzeichnis eine `inventory.yaml` Datei erstellt:

```bash
it-security@host35:~/ansible$ touch inventory.yaml
```

In der Datei wurde folgender Inhalt eingegeben um die Container nach Kategorien zu ordnen:

```bash
it-security@host35:~/ansible$ vim inventory.yaml
```

```yaml
webserver:
  hosts:
    web1:
      ansible_host: 10.153.36.63
    web2:
      ansible_host: 10.153.36.85
  vars:
    ansible_user: ubuntu
mailserver:
  hosts:
    mail1:
      ansible_host: 10.153.36.135
  vars:
    ansible_user: ubuntu
```

Um das angelegte `inventory` zu überprüfen wurde folgender Befehl ausgeführt:

```bash
it-security@host35:~/ansible$ ansible-inventory -i inventory.yaml --list

{
    "_meta": {
        "hostvars": {
            "mail1": {
                "ansible_host": "10.153.36.63",
                "ansible_user": "ubuntu"
            },
            "web1": {
                "ansible_host": "10.153.36.85",
                "ansible_user": "ubuntu"
            },
            "web2": {
                "ansible_host": "10.153.36.135",
                "ansible_user": "ubuntu"
            }
        }
    },
    "all": {
        "children": [
            "ungrouped",
            "webserver",
            "mailserver"
        ]
    },
    "mailserver": {
        "hosts": [
            "mail1"
        ]
    },
    "webserver": {
        "hosts": [
            "web1",
            "web2"
        ]
    }
}
```

Um zu überprüfen ob die in der `inventory.yaml` hinterlegten Hosts auch mit Ansible erreicht werden können, wurden alle eingetragenen Hosts mit dem `ping` Command getestet: 


```bash
it-security@host35:~/ansible$ ansible all -m ping -i inventory.yaml

web1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
web2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
mail1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

## 1.2 Installieren von Python

> Hier sind einige Dinge unklar:
> - Welche Version von Python soll installiert werden, v2 oder v3?
> - Python 3 ist auf allen Containern mit einem Ubuntu Image bereits vorinstalliert
> - Python 2 kann nicht mehr über die offiziellen Ubuntu Repositories bezogen werden

Auf den Containern wurde mit Ansible und dem folgenden ad-hoc Command `python` installiert:

```bash
it-security@host35:~/ansible$ ansible all -m ansible.builtin.raw -a "sudo apt install python -y" -i inventory.yaml

web1 | FAILED | rc=100 >>
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Package python is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source
However the following packages replace it:
  python-is-python3

E: Package 'python' has no installation candidate
Shared connection to 10.153.36.85 closed.
non-zero return code
web2 | FAILED | rc=100 >>
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Package python is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source
However the following packages replace it:
  python-is-python3

E: Package 'python' has no installation candidate
Shared connection to 10.153.36.135 closed.
non-zero return code
mail1 | FAILED | rc=100 >>
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Package python is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source
However the following packages replace it:
  python-is-python3

E: Package 'python' has no installation candidate
Shared connection to 10.153.36.63 closed.
non-zero return code
```
`python`ist als Package nicht mehr in den offiziellen Repositories von Ubuntu vorhanden.

Beim Versuch `python3` zu installieren erhält man folgenden Output, da `python3` auf Ubuntu 24.04 und Debian 12 bereits vorinstalliert ist:

```bash
it-security@host35:~/ansible$ ansible all -m ansible.builtin.raw -a "sudo apt install python3 -y" -i inventory.yaml
web2 | CHANGED | rc=0 >>
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
python3 is already the newest version (3.12.3-0ubuntu2).
python3 set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Shared connection to 10.153.36.135 closed.

web1 | CHANGED | rc=0 >>
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
python3 is already the newest version (3.12.3-0ubuntu2).
python3 set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Shared connection to 10.153.36.85 closed.

mail1 | CHANGED | rc=0 >>
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
python3 is already the newest version (3.12.3-0ubuntu2).
python3 set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Shared connection to 10.153.36.63 closed.
```


### 1.3 Inhalt der /etc/hostname Datei

Um den Inhalt der `/etc/hostname` Datei von allen Containern über Ansible anzeigen zu lassen, wurde das folgende Command verwendet:

```bash
it-security@host35:~/ansible$ ansible all -m ansible.builtin.command -a "cat /etc/hostname" -i inventory.yaml

mail1 | CHANGED | rc=0 >>
mail1
web1 | CHANGED | rc=0 >>
web1
web2 | CHANGED | rc=0 >>
web2
```

# 2. Playbooks

Als zweiten Aufgabenbereich sollten für die Laborübung Ansible-Playbooks geschrieben und ausgeführt werden, welche einige System hardening Maßnahmen, gemäß den _CIS Ubuntu/Debian Linux Benchmark_ durchführen.

Hierfür sollten folgende Dinge durchgeführt werden:

- 2.2.3 Ensure CUPS is not installed
- 7.1.1 Ensure permissions on /etc/passwd are configured
- zwei weitere frei wählbare Items

Für die zwei frei wählbaren Aufgaben haben wir uns für die folgenden Punkte entschieden:

- Ensure nis client is not installed
- Ensure rsh client is not installed

## 2.1 Erstellen der Playbooks

In den angelegten Directory erstellen wir eine `playbook.yaml` Datei erstellt:

```bash
it-security@host35:~/ansible$ touch playbook.yaml
```

mit folgendem Inhalt:

```yaml
- name: System hardening
  hosts: all
  tasks:
    - name: Ensure CUPS is not installed
      ansible.builtin.apt:
        name: cups
        state: absent
        purge: yes
      become: yes

    - name: Ensure permissions on /etc/passwd are 644
      ansible.builtin.file:
        path: /etc/passwd
        owner: root
        group: root
        mode: '644'
      become: yes
    
    - name: Ensure nis client is not installed
      ansible.builtin.apt:
        name: nis
        state: absent
        purge: yes
      become: yes
    
    - name: Ensure rsh client is not installed
      ansible.builtin.apt:
        name: rsh-client
        state: absent
        purge: yes
      become: yes
```

Die Datei enthält den `name` des Playbooks, in diesem Fall _System hardening_, die `hosts` auf denen die `tasks` des Playbooks ausgeführt werden sollen, hier _all_, sowie die auszuführenden `tasks`.

Ein `task` besteht aus den folgenden Bestandteilen:

- `name`: Name des Tasks
- `<module>`: zu verwendenes Modul
- `module parameter`: Einstellungen des Moduls

Für den Zweck der Laborübung kann das Ansible eigene `apt` und `file` Modul verwendet werden.

Mit dem `apt` Modul kann über die eingebaute Funktionalität sicher gestellt werden, dass Pakete entweder installiert oder nicht installiert sind, abhängig davon können diese Pakete installiert oder deinstalliert werden.

Das `file` Modul kann genutzt werden um `owner`, `group` und `mode` festzulegen.

Alle Features der verwendeten Module können unter den folgenden Links eingesehen werden:

- [ansible.builtin.file](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html)
- [ansible.builtin.apt](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html)

# 2.2 Ausführen des Playbooks

Das in Schritt 2.1 erstellte Playbook kann nun mit folgendem command ausgeführt werden:

```bash
it-security@host35:~/ansible$ ansible-playbook -i inventory.yaml playbook.yaml

PLAY [System hardening] ********************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [mail1]
ok: [web1]
ok: [web2]

TASK [Ensure CUPS is not installed] ********************************************************************************************************************************************************
ok: [mail1]
ok: [web1]
ok: [web2]

TASK [Ensure permissions on /etc/passwd are 644] *******************************************************************************************************************************************
ok: [web2]
ok: [web1]
ok: [mail1]

TASK [Ensure nis client is not installed] **************************************************************************************************************************************************
ok: [web1]
ok: [web2]
ok: [mail1]

TASK [Ensure rsh client is not installed] **************************************************************************************************************************************************
ok: [web1]
ok: [web2]
ok: [mail1]

PLAY RECAP *********************************************************************************************************************************************************************************
mail1                      : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web1                       : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web2                       : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Der Output zeigt:

- Den Namen des gelaufenen Playbooks
- Den _Gathering Facts_ Task. Hier werden Informationen über das Inventory gesammelt, welche in den weiteren Tasks verwendet werden.
- Die gelaufenen Tasks, welche im Playbook definiert wurden.
- Ein Play Recap des gelaufenen Playbooks

Der Status von jedem Task wird für jeden Host auf dem der Task ausgeführt wurde angezeigt. Ein `ok` bedeutet hierbei, dass der Task erfolgreich ausgeführt wurde.

Im _Play Recap_ wird noch einmal zusammengefasst dargestellt, wie viele Tasks auf welchem Host mit den entsprechenden Status durchgeführt wurden. Im Fall des ausgeführten Playbooks, wurden 5 Tasks ausgeführt und diese auf allen im Inventory vorhandenen Hosts mit dem Status `ok` abgeschlossen.

# 3. Roles

In der Roles Aufgabe der Übung sollten die folgenden Unteraufgaben durchgeführt werden:

> Legen Sie ein Verzeichnis mit der empfohlenen Struktur an (siehe VO Slide #21,#22).

> - Erstellen Sie eine Role, welche alle Pakete aktualisiert
> - Erstellen Sie eine Role, welche einige Standardpakete installiert
> - Erstellen Sie eine Role, welche einen Webserver installiert und ein htdocs|www|public_html (abhängig vom verwendetem Webserver) Verzeichnis befüllt2
> - Erstellen Sie eine Role, welche einen SMTP Daemon installiert
> - Erstellen Sie eine Role, welche den SSH Server gemäß den Empfehlungen von Mozilla konfiguriert3
> - Achtung: Ergänzen Sie zuerst UsePAM yes, da Sie sich ansonsten aussperren (könnten)4 .
> - Achtung: Überprüfen Sie die Korrektheit des Pfades zum sftp-server.

> Erstellen Sie ein Playbook, welches diese Roles auf den jeweils geeigneten Servern  ausführt.
Ändern Sie den Content für htdocs und lassen Sie das Playbook noch einmal laufen.

## 3.1 Verzeichnis

Für die Aufgabe wurde folgende Verzeichnis-Struktur angelegt:

![Directory structure](https://i.imgur.com/Mv6rp5B.png)

Die Ordner `.vscode, cert_setup, TREE` können hier ignoriert werden. In den task foldern werden die zu den Rollen gehörigen Task-Files erstellt. Handler enthalten Handler-Funktionen, welche aus Tasks heraus aufgerufen werden können. Templates erhalten Template-Files, welche von Tasks dynamisch mit Werten währen der Task-Ausführung befüllt werden können. Unter `vars` können Variablen und Vaults erstellt werden, deren Inhalt in Tasks währen der ausführung genutzt werden können.


## 3.2 Anlegen der tasks, files, templates & handlers

In den Ordnern wurden die folgenden Dateien angelegt:

Mailserver Handler, main.yaml:

```yaml
- name: restart postfix
  ansible.builtin.service:
    name: postfix
    state: restarted
  become: yes
```

Mailserver Tasks, main.yaml:

```yaml
- name: Install Postfix
  ansible.builtin.apt:
    name: postfix
    state: present
  become: yes

- name: Enable and start Postfix service
  ansible.builtin.service:
    name: postfix
    state: started
    enabled: yes
  become: yes

- name: Configure Postfix
  ansible.builtin.template:
    src: main.cf.j2
    dest: /etc/postfix/main.cf
    owner: root
    group: root
    mode: '644'
  become: yes
  notify: restart postfix
 ```
 
 Mailserver Templates, main.cf.j2:
 
 ```j2
 # See /usr/share/postfix/main.cf.dist for a commented, more complete version

smtpd_banner = $myhostname ESMTP $mail_name
biff = no
append_dot_mydomain = no
readme_directory = no

# TLS parameters
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_use_tls=yes
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

# Basic configuration
myhostname = {{ ansible_hostname }}
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
mydestination = $myhostname, localhost.$mydomain, localhost
relayhost = 
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = loopback-only
inet_protocols = all
 ```
 
 ssh_hardening Handler, main.yaml:
 
 ```yaml
 - name: restart ssh
  ansible.builtin.service:
    name: ssh
    state: restarted
  become: yes
 ```
 
 ssh_hardening Tasks, main.yaml:
 
 ```yaml
 - name: Install OpenSSH server
  ansible.builtin.apt:
    name: openssh-server
    state: present
  become: yes

- name: Locate sftp-server binary
  ansible.builtin.command: which sftp-server
  register: sftp_location
  changed_when: false
  ignore_errors: yes

- name: Find sftp-server alternative path
  ansible.builtin.stat:
    path: /usr/lib/openssh/sftp-server
  register: sftp_alt_path
  when: sftp_location.rc != 0

- name: Configure SSH server (Mozilla recommendations)
  ansible.builtin.template:
    src: sshd_config.j2
    dest: /etc/ssh/sshd_config
    owner: root
    group: root
    mode: '600'
    validate: '/usr/sbin/sshd -t -f %s'
  become: yes
  notify: restart ssh
 ```
 
 ssh_hardening Templates, main.yaml:
 
 ```j2
 # Secure SSH configuration based on Mozilla recommendations
# Important: UsePAM yes is required to prevent lockouts

# Security
Protocol 2
HostKey /etc/ssh/ssh_host_ed25519_key
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com

# Authentication
UsePAM yes
PermitRootLogin no
StrictModes yes
MaxAuthTries 5
MaxSessions 10
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no
X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
PrintMotd no
TCPKeepAlive yes
ClientAliveInterval 300
ClientAliveCountMax 2

# SFTP configuration
Subsystem sftp {% if sftp_location.rc == 0 %}{{ sftp_location.stdout }}{% else %}{{ '/usr/lib/openssh/sftp-server' if sftp_alt_path.stat.exists else '/usr/libexec/openssh/sftp-server' }}{% endif %}
```

standard_packages Tasks, main.yaml:

```yaml
- name: Install standard packages
  ansible.builtin.apt:
    name: "{{ item }}"
    state: present
  loop:
    - vim
    - htop
    - tmux
    - git
    - curl
    - wget
    - unzip
    - net-tools
    - traceroute
    - tree
  become: yes
```

updater Tasks, main.yaml:

```yaml
- name: Update apt cache
  ansible.builtin.apt:
    update_cache: yes
    cache_valid_time: 3600
  become: yes

- name: Upgrade all packages
  ansible.builtin.apt:
    upgrade: dist
  become: yes

- name: Autoremove unused packages
  ansible.builtin.apt:
    autoremove: yes
  become: yes
```

webserver Files, index.html:

```html=
<!DOCTYPE html>
    <html>
    <head>
        <title>Willkommen auf dem Server</title>
    </head>
    <body>
        <h1>Willkommen auf dem Server nach einem Change!</h1>
        <p>Diese Seite wurde mit Ansible bereitgestellt.</p>
    </body>
</html>
```

webserver Handler, main.yaml:

```yaml
- name: restart apache2
  ansible.builtin.service:
    name: apache2
    state: restarted
  become: yes
```

webserver Tasks, main.yaml:

```yaml
- name: Install Apache webserver
  ansible.builtin.apt:
    name: apache2
    state: present
  become: yes

- name: Enable and start Apache service
  ansible.builtin.service:
    name: apache2
    state: started
    enabled: yes
  become: yes

- name: Copy website content
  ansible.builtin.copy:
    src: index.html
    dest: /var/www/html/index.html
    owner: www-data
    group: www-data
    mode: '644'
  become: yes
  notify: restart apache2
```

## 3.3 Anlegen und ausführen des Playbooks

Als nächstes wurde die Datei für ein Playbook angelet mit dem Namen `roles_playbook.yaml`.

```bash
it-security@host35:~/ansible$ touch roles_playbook.yaml
```

mit folgendem Inhalt:

```yaml
- name: Apply updates to all servers
  hosts: all
  roles:
    - updater
    - standard_packages

- name: Configure web servers
  hosts: webserver
  roles:
    - webserver

- name: Configure mail servers
  hosts: mailserver
  roles:
    - mailserver

- name: Secure SSH on all servers
  hosts: all
  roles:
    - ssh_hardening
```

Der erste run war mit Fehlern, da die Gruppen falsch benannt waren (webservers/mailservers anstatt webserver/mailserver):

```bash
it-security@host35:~/ansible$ ansible-playbook -i inventory.yaml roles_playbook.yaml

PLAY [Apply updates to all servers] ********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [mail1]
ok: [web1]
ok: [web2]

TASK [updater : Update apt cache] **********************************************************************************************************************************************************
changed: [web2]
changed: [web1]
changed: [mail1]

TASK [updater : Upgrade all packages] ******************************************************************************************************************************************************
changed: [web1]
changed: [web2]
changed: [mail1]

TASK [updater : Autoremove unused packages] ************************************************************************************************************************************************
ok: [web1]
ok: [web2]
ok: [mail1]

TASK [standard_packages : Install standard packages] ***************************************************************************************************************************************
ok: [web1] => (item=vim)
ok: [web2] => (item=vim)
ok: [mail1] => (item=vim)
ok: [web1] => (item=htop)
ok: [web2] => (item=htop)
ok: [mail1] => (item=htop)
ok: [web1] => (item=tmux)
ok: [web2] => (item=tmux)
ok: [mail1] => (item=tmux)
ok: [web2] => (item=git)
ok: [web1] => (item=git)
ok: [mail1] => (item=git)
ok: [web2] => (item=curl)
ok: [web1] => (item=curl)
ok: [mail1] => (item=curl)
ok: [web2] => (item=wget)
ok: [web1] => (item=wget)
ok: [mail1] => (item=wget)
changed: [mail1] => (item=unzip)
changed: [web1] => (item=unzip)
changed: [web2] => (item=unzip)
changed: [web1] => (item=net-tools)
changed: [web2] => (item=net-tools)
changed: [mail1] => (item=net-tools)
changed: [mail1] => (item=traceroute)
changed: [web1] => (item=traceroute)
changed: [web2] => (item=traceroute)
changed: [web1] => (item=tree)
changed: [web2] => (item=tree)
changed: [mail1] => (item=tree)
[WARNING]: Could not match supplied host pattern, ignoring: webservers

PLAY [Configure web servers] ***************************************************************************************************************************************************************
skipping: no hosts matched
[WARNING]: Could not match supplied host pattern, ignoring: mailservers

PLAY [Configure mail servers] **************************************************************************************************************************************************************
skipping: no hosts matched

PLAY [Secure SSH on all servers] ***********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [mail1]
ok: [web2]
ok: [web1]

TASK [ssh_hardening : Install OpenSSH server] **********************************************************************************************************************************************
ok: [web2]
ok: [web1]
ok: [mail1]

TASK [ssh_hardening : Locate sftp-server binary] *******************************************************************************************************************************************
fatal: [mail1]: FAILED! => {"changed": false, "cmd": ["which", "sftp-server"], "delta": "0:00:00.004981", "end": "2025-03-25 22:18:53.543237", "msg": "non-zero return code", "rc": 1, "start": "2025-03-25 22:18:53.538256", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [web1]: FAILED! => {"changed": false, "cmd": ["which", "sftp-server"], "delta": "0:00:00.006683", "end": "2025-03-25 22:18:53.543940", "msg": "non-zero return code", "rc": 1, "start": "2025-03-25 22:18:53.537257", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [web2]: FAILED! => {"changed": false, "cmd": ["which", "sftp-server"], "delta": "0:00:00.004607", "end": "2025-03-25 22:18:53.541862", "msg": "non-zero return code", "rc": 1, "start": "2025-03-25 22:18:53.537255", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring

TASK [ssh_hardening : Find sftp-server alternative path] ***********************************************************************************************************************************
ok: [web2]
ok: [mail1]
ok: [web1]

TASK [ssh_hardening : Configure SSH server (Mozilla recommendations)] **********************************************************************************************************************
ERROR! The requested handler 'restart ssh' was not found in either the main handlers list nor in the listening handlers list
```

Der zweiter run war mit Fehlern, da der Ordner falsch benannt war (handler anstatt handlers):

```bash
it-security@host35:~/ansible$ ansible-playbook -i inventory.yaml roles_playbook.yaml

PLAY [Apply updates to all servers] ********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [mail1]
ok: [web2]
ok: [web1]

TASK [updater : Update apt cache] **********************************************************************************************************************************************************
ok: [web2]
ok: [web1]
ok: [mail1]

TASK [updater : Upgrade all packages] ******************************************************************************************************************************************************
ok: [web1]
ok: [web2]
ok: [mail1]

TASK [updater : Autoremove unused packages] ************************************************************************************************************************************************
ok: [web2]
ok: [web1]
ok: [mail1]

TASK [standard_packages : Install standard packages] ***************************************************************************************************************************************
ok: [web1] => (item=vim)
ok: [web2] => (item=vim)
ok: [mail1] => (item=vim)
ok: [web1] => (item=htop)
ok: [web2] => (item=htop)
ok: [mail1] => (item=htop)
ok: [web1] => (item=tmux)
ok: [web2] => (item=tmux)
ok: [mail1] => (item=tmux)
ok: [web1] => (item=git)
ok: [web2] => (item=git)
ok: [mail1] => (item=git)
ok: [web1] => (item=curl)
ok: [web2] => (item=curl)
ok: [mail1] => (item=curl)
ok: [web1] => (item=wget)
ok: [web2] => (item=wget)
ok: [mail1] => (item=wget)
ok: [web1] => (item=unzip)
ok: [web2] => (item=unzip)
ok: [mail1] => (item=unzip)
ok: [web1] => (item=net-tools)
ok: [web2] => (item=net-tools)
ok: [mail1] => (item=net-tools)
ok: [web1] => (item=traceroute)
ok: [web2] => (item=traceroute)
ok: [mail1] => (item=traceroute)
ok: [web2] => (item=tree)
ok: [web1] => (item=tree)
ok: [mail1] => (item=tree)

PLAY [Configure web servers] ***************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [web2]
ok: [web1]

TASK [webserver : Install Apache webserver] ************************************************************************************************************************************************
changed: [web2]
changed: [web1]

TASK [webserver : Enable and start Apache service] *****************************************************************************************************************************************
ok: [web1]
ok: [web2]

TASK [webserver : Copy website content] ****************************************************************************************************************************************************
ERROR! The requested handler 'restart apache2' was not found in either the main handlers list nor in the listening handlers list
```

Der dritte run wurde korrekt ausgeführt. Die Fehler im Task `ssh_hardening : Locate sftp-server binary` können ignoriert werden, da der Fehlercode vom Playbook weiter verarbeitet wird:

```bash
it-security@host35:~/ansible$ ansible-playbook -i inventory.yaml roles_playbook.yaml

PLAY [Apply updates to all servers] ********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [web1]
ok: [web2]
ok: [mail1]

TASK [updater : Update apt cache] **********************************************************************************************************************************************************
ok: [web1]
ok: [mail1]
ok: [web2]

TASK [updater : Upgrade all packages] ******************************************************************************************************************************************************
ok: [web1]
ok: [web2]
ok: [mail1]

TASK [updater : Autoremove unused packages] ************************************************************************************************************************************************
ok: [web2]
ok: [mail1]
ok: [web1]

TASK [standard_packages : Install standard packages] ***************************************************************************************************************************************
ok: [web1] => (item=vim)
ok: [web2] => (item=vim)
ok: [mail1] => (item=vim)
ok: [web2] => (item=htop)
ok: [web1] => (item=htop)
ok: [mail1] => (item=htop)
ok: [web2] => (item=tmux)
ok: [web1] => (item=tmux)
ok: [mail1] => (item=tmux)
ok: [web2] => (item=git)
ok: [mail1] => (item=git)
ok: [web1] => (item=git)
ok: [web2] => (item=curl)
ok: [web1] => (item=curl)
ok: [mail1] => (item=curl)
ok: [web2] => (item=wget)
ok: [web1] => (item=wget)
ok: [mail1] => (item=wget)
ok: [web2] => (item=unzip)
ok: [mail1] => (item=unzip)
ok: [web1] => (item=unzip)
ok: [web2] => (item=net-tools)
ok: [mail1] => (item=net-tools)
ok: [web1] => (item=net-tools)
ok: [web2] => (item=traceroute)
ok: [web1] => (item=traceroute)
ok: [mail1] => (item=traceroute)
ok: [mail1] => (item=tree)
ok: [web1] => (item=tree)
ok: [web2] => (item=tree)

PLAY [Configure web servers] ***************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [web1]
ok: [web2]

TASK [webserver : Install Apache webserver] ************************************************************************************************************************************************
ok: [web1]
ok: [web2]

TASK [webserver : Enable and start Apache service] *****************************************************************************************************************************************
ok: [web1]
ok: [web2]

TASK [webserver : Copy website content] ****************************************************************************************************************************************************
ok: [web1]
ok: [web2]

PLAY [Configure mail servers] **************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [mail1]

TASK [mailserver : Install Postfix] ********************************************************************************************************************************************************
changed: [mail1]

TASK [mailserver : Enable and start Postfix service] ***************************************************************************************************************************************
ok: [mail1]

TASK [mailserver : Configure Postfix] ******************************************************************************************************************************************************
changed: [mail1]

RUNNING HANDLER [mailserver : restart postfix] *********************************************************************************************************************************************
changed: [mail1]

PLAY [Secure SSH on all servers] ***********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [web1]
ok: [web2]
ok: [mail1]

TASK [ssh_hardening : Install OpenSSH server] **********************************************************************************************************************************************
ok: [web2]
ok: [web1]
ok: [mail1]

TASK [ssh_hardening : Locate sftp-server binary] *******************************************************************************************************************************************
fatal: [web1]: FAILED! => {"changed": false, "cmd": ["which", "sftp-server"], "delta": "0:00:00.003244", "end": "2025-03-25 22:26:51.799531", "msg": "non-zero return code", "rc": 1, "start": "2025-03-25 22:26:51.796287", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [web2]: FAILED! => {"changed": false, "cmd": ["which", "sftp-server"], "delta": "0:00:00.004887", "end": "2025-03-25 22:26:51.798895", "msg": "non-zero return code", "rc": 1, "start": "2025-03-25 22:26:51.794008", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [mail1]: FAILED! => {"changed": false, "cmd": ["which", "sftp-server"], "delta": "0:00:00.004527", "end": "2025-03-25 22:26:51.799552", "msg": "non-zero return code", "rc": 1, "start": "2025-03-25 22:26:51.795025", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring

TASK [ssh_hardening : Find sftp-server alternative path] ***********************************************************************************************************************************
ok: [web1]
ok: [web2]
ok: [mail1]

TASK [ssh_hardening : Configure SSH server (Mozilla recommendations)] **********************************************************************************************************************
ok: [web1]
ok: [web2]
ok: [mail1]

PLAY RECAP *********************************************************************************************************************************************************************************
mail1                      : ok=15   changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
web1                       : ok=14   changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
web2                       : ok=14   changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
```

Für die Website wurde der folgende Code verwendet:

```html=
<!DOCTYPE html>
    <html>
    <head>
        <title>Willkommen auf dem Server</title>
    </head>
    <body>
        <h1>Willkommen auf dem Server</h1>
        <p>Diese Seite wurde mit Ansible bereitgestellt.</p>
    </body>
</html>
```

Hier kann man sehen, dass die Website mit Apache2 auf _web1_ Host (10.153.36.85) läuft:

![Web1 Host mit Apache2 Webserver](https://i.imgur.com/OuGYwTV.png)

Nun haben wir einen Change im Code gemacht:

```html=
<!DOCTYPE html>
    <html>
    <head>
        <title>Willkommen auf dem Server</title>
    </head>
    <body>
        <h1>Willkommen auf dem Server nach einem Change!</h1>
        <p>Diese Seite wurde mit Ansible bereitgestellt.</p>
    </body>
</html>
```

Und es läuft nach einem Change:

![](https://i.imgur.com/nhAOIPv.png)


# 4. Ansible-Vault

Als nächstes wurde der Ansible-Vault mit dem folgenden Befehl erstellt:

```bash
it-security@host35:~/ansible$ ansible-vault create roles/webserver/vars/vault.yaml
New Vault password:
Confirm New Vault password:
```
Als _Password_ wurde `password` gewählt, hiervon ist in einer Produktionsumgebung abzusehen!

Die `vault.yaml` wurde mit folgendem Inhalt befüllt:

```yaml
mysql_credentials:
  host: "10.153.36.85"
  database: "webapp"
  username: "mysql"
  password: "password"
```

In der `main.yaml` unter /roles/webserver/tasks/ wurden folgende Tasks ergänzt:

```yaml
- name: Include encrypted variables
  ansible.builtin.include_vars:
    file: vars/vault.yaml

- name: Copy database configuration
  ansible.builtin.template:
    src: config.php.j2
    dest: /var/www/html/config.php
    owner: www-data
    group: www-data
    mode: '640'
  become: yes
  notify: restart apache2
```

Es wurde der Vault eingebunden und ein Task erstellt der aus einem Template die `config.php` erstellt, welche die Credentials enthält, die aus dem Vault eingefügt wurden. Das `config.php.j2` Jinja Template sieht wie folgt aus und wurde unter `/roles/webserver/templates/` abgelegt:

```j2
<?php
define('DB_HOST', '{{ mysql_credentials.host }}');
define('DB_NAME', '{{ mysql_credentials.database }}');
define('DB_USER', '{{ mysql_credentials.username }}');
define('DB_PASSWORD', '{{ mysql_credentials.password }}');

$conn = new mysqli(DB_HOST, DB_USER, DB_PASSWORD, DB_NAME);

if ($conn->connect_error) {
    die("Verbindung fehlgeschlagen: " . $conn->connect_error);
}

$config = [
    'app_name' => 'Meine Webanwendung',
    'debug' => false,
    'timezone' => 'Europe/Berlin'
];
?>
```

Wenn das `roles_playbook.yaml` ausgeführt wird, werden aus dem Vault die Credentials extrahiert und in das Template an den entsprechenden Stellen, markiert durch `{{}}`, eingefügt.

```bash
it-security@host35:~/ansible$ ansible-playbook roles_playbook.yaml --ask-vault-pass -i inventory.yaml
Vault password:

PLAY [Apply updates to all servers] ********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [mail1]
ok: [web2]
ok: [web1]

TASK [updater : Update apt cache] **********************************************************************************************************************************************************
ok: [web1]
ok: [web2]
ok: [mail1]

TASK [updater : Upgrade all packages] ******************************************************************************************************************************************************
ok: [web2]
ok: [web1]
ok: [mail1]

TASK [updater : Autoremove unused packages] ************************************************************************************************************************************************
ok: [web1]
ok: [mail1]
ok: [web2]

TASK [standard_packages : Install standard packages] ***************************************************************************************************************************************
ok: [web2] => (item=vim)
ok: [mail1] => (item=vim)
ok: [web1] => (item=vim)
ok: [mail1] => (item=htop)
ok: [web2] => (item=htop)
ok: [web1] => (item=htop)
ok: [web2] => (item=tmux)
ok: [mail1] => (item=tmux)
ok: [web1] => (item=tmux)
ok: [web2] => (item=git)
ok: [mail1] => (item=git)
ok: [web1] => (item=git)
ok: [web2] => (item=curl)
ok: [mail1] => (item=curl)
ok: [web1] => (item=curl)
ok: [web2] => (item=wget)
ok: [mail1] => (item=wget)
ok: [web1] => (item=wget)
ok: [mail1] => (item=unzip)
ok: [web2] => (item=unzip)
ok: [web1] => (item=unzip)
ok: [mail1] => (item=net-tools)
ok: [web2] => (item=net-tools)
ok: [web1] => (item=net-tools)
ok: [mail1] => (item=traceroute)
ok: [web2] => (item=traceroute)
ok: [web1] => (item=traceroute)
ok: [mail1] => (item=tree)
ok: [web2] => (item=tree)
ok: [web1] => (item=tree)

PLAY [Configure web servers] ***************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [web2]
ok: [web1]

TASK [webserver : Install Apache webserver] ************************************************************************************************************************************************
ok: [web1]
ok: [web2]

TASK [webserver : Enable and start Apache service] *****************************************************************************************************************************************
ok: [web1]
ok: [web2]

TASK [webserver : Copy website content] ****************************************************************************************************************************************************
ok: [web1]
ok: [web2]

TASK [webserver : Include encrypted variables] *********************************************************************************************************************************************
ok: [web1]
ok: [web2]

TASK [webserver : Copy database configuration] *********************************************************************************************************************************************
changed: [web1]
changed: [web2]

RUNNING HANDLER [webserver : restart apache2] **********************************************************************************************************************************************
changed: [web2]
changed: [web1]

PLAY [Configure mail servers] **************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [mail1]

TASK [mailserver : Install Postfix] ********************************************************************************************************************************************************
ok: [mail1]

TASK [mailserver : Enable and start Postfix service] ***************************************************************************************************************************************
ok: [mail1]

TASK [mailserver : Configure Postfix] ******************************************************************************************************************************************************
ok: [mail1]

PLAY [Secure SSH on all servers] ***********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [mail1]
ok: [web1]
ok: [web2]

TASK [ssh_hardening : Install OpenSSH server] **********************************************************************************************************************************************
ok: [web2]
ok: [web1]
ok: [mail1]

TASK [ssh_hardening : Locate sftp-server binary] *******************************************************************************************************************************************
fatal: [web1]: FAILED! => {"changed": false, "cmd": ["which", "sftp-server"], "delta": "0:00:00.003833", "end": "2025-03-26 14:15:42.801891", "msg": "non-zero return code", "rc": 1, "start": "2025-03-26 14:15:42.798058", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [web2]: FAILED! => {"changed": false, "cmd": ["which", "sftp-server"], "delta": "0:00:00.004562", "end": "2025-03-26 14:15:42.801340", "msg": "non-zero return code", "rc": 1, "start": "2025-03-26 14:15:42.796778", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [mail1]: FAILED! => {"changed": false, "cmd": ["which", "sftp-server"], "delta": "0:00:00.003934", "end": "2025-03-26 14:15:42.803845", "msg": "non-zero return code", "rc": 1, "start": "2025-03-26 14:15:42.799911", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring

TASK [ssh_hardening : Find sftp-server alternative path] ***********************************************************************************************************************************
ok: [web1]
ok: [web2]
ok: [mail1]

TASK [ssh_hardening : Configure SSH server (Mozilla recommendations)] **********************************************************************************************************************
ok: [web1]
ok: [web2]
ok: [mail1]

PLAY RECAP *********************************************************************************************************************************************************************************
mail1                      : ok=14   changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
web1                       : ok=17   changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
web2                       : ok=17   changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
```

Um zu überprüfen ob das Playbook, die Credentials richtig eingefügt hat kann die `config.php` über den Webserver aufgerufen werden, wir erhalten folgende PHP-Datei vom Server:

```php
<?php
define('DB_HOST', '10.153.36.85');
define('DB_NAME', 'webapp');
define('DB_USER', 'mysql');
define('DB_PASSWORD', 'password');

$conn = new mysqli(DB_HOST, DB_USER, DB_PASSWORD, DB_NAME);

if ($conn->connect_error) {
    die("Verbindung fehlgeschlagen: " . $conn->connect_error);
}

$config = [
    'app_name' => 'Meine Webanwendung',
    'debug' => false,
    'timezone' => 'Europe/Berlin'
];
?>
```

# 5. Bonus Aufgabe

In der Bonus Aufgabe sollen für die Webserver SSL-Verschlüsselung eingerichtet werden, wobei das Zertifikat wie auch der Private Key aus dem Ansible Vault kommen sollen.

## 5.1 Generieren von Self-Signed Certificate

Über die folgenden Commands wurde ein Self-Signed Certificate sowie ein Private-Key angelegt:

```bash
it-security@host35:~/ansible$ openssl genrsa -out domain.key 2048
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
```

```bash
it-security@host35:~/ansible$ openssl req -key domain.key -new -out domain.csr
```

## 5.2 Erweitern des webserver Tasks und anwenden des Playbooks

Das `main.yaml` File unter webserver/tasks wurde wie folgt angepasst:

```yaml
- name: Install Apache webserver
  ansible.builtin.apt:
    name: apache2
    state: present
  become: yes

- name: Enable and start Apache service
  ansible.builtin.service:
    name: apache2
    state: started
    enabled: yes
  become: yes

- name: Copy website content
  ansible.builtin.copy:
    src: index.html
    dest: /var/www/html/index.html
    owner: www-data
    group: www-data
    mode: '644'
  become: yes
  notify: restart apache2

- name: Include encrypted variables
  ansible.builtin.include_vars:
    file: vars/vault.yaml

- name: Copy database configuration
  ansible.builtin.template:
    src: config.php.j2
    dest: /var/www/html/config.php
    owner: www-data
    group: www-data
    mode: '640'
  become: yes
  notify: restart apache2

- name: Enable SSL module
  ansible.builtin.command: a2enmod ssl
  args:
    creates: /etc/apache2/mods-enabled/ssl.load
  become: yes
  notify: restart apache2

- name: Create SSL directory
  ansible.builtin.file:
    path: /etc/apache2/ssl
    state: directory
    owner: root
    group: root
    mode: '755'
  become: yes

- name: Copy SSL certificate
  ansible.builtin.copy:
    content: "{{ ssl.certificate }}"
    dest: /etc/apache2/ssl/server.crt
    owner: root
    group: root
    mode: '644'
  become: yes
  notify: restart apache2

- name: Copy SSL private key
  ansible.builtin.copy:
    content: "{{ ssl.private_key }}"
    dest: /etc/apache2/ssl/server.key
    owner: root
    group: root
    mode: '600'
  become: yes
  notify: restart apache2

- name: Configure SSL VirtualHost
  ansible.builtin.template:
    src: ssl-vhost.conf.j2
    dest: /etc/apache2/sites-available/default-ssl.conf
    owner: root
    group: root
    mode: '644'
  become: yes
  notify: restart apache2

- name: Enable SSL site
  ansible.builtin.command: a2ensite default-ssl
  args:
    creates: /etc/apache2/sites-enabled/default-ssl.conf
  become: yes
  notify: restart apache2
```
 
Der Ansible Vault wurde angepasst und das Zertifikat sowie der private Key hinzugefügt:
 
```bash
it-security@host35:~/ansible$ ansible-vault edit roles/webserver/vars/vault.yaml
Vault password:
```
 
 ```vault.yaml
mysql_credentials:
  host: "10.153.36.85"
  database: "webapp"
  username: "mysql"
  password: "password"
ssl:
  certificate: |
    -----BEGIN CERTIFICATE-----
    MIIDhTCCAm0CFB2seP0hxKGtXzAG/0r4hkXsE2hsMA0GCSqGSIb3DQEBCwUAMH8x
    CzAJBgNVBAYTAkFUMQ8wDQYDVQQIDAZWaWVubmExDzANBgNVBAcMBlZpZW5uYTEX
    MBUGA1UECgwORkggQ2FtcHVzIFdpZW4xDDAKBgNVBAsMA0lUUzEnMCUGCSqGSIb3
    DQEJARYYdGVzdEBmaC1jYW1wdXN3aWVuLmFjLmF0MB4XDTI1MDMyNjE2NDA1M1oX
    DTI2MDMyNjE2NDA1M1owfzELMAkGA1UEBhMCQVQxDzANBgNVBAgMBlZpZW5uYTEP
    MA0GA1UEBwwGVmllbm5hMRcwFQYDVQQKDA5GSCBDYW1wdXMgV2llbjEMMAoGA1UE
    CwwDSVRTMScwJQYJKoZIhvcNAQkBFhh0ZXN0QGZoLWNhbXB1c3dpZW4uYWMuYXQw
    ggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDfH2PBY2Yk8Los3kGgFyjx
    A59P9cERqA7iRGPI3Ouwn1mCQY6pH1futu3RkqEzMYH59CWZPDy/S444SJJA6ynr
    sUOhhvRSigRseqWM0P71oWb6h5Z7WLmOq1AMind2r2O2eeErhNM46b+A68cO/zre
    Z5+MA1DCqbTkP5uDZPjzUtJZrLZTq/Xpb81DaRDz3QHl4S6r8m8vfQpPeyAn6rw1
    DWntbkolyaHIekuqZprlpBBOYI5hTJ4r3cqaItGKrEWUcQCDoVyY64KFp58YvRYI
    mbC0PkeWru+10lp1EdVrfVWGW5K2RBJX+v6UL4v5ieBfhKU8hFO7S/0mF70rOMtn
    AgMBAAEwDQYJKoZIhvcNAQELBQADggEBACiuGg9bMcwlfzgPCIo3dxUilLH4jdls
    cHpZ8j5xMnmwPd+DvB6p2DwBdVw8wTPNPXSYVo6Ew4DQzH7nrKW6SyTZPsq+LVQG
    pc2ANvScFvaaBrnlY5ya0ZkozCNY4dH7zCmumM9FVsm9UlV7ccNfCkB+Y64XbYZZ
    GIdOdNEXlOQ4vvm7iGI9/KdPZKVBb4ezYgQBQuAnGn/faCQVm686Nzrg6RIVQ4T0
    2XoACdoICsdnqpXnAxXU7iTDT5lNO46eNX6Myuy9OmoXbAdxTdQHmsXyz6hLCqM9
    5WuhVWm9LbMaeaqMgZaFhGaN6xBPEX3mpr+/EOpUljkrfmk5f0zs0/Y=
    -----END CERTIFICATE-----
  private_key: |
    -----BEGIN PRIVATE KEY-----
    MIIEvAIBADANBgkqhkiG9w0BAQEFAASCBKYwggSiAgEAAoIBAQDfH2PBY2Yk8Los
    3kGgFyjxA59P9cERqA7iRGPI3Ouwn1mCQY6pH1futu3RkqEzMYH59CWZPDy/S444
    SJJA6ynrsUOhhvRSigRseqWM0P71oWb6h5Z7WLmOq1AMind2r2O2eeErhNM46b+A
    68cO/zreZ5+MA1DCqbTkP5uDZPjzUtJZrLZTq/Xpb81DaRDz3QHl4S6r8m8vfQpP
    eyAn6rw1DWntbkolyaHIekuqZprlpBBOYI5hTJ4r3cqaItGKrEWUcQCDoVyY64KF
    p58YvRYImbC0PkeWru+10lp1EdVrfVWGW5K2RBJX+v6UL4v5ieBfhKU8hFO7S/0m
    F70rOMtnAgMBAAECggEAZPklIV8U2jHQ5z4777GbdxrfDYYXdiCaf75YdA26Ycdz
    b/WwFIxZNHA3hui3J95HRnE9VLAEg8OzHHiHK3bhFUc25pIW3oWUQ+1rHyNzxoBh
    BI64xKBd9RlFFC/TqXPtCab1hkbJeg+aeUL6ZiiOIRk/BFN5yGaZtNOuUpOu8BoZ
    3KEyoj2TdJaLGqvw1ghFcEHFJaraKE7u4mZoPnvgKa0ZsIf+ZrCdkO/7HiZ7D5wy
    LEyzicW+ztUgR1zVcoSQqjJKZWPBJdrjanQDA4sAtbx1k4ubWlGDGEn0zexNvDV2
    B+EnKN+F4jTcLDc/0tBms80+HCU1EQuLIRs6ax3QwQKBgQD+BDL9sJjyC8ZXywFc
    kZjpIu6zKS7EygKwEo9O+WzkmpVZlxFXoxCIvsqtCtWmoEwoiekca9mBTcV57X4M
    uSB7SHiFP+7qBntHQgry/m/wFu7WzADWTis3yVMYGNYX6svQsqu0Qcu5zhQNgfMJ
    Gta99bi+60mgOxWdqWId+tYrQQKBgQDg3W5eUQvli8YpXAPx6PlWujo82a4cFx6d
    Qmdtyi/pHuoGBpnuGSrMrZLXZ7OkiSjZCIprNYhOk4KOXDI85wniGtBpsXsfgCtZ
    XSP7OmFpuupu4ByeEaPuqsV8KsX+0EMVkBHhXROwz90FTZMtJQ3L0t+lK91eyuRX
    V4nrfUGUpwKBgAfnK4r3DpshQKeEmmM96GsAejq6wki/HDxMJ4dGfVvTb2gdKh32
    5dHLVdTybFlFzXjJaaZHuLmsKMO1XuGYdOlBBPboWU+Qqg86f8q1ndfzbrTiHfiM
    8A1JRzuNskOGO5cfp/Xwwhziy37sFxqmah344imWenDwxahlF7dlEXrBAoGAQbu5
    Rcz/Y0zm0rRZWuA180phN+SSZxfDFUmSHqAaPVWSJ8zKNrYYstiSsnFYLGMEE93R
    SOHPRNkOMzt0XCVV2EjqX9tIGL9I7Mizef9o9pVzbpIJC5QwjdZSAKgoVQLgeW8y
    KuUgmnFJNQGmYm4QiGLOieQ2xD0fXwDyCmsFPsUCgYB6wFCJs7ShxHXUOUAkgsRF
    BUe14lwMd3bBHGVvCFe6rB9HTtZuP1lhzBOXbq5pv1rrInu4xnmQ0t5xSVadXe5K
    pF1hXTTm+mE841O/oLmp7+7wTukA3uXUXHAJJ2qkMSos9ZhIQ/55EnJgw9jFKqjk
    HP9l12yGM8y1Vi4c82nCcA==
    -----END PRIVATE KEY-----
```

Anschließend wurde noch eine `ssl-vhost.conf.j2`, in dem Ordner /roles/webserver/templates/, Datei erstellt. Hier wurde folgender Inhalt eingefügt:

```conf
<IfModule mod_ssl.c>
    <VirtualHost _default_:443>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        SSLEngine on
        SSLCertificateFile /etc/apache2/ssl/server.crt
        SSLCertificateKeyFile /etc/apache2/ssl/server.key

        <FilesMatch "\.(cgi|shtml|phtml|php)$">
            SSLOptions +StdEnvVars
        </FilesMatch>
        
        <Directory /usr/lib/cgi-bin>
            SSLOptions +StdEnvVars
        </Directory>
    </VirtualHost>
</IfModule>
```
Abschließend wurde noch das `roles_playbook.yaml`ausgeführt:

```bash
it-security@host35:~/ansible$  ansible-playbook -i inventory.yaml roles_playbook.yaml --ask-vault-pass
Vault password:

PLAY [Apply updates to all servers] ********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [mail1]
ok: [web1]
ok: [web2]

TASK [updater : Update apt cache] **********************************************************************************************************************************************************
ok: [web2]
ok: [mail1]
ok: [web1]

TASK [updater : Upgrade all packages] ******************************************************************************************************************************************************
ok: [mail1]
ok: [web2]
ok: [web1]

TASK [updater : Autoremove unused packages] ************************************************************************************************************************************************
ok: [mail1]
ok: [web2]
ok: [web1]

TASK [standard_packages : Install standard packages] ***************************************************************************************************************************************
ok: [mail1] => (item=vim)
ok: [web2] => (item=vim)
ok: [web1] => (item=vim)
ok: [mail1] => (item=htop)
ok: [web2] => (item=htop)
ok: [web1] => (item=htop)
ok: [mail1] => (item=tmux)
ok: [web2] => (item=tmux)
ok: [web1] => (item=tmux)
ok: [mail1] => (item=git)
ok: [web2] => (item=git)
ok: [web1] => (item=git)
ok: [mail1] => (item=curl)
ok: [web2] => (item=curl)
ok: [web1] => (item=curl)
ok: [mail1] => (item=wget)
ok: [web2] => (item=wget)
ok: [web1] => (item=wget)
ok: [mail1] => (item=unzip)
ok: [web2] => (item=unzip)
ok: [web1] => (item=unzip)
ok: [mail1] => (item=net-tools)
ok: [web2] => (item=net-tools)
ok: [web1] => (item=net-tools)
ok: [mail1] => (item=traceroute)
ok: [web2] => (item=traceroute)
ok: [web1] => (item=traceroute)
ok: [mail1] => (item=tree)
ok: [web2] => (item=tree)
ok: [web1] => (item=tree)

PLAY [Configure web servers] ***************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [web1]
ok: [web2]

TASK [webserver : Install Apache webserver] ************************************************************************************************************************************************
ok: [web2]
ok: [web1]

TASK [webserver : Enable and start Apache service] *****************************************************************************************************************************************
ok: [web2]
ok: [web1]

TASK [webserver : Copy website content] ****************************************************************************************************************************************************
ok: [web1]
ok: [web2]

TASK [webserver : Include encrypted variables] *********************************************************************************************************************************************
ok: [web1]
ok: [web2]

TASK [webserver : Copy database configuration] *********************************************************************************************************************************************
ok: [web2]
ok: [web1]

TASK [webserver : Enable SSL module] *******************************************************************************************************************************************************
changed: [web1]
changed: [web2]

TASK [webserver : Create SSL directory] ****************************************************************************************************************************************************
changed: [web1]
changed: [web2]

TASK [webserver : Copy SSL certificate] ****************************************************************************************************************************************************
changed: [web2]
changed: [web1]

TASK [webserver : Copy SSL private key] ****************************************************************************************************************************************************
changed: [web1]
changed: [web2]

TASK [webserver : Configure SSL VirtualHost] ***********************************************************************************************************************************************
changed: [web1]
changed: [web2]

TASK [webserver : Enable SSL site] *********************************************************************************************************************************************************
changed: [web1]
changed: [web2]

RUNNING HANDLER [webserver : restart apache2] **********************************************************************************************************************************************
changed: [web2]
changed: [web1]

PLAY [Configure mail servers] **************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [mail1]

TASK [mailserver : Install Postfix] ********************************************************************************************************************************************************
ok: [mail1]

TASK [mailserver : Enable and start Postfix service] ***************************************************************************************************************************************
ok: [mail1]

TASK [mailserver : Configure Postfix] ******************************************************************************************************************************************************
ok: [mail1]

PLAY [Secure SSH on all servers] ***********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [web1]
ok: [web2]
ok: [mail1]

TASK [ssh_hardening : Install OpenSSH server] **********************************************************************************************************************************************
ok: [mail1]
ok: [web2]
ok: [web1]

TASK [ssh_hardening : Locate sftp-server binary] *******************************************************************************************************************************************
fatal: [web1]: FAILED! => {"changed": false, "cmd": ["which", "sftp-server"], "delta": "0:00:00.003985", "end": "2025-03-26 17:01:29.163278", "msg": "non-zero return code", "rc": 1, "start": "2025-03-26 17:01:29.159293", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [web2]: FAILED! => {"changed": false, "cmd": ["which", "sftp-server"], "delta": "0:00:00.003672", "end": "2025-03-26 17:01:29.168355", "msg": "non-zero return code", "rc": 1, "start": "2025-03-26 17:01:29.164683", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [mail1]: FAILED! => {"changed": false, "cmd": ["which", "sftp-server"], "delta": "0:00:00.004502", "end": "2025-03-26 17:01:29.181948", "msg": "non-zero return code", "rc": 1, "start": "2025-03-26 17:01:29.177446", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring

TASK [ssh_hardening : Find sftp-server alternative path] ***********************************************************************************************************************************
ok: [web1]
ok: [web2]
ok: [mail1]

TASK [ssh_hardening : Configure SSH server (Mozilla recommendations)] **********************************************************************************************************************
ok: [web1]
ok: [web2]
ok: [mail1]

PLAY RECAP *********************************************************************************************************************************************************************************
mail1                      : ok=14   changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
web1                       : ok=23   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
web2                       : ok=23   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
```

Anhand des Outputs können wir sehen, dass unsere Tasks zur Konfiguration von SSL mit dem Status OK ausgeführt wurden.

Über den Browser können wir unsere Website mit TLS/SSL-Encryption aufrufen und uns das per Ansible hinzugefügte Zertifikat anzeigen lassen:

![](https://i.imgur.com/PGhQ0hc.png)
