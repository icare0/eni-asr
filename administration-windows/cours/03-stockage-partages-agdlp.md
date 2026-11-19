[← Retour au Sommaire des Cours](README.md) | [Accueil](../../README.md)

# 3. Stockage d'Entreprise, Partages SMB & Modèle AGDLP

Ce cours détaille les mécanismes de stockage sous Windows Server, les couches de sécurité régissant l'accès aux données partagées et la méthode d'habilitation standard de l'industrie : le modèle **AGDLP**.

---

## 1. Stockage Logiciel Résilient : Les Espaces de Stockage (*Storage Spaces*)

Dans une infrastructure moderne, la virtualisation du stockage permet d'agréger plusieurs disques durs physiques (SAS, SATA, NVMe) sans dépendre d'une carte contrôleur RAID matérielle coûteuse et propriétaire.

La technologie **Espaces de stockage** (*Storage Spaces*) de Windows Server introduit une couche d'abstraction :

```
[ Disque Physique 1 ]       [ Disque Physique 2 ]       [ Disque Physique 3 ]
(ex: 500 Go)                (ex: 500 Go)                (ex: 500 Go)
        └─────────────────────────────┬─────────────────────────────┘
                                      ▼
                        ┌───────────────────────────┐
                        │     POOL DE STOCKAGE      │
                        │    (Capacité brute 1,5 To)│
                        └─────────────┬─────────────┘
                                      │
                 ┌────────────────────┴────────────────────┐
                 ▼                                         ▼
   ┌───────────────────────────┐             ┌───────────────────────────┐
   │ DISQUE VIRTUEL « DATA »   │             │ DISQUE VIRTUEL « BACKUP » │
   │ Mode Résilience : Miroir  │             │ Mode Résilience : Parité  │
   └─────────────┬─────────────┘             └─────────────┬─────────────┘
                 ▼                                         ▼
   ┌───────────────────────────┐             ┌───────────────────────────┐
   │ Volume NTFS (Lecteur E:)  │             │ Point de montage C:\Save  │
   └───────────────────────────┘             └───────────────────────────┘
```

### Modes de Résilience Disponibles

| Mode de résilience | Nombre minimal de disques | Tolérance aux pannes | Rendement utile | Cas d'usage en entreprise |
| :--- | :--- | :--- | :--- | :--- |
| **Simple (Striping)** | 1 disque | **0 panne**. La défaillance d'un seul disque détruit l'intégralité du volume. | 100% | Données volatiles, répertoires temporaires, bancs de test sans criticité. |
| **Miroir (Mirroring)** | 2 disques (miroir double)<br>3 disques (miroir triple) | Tolère la panne simultanée d'un disque (miroir double) ou de deux disques (miroir triple). | 50% (double)<br>33% (triple) | **Recommandé pour la production** : partages de fichiers actifs, bases de données, disques systèmes. Excellente réactivité en lecture et écriture. |
| **Parité (Parity)** | 3 disques | Tolère la perte d'un disque physique (similaire au RAID 5 matériel). | 66% à 80% | Stockage de masse, archivage froid, sauvegardes à faible taux de modification. |

