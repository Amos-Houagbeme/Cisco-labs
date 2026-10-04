# Lab 2 — Réseau d'entreprise

## 📌 Présentation

Ce projet consiste à concevoir, configurer, sécuriser et diagnostiquer un réseau d'entreprise à l'aide de **Cisco Packet Tracer**.

L'objectif est de reproduire une infrastructure réseau réaliste comprenant plusieurs services séparés par des VLAN, un routage inter-VLAN, du routage dynamique avec OSPF, de la redondance au niveau 2, du NAT/PAT et des règles de filtrage avec des ACL.

Le projet met l'accent sur la **compréhension du fonctionnement du réseau**, la capacité à **diagnostiquer des problèmes** et la documentation technique.

---

## 🎯 Objectifs

À la fin du projet, je dois être capable de :

* concevoir une architecture réseau d'entreprise ;
* réaliser un plan d'adressage IPv4 ;
* créer et administrer des VLAN ;
* configurer des ports Access et Trunk ;
* comprendre le fonctionnement de 802.1Q ;
* mettre en place le routage inter-VLAN ;
* configurer DHCP ;
* comprendre et configurer STP ;
* mettre en place EtherChannel ;
* comprendre et configurer du routage statique ;
* configurer OSPF ;
* mettre en place NAT/PAT ;
* contrôler les communications avec des ACL ;
* sécuriser les accès aux équipements ;
* utiliser les commandes de diagnostic Cisco IOS ;
* provoquer et résoudre des pannes réseau ;
* documenter une infrastructure réseau de manière professionnelle.

---

# 🏢 1. Contexte

L'entreprise possède plusieurs services :

* Administration
* Informatique
* Ressources humaines
* Utilisateurs
* Serveurs

Tous les équipements ne doivent pas se trouver dans le même réseau.

L'infrastructure doit donc permettre :

1. de segmenter le réseau ;
2. de permettre les communications nécessaires entre les services ;
3. de contrôler les communications ;
4. de fournir un accès vers un réseau externe ;
5. d'assurer une certaine redondance ;
6. de faciliter l'administration et le dépannage.

---

# 🏗️ 2. Architecture

## Architecture logique

```text
                         INTERNET
                            │
                            │
                           R1
                    Routeur frontière
                            │
                            │
                       CORE-L3
                    Switch multilayer
                       /    |    \
                      /     |     \
                     /      |      \
                   SW1     SW2     SW3
                    │       │       │
                 USERS    ADMIN   SERVERS
```

Le **CORE-L3** assurera principalement le routage entre les VLAN.

Le **R1** représentera la frontière entre le réseau interne et le réseau externe.

Les switches d'accès seront responsables de la connexion des différents postes et serveurs.

---

# 🖥️ 3. Équipements

| Équipement    |  Quantité | Fonction                       |
| ------------- | --------: | ------------------------------ |
| Routeur Cisco |         1 | Routage vers le réseau externe |
| Switch L3     |         1 | Routage inter-VLAN             |
| Switch L2     |         3 | Accès des équipements          |
| PC            | Plusieurs | Postes utilisateurs            |
| Serveurs      |         2 | Services réseau                |

Les modèles exacts seront choisis en fonction des équipements disponibles dans Cisco Packet Tracer.

---

# 🌐 4. Plan des VLAN

| VLAN | Nom        | Réseau          | Passerelle   |
| ---: | ---------- | --------------- | ------------ |
|   10 | ADMIN      | 192.168.10.0/24 | 192.168.10.1 |
|   20 | IT         | 192.168.20.0/24 | 192.168.20.1 |
|   30 | RH         | 192.168.30.0/24 | 192.168.30.1 |
|   40 | USERS      | 192.168.40.0/24 | 192.168.40.1 |
|   50 | SERVERS    | 192.168.50.0/24 | 192.168.50.1 |
|   99 | MANAGEMENT | 192.168.99.0/24 | 192.168.99.1 |

### Principe

```text
VLAN 10 → Administration
VLAN 20 → Informatique
VLAN 30 → Ressources humaines
VLAN 40 → Utilisateurs
VLAN 50 → Serveurs
VLAN 99 → Management
```

La segmentation permet de séparer les domaines de broadcast et de préparer le contrôle des communications entre les différents services.

---

# 📡 5. Plan d'adressage

## Administration

```text
192.168.10.1   → passerelle
192.168.10.10  → PC-ADMIN01
192.168.10.11  → PC-ADMIN02
```

## Informatique

```text
192.168.20.1   → passerelle
192.168.20.10  → PC-IT01
192.168.20.11  → PC-IT02
```

## Ressources humaines

```text
192.168.30.1   → passerelle
192.168.30.10  → PC-RH01
192.168.30.11  → PC-RH02
```

