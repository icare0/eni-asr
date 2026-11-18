# 4. Le Guide Définitif du Modèle AGDLP, Stockage Résilient & Partages SMB

La gestion des fichiers et des droits d'accès est le cœur névralgique de l'infrastructure d'entreprise. Une gestion laxiste des droits entraîne des fuites de données confidentielles, des blocages métiers et une impossibilité totale de réussir un audit de conformité.

---

## 💾 1. Stockage Résilient Logiciel : Les Espaces de Stockage (Storage Spaces)

La technologie **Espaces de stockage** (*Storage Spaces*) intégrée à Windows Server permet d'agréger des disques durs physiques hétérogènes (SAS, SATA, NVMe) au sein d'un **Pool de stockage** virtuel résilient sans dépendre d'un contrôleur RAID matériel coûteux.

```
[ Disque SCSI 1 (20 Go) ]  [ Disque SCSI 2 (20 Go) ]  [ Disque SCSI 3 (20 Go) ]
            └─────────────────────────┬─────────────────────────┘
                                      ▼
                        [ Pool de Stockage SRV1 ]
                                      │
                     ┌────────────────┴────────────────┐
                     ▼                                 ▼
         [ Disque Virtuel DATA ]           [ Disque Virtuel USERS ]
         (Résilience en Miroir)            (Résilience en Miroir)
                     │                                 │
                     ▼                                 ▼
           Volume NTFS (Lecteur E:)          Volume NTFS (Monté dans C:\Base)
```

### Types de Résilience
* **Simple (sans résilience)** : Agrégation par bandes (Striping). Tolérance à zéro panne. Débit maximal, mais perte totale des données en cas de panne d'un seul disque.
* **Miroir (Mirror)** : Les blocs de données sont dupliqués sur au moins deux disques. Excellentes performances en lecture et écriture, tolère la défaillance d'un disque physique à chaud.
* **Parité (Parity)** : Similaire au RAID 5. Les données et les bits de parité sont répartis sur un minimum de 3 disques. Idéal pour archiver de gros volumes de données au détriment des temps d'écriture.

### Point de Montage NTFS vs Lettre de Lecteur
Windows Server permet de monter un volume NTFS :
1. Soit sous une lettre classique (ex: lecteur `E:\DATA`).
2. Soit sous forme d'un **point de montage dans un dossier NTFS existant** (ex: monter le volume USERS directement dans `C:\Base`). Cette approche évite d'épuiser les 26 lettres de l'alphabet et masque la structure des disques physiques aux utilisateurs.

---

## 🛡️ 2. Partages SMB vs Permissions NTFS : La Règle du Cumul

Tout accès à un fichier à travers le réseau traverse **deux barrières de sécurité consécutives** :

```
[ Requête Réseau Utilisateur : \\SRV1\Compta ]
                     │
                     ▼
       [ 1. Autorisations de Partage SMB ]
                     │
                     ▼
       [ 2. Listes de Contrôle d'Accès (ACL) NTFS ]
                     │
                     ▼
       [ ACCÈS EFFECTIF = LE PLUS RESTRICTIF DES DEUX ]
```

> [!CAUTION]
> **La Bonne Pratique de l'Administrateur Chevronné :**  
> Au niveau du partage SMB, on accorde **Contrôle total** au groupe `Utilisateurs authentifiés`.  
> La sécurité réelle et chirurgicale est ensuite appliquée **exclusivement au niveau NTFS**. Cela évite les doublons de maintenance et garantit que les droits restent identiques, que l'utilisateur accède aux données par le réseau ou en local.

---

## 🎯 3. Le Modèle de Référence AGDLP

La méthode universelle Microsoft pour attribuer des autorisations dans un domaine s'intitule **AGDLP** :

