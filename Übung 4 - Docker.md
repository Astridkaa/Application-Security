# Übung 4 - Docker 

### Datum:

08.04.2025

### Gruppenmitglieder: 
Philip Magnus
Lorenzo Haidinger
Astrid Kuzma-Kuzniarski

## 0 Aufgabenstellung

Diese Laborübung beschäftigt sich mit dem praktischen Einsatz von Docker zur sicheren Containerverwaltung über die Kommandozeile – einschließlich Containererstellung, Netzwerk- und Rechtekonfiguration, dem Aufbau eines Webservers sowie der Analyse von Sicherheitsaspekten mittels Docker Security Bench.

Alle Schritte wurden auf einem Ubuntu 24.04 LTS Hostsystem ausgeführt.

## 1 System einrichten

Mit den folgenden Schritten wurde Docker installiert. Die Systemrepositories wurden geupdated und einige benötigte Pakete installiert. Anschließend wurde das Docker repository hinzugefügt und Docker-CE installiert.

```bash
it-security@host35:~$ sudo apt update 
Hit:1 http://at.archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://at.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:3 http://at.archive.ubuntu.com/ubuntu noble-backports InRelease        
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease           
Reading package lists... Done                        
Building dependency tree... Done
Reading state information... Done
1 package can be upgraded. Run 'apt list --upgradable' to see it.
it-security@host35:~$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ca-certificates is already the newest version (20240203).
ca-certificates set to manually installed.
software-properties-common is already the newest version (0.99.49.1).
software-properties-common set to manually installed.
The following packages were automatically installed and are no longer required:
  libllvm17t64 mailcap python3-netifaces
Use 'sudo apt autoremove' to remove them.
The following NEW packages will be installed:
  apt-transport-https curl
0 upgraded, 2 newly installed, 0 to remove and 1 not upgraded.
Need to get 230 kB of archives.
After this operation, 569 kB of additional disk space will be used.
Do you want to continue? [Y/n] 
Get:1 http://at.archive.ubuntu.com/ubuntu noble/universe amd64 apt-transport-https all 2.7.14build2 [3.974 B]
Get:2 http://at.archive.ubuntu.com/ubuntu noble-updates/main amd64 curl amd64 8.5.0-2ubuntu10.6 [226 kB]
Fetched 230 kB in 0s (1.380 kB/s)
Selecting previously unselected package apt-transport-https.
(Reading database ... 219773 files and directories currently installed.)
Preparing to unpack .../apt-transport-https_2.7.14build2_all.deb ...
Unpacking apt-transport-https (2.7.14build2) ...
Selecting previously unselected package curl.
Preparing to unpack .../curl_8.5.0-2ubuntu10.6_amd64.deb ...
Unpacking curl (8.5.0-2ubuntu10.6) ...
Setting up apt-transport-https (2.7.14build2) ...
Setting up curl (8.5.0-2ubuntu10.6) ...
Processing triggers for man-db (2.12.0-4build2) ...
```

```bash
it-security@host35:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
it-security@host35:~$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
it-security@host35:~$ 
```
Hinzufügen unseres Users zur docker group, damit das docker command ohne root-Rechte genutzt werden kann.

```bash
it-security@host35:~$ sudo usermod -aG docker $USER
```

Aktivieren der Gruppe, damit wir das Docker-Command ohne sudo Rechte ausführen können. Alternativ wäre auch ein neuer Login in das System möglich.

```bash
it-security@host35:~$ newgrp docker
```

Um zu testen ob die installation funktioniert hat führen wir das Docker Hello-World image aus.

```bash
it-security@host35:~$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
e6590344b1a5: Pull complete 
Digest: sha256:7e1a4e2d11e2ac7a8c3f768d4166c2defeb09d2a750b010412b6ea13de1efb19
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

Hier kann man sehen, das das Image erfolgreich in einem Container ausgeführt wurde und die installation erfolgreich war.

## 2 Informationen

- _Welche Netzwerke sind standardmäßig konfiguriert, und welche Einstellungen haben diese?_
- _Welches Storage Backend wird verwendet? Wo werden die Images gespeichert?_
- _Welche Security-Optionen sind auf dem System aktiv?_

### 2.1 Welche Netzwerke sind standardmäßig vorhanden?

Die folgenden Netzwerke sind standardmäßig vorhanden:

```bash
it-security@host35:~$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
22e271165c9f   bridge    bridge    local
ee978e9487a5   host      host      local
866dbe581bb1   none      null      local
```

Hier sehen wir, dass folgende Netzwerke standardgemäß konfiguriert sind: Bridge - Default-Netz, in dem Container kommunizieren können. 
Host - Container nutzen das Host-Netzwerk direkt
none - Kein Netzwerkzugriff

Mit dem `docker network inspect` Command können wir uns die Details der einzelnen Netzwerke anzeigen lassen.

```bash
it-security@host35:~$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "22e271165c9fac4cde578693885ebb2c0a0682d2376b3a8599e1da0313ce00e3",
        "Created": "2025-04-08T17:39:31.954417959+02:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

