# 1. Réseaux Avancés & Équipements Actifs (Cisco CCNA)

En 2ème année, les réseaux plats d'entreprise sont proscrits. L'administrateur doit segmenter les flux, garantir la résilience physique et filtrer le trafic au plus près des équipements actifs (commutateurs et routeurs).

---

## 🔀 1. Commutation d'Entreprise : VLANs & Trunk 802.1Q

Un **VLAN** (*Virtual LAN*) isole logiquement les domaines de diffusion au niveau 2 de la couche OSI. Un broadcast émis dans le VLAN Direction ne touchera jamais les postes de la Comptabilité.

```
Postes Clients (VLAN 10) ──┐
Postes Invités (VLAN 20) ──┤ [ Switch L2 ] ═══════ Lien TRUNK ═══════> [ Routeur / Switch L3 ]
Serveurs (VLAN 30)       ──┘                      (Tag 802.1Q)              (Routage Inter-VLAN)
```

### Configuration Cisco IOS d'un Switch :
```cisco
! 1. Création des VLANs
Switch(config)# vlan 10
Switch(config-vlan)# name USERS
Switch(config-vlan)# vlan 20
Switch(config-vlan)# name SERVEURS

! 2. Affectation d'un port d'accès pour un poste client
Switch(config)# interface FastEthernet 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10

! 3. Configuration du port TRUNK vers le routeur
Switch(config)# interface GigabitEthernet 0/1
Switch(config-if)# switchport mode trunk
```

---

## 🔁 2. Protocole Spanning Tree (STP & Rapid-STP)

Pour éviter les pannes en cas de rupture de câble, les commutateurs sont câblés de manière redondante. Sans mécanisme de contrôle, cela provoque une **tempête de diffusion** (*Broadcast Storm*) qui sature les liaisons et fait tomber le réseau en quelques secondes.

* **Rôle de STP (802.1D / 802.1w RSTP)** : Élire un pont racine (*Root Bridge*), identifier les chemins redondants et **bloquer logiquement les ports en boucle**.
* En cas de rupture d'un lien actif, le port bloqué bascule automatiquement en état de transfert (*Forwarding*) en moins d'une seconde avec le Rapid STP.

---

## 🚦 3. Routage Inter-VLANs : Router-on-a-Stick vs Switch L3

Par défaut, deux VLANs ne peuvent pas communiquer. Le passage d'un VLAN à l'autre nécessite un équipement de couche 3 (Réseau).

### Méthode Router-on-a-Stick (Sous-interfaces routeur) :
```cisco
! Sous-interface pour le VLAN 10
Router(config)# interface GigabitEthernet 0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0

! Sous-interface pour le VLAN 20
Router(config)# interface GigabitEthernet 0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
```

---

## 🗺️ 4. Routage Dynamique avec OSPF (Open Shortest Path First)

Sur les réseaux multisites ou complexes, les routes statiques deviennent ingérables. Le protocole à état de liens **OSPF** calcule la route optimale via l'algorithme de Dijkstra :

```cisco
Router(config)# router ospf 1
Router(config-router)# network 192.168.10.0 0.0.0.255 area 0
Router(config-router)# network 10.0.0.0 0.0.0.3 area 0
```
*Si un lien WAN tombe, OSPF recalcule automatiquement une route de contournement en quelques millisecondes sans intervention humaine.*

---

## 🛡️ 5. Filtrage & Translation : ACLs et NAT/PAT

* **Listes de Contrôle d'Accès (ACL)** : Filtrage de paquets IP en entrée ou sortie d'interface (autoriser uniquement le port HTTPS 443 vers les serveurs, bloquer tout le reste).
* **NAT / PAT (Port Address Translation)** : Traduit l'ensemble des adresses IP privées internes RFC 1918 vers l'unique adresse IP publique du routeur de bordure pour permettre la navigation Web des collaborateurs.
