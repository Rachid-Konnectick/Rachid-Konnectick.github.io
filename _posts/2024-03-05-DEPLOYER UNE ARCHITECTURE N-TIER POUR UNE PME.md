---
layout: post
title: "DEPLOYER UNE ARCHITECTURE N-TIER POUR UNE PME"
subtitle: ""
date: 2024-03-05 
background: '/img/posts/n-tier.jpg'
---


# Sommaire
1. Installation de BIND 9 sur Debian 12
2. Installation et Configuration de la Base de données Mariadb
3. Installation d'Apache2 et PHP sur Debian 12
4. Configuration du Virtualhost sur Apache2 et du fichier vars.php
5. Configuration des zones DNS dans BIND 9

## Schema de l'architecture n-tier:
### Adresses IP:
- Webserver : 192.168.1.12
- Mariadb : 192.168.1.10
- Bind9  192.168.1.9

![](Schema%20des%20flux.drawio.png)

## 1. Installation de BIND 9 sur Debian 12:
### Mise a jour du cache et installation:

```sh
sudo apt update && sudo apt upgrade -y
sudo apt install bind9 bind9utils bind9-doc
# Check the version
named -v 
```

## 2.Installation et Configuration de la Base de données Mariadb sur Debian 12:

```sh
sudo apt update && sudo apt upgrade -y
sudo apt install mariadb-server -y
sudo systemctl start mariadb && sudo systemctl enable mariadb
sudo systemctl status mariadb
```
### Creation de la base de données et de l'utilisateur:
```sh
mysql -u root -p
create database beesafe;
create user 'user1'@'192.168.1.12' identified by 'yoursecurepassword';
grant all privileges on beesafe.* to 'user1'@'192.168.1.12' with grant option;
flush privileges;
```
### Sécuriser la base de données:
```sh
sudo mysql_secure_installation
# Enter current password for root (il s'agit du mdp root de la base de données,n'est pas encore configuré donc faire entrer)
# unix_socket authentification [no]
# change the root password
Créer un mot de passe
# suppression des users anonymes [yes]
# Disallow root login remotly [yes]
# suppresion de la base de données test [yes]
# Reload priviliges tables now?[yes]
```

## 3. Installation d'Apache2 sur Debian 12:
```sh
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 php libapache2-mod-php -y
```
- Installer les modules php:
```sh
sudo apt install php-xml php-common php-json php-mysql php-mbstring php-curl php-gd php-intl php-zip php-bz2 php-imap php-apcu -y 
```

### Activer apache2 :
```sh
sudo systemctl start apache2
sudo systemctl enable apache2
sudo systemctl status apache2
```

## 4.Configuration du Virtualhost sur Apache2 et du fichier vars.php:
```sh
<VirtualHost *:80>
  	Servername      beesafe.co
        ServerAlias     www.beesafe.co
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/beesafe/public_html

        
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        
    <Virtualhost>
```
- Fichier vars.php, pour l'accès à la base de données:
```sh
<?php
$servername = "192.168.1.10";
$username = "user1";
$password = "yoursecurepassword";
$db = "beesafe";
$port = 3306;
?>
```

## 5. Configuration des zones DNS dans BIND 9:
### 1.Edit /etc/resolv.conf:
```sh
sudo nano /etc/resolv.conf

search co
domain co
nameserver 192.168.1.9
```
#### Changer vers l'IPv4 par défaut:
Si on veut utiliser que l'IPv4:
```sh
sudo nano /etc/default/named
## add "-4" in the line
OPTIONS= "-u bind -4"
```

### 2.Editer le fichier named.conf.options:
```sh
sudo nano /etc/bind/named.conf.options


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

### 3.Editer le fichier names.conf.local pour les zones:
```sh

zone "beesafe.co" IN {
    type master;
    file "/etc/bind/db.beesafe.fwd";
        allow-update {none; };
};

zone "1.168.192.in-addr.arpa" IN {
    type master;
    file "/etc/bind/db.192.168";
        allow-update {none; };
};
```

### 4.Créer et éditer le fichier pour la zone forward:
```sh
sudo nano db.beesafe.fwd

 BIND data file for local loopback interface
;
$ORIGIN beesafe.co.
$TTL    604800
@       IN      SOA     bind-dns.beesafe.co. root.bind-dns.beesafe.co. (
                             11         ; Serial
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
Windows                 IN      A       192.168.1.100
```

### 5.Créer et éditer le fichier pour la reverse zone:

```sh
sudo nano db.192.168

; BIND reverse data file for local loopback interface

$TTL    604800
@       IN      SOA    bind-dns.beesafe.co. root.bind-dns.beesafe.co. (
                              12         ; Serial
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
12  IN          PTR     www.beesafe.co.  ; 192.168.1.12
100 IN          PTR     Windows          ; 192.168.1.100
```
#### Chercher les erreur dans les 2 fichiers précédents:
```sh
sudo named-checkzone beesafe.co db.beesafe.fwd
sudo named-checkzone beesafe.co db.192.168
```

### 6.Ajouter les règles parefeu:
```sh
sudo ufw allow 53
```
### 7.Tester la fonctionnalité du DNS par nslookup ou dig:
```sh

dig -t ns bind-dns.beesafe.co
dig -x 192.168.1.9

nslookup 192.168.1.9 (inverse)
nslookup www.beesafe.co 
```
