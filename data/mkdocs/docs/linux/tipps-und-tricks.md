---
Title: Linux Tipps und Tricks
Summary: Linux Tipps und Tricks
Authors: Johann Hahn
Date:    Juli 24, 2024
tags: [linux, tipps, tricks]
---

# :fontawesome-brands-linux: Linux Tipps und Tricks

#### Rocky | Oracle - change NIC name

??? tip "Rocky | Oracle - change NIC name"

    ```bash
    # show all network interfaces
    ip a sh
    ---
    ...
    2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    ...
    ---

    # show details of interface ens18
    nmcli device show ens18
    ---
    ...
    GENERAL.DEVICE:                         ens18
    GENERAL.TYPE:                           ethernet
    GENERAL.HWADDR:                         5A:43:10:3E:F4:D6
    ...
    ---

    # create folder and file 70-custom.link
    mkdir /etc/systemd/network
    vim /etc/systemd/network/70-custom.link
    ---
    [Match]
    MACAddress=5A:43:10:3E:F4:D6

    [Link]
    Name=eth0
    ---

    #
    init 6
    ```

#### DNS-Abfragen mit `nslookup`

??? tip "DNS-Abfragen mit `nslookup`"
    
    `nslookup` is a command-line tool for inspecting the Domain Name System (DNS).

    ```bash
    # NS Record
    nslookup -type=ns chip.de

    # MX Record
    nslookup -type=mx chip.de

    # A Record
    nslookup -type=a chip.de

    # AAAA Record
    nslookup -type=aaaa chip.de

    # CNAME Record
    nslookup -type=cname chip.de

    # PTR Record
    nslookup -debug 142.250.181.227
    nslookup -debug 2a02:26f0:7100::213:c6f0
    ```

#### DNS-Abfragen mit `dig`

??? tip "DNS-Abfragen mit `dig`"

    `dig` ( Domain Information Groper) is a command-line tool for inspecting the Domain Name System (DNS).

    ```bash
    # SOA Record
    dig rohrkabel.eu SOA

    dig rohrkabel.eu +nssearch

    # NS Record
    dig rohrkabel.eu NS
    dig +short rohrkabel.eu NS

    # MX Record
    dig rohrkabel.eu MX
    dig +short rohrkabel.eu MX

    # A Record IPv4
    dig rohrkabel.eu A
    dig +short rohrkabel.eu A

    # AAAA Record IPv6
    dig rohrkabel.eu AAAA
    dig +short rohrkabel.eu AAAA

    # ANY - to query any records
    dig rohrkabel.eu ANY

    # SRV Record
    dig _autodiscover._tcp.rohrkabel.eu SRV
    dig _imaps._tcp.rohrkabel.eu SRV
    dig _smtps._tcp.rohrkabel.eu SRV
    dig _submission._tcp.rohrkabel.eu SRV

    # TXT Record
    dig rohrkabel.eu TXT
    dig dkim._domainkey.rohrkabel.eu TXT
    dig _dmarc.rohrkabel.eu TXT

    # CNAME Record
    dig smtp.rohrkabel.eu CNAME
    dig imap.rohrkabel.eu CNAME
    dig ldap.rohrkabel.eu CNAME

    # TTL Record
    dig rohrkabel.eu TTL

    # PTR Record IPv4
    dig -x 109.123.246.64
    dig +short -x 109.123.246.64

    # PTR Record IPv6
    dig -x 2a02:c206:3010:2208::1
    dig +short -x 2a02:c206:3010:2208::1

    # find the delegation path to DNS zone
    dig +trace rohrkabel.eu
    dig +trace mail.rohrkabel.eu

    # 
    dig +nocmd +noall +answer +ttlid A mail.rohrkabel.eu
    dig +nocmd +noall +answer +ttlid AAAA mail.rohrkabel.eu

    #
    dig +multiline +noall +answer +nocmd rohrkabel.eu ANY

    # using a specific DNS server
    dig @79.143.182.242 +short pla.rohrkabel.eu CNAME
    dig @79.143.182.242 +short ldap.rohrkabel.eu CNAME
    dig @8.8.8.8 +short rohrkabel.eu NS
    dig @79.143.182.242 +short rohrkabel.eu A
    docker compose exec postfix-mailcow dig mx gmail.com @unbound
    ```

#### Tipp

??? tip "Tipp"

    ```bash

    ```

