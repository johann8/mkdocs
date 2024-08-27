---
Title: LVM Partition vergrößern
Summary: LVM Partition vergrößern
Authors: Johann Hahn
Date:    Juli 31, 2024
tags: [partition, lvm, parted]
---

# :fontawesome-brands-linux: LVM Partition vergrößern

Dieser Artikel beinhaltet eine Anleitung für das Vergrößern eines [`LVM`][LVM]{target=\_blank} am Beispiel einer Rocky Linux 8 Proxmox VM. Bei anderen Linux-Distributionen sollte dieses HOWTO ähnlich funktionieren.

[LVM]: https://de.wikipedia.org/wiki/Logical_Volume_Manager

##### Virtual Disk unter ProxMox ändern

Login unter `Proxmox -> pve01 -> 202 (RockyLinux) -> Hardware -> Hard Disk (scsi1) -> Sisk Action -> Resize` und gewünschte Zahl eingeben. In diesem Beispiel sind es 10GB.

##### Konfiguration anzeigen

Die geänderte Konfiguration des LVM (Logical Volume Manager) ist in diesem Beispiel dabei wie folgt:

```bash
lsblk
---
NAME              MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda                 8:0    0  25G  0 disk
├─sda1              8:1    0   1G  0 part /boot
├─sda2              8:2    0   2G  0 part [SWAP]
└─sda3              8:3    0  22G  0 part /
sdb                 8:16   0  50G  0 disk
├─sdb1              8:17   0  20G  0 part /var
└─sdb2              8:18   0  20G  0 part
  └─rl_docker-opt 253:0    0  12G  0 lvm  /opt


pvdisplay
---
  --- Physical volume ---
  PV Name               /dev/sdb2
  VG Name               rl_docker
  PV Size               20,00 GiB / not usable 4,00 MiB
  Allocatable           yes
  PE Size               4,00 MiB
  Total PE              5119
  Free PE               2047
  Allocated PE          3072
  PV UUID               m2I0iE-Bl1X-2fkb-FNmu-M2qT-FI5y-7avWhw


parted /dev/sdb print free
---
Modell: QEMU QEMU HARDDISK (scsi)
Festplatte  /dev/sdb:  53,7GB
Sektorgröße (logisch/physisch): 512B/512B
Partitionstabelle: msdos
Disk-Flags:

Nummer  Anfang  Ende    Größe   Typ      Dateisystem   Flags
        1024B   1049kB  1048kB           Freier Platz
 1      1049kB  21,5GB  21,5GB  primary  xfs
 2      21,5GB  42,9GB  21,5GB  primary                lvm
        42,9GB  53,7GB  10,7GB           Freier Platz
```

##### Partition sdb2 vergrößern

Die Partition wird mit Hilfe von Linux Programm [`parted`][parted]{target=\_blank} vergrößert.

[parted]: https://www.gnu.org/software/parted/manual/parted.html

```bash
parted -a optimal /dev/sdb
---
GNU Parted 3.2
/dev/sdb wird verwendet
Willkommen zu GNU Parted! Rufen Sie »help« auf, um eine Liste der verfügbaren Befehle zu erhalten.
(parted) print free
Modell: QEMU QEMU HARDDISK (scsi)
Festplatte  /dev/sdb:  53,7GB
Sektorgröße (logisch/physisch): 512B/512B
Partitionstabelle: msdos
Disk-Flags:

Nummer  Anfang  Ende    Größe   Typ      Dateisystem   Flags
        1024B   1049kB  1048kB           Freier Platz
 1      1049kB  21,5GB  21,5GB  primary  xfs
 2      21,5GB  42,9GB  21,5GB  primary                lvm
        42,9GB  53,7GB  10,7GB           Freier Platz

(parted) resizepart
Partitionsnummer? 2
Ende?  [42,9GB]? 53,7GB
(parted) print free
Modell: QEMU QEMU HARDDISK (scsi)
Festplatte  /dev/sdb:  53,7GB
Sektorgröße (logisch/physisch): 512B/512B
Partitionstabelle: msdos
Disk-Flags:

Nummer  Anfang  Ende    Größe   Typ      Dateisystem   Flags
        1024B   1049kB  1048kB           Freier Platz
 1      1049kB  21,5GB  21,5GB  primary  xfs
 2      21,5GB  53,7GB  32,2GB  primary                lvm

(parted) quit
```

##### PV erweitern

PV (Physical Volume) auf die maximale verfügbare Größe erweitern.

```bash
pvresize /dev/sdb2


pvdisplay
---
  --- Physical volume ---
  PV Name               /dev/sdb2
  VG Name               rl_docker
  PV Size               <30,00 GiB / not usable 3,00 MiB
  Allocatable           yes
  PE Size               4,00 MiB
  Total PE              7679
  Free PE               4607
  Allocated PE          3072
  PV UUID               m2I0iE-Bl1X-2fkb-FNmu-M2qT-FI5y-7avWhw


pvs
---
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sdb2  rl_docker lvm2 a--  <30,00g <18,00g


vgs
---
  VG        #PV #LV #SN Attr   VSize   VFree
  rl_docker   1   1   0 wz--n- <30,00g <18,00g
```

##### LV erweitern

LV (Logical Volume) `/dev/mapper/rl_docker-opt` um 2GB erweitern.

```bash
lvextend -L +2G /dev/mapper/rl_docker-opt
```

##### Das Ergebnis anzeigen

```bash
lvdisplay /dev/mapper/rl_docker-opt

lvs
```
##### Verwendetes Dateisystem anzeigen

Mit Hilfe des Befehls `df -hT` kann man angebundene Dateisysteme anzeigen lassen.

```bash
df -hT
---
Dateisystem                           Typ      Größe Benutzt Verf. Verw% Eingehängt auf
...
/dev/mapper/rl_docker-opt             ext4       12G    1,7G  9,5G   15% /opt
...
```
##### Vergrößern des Dateisystems

Um den zusätzlichen Speicherplatz auch nutzen zu können, muss abschließend noch das Dateisystem mit dem Befehl [`resize2fs`][resize2fs]{target=\_blank} vergrößert werden. In unserem Beispiel wird ext4 als Dateisystem verwendet, welches diese Vergrößerung (auch im eingehängten Zustand) problemlos unterstützt (ext2, ext3, ext4).

[resize2fs]: https://linux.die.net/man/8/resize2fs

```bash
resize2fs -p lvdisplay /dev/mapper/rl_docker-opt
```
Falls die Vergrößerung eines LVs mit `resize2fs -p /dev/mapper/rl_docker-opt` nicht funktionieren sollte, kann alternativ folgender Befehl verwendet werden.

```bash
lvextend --resizefs -l +100%FREE /dev/mapper/rl_docker-opt
```
Bei einem XFS-Dateisystem können Sie mit dem Befehl [`xfs_growfs`][xfs_growfs]{target=\_blank} das Dateisystem vergrößern.

[xfs_growfs]: https://linux.die.net/man/8/xfs_growfs

```bash
xfs_growfs /dev/mapper/<lv-name>
```
