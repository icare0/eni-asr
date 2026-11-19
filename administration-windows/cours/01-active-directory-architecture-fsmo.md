[← Retour au Sommaire des Cours](README.md) | [Accueil](../../README.md)

# 1. Architecture Active Directory (AD DS) & Rôles FSMO

Ce cours présente les fondations des services d'annuaire d'entreprise sous Windows Server : comprendre le rôle central d'Active Directory Domain Services (AD DS), ses dépendances protocolaires, les mécanismes de réplication et la gouvernance des 5 rôles FSMO.

---

## 1. Pourquoi un Annuaire d'Entreprise ? (Workgroup vs Domaine)

Dans un environnement informatique sans annuaire centralisé (mode **Groupe de travail / Workgroup**) :
* Chaque ordinateur possède sa propre base de données locale d'utilisateurs (**SAM** - *Security Accounts Manager*).
* Si une entreprise compte 50 postes et 50 utilisateurs, chaque modification de mot de passe ou chaque partage de dossier oblige à recréer ou modifier les comptes sur chaque machine individuellement. Ce modèle devient ingérable au-delà de quelques machines.

**Active Directory (AD DS)** résout ce problème en introduisant une base de données d'annuaire unique, partagée et distribuée :
* **Authentification unique (SSO - *Single Sign-On*)** : Un utilisateur s'authentifie une seule fois avec son compte de domaine et accède à toutes les ressources autorisées du réseau.
* **Gestion centralisée** : Les créations de comptes, les politiques de sécurité et les droits d'accès sont définis à un seul endroit et appliqués instantanément sur l'ensemble du parc.

---

## 2. Architecture Logique d'Active Directory

Active Directory organise les ressources de l'entreprise selon une hiérarchie logique indépendante de l'implantation physique des équipements :

```
┌────────────────────────────────────────────────────────────────────────┐
│ FORÊT ACTIVE DIRECTORY (Forest)                                        │
│ • Frontière ultime de sécurité du système d'information                │
│ • Schéma unique (définition des classes d'objets et de leurs attributs)│
│ • Catalogue Global (GC) répertoriant tous les objets de la forêt      │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
         ┌──────────────────────────┴──────────────────────────┐
         ▼                                                     ▼
┌───────────────────────────────┐              ┌───────────────────────────────┐
│ DOMAINE RACINE (Root Domain)  │              │ DOMAINE ENFANT (Child Domain) │
│ • Frontière administrative    │ ◄──────────► │ • Espace de noms contigu      │
│ • Unité de réplication de base│ Approbation  │ • Base de données propre      │
│ • Politique de mot de passe   │ automatique  │ • ex: agence.entreprise.local │
│ • ex: entreprise.local        │ bidirection. │                               │
└───────────────────────────────┘              └───────────────────────────────┘
```

### Les Composants Clés
* **La Forêt (*Forest*)** : C'est le conteneur de plus haut niveau. Une forêt partage un schéma unique et un Catalogue Global commun. La forêt représente la frontière de sécurité absolue : un administrateur de la racine de la forêt a potentiellement le contrôle sur l'ensemble des domaines de cette forêt.
* **Le Domaine (*Domain*)** : C'est l'unité de partitionnement et de réplication d'Active Directory. Tous les contrôleurs d'un même domaine possèdent une copie identique de la base de données du domaine.
* **Le Catalogue Global (*Global Catalog* - GC)** : C'est un contrôleur de domaine qui stocke une réplique complète de tous les objets de son propre domaine, ainsi qu'une réplique partielle (les attributs les plus couramment recherchés) de tous les objets des autres domaines de la forêt. Il est indispensable pour l'ouverture de session et la résolution des groupes universels.
* **La Base de Données (`ntds.dit`)** : Fichier de base de données transactionnel stocké sur chaque contrôleur de domaine (par défaut dans `C:\Windows\NTDS\ntds.dit`).

---

## 3. Les Dépendances Protocolaires Vitales

