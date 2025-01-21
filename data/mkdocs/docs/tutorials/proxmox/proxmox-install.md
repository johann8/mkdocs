---
Title: Proxmox VE
Summary: Anleitung zur Installation von Proxmox VE
Authors: Johann Hahn
Date:    Januar 16, 2025
tags: [proxmox, kvm, qemu, vm, lxc]
---

# :fontawesome-brands-linux: Proxmox VE installation

Dieser Artikel beinhaltet eine Anleitung zu Installation und Einrichtung von [`Proxmox VE`][Proxmox VE]{target=\_blank}[^1]

[Proxmox VE]: https://de.wikipedia.org/wiki/Proxmox_VE

Proxmox Virtual Environment ist eine komplette Open Source-Virtualisierungsplattform für Server. Es kombiniert KVM- und Container-basierte Virtualisierung und verwaltet virtuelle Maschinen, Container, Storage, virtuelle Netzwerke und Hochverfügbarkeits-Cluster übersichtlich über die zentrale Managementoberfläche.

---

!!! info

    - Proxmox VE stellt zwei Virtualisierungstechnologien auf einer Plattform bereit <br>
    - Dies bietet maximale Flexibilität für die virtualisierte IT-Infrastruktur <br>
    - Verwenden Sie KVM für virtuelle Maschinen und Container für leichtgewichtige Linux-Anwendungen <br>

---

##### USB-Stck für die Installation von Proxmox VE vorbereiten

??? tip "USB-Stck vorbereiten"

    Porxmox VE ISO Installer kann man von [hier]{target=\_blank} downloaden. <br>
    Mit dem Tool [`Ventoy`][Ventoy]{target=\_blank}[^2] bereitet man den USB-Stick wie folgt vor:

    - Den leeren USB-Stick in ein Windows Rechner einstecken

    - Ventoy App starten, USB-Stick aus Dropdown-Liste wählen und auf `Install` klicken

    - Ein Ordner `tools` erstellen und die ISO-Datei in den Ordner kopieren

[hier]: https://www.proxmox.com/en/downloads
[Ventoy]: https://ventoy.net/en/download.html

##### Proxmox VE installieren

??? tip "Proxmox VE auf neuer Hardware installieren"
    
    USB-Stick ad die neue Hardware anschliessen und den Server starten. Über die Taste `F12` komt man ins `BIOS` und nimmt die notwendigen Einstellungen vor.  
    Auch hier kommt man ins BIOS vom `RAID-Controller`. Wir haben drei 4TB Festplatten an diesem RAID-Controller angeschlossen. Es wird ein `RAID1` mit einer `Hot-Spare-Festplatte` erstellt. Darauf wird Proxmox VE installiert.

    Über die Taste F11 (Supermicro Mainmoard) kommt man ins Boot-Menü und wählt den Boot von USB-Stick.
    Es erscheint kurz danach ein Menü und man wählt `Install Proxmox VE (Graphical)` aus. Die Installation von Proxmox VE wird von einem Installationsassistent begleitet.
    
    Als `Filesysten` wählt man `ext4` und als `Target Hardisk` `sda` und klickt auf die Taste `Options` <br>

    ![Proxmox VE install Image 1](../../assets/screenshots/pve_install_01.jpg "Choose target disk"){width=600} 

    Im nächste Fenster nehmen wir folgende Einstellungen vor:
 
    ```
    swapsize: 16GB  - Die Größe der `SWAP` Partition 
    maxroot:  70GB  - Die Größe der `root` Partition
    minfree:  16GB  - Freier Platz für LVM Snapshots

    ```
    ![Proxmox VE install Image 1](../../assets/screenshots/pve_install_02.jpg "Choose options"){width=600}

    Der Installationsassistent erstellt drei Partitonen: sda1=boot, sda2=UEFI und sda3=Linux LVM. Es werden drei LVs gemäß unseren Angaben angelegt: `LV Name: root` 70GB; `LV Name: swap` 16GB; 16GB werden frei gelassen und `LV Name: data` der Rest des Speicherplatzes als `Lokales Storage`. Nach der Installation über den Befehl `lvdisplay` kann man sich die Partitionierung genau anschauen.

    Wenn im Server mehrere Netzwerkkarten installiert sind, dann wird die aktive Netzwerkkarte mit einem grünen Punkt gekennzeichnet. Nach der erfolgreichen Installation startet der Server neu und es wird die URL angezeigt, über die man das Webinterface erreichen kann. 