## Utilisateurs

```text
192.168.40.1   → passerelle
192.168.40.10  → PC-USER01
192.168.40.11  → PC-USER02
```

## Serveurs

```text
192.168.50.1   → passerelle
192.168.50.10  → SRV01
192.168.50.11  → SRV02
```

## Management

```text
192.168.99.1   → passerelle
192.168.99.11  → SW1
192.168.99.12  → SW2
192.168.99.13  → SW3
192.168.99.254 → CORE-L3
```

Les adresses pourront être ajustées pendant l'implémentation si nécessaire.

---

# 🔧 6. Phases du projet

## Phase 1 — Conception

* définir le besoin ;
* définir les services ;
* définir les VLAN ;
* concevoir la topologie ;
* définir le plan d'adressage ;
* identifier les équipements.

**Livrable :**

```text
docs/01-contexte.md
docs/02-architecture.md
docs/03-plan-adressage.md
```

---

## Phase 2 — VLAN

Créer :

```text
VLAN 10 → ADMIN
VLAN 20 → IT
VLAN 30 → RH
VLAN 40 → USERS
VLAN 50 → SERVERS
VLAN 99 → MANAGEMENT
```

Configurer les ports Access et vérifier leur affectation.

Commande de vérification :

```text
show vlan brief
```

**Objectif :**

Les équipements appartenant au même VLAN doivent pouvoir communiquer au niveau 2.

---

## Phase 3 — Trunk 802.1Q

Configurer les liaisons entre les switches en Trunk.

```text
SW1 ───────── CORE-L3
       TRUNK
```

Les VLAN nécessaires devront être transportés sur les trunks.

Commande de vérification :

```text
show interfaces trunk
```

---

## Phase 4 — Routage inter-VLAN

Le CORE-L3 devra assurer la communication entre les différents VLAN.

Exemple :

```text
VLAN 10
   │
   ▼
CORE-L3
   │
   ▼
VLAN 50
```

Les interfaces VLAN du CORE-L3 serviront de passerelles.

Tests :

```text
ping
traceroute
show ip route
```

---

## Phase 5 — DHCP

Mettre en place l'attribution automatique des adresses IP pour les postes clients.

Les VLAN suivants pourront utiliser DHCP :

```text
VLAN 10
VLAN 20
VLAN 30
VLAN 40
```

Les serveurs et équipements réseau conserveront des adresses statiques.

Vérifications :

* adresse IP ;
* masque ;
* passerelle ;
* configuration obtenue automatiquement ;
* connectivité.

---

# 🔄 7. Redondance niveau 2

## Phase 6 — STP

Mettre en place une topologie contenant des liens redondants afin d'étudier le fonctionnement de **Spanning Tree Protocol**.

Objectifs :

* comprendre les boucles de niveau 2 ;
* identifier le Root Bridge ;
* comprendre les rôles des ports ;
* observer les ports bloqués ;
* comprendre la convergence.

Commandes :

```text
show spanning-tree
show spanning-tree vlan 10
```

---

## Phase 7 — EtherChannel

Regrouper plusieurs liens physiques entre deux équipements dans un canal logique.

Technologie étudiée :

```text
LACP
```

Objectifs :

* augmenter la capacité d'un lien logique ;
* assurer une redondance ;
* comprendre le fonctionnement d'EtherChannel.

Commande :

```text
show etherchannel summary
```

---

# 🚦 8. Routage

## Phase 8 — Routage statique

Avant d'utiliser un protocole de routage dynamique, le fonctionnement des routes statiques sera étudié.

Objectifs :

* comprendre la table de routage ;
* configurer une route statique ;
* configurer une route par défaut ;
* vérifier le chemin utilisé par les paquets.

Commandes :

```text
show ip route
ping
traceroute
```

---

## Phase 9 — OSPF

Mettre en place **OSPF** afin de remplacer progressivement le routage statique dans la partie concernée de l'architecture.

Objectifs :

* configurer OSPF ;
* établir un voisinage ;
* annoncer les réseaux ;
* observer les routes apprises ;
* comprendre la convergence ;
* observer le comportement lors d'une panne de lien.

Commandes :

```text
show ip ospf neighbor
show ip ospf
show ip route ospf
```

---

# 🌍 10. Accès externe

## Phase 10 — NAT/PAT

R1 représentera la frontière entre le réseau interne et le réseau externe.

Architecture :

```text
Réseau interne
      │
      ▼
   CORE-L3
      │
      ▼
      R1
      │
      ▼
Réseau externe
```

Objectifs :

* comprendre les adresses privées ;
* comprendre les adresses publiques ;
* comprendre NAT ;
* comprendre PAT ;
* permettre à plusieurs machines internes de partager une adresse externe.

