---
Title: Zabbix
Summary: Monitoring mit Zabbix
Authors: Johann Hahn
Date:    September 26, 2024
tags: [zabbix, docker, monitoring]
---

# <img src="../../../assets/logos/zabbix.png" width="28" height="26" /> Zabbix

##### Zabbix Tipps

??? tip "Die Größe der Zabbix Datenbank reduzieren"

    #### Die Größe der Datenbank reduzieren

    - Die Größe der Tabelen anzeigen lassen

    ```bash
    ncdu /opt/zabbix-docker/
    ----
    4.2 GiB [############################]  history.ibd
    4.1 GiB [##########################  ]  history_uint.ibd
    896.0 MiB [#####                       ]  auditlog.ibd
    660.0 MiB [####                        ]  trends_uint.ibd
    548.0 MiB [###                         ]  trends.ibd
    660.0 MiB [############################]  trends_uint.ibd
    548.0 MiB [#######################     ]  trends.ibd
    308.0 MiB [#############               ]  history_text.ibd
    144.0 MiB [######                      ]  history_str.ibd
    ----
    ```

    - Service Monit anhalten

    ```bash
    systemctl stop monit.service
    ```

    - Container zabbix-server anhalten

    === "docker compose (Plugin)"
        ```bash
        cd /opt/zabbix-docker
        docker compose down zabbix-server

        # Zeigt Status an
        docker compose ps
        ```

    === "docker-compose (Standalone)"
        ```bash
        cd /opt/zabbix-docker
        docker-compose down zabbix-server

        # Zeigt Status an
        docker-compose ps
        ```
    - Passwort der Datenbank in der Variable speichern

    ```bash
    MYSQL_ROOT_PASSWORD=$(cat env_vars/.MYSQL_ROOT_PASSWORD)
    ```
    - Einige Tabelen der Zabbix Datenbank leeren

    ```bash
    # Vorhandene Datenbanken anzeigen lassen
    # dcexec mysql-server mysql --batch --user=root --password=${MYSQL_ROOT_PASSWORD} -e "show databases;"

    dcexec mysql-server mysql --batch --user=root --password=${MYSQL_ROOT_PASSWORD} -e "TRUNCATE TABLE zabbix.history_uint;"
    dcexec mysql-server mysql --batch --user=root --password=${MYSQL_ROOT_PASSWORD} -e "TRUNCATE TABLE zabbix.history;"
    dcexec mysql-server mysql --batch --user=root --password=${MYSQL_ROOT_PASSWORD} -e "TRUNCATE TABLE zabbix.auditlog;"
    dcexec mysql-server mysql --batch --user=root --password=${MYSQL_ROOT_PASSWORD} -e "TRUNCATE TABLE zabbix.trends_uint;"
    dcexec mysql-server mysql --batch --user=root --password=${MYSQL_ROOT_PASSWORD} -e "TRUNCATE TABLE zabbix.trends;"
    dcexec mysql-server mysql --batch --user=root --password=${MYSQL_ROOT_PASSWORD} -e "TRUNCATE TABLE zabbix.history_text;"
    dcexec mysql-server mysql --batch --user=root --password=${MYSQL_ROOT_PASSWORD} -e "TRUNCATE TABLE zabbix.history_str;"

    ```

    - Alle Zabbix Container neu starten

    === "docker compose (Plugin)"
        ```bash
        cd /opt/zabbix-docker
        docker compose up -d

        # Zeigt Status an
        docker compose ps

        # Zeigt Logdaten an
        docker compose logs -f
        ```

    === "docker-compose (Standalone)"
        ```bash
        cd /opt/zabbix-docker
        docker-compose up -d

        # Zeigt Status an
        docker-compose ps

        # Zeigt Logdaten an
        docker-compose logs -f
        ```
    - Alle laufende Container anzeigen lassen

    ```bash
    ctop
    ``` 

    - Nicht laufende Docker Images löschen 

    ```bash
    docker system prune
    ```

    - Service Monit wieder starten

    ```bash
    systemctl start monit.service
    ncdu /opt/zabbix-docker/
    ```
