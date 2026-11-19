[← Retour aux Cours](README.md) | [Accueil](../../README.md)

# 4. Espaces de Stockage Résilients, Partages SMB & Modèle AGDLP

Ce chapitre détaille la mise en œuvre du stockage logiciel sous Windows Server, les mécanismes de sécurité combinant partages réseau SMB et listes de contrôle d'accès NTFS, ainsi que l'application rigoureuse du modèle d'habilitation **AGDLP**.

---

## 1. Espaces de Stockage Logiciels (*Storage Spaces*)

La fonctionnalité **Espaces de stockage** (*Storage Spaces*) permet d'agréger plusieurs disques physiques (SCSI, SAS, SATA, NVMe) au sein d'un ou plusieurs **Pools de stockage** (*Storage Pools*) virtuels sans recourir à un contrôleur RAID matériel propriétaire.

```
[ Disque Physique 1 ]       [ Disque Physique 2 ]       [ Disque Physique 3 ]
(ex: 20 Go SCSI)            (ex: 20 Go SCSI)            (ex: 20 Go SCSI)
        └─────────────────────────────┬─────────────────────────────┘
                                      ▼
                        ┌───────────────────────────┐
                        │     POOL DE STOCKAGE      │
                        │    (Capacité brute 60 Go) │
                        └─────────────┬─────────────┘
                                      │
                 ┌────────────────────┴────────────────────┐
                 ▼                                         ▼
   ┌───────────────────────────┐             ┌───────────────────────────┐
   │ DISQUE VIRTUEL « DATA »   │             │ DISQUE VIRTUEL « USERS »  │
   │ Résilience Miroir (Mirror)│             │ Résilience Miroir (Mirror)│
   └─────────────┬─────────────┘             └─────────────┬─────────────┘
                 ▼                                         ▼
   ┌───────────────────────────┐             ┌───────────────────────────┐
   │ Volume NTFS Lecteur E:    │             │ Point de montage C:\Base  │
   └───────────────────────────┘             └───────────────────────────┘
```

### Types de Résilience Disponibles

| Mode de résilience | Nombre minimal de disques | Tolérance aux pannes | Rendement capacité | Cas d'usage recommandé |
| :--- | :--- | :--- | :--- | :--- |
| **Simple (Striping)** | 1 disque | **0 panne**. La perte d'un seul disque détruit l'ensemble des données. | 100% de la capacité totale. | Données temporaires, caches de calcul, bancs d'essai non critiques. |
| **Miroir (Mirroring)** | 2 disques (miroir double)<br>3 disques (miroir triple) | Tolère la panne de 1 disque (miroir double) ou 2 disques (miroir triple). | 50% (miroir double)<br>33% (miroir triple) | Données d'entreprise critiques, partages de fichiers actifs, bases de données, disques systèmes. |
| **Parité (Parity)** | 3 disques | Tolère la perte d'un disque physique (similaire au RAID 5). | ~66% à 80% selon le nombre de disques. | Archivage de données froides, sauvegardes, volumétries importantes à faible taux d'écriture. |

### Lettres de Lecteur vs Points de Montage NTFS