Commandes :

```text
show ip nat translations
show ip nat statistics
```

---

# 🔐 11. Contrôle des communications

## Phase 11 — ACL

Mettre en place des **Access Control Lists** afin de contrôler les communications entre les réseaux.

Exemple de politique :

```text
USERS
  │
  ├──► SERVERS       autorisé selon les besoins
  │
  ├──► ADMIN        limité
  │
  └──► MANAGEMENT   interdit
```

Les règles seront définies avant leur configuration.

Pour chaque règle :

```text
Source
Destination
Service
Action
Justification
```

Exemple :

| Source | Destination | Action    | Justification                       |
| ------ | ----------- | --------- | ----------------------------------- |
| USERS  | SERVERS     | Autoriser | Accès aux services nécessaires      |
| USERS  | ADMIN       | Refuser   | Isolation des postes administratifs |
| USERS  | MANAGEMENT  | Refuser   | Protection des équipements          |
| IT     | MANAGEMENT  | Autoriser | Administration réseau               |

Les règles exactes pourront être adaptées à l'architecture finale.

Commande de vérification :

```text
show access-lists
```

---

# 🛡️ 12. Sécurisation des équipements

La sécurisation sera limitée aux éléments directement liés au périmètre de ce lab.

À mettre en œuvre :

* comptes locaux ;
* mot de passe privilégié ;
* accès SSH ;
* désactivation de Telnet ;
* sécurisation des lignes VTY ;
* désactivation des ports inutilisés ;
* configuration d'un VLAN de management ;
* Port Security lorsque pertinent.

Commandes utiles :

```text
show running-config
show ip ssh
show interfaces status
```

---

# 🧪 13. Tests et validation

Le projet devra comporter une véritable campagne de tests.

## Tests VLAN

Vérifier que :

* deux machines du même VLAN communiquent ;
* les ports sont correctement affectés.

## Tests inter-VLAN

Tester :

```text
ADMIN → IT
ADMIN → SERVERS
USERS → SERVERS
IT → MANAGEMENT
```

## Tests DHCP

Vérifier :

* attribution de l'adresse ;
* masque ;
* passerelle ;
* connectivité.

## Tests STP

Déconnecter un lien redondant et observer le comportement du réseau.

## Tests EtherChannel

Vérifier :

```text
show etherchannel summary
```

Puis observer le comportement lors de la perte d'un des liens physiques.

## Tests OSPF

Vérifier :

```text
show ip ospf neighbor
show ip route
```

Puis simuler une panne et observer la convergence.

## Tests NAT

Vérifier les traductions :

```text
show ip nat translations
```

## Tests ACL

Tester chaque règle d'autorisation et d'interdiction.

---

# 🔎 14. Troubleshooting

Le troubleshooting constitue une partie importante du projet.

Plusieurs pannes devront être provoquées volontairement.

### Scénario 1 — Mauvais VLAN

Symptôme :

```text
Un PC ne communique plus correctement.
```

Vérification :

```text
show vlan brief
```

---

### Scénario 2 — Trunk incorrect

Symptôme :

```text
Un VLAN fonctionne sur un switch
mais pas sur un autre.
```

Vérification :

```text
show interfaces trunk
```

---

### Scénario 3 — Mauvaise passerelle

Symptôme :

```text
Communication locale : OK
Communication inter-VLAN : KO
```

Vérifications :

```text
ipconfig
show ip interface brief
show ip route
```

---

### Scénario 4 — OSPF

Symptôme :

```text
Un réseau distant devient inaccessible.
```

Vérifications :

```text
show ip ospf neighbor
show ip route
```

---

### Scénario 5 — ACL

Symptôme :

```text
Le routage fonctionne,
mais une communication précise est bloquée.
```

Vérification :

```text
show access-lists
```

---

### Scénario 6 — EtherChannel

Symptôme :

```text
L'agrégat ne fonctionne pas correctement.
```

Vérification :

```text
show etherchannel summary
```

---

# 🧠 15. Méthode de diagnostic

Pour chaque panne :

```text
Identifier le symptôme
        ↓
Déterminer la couche concernée
        ↓
Vérifier la configuration
        ↓
Tester la connectivité
        ↓
Examiner les tables
        ↓
Identifier la cause
        ↓
Corriger
        ↓
Retester
        ↓
Documenter
```

Commandes de diagnostic principales :

```text
show ip interface brief
show interfaces
show interfaces trunk
show vlan brief
show mac address-table
show arp
show ip route
show ip ospf neighbor
show access-lists
show spanning-tree
show etherchannel summary
```

---

# 📂 16. Structure du dépôt

