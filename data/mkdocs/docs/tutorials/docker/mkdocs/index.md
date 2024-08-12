---
Title: MkDocs 
Summary: A brief description of mkdocs.
Authors: Johann Hahn
Date:    Juli 09, 2024
tags: [Docker, mkdocs, docs]
---

# MkDocs

!!! question "📖 What is Material for MkDocs?"

    [Material for MkDocs] is a powerful documentation framework on top of [MkDocs], a static site generator for project documentation.[^1] If you're familiar with Python, you can install Material for MkDocs with `pip`, the Python package manager. If not, we recommend using [`docker`][docker].

  [^1]:
    In 2016, Material for MkDocs started out as a simple theme for MkDocs, but
    over the course of several years, it's now much more than that – with the
    many built-in plugins, settings, and countless customization abilities,
    Material for MkDocs is now one of the simplest and most powerful frameworks
    for creating documentation for your project.

    [Material for MkDocs]: https://squidfunk.github.io/mkdocs-material/getting-started/
    [MkDocs]: https://www.mkdocs.org
    [docker]: https://docs.int.wassermanngruppe.de/tutorials/docker/

#### MkDocs - Create docker config files

??? tip "MkDocs: Create docker config files"

    === "docker-compose.yml"

        ``` yaml
        networks:
          mkdocsNet:
            ipam:
              driver: default
              config:
                - subnet: ${IPV4_NETWORK:-172.26.27}.0/24
                - subnet: ${IPV6_NETWORK:-fd4d:6169:6c63:6f27}::/64

        services:

        #
        ### === NGINX Container ===
        #
          nginx-mkdocs:
            image: nginx:alpine
            container_name: nginx-mkdocs
            restart: on-failure:10
            #mem_limit: 4G
            #mem_reservation: 2G
            security_opt:
              - no-new-privileges:true
            #ports:
              #- "8005:80"
            volumes:
              - ./data/mkdocs/site:/var/www/html
              - ./data/nginx/templates:/etc/nginx/templates
            environment:
              - TZ=${TZ}
              - NGINX_HOST=localhost                # set your local domain or your live domain
            networks:
              mkdocsNet:
                ipv4_address: ${IPV4_NETWORK:-}.5
                ipv6_address: ${IPV6_NETWORK:-fd4d:6169:6c63:6f27}::5

        #
        ### === MKDOCS Container ===
        #

        #  mkdocs:
        #    container_name: mkdocs
        #    hostname: mkdocs
        #    image: squidfunk/mkdocs-material:${MKDOCS_VERSION}
        #    restart: unless-stopped
        #    security_opt:
        #      - no-new-privileges:true
        #    #ports:
        #      #- "8005:8000"
        #    volumes:
        #      - ./data/mkdocs:/docs
        #    stdin_open: true
        #    tty: true
        #    environment:
        #      TZ: Europe/Berlin
        #    networks:
        #      - mkdocsNet
        ```

    === "docker-compose.override.yml"

        ``` yaml
        services:

          nginx-mkdocs:
            labels:
              - "traefik.enable=true"
              ### ==== to https ====
              - "traefik.http.routers.nginx-mkdocs-secure.entrypoints=websecure"
              - "traefik.http.routers.nginx-mkdocs-secure.rule=Host(`${HOSTNAME0}.${DOMAINNAME}`)"
              - "traefik.http.routers.nginx-mkdocs-secure.tls=true"
              - "traefik.http.routers.nginx-mkdocs-secure.tls.certresolver=production"                      # für eigene Zertifikate
              - "traefik.http.routers.nginx-mkdocs-secure.tls.domains[0].main=${HOSTNAME0}.${DOMAINNAME}"   # Für Letsencrypt: Set main domain
              ### ==== to service ====
              - "traefik.http.routers.nginx-mkdocs-secure.service=nginx-mkdocs"
              - "traefik.http.services.nginx-mkdocs.loadbalancer.server.port=${PORT0}"
              - "traefik.docker.network=proxy"
              ### ==== redirect to authelia for secure login ====
              - "traefik.http.routers.nginx-mkdocs-secure.middlewares=rate-limit@file,secHeaders@file"
              #- "traefik.http.routers.nginx-mkdocs-secure.middlewares=authelia@docker,rate-limit@file,secHeaders@file"
            networks:
              - proxy

        networks:
          proxy:
            external: true

        ```

    === ".env"

        ```
        ### === SYSTEM ===
        TZ=Europe/Berlin
        DOCKERDIR=/opt/mkdocsmaterial

        ### === Network ===
        DOMAINNAME=myfirma.de
        HOSTNAME0=docs
        PORT0=80
        IPV4_NETWORK=172.24.27
        IPV6_NETWORK=fd4d:6169:6c63:6f27

        ### === Mkdocs ====
        MKDOCS_VERSION=latest
        ```
