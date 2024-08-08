---
Title: Änderung des Partitionslayouts unter Linux
Summary: Änderung des Partitionslayouts unter Linux
Authors: Johann Hahn
Date:    Juli 31, 2024
tags: [partition, layout, lvm, parted]
---

# :fontawesome-brands-linux: Änderung des Partitionslayouts

#### Einführung 

Der Mail Server wird bei [`Contabo`][Contabo][^1] gehostet. Die Standardinstallation erstellt sehr einfaches Partitionlayout und zwar:

[Contabo]: https://contabo.com/de/

```bash
lsblk /dev/sda
---
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  800G  0 disk
├─sda1   8:1    0 1000M  0 part /boot
└─sda2   8:2    0  799G  0 part /
---
```
Das Vorhaben ist das Partitionslayout so zu ändern, damit man mehr Flexibilität erreichen kann. Einzelne Schritte sind: 

1. Die ISO `GParted_Live`[^2] auf den Mail Server laden und `GRUB`[^3] Konfiguration anpassen

2. Den Server über [`VNC`][VNC]{target=\_blank} von `GParted_Live` starten

3. Die Partition `sda2` auf 100GB zu verkleinern

4. [`LVM`][LVM]{target=\_blank} (Logical Volume Manager) Partition anlegen

5. Zwei `Logical Volumes` (LV) erstellen und an `/mnt/newopt` und `/mnt/newvar` anbinden

6. Den Server in [`Runlevel`][Runlevel]{target=\_blank} `init 1` starten

7. Die Daten von `/opt` and `/var` auf `/mnt/newopt` und `/mnt/newvar` verschieben

8. Die `fstab` anpassen

9. Server neu starten und testen

[VNC]: https://de.wikipedia.org/wiki/Virtual_Network_Computing
[LVM]: https://de.wikipedia.org/wiki/Logical_Volume_Manager 
[Runlevel]: https://de.wikipedia.org/wiki/Runlevel



