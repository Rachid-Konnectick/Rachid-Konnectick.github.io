---
layout: post
title: "Install Bind 9"
subtitle: "Install and configure Bind 9 on Debian 12."
date: 2025-02-14 19:25:13 -0400
background: '/img/posts/n-tier.jpg'
---

## Mise a jour du cache et installation:

```sh
sudo apt update && apt upgrade -y
sudo apt install bind9 bind9utils bind9-doc
named -v (check the version)

## Mettre une interface up ou down debian
ip link set dev eth0 up
ip link set dev eth0 down
```

# Change default to IPv4:
```sh
sudo nano /etc/default/named
## add "-4" in the line
OPTIONS= "-u bind -4"

sudo nano /etc/resolv.conf
search co
domain co
nameserver 192.168.1.9
```

## Configuration:
```sh
DNS test in ububntu
dig google.com (test dns)
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
sudo systemctl status systemd-resolved.service (on ubunutu)
sudo systemctl disable systemd-resolved.service
sudo systemctl stop systemd-resolved.service
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
```
1.Edit named.conf.options and add Forwarders:
--------------------------------------------------------------------------
```sh
sudo nano /etc/bind/named.conf.options
```

```

acl trusted {
        192.168.1.0/24;

    };
options {
        directory "/var/cache/bind";
        allow-recursion { trusted; };  # allows recursive queries from "trusted" clients
        allow-query     {192.168.1.0/24; localhost; };
        listen-on port 53 { localhost; 192.168.1.9; };
        allow-transfer { localhost; };
        forwarders { 8.8.8.8; 1.1.1.1; };
        dnssec-validation auto;
        listen-on-v6 { any; };
};
```
2.Create a local forward file:
-------------------------------
```sh

sudo cp db.local db.beesafe.fwd
sudo nano db.beesafe.fwd


; BIND data file for local loopback interface
;
$ORIGIN beesafe.co.
$TTL    604800
@       IN      SOA     bind-dns.beesafe.co. root.bind-dns.beesafe.co. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;DNS Server Record
@       IN      NS      bind-dns.beesafe.co.
ns      IN      A       192.168.1.9

;Application DNS Records
                        IN      NS      ns.beesafe.co
bind.beesafe.co         IN      A       192.168.1.9
ns.beesafe.co.          IN      A       192.168.1.9
Mysql                   IN      A       192.168.1.10
www.beesafe.co.         IN      A       192.168.1.12
```
3.Create reverse zone file:
--------------------------------------------
```sh
sudo cp db.127 db.192.168

sudo nano db.192.168



$TTL    604800
@       IN      SOA     ns.beesafe.co. root.ns.beesafe.co. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;DNS Server Records
@       IN      NS      ns.beesafe.co.
ns      IN      A       192.168.1.9
9 	    IN      PTR     ns.beesafe.co.

; PTR Records
10  IN      	PTR     Mysql					 ; 192.168.1.10
12  IN      	PTR     www.beesafe.co.			 ; 192.168.1.12

```

4.Edit named.conf.local config file:
------------------------------------------------------
```sh
sudo nano /etc/bind/named.conf.local

zone "beesafe.co" {
    type master;
    file "/etc/bind/db.beesafe.fwd";
   
};


### Reverse -zone:

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168";  
```


5.Add firewall rules:
------------------------------------
```sh
sudo apt install ufw
sudo ufw version
sudo ufw enable
sudo grep -i '^default_' /etc/default/ufw
sudo ufw default default allow|deny|reject [incoming|outgoing|routed]
ex:
sudo ufw default deny incoming   
sudo ufw default allow outgoing
sudo ufw status verbose   
sudo ufw status numbered

sudo ufw allow ssh
sudo ufw allow 53
output:
Rule added
Rule added (v6)

```
6.Test the file configuration syntax :
----------------------------------------------------------------

```sh

sudo named-checkzone beesafe.co db.beesafe.fwd
sudo named-checkzone beesafe.co db.192.168


test functionality of Bind DNS in client:
dig -t ns bind-dns.beesafe.co
dig -x 192.168.1.9

nslookup 192.168.1.9
nslookup www.beesafe.co
edition du fichier vars.php :

ipconfig /flushdns
ipconfig /registerdns (after changing dns server ip)

FWD ZONE
;
; BIND data file for local loopback interface
;
$ORIGIN beesafe.co.
$TTL    604800
@       IN      SOA     bind-dns.beesafe.co. root.bind-dns.beesafe.co. (
                             16         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;DNS Server Record
@                       IN      NS      bind-dns.beesafe.co.
bind-dns                IN      A       192.168.1.9
;Application DNS Records
Mysql                   IN      A       192.168.1.10
www.beesafe.co.         IN      A       192.168.1.12
Windows                 IN      A       192.168.1.62




Reverse zone:

; BIND reverse data file for local loopback interface

$TTL    604800
@       IN      SOA    bind-dns.beesafe.co. root.bind-dns.beesafe.co. (
                              14         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;DNS Server Records
@               IN     NS       bind-dns.beesafe.co.
bind-dns        IN      A       192.168.1.9
9               IN      PTR     bind-dns.beesafe.co.

; PTR Records
10  IN          PTR     Mysql            ; 192.168.1.10
12  IN          PTR     www.beesafe.co   ; 192.168.1.12
62  IN          PTR     Windows          ; 192.168.1.62

```
```sh
check conf errors:
sudo named-checkconf

sudo systemctl restart named
test if the dns is runing:

dig -t ns konnectik.fr 
test reverse lookup query:
dig -x 192.168.1.9
```
## In the client side windows:
force dns server, open cmd :
```cmd
netsh interface ipv4 set dns name="WI-FI" static 192.168.1.9 primary
netsh interface ipv4 set dns name="WI-FI" static 1.1.1.2 secondary
Replace "Ethernet" with the name of your network interface (e.g., "Wi-Fi" for wireless connections).
```
7.Flush DNS cache In ubuntu dns:
-------------------------------------------------
```sh
sudo resolvectl flush-caches
sudo service network-manager restart
```