```
┌────────────────────────────────────────────────────────┐
│                      MODÈLE AGDLP                      │
└───────────────────────────────────┬────────────────────┘
                                    │
    A   (Account)          : Comptes utilisateurs
                                    │
                                    ▼
    G   (Global Group)     : Groupes Globaux (Rôle métier)
                                    │
                                    ▼
    DL  (Domain Local)     : Groupes de Domaine Local (Autorisation)
                                    │
                                    ▼
    P   (Permission)       : Permissions NTFS / Partage / Imprimante
```

### Matrice d'Attribution AGDLP Complète (Cas Société ASR)

| Dossier Cible | Groupe de Domaine Local (DL) | Membre (Groupe Global) | Droit NTFS Effectif |
| :--- | :--- | :--- | :--- |
| `E:\DATA\Documentation` | `DL-Documentation-Lecture` | `Utilisateurs du domaine` | Lecture & Exécution |
| `E:\DATA\Documentation` | `DL-Documentation-ControleTotal` | `G-Informatique` | Contrôle Total |
| `E:\DATA\Services\Comptabilité` | `DL-Compta-Modification` | `G-Comptabilité` | Modification, Lecture, Écriture |
| `E:\DATA\Services\Comptabilité` | `DL-Compta-ControleTotal` | `G-Informatique` | Contrôle Total |
| `E:\DATA\Informatique` | `DL-Info-ControleTotal` | `G-Informatique` | Contrôle Total |

> **Règle d'Exclusion :** Les intérimaires (`G-Intérimaires`) ne doivent pas accéder à la documentation de l'entreprise. Ils font l'objet d'un refus d'accès explicite sur le dossier `Documentation`.

### Automatisation des Partages SMB en PowerShell
```powershell
# Partage Documentation publié dans l'annuaire Active Directory
New-SmbShare -Name "Documentation" `
             -Path "E:\DATA\Documentation" `
             -FullAccess "domNN\DL-Documentation-ControleTotal" `
             -ReadAccess "domNN\DL-Documentation-Lecture"

# Partage Comptabilité
New-SmbShare -Name "Compta" `
             -Path "E:\DATA\Services\Comptabilité" `
             -FullAccess "domNN\DL-Compta-ControleTotal" `
             -ChangeAccess "domNN\DL-Compta-Modification"

# Partage administratif masqué (se termine obligatoirement par un '$')
New-SmbShare -Name "Info$" `
             -Path "E:\DATA\Informatique" `
             -FullAccess "domNN\DL-Info-ControleTotal"
```

---

## 🔑 4. Délégations Administratives dans l'Annuaire

Pour décharger l'équipe informatique sans distribuer le mot de passe du compte `Administrateur`, on utilise l'assistant **Délégation de contrôle** dans `dsa.msc` :

* **David Grenier (Direction)** : Délégué sur l'OU `Société ASR/Utilisateurs/Comptabilité` pour réinitialiser les mots de passe et modifier les comptes des membres de son équipe.
* **Ivan Tard (Support Technique)** : Délégué sur l'OU racine `Société ASR/Utilisateurs` (à l'exclusion de l'OU Informatique) pour créer et gérer les utilisateurs du parc.
* **Isabelle Védère (Admin ASR)** : Membre officiel du groupe `Administrateurs du domaine`.

---

## 🖨️ 5. Gestion d'un Serveur d'Impression Réseau

La console `printmanagement.msc` sur `SRV1` permet de centraliser la distribution et l'audit des impressions :
* **Déploiement de pilotes multi-architectures** : Installation préalable des pilotes d'impression 32 bits (x86) et 64 bits (x64) pour que les postes clients téléchargent automatiquement le bon pilote lors de la connexion.
* **Restrictions Horaires** : Dans les propriétés avancées de l'imprimante partagée `Dell 5210CN`, restreindre les impressions des comptables à la plage de **20h00 à 06h00** pour ne pas saturer le matériel en journée.
* **Hiérarchie des Droits** :
  * Utilisateurs : Droit `Imprimer`.
  * Responsable de service (David Grenier) : Droit `Gérer les documents` (permet d'annuler ou réordonner les impressions des collaborateurs).
  * Informatique : Droit `Gérer les imprimantes` (Contrôle total).