##### Proxmox VE Netzwerk einrichten

??? tip "Netzwerk einrichten"

    Das Netzwerk richtet man entweder über das Proxmox VE WEB-Interface oder über Shell ein. Die Netzwerk Konfiguration befindet sich in der Datei `/etc/network/interfaces`. <br>
    Während der Installation richtet Proxmox automatisch ein `bridge interface` [`vmbr0`][vmbr0]{target=\_blank}. Man kann es sich als einen virtuellen Switch vorstellen, an den die Gäste und die physischen Schnittstellen angeschlossen sind.

    [vmbr0]: https://pve.proxmox.com/wiki/Network_Configuration

    Ein Beispiel für die Netzwerk-Konfiguration:

    ```bash
    # vim /etc/network/interfaces
    auto lo
    iface lo inet loopback

    iface eno1 inet manual

    iface eno2 inet manual

    #Intel Card Port oben
    iface eth0 inet manual

    #Intel Card Port unten
    iface eth1 inet manual

    #IPMI network card
    iface enxbe3af2b6059f inet manual

    auto vmbr0
    iface vmbr0 inet static
            address 192.168.18.225/24
            gateway 192.168.18.1
            bridge-ports eth0
            bridge-stp off
            bridge-fd 0

    # Link to PBS01
    auto vmbr1
    iface vmbr1 inet static
            address 10.0.20.2/24
            bridge-ports eth1
            bridge-stp off
            bridge-fd 0

    source /etc/network/interfaces.d/*
    ```
    Das zweite `bridge interface` `vmbr1` ist für die direkte Verbindung zum Storage Server PBS01 gedacht. <br>
    Wenn man manuelle Änderungen direkt in der Datei `/etc/network/interfaces` vorgenommen hat, kann man diese mit dem Befehl `ifreload -a` übernehmen. <br>
    Wenn man Proxmox VE auf Debian installiert hat oder von einer älteren Proxmox VE-Installation auf Proxmox VE ab Version 7.0 aktualisiert hat, muss man sicher stellen, dass `ifupdown2` installiert ist: `apt-get install ifupdown2`

    - Überprüfung der Konfiguration 

    ```bash
    # show route
    ip route list
    ----
    default via 192.168.18.1 dev vmbr0 proto kernel onlink
    10.0.20.0/24 dev vmbr1 proto kernel scope link src 10.0.20.2
    192.168.18.0/24 dev vmbr0 proto kernel scope link src 192.168.18.225
    ----

    # Restart networking service
    systemctl restart networking
    systemctl status networking

    # ip fowarding
    echo 1 > /proc/sys/net/ipv4/ip_forward
    cat /proc/sys/net/ipv4/ip_forward

    # Hosts file
    # vim /etc/hosts
    ----------------
    127.0.0.1 localhost.localdomain localhost
    192.168.18.225 pve01.wkt.local pve01
    192.168.18.23  rserver.domain.de rserver
    192.168.18.227  bareos-sd
    ----

    # vim resolv.conf
    ----
    search wkt.local int.wassermanngruppe.de
    nameserver 192.168.18.21
    nameserver 192.168.18.1
    ----
    ```

    - Debian sources

    ```bash
    # vim /etc/apt/sources.list
    ----
    deb http://ftp.de.debian.org/debian bookworm main contrib

    deb http://ftp.de.debian.org/debian bookworm-updates main contrib

    # security updates
    deb http://security.debian.org bookworm-security main contrib
    -----
    ```

    - Proxmox VE sources

    ```bash
    # vim /etc/apt/sources.list.d/pve-enterprise.list
    #deb https://enterprise.proxmox.com/debian/pve bookworm pve-enterprise

    # Proxmox no subscription
    deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription

    ```

    - Update Proxmox VE Installation

    ```bash

    apt-get update && apt-get dist-upgrade

    ```

[^1]: [Proxmox VE Homepage](https://www.proxmox.com/de/){target=\_blank}
[^2]: [Ventoy Homepage](https://ventoy.net/en/index.html){target=\_blank}

