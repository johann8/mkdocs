---
Title: Monit
Summary: Monitoring tool
Authors: Johann Hahn
Date:    Oktober 07, 2024
tags:    [monit, monitoring, linux]
---

# <img src="../../../assets/logos/monit.png" width="28" height="28" /> Monit


[`Monit`][Monit]{target=\_blank}

[Monit]: https://mmonit.com/ 

??? tip "Bash Script um `Monit` upzudaten | zu installieren"

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
