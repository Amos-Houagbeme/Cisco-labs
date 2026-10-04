# Lab 2 --- Réseau d'entreprise Cisco

## État du laboratoire

Ce lab Packet Tracer construit une infrastructure réseau d'entreprise en
appliquant le principe :

**Problème → Besoin → Technologie → Configuration → Test → Résultat →
Documentation**

L'objectif est de ne pas ajouter des technologies uniquement pour les
afficher sur un CV.

## Architecture

``` text
                         RÉSEAU EXTERNE
                              │
                             FAI
                              │
                         10.0.1.0/30
                              │
                             R1
                              │
                         10.0.0.0/30
                              │
                          CORE-L3
                       /     |     \
                     SW1    SW2    SW3
                      │      │      │
                   ADMIN/IT RH/USERS SERVERS
```

### Rôles

-   **CORE-L3** : routage inter-VLAN, DHCP et passerelles des VLAN.
-   **SW1** : ADMIN + IT.
-   **SW2** : RH + USERS.
-   **SW3** : SERVERS.
-   **R1** : routeur frontière de l'entreprise.
-   **FAI** : routeur représentant le fournisseur d'accès à Internet.

## Plan d'adressage

    VLAN Nom          Réseau            Passerelle
  ------ ------------ ----------------- --------------
      10 ADMIN        192.168.10.0/24   192.168.10.1
      20 IT           192.168.20.0/24   192.168.20.1
      30 RH           192.168.30.0/24   192.168.30.1
      40 USERS        192.168.40.0/24   192.168.40.1
      50 SERVERS      192.168.50.0/24   192.168.50.1
      99 MANAGEMENT   192.168.99.0/24   192.168.99.1
     100 NATIF        Aucun SVI         ---

Management : - CORE-L3 : 192.168.99.1 - SW1 : 192.168.99.11 - SW2 :
192.168.99.12 - SW3 : 192.168.99.13

Transit : - CORE-L3 ↔ R1 : 10.0.0.0/30 --- CORE-L3 10.0.0.1, R1
10.0.0.2 - R1 ↔ FAI : 10.0.1.0/30 --- R1 10.0.1.1, FAI 10.0.1.2

## 1. VLAN

VLAN configurés :

-   VLAN 10 --- ADMIN
-   VLAN 20 --- IT
-   VLAN 30 --- RH
-   VLAN 40 --- USERS
-   VLAN 50 --- SERVERS
-   VLAN 99 --- MANAGEMENT
-   VLAN 100 --- NATIF

## 2. Trunks

Trunks 802.1Q configurés :

  Liaison         VLAN autorisés   VLAN natif
  --------------- ---------------- ------------
  CORE-L3 ↔ SW1   10,20,99         100
  CORE-L3 ↔ SW2   30,40,99         100
  CORE-L3 ↔ SW3   50,99            100

**VLAN 99** est le VLAN de management. **VLAN 100** est le VLAN natif
des trunks. Le VLAN natif n'est pas utilisé pour le routage.

## 3. Routage inter-VLAN

`ip routing` est activé sur CORE-L3.

SVI :

``` text
Vlan10 → 192.168.10.1/24
Vlan20 → 192.168.20.1/24
Vlan30 → 192.168.30.1/24
Vlan40 → 192.168.40.1/24
Vlan50 → 192.168.50.1/24
Vlan99 → 192.168.99.1/24
```

Le VLAN 100 n'a pas de SVI.

Les tests inter-VLAN ont réussi : les postes peuvent joindre leur
passerelle et des postes d'autres VLAN.

## 4. DHCP

DHCP est configuré sur CORE-L3 pour les VLAN 10, 20, 30 et 40.

Les adresses `.1` à `.20` sont exclues.

Exemple :

``` cisco
ip dhcp excluded-address 192.168.10.1 192.168.10.20

ip dhcp pool VLAN10-ADMIN
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
```

Test validé : ADMIN01 a reçu `192.168.10.21` avec la passerelle
`192.168.10.1`.

Vérification :

``` cisco
show ip dhcp binding
```

## 5. Routage statique

### CORE-L3 ↔ R1

CORE-L3 Gi1/0/1 est devenue une interface routée avec `no switchport`.

``` text
CORE-L3 = 10.0.0.1/30
R1      = 10.0.0.2/30
```