```bash
it-security@host35:~$ docker network inspect host
[
    {
        "Name": "host",
        "Id": "ee978e9487a5e9b8d60e87ae6d71660b183572db5e1f3dbbe40879cdaae4b34f",
        "Created": "2025-04-08T17:39:31.94914467+02:00",
        "Scope": "local",
        "Driver": "host",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": null
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

```bash
it-security@host35:~$ docker network inspect none
[
    {
        "Name": "none",
        "Id": "866dbe581bb1dea6a12fc3ccbe53b88d99ac609a5eb92aaffbeb38493c6391c5",
        "Created": "2025-04-08T17:39:31.941708559+02:00",
        "Scope": "local",
        "Driver": "null",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": null
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

### 2.2 Welches Storage Backend wird verwendet? Wo werden die Images gespeichert?

```bash
it-security@host35:~$ docker info | grep -A 1000 "Storage Driver"
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Using metacopy: false
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 05044ec0a9a75232cad458027ca83437aae3f4da
 runc version: v1.2.5-0-g59923ef
 init version: de40ad0
 Security Options:
  apparmor
  seccomp
   Profile: builtin
  cgroupns
 Kernel Version: 6.8.0-53-generic
 Operating System: Ubuntu 24.04.2 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 8
 Total Memory: 15.42GiB
 Name: host35
 ID: c4736354-f232-41ba-85cb-54c4067476c8
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  ::1/128
  127.0.0.0/8
 Live Restore Enabled: false
```

Als Storage Backend wird overlay2 als Standard verwendet.
Images und Layer werden unter /var/lib/docker gespeichert. 


### 2.3 Welche Security-Optionen sind auf dem System aktiv?

```bash
it-security@host35:~$ docker info | grep -A 1000 Security
 Security Options:
  apparmor
  seccomp
   Profile: builtin
  cgroupns
 Kernel Version: 6.8.0-53-generic
 Operating System: Ubuntu 24.04.2 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 8
 Total Memory: 15.42GiB
 Name: host35
 ID: c4736354-f232-41ba-85cb-54c4067476c8
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  ::1/128
  127.0.0.0/8
 Live Restore Enabled: false
```

Folgende Security Optionen sind aktiv: 
- AppArmor
- Seccomp (Profile:builtin)
- cgrouns

Diese Mechanismen helfen bei der Isolierung und Rechtebeschränkung von Containern.

## 3 Container

### 3.1 Zeigen Sie den Zustand des Containers während die Shell geöffnet ist und nach dem Schließen.

```bash
it-security@host35:~$ docker pull ubuntu:latest
latest: Pulling from library/ubuntu
5a7813e071bf: Pull complete 
Digest: sha256:72297848456d5d37d1262630108ab308d3e9ec7ed1c3286a32fe09856619a782
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

```bash
it-security@host35:~$ docker run -it --name test-container ubuntu:latest /bin/bash
root@55d2ba5fafb3:/# 
```

In zweitem Terminal:

```bash
it-security@host35:~$ docker ps
CONTAINER ID   IMAGE           COMMAND       CREATED          STATUS          PORTS     NAMES
55d2ba5fafb3   ubuntu:latest   "/bin/bash"   18 seconds ago   Up 17 seconds             test-container
```
Nach dem schließen:

```bash
it-security@host35:~$ docker ps -a
CONTAINER ID   IMAGE           COMMAND       CREATED              STATUS                        PORTS     NAMES
55d2ba5fafb3   ubuntu:latest   "/bin/bash"   About a minute ago   Exited (130) 10 seconds ago             test-container
ee34bea5345b   hello-world     "/hello"      14 minutes ago       Exited (0) 14 minutes ago               objective_lehmann
```

Zustand des Containers: 
Während die Shell offen ist: "docker ps" zeigt aktiven Container an:
Container ID: 55d2ba5fafb3, Names: test-container
Nach Exit "docker ps -a" zeigt er dann Container mit dem Status Exited (mit genauer Zeitinformation):
Container ID: 55d2ba5fafb3, Name: test-container (10 seconds ago)
Container ID: ee34bea5345b, Name : objective_lehmann (14 minutes ago)



### 3.2 Unter welchem User läuft der Prozess auf dem Host?

```bash
it-security@host35:~$ ps aux | grep docker
root       47619  0.2  0.5 2562624 84836 ?       Ssl  17:39   0:03 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
root       48236  0.0  0.0   9088  2688 pts/0    S    17:42   0:00 newgrp docker
root       49345  0.0  0.0   9088  2688 pts/1    S    17:56   0:00 newgrp docker
it-secu+   50009  0.2  0.1 1995884 28288 pts/0   Sl+  18:02   0:00 docker run -it --name test-container ubuntu:latest /bin/bash
it-secu+   50085  0.0  0.0   9632  2432 pts/1    S+   18:02   0:00 grep --color=auto docker
```

Die Containerprozesse laufen meist unter dem User, der den Docker Deamon aufruft(root). Der Container selbst kann aber z.B. als "nobody" laufen (je nach Image)
In diesem Fall sehen wir, dass der User auf dem Host der Prozess läuft: "root" ist. 

### 3.3 Zeigen Sie die gesammelten Einstellungen des Containers an (Netzwerk, Storage, Umgebungsvariable, Volumes, ... ).

Um die komplette Konfiguration eines laufenden oder gestoppten Containers einzusehen, nutzt man folgenden Befehl:

```bash
it-security@host35:~$ docker inspect test-container
[
    {
        "Id": "55d2ba5fafb357d80486bd58dfcdfdbda5d8ea2141a444bb2d21cf604c88c7d8",
        "Created": "2025-04-08T15:56:12.508223975Z",
        "Path": "/bin/bash",
        "Args": [],
        "State": {
            "Status": "exited",
            "Running": false,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 0,
            "ExitCode": 130,
            "Error": "",
            "StartedAt": "2025-04-08T16:00:48.723286071Z",
            "FinishedAt": "2025-04-08T16:02:13.636254636Z"
        },
        "Image": "sha256:a04dc4851cbcbb42b54d1f52a41f5f9eca6a5fd03748c3f6eb2cbeb238ca99bd",
        "ResolvConfPath": "/var/lib/docker/containers/55d2ba5fafb357d80486bd58dfcdfdbda5d8ea2141a444bb2d21cf604c88c7d8/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/55d2ba5fafb357d80486bd58dfcdfdbda5d8ea2141a444bb2d21cf604c88c7d8/hostname",
        "HostsPath": "/var/lib/docker/containers/55d2ba5fafb357d80486bd58dfcdfdbda5d8ea2141a444bb2d21cf604c88c7d8/hosts",
        "LogPath": "/var/lib/docker/containers/55d2ba5fafb357d80486bd58dfcdfdbda5d8ea2141a444bb2d21cf604c88c7d8/55d2ba5fafb357d80486bd58dfcdfdbda5d8ea2141a444bb2d21cf604c88c7d8-json.log",
        "Name": "/test-container",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "bridge",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "ConsoleSize": [
                29,
                105
            ],
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": [],
            "BlkioDeviceWriteBps": [],
            "BlkioDeviceReadIOps": [],
            "BlkioDeviceWriteIOps": [],
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": null,
            "PidsLimit": null,
            "Ulimits": [],
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/interrupts",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware",
                "/sys/devices/virtual/powercap",
                "/sys/devices/system/cpu/cpu0/thermal_throttle",
                "/sys/devices/system/cpu/cpu1/thermal_throttle",
                "/sys/devices/system/cpu/cpu2/thermal_throttle",
                "/sys/devices/system/cpu/cpu3/thermal_throttle",
                "/sys/devices/system/cpu/cpu4/thermal_throttle",
                "/sys/devices/system/cpu/cpu5/thermal_throttle",
                "/sys/devices/system/cpu/cpu6/thermal_throttle",
                "/sys/devices/system/cpu/cpu7/thermal_throttle"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "ID": "55d2ba5fafb357d80486bd58dfcdfdbda5d8ea2141a444bb2d21cf604c88c7d8",
                "LowerDir": "/var/lib/docker/overlay2/7afb259291631a7af52b97e5def22836bbfe1c3b82eb81ceccc5af76c8739a09-init/diff:/var/lib/docker/overlay2/b7b604d29748dfb7ea4b4c897c9a49da04fbe5f42770657c3b493cd0b55a8b47/diff",
                "MergedDir": "/var/lib/docker/overlay2/7afb259291631a7af52b97e5def22836bbfe1c3b82eb81ceccc5af76c8739a09/merged",
                "UpperDir": "/var/lib/docker/overlay2/7afb259291631a7af52b97e5def22836bbfe1c3b82eb81ceccc5af76c8739a09/diff",
                "WorkDir": "/var/lib/docker/overlay2/7afb259291631a7af52b97e5def22836bbfe1c3b82eb81ceccc5af76c8739a09/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "55d2ba5fafb3",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "Image": "ubuntu:latest",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.opencontainers.image.ref.name": "ubuntu",
                "org.opencontainers.image.version": "24.04"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "",
            "SandboxKey": "",
            "Ports": {},
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "",
            "Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "",
            "IPPrefixLen": 0,
            "IPv6Gateway": "",
            "MacAddress": "",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "MacAddress": "",
                    "DriverOpts": null,
                    "GwPriority": 0,
                    "NetworkID": "22e271165c9fac4cde578693885ebb2c0a0682d2376b3a8599e1da0313ce00e3",
                    "EndpointID": "",
                    "Gateway": "",
                    "IPAddress": "",
                    "IPPrefixLen": 0,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DNSNames": null
                }
            }
        }
    }
]
```

Dieser Befehl liefert ein umfangreiches JSON, das in folgende Bereiche unterteilt ist: 
- Config:
    - Hostname, User, Cmd
    Hier sieht man mit welchem Benutzer der Container standardmäßig gestartet wurde(root), welches Standard-Command ausgeführt wird (/bin/bash bei interaktiven Shells), und welches Image zugrunde liegt. 
    - Env
    Enthält alle Umgebungsvariablen des Containers. Diese sind sicherheitsrelevant, wenn z.B. Zugangsdaten als ENV gesetzt wurden. 
     - Volumes 
     Zeigt an, ob der Container Volumes nutzt, also Verzeichnusse vom Host oder persistente Speicher. Wichtig, um unbeabsichtigen Hostzugriff zu vermeiden. 
     
- HostConfig
    - Privileged, CapDrop, CapAdd
    Ob der Container im privilegierten Modus läuft (Sicherheitsrisiko), und welche Kernel-Capabilities explizit hinzugefügt oder entfernt wurden.

    - SecurityOpt
    Zeigt gesetzte Sicherheitsoptionen wie AppArmor- oder SELinux-Profile.

    - NetworkMode
    Gibt an, wie der Container ans Netzwerk angebunden ist (z. B. bridge, host, none). Kritisch, wenn z. B. host verwendet wird, da dann kein Netzwerk-Namespace verwendet wird.

    - ReadonlyRootfs
    Wenn gesetzt: Das Root-Dateisystem des Containers ist schreibgeschützt. Gilt als Best Practice zur Härtung.

    - LogConfig
    Zeigt, welche Logging-Methode verwendet wird (z. B. json-file). Das ist wichtig für Auditing & Fehlerdiagnose.

- Mounts & GraphDriver
    - Hier wird dargestellt, wie und wo Volumes oder Verzeichnisse vom Host eingebunden sind.

    - GraphDriver zeigt, wo die Dateisystem-Layer des Containers gespeichert sind (overlay2 ist Standard).

    - Dies ist zentral, wenn es um Datenpersistenz und mögliche Vertraulichkeitsverletzungen geht (z. B. ungewollte Mounts wie /).

- NetworkSettings

    - Zeigt die IP-Adresse, Gateway, Netzwerknamen usw. des Containers im jeweiligen Docker-Netzwerk.

    - Relevant für die Frage: Wer kann mit wem kommunizieren? → wichtige Grundlage zur Netzwerksegmentierung.

- State 
    - Zeigt, ob der Container aktiv ist, wann er gestartet wurde, ob er abgestürzt ist (z. B. durch OOM = Out Of Memory), und wann er gestoppt wurde.

    - Gut für Troubleshooting und zur Einschätzung, ob z. B. automatische Neustarts sinnvoll wären.

Diese Punkte sind wichtig, da man auf einen Blick erkennt, ob der Container unnötige Rechte besitzt oder des Erkennens von kritischen Volumes. 

### 3.4 Löschen des Containers

Nach der Analyse und Nutzung kann der Container wieder entfernt werden, um das System sauber zu halten:

```bash
it-security@host35:~$ docker rm test-container 
test-container
```
Zur Kontrolle, ob der Container wirklich entfernt wurde:

```bash
it-security@host35:~$ docker container list -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Der Container "test-container" erscheint nicht mehr in der Liste auf --> erfolgreich gelöscht.


