# 4. Stockage, Partages SMB & Méthode AGDLP

La gestion des fichiers et des droits d'accès est l'un des piliers du métier d'administrateur. Une infrastructure de partage mal pensée devient rapidement incontrôlable, génère des fuites de données et paralyse l'audit.

---

## 💾 1. Espaces de Stockage Résilients (Storage Spaces)

Plutôt que d'utiliser des contrôleurs RAID matériels propriétaires et coûteux, Windows Server intègre la technologie logicielle **Espaces de stockage** (*Storage Spaces*).

```
[ Disque SCSI 1 ]   [ Disque SCSI 2 ]   [ Disque SCSI 3 ]
        └───────────────────┬───────────────────┘
                            ▼
                  [ Pool de Stockage ]
                            │
               ┌────────────┴────────────┐
               ▼                         ▼
      [ Disque Virtuel DATA ]   [ Disque Virtuel USERS ]
      (Résilience Miroir/Parité) (Résilience Miroir/Parité)
               │                         │
               ▼                         ▼
      Volume NTFS (Lettre E:\)   Volume NTFS (Montage C:\Base)
```

### Types de résilience disponibles :
* **Simple (sans résilience)** : Concaténation ou entrelacement (Striping). Tolérance à zéro panne, mais débit maximal.
* **Miroir (Mirror)** : Les données sont dupliquées sur 2 ou 3 disques. Excellentes performances en lecture/écriture, tolère la perte d'un disque (ou 2 disques en miroir 3 voies).
* **Parité (Parity)** : Similaire au RAID 5. Les données et les informations de parité sont réparties sur au moins 3 disques. Optimise l'espace disque utile tout en tolérant la panne d'un disque.

---

## 🛡️ 2. Partage SMB vs Sécurité NTFS : La Règle d'Or

L'accès à un fichier partagé sur le réseau traverse **deux couches de sécurité consécutives** :

```
[ Requête Client Réseau ]
           │
           ▼
  [ 1. Autorisations de Partage SMB ]
           │
           ▼
  [ 2. Listes de Contrôle d'Accès (ACL) NTFS ]
           │
           ▼
  [ Accès effectif = LE PLUS RESTRICTIF DES DEUX ]
```

### La bonne pratique de l'administrateur chevronné :
1. **Au niveau du partage SMB** : Donner le droit **Contrôle total** au groupe `Utilisateurs authentifiés` (ou au groupe `Tout le monde`).
2. **Au niveau NTFS** : Verrouiller avec une précision chirurgicale l'accès réel grâce aux listes de contrôle d'accès (ACL).
> Ainsi, la gestion des droits est centralisée au niveau NTFS sans doublon de complexité au niveau du partage.

---

## 🎯 3. Le Modèle AGDLP (Méthode de Référence Microsoft)

La règle d'or pour attribuer des autorisations dans un domaine Active Directory se résume à l'acronyme **AGDLP** :

```
┌────────────────────────────────────────────────────────┐
│                      MODÈLE AGDLP                      │
└───────────────────────────────────┬────────────────────┘
                                    │
                                    ▼
       A (Account)          : Comptes utilisateurs
                                    │
                                    ▼
       G (Global Group)     : Groupes Globaux (Rôle métier)
                                    │
                                    ▼
       DL (Domain Local)    : Groupes de Domaine Local (Autorisation)
                                    │
                                    ▼
       P (Permission)       : Permissions NTFS / Partage / Imprimante
```

### Cas Pratique : Implémentation sur la Société ASR

| Ressource & Accès | Groupe de Domaine Local (DL) | Membre (Groupe Global) | Droit NTFS effectif |
| :--- | :--- | :--- | :--- |
| **Dossier Documentation** | `DL-Documentation-Lecture` | `Utilisateurs du domaine` *(sauf G-Intérimaires)* | Lecture & Exécution |
| **Dossier Documentation** | `DL-Documentation-ControleTotal` | `G-Informatique` | Contrôle Total |
| **Dossier Comptabilité** | `DL-Compta-Modification` | `G-Comptabilité` | Modification, Lecture, Écriture |
| **Dossier Comptabilité** | `DL-Compta-ControleTotal` | `G-Informatique` | Contrôle Total |
| **Dossier Masqué Info$** | `DL-Info-ControleTotal` | `G-Informatique` | Contrôle Total |

### Commandes PowerShell de création des partages SMB
```powershell
New-SmbShare -Name "Documentation" `
             -Path "E:\DATA\Documentation" `
             -FullAccess "domNN\DL-Documentation-ControleTotal" `
             -ReadAccess "domNN\DL-Documentation-Lecture"

New-SmbShare -Name "Compta" `
             -Path "E:\DATA\Services\Comptabilité" `
             -FullAccess "domNN\DL-Compta-ControleTotal" `
             -ChangeAccess "domNN\DL-Compta-Modification"

New-SmbShare -Name "Info$" `
             -Path "E:\DATA\Informatique" `
             -FullAccess "domNN\DL-Info-ControleTotal"
```

---

## 🔑 4. Délégations Administratives dans l'Annuaire

Pour décharger le service informatique des tâches simples sans donner les pleins pouvoirs d'administrateur de domaine, on utilise l'**Assistant Délégation de contrôle** dans `dsa.msc` :

* **David Grenier (Directeur Compta)** : Délégué sur l'OU `Comptabilité` pour réinitialiser les mots de passe et modifier les comptes de ses subordonnés.
* **Ivan Tard (Support Technique)** : Délégué sur l'OU `Utilisateurs` (sauf Informatique) pour créer et gérer l'ensemble des comptes de la société.
* **Isabelle Védère (Administratrice ASR)** : Intégrée au groupe restreint `Administrateurs du domaine`.

---

## 🖨️ 5. Gestion d'un Serveur d'Impression Réseau

Un serveur membre (ex: `SRV1`) peut centraliser les imprimantes d'entreprise :
* **Pilotes multi-architectures** : Intégrer les pilotes 32 bits (x86) et 64 bits (x64) pour que les clients téléchargent automatiquement le bon pilote lors de la connexion.
* **Planification horaire** : Restreindre les gros travaux d'impression (ex. comptabilité autorisée à imprimer uniquement de 20h00 à 06h00).
* **Hiérarchisation des priorités** : Définir deux imprimantes logiques pointant sur la même imprimante physique (ex: priorité 99 pour la direction, priorité 1 pour les utilisateurs standards).
