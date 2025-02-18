---
layout: post
title: "CONFIGURER DES SERVICES RESEAUX ET EQUIPEMENTS D'INTERCONNEXION"
subtitle: "Configuration d'équipments cisco"
date: 2024-08-10
background: '/img/posts/cisco.jpg'
---

# Sommaire:

 1. Plan d’adressage IP 
 2. Configuration DNS 
 3. Configuration des routeurs (comprenant les tables de routage, les sous-interfaces et les NAT) 
 4. Configuration des Switch (comprenant les paramètres DHCP, les VLAN, le LACP et les ACL)
 5.  Trois Préconisations de l'ANSII 

## Schema logique de la structure :
![500](Schéma+réseau+accessible.png)

## Capture du schéma logique sur packet tracer:
![](Pasted%20image%2020250218155551.png)

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
![](Pasted%20image%2020250218141024.png)
![](Pasted%20image%2020250218141046.png)
![](Pasted%20image%2020250218141111.png)
![](Pasted%20image%2020250218141131.png)


##  2.Configuration DNS :
![](Pasted%20image%2020250218151521.png)
## 3.Configuration des routeurs (comprenant les tables de routage, les sous-interfaces et les NAT) :
### Routeur Accueil (Bleu) :
![](Pasted%20image%2020250218151625.png)
![](Pasted%20image%2020250218151637.png)
![](Pasted%20image%2020250218151646.png)
![](Pasted%20image%2020250218151656.png)
![](Pasted%20image%2020250218151707.png)
### Routeur Salle Serveurs (Vert) :
![](Pasted%20image%2020250218151730.png)
![](Pasted%20image%2020250218151739.png)
![](Pasted%20image%2020250218151750.png)
![](Pasted%20image%2020250218151802.png)
![](Pasted%20image%2020250218151813.png)
### Routeur Bureaux (Rouge) :
![](Pasted%20image%2020250218151837.png)
![](Pasted%20image%2020250218151846.png)
![](Pasted%20image%2020250218151856.png)
![](Pasted%20image%2020250218151907.png)
![](Pasted%20image%2020250218151922.png)
## 4.Configuration des Switchs (comprenant les paramètres DHCP, les VLAN, le LACP et les ACL) :
### 4.1. Configuration des Switchs de type layer 3 :

#### **Switch Core:**
![](Pasted%20image%2020250218152017.png)
![](Pasted%20image%2020250218152027.png)
#### **Switch Distributed 1:**
![](Pasted%20image%2020250218152054.png)
![](Pasted%20image%2020250218152102.png)
![](Pasted%20image%2020250218152111.png)
#### **Switch Distributed 2:**
![](Pasted%20image%2020250218153418.png)
![](Pasted%20image%2020250218153427.png)
![](Pasted%20image%2020250218153436.png)
### 4.2. ACL et DHCP sur Routeur Rouge :
![](Pasted%20image%2020250218153509.png)
![](Pasted%20image%2020250218153519.png)
![](Pasted%20image%2020250218153529.png)
### 4.3. Configuration des Switchs de type layer 2 :

#### **Switch Serveur WEB:**
![](Pasted%20image%2020250218153606.png)
#### **Switch Réunion:**
![](Pasted%20image%2020250218153626.png)
#### **Switch Accueil:**
![](Pasted%20image%2020250218153653.png)
#### **Switch Showroom:**
![](Pasted%20image%2020250218153717.png)
#### **Switch Direction:**
![](Pasted%20image%2020250218153742.png)
![](Pasted%20image%2020250218153752.png)
#### **Switch RH:**
![](Pasted%20image%2020250218153814.png)
![](Pasted%20image%2020250218153822.png)
#### **Switch Commerciaux/Chargés de clientèle:**
![](Pasted%20image%2020250218153850.png)
![](Pasted%20image%2020250218153858.png)
#### **Switch Finance:**
![](Pasted%20image%2020250218153920.png)
![](Pasted%20image%2020250218153929.png)
#### **Switch Service Informatique:**
![](Pasted%20image%2020250218153954.png)
![](Pasted%20image%2020250218154002.png)
#### **Switch du serveur DNS :**
![](Pasted%20image%2020250218154022.png)
![](Pasted%20image%2020250218154031.png)

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