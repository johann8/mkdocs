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

Proxmox VE stellt zwei Virtualisierungstechnologien auf einer Plattform bereit. Dies bietet maximale Flexibilität für die virtualisierte IT-Infrastruktur. Verwenden Sie KVM für virtuelle Maschinen und Container für leichtgewichtige Linux-Anwendungen.

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

##### Proxmox VE Netzwerk einrichten

??? tip "Netzwerk einrichten"


[^1]: [Proxmox VE Homepage](https://www.proxmox.com/de/){target=\_blank}
[^2]: [Ventoy Homepage](https://ventoy.net/en/index.html){target=\_blank}

