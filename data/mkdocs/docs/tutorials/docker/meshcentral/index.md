---
Title: MeshCentral
Summary: MeshCentral
Authors: Johann Hahn
Date:    Juli 23, 2024
tags: [Docker, Meschcentral, Remote, Support]
---

# MeshCentral

!!! question "What is MeshCentral?"

    :material-remote-desktop: [`MeshCentral`][MeshCentral]{target=\_blank}[^1] is a `open source`[^2] full computer management web site. With MeshCentral, you can run your own web server to remotely manage and control computers on a local network or anywhere on the internet. Once you get the server started, create device group and download and install an agent on each computer you want to manage. A minute later, the new computer will show up on the web site and you can take control of it. MeshCentral includes full web-based remote desktop, terminal and file management capability.

    [MeshCentral]: https://meshcentral.com/

[^1]: :material-wikipedia: [Wikisuite - MeshCentral](https://wikisuite.org/MeshCentral){target=\_blank}
[^2]: :material-github: [Github - MeshCentral](https://github.com/Ylianst/MeshCentral){target=\_blank}
[^3]: :material-docker: [Docker Hub](https://hub.docker.com/repository/docker/johann8/alpine-meshcentral/general){target=\_blank}
[^4]: :simple-mongodb: [Wikipedia - MongoDB](https://de.wikipedia.org/wiki/MongoDB){target=\_blank}

#### Algemeine Information

!!! note "Allgemeine Information über MeshCentral"

    Am einfachsten lässt sich `MeshCentral` as Docker Container installieren. Man kann entweder [`offizielles MeshCentral Docker Image`][offizielles MeshCentral Docker Image]{target=\_blank} oder ein [`selbstgebautes Docker Image`][selbstgebautes Docker Image]{target=\_blank} benutzen. Für den Fernzugriff muss ein [`MeshCentral Agent`][MeshCentral Agent]{target=\_blank} auf dem Gerät installiert sein.
    Selbstgebautes MeshCentral Docker Image befindet sich auf dem [`Docker Hub`][Docker Hub][^3]. Anweisungen zu Installation und Konfiguration findet man weiter unten.

   
    [MeshCentral Agent]: https://github.com/Ylianst/MeshCentral/tree/master/agents 
    [offizielles MeshCentral Docker Image]: https://ghcr.io/ylianst/meshcentral
    [selbstgebautes Docker Image]: https://hub.docker.com/repository/docker/johann8/alpine-meshcentral/general
    [Docker Hub]: https://hub.docker.com/repository/docker/johann8/alpine-meshcentral/general

### Install MeshCentral as `docker container`

Jeder Container repräsentiert eine einzelne Anwendung, die in einem überbrückten (Bridged) Netzwerk verbunden sind:

<div class="grid cards" markdown>
   - <img src="../../../assets/logos/meshcentral_logo.png" width="25" height="25" /> [__MeshCentral__](https://meshcentral.com/){target=\_blank} Full computer management web site.
   - :simple-mongodb: [__MongoDB__](https://www.mongodb.com/de-de){target=\_blank} Datenbank.
</div>

#### Bereiten Sie Ihr System vor

Bevor Sie `OCS Inventory` installieren, sollten Sie einige Voraussetzungen überprüfen:

!!! info
    - [`Docker`][Docker] Dienst muss auf dem Host installiert sein.
    - Traefik muss als `Docker Service` installiert sein.
    - [`MongoDB`][MongoDB][^4] muss entweder als `Container` im gleichen `Docker Stack` laufen oder als extra `Docker Service` installiert sein.
    - Einen DNS-Record z.B. `mc.myfirma.de` erstellen.

    [Docker]: ../dockerinstall/index.md
    [MongoDB]: https://www.mongodb.com/de-de


#### MeshCentral installieren

!!! note "Alle genutzte Netzwerke anzeigen lassen"

    ```bash
    egrep -r 'SUBNET|IPV4_NETWORK' /opt/*
    ```

??? tip "MeshCentral unter `Rocky Linux | Oracle` einrichten"

    === "Create folders"

        ```bash
        DOCKERDIR=/opt/meshcentral
        mkdir -p ${DOCKERDIR}/data/{mongodb,mc}
        mkdir -p ${DOCKERDIR}/data/mc/{data,user_files,backups,web}
        mkdir -p ${DOCKERDIR}/data/mongodb/dbdata
        cd ${DOCKERDIR}
        tree -d -L 5 ${DOCKERDIR}
        ```
    === "Download config files"

        ```bash
        DOCKERDIR=/opt/meshcentral
        cd ${DOCKERDIR}
        wget https://raw.githubusercontent.com/johann8/alpine-meshcentral/master/docker-compose.yml
        wget https://raw.githubusercontent.com/johann8/alpine-meshcentral/master/docker-compose.override.yml
        wget https://raw.githubusercontent.com/johann8/alpine-meshcentral/master/.env
        ```

#### MeshCentral: Customise compose and config files

??? tip "MeshCentral: Customise compose and config files"

    === "docker-compose.yml"

        ``` yaml
        ---
        networks:
          meshcentralNet:
            driver: bridge
            driver_opts:
              com.docker.network.enable_ipv6: "true"
            ipam:
              driver: default
              config:
                - subnet: ${IPV4_NETWORK:-172.26.8}.0/24
                - subnet: ${IPV6_NETWORK:-fd4d:6169:6c63:6f08}::/64

        services:

          #
          ### === MeshCentral Container ===
          #
          meshcentral:
            restart: on-failure:10
            container_name: meshcentral
            image: ghcr.io/ylianst/meshcentral:${MC_VERSION}
            #image: johann8/alpine-meshcentral:${MC_VERSION}
            # Only for Apache comment out
            #ports:
              #- 127.0.0.1:8084:443  #MeshCentral will moan and try everything not to use port 80, but you can also use it if you so desire, just change the config.json according to your needs
              #- 8086:443
            environment:
              - TZ=${TZ}
              - HOSTNAME=${HOSTNAME_MC}
              - REVERSE_PROXY=${REVERSE_PROXY}
              - REVERSE_PROXY_TLS_PORT=${REVERSE_PROXY_TLS_PORT}
              - NODE_ENV=${NODE_ENV}
              - USE_TRAEFIK=${USE_TRAEFIK}
              - PORT=${PORT}
              - USE_MONGODB=${USE_MONGODB}
              - MONGO_URL=${MONGO_URL}
              - MONGO_DB_NAME=${MONGO_DB_NAME}
              - MONGO_MC_USERNAME=${MONGO_MC_USERNAME}
              - MONGO_MC_USER_PASSWORD=${MONGO_MC_USER_PASSWORD}
              - IFRAME=${IFRAME}
              - ALLOW_NEW_ACCOUNTS=${ALLOW_NEW_ACCOUNTS}
              - WEBRTC=${WEBRTC}
              - LOCALSESSIONRECORDING=${LOCALSESSIONRECORDING}
              - MINIFY=${MINIFY}
            volumes:
              # config.json and other important files live here. A must for data persistence
              - "${DOCKERDIR}/data/mc/data:/opt/meshcentral/meshcentral-data"
              # where file uploads for users live
              - "${DOCKERDIR}/data/mc/user_files:/opt/meshcentral/meshcentral-files"
              # location for the meshcentral-backups
              - "${DOCKERDIR}/data/mc/backups:/opt/meshcentral/meshcentral-backups"
              # location for site customization files
              - "${DOCKERDIR}/data/mc/web:/opt/meshcentral/meshcentral-web"
            env_file:
              - .env
            depends_on:
              - mongodb
            networks:
            - meshcentralNet

          #
          ### === MongoDB Container ===
          #
          mongodb:
            #image: mongo:latest
            image: mongo:6.0
            container_name: mongodb
            restart: on-failure:10
            environment:
              - MONGO_INITDB_ROOT_USERNAME=${MONGO_INITDB_ROOT_USERNAME}
              - MONGO_INITDB_ROOT_PASSWORD=${MONGO_INITDB_ROOT_PASSWORD}
            volumes:
              # mongodb data-directory - A must for data persistence
              - "${DOCKERDIR}/data/mongodb/dbdata:/data/db"
            env_file:
              - .env
            networks:
              - meshcentralNet


        ```

    === "docker-compose.override.yml"

        ``` yaml
        ---
        services:

          meshcentral:
            labels:
              - "traefik.enable=true"

              ### ==== to https ====
              - "traefik.http.routers.meshcentral-secure.entrypoints=websecure"
              - "traefik.http.routers.meshcentral-secure.rule=Host(`${HOSTNAME0}.${DOMAINNAME}`)"
              - "traefik.http.routers.meshcentral-secure.tls=true"
              - "traefik.http.routers.meshcentral-secure.tls.certresolver=production"  # für eigene Zertifikate
              ### ==== to service ====
              - "traefik.http.routers.meshcentral-secure.service=meshcentral"
              - "traefik.http.services.meshcentral.loadbalancer.server.port=${PORT}"
              - "traefik.http.services.meshcentral.loadbalancer.passHostHeader=true"
              - "traefik.docker.network=proxy"
              - "traefik.http.routers.meshcentral-secure.middlewares=secHeaders@file"
              #- "traefik.http.routers.meshcentral-secure.middlewares=secHeaders@file,traefik-compress@file"
              #- "traefik.http.routers.meshcentral-secure.middlewares=authelia@docker,rate-limit@file,secHeaders@file"
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
        DOCKERDIR=/opt/meshcentral

        ### === Network ===
        DOMAINNAME=myfirma.de
        HOSTNAME0=mc
        HOSTNAME_MC=${HOSTNAME0}.${DOMAINNAME}
        PORT=4430
        REVERSE_PROXY=${HOSTNAME_MC}
        REVERSE_PROXY_TLS_PORT=443
        IPV4_NETWORK=172.24.8
        IPV6_NETWORK=fd4d:6169:6c63:6f08

        #
        ### === APP Meshcentral ===
        #
        # Set MeshCentral to Produktion
        NODE_ENV=production

        # meshcentral version
        MC_VERSION=latest

        # RP traefik
        USE_TRAEFIK="true"

        # use mongodb
        # generate password: pwgen -1cnsB 25 1
        USE_MONGODB=true
        MONGO_MC_USERNAME=meshuser
        MONGO_MC_USER_PASSWORD=MySuperPassword123
        MONGO_DB_NAME=meshcentral

        # set already exist mongo connection string url here
        MONGO_URL=

        # or set following init params for new mongodb, use it with docker-compose file with mongodb version
        # pwgen -1cnsB 30 1
        MONGO_INITDB_ROOT_USERNAME=admin
        MONGO_INITDB_ROOT_PASSWORD=MySuperRootPassword456

        #set to true if you wish to enable iframe support
        IFRAME=false

        #set to false if you want disable self-service creation of new accounts besides the first (admin)
        ALLOW_NEW_ACCOUNTS=false

        #set to true to enable WebRTC - per documentation it is not officially released with meshcentral, but is solid enough to work with. Use with caution
        WEBRTC=true

        # set to true to allow session recording
        LO CALSESSIONRECORDING=false

        # set to enable or disable minification of json, reduces traffic
        MINIFY=true
        ```

    === "data/mc/data/config.json"

        ``` yaml
        SESSIONKEY=$(pwgen -1ycnsB --remove-chars=". ^ $ * ; ~ ' _ ? @ : , & | ! \" \`" 50); echo "SessionKey: ${SESSIONKEY}"
        DBENCRYPTKEY=$(pwgen -1ycnsB --remove-chars=". ^ $ * ; ~ ' _ ? @ : , & | ! \" \`" 35); echo "DbEncryptKey: ${DBENCRYPTKEY}"
        cd /opt/meshcentral
        cat > data/mc/data/config.json << 'EOL'
        {
          "$schema": "http://info.meshcentral.com/downloads/meshcentral-config-schema.json",
          "settings": {
            "cert": "mc.myfirma.de",
            "WANonly": true,
            "_LANonly": true,
            "_debug": true,
            "sessionKey": "ChangeME=MySuperSessionKey777777",
            "_CookieEncoding": "hex",
            "Port": 4430,
            "AliasPort": 443,
            "RedirPort": 800,
            "RedirAliasPort": 80,
            "TLSOffload": "traefik",
            "TrustedProxy": "traefik",
            "_UserAllowedIP": "127.0.0.1,192.168.0.0/16",
            "_UserBlockedIP": "127.0.0.1,::1",
            "_MaintenanceMode": true,
            "mongoDb": "mongodb://meshuser:MySuperPassword123@mongodb:27017/meshcentral",
            "MongoDbName": "meshcentral",
            "Mongodbcol": "meshcentral",
            "MongoDbBulkOperations": true,
            "DbEncryptKey": "ChangeME=MySupaerPW8888",
            "_MongoDbChangeStream": true,
            "agentSignLock": true,
            "_AgentPing": 40,
            "AgentPong": 60,
            "_BrowserPing": 40,
            "BrowserPong": 60,
            "AllowHighQualityDesktop": true,
            "desktopMultiplex": true,
            "plugins": {
              "enabled": true
            },
            "WebRTC": true,
            "WebRtcConfig": {
              "iceServers": [
                { "urls": "stun:stun.l.google.com:19302" },
                { "urls": "stun:stun.gmx.de:3478" },
                { "urls": "stun:stun.1und1.de:3478" }
              ]
            },
            "watchDog": { "interval": 100, "timeout": 400 },
            "SelfUpdate": false,
            "nice404": true,
            "_CompressionWirdUnterTraefikEingeschaltetDeshalbCompressionUntenAus": true,
            "Compression": true,
            "WsCompression": true,
            "AgentWsCompression": false,
            "NoAgentUpdate": 0,
            "AllowFraming": false,
            "_AllowLoginToken": true,
            "AuthLog": "/opt/meshcentral/meshcentral-data/auth.log",
            "no2FactorAuth": false,
            "CookieIpCheck": true,
            "MaxInvalidLogin": {
              "time": 5,
              "count": 5,
              "coolofftime": 30
            },
            "AutoBackup": {
              "backupIntervalHours": 24,
              "keepLastDaysBackup": 15,
              "backupPath": "/opt/meshcentral/meshcentral-backups"
            },
            "DesktopMultiplexor": true
          },
          "domains": {
                "": {
            "title": "REMOTE SUPPORT PORTAL",
            "title2": "MyFirma",
            "NewAccounts": false,
            "AllowSavingDeviceCredentials": true,
            "scrollToTop": true,
            "nightMode": 0,
            "welcomeText": "<a href='https://myfirma.de/' target='_blank' style='text-decoration: none;'>Wassermann Gruppe</a> - Ihr Dienstleister in Berlin und Brandenburg",
            "_welcomeText": "<a href='https://myfirma.de/' target='_blank' onMouseOver='this.style.color='red'' style='text-decoration: none;'>Wassermann Gruppe</a> - Ihr Dienstleister in Berlin und Brandenburg",
            "Footer": "<a href='https://myfirma.de/' target='_blank' style='text-decoration: none;'>Wassermann Gruppe</a> - Ihr Dienstleister in Berlin und Brandenburg",
            "_LoginFooter": "<a href='https://myfirma.de/' target='_blank' style='text-decoration: none;'>Wassermann Gruppe</a> - Ihr Dienstleister in Berlin und Brandenburg",
            "_welcomePicture": "sdc.jpg",
            "__comment__": "Autogenehmigung nach 30 Sec",
            "consentMessages": {
              "con sentTimeout" : 30,
              "autoAcceptOnTimeout": true
            },
            "agentCustomization": {
              "_displayName": "MyCompany Agent",
              "_description": "Allows MyCompany to provide full service remote support to this computer.",
              "companyName": "Wassermann Gruppe",
              "_serviceName": "MyCompany Remote Support Agent",
              "_fileName": "MyCompanyAgent"
            },
            "_agentConfig": [ "webSocketMaskOverride=1" ],
            "certUrl": "mc.reduktor.de",
            "LoginKey":"HITC94",
            "agentInviteCodes": true,
            "PasswordRequirements": {
              "min": 10,
              "max": 64,
              "upper": 1,
              "lower": 1,
              "numeric": 1,
              "nonalpha": 1,
              "Force2Factor": true,
              "skip2factor": "127.0.0.1,172.26.0.0/24"
            },
            "terminal": {
              "linuxshell": "any",
              "launchCommand": {
                "linux": "cd /tmp\nclear\necho \"Hello Linux\"\n",
                "_darwin": "clear\necho \"Hello MacOS\"\n",
                "_freebsd": "clear\necho \"Hello FreeBSD\"\n"
              }
            },
            "_userNameIsEmail": true,
            "_ManageAllDeviceGroups":["info@reduktor.de"],
            "_IpBlockedUserRedirect": "https://www.microsoft.com",
            "_sessionRecording": {
              "filepath": "/opt/meshcentral/meshcentral-recordings"
            },
            "GuestDeviceSharing": {
              "__comment__": "Time in Minutes",
              "maxSessionTime": 240
            },
            "novnc":true,
            "mstsc":true,
            "ssh":true,
            "GeoLocation": true
            }
          },
          "_letsencrypt": {
            "__comment__": "Requires NodeJS 8.x or better, Go to https://letsdebug.net/ first before>",
            "_email": "myemail@mydomain.com",
            "_names": "myserver.mydomain.com",
            "production": false
          },
          "smtp": {
          "host": "mx01.myfirma.de",
          "port": 587,
          "from": "meshcentral@myfirma.de",
          "user": "meshcentral@myfirma.de",
          "pass": "MySupperPW33",
          "tls": false
          }
        }
        EOL
        ```

#### Der erste Start von `MeshCentral` Docker Container

=== "docker compose (Plugin)"
    ```bash
    cd /opt/meshcentral
    docker compose up -d

    # Zeigt Status an
    docker compose ps

    # Zeigt Logdaten an
    docker compose logs -f
    ```

=== "docker-compose (Standalone)"

    ```bash
    cd /opt/meshcentral
    docker-compose up -d

    # Zeigt Status an
    docker-compose ps

    # Zeigt Logdaten an
    docker-compose logs -f
    ```

#### MongoDB `Benutzer` und `Passwort` erstellen

??? note "MongoDB `Benutzer` und `Passwort` erstellen"

    ```bash
    DOCKERDIR=/opt/meshcentral
    cd ${DOCKERDIR}
    cat .env |grep MONGO_
    docker compose exec mongodb mongosh --host localhost -u admin
    use meshcentral

    # Das Passwort ist in der Datei .env definiert
    db.createUser(
        {
            user: "meshuser",
            pwd: "-ChangeMe-",
            roles:[
                {
                    role: "readWrite",
                    db: "meshcentral"
                }
            ]
        }
    );

    show users
    quit
    ```
    #### Docker Stack neu starten

    ```bash
    docker-compose down && docker-compose up -d
    docker-compose logs -f
    ```

#### Seite aufrufen

Wenn keine Fehlermeldungen erschienen sind kann man die Startseite von `MeshCentral` aufrufen.

