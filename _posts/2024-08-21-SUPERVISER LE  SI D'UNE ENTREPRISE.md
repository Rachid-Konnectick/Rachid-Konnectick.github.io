---
layout: post
title: "SUPERVISER LE SI D'UNE ENTREPRISE"
subtitle: "Monitoring avec Nagios et Rsyslog"
date: 2024-08-21
background: '/img/posts/Supervision/nagios.jpg'
---
# Sommaire
1. [Objectif](#1objectif-)
2. [Installer Nagios](#Installer-Nagios)
3. [Installer Wordpress sur Debian 12](#3installer-wordpress-sur-debian-12)
4. [Installer NRPE Plugin](#4installer-nrpe-plugin)
5. [Fichiers de configuration Nagios et Wordpress](#5fichiers-de-configuration-nagios-et-wordpress)
6. [Interface Web Nagios](#6interface-nagios)
7. [Documentation conduite à tenir lors des alertes](#7documentation-conduite-à-tenir-lors-des-alertes)
8. [Installation et configuration de Rsyslog](#8installation-et-configuration-de-rsyslog)

# 1.Objectif :
- Installation d'un serveur Nagios
- Installation d'un serveur Wordpress pour  héberger un site web
- Installation et configuration d'un serveur Rsyslog
- Mise en place d'un monitoring du serveur Nagios et du serveur Wordpress
- Sonde Nagios a mettre en place:
    - **RAM** : *Warning* à 70% d'utilisation et *Critical* à 80%
    - **CPU** : 
        - *Warning* 
            - 70%: la dernière minute
            - 60%: 5 dernières min
            - 50%: 15 dernières min
        - *Critical*
            - 90%: la dernière minute
            - 80%: 5 dernières min
            - 70%: 15 dernières min
    - **Espace disque**: *Warning* 70% d'utilisation, *Critical* à 80%
    - **Session utilisateurs**: *Warning* si > 1 utilisateur sur le serveur Nagios
    - **Le processus du serveur Web tourne**: (HTTP/HTTPS/URL)
    - **La base de données** tourne
    - Rédiger une documentation pour une conduite à tenir face aux alertes de chaque sonde

# 2.<a name="Installer-Nagios"> </a> Installer Nagios:

```sh
# Mise à jour du cache:
apt-get update 
# Installer les prérequis:
apt-get install -y autoconf gcc libc6 make wget unzip apache2 apache2-utils php libgd-dev  
apt-get install openssl libssl-dev
# Telecharger la source:
cd /tmp  
wget -O nagioscore.tar.gz https://github.com/NagiosEnterprises/nagioscore/archive/nagios-4.4.14.tar.gz tar xzf nagioscore.tar.gz
# Compiler :
cd /tmp/nagioscore-nagios-4.4.14/  
./configure --with-httpd-conf=/etc/apache2/sites-enabled  
make all
# Creer les groupes et le user :
make install-groups-users  
usermod -a -G nagios www-data
# Installer commandmode sample config
make install
make install-daemoninit
make install-commandmode
make install-config
# Installer apache config
make install-webconf  
a2enmod rewrite  
a2enmod cgi
# configurer le parefeu
iptables -I INPUT -p tcp --destination-port 80 -j ACCEPT  
apt-get install -y iptables-persistent
# Configurer le compte nagiosadmin:
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
systemctl restart apache2.service
systemctl start nagios.service
```
## Installer les plugins Nagios:

```sh
# Installer les prérequis:
apt-get install -y autoconf gcc libc6 libmcrypt-dev make libssl-dev wget bc gawk dc build-essential snmp libnet-snmp-perl gettext
# Telecharger la source:
cd /tmp  
wget --no-check-certificate -O nagios-plugins.tar.gz https://github.com/nagios-plugins/nagios-plugins/archive/release-2.4.6.tar.gz 
wget https://nagios-plugins.org/download/nagios-plugins-2.4.6.tar.gz
tar zxf nagios-plugins.tar.gz
#Compile and install
cd /tmp/nagios-plugins-release-2.4.6/  
./tools/setup  
./configure  
make  
make install
```
## Commandes de Service Nagios :

```sh
systemctl start nagios.service  
systemctl stop nagios.service  
systemctl restart nagios.service  
systemctl status nagios.service

```
-Tester la config Nagios:

```sh
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```
-Donner au compte `nagios` le droit de relancer Nagios:

```sh
echo "nagios ALL=NOPASSWD:/bin/systemctl restart nagios" >> /etc/sudoers
```
# 3.Installer Wordpress sur Debian 12:
## Preparation accès SSH:
### Installer SUDO et Open-SSH sur Debian 12:

```sh
apt update && apt upgrade -y
apt install sudo openssh-server -y
```
### Ajouter un user au groupe sudo :
```sh
usermod -aG sudo 'username'
apt install openssh-server
```
### Connexion au serveur debian via ssh :
Ouvrir cmd:
`ssh 'username'@192.168.x.x`
### Install Apache2 :

```sh
sudo apt-get update -y && sudo apt-get upgrade -y
sudo apt install apache2 ghostscript libapache2-mod-php -y
sudo systemctl enable apache2 && sudo systemctl start apache2
sudo systemctl status apache2
```
### Install PHP 8.2 :
```sh
sudo apt-get install php8.2 php8.2-cli php8.2-common php8.2-imap php8.2-redis php8.2-snmp php8.2-xml php8.2-mysqli php8.2-zip php8.2-mbstring php8.2-curl libapache2-mod-php -y
php -v
```
### Install MariaDB :

```sh
sudo apt install mariadb-server -y
sudo systemctl start mariadb && sudo systemctl enable mariadb
sudo systemctl status mariadb
```
### Create Mysql Wordpress DB and User :

```sh
sudo mysql -u root 
CREATE USER 'wdp1'@'localhost' IDENTIFIED BY 'yoursecurepassword';
CREATE DATABASE wordpress;
GRANT ALL PRIVILEGES ON wordpress.* TO 'wdp1'@'localhost';
FLUSH PRIVILEGES;
EXIT
```
### Sécuriser MariaDB :

```sh
sudo mysql_secure_installation
```

```sh
enter root password, enter for none
unix_socket authentification [no]
change the root password (ajouter votre mot de pase root)
suppression des users anonymes [yes]
Disallow root login remotly [yes]
suppresion de la base de données test [yes]
Reload priviliges tables now?[yes]
```

```sh
sudo systemctl restart mariadb
```
### Install Wordpress:

```sh
sudo apt install unzip
cd /var/www/html
sudo wget https://wordpress.org/latest.zip
sudo unzip latest.zip
sudo rm latest.zip
```
### Changer les permissions dans Wordpress Dir:
```sh
sudo chown -R www-data:www-data wordpress/
cd wordpress/
sudo find . -type d -exec chmod 755 {} \;
sudo find . -type f -exec chmod 644 {} \;

sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
```

### Entrer les DB credentials dans le fichier wp-config.php:

```sh
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php
```

```sh
sudo rm wp-config-sample.php
sudo chmod 400 /var/www/html/wordpress/wp-config.php
```
- Changer : DB_name, DB_user and DB_password :

```sh
** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress_db' );

/** MySQL database username */
define( 'DB_USER', 'wordpress_user' );

/** MySQL database password */
define( 'DB_PASSWORD', 'password' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );
```


### Genrate secret key in wp-config.php:

Allez à  :https://api.wordpress.org/secret-key/1.1/salt/
 
copier /coller pour remplacer les clés par défaut:

```sh
define('AUTH_KEY',         'O1EG$6bUu9g1jJ_u]VVIN.M{5xeAXUT)@Deu*L1.AZUCMe)<T+QJ|7x_v&$hUFef');
define('SECURE_AUTH_KEY',  '=xCDj2[T!:A1-1![RZaxG$#uZ^pZx!ni=U|[qnBj>{U)^jGR9 1eUEi-Rq0=|m?H');
define('LOGGED_IN_KEY',    '-9KplkmXt3+J9[Dg[fHsK?z*frxf)j->h`rZ9$%%-ur9Z9TMGh(}~b?m:X>=W76E');
define('NONCE_KEY',        'epOdHL-m-u~saV#0gg@*#m?8c.yK[n!>}.;-VG1 |XBdwj^?7~Ktq&ig0^~!Qgie');
define('AUTH_SALT',        'fD[(*6tNMe+G0u@#Vl>PSYLt)!06/hOI0|a-v~YmQn!{$P_&Wu3h(p[T>R638!!G');
define('SECURE_AUTH_SALT', '!M|=kw_Q!fWaLcP8HK,9*W_-MZN!3JO4kBR[vf}-Xdn{AbEl-..b^l&|s&6v3r-b');
define('LOGGED_IN_SALT',   'c/j9bs=$|3V@c0SPv?#H=NJa+19+pYn,%FYN:16yX-,&@8|bvs<^/K$nt4@$_0CF');
define('NONCE_SALT',       '`%,>]I9qq(ZySP3-mY**P_.H>Zb/Ughs%D]WD u7*>F,%t7A|NH>sOs`a*fcyN1V');
```
### Créer un Virtualhost Wordpress:

```sh
cd /etc/apache2/sites-available/
sudo touch wordpress.conf
sudo nano wordpress.conf
```

```sh
<VirtualHost *:80>
ServerName www.mediasante.com
DocumentRoot /var/www/html/wordpress
ServerAlias       192.168.1.20

<Directory /var/www/html/wordpress>
AllowOverride All
</Directory>

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```
### Activer Wordpress Vhost et le rewrite module:

```sh
sudo a2enmod rewrite
a2dissite 000-default
sudo a2ensite wordpress.conf
sudo a2enmod vhost_alias
sudo systemctl restart apache2
sudo systemctl status apache2.service -l --no-pager
reboot
```
### Vérifier les erreurs de syntaxe:

```sh
sudo apachectl configtest
```
```sh
sudo systemctl reload apache2
```
### Pour l'Erreur: AH00558: Could not reliably determine the server’s fully qualified domain name:

```sh
nano /etc/apache2/apache2.conf
```
-Rajouter a la fin:

```
ServerName 127.0.0.1
```
### Configuration de PHP pour l’installation de WordPress sur Debian :
```sh
sudo nano /etc/php/8.2/apache2/php.ini
```

```sh
upload_max_filesize 2048M
post_max_size 2048M
memory_limit 2048M
max_execution_time 300
max_input_time 300
```

### Finir l'installation de Wordpress:
``Allez à: http://yourdomain.com``
# 4.Installer NRPE PLUGIN:
## Install NRPE plugin sur le Client:
<img src="/img\posts\Supervision\nrpe.png" alt="nrpe" width="800" height="200">
1. Installer NRPE (Nagios Remote Plugin Executor) sur le webserver : 

```sh
sudo apt update && sudo apt upgrade -y
sudo apt install nagios-nrpe-server nagios-plugins
```
2. Editer NRPE Config file:

```sh
sudo nano /etc/nagios/nrpe.cfg 
#search for : allowed_hosts
allowed_hosts= 127.0.0.1,nagios_ip_address
#save and quit
sudo systemctl restart nagios-nrpe-server 
```

## Install NRPE plugin sur Nagios server :

```sh
sudo apt update && sudo apt upgrade -y
sudo apt install nagios-nrpe-plugin
```
Ce plugin sera installé sur : `/usr/lib/nagios/plugins`, on devra le copier avec les autre plugins dans:
```sh
sudo cp /usr/lib/nagios/plugins/check_nrpe  /usr/local/nagios/libexec/
cd /usr/local/nagios/libexec/
ls
```
### Lancer check_nrpe pour le webserver et voir la version nrpe:
Depuis `/usr/nagios/libexec/`

```sh
./check_nrpe -H webserv_ip_add 
output: NRPE v.2.15
# verifier les utilisateurs connectés sur le webserver:
./check_nrpe -H webserv_ip_address -c check_users
```
# 5.Fichiers de configuration Nagios et Wordpress:
## Créer un Dossier pour les fichiers '.cfg' :
` mkdir /usr/local/nagios/server_conf `

```sh
echo "#CONFIG SERVERS" >> /usr/local/nagios/etc/nagios.cfg 
echo "cfg_dir=/usr/local/nagios/server_conf" >> /usr/local/nagios/etc/nagios.cfg
```
### Créer les fichiers de config du serveur Nagios et les fichiers de commandes:
On va dans le fichier de config créé: 
```sh
cd /usr/local/nagios/server_conf/
on crée un fichier nagios-server.cfg  pour la config du serveur Nagios
et un fichier pour les commandes commands.cfg
```
- **nagios-server.cfg** :
<img src="/img\posts\Supervision\Nagios_srv.cfg_1.png" width="700" height="400">
<img src="/img\posts\Supervision\Nagios_srv.cfg_2.png" width="500" height="500">
<img src="/img\posts\Supervision\Nagios_srv_cfg_3.png" width="600" height="250">
- Nagios fichier **commands.cfg**:
<img src="/img\posts\Supervision\Nagios_srv_commands.cfg_1.png" width="800" height="450">
<img src="/img\posts\Supervision\Nagios_srv_commands.cfg_2.png" width="800" height="250">

### Créer les fichiers de config du Webserver et les fichiers de commandes:
-Dans `/usr/local/nagios/server_conf`
-**wordpress.cfg**:
<img src="/img\posts\Supervision\wordpress.cfg_1.png" width="500" height="600">
<img src="/img\posts\Supervision\wordpress.cfg_2.png" width="500" height="500">

-**Commandes** sur le serveur Wordpress dans le fichier: `/etc/nagios/nrpe.cfg`
<img src="/img\posts\Supervision\wordpres_nrpe.cfg_1.png" width="900" height="200">
<img src="/img\posts\Supervision\wordpres_nrpe.cfg_2.png" width="400" height="40">

# 6.Interface Nagios:
-Aller à : http://nagios-ip-add/nagios

<img src="/img\posts\Supervision\nagios-interf.png" alt="Interface Nagios" width="900" height="500">

# 7.Documentation conduite à tenir lors des alertes:
## Serveur Nagios :
### Sonde numéro: 1
- Nom de la sonde : RAM
- Objectif de la sonde : Mémoire vive utilisée
- Paramétrage de la sonde (valeur pour warning et critical) : w =75%, c =85%
- Action à entreprendre en cas d’anomalie (dont les commandes Linux Correspondantes) :
    - Commande : ` free -h `
        - Retourne la valeur de mémoire vive libre et celle utilisée sans détails
    - Commande: `ps -e --format uid,pid,ppid,tty,rss,cmd --sort -rss`
        - e : tous les utilisateurs, format : définir les champs user ID process ID et parent process ID, sort : trier par la plus haute utilisation de RAM
     - Actions :
       Les résultats de la commande renseignent sur les processus qui consomment le plus de mémoire vive.
	   L’équipe IT de niveau 1 doit envoyer un rapport au niveau 2 avec la liste des processus qui consomment le plus de mémoire vive.
     - Commande : ` top `
         - Permet d’afficher en temps réel, l’utilisation de mémoire vive et du CPU avec les infos sur le PID : processus ID et le nom de l’utilisateur.
         - Taper pendant l’affichage des résultats `M` : tri par mémoire vive, `P` : tri par utilisation CPU, `N` : tri par processus.
     - Actions :
       Les IT de niveau 1 en complément de la commande précédente, permettent de faire une investigation initiale, et envoyer un rapport au niveau 2 pour approfondir les explorations.
### Sonde numéro: 2
- Nom de la sonde : CPU
- Objectif de la sonde : Utilisation du CPU à : 1/5/15 dernières minutes
- Paramétrage de la sonde (valeur pour warning et critical) : w = 70% la dernière minute, 60% 5 dernières minutes, 50% 15 dernières minutes, c= 90% la dernière minute, 80% 5 dernières minutes, 70% 15 dernières minutes
- Action à entreprendre en cas d’anomalie (dont les commandes Linux Correspondantes) :
    - Commande : ` top `
        - Permet d’afficher en temps réel, l’utilisation de mémoire vive et du CPU avec les infos sur le PID : processus ID et le nom de l’utilisateur
        - Taper pendant l’affichage des résultats `U`: tri par mémoire vive, `P`: tri par  utilisation CPU , `N` : tri par processus.
    - Commande : `ps -e --format uid,pid,ppid,tty,%cpu,cmd --sort -%cpu`
        - Utilisation du %CPU avec les champs uid,pid,ppid et tty et tri descendant.
    - Actions : 
	 Les IT de niveau 1 en complément de la commande précédente, permettent de faire une investigation initiale, et envoyer un rapport au niveau 2 pour approfondir les explorations.
### Sonde numéro: 3
-	Nom de la sonde : Espace disque
-	Objectif de la sonde : Espace disque utilisé en pourcentage
-	Paramétrage de la sonde (valeur pour warning et critical) : w = 70% , c= 85%
-	Action à entreprendre en cas d’anomalie (dont les commandes Linux Correspondantes) :
    - Commande : `df -h`
        - Permet de lister les points de montage des partitions et l’espace disque utilisé.
    - Commande : ` sudo du -sh /* | sort -hr `
        - Permet d’explorer la taille des répertoires à la racine du système
    - Actions :
        - Il faut explorer par la suite le chemin de la racine par le chemin d’un répertoire qui occupe un volume important 
        - Identifier les fichiers qui prennent le plus de place sur le disque : fichiers logs par exemple à archiver et à transférer sur un serveur de centralisation de logs.
        - Taille importante prise par des programmes non utilisés, penser à désinstaller ces programmes.
### Sonde numéro: 4
- Nom de la sonde : Sessions Utilisateurs
- Objectif de la sonde : Nombre d’utilisateurs connectés sur le serveur Nagios
- Paramétrage de la sonde (valeur pour warning et critical) : ` w = 2, c= 2 ` 
- Action à entreprendre en cas d’anomalie (dont les commandes Linux Correspondantes) :
    - Commande : ` who or users `
        - Who : Permet de vérifier le nom de l’utilisateur connecté sur la machine, depuis combien de temps, et depuis qu’elle adresse IP, tty depuis terminal local ou en SSH
        - Users : permet d’afficher les noms des utilisateurs connectés à la machine.
    - Commande : ` last `
        - Permet de donner des informations sur les utilisateurs connectés depuis le dernier redémarrage, avec l’horaire de connexion et de déconnexion.
    - Actions :
        - Identifier les noms des users connectés et l’IP de la machine, si cette dernière     fait partie des machines locales.
        - Risque d’une brèche de sécurité, avec possibilité de corruption d’un compte administrateur, ou élévation de privilèges.
        - Faire un rapport des commandes précédentes, et alerter le niveau 2.

### Sonde numéro: 5
- Nom de la sonde : Etat du serveur web
- Objectif de la sonde : état du serveur et temps de réponse
- Paramétrage de la sonde (valeur pour warning et critical) : w = 500 ms, c= 1000 ms
- Action à entreprendre en cas d’anomalie (dont les commandes Linux Correspondantes)
    - Commande: `sudo systemctl status apache2`
        - Permet de voir si le service est actif et depuis combien de temps
    - Commande : `ps -e -o comm,etime,user | grep apache2`
        - Permet d’afficher les processus apache2 qui sont en cours d’exécution, avec les
        - paramètres : l’utilisateur du processus, depuis combien de temps le processus s’exécute, et la commande associée.
- Actions :
    - Voir le status du serveur web, relancer le service apache2 s’il est arrêté avec la commande ` sudo systemctl restart apache2 `
    - Voir les fichiers de log d’apache : `/var/log/apache2/error.log` et `/var/log/apach2/access.log`, faire un rapport au niveau 2 si des erreurs critiques sont détectés, ou des tentatives d’accès non autorisés.
        - « emerg » Le plus haut niveau d’erreur (type urgence) 
        - « alert » Niveau d’alerte élevé, il faut rapidement réagir : fichier inaccessible à cause de droit utilisateurs par exemple.
        - « crit » Conditions critiques (le socket n’a pu être trouvé)
        - « error » Erreurs de type script (problème dans les headers)
        - « warn » Avertissements (problème de type process linux)
        - « notice » Evènement important mais normal, souvent de type erreur mineur de code.

### Sonde numéro: 6
- Nom de la sonde : Page Web tourne
- Objectif de la sonde : temps de réponse de la page web
- Paramétrage de la sonde (valeur pour warning et critical) : w = 500 ms, c= 1000ms
- Action à entreprendre en cas d’anomalie (dont les commandes Linux Correspondantes) :
    - Actions :
        - Tester la page web sur le navigateur, et le temps de réponse.

## Serveur Wordpress :
- Idem que Nagios pour les sondes de 1 à 6
### Sonde numéro: 7
- Nom de la sonde : Base de Données
- Objectif de la sonde : La base de données est fonctionnelle
- Paramétrage de la sonde (valeur pour warning et critical) : w = 1000 ms, c= 2000ms
- Action à entreprendre en cas d’anomalie (dont les commandes Linux Correspondantes)
    - Commandes : `sudo systemctl status mariadb`
        - Voir le status de la base de données.
    - Commande: `sudo netstat -tlpn | grep mariadbd `
        - Vérifier si la base de données écoute bien sur le port défini : 3306

# 8.Installation et configuration de Rsyslog:
## Installer Rsyslog:
```sh
sudo apt update && sudo apt upgrade -y
sudo apt install rsyslog
systemctl status rsyslog

```
## Désactiver systemd-journald logging:
Afin d'eviter d'avoir des doublons de logging:

```sh
rm -rf /var/log/journal
```
## Fichiers de configurations:
-Dans   `/etc/rsyslog.conf `
Creation d'un template pour récupérer les logs selon la structure /Nom_de_la_Machine/Programme.log:
`$template DynamicFile,"/var/log/%HOSTNAME%/%PROGRAMNAME%.log"
if $fromhost-ip != '127.0.0.1' then -?DynamicFile
& stop`

```sh
# /etc/rsyslog.conf configuration file for rsyslog
#
# For more information install rsyslog-doc and see
# /usr/share/doc/rsyslog-doc/html/configuration/index.html


#################
#### MODULES ####
#################

module(load="imuxsock") # provides support for local system logging
module(load="imklog")   # provides kernel logging support
#module(load="immark")  # provides --MARK-- message capability

# provides UDP syslog reception
module(load="imudp")
input(type="imudp" port="514")
$AllowedSender UDP, 127.0.0.1, 192.168.1.20/24, 
# provides TCP syslog reception
#module(load="imtcp")
#input(type="imtcp" port="514")


###########################
#### GLOBAL DIRECTIVES ####
###########################

#
# Set the default permissions for all log files.
#
$FileOwner root
$FileGroup adm
$FileCreateMode 0640
$DirCreateMode 0755
$Umask 0022

#
# Where to place spool and state files
#
$WorkDirectory /var/spool/rsyslog

#
# Include all config files in /etc/rsyslog.d/
#
$IncludeConfig /etc/rsyslog.d/*.conf
#################
## TEMPLATES ####
#################

$template DynamicFile,"/var/log/%HOSTNAME%/%PROGRAMNAME%.log"
if $fromhost-ip != '127.0.0.1' then -?DynamicFile
& stop



###############
#### RULES ####
###############

#
# Log anything besides private authentication messages to a single log file
#
*.*;auth,authpriv.none		-/var/log/syslog

#
# Log commonly used facilities to their own log file
#
auth,authpriv.*			/var/log/auth.log
cron.*				-/var/log/cron.log
kern.*				-/var/log/kern.log
mail.*				-/var/log/mail.log
user.*				-/var/log/user.log

#
# Emergencies are sent to everybody logged in.
#
*.emerg				:omusrmsg:*

```

- Sur les machines clientes installer Rsyslog et configurer le fichier `/etc/rsylog.conf`

```sh
##############################
# Send logs to rsyslog server:
##############################
*.*		@192.168.1.30:514
############################################
# Put logs in queue if server no responding
############################################
$ActionQueueFileName fwdRule1 # unique name prefix for spool files
$ActionQueueMaxDiskSpace 1g  # 1gb space limit (use as much as possible)
$ActionQueueSaveOnShutdown on  # save messages to disk on shutdown
$ActionQueueType LinkedList # run asynchronously
$ActionResumeRetryCount -1 # infinite retries if host is down
##################
# Rate limit logs
##################
$SystemLogRateLimitInterval 5
$SystemLogRateLimitBurst 100
```