### Lettre de Lecteur vs Point de Montage NTFS
Pour présenter un volume aux utilisateurs, deux méthodes existent :
* **Lettre de lecteur standard** : Montage classique (ex: lecteur `E:\` ou `F:\`). Limité aux 26 lettres de l'alphabet.
* **Point de montage dans un dossier NTFS (*Mount Point*)** : Le volume est rattaché directement à un sous-dossier vide situé sur un autre volume existant (ex: monter un volume de données dans `C:\BaseDonnees`).
  * *Avantage majeur* : Évite d'épuiser les lettres de lecteurs et permet d'étendre la capacité d'un dossier sans modifier le chemin réseau partagé aux utilisateurs.

---

## 2. Sécurité d'Accès : Partages SMB vs Permissions NTFS

L'accès à un fichier à travers le réseau traverse obligatoirement **deux barrières de sécurité successives** :

```
Requête réseau d'un collaborateur (ex: \\Serveur\Comptabilite)
                        │
                        ▼
┌───────────────────────────────────────────────┐
│ 1. AUTORISATIONS DE PARTAGE SMB               │
│ • S'appliquent uniquement via le réseau       │
│ • Droits basiques : Lecture, Modifier, Total  │
└───────────────────────┬───────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────┐
│ 2. PERMISSIONS DE FICHIERS / DOSSIERS NTFS    │
│ • S'appliquent via le réseau ET en local      │
│ • Listes de contrôle d'accès fines (ACL)      │
└───────────────────────┬───────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────┐
│ ACCÈS EFFECTIF FINAL                          │
│ Règle d'or : LE PLUS RESTRICTIF DES DEUX      │
└───────────────────────────────────────────────┘
```

> [!CAUTION]
> **La Règle d'Or de l'Administrateur Système :**  
> Si vous restreignez les accès au niveau du partage SMB ET au niveau NTFS, la maintenance devient un calvaire et les conflits d'autorisations sont inévitables.  
> **La bonne pratique universelle :**  
> 1. Au niveau du partage SMB : on accorde la permission **Contrôle total** au groupe `Utilisateurs authentifiés`.  
> 2. Au niveau NTFS : on applique la totalité des restrictions précises (Lecture, Modification, Écriture).  
> Ainsi, la sécurité reste parfaitement homogène, que le collaborateur accède aux documents à travers le réseau ou directement sur le serveur.

### Partages Administratifs et Masqués
Tout nom de partage réseau se terminant par le symbole `$` (ex: `C$`, `ADMIN$`, `Direction$`) est un **partage masqué** :
* Il est invisible lors de la découverte du réseau dans l'Explorateur de fichiers (`\\Serveur`).
* Il reste accessible uniquement aux utilisateurs qui en connaissent le chemin exact et disposent des autorisations nécessaires.

---

## 3. Le Modèle d'Habilitation Standard : AGDLP

Le modèle **AGDLP** (*Accounts → Global Groups → Domain Local Groups → Permissions*) est la norme d'architecture Microsoft pour gérer les droits d'accès au sein d'un domaine Active Directory.

```
┌──────────────────────────────────────────┐
│ A  = Accounts (Comptes Utilisateurs)     │
│      Ex: Utilisateur individuel          │
└────────────────────┬─────────────────────┘
                     │  Sont membres de
                     ▼
┌──────────────────────────────────────────┐
│ G  = Global Groups (Groupes Globaux)     │
│      Portent la fonction ou le métier     │
│      Ex: G-Direction, G-Comptabilite     │
└────────────────────┬─────────────────────┘
                     │  Sont imbriqués dans
                     ▼
┌──────────────────────────────────────────┐
│ DL = Domain Local (Groupes Domaine Local)│
│      Portent l'autorisation sur l'objet   │
│      Ex: DL-Compta-Lecture, DL-Compta-Mod│
└────────────────────┬─────────────────────┘
                     │  Reçoivent directement
                     ▼
┌──────────────────────────────────────────┐
│ P  = Permissions (Droits NTFS)           │
│      Ex: Lecture, Modification           │
└──────────────────────────────────────────┘
```

### Pourquoi AGDLP est-il Incontournable ?

1. **Aucun utilisateur direct dans une liste de contrôle d'accès (ACL)** : Assigner des droits NTFS directement à des comptes individuels rend tout audit impossible. Lorsqu'un collaborateur change de service, il suffit de changer son groupe global d'appartenance pour mettre à jour instantanément tous ses accès, sans toucher à l'arborescence des fichiers.
2. **Prévention du gonflement de jeton Kerberos (*Token Bloat*)** :
   * À l'ouverture de session, Windows compile un jeton d'authentification listant l'ensemble des identifiants (SID) des groupes de l'utilisateur.
   * Si les administrateurs multiplient les groupes Domaine Local sur les comptes, le jeton dépasse la taille mémoire autorisée par les protocoles réseau, provoquant des échecs d'authentification incompréhensibles.
3. **Évolutivité inter-domaines** :
   * En cas de fusion d'entreprises ou de création de domaines multiples dans une forêt, les groupes globaux d'autres domaines peuvent être directement intégrés dans les groupes Domaine Local locaux sans avoir à recalculer les listes de droits sur des téraoctets de données.

---

[← Chapitre précédent : 2. Gouvernance d'Annuaire & Identités](02-gouvernance-annuaire-identites.md) | [Atelier pratique associé : TP 04](../tp/tp04-stockage-partages-agdlp.md) | [Chapitre suivant : 4. Stratégies de Groupe (GPO) & Sécurité PSO →](04-gpo-securite-pso.md)