Schema Database:
représente n'importe quel type de structure qu'on defint par rapport  données
ceci inclut:
tableaux
types
Indexes
Vues
Champs
Fonctions
Procedures
Sequences
queues
triggers
synonyms
directories 
xml schemas

Schema on read:
application de la structure seulment en cas de requete
Schema On write:
structure imposée avant l'ecriture des données dans la Db

## Tuto Bind 9 christian lempa with Docker:

Create a real public domain ex:
```
konnectik.fr
and subdomains like:
srv-dns.lan.konnectik.fr
srv2.lan.konnectik.fr
with wildcard for kubernetees:
*.kub1.lan.konnectik.fr
Have split-horizon DNS
internal home DNS resolve internal ip addresses
externam DNS resolmve public IP for the same server e:
srv1.lan.konnectik.fr > 192.168.1.100 in home
srv1.lan.konnectik.fr > 147.25.1.100 in external

```

## Compose yaml file to install bind9:
```yml
services:
  bind9:
    container_name: dns-konnectik
    image: ubuntu/bind9:latest
    environnment:
      - Bind9_USER=root
      - TZ-Europe/Paris
    ports:
      - "53:53/tcp"
      - "53:53/udp"
    vomumes:
      - ./config:/etc/bind
      - ./cache:/var/cache/bind
      - ./records:/var/lib/bind
    restart: unless-stoopped
```

### named.conf config file:
```
acl trusted {
        192.168.1.0/24;

    };
options {
        directory "/var/cache/bind";
        allow-recursion { trusted; };  # allows recursive queries from "trusted" clients
        allow-query     {192.168.1.0/24; localhost; };
        listen-on port 53 { localhost; 192.168.1.9; };
        allow-transfer { localhost; };
        forwarders { 9.9.9.9; 1.1.1.2; };
        dnssec-validation auto;
        listen-on-v6 { any; };
};
```
christian lempa:
```
acl internal {
    192.168.1.0/24;

};
options {
    forwarders { 
        9.9.9.9;
        1.1.1.2; 
    };
    allow-query { internal; };
};
```
before installing we must disable the machine dns that is already listening on port 53
```sh
sudo nano /systemd/resolved.conf
uncomment and set :
DNSStublistener=no
than
sudo systemctl restart systemd-resolved
```
` docker-compose up
we can test our bind9 dns with the command:
` nslookup youtube.com bind9-dns-ip

### Define zones in named.conf file:
```
acl internal {
    192.168.1.0/24;

};
options {
    forwarders { 
        9.9.9.9;
        1.1.1.2; 
    };
    allow-query { internal; };
};

zone "lan.konnectik.fr" IN {
        type master;
        file "/etc/bind/fwd-konnectik.zone"
};

```

In /etc/bind/ make file for forward zone:
TTL: time to keep dns zone in cache
$ORIGIN : name of the dns zone to configure, help when adding new records we don't need to add the entire FQDN,just the hostname
SOA: administrative infos about the zone
```
$TTL 2d

$ORIGIN konnectik.fr
@              IN      SOA     ns.konnectik.fr.  info.konnectik.fr. (
                               2024111100   ; serial
                               12h          ; refresh
                               15m          ; retry
                               3w           ; expire
                               2h )         ; minimum ttl
                               

@               IN      NS     ns.konnectik.fr. 
ns             IN      A      192.168.1.9
; add dns records below
pmox           IN      A     192.168.1.4
minisf         IN      A     192.168.1.62

```

` docker-compose restart
Reverse zone:
```
$TTL 2d

@              IN      SOA     ns.konnectik.fr.  info.konnectik.fr. (
                               2024111201   ; serial
                               12h          ; refresh
                               15m          ; retry
                               3w           ; expire
                               2h )         ; minimum ttl
                               

;DNS Server Records
@       IN      NS      ns.konnectik.fr.
ns      IN      A       192.168.1.9
9 	    IN      PTR     ns.konnectik.fr.

; PTR Records
4  IN      	PTR     pmox		 ; 192.168.1.4
62  IN      PTR     beelink-rt	     ; 192.168.1.62
```

```
sudo named-checkzone konnectik.fr fwd.konnectik.fr
sudo named-checkzone konnectik.fr rev.konnectik.fr
```