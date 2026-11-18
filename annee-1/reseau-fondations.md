# 6. Fondations Réseau & Services IP

Un bon administrateur système doit impérativement maîtriser les mécanismes réseau sous-jacents. L'immense majorité des incidents imputés aux serveurs résultent en réalité d'un problème d'adressage, de routage ou de résolution de noms.

---

## 🌐 1. Modèles de Communication : OSI & TCP/IP

| Couche OSI | Équivalent TCP/IP | Unité de Donnée (PDU) | Rôle & Protocoles | Équipements |
| :--- | :--- | :--- | :--- | :--- |
| **7. Application**<br>**6. Présentation**<br>**5. Session** | **Application** | Données | Interaction avec les logiciels utilisateurs (HTTP, DNS, DHCP, SMB, SSH, RDP, Kerberos). | Passerelle applicative, Proxy |
| **4. Transport** | **Transport** | Segment (TCP) / Datagramme (UDP) | Communication de bout en bout, multiplexage de ports, contrôle d'erreur (TCP fiable, UDP rapide). | Pare-feu d'état |
| **3. Réseau** | **Internet** | Paquet | Adressage logique universel et routage inter-réseaux (IPv4, IPv6, ICMP, ARP). | Routeur, Switch L3 |
| **2. Liaison de données** | **Accès Réseau** | Trame | Adressage physique local (adresses MAC) et détection d'erreurs (Ethernet 802.3, Wi-Fi 802.11). | Commutateur (Switch L2) |
| **1. Physique** | **Accès Réseau** | Bit | Transmission électrique, optique ou hertzienne (câbles cuivre RJ45, fibre optique). | Câbles, Répéteur |

---

## 🔢 2. Adressage IPv4 & Découpage en Sous-Réseaux (CIDR)

### Les Plages d'Adresses Privées (RFC 1918)
Non routables sur Internet, réservées aux réseaux locaux d'entreprise :
* `10.0.0.0/8` : de `10.0.0.0` à `10.255.255.255` (16,7 millions d'hôtes)
* `172.16.0.0/12` : de `172.16.0.0` à `172.31.255.255` (1 million d'hôtes)
* `192.168.0.0/16` : de `192.168.0.0` à `192.168.255.255` (65 536 hôtes)

### Découpage VLSM (Variable Length Subnet Masking)
Pour optimiser l'utilisation des adresses et segmenter les départements :
* Adresse réseau : Tous les bits de la partie hôte à `0`.
* Adresse de diffusion (Broadcast) : Tous les bits de la partie hôte à `1`.
* Nombre d'hôtes utilisables : $2^{\text{bits hôtes}} - 2$.

---

## 📡 3. Service DHCP d'Entreprise

Le protocole DHCP (*Dynamic Host Configuration Protocol*) automatise l'attribution des paramètres réseau aux clients.

```
[ Client ]                                     [ Serveur DHCP ]
    │                                                 │
    │────── D (DHCP Discover - Broadcast) ───────────>│
    │<───── O (DHCP Offer - Proposition d'IP) ────────│
    │────── R (DHCP Request - Choix du bail) ────────>│
    │<───── A (DHCP Acknowledge - Confirmation) ──────│
```

### Éléments de configuration :
* **Étendue (Scope)** : Plage d'adresses IP distribuées (ex: `192.168.21.50` à `192.168.21.150`).
* **Options de serveur/étendue** : Masque de sous-réseau (option 003 Passerelle, option 006 Serveurs DNS, option 015 Nom de domaine DNS).
* **Réservations** : Fixer une adresse IP constante à un équipement donné en la liant à son adresse MAC physique.

---

## 🔎 4. Service DNS (Domain Name System) & Active Directory

Sans DNS fonctionnel, Active Directory cesse immédiatement d'opérer.

### Enregistrements Indispensables
* **A / AAAA** : Associe un nom d'hôte (`srv1.domNN.ad`) à une adresse IPv4 / IPv6.
* **PTR** : Résolution inverse (IP vers nom FQDN).
* **CNAME** : Alias pointant vers un autre nom d'hôte.
* **SRV (Service Location)** : Permet aux clients de localiser dynamiquement les contrôleurs de domaine, le catalogue global et les serveurs Kerberos.
  * Exemple : `_ldap._tcp.dc._msdcs.domNN.ad` (Port TCP 389).
