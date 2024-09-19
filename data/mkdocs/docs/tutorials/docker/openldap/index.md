---
Title: OpenLDAP
Summary: OpenLDAP
Authors: Johann Hahn
Date:    August 02, 2024
tags: [Docker, OpenLDAP]
---

# OpenLDAP

!!! question "What is OpenLDAP?"

    :simple-traefikproxy: [`OpenLDAP`][OpenLDAP]{target=\_blank}[^1] Das Lightweight Directory Access Protocol (LDAP) ist ein Netzwerkprotokoll zur Abfrage und Änderung von Informationen verteilter Verzeichnisdienste. LDAP ist der De-facto-Industriestandard für Authentifizierung, Autorisierung sowie Adress- und Benutzerverzeichnisse. Die meisten Softwareprodukte, die mit Benutzerdaten umgehen müssen und am Markt relevant sind, unterstützen LDAP. `Source Repository` von OpenLDAP findet man über diesen [`Link`][Link]{target=\_blank}[^2]
    Standardport ist:
    - 389 für ungesicherte oder mit STARTTLS gesicherte Verbindungen
    -  636 für mit TLS gesicherte Verbindungen (LDAPS)

    [OpenLDAP]: https://www.openldap.org/
    [Link]: https://www.openldap.org/software/repo.html

### Algemeine Information

!!! note "Allgemeine Information"


    Für jede Anwendungen habe ich ein `Docker Image` gebaut, sodass die Installation und das Update sehr schnell erfolgen können.
    Docker Images befinden sich auf dem [Docker Hub][^3]. Die Links zu Docker Images: [`OpenLDAP Docker Image`][OpenLDAP Docker Image]{target=\_blank} und [`PhpLDAPadmin Docker Image`][PhpLDAPadmin Docker Image]{target=\_blank}

    [OpenLDAP Docker Image]: https://hub.docker.com/repository/docker/johann8/alpine-glpi/general
    [PhpLDAPadmin Docker Image]: https://hub.docker.com/repository/docker/johann8/phpldapadmin/general


### Install OpenLDAP as `docker container`

Jeder Container repräsentiert eine einzelne Anwendung, die in einem überbrückten (Bridged) Netzwerk verbunden sind:

<div class="grid cards" markdown>
   - <img src="../../../assets/logos/active-directory-80.png" width="20" height="20" /> [__OpenLDAP__](https://glpi-project.org/){target=\_blank}- Benutzer-Datenbank.
   - <img src="../../../assets/logos/phpldapadmin_tr.png" width="40" height="30" /> [__PhpLDAPadmin__](https://ocsinventory-ng.org/?lang=en){target=\_blank}- Web-Anwendung zur Steuerung von OpenLDAP.
</div>

### Bereiten Sie Ihr System vor

Bevor Sie `OpenLDAP` installieren, sollten Sie einige Voraussetzungen überprüfen:

!!! info
    - [`Docker`][Docker] Dienst muss auf dem Host installiert sein.
    - Traefik muss als `Docker Service` installiert sein.
    - Einen DNS-Record z.B. `pla.myfirma.de` erstellen.

    [Docker]: ../dockerinstall/index.md

#### OpenLDAP installieren

??? tip "OpenLDAP unter `Rocky Linux | Oracle` einrichten"

    === "Create folders"
        ```bash
        DOCKERDIR=/opt/inventory
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
        EOL
        ```

#### OpenLDAP: Customise compose and config files

??? tip "OpenLDAP: Customise compose and config files"

    === "docker-compose.yml"

        ``` yaml
        ---
        networks:
          ldapNet:
            driver: bridge
            driver_opts:
              com.docker.network.bridge.name: br-ldapnet
            enable_ipv6: true
            ipam:
              driver: default
              config:
                - subnet: ${IPV4_NETWORK:-172.22.18}.0/24
                - subnet: ${IPV6_NETWORK:-fd4d:6169:6c63:6f18::/64}

        services:

          #
          ### === OpenLDAP COntainer ===
          #

          openldap:
            image: johann8/alpine-openldap:${OPENLDAP_VERSION:-latest}
            container_name: openldap
            restart: unless-stopped
            environment:
              SLAPD_ROOTDN:            ${SLAPD_ROOTDN}
              SLAPD_ROOTPW:            ${SLAPD_ROOTPW}
              SLAPD_ROOTPW_HASH:       ${SLAPD_ROOTPW_HASH}
              SLAPD_ORGANIZATION:      ${SLAPD_ORGANIZATION}
              SLAPD_FQDN:              ${SLAPD_FQDN}
              SLAPD_SUFFIX:            ${SLAPD_SUFFIX}
              SLAPD_PWD_CHECK_QUALITY: ${SLAPD_PWD_CHECK_QUALITY}
              SLAPD_PWD_MIN_LENGTH:    ${SLAPD_PWD_MIN_LENGTH}
              SLAPD_PWD_MAX_FAILURE:   ${SLAPD_PWD_MAX_FAILURE}
              SLAPD_ROOTPW_SECRET:     ${SLAPD_ROOTPW_SECRET}
              SLAPD_USERPW_SECRET:     ${SLAPD_USERPW_SECRET}
              SLAPD_PASSWORD_HASH:     ${SLAPD_PASSWORD_HASH}
              LDAP_BACKUP_TTL:         ${LDAP_BACKUP_TTL}
              TZ:                      ${TZ}
            hostname: ${HOSTNAME0}.${DOMAINNAME}
            volumes:
              - ${DOCKERDIR}/data/backup:/data/backup
              - ${DOCKERDIR}/data/prepopulate:/etc/openldap/prepopulate:ro
              - ${DOCKERDIR}/data/ldapdb:/var/lib/openldap/openldap-data
              - ${DOCKERDIR}/data/ssl:/etc/ssl/openldap
              - ${DOCKERDIR}/data/config/ldap/ldif:/etc/openldap/ldif:ro
              - ${DOCKERDIR}/data/config/ldap/slapd.d:/etc/openldap/slapd.d
              - ${DOCKERDIR}/data/config/ldap/custom-schema:/etc/openldap/custom-schema
              - ${DOCKERDIR}/data/config/ldap/secrets:/run/secrets
            ports:
              - ${PORT_LDAP:-389}:389
              - ${PORT_LDAPS:-636}:636
            #labels:
              #ofelia.enabled: "true"
              ##ofelia.job-exec.slapd_backup_config.schedule: "@every 24h"
              #ofelia.job-exec.slapd_backup_config.schedule: "0 0 2 * * *"
              #ofelia.job-exec.slapd_backup_config.command: "/bin/sh -c \"/sbin/slapd-backup.sh 0 slapd-config || exit 0\""
              #ofelia.job-exec.slapd_backup_data.schedule: "0 0 2 * * *"
              #ofelia.job-exec.slapd_backup_data.command: "/bin/sh -c \"/sbin/slapd-backup.sh 1 slapd-data || exit 0\""
            networks:
              - ldapNet
            secrets:
              - ${SLAPD_ROOTPW_SECRET}
              - ${SLAPD_USERPW_SECRET}


          #
          ### === PhpLdapAdmin Container ===
          #

          phpldapadmin:
            image: johann8/phpldapadmin:${PLA_VERSION}
            container_name: phpldapadmin
            restart: unless-stopped
            #volumes:
              #- ${DOCKERDIR}/data/html:/var/www/html
            #ports:
              #- ${PORT_PLA:-8083}:8080
            environment:
              - TZ=${TZ}
              - PHPLDAPADMIN_LANGUAGE=${PHPLDAPADMIN_LANGUAGE}
              - PHPLDAPADMIN_PASSWORD_HASH=${PHPLDAPADMIN_PASSWORD_HASH}
              - PHPLDAPADMIN_SERVER_NAME=${PHPLDAPADMIN_SERVER_NAME}
              - PHPLDAPADMIN_SERVER_HOST=${PHPLDAPADMIN_SERVER_HOST}
              - PHPLDAPADMIN_BIND_ID=${PHPLDAPADMIN_BIND_ID}
            depends_on:
              - openldap
            networks:
              - ldapNet

        secrets:
          openldap-root-password:
            file: ${DOCKERDIR}/data/config/ldap/secrets/${SLAPD_ROOTPW_SECRET}
          openldap-user-passwords:
            file: ${DOCKERDIR}/data/config/ldap/secrets/${SLAPD_USERPW_SECRET}
        ```

    === "docker-compose.override.yml"

        ``` yaml
        ---
        services:

          phpldapadmin:
            labels:
              - "traefik.enable=true"
              ### ==== to https ====
              - "traefik.http.routers.phpldapadmin-secure.entrypoints=websecure"
              - "traefik.http.routers.phpldapadmin-secure.rule=Host(`${HOSTNAME_PLA}.${DOMAINNAME_PLA}`)"
              - "traefik.http.routers.phpldapadmin-secure.tls=true"
              - "traefik.http.routers.phpldapadmin-secure.tls.certresolver=production"  # für eigene Zertifikate
              ### ==== to service ====
              - "traefik.http.routers.phpldapadmin-secure.service=phpldapadmin"
              - "traefik.http.services.phpldapadmin.loadbalancer.server.port=${PORT_PLA}"
              - "traefik.docker.network=proxy"
              ### ==== redirect to authelia for secure login ====
              #- "traefik.http.routers.phpldapadmin-secure.middlewares=rate-limit@file,secHeaders@file"
              - "traefik.http.routers.phpldapadmin-secure.middlewares=authelia@docker,rate-limit@file,secHeaders@file"
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
        DOCKERDIR=/opt/openldap        

        ### === Network ===
        DOMAINNAME=myfirma.de
        HOSTNAME0=ldap
        PORT_LDAP=389
        PORT_LDAPS=636

        DOMAINNAME_PLA=${DOMAINNAME}
        HOSTNAME_PLA=pla
        PORT_PLA=8080
 
        IPV4_NETWORK=172.26.18
        IPV6_NETWORK=fd4d:6169:6c63:6f18::/64

        ### === APP OpenLDAP ===
        OPENLDAP_VERSION=latest
        SLAPD_ORGANIZATION="MyFirma Name"
        SLAPD_FQDN=${DOMAINNAME}
        SLAPD_SUFFIX="dc=myfirma,dc=de"
        SLAPD_ROOTDN="cn=admin,${SLAPD_SUFFIX}"
        SLAPD_OU="ou=Users,"
        # Plain-text admin password (pwgen -1cnsB 25 3)
        SLAPD_ROOTPW=-ChangeME=PassWord223
        SLAPD_ROOTPW_HASH=
        SLAPD_PASSWORD_HASH=ARGON2
        SLAPD_PWD_CHECK_QUALITY=2
        SLAPD_PWD_MIN_LENGTH=10
        SLAPD_PWD_MAX_FAILURE=5
        SLAPD_ROOTPW_SECRET=openldap-root-password
        SLAPD_USERPW_SECRET=openldap-user-passwords
        LDAP_BACKUP_TTL=5

        ### === PHPLDAPAdmin Alpine ===
        PLA_VERSION=latest
        PHPLDAPADMIN_LANGUAGE="de_DE"
        PHPLDAPADMIN_PASSWORD_HASH="ssha"
        PHPLDAPADMIN_SERVER_NAME="${SLAPD_ORGANIZATION} LDAP Server"
        PHPLDAPADMIN_SERVER_HOST="ldap://${HOSTNAME0}.${DOMAINNAME}"
        PHPLDAPADMIN_BIND_ID="cn=admin,${SLAPD_SUFFIX}"
        #PHPLDAPADMIN_BIND_ID="cn=admin,dc=int,dc=mydomain,dc=de"

        ```

#### Der erste Start von `OpenLDAP` Docker Container

=== "docker compose (Plugin)"
    ```bash
    cd /opt/openldap
    docker compose up -d

    # Zeigt Status an
    docker compose ps

    # Zeigt Logdaten an
    docker compose logs -f
    ```

=== "docker-compose (Standalone)"

    ```bash
    cd /opt/openldap
    docker-compose up -d

    # Zeigt Status an
    docker-compose ps

    # Zeigt Logdaten an
    docker-compose logs -f
    ```

#### Seite aufrufen

Wenn keine Fehlermeldungen erschienen sind kann man die Startseite von Traefik aufrufen.

!!! abstract "Die Anmeldedaten für `OpenLDAP Login`"

        - **URL**: https://glpi.myfirma.de
        - **User**: glpi
        - **Password**: glpi (Bitte das Passwort sofort ändern)


[^1]: :material-wikipedia: [Wikipedia - OpenLDAP](https://de.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol){target=\_blank}
[^2]: :material-github: [Github - OpenLDAP](https://www.openldap.org/software/repo.html){target=\_blank}
[^3]: :material-docker: [Docker Hub](https://github.com/johann8/openldap-alpine){target=\_blank}

