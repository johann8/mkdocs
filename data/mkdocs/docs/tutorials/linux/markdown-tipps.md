---
Title: Markdown Beispiele
Summary: Markdown Formatierung Beispiele
Authors: Johann Hahn
Date:    August 29, 2024
tags: [markdown, beispiele]
---

### Einige Beispiele

Tabbs

=== "Chinese (Simplified)"

    ``` yaml
    plugins:
      - social:
          cards_layout_options:
            font_family: Noto Sans SC
    ```

=== "Chinese (Traditional)"

    ``` yaml
    plugins:
      - social:
          cards_layout_options:
            font_family: Noto Sans TC
    ```

=== "Japanese"

    ``` yaml
    plugins:
      - social:
          cards_layout_options:
            font_family: Noto Sans JP
    ```

=== "Korean"

    ``` yaml
    plugins:
      - social:
          cards_layout_options:
            font_family: Noto Sans KR
    ```
---

Layouts are written in YAML syntax. Before starting to create a custom layout,
it is a good idea to [study the pre-designed layouts] (link to [Insiders]
repository), in order to get a better understanding of how they work. Then,
create a new layout and reference it in `mkdocs.yml`:

=== ":octicons-file-code-16: `custom.yml`"

    ``` yaml
    size: { width: 1200, height: 630 }
    layers: []
    ```

=== ":octicons-file-code-16: `mkdocs.yml`"

    ``` yaml
    plugins:
      - social:
          cards_layout_dir: layouts
          cards_layout: custom
          debug: true
    ```
---
### tip

!!! tip "Automatically bundle Google Fonts"

    The [built-in privacy plugin] makes it easy to use Google Fonts
    while complying with the __General Data Protection Regulation__ (GDPR),
    by automatically downloading and self-hosting the web font files.

  [data privacy]: https://developers.google.com/fonts/faq/privacy
  [built-in privacy plugin]: ./plugins/privacy.md
---

[Docker] improves the user experience when switching between languages, e.g.,
if language `en` and `de` contain a page with the same path name, the user will
stay on the current page:

No configuration is necessary. We're working hard on improving multi-language
support in 2024, including making switching between languages even more seamless
in the future.

  [Docker]: ./tutorials/docker/index.md

---
<!-- This is a comment -->


This property must contain an [ISO 639-1 language code] and is used for the `hreflang` attribute of the link, improving discoverability via search engines.

[![Language selector preview]][Language selector preview]

  [site_url]: https://www.mkdocs.org/user-guide/configuration/#site_url
  [ISO 639-1 language code]: https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes
  [Language selector preview]: ./assets/screenshots/language-selection.png
---

### info

!!! info

    Values in **bold** are the default values. If an option doesn't work as documented here, check if you are running the latest image. The current `master` branch corresponds to the image `ghcr.io/docker-mailserver/docker-mailserver:edge`.

---

### quote

!!! quote "What is Fail2Ban (F2B)?"

    Fail2ban is an intrusion prevention software framework. Written in the Python programming language, it is designed to prevent against brute-force attacks. It is able to run on POSIX systems that have an interface to a packet-control system or firewall installed locally, such as \[NFTables\] or TCP Wrapper.

    [Source][wikipedia-fail2ban]

    [wikipedia-fail2ban]: https://en.wikipedia.org/wiki/Fail2ban

### warning

!!! warning

    DMS must be launched with the `NET_ADMIN` capability in order to be able to install the NFTables rules that actually ban IP addresses. Thus, either include `--cap-add=NET_ADMIN` in the `docker run` command, or the equivalent in the `compose.yaml`:

    ```yaml
    cap_add:
      - NET_ADMIN
    ```

### bug

