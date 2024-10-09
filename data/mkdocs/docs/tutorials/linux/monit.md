---
Title: Monit
Summary: Monitoring tool
Authors: Johann Hahn
Date:    Oktober 07, 2024
tags:    [monit, monitoring, linux]
---

# <img src="../../../assets/logos/monit.png" width="28" height="28" /> Monit


[`Monit`][Monit]{target=\_blank}[^1] st ein rasch eingerichtetes, einfach zu bedienendes aber effektives Programm zur Überwachung von Serverdiensten. Es kann auch wichtige Basisdaten wie CPU-Nutzung, Festplattenbelegung und mehr einbeziehen. Falls ein Serverdienst ausfällt, kann er automatisch neu gestartet werden. Bei Problemen werden ein oder mehrere Empfänger per E-Mail informiert.

[Monit]: https://mmonit.com/ 

##### Monit installieren und konfigurieren

??? tip "`Monit` - Installation und Konfiguration"

    === "Monit Installation"

        ```bash
        VERSION=5.34.1
        cd /tmp/ && wget https://mmonit.com/monit/dist/binary/${VERSION}/monit-${VERSION}-linux-x64.tar.gz
        tar -xzvf monit-${VERSION}-linux-x64.tar.gz
        cd monit-${VERSION}/
        cp bin/monit /usr/local/bin/
        cp conf/monitrc /etc/
        cp man/man1/monit.1 /usr/share/man/man1/
        mkdir /etc/monit.d
        cd /tmp
        rm -rf monit-5.32.0*
        ln -s /usr/local/bin/monit /bin/monit
        mkdir /var/lib/monit/
        mkdir /var/monit
        dnf install libnsl -y
        monit --version
        systemctl enable --now monit
        systemctl status monit
        ```

    === "Monit Konfiguration"

        ```bash
        # Die Konfigurationsdatei wie im Beispiel unten editieren
        vim /etc/monitrc

        # egrep -v '(^.*#|^$)' /etc/monitrc
        set log /var/log/monit.log
        set pidfile /var/run/monit.pid
        set idfile /var/lib/monit/.monit.id
        set statefile /var/lib/monit/.monit.state
        set limits {
        }
        set mailserver smtp.mydomain.de port 587
              username "info@mydomain.de" password "MySuperPassword123"
              using tls
         set eventqueue
         set mail-format {
           from:    Monit <monit@$HOST>
           subject: monit alert --  $EVENT $SERVICE
           message: $EVENT Service $SERVICE
                         Date:        $DATE
                         Action:      $ACTION
                         Host:        $HOST
                         Description: $DESCRIPTION
                    Your faithful employee,
                    Monit
         }
        set httpd port 2812 and
            allow 0.0.0.0/0
            allow @wheel
          check system $HOST
            if loadavg (1min) per core > 2 for 5 cycles then alert
            if loadavg (5min) per core > 1.5 for 10 cycles then alert
            if cpu usage > 95% for 10 cycles then alert
            if memory usage > 95% then alert
            if swap usage > 75% then alert
         check filesystem root_fs with path /
            if space usage > 80% for 5 times within 15 cycles then alert
            if inode usage > 80% for 5 times within 15 cycles then alert
         check filesystem boot_fs with path /boot
            if space usage > 90% for 5 times within 15 cycles then alert
            if inode usage > 90% for 5 times within 15 cycles then alert
          include /etc/monit.d/*
        ```

##### Monit -  Beispiele

    === "Monitor Docker"

        ```bash

        ```

##### Monit Update

??? tip "Bash Script um `Monit` zu aktualisieren | zu installieren"

    === "monit_update_install.sh"
        
        ```bash
        #!/bin/bash

        # Abort on all errors, set -x
        set -o errexit
        #set -x

        #
        ### === Set Variables ===
        #
        VERSION=5.34.1

        #
        ### ============= Main script ============
        #

        # Prüfen, ob Monit existiert
        if [ -f /usr/local/bin/monit ]; then
           echo "Removing old version..."
           if [ -f /usr/local/bin/monit_old ]; then
              rm -rf /usr/local/bin/monit_old
                  mv /usr/local/bin/monit /usr/local/bin/monit_old
              RES=0
           else
              mv /usr/local/bin/monit /usr/local/bin/monit_old
              RES=0
           fi
        else
           echo "Monit ist nicht installieret";
           RES1=0
        fi

        # Update | Install Monit
        if [ x${RES} = x0 ]; then
           echo "Stopping monit service... "
           systemctl stop monit
           cd /tmp/ && wget https://mmonit.com/monit/dist/binary/${VERSION}/monit-${VERSION}-linux-x64.tar.gz
           tar -xzvf monit-${VERSION}-linux-x64.tar.gz
           cd monit-${VERSION}/
           \cp bin/monit /usr/local/bin/
           \cp man/man1/monit.1 /usr/share/man/man1/
           cd /tmp
           echo "removing downloaded version... "
           rm -rf monit-${VERSION}*
           systemctl start monit
           systemctl status monit
           monit --version
        elif [ x${RES1} = x0 ]; then
           cd /tmp/ && wget https://mmonit.com/monit/dist/binary/${VERSION}/monit-${VERSION}-linux-x64.tar.gz
           tar -xzvf monit-${VERSION}-linux-x64.tar.gz
           cd monit-${VERSION}/
           cp bin/monit /usr/local/bin/
           cp conf/monitrc /etc/
           cp man/man1/monit.1 /usr/share/man/man1/
           mkdir /etc/monit.d
           cd /tmp
           rm -rf monit-${VERSION}*
           ln -s /usr/local/bin/monit /bin/monit
           mkdir /var/lib/monit/
           mkdir /var/monit
           dnf install libnsl -y
           monit --version
        fi 
        ```

[^1]: :material-wikipedia: [Wikipedia - Monit](https://en.wikipedia.org/wiki/Monit){target=\_blank}
