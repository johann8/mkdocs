---
Title: Proxmox VE - maxView Storage Manager einrichten
Summary: Anleitung zur Einrichtung von maxView Storage Manager unter Proxmox VE
Authors: Johann Hahn
Date:    Februar 03, 2025
tags: [proxmox, mdadm, raid, lvm]
---

# :fontawesome-brands-linux: Proxmox VE - maxView Storage Manager einrichten

Dieser Artikel beinhaltet eine Anleitung zur Einrichtung von [`maxView Storage Manager`][maxView Storage Manager]{target=\_blank}[^1] unter [`Proxmox VE`][Proxmox VE]{target=\_blank}[^2]

Als [`RAID controller`][RAID controller]{target=\_blank} dient im Proxmox Host ein Controller von [`Adaptec`][Adaptec]{target=\_blank}.

##### Adaptec RAID controler

??? tip "Adaptec RAID controler anzeigen lassen"

    ```bash
    lspci -v |grep Adaptec
    ----
    c7:00.0 Serial Attached SCSI controller: Adaptec Smart Storage PQI SAS (rev 01)
            Subsystem: Adaptec SmartRAID 3102-8i
    ----
    ```
##### maxView Storage Manager

??? tip "maxView Storage Manager installieren"

    - Software installieren

    ```bash
    apt-get update && apt-get install unzip libpam-cracklib
    ```

    - maxView Storage Manager auf den Windows Rechner [`herunterladen`](https://storage.microsemi.com/en-us/downloads/storage_manager/sm/productid=asr-3102-8i&dn=microsemi+adaptec+smartraid+3102-8i.php){target=\_blank} und mit Hilfe von [`WinSCP`][WinSCP]{target=\_blank} nach `/tmp` auf den `Proxmox Host` kopieren.

    - SSH Login auf dem `Proxmox Host`

    ```bash
    cd /tmp
    mkdir 33
    mv msm_linux_x64_B27147.tgz 33/
    cd 33
    tar -xzvf msm_linux_x64_B27147.tgz
    ```
    - Alte Version deinstallieren, falls existiert

    ```bash
    apt-get purge storman

    ```

    - maxView Storage Manager installieren

    ```bash     
    dpkg -i dpkg -i StorMan-4.23-27147_amd64.deb
    ----
    Do you want to install maxView in Standalone Mode: [default: no]: enter
    yes
    ----

    # check service
    netstat -tulpan |grep 84

    cd /usr/local/bin/
    ln -s /usr/StorMan/arcconf ./arcconf

    # log file
    journalctl --since=today

    cat /usr/StorMan/config/server.properties
    ----
    #Tue Jan 14 11:51:46 CET 2025
    cimomip=127.0.0.1
    comm_protocol=http
    consoleonly=false
    desktop_application=false
    event_listener_port=49161
    event_protocol=http
    interop_namespace=root/interop
    localip=
    localmode=false
    redfish_event_listener_port=49162
    vmware_mode=false
    webserverport=8443
    ----

    # Ob die Dienste laufen
    systemctl list-units -t service --state=running
    systemctl list-units -t service 

    # Steuerung der Diensten
    /etc/init.d/stor_redfishserver status   # (start|stop|restart|status)
    /etc/init.d/stor_tomcat status          # (start|stop|restart|status)

    # Web Login
    https://192.168.18.225:8443/maxview/manager/login.xhtml

    ```


[maxView Storage Manager]: https://developerhelp.microchip.com/xwiki/bin/view/products/data-center-solutions/adapatec-storage-adapters/maxview-storage-manager/
[Proxmox VE]: https://de.wikipedia.org/wiki/Proxmox_VE
[RAID controller]: https://en.wikipedia.org/wiki/Disk_array_controller
[Adaptec]: https://en.wikipedia.org/wiki/Adaptec
[WinSCP]: https://winscp.net/eng/download.php 


[^1]: [maxView Storage Manager](https://storage.microsemi.com/en-us/support/raid/sas_raid/asr-3102-8i/){target=\_blank}
[^2]: [Proxmox VE Homepage](https://www.proxmox.com/de/){target=\_blank}