La connectivité a été validée.

### Route par défaut de CORE-L3

``` cisco
ip route 0.0.0.0 0.0.0.0 10.0.0.2
```

Signification : toute destination inconnue est envoyée à R1.

### Routes de retour sur R1

``` cisco
ip route 192.168.10.0 255.255.255.0 10.0.0.1
ip route 192.168.20.0 255.255.255.0 10.0.0.1
ip route 192.168.30.0 255.255.255.0 10.0.0.1
ip route 192.168.40.0 255.255.255.0 10.0.0.1
ip route 192.168.50.0 255.255.255.0 10.0.0.1
ip route 192.168.99.0 255.255.255.0 10.0.0.1
```

Le routage retour a été validé avec notamment :

``` text
R1 → 192.168.10.1  : OK
R1 → 192.168.10.10 : OK
```

Une erreur de frappe (`102.168.10.0` au lieu de `192.168.10.0`) a été
détectée et corrigée.

## 6. Routeur FAI

Un deuxième Cisco 2911 a été ajouté pour représenter le **routeur du
fournisseur d'accès à Internet (FAI)**.

Il ne représente pas un deuxième routeur de l'entreprise.

Lien :

``` text
R1 Gi0/1 → 10.0.1.1/30
FAI Gi0/0 → 10.0.1.2/30
```

Le lien est opérationnel :

``` cisco
R1# ping 10.0.1.2
```

La route par défaut de R1 vers le FAI est configurée :

``` cisco
ip route 0.0.0.0 0.0.0.0 10.0.1.2
```

## 7. Pourquoi `8.8.8.8` ne répondait pas

Le test :

``` text
PC → 8.8.8.8
```

atteignait R1, mais R1 ne disposait pas encore d'un véritable réseau
externe derrière le FAI dans la simulation.

Point important : **une route par défaut ne crée pas Internet**. Elle
indique seulement où envoyer les destinations inconnues.

Nous avons décidé de ne pas ajouter artificiellement un serveur externe
pour « faire marcher » `8.8.8.8`. Le FAI doit représenter la sortie vers
Internet ; la simulation externe sera traitée proprement lors de la
reprise.

## 8. Technologies volontairement non implémentées

### STP / RSTP

Pas de redondance ou de boucle de couche 2 dans cette topologie. STP
sera traité dans un futur lab dédié à la haute disponibilité.

### EtherChannel

Une seule liaison par switch vers CORE-L3. Aucun besoin réel
d'agrégation. À traiter dans un futur lab de redondance.

### OSPF

Le routage statique est utilisé ici pour comprendre les mécanismes de
base. OSPF sera introduit dans une architecture où plusieurs routeurs et
chemins justifient son utilisation.

## 9. Vérifications utiles

``` cisco
show vlan brief
show interfaces trunk
show ip interface brief
show ip route
show ip dhcp binding
show ip dhcp pool
show running-config
ping <adresse-ip>
```

## 10. État actuel

### Terminé

-   [x] VLAN
-   [x] Affectation des ports
-   [x] Trunks 802.1Q
-   [x] VLAN natif 100
-   [x] VLAN management 99
-   [x] Routage inter-VLAN
-   [x] DHCP
-   [x] Liaison routée CORE-L3 ↔ R1
-   [x] Route par défaut CORE-L3 → R1
-   [x] Routes statiques de retour sur R1
-   [x] Ajout du routeur FAI
-   [x] Liaison R1 ↔ FAI
-   [x] Route par défaut R1 → FAI

### À faire à la reprise

-   [ ] Finaliser la simulation de la sortie Internet
-   [ ] Comprendre puis configurer NAT/PAT sur R1
-   [ ] Tester les flux internes → externes
-   [ ] Configurer les ACL
-   [ ] Ajouter la sécurité pertinente
-   [ ] Tests finaux
-   [ ] Troubleshooting
-   [ ] Documentation finale GitHub

## 11. Principe du projet

Le lab suit :

``` text
Problème
   ↓
Besoin
   ↓
Choix technologique
   ↓
Configuration
   ↓
Test
   ↓
Résultat
   ↓
Documentation
```

Les technologies ne sont pas ajoutées uniquement pour augmenter la liste
de compétences du portfolio.