Active Directory ne fonctionne pas de manière autonome : il repose sur un écosystème de protocoles standards indispensables à sa stabilité.

### 1. Le Rôle Critique du DNS (Domain Name System)
Active Directory et le DNS sont indissociables. Si le DNS tombe en panne, Active Directory cesse immédiatement de fonctionner pour les clients.
Les contrôleurs de domaine s'enregistrent dynamiquement dans la zone DNS via des **enregistrements de service (SRV)** :
* `_ldap._tcp.dc._msdcs.<domaine>` (port 389) : Permet aux ordinateurs clients et serveurs membres de localiser un contrôleur de domaine pour interroger l'annuaire.
* `_kerberos._tcp.dc._msdcs.<domaine>` (port 88) : Permet aux clients de localiser le service KDC pour demander leurs tickets d'authentification.

### 2. Le Protocole d'Authentification Kerberos
Windows Server utilise par défaut le protocole **Kerberos v5** pour authentifier les utilisateurs et les ordinateurs. Contrairement à l'ancien protocole NTLM qui envoyait des condensats de mots de passe sur le réseau, Kerberos fonctionne sur un système d'échange de tickets chiffrés délivrés par le **KDC** (*Key Distribution Center*) hébergé sur les contrôleurs de domaine.

> [!IMPORTANT]
> **Tolérance Temporelle Kerberos :**  
> Pour empêcher les attaques par rejeu de tickets, Kerberos exige que l'horloge système de tous les clients et serveurs soit synchronisée avec celle du contrôleur de domaine détenant le rôle Émulateur PDC. La dérive maximale acceptée par défaut est de **5 minutes**.

---

## 4. Haute Disponibilité & Réplication Multi-Maîtres

Dans un environnement de production, une infrastructure Active Directory ne doit **jamais** reposer sur un seul contrôleur de domaine. La panne matérielle de cet hôte unique paralyserait instantanément l'ensemble de l'entreprise (ouvertures de sessions bloquées, partages inaccessibles).

On déploie donc au minimum **deux contrôleurs de domaine** :
* **Réplication Multi-Maîtres** : La majorité des modifications (création d'un utilisateur, réinitialisation de mot de passe) peuvent être opérées sur **n'importe quel contrôleur**. La modification est ensuite répliquée automatiquement sur les autres contrôleurs.
* **Numéros de séquence USN (*Update Sequence Numbers*)** : Chaque contrôleur gère un compteur local incrémenté à chaque modification. Lors d'un cycle de réplication, deux contrôleurs comparent leurs USN pour identifier les données à synchroniser.

---

## 5. Les 5 Rôles FSMO (*Flexible Single Master Operations*)

Bien que la réplication soit multi-maîtres pour les opérations courantes, certaines actions sensibles ne peuvent tolérer aucun conflit d'écriture simultané (par exemple : créer simultanément deux domaines du même nom ou modifier la structure du schéma).  
Ces opérations spécifiques sont déléguées à un contrôleur unique à un instant donné : ce sont les **Rôles FSMO**.

```
┌────────────────────────────────────────────────────────────────────────┐
│ RÔLES À L'ÉCHELLE DE LA FORÊT (1 seul détenteur pour toute la forêt)   │
├────────────────────────────────────┬───────────────────────────────────┤
│ 1. Maître de Schéma                │ 2. Maître d'Attribution de Noms   │
│    (Schema Master)                 │    (Domain Naming Master)         │
└────────────────────────────────────┴───────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│ RÔLES À L'ÉCHELLE DU DOMAINE (1 détenteur par domaine)                 │
├───────────────────┬────────────────────┬───────────────────────────────┤
│ 3. Émulateur PDC  │ 4. Maître RID      │ 5. Maître d'Infrastructure    │
│    (PDC Emulator) │    (Relative ID)   │    (Infrastructure Master)    │
└───────────────────┴────────────────────┴───────────────────────────────┘
```

### Description Détaillée des 5 Rôles

| Rôle FSMO | Portée | Fonction technique | Conséquence en cas de panne prolongée |
| :--- | :--- | :--- | :--- |
| **Maître de Schéma** (*Schema Master*) | Forêt | Gère les modifications du schéma de l'annuaire (ajout de classes ou d'attributs). Sollicité lors de mises à niveau du système d'exploitation ou de l'installation de serveurs applicatifs (ex: Microsoft Exchange). | Faible à court terme. Les utilisateurs travaillent normalement ; seules les extensions de schéma sont bloquées. |
| **Maître d'Attribution de Noms** (*Domain Naming Master*) | Forêt | Supervise l'ajout, la suppression ou le renommage de domaines dans la forêt. | Faible à court terme. Empêche uniquement la modification de la topologie des domaines. |
| **Émulateur PDC** (*PDC Emulator*) | Domaine | **Le rôle le plus sollicité au quotidien** :<br>• Source de temps officielle (NTP) du domaine.<br>• Réception prioritaire des changements de mots de passe pour éviter les rejets injustifiés.<br>• Gestion des verrouillages de comptes de sécurité.<br>• Rétrocompatibilité avec les systèmes anciens. | **Critique et immédiate** : Risque de désynchronisation de l'heure (échecs Kerberos) et retards de prise en compte des mots de passe modifiés. |
| **Maître RID** (*RID Master*) | Domaine | Distribue des pools de 500 identifiants relatifs (RID) à chaque contrôleur de domaine afin que chaque nouvel objet (compte, groupe) reçoive un **SID** unique au monde. | Modérée. Les contrôleurs continuent de créer des objets tant que leur pool local de RID n'est pas épuisé. |
| **Maître d'Infrastructure** (*Infrastructure Master*) | Domaine | Met à jour les références croisées inter-domaines (utilisateurs d'un domaine membres d'un groupe d'un autre domaine). | Nulle dans un environnement à domaine unique. Perturbations inter-domaines en forêt complexe. |

