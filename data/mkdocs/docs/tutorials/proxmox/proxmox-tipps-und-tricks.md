---
Title: Proxmox Tipps und Tricks
Summary: Proxmox Tipps und Tricks
Authors: Johann Hahn
Date:    Februar 03, 2025
tags: [proxmox, postfix, tips, ipmi, arcconf]
---

# :fontawesome-brands-linux: Proxmox VE Tipps und Tricks

Dieser Artikel beinhaltet eine Anleitung zur Einrichtung von [`Postfix`][Postfix]{target=\_blank}[^1] unter [`Proxmox VE`][Proxmox VE]{target=\_blank}[^2]

#### Proxmox VE - Postfix einrichten

??? tip "Postfix auf Proxmox VE Host einrichten"

    - `Postfix` installieren

    ```bash
    apt-get install postfix
    
    ```

    - `Postfix` konfigurieren

    === "main.cf"

        ```bash
        # vim /etc/postfix/main.cf
        ----
        ...
        # jhahn changed am 12.01.2025
        #mydestination = $myhostname, localhost.$mydomain, localhost
        #relayhost =
        ...
        # jhahn changed am 12.01.2025
        relayhost = [smtp.mydomain.de]:587
        smtp_use_tls = yes
        smtp_sasl_auth_enable = yes
        smtp_sasl_security_options = noanonymous
        smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
        smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
        # -------------- Generic maps
        smtp_generic_maps = hash:/etc/postfix/generic
        ...
        ----

        ```

    === "sasl_passwd"

        ```bash
        # vim /etc/postfix/sasl_passwd
        [smtp.mydomain.de]:587    helpdesk@mydomain.de:MySuperPassword123

        ```

    === "generic"

        ```bash
        # vim /etc/postfix/generic
        root@pve01.wkt.local        pve01@mydomain.de
        root@pbs.wkt.local          pbs@mydomain.de

        ```

[Postfix]: https://www.postfix.org/
[Proxmox VE]: https://de.wikipedia.org/wiki/Proxmox_VE

#### Proxmox VE - Intelligent Platform Management Interface (IPMI) einrichten

??? tip "IPMI auf Proxmox VE Host einrichten"

    [`IPMI`][IPMI]{target=\_blank}[^3]: ist eine standardisierte Schnittstelle in Computer-Hardware und -Firmware, über die Rechner auf Hardwareebene ferngesteuert überwacht und verwaltet werden können, auch wenn sie ausgeschaltet sind oder kein Betriebssystem installiert ist. Das Passwort für `ADMIN` User ist direkt auf dem `Motherboard` gespeichert. Mit dem Tool `IPMICFG` werden wir einen neuen User einrichten und das `Passwort` von User `ADMIN` ändern.

    - `IPMICFG` installieren: IPMICFG auf den Windows Rechner [`herunterladen`](https://www.supermicro.com/en/support/resources/downloadcenter/smsdownload){target=\_blank} und mit Hilfe von [`WinSCP`][WinSCP]{target=\_blank} nach `/tmp` auf den `Proxmox Host` kopieren.

    - SSH Login auf dem `Proxmox Host`

    ```bash
    cd /tmp 
    unzip IPMICFG_1.35.2_build.240627.zip
    cd /usr/local/bin
    mv /tmp/IPMICFG_1.35.2_build.240627/Linux/64bit/IPMICFG-Linux.x86_64 ./
    mv IPMICFG-Linux.x86_64 ipmicfg
    chmod 0700 ipmicfg
    ```

    - Neuen User `admin123` mit dem Tool `IPMICFG` hinzufügen

    ```bash
    # 3 - User-ID
    # 4 - privilege (Administrator)
    ipmicfg -user add 3 admin123 MySuperPassword135 4
    ipmicfg -user list
    ----
    Maximum number of Users          : 16
    Count of currently enabled Users : 2
    User ID | User Name        | Privilege Level | Enable
    ------- | ---------        | --------------- | ------
          2 | ADMIN            | Administrator   | Yes
          3 | adminJH          | Administrator   | Yes
    ----
    ```

    - Passwort vom User `ADMIN` ändern 

    ```bash
    # Set password for user ADMIN
    # 2 - User-ID
    ipmicfg -user setpwd 2 MySuperPassword789
    ```

    - Login über IPMI Web-Interface


    ```bash
    URL: https://192.168.xxx.254
    User: admin123
    PW: MySuperPassword135
    ```

    - Einige Befehle


    ```bash
    # show IP and MAC
    ipmicfg -m
    ----
    IP=192.168.xxx.254
    MAC=3C:EC:EF:E8:2C:11
    ----

    # show subnet mask
    ipmicfg -k
    ----
    Subnet Mask=255.255.255.0
    ----

    # show gateway 
    ipmicfg -g
    ----
    Gateway=192.168.xxx.1
    ----

    # show help
    ipmicfg -user add help
    -------------------
    Command(s):
     -user list                 Lists user privileges.
     -user help                 Shows a user privilege code.
     -user add <user id> <name> Adds a user.
      <password> <privilege>
     -user del <user id>        Deletes users.
     -user level <user id>      Updates user privileges.
      <privilege>
     -user setpwd <user id>     Updates a user password.
      <password>
    ----

    # show motherboard info
    dmidecode -t 2
    ```


[IPMI]: https://www.supermicro.com/en/support/resources/downloadcenter/smsdownload 
[WinSCP]: https://winscp.net/eng/download.php


#### Proxmox VE - ACCONF Command Line Utility

??? tip "Umgang mit dem ACCONF Command Line Utility"

    - Einige Beispiele

    ```bash

    # show all config
    arcconf getconfig 1

    # show controller status
    arcconf getconfig 1 | grep Status

    # controller information
    arcconf list

    # show drive status
    arcconf getconfig 1 pd |grep -iE "#|name|state|serial|firmware"

    # Logical device information
    arcconf getconfig 1 ld

    # Controller information
    arcconf getconfig 1 ad

    # Event log
    arcconf GETLOGS 1 EVENT tabular

    # show device status
    arcconf getconfig 1 pd|egrep "Device #|State\>|Reported Location|Reported Channel|S.M.A.R.T. warnings"

    # show array status
    arcconf getconfig 1 PD | grep -i 'state\|Array'

    # show smart staus of drives
    arcconf getsmartstats 1  tabular|grep -i SmartStats -A 5

    ```


[^1]: [Postfix](https://de.wikipedia.org/wiki/Postfix_(Mail_Transfer_Agent)){target=\_blank}
[^2]: [Proxmox VE Homepage](https://www.proxmox.com/de/){target=\_blank}
[^3]: [IPMI](https://de.wikipedia.org/wiki/Intelligent_Platform_Management_Interface){target=\_blank}
