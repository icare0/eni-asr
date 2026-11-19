[← Retour aux Cours](README.md) | [Accueil](../../README.md)

# 2. Active Directory Domain Services (AD DS) & Rôles FSMO

Ce chapitre analyse en profondeur le fonctionnement interne des services d'annuaire Active Directory Domain Services (AD DS), les mécanismes de réplication multi-maîtres et la gouvernance des 5 rôles FSMO indispensables à la stabilité de l'infrastructure d'entreprise.

---

## 1. Principes Fondamentaux d'Active Directory

Active Directory Domain Services (AD DS) est un service d'annuaire distribué basé sur le protocole standard **LDAP** (*Lightweight Directory Access Protocol*). Il centralise la gestion des identités, des machines et des politiques de sécurité au sein d'une base de données transactionnelle (`ntds.dit`).

```
┌────────────────────────────────────────────────────────────────────────┐
│ RACINE DE FORÊT (Forest Root)                                          │
│ • Frontière ultime de sécurité                                         │
│ • Schéma d'annuaire unique (définition des classes et attributs)       │
│ • Catalogue Global (GC) répertoriant tous les objets de la forêt      │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
         ┌──────────────────────────┴──────────────────────────┐
         ▼                                                     ▼
┌───────────────────────────────┐              ┌───────────────────────────────┐
│ DOMAINE AD PRINCIPAL          │              │ DOMAINE AD ENFANT             │
│ • Frontière administrative    │ ◄──────────► │ • Espace de noms hiérarchique │
│ • Unité de réplication        │ Approbation  │ • Base de données propre      │
│ • Politique de mot de passe   │ automatique  │ • Réplication de domaine      │
└───────────────────────────────┘ Bidirectionnelle └───────────────────────────┘
```

### Les Dépendances Clés d'Active Directory
* **Intégration DNS stricte** : Active Directory ne peut pas fonctionner sans DNS. Les contrôleurs de domaine s'enregistrent dans des zones DNS dédiées via des enregistrements de service (**SRV**), notamment :
  * `_ldap._tcp.dc._msdcs.<domaine>` (port 389) : Permet aux clients de localiser un contrôleur de domaine pour les requêtes d'annuaire.
  * `_kerberos._tcp.dc._msdcs.<domaine>` (port 88) : Permet aux clients de localiser le service KDC pour l'authentification et l'obtention des tickets TGT.
* **Le Catalogue Global (GC)** : Il s'agit d'un contrôleur de domaine détenant une réplique partielle en lecture seule de tous les objets de tous les domaines de la forêt. Il est requis lors de l'ouverture de session pour résoudre les appartenances aux groupes universels et pour la recherche d'objets dans l'annuaire.
* **Partages système SYSVOL et NETLOGON** : Répertoires partagés répliqués entre tous les contrôleurs de domaine hébergeant les scripts d'ouverture de session et les fichiers des stratégies de groupe (GPO).

---

## 2. Déploiement et Promotion des Contrôleurs de Domaine

La promotion d'un serveur Windows en contrôleur de domaine s'effectue via l'installation du rôle `AD-Domain-Services` suivie de l'exécution de l'assistant de configuration ou des cmdlets PowerShell spécialisées du module `ADDSDeployment`.

### Étapes Clés du Processus de Promotion
1. **Validation préalable des prérequis** :
   * Attribution d'une adresse IP statique et d'un masque de sous-réseau cohérent.
   * Configuration de la boucle locale `127.0.0.1` ou de l'IP du contrôleur comme premier serveur DNS.
   * Vérification de la synchronisation de l'horloge système (tolérance maximale Kerberos : 5 minutes).
2. **Création d'une nouvelle forêt / domaine** :
   * Création de la partition de base d'annuaire `ntds.dit`.
   * Définition du mot de passe de restauration des services d'annuaire (**DSRM** - *Directory Services Restore Mode*).
   * Configuration automatique de la zone DNS intégrée à Active Directory.
3. **Ajout d'un contrôleur de domaine supplémentaire (Tolérance aux pannes)** :
   * Un domaine de production doit impérativement disposer d'au moins **deux contrôleurs de domaine** pour garantir la continuité de service en cas de panne matérielle ou de maintenance.
   * Le second contrôleur se synchronise avec le premier par réplication multi-maîtres.

---

## 3. Mécanisme de Réplication Multi-Maîtres

