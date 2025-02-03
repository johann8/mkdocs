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

    ```bash
    # vim /etc/postfix/
    
    ```

[Postfix]: https://www.postfix.org/
[Proxmox VE]: https://de.wikipedia.org/wiki/Proxmox_VE


[^1]: [Postfix](https://de.wikipedia.org/wiki/Postfix_(Mail_Transfer_Agent)){target=\_blank}
[^2]: [Proxmox VE Homepage](https://www.proxmox.com/de/){target=\_blank}

