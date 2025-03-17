---
Title: Traefik
Summary: Traefik Reverse Proxy 
Authors: Johann Hahn
Date:    Juli 09, 2024
tags: [docker, traefik,proxy]
---

# :simple-traefikproxy: Traefik Reverse Proxy 

!!! question "What Traefik?"

    :simple-traefikproxy: [`Traefik`][Traefik][^1] is a modern HTTP reverse proxy (RP) and load balancer that makes deploying microservices easy.

    [Traefik]: https://github.com/traefik/traefik

[^1]: [Wikipedia - Traefik](https://de.wikipedia.org/wiki/Traefik)

## Install Trafik as `docker container`

#### Bereiten Sie Ihr System vor

Bevor Sie `Traefik` installieren, sollten Sie einige Voraussetzungen überprüfen:

!!! info
    - [`Docker`][Docker] Dienst muss auf dem Host installiert sein.
    - Traefik benötigt offene Ports `80` und `443` für eingehende Verbindungen. Stellen Sie daher sicher, dass Ihre Firewall diese nicht blockiert.
    - Falls ein anderer Dienst auf diesen Ports lauscht, bitte diesen Dienst stoppen und dektivieren.
    - Einen DNS-Record z.B. `traefik.myfirma.de` erstellen.

    [Docker]: ../dockerinstall/index.md

#### Traefik installieren

??? tip "Traefik unter `Rocky Linux | Oracle` einrichten"

    === "Create docker network"
        ```bash
        docker network ls
        docker network create proxy
        docker network ls
        ```

    === "Create folders"
        ```bash
        DOCKERDIR=/opt/traefik
        mkdir -p ${DOCKERDIR}/data/{certs,config,logs,secrets}
        touch ${DOCKERDIR}/data/config/acme.json
        chmod 600 ${DOCKERDIR}/data/config/acme.json
        touch ${DOCKERDIR}/data/secrets/basic_auth_credentials
        chmod 0600 /opt/traefik/data/secrets/basic_auth_credentials
        cd ${DOCKERDIR}
        tree -d -L 3 ${DOCKERDIR}
        ```
    === "Generate password"
        ```bash
        # htpasswd and pwgen must be installed: `dnf -y httpd-tools pwgen`
        DOCKERDIR=/opt/traefik
        USER_NAME=user1
        PASSWORD_LENGTH=20
        TRAEFIK_PASSWORD=$(pwgen -1ycnsB --remove-chars=". ^ $ * ; ~ ' _ ? @ : , & | ! \" \`" ${PASSWORD_LENGTH} 1)
        echo ${TRAEFIK_PASSWORD} > ${DOCKERDIR}/traefik_dashboard_password
        echo $(htpasswd -nb ${USER_NAME} ${TRAEFIK_PASSWORD}) | sed -e s/\\$/\\$\\$/g

        # Result - Will be in treafik secret file inserted (data/secrets/basic_auth_credentials) 
        user1:$$apr1$$ht6O3iSE$$5YxApoPCqeGyMNRsCvXwh/

        # add to file basic_auth_credentials
        vim  ${DOCKERDIR}/data/secrets/basic_auth_credentials
        ----
        user1:$$apr1$$ht6O3iSE$$5YxApoPCqeGyMNRsCvXwh/
        ----

        # You can also create the password credentials with the help of "HTPasswd Generator"
        URL: https://www.web2generators.com/apache-tools/htpasswd-generator
        ```
    === "Download config files"
        ```bash
        DOCKERDIR=/opt/traefik
        cd ${DOCKERDIR}
        wget https://raw.githubusercontent.com/johann8/traefik/master/.env
        wget https://raw.githubusercontent.com/johann8/traefik/master/docker-compose.yml
        wget https://raw.githubusercontent.com/johann8/traefik/master/assets/config/traefik.yml -P data/conf/
        wget https://raw.githubusercontent.com/johann8/traefik/master/assets/config/dynamic_conf.yml -P data/conf/
        ```

    === "Rotate traefik log file"
        ```bash
        cat > /etc/logrotate.d/traefik << 'EOL'
        /opt/traefik/data/logs/*.log {
          weekly
          rotate 5
          compress
          #size=1M
          missingok
          notifempty  
          postrotate
            docker kill --signal="USR1" traefik
          endscript
        }  
        EOL
        ls -la /etc/logrotate.d/
        ls -lah /opt/traefik/data/logs/

        # Run logrotate manuell
        logrotate --force /etc/logrotate.d/traefik
        ls -lah /opt/traefik/data/logs/
        ```

#### Traefik: Customise compose and config files

??? tip "Traefik: Customise compose and config files"

    === "docker-compose.yml"

        ``` yaml
        ---
        #################### NETWORKS
        networks:
          default:
            driver: bridge
          proxy:
            external: true
            enable_ipv6: true
            ipam:
              config:
                - subnet: ${SUBNET}.0/24

        #################### SECRETS
        secrets:
          basic_auth_credentials:
            file: ${DOCKERDIR}/data/secrets/basic_auth_credentials

        #################### SERVICES
        services:
        #
        ### === Traefik Container ===
        #
          traefik:
            image: traefik:latest
            container_name: traefik
            restart: unless-stopped
            security_opt:
              - no-new-privileges:true
            networks:
              proxy:
                ipv4_address: ${SUBNET}.254
            ports:
              - target: 80
                published: 80
                protocol: tcp
                mode: host
              - target: 443
                published: 443
                protocol: tcp
                mode: host
              - target: 443
                published: 443
                protocol: udp
                mode: host
            environment:
              - PUID=$PUID
              - PGID=$PGID
              - TZ=$TZ
              - HTPASSWD_FILE=/run/secrets/basic_auth_credentials                        # HTTP Basic Auth Credentials
            secrets:
              - basic_auth_credentials
            healthcheck:
              test: ["CMD", "traefik", "healthcheck", "--ping"]
              interval: 20s
              timeout: 1s
              retries: 3
              start_period: 10s
            volumes:
              - /etc/localtime:/etc/localtime:ro
              - /var/run/docker.sock:/var/run/docker.sock:ro
              - $DOCKERDIR/data/conf:/etc/traefik
              - $DOCKERDIR/data/certs:/etc/traefik/certs
              - $DOCKERDIR/data/logs:/var/log/traefik:rw
            networks:
              - proxy
            labels:
              - "traefik.enable=true"
              - "traefik.docker.network=proxy"
              - "traefik.http.routers.traefik-secure.entrypoints=websecure"
              - "traefik.http.routers.traefik-secure.rule=Host(`${HOSTNAME01}.${DOMAINNAME}`) || Host(`storage.${DOMAINNAME}`) || Host(`example.${DOMAINNAME}`)"
              - "traefik.http.routers.traefik-secure.tls=true"
              - "traefik.http.routers.traefik-secure.tls.certresolver=production"        # für eigene Zertifikate
              - "traefik.http.routers.traefik-secure.service=api@internal"
              - "traefik.http.routers.traefik-secure.middlewares=rate-limit@file,secHeaders@file,traefik-basic-auth@file,autodetectContenttype@file"
              #- "traefik.http.routers.traefik-secure.middlewares=rate-limit@file,secHeaders@file,authelia@docker"
              # helthcheck
              - "traefik.http.routers.pingweb.rule=PathPrefix(`/ping`)"
              - "traefik.http.routers.pingweb.service=ping@internal"
              - "traefik.http.routers.pingweb.entrypoints=websecure"

        #
        ### === Certdumper Container ===
        #
          certdumper:
            image: humenius/traefik-certs-dumper
            container_name: certdumper
            network_mode: none
            #command: --restart-containers container_name1, container_name2
            volumes:
              - "$DOCKERDIR/data/certs:/traefik:ro"                             # mounten Sie den Ordner, der Traefiks `acme.json' Datei enthält
              - "$DOCKERDIR/data/assets/ssl:/output:rw"                         # SSL-Ordner von mailcow einhängen
              #- "$DOCKERDIR/data/assets/script/hook.sh:/hook/hook.sh:ro"       # Hook script running after cert dump
              - "/var/run/docker.sock:/var/run/docker.sock:ro"                  # Um Container starten zu können
            restart: always
        ```

    === ".env"

        ``` yaml
        #
        ### === SYSTEM ===
        #
        TZ=Europe/Berlin
        DOCKERDIR=/opt/traefik
        PUID=1000
        PGID=1000

        #
        ### === Network ===
        #
        DOMAINNAME=myfirma.de
        HOSTNAME01=traefik
        SUBNET="172.18.0"

        #
        ### === Basic Auth ===
        #
        # User: user1
        # PW: Hz9yy{7fU{7Nvbr}qk%(        
        ```

    === "data/conf/traefik.yml"

        ``` yaml
        global:
          checkNewVersion: true                # Periodically check if a new version has been released
          sendAnonymousUsage: false            # true by default

        # (Optional) Log information
        # ---
        log:
        level: INFO                            # DEBUG, INFO, WARNING, ERROR, CRITICAL
        format: common                         # common, json, logfmt
        filePath: /var/log/traefik/traefik.log

        # (Optional) Accesslog
        # ---
        accesslog:
          format: common                       # common, json, logfmt
          filePath: /var/log/traefik/access.log
          filters:
            statusCodes:
              - "400-600"

        # (Optional) Enable API and Dashboard
        # ---
        api:
          dashboard: true                      # true by default
          insecure: false                      # true - Don't do this in production!

        # Entry Points configuration
        # ---
        entryPoints:
          ping:
            address: :88

          web:
            address: :80
            http:
              #middlewares:                    # For crowdsec
                #- crowdsec-bouncer@file
              redirections:
                entryPoint:
                  to: websecure
                  scheme: https

          websecure:
            address: :443
            http:
              #middlewares:                     # For crowdsec
                #- crowdsec-bouncer@file
            http3:
              advertisedPort: 443

        # Ping
        # ---
        ping:
          entryPoint: ping

        # Configure your CertificateResolver here...
        # ---
        certificatesResolvers:
          #staging:
          #  acme:
          #    email: le@myfirma.de
          #    storage: /etc/traefik/certs/acme.json
          #    caServer: "https://acme-staging-v02.api.letsencrypt.org/directory"
          #    httpChallenge:
          #      entryPoint: web

          production:
            acme:
              email: le@myfirma.de
              storage: /etc/traefik/certs/acme.json
              tlsChallenge: {}
              httpChallenge:
                entryPoint: web
              #dnsChallenge:
                #provider: digitalocean
                #delayBeforeCheck:

        providers:
          docker:
            exposedByDefault: false                          # Default is true
            endpoint: "unix:///var/run/docker.sock"
            network: "proxy
          file:
            # watch for dynamic configuration changes
            directory: /etc/traefik
            watch: true
            filename: "/etc/traefik/dynamic_conf.yml"
          providersThrottleDuration: 10                        # Configuration reload frequency

        ## For ProxMox VE
        #serversTransport:
          #insecureSkipVerify: true

        pilot:
          dashboard: false 
        ```

    === "data/conf/dynamic_conf.yml"

        ```yaml
        tls:
          options:
            default:
              minVersion: VersionTLS12
              cipherSuites:
                - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
                - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
                - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
                - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
                - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
                - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
              curvePreferences:
                - CurveP521
                - CurveP384
              sniStrict: false                          # true, wenn Zert. auf die FQDN ausgestellt ist

        # http routing section
        # ---
        http:
          middlewares:

            # A basic authentification middleware, to protect the Traefik dashboard to anyone except myself
            # Use with traefik.http.routers.myRouter.middlewares: "traefik-auth@file"
            traefik-auth:
              basicAuth:
                users:
                  - "user1:$$apr1$$ht6O3iSE$$5YxApoPCqeGyMNRsCvXwh/"   # htpasswd -nB user1; PW: MySuperPW
                realm: "Traefik dashboard on OracleL8"

            https-redirect:
              redirectScheme:
                scheme: https
                permanent: true

            # Use with traefik.http.routers.myRouter.middlewares: "ipallowlist@file"
            ipallowlist:
              IPAllowList:
                sourceRange:
                  - "10.0.0.0/8"
                  - "192.168.0.0/16"
                  - "172.0.0.0/12"

            # # Use with traefik.http.routers.myRouter.middlewares: "rate-limit@file"
            rate-limit:
              rateLimit:
                average: 100            # an average of 100 requests per second is allowed
                period: 10s             # Period is in s oder m: 100 requests per 10 second is allowed
                burst: 100              # a burst (max) of 50 requests is allowed

            # Use with traefik.http.routers.myRouter.middlewares: "traefik-compress@file"
            traefik-compress:
              compress: {}

            # # Use with traefik.http.routers.myRouter.middlewares: "secured@file"
            secured:
              chain:
                middlewares:
                  - ipallowlist
                  - secHeaders
                  - traefik-compress

            # Middleware for Authelia
            authelia:
              forwardAuth:
                #address: "http://authelia:9091/api/authz/forward-auth"                   # Since Authelia version 4.38
                address: "http://authelia:9091/api/verify?rd=https://auth.myfirma.de"     # Since Authelia version 4.38
              trustForwardHeader: true
                #authResponseHeaders:
                  #- "Remote-User"
                  #- "Remote-Groups"
                  #- "Remote-Name"
                  #- "Remote-Email"

            # Middleware for crowdsec
            crowdsec-bouncer:
              forwardauth:
                address: http://bouncer-traefik:8080/api/v1/forwardAuth                  # Since Authelia version 4.38
                trustForwardHeader: true

            # needed for traefik v3 - see https://doc.traefik.io/traefik/v3.0/migration/v2-to-v3/
            autodetectContenttype:
              contentType: {}

            secHeaders:
              headers:
                BrowserXssFilter: true
                ContentTypeNosniff: true
                ForceSTSHeader: true
                frameDeny: true

                #HSTS Configuration
                STSIncludeSubdomains: true
                STSPreload: true
                stsSeconds: 31536000
                customFrameOptionsValue: "SAMEORIGIN"
                customRequestHeaders:
                  X-Forwarded-Proto: https
                  X-Frame-Options: "SAMEORIGIN"
        ```
#### Der erste Start von `Traefik` Docker Container

=== "docker compose (Plugin)"
    ```bash
    cd /opt/traefik
    docker compose up -d

    # Zeigt Status an
    docker compose ps

    # Zeigt Logdaten an
    docker compose logs -f
    ```

=== "docker-compose (Standalone)"

    ```bash
    cd /opt/traefik
    docker-compose up -d

    # Zeigt Status an
    docker-compose ps

    # Zeigt Logdaten an
    docker-compose logs -f
    ```

#### Seite aufrufen

Wenn keine Fehlermeldungen erschienen sind kann man die Startseite von Traefik aufrufen.

!!! abstract "Die Anmeldedaten für `Traefik Dashboard`"

	- **URL**: https://traefik.myfirma.de
	- **User**: user1
	- **Password**: (cat /opt/traefik/traefik_dashboard_password)

	*Benutzen Sie die Anmeldedaten, die Sie generiert haben.*
