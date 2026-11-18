# 2. Architecture Windows Server & Active Directory (AD DS)

Active Directory Domain Services (AD DS) est le service d'annuaire au cœur des réseaux d'entreprise Microsoft. Il fournit un mécanisme centralisé pour authentifier les utilisateurs, appliquer les politiques de sécurité et répertorier l'ensemble des ressources (postes, serveurs, partages, imprimantes).

---

## 🏛️ 1. Les Composants Logiques d'Active Directory

```
[ Racine de Forêt : domNN.ad ]
         │
         ├── Catalogue Global (GC)
         ├── Maître de Schéma (FSMO)
         ├── Maître d'Attribution de Noms (FSMO)
         │
         ▼
[ Contrôleurs de Domaine (DC) ]
    ├── CD1 (Contrôleur Principal / DNS)
    └── CD2 (Contrôleur Réplica / Tolérance aux pannes)
```

* **La Forêt** : Limite de sécurité globale d'une organisation AD. Elle regroupe un ou plusieurs arbres de domaines partageant un schéma et un catalogue global uniques.
* **Le Domaine** : Limite administrative et unité de réplication. Exemple : `domNN.ad`.
* **L'Arborescence d'Unités d'Organisation (OU)** : Conteneurs logiques permettant de structurer les objets (utilisateurs, ordinateurs) et de leur appliquer des stratégies spécifiques (GPO) ou des délégations administratives.
* **Le Catalogue Global (GC)** : Base de données distribuée contenant une réplique partielle et en lecture seule de tous les objets de la forêt, permettant des recherches rapides sans interroger chaque domaine.

---

## ⚡ 2. Déploiement d'un Contrôleur de Domaine en PowerShell

L'installation moderne d'un contrôleur de domaine s'effectue intégralement en ligne de commande via le module `ADDSDeployment`.

### Étape 1 : Installation des binaires et outils RSAT
```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

### Étape 2 : Validation des prérequis de la forêt
```powershell
Test-ADDSForestInstallation -DomainName "domNN.ad"
```

### Étape 3 : Promotion de la racine de forêt
```powershell
Install-ADDSForest -DomainName "domNN.ad" -InstallDns:$true
```
> [!IMPORTANT]
> L'argument `-InstallDns:$true` est capital : Active Directory dépend entièrement du serveur DNS pour localiser les contrôleurs via les enregistrements de service `SRV` (notamment `_ldap._tcp.dc._msdcs.domaine`).

---

## 🛡️ 3. Tolérance aux Pannes : Le Contrôleur Réplica (CD2)

En production, un domaine ne doit **jamais** reposer sur un unique contrôleur de domaine. La panne du contrôleur unique bloque immédiatement l'authentification Kerberos, l'application des GPO et la résolution DNS.

### Procédure d'ajout du contrôleur secondaire :
1. Configurer l'adresse IP statique du second serveur (ex: `192.168.21.2/16`).
2. Définir son serveur DNS primaire sur l'IP du contrôleur existant (CD1 : `192.168.21.1`).
3. Joindre le serveur au domaine :
   ```powershell
   Add-Computer -DomainName "domNN.ad" -Restart
   ```
4. Installer le rôle et promouvoir le serveur en tant que contrôleur additionnel :
   ```powershell
   Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
   Install-ADDSDomainController -DomainName "domNN.ad" -InstallDns:$true
   ```

---

## 👑 4. Les 5 Rôles FSMO (Flexible Single Master Operations)

Active Directory fonctionne selon un modèle multimaître (tous les DC acceptent des écritures), à l'exception de **5 rôles critiques** qui ne peuvent être détenus que par un seul contrôleur à un instant T :

| Niveau | Rôle FSMO | Responsabilité | Conséquence en cas de panne prolongée |
| :--- | :--- | :--- | :--- |
| **Forêt** | **Maître de schéma** (*Schema Master*) | Contrôle les modifications de la structure de l'annuaire (ajout de classes/attributs). | Impossible d'étendre le schéma (ex: mise à jour Exchange ou Windows Server). |
| **Forêt** | **Maître d'attribution de noms** (*Domain Naming*) | Gère l'ajout ou la suppression de domaines dans la forêt. | Impossible d'ajouter ou renommer un domaine. |
| **Domaine** | **Maître RID** (*Relative ID Master*) | Distribue des pools de RID uniques aux DC pour générer les SID des nouveaux objets. | Impossible de créer de nouveaux comptes une fois le pool local du DC épuisé. |
| **Domaine** | **Émulateur PDC** (*PDC Emulator*) | Serveur de temps NTP de référence, traitement prioritaire des changements de mots de passe, compatibilité NTLM. | Dérive d'horloge sur le réseau, lenteurs de synchronisation de mots de passe. |
| **Domaine** | **Maître d'infrastructure** (*Infrastructure Master*) | Met à jour les références inter-domaines et les identifiants d'objets. | Problèmes de résolution de noms de groupes multi-domaines. |

### Vérification des rôles FSMO
```cmd
netdom query fsmo
```

### Transfert de rôles en PowerShell
Pour équilibrer la charge ou préparer une maintenance de CD1, on peut transférer certains rôles vers CD2 :
```powershell
Move-ADDirectoryServerOperationMasterRole -Identity "CD2" -OperationMasterRole RidMaster, PDCEmulator
```

---

## 💻 5. Administration Distante via RSAT

Pour respecter le principe de moindre privilège et sécuriser les serveurs, l'administrateur ne se connecte pas en RDP sur les contrôleurs de domaine pour ses tâches quotidiennes. Il utilise un poste client d'administration (ex: `Win-CL1`) doté des **Outils d'administration de serveur distant (RSAT)**.

Depuis ce poste, il pilote l'annuaire via :
* Le Centre d'administration Active Directory (**ADAC**)
* La console Utilisateurs et ordinateurs Active Directory (`dsa.msc`)
* Le module PowerShell `ActiveDirectory`
