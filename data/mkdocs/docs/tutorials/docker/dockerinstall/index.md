---
Title: Docker installation
Summary: Docker unter Linux installieren.
Authors: Johann Hahn
Date:    Juli 09, 2024
---

# :fontawesome-brands-docker: Docker unter :fontawesome-brands-linux: Linux installieren

!!! info

    Es werden überwiegend die `virtuellen Maschienen` (VM) mit den Betriebssystemen `Rocky Linux`, `Oracle`, `Debian` und `Ubuntu` benutzt. Die VM's laufen entweder auf dem eigenen Server (ProxMox VE)[^1] oder in `Rechenzentren` cloudbasiert.

??? tip "Docker unter `Rocky Linux | Oracle` installieren"
    
    === "Docker install"
        ```bash
        # Install docker Repository on Rocky Linux | Oracle
        dnf install -y yum-utils
        dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

        # Install docker
        dnf -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
        systemctl enable --now docker containerd
        systemctl status docker

        # Running Docker as a non-root user
        usermod -aG docker jhahn
        groups jhahn
        lid -g docker
    
        # check docker version
        docker --version
        ```

    === "Docker update"
        ```bash
        # update docker
        dnf update

        # check vocker version
        docker version
        ```

??? tip "Docker unter `Debian | Ubuntu` installieren"
    === "docker install"
        ```bash
        # Install docker Repository on Debian | Ubuntu
        
        # Add Docker's official GPG key:
        apt-get update
        apt-get install ca-certificates curl
        install -m 0755 -d /etc/apt/keyrings
        curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
        chmod a+r /etc/apt/keyrings/docker.asc

        # Add the repository to Apt sources:
        echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
        $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
        tee /etc/apt/sources.list.d/docker.list > /dev/null
        apt-get update

        # install docker
        apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

        # Running Docker as a non-root user
        usermod -aG docker jhahn
        groups jhahn
        libuser-lid -g docker
        ```

    === "Docker update"
        ```bash
        # update docker
        apt-get update && apt-get dist-upgrade

        # check vocker version
        docker version
        ```
    

??? tip "`Docker-compose` binary installieren"
    
    === "docker-compose binary `install | update`"
        ```bash
        # check latest version under
        https://api.github.com/repos/docker/compose/releases/latest
        
        # install docker-compose binary
        if [ -f /usr/local/bin/docker-compose ]; then \mv /usr/local/bin/docker-compose /usr/local/bin/docker-compose_old; fi
        cd /tmp && curl -s https://api.github.com/repos/docker/compose/releases/latest \
        | grep browser_download_url \
        | grep docker-compose-linux-x86_64 \
        | cut -d '"' -f 4 \
        | wget -qi -
        chmod +x docker-compose-linux-x86_64
        \mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose
        cd /usr/local/bin/ && docker-compose --version
        ln -s /usr/local/bin/docker-compose /usr/bin/
        docker-compose version
        ```    

    === "Aliases für docker-compose - alle OS"
        ```bash
        # ganz unten einfügen
        vim /root/.bashrc
        ---
        ...
        # Aliases für docker-compose
        alias dc='docker-compose'
        alias dclogs='docker-compose logs -f --tail=1000'
        alias dcup='docker-compose up'
        alias dcdown='docker-compose down'
        alias dcexec='docker-compose exec'
        ```

    === "PATH unter `RockyLinux | Oracle` ergänzen"
        ```bash
        # File /etc/profile.d/usr_local.sh erstellen
        echo 'export PATH=$PATH:/usr/local/bin' > /etc/profile.d/usr_local.sh

        # Check PATH
        sudo su
        env |grep PATH
        docker-compose version
        ```

??? question "Was benutzen: Docker compose `Plugin` oder binary `Datei`?"

    Docker compose ist ein Tool, das die Verwaltung von Docker-Containern in komplexen Anwendungen vereinfacht. Man kann entweder das Binary extra installieren oder docker compose Plugin installieren. Die Syntax der Befehlen ist ähnlich. Unten sind ein paar Beispiele.

    === "docker compose plugin"
        ```bash
        cd /opt/mkdocsmaterial
        docker compose --help
        docker compose ps
        docker compose exec nginx-mkdocs sh
        docker compose stop nginx-mkdocs
        docker compose start nginx-mkdocs
        docker compose version
        ```

    === "docker-compose binary"
        ```bash
        cd /opt/mkdocsmaterial
        docker-compose --help
        docker-compose ps
        docker-compose exec nginx-mkdocs sh
        docker-compose stop nginx-mkdocs
        docker-compose start nginx-mkdocs
        docker-compose version
        ```


[^1]: [ProxMox VE](https://www.proxmox.com/de)
