---
Title: Planke
Summary: Planka
Authors: Johann Hahn
Date:    März 17, 2025
tags: [Docker, planka, project, board]
---

# <img src="../../../assets/logos/planka.png" width="28" height="26" /> Planka

!!! question "What is Planka?"

    :material-trello: [`Planka`][Planka]{target=\_blank}[^1] is a open source project management app.

    [Planka]: https://planka.app/

#### Algemeine Information

!!! note "Allgemeine Information über Planka"

    Am einfachsten lässt sich `Planka` as Docker Container installieren. Es gibt ein [`offizielles Planka Docker Image`][offizielles Planka Docker Image]{target=\_blank}[^2]. Es gibt auch eine [`Planka Live-Demo`][Planka Live-Demo]{target=\_blank}. Den `Source Code` findet man auf [`Planka GitHub`][Planka GitHub]{target=\_blank}[^3]. [`Planka Docs`][Planka Docs]{target=\_blank} ist sehr hilfreich für die Installation und Konfiguration der APP.

    [Planka GitHub]: https://github.com/plankanban/planka
    [offizielles Planka Docker Image]: https://github.com/plankanban/planka
    [Planka Live-Demo]: https://plankanban.github.io/planka/#/
    [Planka Docs]: https://docs.planka.cloud/docs/category/docker/

#### Install Planka as `docker container`


#### Upgrade Postgres Datenbank

??? tip "Planka: Upgrade Postgres Version 14 zu Version 16"

    ```bash
    cd /opt/planka
    docker compose exec postgres bash

    ### show databases, users and rights 
    # psql --username postgres --dbname plankadb
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
    docker compose exec postgres pg_dump -U postgres -d plankadb -cC > upgrade_backup_pg14.sql

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
    cd /opt/planka
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
        env_file:
          - .env
        environment:
          - TZ=${TZ}
          - POSTGRES_DB=${POSTGRES_DB}
          - POSTGRES_HOST_AUTH_METHOD=${POSTGRES_HOST_AUTH_METHOD}
    #    healthcheck:
    #      test: ["CMD-SHELL", "pg_isready -U postgres -d plankadb"]
    #      interval: 30s
    #      timeout: 10s
    #      retries: 5
        network_mode: none
    #    networks:
    #      plankaNet:
    #        #ipv4_address: ${SUBNET}.3
    #        aliases:
    #          - postgres-server
    #    labels:
    #      # For Service BACKUP: Adding this label means this container should be stopped while it's being backed up:
    #      - "docker-volume-backup.stop-during-backup=true"
    ----

    ### pull postgres 16 docker image
    docker compose pull

    ### run only postgres docker container
    docker compose up --force-recreate -d postgres
    docker compose logs -f

    ### restore db dump of "plankadb"
    cat upgrade_backup_pg14.sql | docker compose exec -T postgres psql -U postgres

    ### show databases, users and rights
    # psql --username postgres --dbname plankadb
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
        env_file:
          - .env
        environment:
          - TZ=${TZ}
          - POSTGRES_DB=${POSTGRES_DB}
          - POSTGRES_HOST_AUTH_METHOD=${POSTGRES_HOST_AUTH_METHOD}
        healthcheck:
          test: ["CMD-SHELL", "pg_isready -U postgres -d plankadb"]
          interval: 30s
          timeout: 10s
          retries: 5
    #    network_mode: none
        networks:
          plankaNet:
            #ipv4_address: ${SUBNET}.3
            aliases:
              - postgres-server
        labels:
          # For Service BACKUP: Adding this label means this container should be stopped while it's being backed up:
          - "docker-volume-backup.stop-during-backup=true"
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
    cd /opt/planka
    rm -rf data/postgres/db-data_14

    # Restart docker stack
    docker compose down && docker compose up -d
    docker compose logs

    ### run monit
    systemctl start monit
    systemctl status monit

    ```
[^1]: :material-trello: [Planka APP](https://planka.app/){target=\_blank}
[^2]: :material-docker: [Docker Hub](https://github.com/plankanban/planka){target=\_blank}
[^3]: :material-github: [Github - Planka](https://github.com/plankanban/planka){target=\_blank}
