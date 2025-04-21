# Übung 1: Virtualisierung 

### Datum:

18.02.2025

### Gruppenmitglieder: 
Philip Magnus
Lorenzo Haidinger
Astrid Kuzma-Kuzniarski

## 0 Aufgabenstellung

Diese Laborübung beschäftigt sich mit der Einrichtung und Verwaltung virtueller Maschinen (VMs) mithilfe von `libvirt`. Ziel ist es, ein vollständig funktionierendes System mit einer virtualisierten Umgebung aufzubauen, deren Management gesichert über ein Netzwerk stattfinden kann.

## 1 System einrichten 

Die Tools für die Laborübung wurden mit folgendem Command installiert:

```bash
$ sudo apt install libvirt-daemon virt-manager
```

Mit dem `useradd` Befehl wurde ein Benutzer zum Management der VMs erstellt:

```bash 
 sudo useradd vm-manager
```

Anschließend wurde der User zur `libvirt`-Gruppe hinzugefügt um diese ohne die Verwendung von `sudo` benutzen zu können.

```bash 
 sudo adduser vm-manager libvirt
```

Wechsel auf unseren erstellten User "vm-manager":

```bash
su vm-manager
```



## 2 Netzwerk konfigurieren

### 2.1. Hinzufügen einer Netzwerkbridge auf dem Hostsystem



In der yaml "`/etc/netplan/01-eno1.yaml`" haben wir folgendes eingefügt: 



```yaml
network:
  version: 2
  renderer: networkd

  ethernets:
    eno1:
      dhcp4: yes
  bridges:
    grennbridge0: 
      interfaces: [eno1]
      dhcp4: true
```

Mit den folgenden zwei Befehlen wurde die neue Netzwerkkonfiguration angewandt: 

```bash
sudo netplan generate 
```

```bash
sudo netplan --debug apply
```

Wir bearbeiten die Datei, `qemu.conf`, wie folgt: 

```bash
sudo nano /etc/libvirt/qemu.conf
```


 
 ```conf
 [...] 
 Some examples of valid values are:
 #
 user = "qemu"   # A user named "qemu"
 user = "+0"     # Super user (uid=0)
 user = "100"    # A user named "100" or a user with uid=100
 #
 user = "vm-manager"
 The group for QEMU processes run by the system instance. It can be
 specified in a similar way to user.
 group = "libvirt"
 [...]
```

Das machen wir damit `libvirt` auf das Storage File, in unserem User Kontext, zugreifen kann.

### 2.2. Erstellen einer XML-Netzwerk Konfiguration für libvirt

Wir erstellen folgendes XML-File:

```bash 
nano bridged-network.xml
```

Mit dem Inhalt: 

```bash
<network>
  <name>bridged-network</name>
  <bridge name="greenbridge0" />
  <forward mode="bridge" />
</network>
```

Hiermit lässt sich über `virsh` die auf dem Host vorkonfigurierte Network-Bridge für die VMs einrichten.

Über `virsh` verbinden wir uns mit dem libvirt-daemon:

```bash
virsh --connect qemu:///system
```

Nach der Verbindungsaufnahme mit dem Daemon konfigurieren wir die Network-Bridge mit dem vorher erstellten XML-File.

```bash
net-create --file bridged-network.xml
```

Anschließend starten wir die Bridge für libvirt:

```bash
net-start bridged-network
```

Mit dem Befehl `net-autostart` wird das Netzwerkinterface automatisch mit dem libvirt-daemon gestartet.

```bash
net-autostart bridged-network
```

 Der folgende Befehl initialisiert und startet die virtuelle Maschine, in der wir das Ubuntu Gastsystem installieren und konfigurieren:
 
 ```bash
 sudo virt-install --vcpus=1 --memory=4096 --cdrom=/var/lib/libvirt/images/ubuntu-24.04.1-live-server-amd64.iso --disk pool=vm-manager,size=7 --os-variant=ubuntu24.04 --network network=bridged-network
 ```
 
 