```text
network-enterprise/
│
├── README.md
│
├── docs/
│   ├── 01-contexte.md
│   ├── 02-architecture.md
│   ├── 03-plan-adressage.md
│   ├── 04-vlan.md
│   ├── 05-trunk.md
│   ├── 06-inter-vlan.md
│   ├── 07-dhcp.md
│   ├── 08-stp.md
│   ├── 09-etherchannel.md
│   ├── 10-routage-statique.md
│   ├── 11-ospf.md
│   ├── 12-nat.md
│   ├── 13-acl.md
│   ├── 14-securite.md
│   ├── 15-tests.md
│   └── 16-troubleshooting.md
│
├── topology/
│   └── network-enterprise.pkt
│
├── diagrams/
│   ├── topology-logique.png
│   └── topology-physique.png
│
├── configs/
│   ├── R1.txt
│   ├── CORE-L3.txt
│   ├── SW1.txt
│   ├── SW2.txt
│   └── SW3.txt
│
└── screenshots/
    ├── vlan/
    ├── trunk/
    ├── inter-vlan/
    ├── dhcp/
    ├── stp/
    ├── etherchannel/
    ├── ospf/
    ├── nat/
    ├── acl/
    └── troubleshooting/
```

---

# ✅ 17. Critères de réussite

Le Lab 2 sera considéré comme terminé lorsque :

* [ ] la topologie est fonctionnelle ;
* [ ] le plan d'adressage est documenté ;
* [ ] les VLAN sont configurés ;
* [ ] les ports Access sont correctement affectés ;
* [ ] les trunks fonctionnent ;
* [ ] le routage inter-VLAN fonctionne ;
* [ ] DHCP fonctionne ;
* [ ] STP est compris et fonctionnel ;
* [ ] EtherChannel est fonctionnel ;
* [ ] le routage statique a été testé ;
* [ ] OSPF établit correctement ses voisinages ;
* [ ] les routes OSPF sont apprises ;
* [ ] NAT/PAT fonctionne ;
* [ ] les ACL appliquent les politiques prévues ;
* [ ] les équipements sont administrables de manière sécurisée ;
* [ ] plusieurs pannes ont été provoquées ;
* [ ] les pannes ont été diagnostiquées et corrigées ;
* [ ] les configurations sont sauvegardées ;
* [ ] la documentation est complète ;
* [ ] le projet est publié sur GitHub.

---

# 💼 18. Compétences démontrées

## Réseaux

* VLAN
* Trunk 802.1Q
* Inter-VLAN Routing
* DHCP
* STP
* EtherChannel
* Routage statique
* OSPF
* NAT/PAT
* ACL

## Cisco

* Cisco IOS
* configuration de switches
* configuration de routeurs
* administration SSH
* analyse des tables réseau

## Diagnostic

* analyse de connectivité ;
* analyse ARP/MAC ;
* analyse de la table de routage ;
* diagnostic VLAN/Trunk ;
* diagnostic OSPF ;
* diagnostic ACL ;
* résolution méthodique d'incidents.

## Documentation

* conception d'architecture ;
* plan d'adressage ;
* procédures de configuration ;
* procédures de test ;
* rapports de troubleshooting ;
* gestion de versions avec Git.

---

# 🚀 19. Évolutions hors périmètre

Les technologies suivantes ne font volontairement **pas partie du Lab 2 principal** :

* IPv6 avancé ;
* Wireshark approfondi ;
* firewall ;
* VPN ;
* supervision réseau ;
* IDS/IPS ;
* analyse de sécurité réseau avancée.

Elles pourront constituer un **lab dédié à la sécurité et à l'analyse réseau** afin de permettre une étude plus approfondie de ces sujets.

---

# 📌 20. Principe du projet

Chaque technologie doit répondre à un besoin concret.

La démarche suivie est :

```text
Problème
   ↓
Besoin
   ↓
Choix de la technologie
   ↓
Configuration
   ↓
Test
   ↓
Résultat
   ↓
Documentation
```

L'objectif n'est donc pas d'accumuler des commandes Cisco.

L'objectif est de comprendre :

> **comment les équipements communiquent, pourquoi une technologie est utilisée, comment vérifier qu'elle fonctionne et comment diagnostiquer une panne lorsqu'elle ne fonctionne plus.**

---

# 🏁 Résultat attendu

À la fin du projet, le dépôt GitHub devra présenter une infrastructure réseau d'entreprise complète et reproductible, accompagnée de sa documentation, de ses configurations, de ses tests et de plusieurs scénarios de troubleshooting.

Le projet doit permettre de démontrer une capacité à passer de :

```text
Besoin
  ↓
Conception
  ↓
Configuration
  ↓
Validation
  ↓
Diagnostic
  ↓
Documentation
```

et non simplement à exécuter une liste de commandes Cisco.

