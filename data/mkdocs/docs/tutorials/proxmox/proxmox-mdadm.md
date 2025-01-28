---
Title: Proxmox VE - Software RAID einrichten
Summary: Anleitung zur Einrichtung von Software RAID unter Proxmox VE
Authors: Johann Hahn
Date:    Januar 16, 2025
tags: [proxmox, mdadm, raid, lvm]
---

# :fontawesome-brands-linux: Proxmox VE - Software RAID einrichten

Dieser Artikel beinhaltet eine Anleitung zur Einrichtung von Software RAID [`MDADM`][MDADM]{target=\_blank}[^2] unter [`Proxmox VE`][Proxmox VE]{target=\_blank}[^1]

Als `Storage` für die VMs wird ein [`RAID1`][RAID1] aus zwei `Enterprise-SSDs` [`KIOXIA`][KIOXIA]{target=\_blank}[^3] 3.2TB erstellt. Danach wird ein [`LVM`][LVM]{target=\_blank} [`thin pool`][thin pool]{target=\_blank} erstellt.


[Proxmox VE]: https://de.wikipedia.org/wiki/Proxmox_VE
[MDADM]: https://de.wikipedia.org/wiki/Mdadm
[RAID1]: https://de.wikipedia.org/wiki/RAID#RAID_1:_Mirroring_%E2%80%93_Spiegelung 
[KIOXIA]: https://en.wikipedia.org/wiki/Kioxia
[LVM]: https://de.wikipedia.org/wiki/Logical_Volume_Manager
[thin pool]: https://pve.proxmox.com/wiki/Storage:_LVM_Thin

[^1]: [Proxmox VE Homepage](https://www.proxmox.com/de/){target=\_blank}
[^2]: [MDADM RAID](https://de.wikipedia.org/wiki/Mdadm){target=\_blank}
[^3]: [KIOXIA KCD8XVUG3T20](https://europe.kioxia.com/de-de/business/ssd/data-center-ssd/cd8-v.html){target=\_blank}
