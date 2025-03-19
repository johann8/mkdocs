---
Title: DocuSeal
Summary: DocuSeal
Authors: Johann Hahn
Date:    März 17, 2025
tags: [Docker, docuseal, sign]
---

# <img src="../../../assets/logos/docuseal.png" width="28" height="26" /> DocuSeal

!!! question "What is DocuSeal?"

    :material-trello: [`DocuSeal`][DocuSeal]{target=\_blank}[^1] is a open source platform that provides secure and efficient digital document signing and processing.

    [DocuSeal]: https://www.docuseal.com/

#### Algemeine Information

!!! note "Allgemeine Information über DocuSeal"

    Am einfachsten lässt sich `DocuSeal` als Docker Container installieren. Es gibt ein [`offizielles DocuSeal Docker Image`][offizielles DocuSeal Docker Image]{target=\_blank}[^2]. Es gibt auch eine [`DocuSeal Live-Demo`][DocuSeal Live-Demo]{target=\_blank}. Den `Source Code` findet man auf [`DocuSeal GitHub`][DocuSeal GitHub]{target=\_blank}[^3]. [`DocuSeal Docs`][DocuSeal Docs]{target=\_blank} ist sehr hilfreich für die Installation und Konfiguration der APP.

    [DocuSeal GitHub]: https://github.com/docusealco/docuseal
    [offizielles DocuSeal Docker Image]: https://hub.docker.com/r/docuseal/docuseal
    [DocuSeal Live-Demo]: https://demo.docuseal.tech/
    [DocuSeal Docs]: https://www.docuseal.com/docs

#### Install DocuSeal als `docker container`


#### Upgrade Postgres Datenbank

??? tip "DocuSeal: Upgrade Postgres Version 14 zu Version 16"

    ```bash
    cd /opt/docuseal
    docker compose exec postgres bash

    ### show databases, users and rights 
    # psql --username postgres --dbname docuseal
    psql --username postgres

    postgres=# \l
    postgres=# \du
    postgres=# \q
    exit

    # show db directory rights
    ls -la data/postgres/
    ----
    drwx------ 19   70 root 4096 20. Feb 23:26 db-data
    ----

    ls -la data/postgres/db-data/
    ----
    ...
    drwx------  6   70   70  4096 18. Jul 2022  base
    drwx------  2   70   70  4096 20. Feb 23:27 global
    ...
    ----

    ### create dump of db "plankadb"
    docker compose exec postgres pg_dump -U postgres -d docuseal -cC > upgrade_backup_pg14.sql

    ### show result
    ls -lah
    ----
    ...
    -rw-r--r--  1 root root 107K 21. Feb 12:03 upgrade_backup_pg14.sql
    ----

    ### stop monit - monitoring tool
    systemctl stop monit
    systemctl status monit

    ### stop planka docker stack
    cd /opt/docuseal
    docker compose down

    ### move "old" db directory
    mv data/postgres/db-data data/postgres/db-data_14

    ### create Enew" db directory
    mkdir data/postgres/db-data
    chown 70 data/postgres/db-data
    chmod 0700 data/postgres/db-data

    ### show all db directories
    ls -la data/postgres/
    ----
    drwx------  2   70 root 4096 21. Feb 12:07 db-data
    drwx------ 19   70 root 4096 21. Feb 12:07 db-data_14
    ----

    ### Edit the docker-compose.yml: add "network_mode: none" to the postgres config and change "image: postgres:14-alpine" to "image: postgres:16-alpine"
    vim docker-compose.yml
    ----
      postgres:
        #image: postgres:14-alpine
        image: postgres:16-alpine
        container_name: postgres
        restart: unless-stopped
        volumes:
          - ${DOCKERDIR}/data/postgres/db-data:/var/lib/postgresql/data
        environment:
          TZ: ${TZ}
          POSTGRES_USER: ${POSTGRES_USER}
          POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
          POSTGRES_DB: ${POSTGRES_DB}
    #    healthcheck:
    #      test: ["CMD-SHELL", "pg_isready -U postgres -d docuseal"]
    #      interval: 30s
    #      timeout: 10s
    #      retries: 5
        network_mode: none
    #    networks:
    #      docusealNet:
    #      ipv4_address: ${IPV4_NETWORK:-}.10
    #      ipv6_address: ${IPV6_NETWORK:-fd4d:6169:6c63:6f28}::10
    ----

    ### pull postgres 16 docker image
    docker compose pull

    ### run only postgres docker container
    docker compose up --force-recreate -d postgres
    docker compose logs -f

    ### restore db dump of "plankadb"
    cat upgrade_backup_pg14.sql | docker compose exec -T postgres psql -U postgres

    ### show databases, users and rights
    # psql --username postgres --dbname docuseal
    psql --username postgres

    postgres=# \l
    postgres=# \du
    postgres=# \q
    exit

    ### change docker-compose.yml as below
    vim docker-compose.yml
    ----
      postgres:
        #image: postgres:14-alpine
        image: postgres:16-alpine
        container_name: postgres
        restart: unless-stopped
        volumes:
          - ${DOCKERDIR}/data/postgres/db-data:/var/lib/postgresql/data
        environment:
          TZ: ${TZ}
          POSTGRES_USER: ${POSTGRES_USER}
          POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
          POSTGRES_DB: ${POSTGRES_DB}
        healthcheck:
          test: ["CMD-SHELL", "pg_isready -U postgres -d docuseal"]
          interval: 30s
          timeout: 10s
          retries: 5
    #    network_mode: none
        networks:
          docusealNet:
          ipv4_address: ${IPV4_NETWORK:-}.10
          ipv6_address: ${IPV6_NETWORK:-fd4d:6169:6c63:6f28}::10
    ----

    ### recreate docker stack
    docker compose up --force-recreate -d
    docker compose ps
    docker compose logs -f

    ### show all docker images and delete image  "postgres:14-alpine"
    docker images -a
    docker rmi 919eab2f108d

    ### check that everything is running.
    ### If everything works, then delete the old DB folder
    cd /opt/docuseal
    rm -rf data/postgres/db-data_14

    # Restart docker stack
    docker compose down && docker compose up -d
    docker compose logs

    ### run monit
    systemctl start monit
    systemctl status monit

    ```
