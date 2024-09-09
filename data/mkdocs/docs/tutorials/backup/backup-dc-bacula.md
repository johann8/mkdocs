---
Title: Backup Docker Container mit Bacula
Summary: Anleitung für Backup Docker Container mit Bacula
Authors: Johann Hahn
Date:    September 05, 2024
tags: [backup, docker, container, lvm, snapshot, bacula]
---

# :fontawesome-brands-linux: Backup Docker Container mit TAR

Dieser Artikel beinhaltet eine Anleitung für Backup der `Docker Container` mit Hilfe von [`LVM`][LVM][^1]{target=\_blank} Snapshot und [`Bacula`][Bacula]][^2]{target=\_blank}

[LVM]: https://de.wikipedia.org/wiki/Logical_Volume_Manager
[Bacula]: https://de.wikipedia.org/wiki/Bacula

##### LVM Snapshot

??? tip "Script um LVM Snapshot erstellen, anbinden und löschen"

    === "Bash Script installieren"
        ```bash
        DOCKER_DIR=/opt/bacularis
        cd ${DOCKER_DIR}
        wget https://raw.githubusercontent.com/johann8/bacularis-alpine/master/scripts/container_backup_before_after.sh -O /opt/bacula/scripts/script_before_after.sh
        chmod a+x /opt/bacula/scripts/script_before_after.sh
        
        # Script anpassen
        vim /opt/bacula/scripts/script_before_after.sh

        ```

    === "backupLVS.sh"
        ```bash
        
        
        ```

##### Job aus Template hinzufügen

??? tip "Bacula Job hinzufügen"

    === "Job erstellen, wenn Alpine Docker Container (DC) benutzt wird"
        ```bash
        cd /tmp
        wget https://raw.githubusercontent.com/johann8/bacularis-alpine/master/add_job_to_backup_docker_container_template 

        # Set the Name of bacula client
        B_CLIENT=rockyl8
        sed -i -e "s/{{ B_CLIENT }}/${B_CLIENT}/g" /tmp/bacula_template
        less /tmp/bacula_template

        # copy template to bacula-dir config
        cat /tmp/bacula_template >> /opt/bacularis/data/bacula/config/etc/bacula-dir.conf
        rm -rf /tmp/bacula_template        

        # Restart Docker Container
        DOCKER_DIR=/opt/bacularis
        cd ${DOCKER_DIR}
        docker-compose up -d --force-recreate
        docker-compose ps
        docker-compose logs
        docker-compose logs bacularis

        ```

??? tip "Bacula Job hinzufügen"

    === "Job erstellen, wenn Ubuntu Docker Container (DC) benutzt wird"
        ```bash
        cd /tmp
        wget https://raw.githubusercontent.com/johann8/bacularis-alpine/master/add_job_to_backup_docker_container_template -O /tmp/bacula_template

        # Set the Name of bacula client
        B_CLIENT=rockyl8
        sed -i -e "s/{{ B_CLIENT }}/${B_CLIENT}/g" /tmp/bacula_template
        less /tmp/bacula_template

        # Pfad ändern, wenn Ubuntu Docker Container (DC) benutzt wird
        sed -i -e 's+/var/lib/bacula/+/opt/bacula/working/+g' /tmp/bacula_template
        less /tmp/bacula_template  

        # copy template to bacula-dir config
        cat /tmp/bacula_template >> /opt/bacularis/data/bacula/config/etc/bacula-dir.conf
        rm -rf /tmp/bacula_template

        # Restart Docker Container
        DOCKER_DIR=/opt/bacularis
        cd ${DOCKER_DIR}
        docker-compose up -d --force-recreate
        docker-compose ps
        docker-compose logs
        docker-compose logs bacularis
        ```

[^1]: [LVM Docu](https://docs.redhat.com/de/documentation/red_hat_enterprise_linux/6/html/logical_volume_manager_administration/lvm_overview){target=\_blank}
[^2]: [Bacula Homepage](https://www.bacula.org/){target=\_blank}