## 4 Ausbruch

Da DockerContainer standardmäßig mit vollen Root-Rechten innerhalb des Containers laufen und der Docker-Daemon auf Host-Ebene Root-Zugriff benötigt, kann ein Nutzer mit Zugriff auf den Docker-Daemon das gesamte Hostsystem kompromittieren.

### 4.1 Lesen von Dateien auf dem Hostsystem

Wir mounten /etc/passwd vom Host in den Container und lesen die Datei aus: 

```bash
it-security@host35:~$ docker run --rm -v /etc/passwd:/tmp/passwd:ro ubuntu:latest cat /tmp/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:102:105::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:103:106:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
syslog:x:104:111::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:112:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:115::/run/uuidd:/usr/sbin/nologin
systemd-oom:x:108:116:systemd Userspace OOM Killer,,,:/run/systemd:/usr/sbin/nologin
tcpdump:x:109:117::/nonexistent:/usr/sbin/nologin
usbmux:x:111:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
dnsmasq:x:112:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
kernoops:x:113:65534:Kernel Oops Tracking Daemon,,,:/:/usr/sbin/nologin
avahi:x:114:121:Avahi mDNS daemon,,,:/run/avahi-daemon:/usr/sbin/nologin
cups-pk-helper:x:115:122:user for cups-pk-helper service,,,:/home/cups-pk-helper:/usr/sbin/nologin
rtkit:x:116:123:RealtimeKit,,,:/proc:/usr/sbin/nologin
whoopsie:x:117:124::/nonexistent:/bin/false
sssd:x:118:125:SSSD system user,,,:/var/lib/sss:/usr/sbin/nologin
speech-dispatcher:x:119:29:Speech Dispatcher,,,:/run/speech-dispatcher:/bin/false
saned:x:121:128::/var/lib/saned:/usr/sbin/nologin
colord:x:122:129:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin
geoclue:x:123:130::/var/lib/geoclue:/usr/sbin/nologin
pulse:x:124:131:PulseAudio daemon,,,:/run/pulse:/usr/sbin/nologin
gnome-initial-setup:x:125:65534::/run/gnome-initial-setup/:/bin/false
hplip:x:126:7:HPLIP system user,,,:/run/hplip:/bin/false
gdm:x:127:133:Gnome Display Manager:/var/lib/gdm3:/bin/false
it-security:x:1000:1000:it-security,,,:/home/it-security:/bin/bash
fwupd-refresh:x:120:137:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
dhcpcd:x:128:65534:DHCP Client Daemon,,,:/usr/lib/dhcpcd:/bin/false
cups-browsed:x:129:122::/nonexistent:/usr/sbin/nologin
polkitd:x:999:999:User for polkitd:/:/usr/sbin/nologin
gnome-remote-desktop:x:996:996:GNOME Remote Desktop:/var/lib/gnome-remote-desktop:/usr/sbin/nologin
```

Der Container hat Lesezugriff auf /etc/passwd vom Host, obwohl er isoliert ist - nur durch das Volume-Mounting.

### 4.2 Write file

Nun wird eine Datei erstellt die teils außerhalb des Containers sind, sondern direkt auf dem Host-Dateisystem: 

```bash
it-security@host35:~$ docker run --rm -v /tmp:/mnt ubuntu:latest sh -c 'echo "Dies ist ein Test" > /mnt/test-file.txt'
```

```bash
it-security@host35:~$ ls -la /tmp/ | grep test-file
-rw-r--r--  1 root        root           18 Apr  8 18:11 test-file.txt
```

```bash
it-security@host35:~$ cat /tmp/test-file.txt 
Dies ist ein Test
```

Die Datei liegt nun in /tmp/test-file.txt auf dem Host, erstellt durch einen Containerprozess. Sicherheitsrisiko bei unkontrollierten Mounts. 

### 4.3 Zugriff auf ganzes Dateisystem

Das gesamte Host-Dateisystem wird in den Container gemountet und es wird dann chroot /host eingesetzt: 

```bash
it-security@host35:~$ docker run -it --rm -v /:/host ubuntu:latest chroot /host
# ls -la
total 2097260
drwxr-xr-x  23 root root       4096 Nov 18 18:59 .
drwxr-xr-x  23 root root       4096 Nov 18 18:59 ..
lrwxrwxrwx   1 root root          7 Jun 22  2022 bin -> usr/bin
drwxr-xr-x   2 root root       4096 Nov 18 18:59 bin.usr-is-merged
drwxr-xr-x   4 root root       4096 Apr  8 17:24 boot
drwxrwxr-x   2 root root       4096 Jun 22  2022 cdrom
drwxr-xr-x  21 root root       4980 Apr  8 17:23 dev
drwxr-xr-x 145 root root      12288 Apr  8 17:41 etc
drwxr-xr-x   3 root root       4096 Jun 22  2022 home
lrwxrwxrwx   1 root root          7 Jun 22  2022 lib -> usr/lib
drwxr-xr-x   2 root root       4096 Nov 18 18:59 lib.usr-is-merged
lrwxrwxrwx   1 root root          9 Nov 18 15:35 lib32 -> usr/lib32
lrwxrwxrwx   1 root root          9 Jun 22  2022 lib64 -> usr/lib64
lrwxrwxrwx   1 root root         10 Nov 18 15:35 libx32 -> usr/libx32
drwx------   2 root root      16384 Jun 22  2022 lost+found
drwxr-xr-x   3 root root       4096 Apr 26  2023 media
drwxr-xr-x   2 root root       4096 Jun 22  2022 mnt
drwxr-xr-x   3 root root       4096 Apr  8 17:39 opt
dr-xr-xr-x 360 root root          0 Apr  8 16:21 proc
drwx------   7 root root       4096 Feb 19 20:13 root
drwxr-xr-x  36 root root       1040 Apr  8 17:39 run
lrwxrwxrwx   1 root root          8 Jun 22  2022 sbin -> usr/sbin
drwxr-xr-x   2 root root       4096 Nov 18 18:59 sbin.usr-is-merged
drwxr-xr-x  17 root root       4096 Nov 18 18:59 snap
drwxr-xr-x   2 root root       4096 Jun 22  2022 srv
-rw-------   1 root root 2147483648 Jun 22  2022 swapfile
dr-xr-xr-x  13 root root          0 Apr  8 16:21 sys
drwxrwxrwt  20 root root      12288 Apr  8 18:10 tmp
drwxr-xr-x  14 root root       4096 Apr 19  2022 usr
drwxr-xr-x  14 root root       4096 Nov 18 15:52 var
# 
```

Nun ist man im Rootverzeichnis des Hostsystems, mit Root Rechten --> vollständiger Containerausbruch. 
Deshalb sollte Docker Zugriff niemals unkontrolliert vergeben werden. 

## 5 Dockerfile