??? tip "Änderung des Partitionslayouts unter Linux."

    #### `GParted Live` on Hard Disk using `GRUB`
    
    ```bash
    mkdir -p /boot/iso
    cd /boot/iso

    # Download last `Gparted Live` ISO file
    wget https://ftp.upjs.sk/pub/mirrors/gparted/gparted-live-stable/1.6.0-3/gparted-live-1.6.0-3-amd64.iso

    # Add to file `/etc/grub.d/40_custom`
    vim /etc/grub.d/40_custom
    ---
    ...
    menuentry "Gparted Live" {
      set isofile="/iso/gparted-live-1.6.0-3-amd64.iso"
      loopback loop $isofile
      linux (loop)/live/vmlinuz boot=live config union=overlay username=user components noswap noeject vga=788 ip= net.ifnames=0 toram=filesystem.squashfs findiso=$isofile
      initrd (loop)/live/initrd.img
    }
    ---

    # Update `GRUB` config
    grub2-mkconfig -o /boot/grub2/grub.cfg
    
    # UEFI-based Centos/RHEL server
    grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg

    # Reboot Server über VNC-Console
    reboot

    ```
    Über VNC Console starten von `Gparted Live` und die Partition `sda2` auf 100GB reduzieren.

    #### LVM Partition und LV's erstellen

    ```bash
    # Aktuelle Konfiguration anzeigen lassen
    parted -a optimal /dev/sda print
    ---
    Model: QEMU QEMU HARDDISK (scsi)
    Disk /dev/sda: 859GB
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags:

    Number  Start   End     Size    Type     File system  Flags
     1      1049kB  1050MB  1049MB  primary  ext4         boot
     2      1050MB  111GB   110GB   primary  ext4
    ---

    # LVM Partition erstellen
    parted -a optimal /dev/sda

    (parted) print free
    ---
    Model: QEMU QEMU HARDDISK (scsi)
    Disk /dev/sda: 859GB
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags:

    Number  Start   End     Size    Type     File system  Flags
            32.3kB  1049kB  1016kB           Free Space
     1      1049kB  1050MB  1049MB  primary  ext4         boot
     2      1050MB  111GB   110GB   primary  ext4
            111GB   859GB   748GB            Free Space
    ---

    (parted) mkpart
    Partition type?  primary/extended? primary
    File system type?  [ext2]? ext4
    Start? 111GB
    End? 859GB

    (parted) print free
    Model: QEMU QEMU HARDDISK (scsi)
    Disk /dev/sda: 859GB
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags:

    Number  Start   End     Size    Type     File system  Flags
            32.3kB  1049kB  1016kB           Free Space
     1      1049kB  1050MB  1049MB  primary  ext4         boot
     2      1050MB  111GB   110GB   primary  ext4
     3      111GB   859GB   748GB   primary  ext4         lba

    (parted) set
    Partition number? 3
    Flag to Invert? lvm
    New state?  [on]/off? on

    (parted) set
    Partition number? 3
    Flag to Invert? lba
    New state?  on/[off]? off

    (parted) print
    Model: QEMU QEMU HARDDISK (scsi)
    Disk /dev/sda: 859GB
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags:

    Number  Start   End     Size    Type     File system  Flags
     1      1049kB  1050MB  1049MB  primary  ext4         boot
     2      1050MB  111GB   110GB   primary  ext4
     3      111GB   859GB   748GB   primary  ext4         lvm

     (parted) quit
 
    # Layout anzeigen lassen
    lsblk
    ---
    NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda      8:0    0   800G  0 disk
    ├─sda1   8:1    0  1000M  0 part /boot
    ├─sda2   8:2    0 102.6G  0 part /
    └─sda3   8:3    0 696.5G  0 part
    ---

    # PV (Physical Volume) erstellen
    pvcreate -v  /dev/sda3
    pvs
    pvdisplay
    ---
    "/dev/sda3" is a new physical volume of "696.48 GiB"
    --- NEW Physical volume ---
    PV Name               /dev/sda3
    VG Name
    PV Size               696.48 GiB
    Allocatable           NO
    PE Size               0
    Total PE              0
    Free PE               0
    Allocated PE          0
    PV UUID               wccyM3-qvNK-e0cq-bvdJ-S55p-DQOS-eooHVB
    ---

    # VG (Volume Group) erstellen
    vgcreate rl_mx01 /dev/sda3
    vgs
    vgdisplay
    --- Volume group ---
    VG Name               rl_mx01
    System ID
    Format                lvm2
    Metadata Areas        1
    Metadata Sequence No  1
    VG Access             read/write
    VG Status             resizable
    MAX LV                0
    Cur LV                0
    Open LV               0
    Max PV                0
    Cur PV                1
    Act PV                1
    VG Size               696.48 GiB
    PE Size               4.00 MiB
    Total PE              178299
    Alloc PE / Size       0 / 0
    Free  PE / Size       178299 / 696.48 GiB
    VG UUID               h8tZeA-8kD0-xc4h-F052-mlQ3-EueX-05s3HV
    ---

    # LV (Logical Volume) erstellen
    # Create LV swap 
    lvcreate -n swap -L 4GB rl_mx01

    # Create LV opt
    lvcreate -n opt -L 10GB rl_mx01

    # Create LV var
    lvcreate -n var -L 200GB rl_mx01

    # LV Layout anzeigen lassen
    lvs
    lvdisplay
    --- Logical volume ---
    LV Path                /dev/rl_mx01/swap
    LV Name                swap
    VG Name                rl_mx01
    LV UUID                XJj30y-JUNd-p6f9-H6T3-ErSB-Rd9R-xvTlAu
    LV Write Access        read/write
    LV Creation host, time mx01.myfirma.de, 2024-08-05 12:11:58 +0200
    LV Status              available
    # open                 0
    LV Size                4.00 GiB
    Current LE             1024
    Segments               1
    Allocation             inherit
    Read ahead sectors     auto
    - currently set to     8192
    Block device           253:0

    --- Logical volume ---
    LV Path                /dev/rl_mx01/opt
    LV Name                opt
    VG Name                rl_mx01
    LV UUID                ziEhqh-wGrE-rwmf-uKtT-kfyd-QBtu-tJIsFr
    LV Write Access        read/write
    LV Creation host, time mx01.myfirma.de, 2024-08-05 12:13:45 +0200
    LV Status              available
    # open                 0
    LV Size                10.00 GiB
    Current LE             2560
    Segments               1
    Allocation             inherit
    Read ahead sectors     auto
    - currently set to     8192
    Block device           253:1

    --- Logical volume ---
    LV Path                /dev/rl_mx01/var
    LV Name                var
    VG Name                rl_mx01
    LV UUID                uxXGwO-Mq47-AHTI-oqao-qfM0-S61u-Tkptom
    LV Write Access        read/write
    LV Creation host, time mx01.myfirma.de, 2024-08-05 12:13:59 +0200
    LV Status              available
    # open                 0
    LV Size                200.00 GiB
    Current LE             51200
    Segments               1
    Allocation             inherit
    Read ahead sectors     auto
    - currently set to     8192
    Block device           253:2
    ---

    #
    lsblk
    ---
    NAME             MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda                8:0    0   800G  0 disk
    ├─sda1             8:1    0  1000M  0 part /boot
    ├─sda2             8:2    0 102.6G  0 part /
    └─sda3             8:3    0 696.5G  0 part
      ├─rl_mx01-swap 253:0    0     4G  0 lvm  [SWAP]
      ├─rl_mx01-opt  253:1    0    10G  0 lvm  /mnt/newopt
      └─rl_mx01-var  253:2    0   200G  0 lvm  /mnt/newvar
    ---

    # FS erstellen
    mkfs.ext4 /dev/rl_mx01/opt
    mkfs.ext4 /dev/rl_mx01/var
    ```
    #### `SWAP` erstellen und anbinden

    ```bash
    # Zeigt an, ob eine Swap-Partition oder ein Swap-File existiert
    free -h

    # Ort der Swap-Datei herausfinden 
    swapon -show

    # Swap abschalten
    swapoff -a

    # LV Volume zum Swap deklarieren
    mkswap /dev/rl_mx01/swap

    # Swap einschalten
    swapon /dev/rl_mx01/swap

    # Das Ergebnis anzeigen lassen
    free -h
    swapon -show
    ```

    #### Mount Partition and specify Mount Point as `/mnt/newvar` and `/mnt/newopt`

    ```bash
    # Ordner erstellen
    mkdir -p /mnt/newvar
    mkdir -p /mnt/newopt

    # fstab anpassen
    vim /etc/fstab
    ---
    ...
    /dev/mapper/rl_mx01-swap  none                           swap     defaults        0 0
    /dev/mapper/rl_mx01-var   /mnt/newvar                    ext4     defaults        1 0
    /dev/mapper/rl_mx01-opt   /mnt/newopt                    ext4     defaults        1 0
    ---
    ```
    #### Über VNC reboot Server in runlevel 1

    ```bash
    init 1
    ```

    #### Copy the data in var only to the new mounted filesystem

    ```bash
    rsync -aHhP /opt/ /mnt/newopt/
    rsync -aHhP /var/ /mnt/newvar/
    ```

    #### Rename the current `/var` and `/opt` directory for backup purposes:

    ```bash
    mv /opt /opt.old
    mv /var /var.old
    ```

    #### Make the new `var` and `opt` directory
    ```bash
    mkdir /opt
    mkdir /var
    ```

    #### Edit the `/etc/fstab` file

    ```bash

    vim /etc/fstab
    ---
    ...
    /dev/mapper/rl_mx01-swap  none                    swap     defaults        0 0
    /dev/mapper/rl_mx01-var   /var                    ext4     defaults        1 0
    /dev/mapper/rl_mx01-opt   /opt                    ext4     defaults        1 0
    ---
    ```

    !!! note ":bulb: **Note:** The new partitions are now configured to be mounted at `/var` and `/opt` on boot."

    #### Restart the server:

    ```bash
    reboot
    ```
    #### Den Mail Server testen

    Nach dem Reboot mussen das neue Partitionslayout und die Funktion des Mails Servers getestet werden. Über irgendein E-Mail Client prüfen, ob alte E-Mails da sind.

    ```bash
    # Partitionslayout anzeigen lassen
    lsblk
    ---
    NAME             MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda                8:0    0   800G  0 disk
    ├─sda1             8:1    0  1000M  0 part /boot
    ├─sda2             8:2    0 102.6G  0 part /
    └─sda3             8:3    0 696.5G  0 part
      ├─rl_mx01-swap 253:0    0     4G  0 lvm  [SWAP]
      ├─rl_mx01-opt  253:1    0    10G  0 lvm  /opt
      └─rl_mx01-var  253:2    0   200G  0 lvm  /var
    ---

    # Belegung der Partitionen anzeigen lassen
    df -hT
    ---
    Filesystem                     Type      Size  Used Avail Use% Mounted on
    ...
    /dev/sda2                      ext4      101G   42G   55G  43% /
    /dev/sda1                      ext4      966M  662M  239M  74% /boot
    /dev/mapper/rl_mx01-var        ext4      196G   78G  109G  42% /var
    /dev/mapper/rl_mx01-opt        ext4      9.8G  361M  8.9G   4% /opt
    ...
    ---
    ```

    #### Prüfen, ob alle Docker Container laufen

    === "docker compose (Plugin)"

        ```bash
        cd /opt/mailcow-dockerized
        docker compose up -d

        # Zeigt Status an
        docker compose ps

        # Zeigt Logdaten an
        docker compose logs -f
        ```

    === "docker-compose (Standalone)"

        ```bash
        cd /opt/mailcow-dockerized
        docker-compose up -d

        # Zeigt Status an
        docker-compose ps

        # Zeigt Logdaten an
        docker-compose logs -f
        ```
    ####  Mailversand testen

    Über irgendeinen E-Mail Client E-Mails an verschiedene Empfänger senden und empfangen. Dabei muss man die Logs auf eventuellen Fehler überwachen.

    #### Aufräumen nach den Änderungen

    ```bash
    # Alte `var` und `opt` Verzeichnisse löschen 
    df -hT
    rm -rf /opt.old
    rm -rf /var.old
    df -hT
    ```
    #### Partition `sda2` reduzieren, Partition `sda3` vergrössern 
    
    Nach dem Entfernen der Verzeichnissen `var` und `opt` ist die Partition `sda2` sehr groß und man kann sie wieder mit `GParted_Live` um 50GB reduzieren und die  Partition `sda3` um diese 50GB vergrösern. Dazu muss man wieder über VNC Console von `Gparted Live` starten  und die besagten Änderungen vornehmen.


[^1]: [Contabo](https://contabo.com/de/){target=\_blank} 
[^2]: [GParted Live - Grub](https://gparted.org/livehd.php#live-hd-grub){target=\_blank}
[^3]: [Wikipedia - Grub](https://de.wikipedia.org/wiki/Grand_Unified_Bootloader){target=\_blank}