## 3 VM einrichten 

1. Man folgt dem Installationsassistenten des Gastsystems.


2. Persistenz und Autostart der VM wurde wie folgt konfiguriert:

Die Persistenz war schon auf `yes` gesetzt. Wir mussten mit folgendem Befehl in `virsh` den Autostart enablen:

```bash
virsh # autostart ubuntu24.04
```

Ausgabe : 

```bash
virsh # dominfo ubuntu24.04 
Id:             1
Name:           ubuntu24.04
UUID:           9f1b4425-faeb-4910-8536-411106847158
OS Type:        hvm
State:          running
CPU(s):         1
CPU time:       273,8s
Max memory:     4194304 KiB
Used memory:    4194304 KiB
Persistent:     yes
Autostart:      enable
Managed save:   no
Security model: apparmor
Security DOI:   0
Security label: libvirt-9f1b4425-faeb-4910-8536-411106847158 (enforcing)
```
 
Jetzt sehen wir, dass die Persistenz auf `yes` gesetzt und der Autostart enabled ist. 

## 4 Informationen libvirt daemon:

- _Welche Storage Pools sind vorhanden? Wie sind diese konfiguriert?_
- Welche Netzwerke sind vorhanden? Wie sind diese konfiguriert?_
- Wo im lokalen Dateisystem können Sie diese Informationen ohne Einsatz eines `libvirt-Clients` bekommen?

Wir können die verfügbaren Speicherpools in der `virsh`-Konsole mit `pool-list --all --details` auflisten. Wir sehen, dass es einen persistenten Pool gibt und mit dem Befehl `pool-info` haben wir zusätzliche Details überprüft. 

Um Informationen über Netzwerke zu erhalten, haben wir den Befehl net-list und den Befehl net-info verwendet, um mehr Informationen über ein bestimmtes Netzwerk zu erhalten.


```bash
virsh # pool-list --all --details 
virsh # pool-list
 Name         State    Autostart
----------------------------------
 default      active   yes
 vm-manager   active   yes
 
 
virsh # pool-info vm-manager 
Name:           vm-manager
UUID:           57e9e01b-35f7-461c-b7fb-56979221c572
State:          running
Persistent:     yes
Autostart:      yes
Capacity:       116,32 GiB
Allocation:     28,56 GiB
Available:      87,76 GiB
 
 
virsh # net-list 
 Name              State    Autostart   Persistent
----------------------------------------------------
 bridged-network   active   yes         yes
 default           active   yes         yes
 
virsh # net-info bridged-network 
Name:           bridged-network
UUID:           ddf2961d-9227-44e7-a424-d0f8c0bc20da
Active:         yes
Persistent:     yes
Autostart:      yes
Bridge:         greenbridge0
```

Ohne die `virsh`-Befehle zu verwenden, können wir die Konfiguration überprüfen:

- Für die Speicherpools in `/etc/libvirt/storage/images.xml` und `/etc/libvirt/storage/autostart/`

- Für die Netzwerke in `/etc/libvirt/qemu/networks/` und `/etc/libvirt/qemu/networks/autostart`

## 5 Zugriff über SSH

1. Man generiert einen SSH Key

Um die SSH-Verbindung zum libvirt-Daemon zu konfigurieren, benötigen wir einen SSH Key, daher führen wir als vmanager `ssh-keygen` aus.

```bash
ssh-keygen
```

```bash
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/it-security/.ssh/id_ed25519): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/it-security/.ssh/id_ed25519
Your public key has been saved in /home/it-security/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:G8wI2PA80JRJeBnEIQa5RTgr4wUVpPV4BDFngEPyGNw it-security@host35
The key's randomart image is:
+--[ED25519 256]--+
|**@^@X           |
|=X=E#            |
|.*=o*o           |
|=  ..o +         |
|o..   . S        |
| .       o       |
|        .        |
|                 |
|                 |
+----[SHA256]-----+
```