Nun wird ein eigenes Docker-Image mit einem einfachen Webserver (NGINX) gebaut, der eine statische HTML-Seite bereitstellt.

### 5.1 Vorbereitung 

Verzeichnisstruktur: 

```bash
it-security@host35:~$ mkdir docker-webserver
it-security@host35:~$ cd docker-webserver/
it-security@host35:~/docker-webserver$ 
it-security@host35:~/docker-webserver$ mkdir html
```

```bash
it-security@host35:~/docker-webserver$ echo "<html><body><h1>Mein Webserver</h1></body></html>" > html/index.html
```
### 5.2 Dockerfile

```bash
it-security@host35:~/docker-webserver$ vim Dockerfile
it-security@host35:~/docker-webserver$ cat Dockerfile 
FROM nginx:alpine
COPY html /usr/share/nginx/html
EXPOSE 80
```

### 5.3 Bauen des Images:

```bash
it-security@host35:~/docker-webserver$ docker build -t my-webserver .
[+] Building 3.4s (7/7) FINISHED                                                          docker:default
 => [internal] load build definition from Dockerfile                                                0.0s
 => => transferring dockerfile: 108B                                                                0.0s
 => [internal] load metadata for docker.io/library/nginx:alpine                                     1.6s
 => [internal] load .dockerignore                                                                   0.0s
 => => transferring context: 2B                                                                     0.0s
 => [internal] load build context                                                                   0.0s
 => => transferring context: 120B                                                                   0.0s
 => [1/2] FROM docker.io/library/nginx:alpine@sha256:4ff102c5d78d254a6f0da062b3cf39eaf07f01eec0927  1.4s
 => => resolve docker.io/library/nginx:alpine@sha256:4ff102c5d78d254a6f0da062b3cf39eaf07f01eec0927  0.0s
 => => sha256:4ff102c5d78d254a6f0da062b3cf39eaf07f01eec0927fd21e219d0af8bc0591 10.36kB / 10.36kB    0.0s
 => => sha256:a71e0884a7f1192ecf5decf062b67d46b54ad63f0cc1b8aa7e705f739a97c2fc 2.50kB / 2.50kB      0.0s
 => => sha256:1ff4bb4faebcfb1f7e01144fa9904a570ab9bab88694457855feb6c6bba3fa07 11.23kB / 11.23kB    0.0s
 => => sha256:ccc35e35d420d6dd115290f1074afc6ad1c474b6f94897434e6befd33be781d2 1.79MB / 1.79MB      0.2s
 => => sha256:43f2ec460bdf5babee56d7e13b060b308f42cbf2103f6665c349f0b98dac8962 627B / 627B          0.4s
 => => sha256:f18232174bc91741fdf3da96d85011092101a032a93a388b79e99e69c2d5c870 3.64MB / 3.64MB      0.3s
 => => sha256:984583bcf083fa6900b5e7834795a9a57a9b4dfe7448d5350474f5d309625ece 955B / 955B          0.4s
 => => extracting sha256:f18232174bc91741fdf3da96d85011092101a032a93a388b79e99e69c2d5c870           0.1s
 => => sha256:8d27c072a58f81ecf2425172ac0e5b25010ff2d014f89de35b90104e462568eb 402B / 402B          0.5s
 => => sha256:ab3286a7346303a31b69a5189f63f1414cc1de44e397088dcd07edb322df1fe9 1.21kB / 1.21kB      0.5s
 => => extracting sha256:ccc35e35d420d6dd115290f1074afc6ad1c474b6f94897434e6befd33be781d2           0.0s
 => => sha256:6d79cc6084d434876ce0f038c675d20532f28e476283a29e7e63bc0bf13a4ed6 1.40kB / 1.40kB      0.5s
 => => sha256:0c7e4c092ab716c95e1a8840943549b4ac5e6c5d1a1cc74496528e1b03e1e67a 15.38MB / 15.38MB    0.8s
 => => extracting sha256:43f2ec460bdf5babee56d7e13b060b308f42cbf2103f6665c349f0b98dac8962           0.0s
 => => extracting sha256:984583bcf083fa6900b5e7834795a9a57a9b4dfe7448d5350474f5d309625ece           0.0s
 => => extracting sha256:8d27c072a58f81ecf2425172ac0e5b25010ff2d014f89de35b90104e462568eb           0.0s
 => => extracting sha256:ab3286a7346303a31b69a5189f63f1414cc1de44e397088dcd07edb322df1fe9           0.0s
 => => extracting sha256:6d79cc6084d434876ce0f038c675d20532f28e476283a29e7e63bc0bf13a4ed6           0.0s
 => => extracting sha256:0c7e4c092ab716c95e1a8840943549b4ac5e6c5d1a1cc74496528e1b03e1e67a           0.3s
 => [2/2] COPY html /usr/share/nginx/html                                                           0.2s
 => exporting to image                                                                              0.1s
 => => exporting layers                                                                             0.0s
 => => writing image sha256:ac3d6bc08314912d2ecdacfda2ae82e8c7274366a78ea1f0a3571e27fdc7bd28        0.0s
 => => naming to docker.io/library/my-webserver                                                     0.0s
```

Das Image wurde erfolgreich erstellt. 

### 5.4 Reverse Proxy & Netzwerk

Damit der Webserver nicht direkt erreichbar ist, sondern nur über einen Reverse-Proxy, wird ein eigenes Docker-Netzwerk eingerichtet:

```bash
it-security@host35:~/docker-webserver$ docker network create webserver
```

Start Webserver (im privaten Netz):

```bash
it-security@host35:~/docker-webserver$ docker run -d --name webserver --network webserver my-webserver
41ffa80f2c15baa69e21480ff42c033a79234b25f0cdd07fbe1eef180e196829
```

Reverse Proxy mit NGINX:

```bash
it-security@host35:~/docker-webserver$ vim proxy.conf
it-security@host35:~/docker-webserver$ cat proxy.conf 
server {
    listen 80;
    location / {
        proxy_pass http://webserver:80;
    }
}
```

```bash
it-security@host35:~/docker-webserver$ docker run -d --name reverse-proxy \
    -p 8080:80 \
    -v $(pwd)/proxy.conf:/etc/nginx/conf.d/default.conf \
    --network webserver \
    nginx:alpine
Unable to find image 'nginx:alpine' locally
alpine: Pulling from library/nginx
f18232174bc9: Already exists 
ccc35e35d420: Already exists 
43f2ec460bdf: Already exists 
984583bcf083: Already exists 
8d27c072a58f: Already exists 
ab3286a73463: Already exists 
6d79cc6084d4: Already exists 
0c7e4c092ab7: Already exists 
Digest: sha256:4ff102c5d78d254a6f0da062b3cf39eaf07f01eec0927fd21e219d0af8bc0591
Status: Downloaded newer image for nginx:alpine
1a7a37eb923f413137af79d939596c956ed57c0996a3eb87c45ce2a316e316ea
```

Jetzt ist nur der Reverse Proxy auf Port 8080 von außen erreichbar – der Webserver ist isoliert.


### 5.5 Log-Dateien auf dem Host anzeigen

Logs des Webservers lassen sich über den Befehl ansehen:

```bash
it-security@host35:~/docker-webserver$ docker logs -f webserver 
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2025/04/08 16:50:30 [notice] 1#1: using the "epoll" event method
2025/04/08 16:50:30 [notice] 1#1: nginx/1.27.4
2025/04/08 16:50:30 [notice] 1#1: built by gcc 14.2.0 (Alpine 14.2.0) 
2025/04/08 16:50:30 [notice] 1#1: OS: Linux 6.8.0-53-generic
2025/04/08 16:50:30 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2025/04/08 16:50:30 [notice] 1#1: start worker processes
2025/04/08 16:50:30 [notice] 1#1: start worker process 30
2025/04/08 16:50:30 [notice] 1#1: start worker process 31
2025/04/08 16:50:30 [notice] 1#1: start worker process 32
2025/04/08 16:50:30 [notice] 1#1: start worker process 33
2025/04/08 16:50:30 [notice] 1#1: start worker process 34
2025/04/08 16:50:30 [notice] 1#1: start worker process 35
2025/04/08 16:50:30 [notice] 1#1: start worker process 36
2025/04/08 16:50:30 [notice] 1#1: start worker process 37
172.18.0.3 - - [08/Apr/2025:16:51:06 +0000] "GET / HTTP/1.0" 200 50 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:132.0) Gecko/20100101 Firefox/132.0" "-"
2025/04/08 16:51:06 [error] 31#31: *2 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.18.0.3, server: localhost, request: "GET /favicon.ico HTTP/1.0", host: "webserver", referrer: "http://localhost:8080/"
172.18.0.3 - - [08/Apr/2025:16:51:06 +0000] "GET /favicon.ico HTTP/1.0" 404 153 "http://localhost:8080/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:132.0) Gecko/20100101 Firefox/132.0" "-"
172.18.0.3 - - [08/Apr/2025:16:53:54 +0000] "GET / HTTP/1.0" 200 50 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:137.0) Gecko/20100101 Firefox/137.0" "-"
172.18.0.3 - - [08/Apr/2025:16:53:54 +0000] "GET /favicon.ico HTTP/1.0" 404 153 "http://192.168.10.35:8080/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:137.0) Gecko/20100101 Firefox/137.0" "-"
2025/04/08 16:53:54 [error] 33#33: *4 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.18.0.3, server: localhost, request: "GET /favicon.ico HTTP/1.0", host: "webserver", referrer: "http://192.168.10.35:8080/"
```

Zeigt alle HTTP-Anfragen und Fehler (z. B. fehlende favicon.ico).

Logs reverse-proxy:

```bash
it-security@host35:~docker logs -f reverse-proxy 
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: /etc/nginx/conf.d/default.conf differs from the packaged version
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2025/04/08 16:53:44 [notice] 1#1: using the "epoll" event method
2025/04/08 16:53:44 [notice] 1#1: nginx/1.27.4
2025/04/08 16:53:44 [notice] 1#1: built by gcc 14.2.0 (Alpine 14.2.0) 
2025/04/08 16:53:44 [notice] 1#1: OS: Linux 6.8.0-53-generic
2025/04/08 16:53:44 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2025/04/08 16:53:44 [notice] 1#1: start worker processes
2025/04/08 16:53:44 [notice] 1#1: start worker process 29
2025/04/08 16:53:44 [notice] 1#1: start worker process 30
2025/04/08 16:53:44 [notice] 1#1: start worker process 31
2025/04/08 16:53:44 [notice] 1#1: start worker process 32
2025/04/08 16:53:44 [notice] 1#1: start worker process 33
2025/04/08 16:53:44 [notice] 1#1: start worker process 34
2025/04/08 16:53:44 [notice] 1#1: start worker process 35
2025/04/08 16:53:44 [notice] 1#1: start worker process 36
192.168.10.36 - - [08/Apr/2025:16:53:54 +0000] "GET / HTTP/1.1" 200 50 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:137.0) Gecko/20100101 Firefox/137.0" "-"
192.168.10.36 - - [08/Apr/2025:16:53:54 +0000] "GET /favicon.ico HTTP/1.1" 404 153 "http://192.168.10.35:8080/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:137.0) Gecko/20100101 Firefox/137.0" "-"
```


