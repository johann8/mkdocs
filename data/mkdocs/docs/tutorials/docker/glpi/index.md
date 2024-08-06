---
Title: GLPI 
Summary: GLPI - IT Asset Management
Authors: Johann Hahn
Date:    Juli 09, 2024
tags: [Docker, GLPI, Inventory]
---

# GLPI - IT Asset Management

!!! question "What is GLPI?" 

    :simple-traefikproxy: [`GLPI`][GLPI][^1] is a web-based `open source`[^2] application helping companies to manage their information system. The solution is able to build an inventory of all the organization's assets and to manage administrative and financial tasks. The system's functionalities help IT Administrators to create a database of technical resources, as well as a management and history of maintenances actions. Users can declare incidents or requests (based on asset or not) thanks to the Helpdesk feature.

    [GLPI]: https://glpi-project.org

[^1]: :material-wikipedia: [Wikipedia - GLPI](https://en.wikipedia.org/wiki/GLPi)
[^2]: :material-github: [Github - GLPI](https://github.com/glpi-project/glpi)
[^3]: :material-docker: [Docker Hub](https://hub.docker.com/repository/docker/johann8/alpine-glpi/general)

#### Algemeine Information

!!! note "Allgemeine Information über GLPI"

    <img src="../../../assets/logos/logo-G-100-white.png" width="20" height="20" /> [`GLPI`][GLPI] ist sehr vielseitig. Die Anwendung ist modular aufgebaut: Inventarisierung, Verwaltung von Hard- und Software, Ticketsystem usw. Man kann den Funktionsumfang mit Hilfe von [`Plugins`][Plugins] erweitern. Für die Inventarisierung wird entweder ein [`GLPI Agent`][GLPI Agent] oder eine Anwendung namens [`OCS Inventory`][OCS Inventory] verwendet. Ich bevorzuge `OCS Inventory`.
    Für jede Anwendungen habe ich ein `Docker Image` gebaut, sodass die Installation und das Update sehr schnell erfolgen können.
    Die Docker Images befinden sich auf dem [Docker Hub][^3]. Die Links zu Docker Images: [`GLPI Docker Image`][GLPI Docker Image] und [`OCS Inventory Docker image`][OCS Inventory Docker image].
    [`Hier`][Hier] ist die Beschreibung wie man `OCS Inventory` installiert und konfiguriert.

    [Plugins]: https://github.com/pluginsGLPI 
    [GLPI Agent]: https://github.com/glpi-project/glpi-agent
    [OCS Inventory]: https://github.com/OCSInventory-NG/OCSInventory-ocsreports
    [GLPI]: https://glpi-project.org
    [GLPI Docker Image]: https://hub.docker.com/repository/docker/johann8/alpine-glpi/general
    [OCS Inventory Docker image]: https://hub.docker.com/repository/docker/johann8/alpine-ocs/general
    [Hier]: https://docs.int.wassermanngruppe.de/tutorials/docker/ocsinventory/
    

## Install GLPI as `docker container`

Jeder Container repräsentiert eine einzelne Anwendung, die in einem überbrückten (Bridged) Netzwerk verbunden sind:

<div class="grid cards" markdown>
   - <img src="../../../assets/logos/logo-G-100-white.png" width="20" height="20" /> [__GLPI__](https://glpi-project.org/) IT Asset Management.
   - <img src="../../../assets/logos/logo-ocs.png" width="20" height="20" /> [__OCS Inventory__](https://ocsinventory-ng.org/?lang=en) Open computers and software inventory.
   - :simple-mariadb: [__MariaDB__](https://mariadb.org/) Datenbank.
   - :simple-nginx: [__Nginx__](https://nginx.org/) Webserver für Komponenten des Stacks.
   - :fontawesome-solid-memory: [__Memcached__](https://www.memcached.org/) Distributed memory object caching system.
</div>

#### Bereiten Sie Ihr System vor

Bevor Sie `GLPI` installieren, sollten Sie einige Voraussetzungen überprüfen:

!!! info
    - [`Docker`][Docker] Dienst muss auf dem Host installiert sein.
    - Traefik muss als `Docker Service` installiert sein.
    - [`MariaDB`][MariaDB] muss entweder als `Container` im gleichen `Docker Stack` laufen oder als extra `Docker Service` installiert sein.
    - For OCS Inventory: Port `4443 TCP` muss auf dem `Docker Host` geöffnet werden, damit der OCS Inventory Agent eine Verbindung zum Server aufbauen kann.
    - Einen DNS-Record z.B. `glpi.myfirma.de` erstellen.

    [Docker]: ../dockerinstall/index.md
    [MariaDB]: ../mariadb/index.md


#### GLPI installieren

??? tip "GLPI unter `Rocky Linux | Oracle` einrichten"

    === "Create folders"
        ```bash
        DOCKERDIR=/opt/inventory
        mkdir -p ${DOCKERDIR}/data/{glpi,crond,crontabs,mariadb}
        mkdir -p ${DOCKERDIR}/data/glpi/{files,plugins,config}
        mkdir -p ${DOCKERDIR}/data/crond/{5min,15min,30min,hourly,daily,weekly,monthly}
        mkdir -p ${DOCKERDIR}/data/mariadb/{dbdata,socket,config}
        chown -R 100:101 ${DOCKERDIR}/data/glpi/*
        cd ${DOCKERDIR}
        tree -d -L 5 ${DOCKERDIR}
        ```
    === "Download config files"
        ```bash
        DOCKERDIR=/opt/inventory
        cd ${DOCKERDIR}
        wget https://raw.githubusercontent.com/johann8/alpine-glpi/master/docker-compose.yml
        wget https://raw.githubusercontent.com/johann8/alpine-glpi/master/docker-compose.override.yml
        wget https://raw.githubusercontent.com/johann8/alpine-glpi/master/.env
        ```
    === "Set firewall rules"
        ```bash
        # Set firewall rules
        firewall-cmd --zone=public --add-port=4443/tcp --permanent
        firewall-cmd --reload
        firewall-cmd --zone=public --list-all
        ```

    === "MariaDB create my.cnf"
        ```bash
        DOCKERDIR=/opt/inventory
        cd ${DOCKERDIR}
        cat > data/mariadb/config/my.cnf << 'EOL'
        [mysqld]
        default-time-zone              = 'Europe/Berlin'
        character-set-client-handshake = FALSE
        character-set-server           = utf8mb4
        collation-server               = utf8mb4_unicode_ci
        max_allowed_packet             = 192M
        max-connections                = 350
        key_buffer_size                = 0
        read_buffer_size               = 192K
        sort_buffer_size               = 2M
        innodb_buffer_pool_size        = 24M
        read_rnd_buffer_size           = 256K
        tmp_table_size                 = 24M
        performance_schema             = 0
        innodb-strict-mode             = 0
        thread_cache_size              = 8
        query_cache_type               = 0
        query_cache_size               = 0
        max_heap_table_size            = 48M
        thread_stack                   = 256K
        skip-host-cache
        log-warnings                   = 0
        event_scheduler                = 1

        [client]
        default-character-set          = utf8mb4

        [mysql]
        default-character-set          = utf8mb4
        EOL
        ```

#### GLPI: Customise compose and config files

??? tip "GLPI: Customise compose and config files"

    === "docker-compose.yml"

        ``` yaml
        ---
        networks:
          inventoryNet_frontend:
            driver: bridge
            driver_opts:
              com.docker.network.enable_ipv6: "false"
            ipam:
              driver: default
              config:
                - subnet: ${SUBNET_FRONTEND}.0/24
          inventoryNet_backend:
            driver: bridge
            driver_opts:
              com.docker.network.enable_ipv6: "false"
            internal: true
            ipam:
              driver: default
              config:
                - subnet: ${SUBNET_BACKEND}.0/24

        services:

          #
          ### === OCS Inventory Web APP ===
          #
          ocsapp:
            image: johann8/alpine-ocs:${VERSION_OCS}
            container_name : ocsapp
            restart: always
            #ports:                                # commented, if traefik is used
            #- ${PORT_OCS_EXT}:80                # commented, if traefik is used
          volumes:
            - "${DOCKERDIR}/data/ocsinventory/perlcomdata:/etc/ocsinventory-server"
            - "${DOCKERDIR}/data/ocsinventory/ocsreportsdata:/usr/share/ocsinventory-reports/ocsreports/extensions"
            - "${DOCKERDIR}/data/ocsinventory/varlibdata:/var/lib/ocsinventory-reports"
            - "${DOCKERDIR}/data/ocsinventory/httpdconfdata:/etc/apache2/conf.d"
          environment:
            TZ: ${TZ}
            OCS_INVENTOTRY_INSTALL: false        # should be 'true' if Ocsinventory does not install. After installation please set to 'false'
            OCS_DB_SERVER: ${OCS_DB_SERVER}
            OCS_DB_USER: ${OCS_DB_USER}
            OCS_DB_PASS: ${OCS_DB_PASS}
            OCS_DB_NAME: ${OCS_DB_NAME}
            # See documentation to set up SSL for MySQL
            OCS_SSL_ENABLED: 0
            #OCS_DISABLE_API_MODE: 0              # comment this var, if api should be activated
          depends_on:
            - mariadb
          hostname: ${HOSTNAME_INVENTORY}.${DOMAINNAME}
          networks:
            - inventoryNet_frontend
            - inventoryNet_backend

          #
          ### === OCS Inventory Nginx Proxy ===
          #
          ocsproxy:
            image: nginx:stable-alpine3.17
            container_name: ocsproxy
            restart: always
          ports:
            - "4443:443"                          # For OCS Agents
          volumes:
            - ${DOCKERDIR}/data/nginx/config:/etc/nginx/templates
            - ${DOCKERDIR}/data/nginx/certs:/etc/nginx/certs
            - ${DOCKERDIR}/data/nginx/auth:/etc/nginx/auth
          environment:
            # 80 or 443
            LISTEN_PORT: 443
            # empty or ssl
            PORT_TYPE: "ssl"
            SSL_CERT: ocs.crt
            SSL_KEY: ocs.key
            # OCS Api user restriction (default ocsapi/ocapi)
            API_AUTH_FILE: ocsapi.htpasswd
            # OCS Download
            READ_TIMEOUT: 300
            CONNECT_TIMEOUT: 300
            SEND_TIMEOUT: 300
            MAX_BODY_SIZE: 1G
          depends_on:
            - ocsapp
          networks:
            - inventoryNet_frontend

          #
          ### === GLPI Container ===
          #
          glpi:
            image: johann8/alpine-glpi:${VERSION}
            container_name: glpi
            hostname: glpi
            restart: unless-stopped
            depends_on:
              - mariadb
            volumes:
              - ${DOCKERDIR}/data/glpi/files:/var/www/glpi/files/:rw
              - ${DOCKERDIR}/data/glpi/plugins:/var/www/glpi/plugins/:rw
              - ${DOCKERDIR}/data/glpi/config:/var/www/glpi/config/:rw
              # For crontab: comment out what you need
              #- ${DOCKERDIR}/data/crond/5min:/etc/periodic/5min/
              #- ${DOCKERDIR}/data/crond/15min:/etc/periodic/15min/
              #- ${DOCKERDIR}/data/crond/30min:/etc/periodic/30min/
              #- ${DOCKERDIR}/data/crond/hourly:/etc/periodic/hourly/
              - ${DOCKERDIR}/data/crond/daily:/etc/periodic/daily/
              - ${DOCKERDIR}/data/crond/weekly:/etc/periodic/weekly/
              - ${DOCKERDIR}/data/crond/monthly:/etc/periodic/monthly/
              - ${DOCKERDIR}/data/crontabs:/etc/crontabs/
            environment:
              GLPI_LANG: ${GLPI_LANG}
              TZ: ${TZ}
              MARIADB_HOST: ${MARIADB_GLPI_HOST}
              MARIADB_PORT: ${MARIADB_GLPI_PORT}
              MARIADB_DATABASE: ${MARIADB_GLPI_DATABASE}
              MARIADB_USER: ${MARIADB_GLPI_USER}
              MARIADB_PASSWORD: ${MARIADB_GLPI_PASSWORD}
            #ports:                                 # commented, if traefik is used
              #- ${PORT_GLPI_EXT}:8080              # commented, if traefik is used
            networks:
              - inventoryNet_frontend
              - inventoryNet_backend

          #
          ### === MEMCACHED ===
          #
          memcached:
            image: memcached:alpine3.18
            container_name: memcached
            restart: always
            depends_on:
              - glpi
            environment:
              - TZ=${TZ}
            command: ["-m", "128"]
            networks:
              - inventoryNet_frontend

          #
          ### === MariaDB Database ===
          #
          mariadb:
            image: mariadb:${VERSION_DB}
            container_name: mariadb
            stop_grace_period: 45s
            restart: unless-stopped
            # remove comment after install
            healthcheck:
              test: "mysqladmin ping -h localhost -u$${OCS_DB_USER} --password=$${OCS_DB_PASS}"
              interval: 45s
              timeout: 10s
              retries: 5
            volumes:
              - "${DOCKERDIR}/data/mariadb/dbdata:/var/lib/mysql:rw"
              - "${DOCKERDIR}/data/mariadb/config:/etc/mysql/conf.d:ro"
              #- "${DOCKERDIR}/data/mariadb/sql:/docker-entrypoint-initdb.d"
              #- "${DOCKERDIR}/data/mariadb/socket:/var/run/mysqld"
            environment:
              MARIADB_ROOT_PASSWORD: ${MARIADB_ROOT_PASSWORD}
              # OCS Inventory Database - created automatically
              MARIADB_DATABASE:      ${OCS_DB_NAME}
              MARIADB_USER:          ${OCS_DB_USER}
              MARIADB_PASSWORD:      ${OCS_DB_PASS}
              # GLPI Database - must be created manually (see below)
              MARIADB_GLPI_DATABASE: ${MARIADB_GLPI_DATABASE}
              MARIADB_GLPI_USER: ${MARIADB_GLPI_USER}
              MARIADB_GLPI_PASSWORD: ${MARIADB_GLPI_PASSWORD}
            ports:
              - "3306:3306"
            networks:
              inventoryNet_backend:
                ipv4_address: ${IP_ADDRESS}
        ```

    === "docker-compose.override.yml"

        ``` yaml
        ---
        services:

        ocsapp:
          labels:
            - "traefik.enable=true"
            - "traefik.docker.network=proxy"
            - "traefik.http.routers.ocs-secure.entrypoints=websecure"
            - "traefik.http.routers.ocs-secure.middlewares=default-chain@file,rate-limit@file"
            - "traefik.http.routers.ocs-secure.rule=Host(`${HOSTNAME_OCS}.${DOMAINNAME}`)"
            - "traefik.http.routers.ocs-secure.service=ocs"
            #- "traefik.http.routers.ocs-secure.tls.certresolver=produktion"             # für eigene Zertifikate
            - "traefik.http.routers.ocs-secure.tls=true"
            - "traefik.http.services.ocs.loadbalancer.sticky.cookie.httpOnly=true"
            - "traefik.http.services.ocs.loadbalancer.sticky.cookie.secure=true"
            - "traefik.http.services.ocs.loadbalancer.server.port=${PORT_OCS}"
          networks:
            - proxy

        glpi:
          labels:
            - "traefik.enable=true"
            - "traefik.docker.network=proxy"
            - "traefik.http.routers.glpi-secure.entrypoints=websecure"
            - "traefik.http.routers.glpi-secure.middlewares=default-chain@file,rate-limit@file"
            - "traefik.http.routers.glpi-secure.rule=Host(`${HOSTNAME_GLPI}.${DOMAINNAME}`)"
            - "traefik.http.routers.glpi-secure.service=glpi"
            #- "traefik.http.routers.glpi-secure.tls.certresolver=produktion"             # für eigene Zertifikate
            - "traefik.http.routers.glpi-secure.tls=true"
            - "traefik.http.services.glpi.loadbalancer.sticky.cookie.httpOnly=true"
            - "traefik.http.services.glpi.loadbalancer.sticky.cookie.secure=true"
            - "traefik.http.services.glpi.loadbalancer.server.port=${PORT_GLPI}"
          networks:
            - proxy

        networks:
          proxy:
            external: true
        ```

    === ".env"

        ``` yaml
        ### === SYSTEM ===
        TZ=Europe/Berlin
        DOCKERDIR=/opt/inventory

        ### === Network ===
        DOMAINNAME=myfirma.de
        HOSTNAME_OCS=ocs
        HOSTNAME_INVENTORY=ocsinventory            # For inventory - client access
        PORT_OCS=80
        HOSTNAME_GLPI=glpi
        PORT_GLPI=8080
        SUBNET_FRONTEND=172.16.208
        SUBNET_BACKEND=172.16.209

        ### === APP OCS ===
        VERSION_OCS=latest
        OCS_SSL_ENABLED=0
        OCS_DB_SERVER=mariadb
        OCS_DB_PORT=3306
        OCS_DB_NAME=ocsweb
        OCS_DB_USER=ocsuser
        # pwgen -1cnsB 25 1
        OCS_DB_PASS=7mRaTF9R7oKzJRNAiguokMnLL


        ### === APP GLPI ===
        GLPI_LANG=de_DE
        VERSION=latest
        UPLOAD_MAX_FILESIZE=100M
        POST_MAX_SIZE=50M
        MARIADB_GLPI_HOST=mariadb
        MARIADB_GLPI_PORT=3306
        MARIADB_GLPI_DATABASE=glpidb
        MARIADB_GLPI_USER=glpiuser
        # pwgen -1cnsB 25 1
        MARIADB_GLPI_PASSWORD=Mt3AofMNdAYUjPEtdYboVJXEn

        ### === MariaDB ===
        VERSION_DB=10.11
        # pwgen -1cnsB 30 1
        MARIADB_ROOT_PASSWORD=3rjWcLssmfzaiwWpWYJWrqP9gaboW7
        IP_ADDRESS=${SUBNET_BACKEND}.10
        ```

#### MySQL Datenbank anlegen, falls externe MySQL Datenbank benutzt wird

??? tip "GLPI: MySQL Datenbank anlegen"

    ```bash
    # set variables
    MYSQL_ROOT_PW=MySuperRootPassWord-
    MYSQL_GLPI_DB="glpi"
    MYSQL_GLPI_DB_USER_NAME="glpi"
    MYSQL_GLPI_DB_USER_PW=$(pwgen -s1 25)
    MYSQL_GLPI_DB_USER_SYNC_PW=$(pwgen -s1 25) 
    MySQL_HOST_IP_ADDRESS=172.16.18.2

    # create DB and user
    mysql --batch --user=root -h ${MySQL_HOST_IP_ADDRESS} --password=${MYSQL_ROOT_PW} -e "create database "${MYSQL_GLPI_DB}" character set utf8"
    mysql --batch --user=root -h ${MySQL_HOST_IP_ADDRESS} --password=${MYSQL_ROOT_PW} -e "CREATE USER "${MYSQL_GLPI_DB_USER_NAME}""
    mysql --batch --user=root -h ${MySQL_HOST_IP_ADDRESS} --password=${MYSQL_ROOT_PW} -e "grant all on "${MYSQL_GLPI_DB}".*  to 'glpi'@'%' identified by '${MYSQL_GLPI_DB_USER_PW}'"
    mysql --batch --user=root -h ${MySQL_HOST_IP_ADDRESS} --password=${MYSQL_ROOT_PW} -e "FLUSH PRIVILEGES"

    # create sync user - synchro
    mysql --batch --user=root -h ${MySQL_HOST_IP_ADDRESS} --password=${MYSQL_ROOT_PW} -e "CREATE USER 'synchro'@'%' IDENTIFIED BY '${MYSQL_GLPI_DB_USER_SYNC_PW}'" 
    mysql --batch --user=root -h ${MySQL_HOST_IP_ADDRESS} --password=${MYSQL_ROOT_PW} -e "GRANT USAGE ON *.* TO 'synchro'@'%' IDENTIFIED BY '${MYSQL_GLPI_DB_USER_SYNC_PW}'" 
    mysql --batch --user=root -h ${MySQL_HOST_IP_ADDRESS} --password=${MYSQL_ROOT_PW} -e "GRANT SELECT ON ocsweb.* TO 'synchro'@'%'" 
    mysql --batch --user=root -h ${MySQL_HOST_IP_ADDRESS} --password=${MYSQL_ROOT_PW} -e "GRANT DELETE ON ocsweb.deleted_equiv TO 'synchro'@'%'" 
    mysql --batch --user=root -h ${MySQL_HOST_IP_ADDRESS} --password=${MYSQL_ROOT_PW} -e "GRANT UPDATE (CHECKSUM) ON ocsweb.hardware TO 'synchro'@'%'" 
    #mysql --batch --user=root -h ${MySQL_HOST_IP_ADDRESS} --password=${MYSQL_ROOT_PW} -e "UPDATE mysql.user SET Password=PASSWORD('${GLPI_PW}') WHERE User='synchro'" 
    mysql --batch --user=root -h ${MySQL_HOST_IP_ADDRESS} --password=${MYSQL_ROOT_PW} -e "FLUSH PRIVILEGES"

    # Safe access data
    echo "# **** GLPI Install *****" > /root/glpi_install.txt
    echo "MYSQL_GLPI_DB: $MYSQL_GLPI_DB" >> /root/glpi_install.txt
    echo "MYSQL_GLPI_DB_USER_NAME: $MYSQL_GLPI_DB_USER_NAME" >> /root/glpi_install.txt
    echo "MYSQL_GLPI_DB_USER_PW: $MYSQL_GLPI_DB_USER_PW" >> /root/glpi_install.txt
    echo "MYSQL_ROOT_PW: $MYSQL_ROOT_PW" >> /root/glpi_install.txt
    echo "MYSQL_GLPI_DB_USER_SYNC_PW: $MYSQL_GLPI_DB_USER_SYNC_PW" >> /root/glpi_install.txt
    echo "MySQL_HOST_IP_ADDRESS: $MySQL_HOST_IP_ADDRESS" >> /root/glpi_install.txt
    ```

#### Der erste Start von `GLPI` Docker Container

=== "docker compose (Plugin)"
    ```bash
    cd /opt/inventory
    docker compose up -d

    # Zeigt Status an
    docker compose ps

    # Zeigt Logdaten an
    docker compose logs -f
    ```

=== "docker-compose (Standalone)"

    ```bash
    cd /opt/inventory
    docker-compose up -d

    # Zeigt Status an
    docker-compose ps

    # Zeigt Logdaten an
    docker-compose logs -f
    ```

#### Seite aufrufen

Wenn keine Fehlermeldungen erschienen sind kann man die Startseite von Traefik aufrufen.

!!! abstract "Die Anmeldedaten für `GLPI Login`"

        - **URL**: https://glpi.myfirma.de
        - **User**: glpi
        - **Password**: glpi (Bitte das Passwort sofort ändern)