Dans Active Directory, la majorité des modifications (création d'un utilisateur, mise à jour d'un mot de passe, modification d'un groupe) peuvent être opérées sur **n'importe quel contrôleur de domaine**. La modification est ensuite propagée aux autres contrôleurs via la réplication multi-maîtres.

* **Numéros de séquence de mise à jour (USN - *Update Sequence Numbers*)** : Chaque contrôleur gère son propre compteur USN incrémenté à chaque modification locale. Lors de la réplication, deux contrôleurs comparent leurs USN respectifs pour déterminer les objets à transmettre.
* **Résolution des conflits** : Si deux administrateurs modifient simultanément le même attribut d'un même objet sur deux contrôleurs distincts, Active Directory tranche en utilisant la métadonnée d'horodatage la plus récente (*Last-Writer-Wins*).

---

## 4. Les 5 Rôles FSMO (*Flexible Single Master Operations*)

Bien qu'Active Directory utilise une réplication multi-maîtres, certaines opérations critiques ne peuvent tolérer aucun conflit d'écriture simultané. Ces opérations sont confiées à un seul contrôleur à la fois, appelé **Maître d'opérations** (*FSMO*).

```
┌────────────────────────────────────────────────────────────────────────┐
│ RÔLES À L'ÉCHELLE DE LA FORÊT (1 seul détenteur pour toute la forêt)   │
├────────────────────────────────────┬───────────────────────────────────┤
│ Maître de Schéma                   │ Maître d'Attribution de Noms      │
│ (Schema Master)                    │ de Domaine (Domain Naming Master) │
└────────────────────────────────────┴───────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│ RÔLES À L'ÉCHELLE DU DOMAINE (1 détenteur par domaine)                 │
├───────────────────┬────────────────────┬───────────────────────────────┤
│ Émulateur PDC     │ Maître RID         │ Maître d'Infrastructure       │
│ (PDC Emulator)    │ (Relative ID)      │ (Infrastructure Master)       │
└───────────────────┴────────────────────┴───────────────────────────────┘
```

### Description Détaillée des 5 Rôles FSMO

| Rôle FSMO | Portée | Description technique et responsabilité | Impact en cas d'indisponibilité |
| :--- | :--- | :--- | :--- |
| **Maître de Schéma** (*Schema Master*) | Forêt | Gère en écriture la structure de l'annuaire (ajout ou modification de classes d'objets et d'attributs). Requis lors des montées de version d'OS ou de l'installation d'Exchange Server. | Faible impact immédiat. Les opérations courantes continuent ; seules les modifications de schéma sont impossibles. |
| **Maître d'Attribution de Noms** (*Domain Naming Master*) | Forêt | Contrôle l'ajout, la suppression ou le renommage de domaines et de partitions d'annuaire au sein de la forêt. | Faible impact immédiat. Empêche la création ou la suppression d'un domaine enfant. |
| **Émulateur PDC** (*PDC Emulator*) | Domaine | Rôle le plus sollicité :<br>• Source de temps officielle de référence (NTP) du domaine.<br>• Récepteur prioritaire des modifications et échecs de mot de passe (vérification immédiate avant rejet).<br>• Rétrocompatibilité avec les systèmes NT4 et gestion des verrouillages de comptes. | **Impact critique immédiat** : Dérive de l'horloge système (échec Kerberos), retards dans la réplication des mots de passe, désynchronisation des verrouillages. |
| **Maître RID** (*RID Master*) | Domaine | Alloue des pools de 500 identifiants relatifs (RID) successifs à chaque contrôleur de domaine pour générer des identifiants de sécurité uniques (**SID** = `SID Domaine` + `RID`). | Impact modéré à court terme. Les contrôleurs continuent de créer des objets jusqu'à épuisement de leur réserve locale de RID. |
| **Maître d'Infrastructure** (*Infrastructure Master*) | Domaine | Met à jour les références croisées inter-domaines (lorsqu'un utilisateur d'un domaine A est membre d'un groupe d'un domaine B). Ne doit pas cohabiter avec le Catalogue Global dans un environnement multi-domaines sans GC sur tous les DC. | Aucun impact dans un environnement à domaine unique. Problèmes de résolution de noms inter-domaines en environnement multi-domaines. |

---

## 5. Stratégie de Répartition et Transfert des Rôles

### Règle de Répartition sur Deux Contrôleurs
Lors de la création de la racine du domaine, le premier contrôleur (ex: `CD1`) hérite de la totalité des 5 rôles. Dès qu'un second contrôleur (ex: `CD2`) est promu, il est fortement recommandé de séparer les charges :

* **Contrôleur 1 (CD1)** : Héberge les rôles de forêt (**Schéma**, **Attribution de noms**) et le rôle **Infrastructure**.
* **Contrôleur 2 (CD2)** : Héberge les rôles les plus sollicités au quotidien : **Émulateur PDC** et **Maître RID**.

### Transfert vs Saisie Forcée (*Seizure*)

* **Transfert gracieux (*Transfer*)** : S'exécute lorsque les deux contrôleurs de domaine sont en ligne et communicants. L'ancien détenteur cède le rôle proprement au nouveau détenteur sans risque de perte d'intégrité.
* **Saisie forcée (*Seizure*)** : S'utilise **uniquement** si le contrôleur hébergeant le rôle a subi un sinistre matériel irréversible et ne redémarrera jamais. Si un rôle est saisi de force, l'ancien contrôleur ne doit **jamais** être rebranché sur le réseau sans réinstallation complète.

```powershell
# Identifier les contrôleurs détenteurs des 5 rôles FSMO
Get-ADForest | Select-Object SchemaMaster, DomainNamingMaster
Get-ADDomain | Select-Object PDCEmulator, RIDMaster, InfrastructureMaster

# Transférer gracieusement les rôles RID et PDC vers CD2
Move-ADDirectoryServerOperationMasterRole -Identity "CD2" `
                                         -OperationMasterRole RIDMaster, PDCEmulator

# Saisir de force un rôle en cas de crash irréversible (Option -Force)
# Move-ADDirectoryServerOperationMasterRole -Identity "CD2" -OperationMasterRole PDCEmulator -Force
```

---

[← Chapitre précédent : 1. Métrologie & Supervision](01-metrologie-supervision.md) | [Atelier pratique associé : TP 02](../tp/tp02-domaine-fsmo.md) | [Chapitre suivant : 3. Gouvernance d'Annuaire & UO →](03-gouvernance-annuaire.md)

