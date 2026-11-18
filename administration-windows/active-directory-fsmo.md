# 2. Architecture Active Directory (AD DS), Contrôleurs & Haute Disponibilité des Rôles FSMO

Ce dossier technique constitue la pierre angulaire de l'infrastructure d'entreprise Microsoft : comprendre le fonctionnement interne d'Active Directory Domain Services (AD DS), automatiser le déploiement des contrôleurs de domaine et maîtriser la répartition stratégique des 5 rôles FSMO.

---

## 🏛️ 1. Architecture Logique & Physique d'Active Directory

Active Directory fournit un annuaire unifié, hiérarchique et sécurisé permettant de gérer les identités, d'authentifier les accès et de distribuer les politiques de sécurité sur l'ensemble du réseau.

```
┌────────────────────────────────────────────────────────────────────────┐
│ RACINE DE FORÊT : domNN.ad                                            │
│ • Limite de sécurité globale du Système d'Information                  │
│ • Schéma unique partagé par tous les domaines                         │
│ • Catalogue Global (GC) répertoriant tous les objets                   │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
        ┌───────────────────────────┴───────────────────────────┐
        ▼                                                       ▼
┌───────────────────────────────┐       ┌───────────────────────────────┐
│ CONTRÔLEUR DE DOMAINE CD1     │       │ CONTRÔLEUR DE DOMAINE CD2     │
│ (Contrôleur Principal)        │ ◄═══► │ (Contrôleur Réplica)          │
│ • Rôle de Schéma              │       │ • Rôle Maître RID             │
│ • Rôle Attribution de Noms    │       │ • Rôle Émulateur PDC          │
│ • Rôle Infrastructure         │       │ • Haute disponibilité DNS     │
│ • Serveur DNS Intégré         │       │ • Tolérance aux pannes        │
└───────────────────────────────┘       └───────────────────────────────┘
```

### Les Concepts Clés
* **La Forêt** : Limite de sécurité ultime. Deux forêts ne partagent aucune donnée sans mise en place explicite d'une relation d'approbation (*Trust Relationship*).
* **Le Domaine** : Limite administrative et unité de réplication. Tous les contrôleurs d'un même domaine répliquent la totalité des objets du domaine.
* **Le Catalogue Global (GC)** : Base de données distribuée contenant une réplique partielle de tous les objets de tous les domaines de la forêt. Il est indispensable pour la validation des connexions et la résolution des appartenances aux groupes universels.
* **L'Intégration DNS** : AD DS repose à 100% sur le service DNS. Le contrôleur s'y enregistre dynamiquement grâce aux enregistrements de service `SRV` (notamment `_ldap._tcp.dc._msdcs.domaine` sur le port 389 et `_kerberos._tcp` sur le port 88).

---

## ⚡ 2. Déploiement Automatisé du Contrôleur Principal (CD1)

En production moderne, la promotion d'un contrôleur s'effectue intégralement en ligne de commande via le module PowerShell `ADDSDeployment`.

```powershell
# 1. Installation du rôle AD DS et des modules d'administration RSAT
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# 2. Validation préalable des prérequis système et réseau
Test-ADDSForestInstallation -DomainName "domNN.ad"

# 3. Promotion de la nouvelle forêt avec installation automatique du DNS intégré
Install-ADDSForest -DomainName "domNN.ad" -InstallDns:$true
```
*Le serveur redémarre automatiquement. À l'issue du redémarrage, la zone DNS intégrée à Active Directory est opérationnelle et sécurisée.*

---

## 🛡️ 3. Tolérance aux Pannes : Le Contrôleur Réplica (CD2)

Un domaine Active Directory ne doit **jamais** reposer sur un unique contrôleur de domaine. La panne matérielle de cet hôte unique paralyse instantanément les connexions utilisateurs, les partages de fichiers et la résolution de noms.

### Déploiement pas à pas du second contrôleur (CD2)
1. **Configuration IP statique** : Attribuer une adresse IP sur le même sous-réseau (ex: `192.168.21.2/16`).
2. **Pointeurs DNS** : Définir le serveur DNS primaire sur l'adresse IP du premier contrôleur `CD1` (`192.168.21.1`) afin de pouvoir résoudre le domaine `domNN.ad`.
3. **Jonction et promotion en PowerShell** :
```powershell
# Jonction de la machine au domaine
Add-Computer -DomainName "domNN.ad" -Restart

# Après redémarrage, installation des fonctionnalités AD DS
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# Promotion en contrôleur de domaine additionnel (réplica)
Install-ADDSDomainController -DomainName "domNN.ad" -InstallDns:$true
```

---

## 👑 4. Maîtrise des 5 Rôles FSMO (Flexible Single Master Operations)

Active Directory fonctionne selon un modèle multimaître (tous les DC acceptent des écritures), à l'exception de **5 rôles stratégiques** qui ne peuvent être attribués qu'à un seul serveur à un instant donné.

| Portée | Rôle FSMO | Responsabilité Opérationnelle | Conséquence en cas de panne prolongée |
| :--- | :--- | :--- | :--- |
| **Forêt** | **Maître de Schéma** (*Schema Master*) | Contrôle l'ajout de nouvelles classes d'objets ou d'attributs dans toute la forêt. | Impossible de faire évoluer le schéma (ex: mise à niveau Windows Server, déploiement Exchange). |
| **Forêt** | **Maître d'Attribution de Noms** (*Domain Naming*) | Supervise l'ajout ou la suppression de domaines dans la forêt. | Impossible d'ajouter ou de retirer un domaine enfant. |
| **Domaine** | **Maître RID** (*Relative ID Master*) | Alloue des pools de 500 identifiants relatifs (RID) à chaque DC pour composer les identifiants de sécurité uniques (SID). | Impossible de créer de nouveaux utilisateurs ou ordinateurs une fois le pool local du DC épuisé. |
| **Domaine** | **Émulateur PDC** (*PDC Emulator*) | Horloge de référence NTP (W32Time), traitement prioritaire des modifications de mot de passe, compatibilité NTLM. | Dérive d'horloge entre serveurs (rupture Kerberos), lenteurs de mise à jour des mots de passe. |
| **Domaine** | **Maître d'Infrastructure** (*Infrastructure Master*) | Met à jour les références entre objets de domaines différents. | Anomalies de résolution des noms de groupes multi-domaines. |

### Vérification des Rôles FSMO Actifs
```cmd
netdom query fsmo
```

### Procédure de Transfert de Rôles en PowerShell
Pour décharger CD1 et équilibrer l'infrastructure, on transfère les rôles **Maître RID** et **Émulateur PDC** vers le second contrôleur **CD2** :
```powershell
Move-ADDirectoryServerOperationMasterRole -Identity "CD2" `
                                        -OperationMasterRole RidMaster, PDCEmulator
```

---

## 💻 5. Administration Sécurisée via les Outils RSAT

Pour respecter le principe de moindre privilège et réduire la surface d'attaque, **un administrateur ne doit jamais ouvrir de session interactive sur les contrôleurs de domaine pour ses tâches de routine**.

Il utilise un poste client d'administration dédié (ex: `Win-CL1`) doté des **Outils d'administration de serveur distant (RSAT)** :
* Console **Centre d'administration Active Directory (ADAC)**.
* Console **Utilisateurs et ordinateurs Active Directory (`dsa.msc`)**.
* Module PowerShell `ActiveDirectory`.