Um mit dem ssh-Key eine Verbindung zum `virsh` aufbauen zu können, wird der ssh-Key mit `ssh-copy-id` zum passenden Benutzer auf das Hostsystem übertragen. 

```bash
ssh-copy-id vm-manager@192.168.10.189
```

Vom Remote-System aus bauen wir als `vmanager` eine Verbindung zu `virsh` wie folgt auf:

```bash
virsh -c qemu+ssh://vm-manager@192.168.10.189/system
```

```bash
Ausgabe: 
Welcome to virsh, the virtualization interactive terminal.
```

Mit dem `list` Befehl können wir Überprüfen, dass unsere VM auf dem System tatsächlich läuft und wir diese auch remote managen können.

```bash
virsh # list
 Id   Name          State
-----------------------------
 2    ubuntu24.04   running
```

## 6 TLS

Für die Einrichtung einer TLS Verbindung benötigen wir ein CA Zertifikat, ein Server Zertifikat und ein Client Zertifikat. 
Zuerst wird ein privater Schlüssel für die CA mit folgendem Befehl generiert:

```bash
certtool --generate-privkey > cakey.pem
```

Anschließend erstellen wir eine Datei mit dem Namen `ca.info`, diese enthält die Signaturdetails für das CA Zertifikat.

```bash
cn = host35
ca
cert_signing_key
```

Das self-signed Zertifikat wird mit folgendem Befehl generiert:

```bash
certtool --generate-self-signed --load-privkey cakey.pem \
  --template ca.info --outfile cacert.pem
```

Jetzt hat man zwei Dateien, die von Bedeutung sind:

    cakey.pem - Your CA's private key (keep this very secret!)

    cacert.pem - Your CA's certificate (this is public).


Das Erstellen eines Server Zertifikats erfolgt analog zum Erstellen des CA Zertifikats:

```bash
certtool --generate-privkey > serverkey.pem
```

Erstellen der Signaturdetails: 
```bash
sudo nano server.info
```

Dort schreiben wir: 

```bash
organization = Team
cn = host35

tls_www_server
encryption_key
signing_key
```


Mit dem leicht angepassten Befehl wird das Server Zertifikat erstellt.

```bash
certtool --generate-certificate --load-privkey serverkey.pem \
  --load-ca-certificate cacert.pem --load-ca-privkey cakey.pem \
  --template server.info --outfile servercert.pem
  ```

Das Client Zertifikat wird analog zum Server Zertifikat erstellt:
  
  Erstellen des private key : 
  ```bash
  certtool --generate-privkey > clientkey.pem
  ```
  
  
wir erstellen eine client.info: 

```bash
country = AT
state = Vienna
locality = Vienna
organization = Libvirt Project
cn = host35
tls_www_client
encryption_key
signing_key
```

und signieren diese :
```bash
certtool --generate-certificate --load-privkey clientkey.pem \
  --load-ca-certificate cacert.pem --load-ca-privkey cakey.pem \
  --template client.info --outfile clientcert.pem
  ```
  
Zertifikate befinden sich im folgenden Pfad:
```bash
cp clientkey.pem /etc/pki/libvirt/private/clientkey.pem
cp clientcert.pem /etc/pki/libvirt/clientcert.pem
```



Dann konfigiurieren wir libvirt daemon um TLS zu nutzen:


```
sudo systemctl stop libvirtd
sudo systemctl start libvirtd-tls.socket
```

Jetzt kontrollieren wir ob die Verbindung geklappt hat: 

```bash
virsh -c qemu+tls://host35/system hostname
```

und bekommen folgendes zurück:

```bash
host35.nwlab
```

`host35.nwlab` ist die Antwort des anderen Hosts, die bestätigt, dass die Verbindung erfolgreich war.

