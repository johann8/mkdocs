---
Title: Proxmox Tipps und Tricks
Summary: Proxmox Tipps und Tricks
Authors: Johann Hahn
Date:    Februar 03, 2025
tags: [proxmox, postfix, tipps, tricks]
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


[^1]: [Postfix](https://de.wikipedia.org/wiki/Postfix_(Mail_Transfer_Agent)){target=\_blank}
[^2]: [Proxmox VE Homepage](https://www.proxmox.com/de/){target=\_blank}

