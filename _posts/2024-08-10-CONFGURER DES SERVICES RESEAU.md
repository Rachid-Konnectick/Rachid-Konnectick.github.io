---
layout: post
title: "CONFIGURER DES SERVICES RESEAUX ET EQUIPEMENTS D'INTERCONNEXION"
subtitle: "Configuration d'équipments Cisco"
date: 2024-08-10
background: '/img/posts/Reseau/cisco.jpg'
---

# Sommaire:

 1. Plan d’adressage IP 
 2. Configuration DNS 
 3. Configuration des routeurs (comprenant les tables de routage, les sous-interfaces et les NAT) 
 4. Configuration des Switch (comprenant les paramètres DHCP, les VLAN, le LACP et les ACL)
 5.  Trois Préconisations de l'ANSII 

## Schema logique de la structure :
<img src="/img\posts\Reseau\schema.png" alt="Schema logique" width="500" height="600">

## Capture du schéma logique sur packet tracer:
<img src="/img\posts\Reseau\pkt.png" alt="Schema packet tracer" width="900" height="600">


## Objectifs:
- Réalisation du plan d’adressage réseau IPv6 et IPv4
- Liaison des routeurs des trois bâtiments via un protocole de routage
- Création et configuration du LAN du bâtiment Rouge :
    - VLAN ;
    - DHCP ;
    - ACL ;
    - NAT surchargé (les VLAN doivent avoir accès aux différents routeurs de l’infrastructure via des pings)
- Serveur DNS (le rendre accessible uniquement depuis le LAN)
- LACP : Doubler les switch de distribution et faire du load-balancing entre eux pour assurer la redondance
- Création et configuration du LAN du bâtiment Vert :
    - serveur web ;
    - NAT statique (les postes des VLAN – ou un poste branché sur un des routeurs – doivent avoir accès au serveur web).
- Rendre le serveur web accessible depuis le WAN et le LAN

## 1.Plan d’adressage IP :
<img src="/img\posts\Reseau\IP1.png" alt="adressage IP" width="500" height="300">
<img src="/img\posts\Reseau\IP2.png" alt="adressage IP" width="600" height="300">
<img src="/img\posts\Reseau\IP3.png" alt="adressage IP" width="700" height="200" >
<img src="/img\posts\Reseau\IP4.png" alt="adressage IP" width="700" height="400">

##  2.Configuration DNS :
<img src="/img\posts\Reseau\DNS.png" alt="configuration DNS" width="541" height="375">

## 3.Configuration des routeurs (comprenant les tables de routage, les sous-interfaces et les NAT) :
### Routeur Accueil (Bleu) :
<img src="/img\posts\Reseau\Routeur-accueil1.png" alt="Routeur-accueil">
<img src="/img\posts\Reseau\Routeur-accueil2.png" alt="Routeur-accueil">

### Routeur Salle Serveurs (Vert) :
<img src="/img\posts\Reseau\Routeur-serveur1.png" alt="Routeur-serveur">
<img src="/img\posts\Reseau\Routeur-serveur2.png" alt="Routeur-serveur">

### Routeur Bureaux (Rouge) :
<img src="/img\posts\Reseau\Routeur-bureaux1.png" alt="Routeur-bureaux">
<img src="/img\posts\Reseau\Routeur-bureaux2.png" alt="Routeur-bureaux">
<img src="/img\posts\Reseau\Routeur-bureaux3.png" alt="Routeur-bureaux">

## 4.Configuration des Switchs (comprenant les paramètres DHCP, les VLAN, le LACP et les ACL) :
### 4.1. Configuration des Switchs de type layer 3 :

#### **Switch Core:**
<img src="/img\posts\Reseau\core-switch.png" alt="core-switch">

#### **Switch Distributed 1:**
<img src="/img\posts\Reseau\switch1-dist1.png" alt="Distribution switch1">
<img src="/img\posts\Reseau\switch1-dist2.png" alt="Distribution switch1">

#### **Switch Distributed 2:**
<img src="/img\posts\Reseau\switch2-dist1.png" alt="Distribution switch2">
<img src="/img\posts\Reseau\switch2-dist2.png" alt="Distribution switch2">

### 4.2. ACL et DHCP sur Routeur Rouge :
<img src="/img\posts\Reseau\RR-ACL.png" alt="Routeur-rouge">
<img src="/img\posts\Reseau\RR-ACL2.png" alt="Routeur-rouge">
<img src="/img\posts\Reseau\RR-ACL3.png" alt="Routeur-rouge">

### 4.3. Configuration des Switchs de type layer 2 :

#### **Switch Serveur WEB:**
<img src="/img\posts\Reseau\switch-web.png" alt="switch-web">

#### **Switch Réunion:**
<img src="/img\posts\Reseau\switch-reun.png" alt="switch-reunion">

#### **Switch Accueil:**
<img src="/img\posts\Reseau\switch-accu.png" alt="switch-accueil">

#### **Switch Showroom:**
<img src="/img\posts\Reseau\switch-showroom.png" alt="switch-showroom">

#### **Switch Direction:**
<img src="/img\posts\Reseau\switch-direct.png" alt="switch-direction">

#### **Switch RH:**
<img src="/img\posts\Reseau\switch-RH.png" alt="switch-RH">

#### **Switch Commerciaux/Chargés de clientèle:**
<img src="/img\posts\Reseau\switch-commerce.png" alt="switch-commercial">

#### **Switch Finance:**
<img src="/img\posts\Reseau\switch-finance.png" alt="switch-finance">

#### **Switch Service Informatique:**
<img src="/img\posts\Reseau\switch-informat.png" alt="switch-informatique">

#### **Switch du serveur DNS :**
<img src="/img\posts\Reseau\switch-dns.png" alt="switch-DNS">

## 5. Recommandations ANSSI:
### 1.Première Recommandation :
Protéger les flux d'administration transitant sur un réseau tiers **R21** :
Lorsque les flux d’administration transitent par un réseau tiers ou hors de locaux sécurisés, ils doivent être chiffrés et authentifiés de bout en bout, en utilisant par exemple un tunnel IPsec.
![600](Pasted%20image%2020250218160317.png)
### 2.Deuxième Recommandation :
Recommandation **R45** : Une politique de sauvegarde doit être définie et appliquée pour le SI d'administration.
Prévoir des sauvegardes hors ligne pour les éléments les plus critiques.

Recommandation **R46** : Créer une zone d'administration dédiée à la journalisation.
Mettre en place un contrôle d'accès spécifique si nécessaire.
Centralisation : Remonter de manière centralisée l'ensemble des journaux pour faciliter la corrélation des événements.

### 3.Troisième Recommandation :
Privilégier une authentification double facteur pour les actions d'administration **R36** :
Utilisation recommandée d'une authentification multi-facteurs (au moins deux facteurs) pour les actions d'administration.
Recommandation d'utiliser des certificats électroniques de confiance comme élément d'authentification.