![](https://i.imgur.com/S0UHNip.png)


## 6 Netzwerk

Webserver und Datenbank → gleiches Netz (webdb-network)
Separater Container (isolated-container) → eigenes Netz (isolated-network)

### 6.1 Erstellen der Networks:

```bash
it-security@host35:~$ docker network create webdb-network
355905412f636162c3446d7002175b0095c2eb7e6dcaf63eab16a6a3555c9db3
```

```bash
it-security@host35:~$ docker network create isolated-network
470c71399d568a010ae4caab08f8f6f5ba0d28124b0afcf9f5e7cc45b043b2cf
```

### 6.2 Container starten 

Starten des webserver in webdb-network:


 #### Webserver:
```bash
it-security@host35:~$ docker stop webserver
it-security@host35:~$ docker rm webserver 
webserver
it-security@host35:~$ docker run -d --name webserver --network webdb-network my-webserver
9722fd737990d2bbe15c8bd70cc9b70d7c0905071d0bd1277f0f8c3738f68da2
```
#### Datenbank:
```bash
it-security@host35:~$ docker run -d --name database --network webdb-network -e POSTGRES_PASSWORD=password postgres:alpine
Unable to find image 'postgres:alpine' locally
alpine: Pulling from library/postgres
f18232174bc9: Already exists 
d8d8fb695a5a: Pull complete 
c24c1ba610df: Pull complete 
83efd74bc97e: Pull complete 
215ba3ecdc26: Pull complete 
15ca4c67ed92: Pull complete 
4f18957d9158: Pull complete 
404c53e09e31: Pull complete 
9e4acb9ca7d3: Pull complete 
8f4971a5dfe7: Pull complete 
Digest: sha256:7062a2109c4b51f3c792c7ea01e83ed12ef9a980886e3b3d380a7d2e5f6ce3f5
Status: Downloaded newer image for postgres:alpine
629584ec8b0c0809122fe6bd7aa0506b06444cc4e0309053f9796a72935d2a4f
```
#### Isolierter Container: 

```bash
it-security@host35:~$ docker run -d --name isolated-container --network isolated-network alpine sleep infinity
a53ffac940a4c5bf45d040b3fb6d409a3049456a8ecf676bde350f87beeaf531
```


```bash
it-security@host35:~$ docker inspect webdb-network
[
    {
        "Name": "webdb-network",
        "Id": "355905412f636162c3446d7002175b0095c2eb7e6dcaf63eab16a6a3555c9db3",
        "Created": "2025-04-08T19:10:20.005496603+02:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "629584ec8b0c0809122fe6bd7aa0506b06444cc4e0309053f9796a72935d2a4f": {
                "Name": "database",
                "EndpointID": "dca1a0db36e1a7771f410abec51a9fa805e39d8d25c14d449d3980026cb29a1a",
                "MacAddress": "ee:5e:10:cd:92:b7",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            },
            "9722fd737990d2bbe15c8bd70cc9b70d7c0905071d0bd1277f0f8c3738f68da2": {
                "Name": "webserver",
                "EndpointID": "66f6b9979228879f3bc5cc17275110d8b195168210f9e96af8fb8cf8e0be1596",
                "MacAddress": "3e:aa:f7:5e:4d:1a",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

```bash
it-security@host35:~$ docker inspect isolated-network
[
    {
        "Name": "isolated-network",
        "Id": "470c71399d568a010ae4caab08f8f6f5ba0d28124b0afcf9f5e7cc45b043b2cf",
        "Created": "2025-04-08T19:20:34.511538602+02:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.21.0.0/16",
                    "Gateway": "172.21.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "a53ffac940a4c5bf45d040b3fb6d409a3049456a8ecf676bde350f87beeaf531": {
                "Name": "isolated-container",
                "EndpointID": "b931c63aa21a43ac8be0db06e46a71eb2938ae2259bef02c847f04e62f69e2d9",
                "MacAddress": "2e:04:d6:fd:c1:f5",
                "IPv4Address": "172.21.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```
#### Verbindung testen: 

```bash
it-security@host35:~$ docker exec webserver ping -c 2 database
PING database (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.080 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.074 ms

--- database ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.074/0.077/0.080 ms
```

```bash
it-security@host35:~$ docker exec isolated-container ping -c 2 webserver
ping: bad address 'webserver'
```

 Die Isolation funktioniert: Container im isolated-network haben keinen Zugriff auf andere Container.


## 7 Capabilities

Docker-Container starten standardmäßig mit vielen Kernel-Capabilities. Diese werden nun reduziert auf ein Minimum für einen NGINX-Container:

```bash
it-security@host35:~$ docker run -d --name secure-nginx \
    --cap-drop ALL \
    --cap-add NET_BIND_SERVICE \
    --cap-add CHOWN \
    --cap-add SETGID \
    --cap-add SETUID \
    --cap-add DAC_OVERRIDE \
    -p 127.0.0.1:8081:80 \
    nginx:alpine
a347d2d1219e3039368510df8bf4a330e008da77f480041e9036652c169208ad
```

![](https://i.imgur.com/usEhYiz.png)

Nur nötige Rechte, um statische Dateien auf Port 80 bereitzustellen – nach dem Prinzip "least privilege".

#### Prüfung: 

```bash
it-security@host35:~$ docker inspect --format '{{.HostConfig.CapAdd}}' secure-nginx
[CAP_CHOWN CAP_DAC_OVERRIDE CAP_NET_BIND_SERVICE CAP_SETGID CAP_SETUID]
```

```bash
it-security@host35:~$ docker inspect --format '{{.HostConfig.CapDrop}}' secure-nginx
[ALL]
```

## 8 Docker Security Bench

### 8.1 Installation: 

```bash
it-security@host35:~$ git clone https://github.com/docker/docker-bench-security.git
Cloning into 'docker-bench-security'...
remote: Enumerating objects: 2730, done.
remote: Counting objects: 100% (990/990), done.
remote: Compressing objects: 100% (186/186), done.
remote: Total 2730 (delta 859), reused 804 (delta 804), pack-reused 1740 (from 2)
Receiving objects: 100% (2730/2730), 4.46 MiB | 35.98 MiB/s, done.
Resolving deltas: 100% (1898/1898), done.
```

```bash
t-security@host35:~$ cd docker-bench-security/
it-security@host35:~/docker-bench-security$ sudo sh docker-bench-security.sh
[sudo] password for it-security: 
# --------------------------------------------------------------------------------------------
# Docker Bench for Security v1.6.0
#
# Docker, Inc. (c) 2015-2025
#
# Checks for dozens of common best-practices around deploying Docker containers in production.
# Based on the CIS Docker Benchmark 1.6.0.
# --------------------------------------------------------------------------------------------

Initializing 2025-04-08T19:43:14+02:00


Section A - Check results

[INFO] 1 - Host Configuration
[INFO] 1.1 - Linux Hosts Specific Configuration
[WARN] 1.1.1 - Ensure a separate partition for containers has been created (Automated)
[INFO] 1.1.2 - Ensure only trusted users are allowed to control Docker daemon (Automated)
[INFO]       * Users: it-security
[WARN] 1.1.3 - Ensure auditing is configured for the Docker daemon (Automated)
[WARN] 1.1.4 - Ensure auditing is configured for Docker files and directories -/run/containerd (Automated)
[WARN] 1.1.5 - Ensure auditing is configured for Docker files and directories - /var/lib/docker (Automated)
[WARN] 1.1.6 - Ensure auditing is configured for Docker files and directories - /etc/docker (Automated)
[WARN] 1.1.7 - Ensure auditing is configured for Docker files and directories - docker.service (Automated)
[INFO] 1.1.8 - Ensure auditing is configured for Docker files and directories - containerd.sock (Automated)
[INFO]        * File not found
[WARN] 1.1.9 - Ensure auditing is configured for Docker files and directories - docker.socket (Automated)
[WARN] 1.1.10 - Ensure auditing is configured for Docker files and directories - /etc/default/docker (Automated)
[INFO] 1.1.11 - Ensure auditing is configured for Dockerfiles and directories - /etc/docker/daemon.json (Automated)
[INFO]        * File not found
[WARN] 1.1.12 - 1.1.12 Ensure auditing is configured for Dockerfiles and directories - /etc/containerd/config.toml (Automated)
[INFO] 1.1.13 - Ensure auditing is configured for Docker files and directories - /etc/sysconfig/docker (Automated)
[INFO]        * File not found
[WARN] 1.1.14 - Ensure auditing is configured for Docker files and directories - /usr/bin/containerd (Automated)
[WARN] 1.1.15 - Ensure auditing is configured for Docker files and directories - /usr/bin/containerd-shim (Automated)
[WARN] 1.1.16 - Ensure auditing is configured for Docker files and directories - /usr/bin/containerd-shim-runc-v1 (Automated)
[WARN] 1.1.17 - Ensure auditing is configured for Docker files and directories - /usr/bin/containerd-shim-runc-v2 (Automated)
[WARN] 1.1.18 - Ensure auditing is configured for Docker files and directories - /usr/bin/runc (Automated)
[INFO] 1.2 - General Configuration
[NOTE] 1.2.1 - Ensure the container host has been Hardened (Manual)
[PASS] 1.2.2 - Ensure that the version of Docker is up to date (Manual)
[INFO]        * Using 28.0.4 which is current
[INFO]        * Check with your operating system vendor for support and security maintenance for Docker

[INFO] 2 - Docker daemon configuration
[NOTE] 2.1 - Run the Docker daemon as a non-root user, if possible (Manual)
docker-bench-security.sh: 37: [[: not found
[WARN] 2.2 - Ensure network traffic is restricted between containers on the default bridge (Scored)
[PASS] 2.3 - Ensure the logging level is set to 'info' (Scored)
docker-bench-security.sh: 96: [[: not found
[PASS] 2.4 - Ensure Docker is allowed to make changes to iptables (Scored)
docker-bench-security.sh: 118: [[: not found
[PASS] 2.5 - Ensure insecure registries are not used (Scored)
[PASS] 2.6 - Ensure aufs storage driver is not used (Scored)
[INFO] 2.7 - Ensure TLS authentication for Docker daemon is configured (Scored)
[INFO]      * Docker daemon not listening on TCP
docker-bench-security.sh: 185: [[: not found
[INFO] 2.8 - Ensure the default ulimit is configured appropriately (Manual)
[INFO]      * Default ulimit doesn't appear to be set
docker-bench-security.sh: 208: [[: not found
[WARN] 2.9 - Enable user namespace support (Scored)
[PASS] 2.10 - Ensure the default cgroup usage has been confirmed (Scored)
[PASS] 2.11 - Ensure base device size is not changed until needed (Scored)
docker-bench-security.sh: 276: [[: not found
[WARN] 2.12 - Ensure that authorization for Docker client commands is enabled (Scored)
[WARN] 2.13 - Ensure centralized and remote logging is configured (Scored)
[WARN] 2.14 - Ensure containers are restricted from acquiring new privileges (Scored)
[WARN] 2.15 - Ensure live restore is enabled (Scored)
[WARN] 2.16 - Ensure Userland Proxy is Disabled (Scored)
[INFO] 2.17 - Ensure that a daemon-wide custom seccomp profile is applied if appropriate (Manual)
[INFO] Ensure that experimental features are not implemented in production (Scored) (Deprecated)

[INFO] 3 - Docker daemon configuration files
[PASS] 3.1 - Ensure that the docker.service file ownership is set to root:root (Automated)
[PASS] 3.2 - Ensure that docker.service file permissions are appropriately set (Automated)
[PASS] 3.3 - Ensure that docker.socket file ownership is set to root:root (Automated)
[PASS] 3.4 - Ensure that docker.socket file permissions are set to 644 or more restrictive (Automated)
[PASS] 3.5 - Ensure that the /etc/docker directory ownership is set to root:root (Automated)
[PASS] 3.6 - Ensure that /etc/docker directory permissions are set to 755 or more restrictively (Automated)
[INFO] 3.7 - Ensure that registry certificate file ownership is set to root:root (Automated)
[INFO]      * Directory not found
[INFO] 3.8 - Ensure that registry certificate file permissions are set to 444 or more restrictively (Automated)
[INFO]      * Directory not found
[INFO] 3.9 - Ensure that TLS CA certificate file ownership is set to root:root (Automated)
[INFO]      * No TLS CA certificate found
[INFO] 3.10 - Ensure that TLS CA certificate file permissions are set to 444 or more restrictively (Automated)
[INFO]       * No TLS CA certificate found
[INFO] 3.11 - Ensure that Docker server certificate file ownership is set to root:root (Automated)
[INFO]       * No TLS Server certificate found
[INFO] 3.12 - Ensure that the Docker server certificate file permissions are set to 444 or more restrictively (Automated)
[INFO]       * No TLS Server certificate found
[INFO] 3.13 - Ensure that the Docker server certificate key file ownership is set to root:root (Automated)
[INFO]       * No TLS Key found
[INFO] 3.14 - Ensure that the Docker server certificate key file permissions are set to 400 (Automated)
[INFO]       * No TLS Key found
[PASS] 3.15 - Ensure that the Docker socket file ownership is set to root:docker (Automated)
[PASS] 3.16 - Ensure that the Docker socket file permissions are set to 660 or more restrictively (Automated)
[INFO] 3.17 - Ensure that the daemon.json file ownership is set to root:root (Automated)
[INFO]       * File not found
[INFO] 3.18 - Ensure that daemon.json file permissions are set to 644 or more restrictive (Automated)
[INFO]       * File not found
[PASS] 3.19 - Ensure that the /etc/default/docker file ownership is set to root:root (Automated)
[PASS] 3.20 - Ensure that the /etc/default/docker file permissions are set to 644 or more restrictively (Automated)
[INFO] 3.21 - Ensure that the /etc/sysconfig/docker file permissions are set to 644 or more restrictively (Automated)
[INFO]       * File not found
[INFO] 3.22 - Ensure that the /etc/sysconfig/docker file ownership is set to root:root (Automated)
[INFO]       * File not found
[PASS] 3.23 - Ensure that the Containerd socket file ownership is set to root:root (Automated)
[PASS] 3.24 - Ensure that the Containerd socket file permissions are set to 660 or more restrictively (Automated)

[INFO] 4 - Container Images and Build File
[WARN] 4.1 - Ensure that a user for the container has been created (Automated)
[WARN]      * Running as root: secure-nginx
[WARN]      * Running as root: database
[WARN]      * Running as root: isolated-container
[WARN]      * Running as root: webserver
[NOTE] 4.2 - Ensure that containers use only trusted base images (Manual)
[NOTE] 4.3 - Ensure that unnecessary packages are not installed in the container (Manual)
[NOTE] 4.4 - Ensure images are scanned and rebuilt to include security patches (Manual)
[WARN] 4.5 - Ensure Content trust for Docker is Enabled (Automated)
[WARN] 4.6 - Ensure that HEALTHCHECK instructions have been added to container images (Automated)
[WARN]      * No Healthcheck found: ac3d6bc08314
[WARN]      * No Healthcheck found: [my-webserver:latest]
[WARN]      * No Healthcheck found: [postgres:alpine]
[WARN]      * No Healthcheck found: [alpine:latest]
[WARN]      * No Healthcheck found: [nginx:alpine]
[WARN]      * No Healthcheck found: [ubuntu:latest]
[WARN]      * No Healthcheck found: [hello-world:latest]
[PASS] 4.7 - Ensure update instructions are not used alone in the Dockerfile (Manual)
[NOTE] 4.8 - Ensure setuid and setgid permissions are removed (Manual)
[INFO] 4.9 - Ensure that COPY is used instead of ADD in Dockerfiles (Manual)
[INFO]      * ADD in image history: [ubuntu:latest]
[NOTE] 4.10 - Ensure secrets are not stored in Dockerfiles (Manual)
[NOTE] 4.11 - Ensure only verified packages are installed (Manual)
[NOTE] 4.12 - Ensure all signed artifacts are validated (Manual)

[INFO] 5 - Container Runtime
[PASS] 5.1 - Ensure swarm mode is not Enabled, if not needed (Automated)
[PASS] 5.2 - Ensure that, if applicable, an AppArmor Profile is enabled (Automated)
[WARN] 5.3 - Ensure that, if applicable, SELinux security options are set (Automated)
[WARN]      * No SecurityOptions Found: secure-nginx
[WARN]      * No SecurityOptions Found: database
[WARN]      * No SecurityOptions Found: isolated-container
[WARN]      * No SecurityOptions Found: webserver
[PASS] 5.4 - Ensure that Linux kernel capabilities are restricted within containers (Automated)
[PASS] 5.5 - Ensure that privileged containers are not used (Automated)
[PASS] 5.6 - Ensure sensitive host system directories are not mounted on containers (Automated)
[PASS] 5.7 - Ensure sshd is not run within containers (Automated)
[PASS] 5.8 - Ensure privileged ports are not mapped within containers (Automated)
[WARN] 5.9 - Ensure that only needed ports are open on the container (Manual)
[WARN]      * Port in use: 8081 in secure-nginx
[PASS] 5.10 - Ensure that the host's network namespace is not shared (Automated)
[WARN] 5.11 - Ensure that the memory usage for containers is limited (Automated)
[WARN]       * Container running without memory restrictions: secure-nginx
[WARN]       * Container running without memory restrictions: database
[WARN]       * Container running without memory restrictions: isolated-container
[WARN]       * Container running without memory restrictions: webserver
[WARN] 5.12 - Ensure that CPU priority is set appropriately on containers (Automated)
[WARN]       * Container running without CPU restrictions: secure-nginx
[WARN]       * Container running without CPU restrictions: database
[WARN]       * Container running without CPU restrictions: isolated-container
[WARN]       * Container running without CPU restrictions: webserver
[WARN] 5.13 - Ensure that the container's root filesystem is mounted as read only (Automated)
[WARN]       * Container running with root FS mounted R/W: secure-nginx
[WARN]       * Container running with root FS mounted R/W: database
[WARN]       * Container running with root FS mounted R/W: isolated-container
[WARN]       * Container running with root FS mounted R/W: webserver
[PASS] 5.14 - Ensure that incoming container traffic is bound to a specific host interface (Automated)
0
0
0
0
[PASS] 5.15 - Ensure that the 'on-failure' container restart policy is set to '5' (Automated)
[PASS] 5.16 - Ensure that the host's process namespace is not shared (Automated)
[PASS] 5.17 - Ensure that the host's IPC namespace is not shared (Automated)
[PASS] 5.18 - Ensure that host devices are not directly exposed to containers (Manual)
[INFO] 5.19 - Ensure that the default ulimit is overwritten at runtime if needed (Manual)
[INFO]       * Container no default ulimit override: secure-nginx
[INFO]       * Container no default ulimit override: database
[INFO]       * Container no default ulimit override: isolated-container
[INFO]       * Container no default ulimit override: webserver
[PASS] 5.20 - Ensure mount propagation mode is not set to shared (Automated)
[PASS] 5.21 - Ensure that the host's UTS namespace is not shared (Automated)
[PASS] 5.22 - Ensure the default seccomp profile is not Disabled (Automated)
[NOTE] 5.23 - Ensure that docker exec commands are not used with the privileged option (Automated)
[NOTE] 5.24 - Ensure that docker exec commands are not used with the user=root option (Manual)
[PASS] 5.25 - Ensure that cgroup usage is confirmed (Automated)
[WARN] 5.26 - Ensure that the container is restricted from acquiring additional privileges (Automated)
[WARN]       * Privileges not restricted: secure-nginx
[WARN]       * Privileges not restricted: database
[WARN]       * Privileges not restricted: isolated-container
[WARN]       * Privileges not restricted: webserver
[WARN] 5.27 - Ensure that container health is checked at runtime (Automated)
[WARN]       * Health check not set: secure-nginx
[WARN]       * Health check not set: database
[WARN]       * Health check not set: isolated-container
[WARN]       * Health check not set: webserver
[INFO] 5.28 - Ensure that Docker commands always make use of the latest version of their image (Manual)
[WARN] 5.29 - Ensure that the PIDs cgroup limit is used (Automated)
[WARN]       * PIDs limit not set: secure-nginx
[WARN]       * PIDs limit not set: database
[WARN]       * PIDs limit not set: isolated-container
[WARN]       * PIDs limit not set: webserver
[INFO] 5.30 - Ensure that Docker's default bridge 'docker0' is not used (Manual)
[INFO]       * Container in docker0 network: secure-nginx
[PASS] 5.31 - Ensure that the host's user namespaces are not shared (Automated)
[PASS] 5.32 - Ensure that the Docker socket is not mounted inside any containers (Automated)

[INFO] 6 - Docker Security Operations
[INFO] 6.1 - Ensure that image sprawl is avoided (Manual)
[INFO]      * There are currently: 7 images
[INFO] 6.2 - Ensure that container sprawl is avoided (Manual)
[INFO]      * There are currently a total of 7 containers, with 4 of them currently running

[INFO] 7 - Docker Swarm Configuration
[PASS] 7.1 - Ensure that the minimum number of manager nodes have been created in a swarm (Automated) (Swarm mode not enabled)
[PASS] 7.2 - Ensure that swarm services are bound to a specific host interface (Automated) (Swarm mode not enabled)
[PASS] 7.3 - Ensure that all Docker swarm overlay networks are encrypted (Automated)
[PASS] 7.4 - Ensure that Docker's secret management commands are used for managing secrets in a swarm cluster (Manual) (Swarm mode not enabled)
[PASS] 7.5 - Ensure that swarm manager is run in auto-lock mode (Automated) (Swarm mode not enabled)
[PASS] 7.6 - Ensure that the swarm manager auto-lock key is rotated periodically (Manual) (Swarm mode not enabled)
[PASS] 7.7 - Ensure that node certificates are rotated as appropriate (Manual) (Swarm mode not enabled)
[PASS] 7.8 - Ensure that CA certificates are rotated as appropriate (Manual) (Swarm mode not enabled)
[PASS] 7.9 - Ensure that management plane traffic is separated from data plane traffic (Manual) (Swarm mode not enabled)


Section C - Score

[INFO] Checks: 117
[INFO] Score: 8
```

 Das Tool prüft das Docker-Setup auf Sicherheitslücken nach dem CIS Docker Benchmark.

#### Fix 2.14 - "No new privileges"

```bash
it-security@host35:~/docker-bench-security$ sudo mkdir -p /etc/docker
echo '{
  "no-new-privileges": true
}' | sudo tee /etc/docker/daemon.json
{
  "no-new-privileges": true
}
it-security@host35:~/docker-bench-security$ sudo systemctl restart docker
```
Ergebnis: PASS bei Check 2.14




```bash
it-security@host35:~/docker-bench-security$ sudo sh docker-bench-security.sh | grep 2.14
[PASS] 2.14 - Ensure containers are restricted from acquiring new privileges (Scored)
```

#### Fix 2.15 - Live- Restore aktivieren

Enabling dockerd LiveRestore according to 2.15:

```bash
it-security@host35:~/docker-bench-security$ sudo cat /etc/docker/daemon.json 
{
  "no-new-privileges": true,
  "live-restore": true
}
```

```bash
it-security@host35:~/docker-bench-security$ sudo systemctl reload docker
```

```bash
it-security@host35:~/docker-bench-security$ sudo sh docker-bench-security.sh | grep 2.15
# Docker, Inc. (c) 2015-2025
[PASS] 2.15 - Ensure live restore is enabled (Scored)
```

 Container laufen weiter, auch wenn der Docker-Daemon neu gestartet wird.

## 9 Cleanup

Zum Abschluss werden alle nicht verwendeten Ressourcen aufgeräumt. Dies wird automatisiert durchgeführt:

```bash
it-security@host35:~/docker-bench-security$ docker container prune -f
Deleted Containers:
a347d2d1219e3039368510df8bf4a330e008da77f480041e9036652c169208ad
629584ec8b0c0809122fe6bd7aa0506b06444cc4e0309053f9796a72935d2a4f
a53ffac940a4c5bf45d040b3fb6d409a3049456a8ecf676bde350f87beeaf531
9722fd737990d2bbe15c8bd70cc9b70d7c0905071d0bd1277f0f8c3738f68da2
fd302263942303056823c12d39ace8b70413ba42a4d38f55e5c17ee06e4c172f
efc0326f1776f8c0418216a2e10fba9716b7285caa2b9fd014b5d0c859ff1b76
5a41a2849862bdbdb398516040aa6802568ed6cc3e200b7fe029143b1641f8c4

Total reclaimed space: 62.85MB
```

```bash
it-security@host35:~/docker-bench-security$ docker image prune -a -f
Deleted Images:
untagged: hello-world:latest
untagged: hello-world@sha256:7e1a4e2d11e2ac7a8c3f768d4166c2defeb09d2a750b010412b6ea13de1efb19
deleted: sha256:74cc54e27dc41bb10dc4b2226072d469509f2f22f1a3ce74f4a59661a1d44602
deleted: sha256:63a41026379f4391a306242eb0b9f26dc3550d863b7fdbb97d899f6eb89efe72
untagged: ubuntu:latest
untagged: ubuntu@sha256:72297848456d5d37d1262630108ab308d3e9ec7ed1c3286a32fe09856619a782
deleted: sha256:a04dc4851cbcbb42b54d1f52a41f5f9eca6a5fd03748c3f6eb2cbeb238ca99bd
deleted: sha256:4b7c01ed0534d4f9be9cf97d068da1598c6c20b26cb6134fad066defdb6d541d
deleted: sha256:ac3d6bc08314912d2ecdacfda2ae82e8c7274366a78ea1f0a3571e27fdc7bd28
untagged: alpine:latest
untagged: alpine@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c
deleted: sha256:aded1e1a5b3705116fa0a92ba074a5e0b0031647d9c315983ccba2ee5428ec8b
untagged: postgres:alpine
untagged: postgres@sha256:7062a2109c4b51f3c792c7ea01e83ed12ef9a980886e3b3d380a7d2e5f6ce3f5
deleted: sha256:0150e3200277b907866758e22a3b9bba7857325b994848fe9f44b3610c3ae17e
deleted: sha256:c2474619a6cb529b8c1f97d72b2392fb35851bbea219e99b70c17d3c5b51727e
deleted: sha256:cee2be48ffaf7cfd283a17682b241131d0fe896854e6827800e2ed10eb358444
deleted: sha256:afbafd88af4bac18e164f8bcd2d5ad0e53d4f65628cc565ab384c9f39b9c47c3
deleted: sha256:aa13ccabfcdc4967e7be5ec1f9f282febbc0572c1ddfe971b7fc54831683df69
deleted: sha256:6acba8b80221e54a955de47105c0e20e2ad2ca2e6ec019c14f15d97476f60d5d
deleted: sha256:4a0c6dccf9a2ce4a4a57fe792cdf65a9f62d8a6144cc724c589ec81940eb67b2
deleted: sha256:9ca7d9847731428c757f47d4198e9210abc94d33efa4efa85f65d110e6bd1b79
deleted: sha256:cc76bfd5e996e6456a4a91b0d639552aa08705da364959d921ec13f75a1c1f56
deleted: sha256:062e461804935daef2beb65d8588ad88a52784500fafab113bf6cfe553b709e5
untagged: nginx:alpine
untagged: nginx@sha256:4ff102c5d78d254a6f0da062b3cf39eaf07f01eec0927fd21e219d0af8bc0591
deleted: sha256:1ff4bb4faebcfb1f7e01144fa9904a570ab9bab88694457855feb6c6bba3fa07
untagged: my-webserver:latest
deleted: sha256:323cb26f3639acdef319f21151e580bc7e2cf562f33b02bb03ea53849d32689c

Total reclaimed space: 348.5MB
```

```bash
it-security@host35:~/docker-bench-security$ docker network prune -f
Deleted Networks:
webserver
webdb-network
isolated-network
isloated-network
```

```bash
it-security@host35:~/docker-bench-security$ docker volume prune -f
Deleted Volumes:
57a382af4efe9a8bb1ba3e7d9fa1dee552bd39c6dae4dddb46da94c6893b3698
8605c95880ba7f69a54ecf59d539a9ce33b1509aa36dfd31076882d97b6c4b87
873e744d9e3137071d89188e865034a5d78c2762cd8d8112069f0325a62e7597
d6cd68faf2c49926f04df04fb400a49c5af31ee6e9762fe93d2912b8672979ea

Total reclaimed space: 40.46MB
```

```bash
it-security@host35:~/docker-bench-security$ docker system prune -a -f
Deleted build cache objects:
4acinsdfb70szb4vbm124c42s
tdmx2b90egq3bky0e3utvjfop
xmnifm5d0xprrxjs2ov0a48n4
kaj9omgh9w81bwk4agv020l59
4loz8fk5idjmjttkq7ksypth4
620zoy0h27pg2joi845dgqsxg
ukqskukbkfsfol85b8o1z5fvq
4paccsxhe22b6wzdo32kdf77g
b7mmqgi9gb39d8temqngff7cc
qoajncgnzma01s1ez1a0r5rpc
pagvdv8r1q1c0te14b2ula5da
dtv6oko60do215he0a4ymiwyx

Total reclaimed space: 160B
```

So bleibt das System sauber – ohne händisches Löschen.



Diese Übung demonstriert, wie wichtig sichere Docker-Konfigurationen, Netzwerkisolation und Rechtebeschränkungen sind. Durch das gezielte Anwenden von Best Practices (Capabilities, Reverse Proxy, Security Bench) kann ein sicherer und kontrollierter Betrieb gewährleistet werden.