---
Title: Neue HDD zu LVM hinzufügen
Summary: Neue HDD zu LVM hinzufügen
Authors: Johann Hahn
Date:    August 28, 2024
tags: [lvm, Festplatte]
---

# :fontawesome-brands-linux: Neue HDD zu LVM hinzufügen

Dieser Artikel beinhaltet eine Anleitung für das Hinzufügen[^1] neuer `HDD | VD` (Virtual Disk) zu [`LVM`][LVM]{target=\_blank}, Verschiebung der Daten auf die neue Festplatte und die Entfernung der alten Festplatte. Dies ist z.B. notwendig, wenn die alte Festplatte Probleme aufweist. Oder wenn man nicht genügend freie [`Physische Extents`][Physische Extents][LVM]{target=\_blank} hat, muss man der Volume Group einen Datenträger hinzufügen und die `Extents` dorthin verschieben.

[LVM]: https://de.wikipedia.org/wiki/Logical_Volume_Manager
[Physische Extents]: https://de.wikipedia.org/wiki/Logical_Volume_Manager
##### Vorbereiten der Festplatte

Die `Festplatte | VD` muss angeschlossen / erstellt werden. Danach mit Hilfe des Befehls `lsblk` ermittellt man die Bezeichnung.

```bash
lsblk
---

```

##### Hinzufügen zum Physical Volume

Zunächst müssen Sie die neue Festplatte mit Hilfe des Befehls `pvcreate` vorbereiten, um sie für LVM verfügbar zu machen. In diesem Beispiel wird die gesamte Festplatte zu LVM hinzugefügt ohne dass man sie vorher partitioniert.

```bash
pvcreate /dev/sdf
---
pvcreate -- physical volume "/dev/sdf" successfully created
```

##### Hinzufügen zur Volume Group

Nun wird die Festplatte zur existierenden `Volume Group` ninzugefügt.

```bash
vgextend dev /dev/sdf
---
vgextend -- INFO: maximum logical volume size is 255.99 Gigabyte
vgextend -- doing automatic backup of volume group "dev"
vgextend -- volume group "dev" successfully extended
```

##### Verschieben der Daten

Als Nächstes verschiebt man die Daten von der alten Platte auf die neue. Beachten Sie, dass es nicht notwendig ist, das Dateisystem vor diesem Vorgang auszuhängen. Es ist jedoch *sehr* empfehlenswert, vor diesem Vorgang ein vollständiges Backup zu erstellen, für den Fall, dass der Strom ausfällt oder ein anderes Problem auftritt, das den Vorgang unterbrechen könnte. Der Befehl `pvmove` kann eine beträchtliche Zeit in Anspruch nehmen und belastet die beiden Volumes, so dass es ratsam ist, diesen Vorgang durchzuführen, wenn die Volumes nicht zu stark ausgelastet sind, auch wenn dies nicht notwendig ist. 

```bash
pvmove /dev/hdb /dev/sdf
---
pvmove -- moving physical extents in active volume group "dev"
pvmove -- WARNING: moving of active logical volumes may cause data loss!
pvmove -- do you want to continue? [y/n] y
pvmove -- 249 extents of physical volume "/dev/hdb" successfully moved
```

##### Entfernen der unbenutzter Festplatte

Man kann nun die alte Festplatte aus der Volume Gruppe entfernen[^1].

```bash
vgreduce dev /dev/hdb
---
vgreduce -- doing automatic backup of volume group "dev"
vgreduce -- volume group "dev" successfully reduced by physical volume:
vgreduce -- /dev/hdb

```
Die Festplatte kann nun entweder physisch entfernt werden, wenn der Server das nächste Mal ausgeschaltet wird, oder sie kann anderweitig eingesetzt werden.

[^1]: [Add / Remove Disk to LVM](https://tldp.org/HOWTO/LVM-HOWTO/removeadisk.html){target=\_blank}