#### MkDocs - Create mkdocs config and folder tree

??? tip "MkDocs: Create mkdocs config files and folder tree"

    === "mkdocs.yml"
        ```yaml
        site_name: HITC Docs
        site_url: https://docs.myfirma.de/
        site_author: Vorname Nachname
        site_description: >-
          Write your documentation in Markdown and create a professional static site in
          minutes – searchable, customizable, in 60+ languages, for all devices

        # Repository
        #repo_name: squidfunk/mkdocs-material
        #repo_url: https://github.com/squidfunk/mkdocs-material

        # Copyright
        copyright: >
          Copyright &copy; 2023 - 2024 Myfirma -
          <a href="#__consent"> Change cookie settings</a>

        # Page tree
        nav:
          - 'Home': 'index.md'
          - 'Tutorials':
            - 'Docker':
            - tutorials/docker/index.md
            - 'Docker Install': 'tutorials/docker/dockerinstall/index.md'
            - 'Traefik RP': 'tutorials/docker/traefik/index.md'
            - 'Authelia': 'tutorials/docker/authelia/index.md'
            - 'Bacula': 'tutorials/docker/bacula/index.md'
            - 'GLPI': 'tutorials/docker/glpi/index.md'
            - 'OCS Inventory': 'tutorials/docker/ocsinventory/index.md'
            - 'Mkdocs': 'tutorials/docker/mkdocs/index.md'
            - 'OpenLDAP': 'tutorials/docker/openldap/index.md'
            - 'PhpLdapAdmin': 'tutorials/docker/phpldapadmin/index.md'
          - 'About': 'about.md'

        # Configuration
        theme:
          name: 'material'
            language: de
            features:
              - announce.dismiss
              - content.action.edit
              - content.action.view
              - content.code.annotate
              - content.code.copy
              # - content.code.select
              # - content.footnote.tooltips
              # - content.tabs.link
              - content.tooltips
              #- header.autohide
              #- navigation.expand
              #- navigation.footer
              - navigation.indexes
              #- navigation.instant
              # - navigation.instant.prefetch
              # - navigation.instant.progress
              # - navigation.prune
              #- navigation.sections
              - navigation.tabs
              # - navigation.tabs.sticky
              - navigation.path
              - navigation.top
              - navigation.tracking
              - search.highlight
              - search.share
              - search.suggest
              - toc.follow
              # - toc.integrate

          palette:
          # Palette toggle for automatic mode
            - media: "(prefers-color-scheme)"
              toggle:
                icon: material/brightness-auto
                name: Switch to light mode

          # Light mode
          - media: "(prefers-color-scheme: light)"
            toggle:
              icon: material/brightness-7
              name: Switch to dark mode
            scheme: default
            primary: indigo
            accent: indigo
            toggle:
              icon: material/weather-sunny
              name: Switch to dark mode

          # Dark mode
          - media: "(prefers-color-scheme: dark)"
            scheme: slate
            #primary: black
            #accent: indigo
            primary: indigo
            accent: blue
            toggle:
              icon: material/brightness-4
              name: Switch to system preference

          font:
            text: Roboto
            code: Roboto Mono

          #  favicon: assets/favicon.png
          icon:
            logo: logo

        # Plugins
        plugins:
          #- blog
          - search:
              separator: '[\s\u200b\-_,:!=\[\]()"`/]+|\.(?!\d)|&[lg]t;|(?!\b)(?=[A-Z][a-z])'
          - minify:
              minify_html: true

        # Extensions
        markdown_extensions:
          - abbr
          - admonition
          - attr_list
          - def_list
          - footnotes
          - md_in_html
          - toc:
              permalink: true
          - pymdownx.arithmatex:
              generic: true
          - pymdownx.betterem:
              smart_enable: all
          - pymdownx.caret
          - pymdownx.details
          - pymdownx.emoji:
              emoji_generator: !!python/name:material.extensions.emoji.to_svg
              emoji_index: !!python/name:material.extensions.emoji.twemoji
          - pymdownx.highlight:
              anchor_linenums: true
              line_spans: __span
              pygments_lang_class: true
          - pymdownx.inlinehilite
          - pymdownx.keys
          - pymdownx.magiclink:
              normalize_issue_symbols: true
          - pymdownx.mark
          - pymdownx.smartsymbols
          - pymdownx.snippets
          - pymdownx.superfences
          - pymdownx.tabbed:
              alternate_style: true
              combine_header_slug: true
          - pymdownx.tasklist:
              custom_checkbox: true
          - pymdownx.tilde

        extra:
          status:
            new: Recently added
            deprecated: Deprecated
          consent:
            actions:
              - accept
              - reject
              #- manage
            title: Cookie consent
            description: >-
              We use cookies to recognize your repeated visits and preferences, as well
              as to measure the effectiveness of our documentation and whether users
              find what they're searching for. With your consent, you're helping us to
              make our documentation better.
        ```

    === "directory tree"
        ```bash
        # Erstellt Ordnerstruktur
        mkdir -p /opt/mkdocsmaterial/data/{mkdocs,nginx}
        mkdir -p /opt/mkdocsmaterial/data/mkdocs/docs
        mkdir -p /opt/mkdocsmaterial/data/nginx/templates
        cd /opt/mkdocsmaterial/

        # Zeigt Ordnerstruktur an
        tree -d -L 3 /opt/mkdocsmaterial/

        # Zeigt belegte Docker-Netzwerke an
        egrep -r 'SUBNET|IPV4_NETWORK' /opt/*
        ```
    === "index.md"

        ```bash
        cd /opt/mkdocsmaterial
        cat > data/mkdocs/docs/index.md << 'EOL'
        ---
        title: Home
        Summary: A brief description .
        Authors: Johann Hahn
        Date:    Juli 08, 2024
        ---

        # Welcome to the Documentation of `Wassermann Gruppe`!
        EOL
        ```

    === "about.md"

        ```bash
        cd /opt/mkdocsmaterial
        cat > data/mkdocs/docs/index.md << 'EOL'
        ---
        title: Home
        Summary: A brief description .
        Authors: Johann Hahn
        Date:    Juli 08, 2024
        ---

        # Über Firma `Wassermann Gruppe`!
        EOL
        ```

#### Der esrste Start von `MkDocs` Docker Container

HTML-Dateien werden im Verzeichnis `/opt/mkdocsmaterial/data/mkdocs/site` erstellt.

```bash
cd /opt/mkdocsmaterial

# Erstellt neues Projekt
docker run --rm -it -v /opt/mkdocsmaterial/data/mkdocs:/docs squidfunk/mkdocs-material new .

# Erstellt Doku's
docker run --rm -it -v /opt/mkdocsmaterial/data/mkdocs:/docs squidfunk/mkdocs-material build
```

#### Der esrste Start von `NGINX` Docker Container

=== "docker compose (Plugin)"
    ```bash
    cd /opt/mkdocsmaterial
    docker compose up -d

    # Zeigt Status an
    docker compose ps

    # Zeigt Logdaten an
    docker compose logs -f
    ```

=== "docker-compose (Standalone)"

    ```bash
    cd /opt/mkdocsmaterial
    docker-compose up -d

    # Zeigt Status an
    docker-compose ps

    # Zeigt Logdaten an
    docker-compose logs -f
    ```

#### Seite aufrufen

Wenn keine Fehlermeldungen erschienen sind kann man die Seite unter `https://docs.myfirma.de` aufrufen.