!!! bug "Running Fail2Ban on Older Kernels"

    DMS configures F2B to use NFTables, not IPTables (legacy). We have observed that older systems, for example NAS systems, do not support the modern NFTables rules. You will need to configure F2B to use legacy IPTables again, for example with the [``fail2ban-jail.cf``][github-file-f2bjail], see the [section on configuration further down below](#custom-files).


### question
!!! question "What is [`docker-data/dms/config/`][docs::dms-volumes-config]?"

---

### note

!!! note "About The Mail Server's Fully Qualified Domain Name"

    The mail server's fully qualified domain name (FQDN) in our example above is `mail.example.com`. Please note though that this is more of a convention, and not due to technical restrictions. One could also run the mail server

    1. on `foo.example.com`: you would just need to change your `MX` record;
    2. on `example.com` directly: you would need to change your `MX` record and probably [read our docs on bare domain setups][docs-faq-bare-domain], as these setups are called "bare domain" setups.

    The FQDN is what is relevant for TLS certificates, it has no (inherent/technical) relation to the email addresses and accounts DMS manages. That is to say: even though DMS runs on `mail.example.com`, or `foo.example.com`, or `example.com`, there is nothing that prevents it from managing mail for `barbaz.org` - `barbaz.org` will just need to set its `MX` record to `mail.example.com` (or `foo.example.com` or `example.com`).

    [docs-faq-bare-domain]: ./faq.md#can-i-use-a-nakedbare-domain-ie-no-hostname

---

Mailserver - dockerized 🐋

---
### Font awesome

- :fontawesome-brands-github: [mailcow/mailcow-dockerized @ GitHub](https://github.com/mailcow/mailcow-dockerized)

- :fontawesome-solid-globe: [mailcow.email](https://mailcow.email)

### question

!!! question
    Mailcow, MailCow or mailcow? What is the exact name of the project?

    Correct: **mailcow**, because mailcow is a registered word mark with a small m :grin:

mailcow: dockerized is an open source groupware/email suite based on docker.

### abstract

!!! abstract "Use these credentials for the demos"
    - **Administrator**: admin / moohoo
    - **Domain-Administrator**: department / moohoo
    - **Mailbox**: demo@440044.xyz / moohoo

    *The login credentials work for both variants*.

### success

!!! success "Always up to date"
        The demo instances get the latest updates directly after releases from GitHub. Fully automatic, without any downtime!


[^1]: Servercow is a hosting/support division of The Infrastructure Company GmbH (mailcow maintainer).

Each container represents a single application, connected in a bridged network:

<div class="grid cards" markdown>

- :simple-letsencrypt: [__ACME__](https://letsencrypt.org/) Automatic generation of Let's Encrypt certificates
- :fontawesome-solid-file-shield: [__ClamAV__](https://www.clamav.net/) Antivirus scanner (optional)
- :simple-dovecot: [__Dovecot__](https://www.dovecot.org/) IMAP/POP server for retrieving e-mails with integrated Full-Text Search Engine "Flatcurve"
- :simple-mariadb: [__MariaDB__](https://mariadb.org/) Database for storing user information etc.
- :fontawesome-solid-memory: [__Memcached__](https://www.memcached.org/) Cache for the webmailer SOGo
- :fontawesome-solid-ban: __Netfilter__ Fail2ban-like integration of [@mkuron](https://github.com/mkuron)
- :simple-nginx: [__Nginx__](https://nginx.org/) Web server for components of the stack
- :material-microsoft-office: [__Olefy__](https://github.com/HeinleinSupport/olefy) Analysis of Office documents for viruses, macros, etc.
- :simple-php: [__PHP__](https://php.net/) Programming language of most web-based mailcow applications
- :material-email-newsletter: [__Postfix__](http://www.postfix.org/) MTA (Mail Transfer Agent) for e-mail traffic on the Internet
- :simple-redis: [__Redis__](https://redis.io/) Storage for spam information, DKIM key, etc.
- :fontawesome-solid-trash-can: [__Rspamd__](https://www.rspamd.com/) Spam filter with automatic learning of spam mails
- :material-calendar: [__SOGo__](https://sogo.nu/) Integrated webmailer and Cal-/Carddav interface
- :simple-apachesolr: [__Solr__](https://solr.apache.org/) Full text search for IMAP connections to quickly search emails (Deprecated) (Optional)
- :material-dns: [__Unbound__](https://unbound.net/) Integrated DNS server for verifying DNSSEC etc.
- :material-watch: __Watchdog__ For basic monitoring of the container status within mailcow
</div>
---
## Aufklappbar
??? question "How to add plugins to the Docker image?"

    Material for MkDocs only bundles selected plugins in order to keep the size
    of the official image small. If the plugin you want to use is not included,
    you can add them easily:

    === "Material for MkDocs"

        Create a `Dockerfile` and extend the official image:

        ``` Dockerfile title="Dockerfile"
        FROM squidfunk/mkdocs-material
        RUN pip install mkdocs-macros-plugin
        RUN pip install mkdocs-glightbox
        ```

    === "Insiders"

        Clone or fork the Insiders repository, and create a file called
        `user-requirements.txt` in the root of the repository. Then, add the
        plugins that should be installed to the file, e.g.:

        ``` txt title="user-requirements.txt"
        mkdocs-macros-plugin
        mkdocs-glightbox
        ```

    Next, build the image with the following command:

    ```
    docker build -t squidfunk/mkdocs-material .
    ```

    The new image will have additional packages installed and can be used
    exactly like the official image.

---
## Example (Authentik & Roundcube)

This example assumes you have:

- A working DMS server set up
- An Authentik server set up ([documentation](https://goauthentik.io/docs/installation/))
- A Roundcube server set up (either [docker](https://hub.docker.com/r/roundcube/roundcubemail/) or [bare metal](https://github.com/roundcube/roundcubemail/wiki/Installation))

!!! example "Setup Instructions"

    === "1. Docker Mailserver"
        Edit the following values in `mailserver.env`:
        ```env
        # -----------------------------------------------
        # --- OAUTH2 Section ----------------------------
        # -----------------------------------------------

        # empty => OAUTH2 authentication is disabled
        # 1 => OAUTH2 authentication is enabled
        ENABLE_OAUTH2=1

        # Specify the user info endpoint URL of the oauth2 provider
        OAUTH2_INTROSPECTION_URL=https://authentik.example.com/application/o/userinfo/
        ```

    === "2. Authentik"
        1. Create a new OAuth2 provider
        2. Note the client id and client secret
        3. Set the allowed redirect url to the equivalent of `https://roundcube.example.com/index.php/login/oauth` for your RoundCube instance.
---

<!-- This is a comment -->

<div align="center">
        <code>Hier ist ein Test<img width="30" src="https://user-images.githubusercontent.com/25181517/117207330-263ba280-adf4-11eb-9b97-0ac5b40bc3be.png" alt="Docker" title="Docker"/></code>
</div>
![](https://user-images.githubusercontent.com/25181517/117207330-263ba280-adf4-11eb-9b97-0ac5b40bc3be.png "Docker"){width=40}

:fontawesome-brands-docker: Docker container

---

!!! Warning ""

    **Your mileage may vary depending on your hardware.**

---

## Developers

{{ external_markdown('https://raw.githubusercontent.com/jordanhillis/pvekclean/master/README.md', '## Developers') }}

The original [pvekclean github page][pvekclean-url]{target=\_blank}

## Updating

{{ external_markdown('https://raw.githubusercontent.com/jordanhillis/pvekclean/master/README.md', '## Updating') }}

## Usage

{{ external_markdown('https://raw.githubusercontent.com/jordanhillis/pvekclean/master/README.md', '## Usage') }}

<!-- appendices -->

[pvekclean-url]: https://github.com/jordanhillis/pvekclean

<!-- end appendices -->

---

:bulb: **Tip:** Remember to appreciate the little things in life.

!!! tip ":bulb: **Tip:** Automatically bundle Google Fonts"

!!! tip ":bulb: **Note:** Automatically bundle Google Fonts"

!!! note ":bulb: **Note:** Automatically bundle Google Fonts"

!!! note ""
    :bulb: **Note:** Automatically bundle Google Fonts

!!! warning ""
    :bulb: **Note:** Automatically bundle Google Fonts


!!! info ""
    :material-lightbulb-on: **Note:** Automatically bundle Google Fonts


!!! info ""
    :material-lightbulb: **Note:** Automatically bundle Google Fonts

---
#### Test1

> :warning: **Warning:** Do not push the big red button.

#### Test2

> :memo: **Note:** Sunrises are beautiful.

#### Test3

> :bulb: **Tip:** Remember to appreciate the little things in life.

---

<p style="color:red; text-align:center">Make this text blue.</p>

---

##### How to add multiple spaces between texts in Markdown

I have a long space before &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;me.

And finally one more thing, there are other codes like &ensp;(2 x &nbsp;) and &emsp;(4 x &nbsp;).

I have a long space before &emsp;me.

---
Enoj