---

## 6. Bonne Pratique de Répartition & Déplacement des Rôles

### Répartition Recommandée sur Deux Contrôleurs
Lors de la création du premier contrôleur d'un domaine (DC1), il hérite de la totalité des 5 rôles. Dès qu'un second contrôleur (DC2) est mis en service, la bonne pratique d'ingénierie consiste à répartir la charge :
* **Contrôleur 1 (DC1)** : Conserve les rôles de forêt (**Schéma**, **Attribution de noms**) et le rôle **Infrastructure**.
* **Contrôleur 2 (DC2)** : Prend en charge les rôles les plus actifs : **Émulateur PDC** et **Maître RID**.

### Transfert Gracieux vs Saisie Forcée (*Seizure*)
* **Transfert gracieux (*Transfer*)** : S'effectue lors d'une maintenance planifiée alors que les deux contrôleurs sont en ligne et communicants. L'ancien détenteur cède le rôle proprement au nouveau contrôleur.
* **Saisie forcée (*Seizure*)** : S'utilise **exclusivement en cas de sinistre matériel définitif** du serveur détenant le rôle. Si un rôle est saisi de force avec l'option `-Force`, le serveur détruit ne doit **jamais** être rebranché sur le réseau sans avoir été préalablement réinstallé à zéro.

```powershell
# Identifier les contrôleurs détenteurs des rôles FSMO
Get-ADForest | Select-Object SchemaMaster, DomainNamingMaster
Get-ADDomain | Select-Object PDCEmulator, RIDMaster, InfrastructureMaster

# Transférer gracieusement les rôles RID et PDC vers un second contrôleur
Move-ADDirectoryServerOperationMasterRole -Identity "DC2" `
                                         -OperationMasterRole RIDMaster, PDCEmulator
```

---

[← Sommaire des Cours](README.md) | [Atelier pratique associé : TP 02](../tp/tp02-domaine-fsmo.md) | [Chapitre suivant : 2. Gouvernance d'Annuaire & Identités →](02-gouvernance-annuaire-identites.md)