Windows Server permet de présenter un volume formaté en NTFS de deux manières :
* **Lettre de lecteur standard** : Montage classique (ex: lecteur `E:\`). Approche simple mais limitée aux 26 lettres de l'alphabet latin.
* **Point de montage dans un dossier NTFS (*Mount Point*)** : Le volume est monté directement au sein d'un dossier vide existant sur un autre volume (ex: monter le volume virtuel `USERS` dans le dossier `C:\Base`).
  * *Avantages* : Dépasse la contrainte des 26 lettres, masque l'organisation physique du stockage aux utilisateurs et permet d'étendre la capacité d'un répertoire sans modifier son arborescence de partage.

---

## 2. Sécurité d'Accès : Partages SMB vs Permissions NTFS

L'accès à un dossier partagé à travers le réseau met en jeu **deux couches de sécurité consécutives** :

```
Requête réseau client (ex: \\SRV1\Compta)
              │
              ▼
┌───────────────────────────────────────────────┐
│ 1. AUTORISATIONS DE PARTAGE SMB               │
│ • S'appliquent uniquement via le réseau       │
│ • Droits : Lecture, Modifier, Contrôle total   │
└───────────────────────┬───────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────┐
│ 2. PERMISSIONS DE FICHIERS / DOSSIERS NTFS    │
│ • S'appliquent via le réseau ET en local      │
│ • Listes de contrôle d'accès granulaires (ACL)│
└───────────────────────┬───────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────┐
│ ACCÈS EFFECTIF FINAL                          │
│ Règle : LE PLUS RESTRICTIF DES DEUX L'EMPORTE │
└───────────────────────────────────────────────┘
```

### La Règle d'Or de l'Administrateur Système
Pour simplifier l'exploitation et éliminer les conflits d'autorisations :
1. **Au niveau du partage SMB** : On accorde la permission **Contrôle total** (ou *Modifier*) au groupe `Utilisateurs authentifiés` (ou au groupe du service concerné).
2. **Au niveau NTFS** : On gère l'intégralité de la granularité fine des droits (Lecture, Écriture, Modification, Suppression, Héritage).
3. **Résultat** : Les droits restent parfaitement homogènes, que le collaborateur accède aux fichiers via un partage réseau SMB ou directement lors d'une session locale/RDP sur le serveur.

### Partages SMB Administratifs et Masqués
Tout nom de partage réseau se terminant par le caractère `$` (ex: `Info$`, `C$`, `ADMIN$`) constitue un **partage masqué** :
* Il est invisible lors de l'énumération réseau dans l'Explorateur de fichiers (`\\SRV1`).
* Il reste accessible uniquement aux utilisateurs qui en saisissent explicitement le chemin réseau exact dans la barre d'adresse.

---

## 3. Le Modèle d'Habilitation AGDLP

Le modèle **AGDLP** (*Accounts → Global Groups → Domain Local Groups → Permissions*) est la norme d'architecture Microsoft pour l'affectation des droits d'accès dans un domaine Active Directory.

```
┌──────────────────────────────────────────┐
│ A  = Accounts (Comptes Utilisateurs)     │
│      Ex: David Grenier, Ivan Tard        │
└────────────────────┬─────────────────────┘
                     │  Sont membres de
                     ▼
┌──────────────────────────────────────────┐
│ G  = Global Groups (Groupes Globaux)     │
│      Portent le rôle métier / la fonction│
│      Ex: G-Direction, G-Informatique     │
└────────────────────┬─────────────────────┘
                     │  Sont imbriqués dans
                     ▼
┌──────────────────────────────────────────┐
│ DL = Domain Local (Groupes Domaine Local)│
│      Portent le droit sur la ressource   │
│      Ex: DL-Compta-Lecture, DL-Compta-Mod│
└────────────────────┬─────────────────────┘
                     │  Reçoivent directement
                     ▼
┌──────────────────────────────────────────┐
│ P  = Permissions (Droits NTFS)           │
│      Ex: Lecture & Exécution, Modifier   │
└──────────────────────────────────────────┘
```

### Pourquoi le Modèle AGDLP est-il Incontournable ?

1. **Aucun utilisateur directement dans une ACL** : Affecter des droits NTFS directement à des comptes utilisateurs individuels rend l'audit impossible et brise la traçabilité lors des départs ou mutations.
2. **Limitation de la taille des jetons Kerberos (*Token Bloat*)** :
   * Chaque fois qu'un utilisateur ouvre une session, Windows génère un jeton d'accès contenant son SID et les SID de tous ses groupes.
   * Si des groupes Domaine Local étaient multipliés sur les utilisateurs, le jeton dépasserait la taille maximale autorisée par HTTP et RPC (générant des erreurs d'authentification réseau incompréhensibles).
3. **Évolutivité inter-domaines** :
   * Si une entreprise crée une forêt multi-domaines ou réalise une fusion, les groupes globaux d'autres domaines peuvent être directement ajoutés dans les groupes Domaine Local locaux sans recalculer l'ensemble des ACL NTFS sur les téraoctets de données du serveur.

---

[← Chapitre précédent : 3. Gouvernance d'Annuaire & UO](03-gouvernance-annuaire.md) | [Atelier pratique associé : TP 04](../tp/tp04-stockage-partages-agdlp.md) | [Chapitre suivant : 5. Stratégies GPO & Sécurité PSO →](05-gpo-securite-pso.md)

