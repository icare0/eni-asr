# Atelier 04 — Gestion des Ressources & Modèle AGDLP

Cet atelier pratique couvre la résilience matérielle du stockage sur SRV1, l'application rigoureuse du modèle **AGDLP** pour les partages SMB et la gestion d'un serveur d'impression réseau.

---

## 🎯 Objectifs Opérationnels
1. Configurer un pool d'espaces de stockage résilient (*Storage Spaces*) à partir de 3 disques SCSI virtuels.
2. Formater et monter deux volumes NTFS (dont un point de montage dans un dossier NTFS).
3. Mettre en œuvre le modèle **AGDLP** pour sécuriser 3 partages de fichiers SMB.
4. Mettre en place des délégations administratives ciblées dans Active Directory.
5. Configurer une imprimante réseau partagée avec gestion des files d'attente et restrictions horaires.

---

## 💾 Étape 1 : Pool de Stockage Résilient sur SRV1

1. Dans les paramètres de la machine virtuelle `SRV1` sous VMware, ajouter à chaud **3 disques durs virtuels SCSI de 20 Go chacun**.
2. Dans le **Gestionnaire de serveur** > **Services de fichiers et de stockage** > **Pools de stockage** :
   * Créer un nouveau pool nommé `Pool-SRV1` regroupant les 3 disques physiques.
   * Créer un disque virtuel résilient en mode **Miroir** (tolère la panne d'un disque physique).
3. **Création des deux volumes NTFS** :
   * Volume 1 : Taille = **10 Go**, Système de fichiers = NTFS, Lettre de lecteur = **`E:`**, Nom = `DATA`.
   * Volume 2 : Taille = **5 Go**, Système de fichiers = NTFS, **Montage dans le dossier** `C:\Base` (aucun lettre de lecteur attribuée), Nom = `USERS`.

---

## 📂 Étape 2 : Arborescence des Dossiers & Partages SMB

Créer l'arborescence suivante sur le volume `E:\DATA` :
```
E:\DATA
├── Documentation\          -> Partage "Documentation" (Publié dans l'annuaire AD)
├── Services\
│   └── Comptabilité\      -> Partage "Compta"
└── Informatique\           -> Partage masqué "Info$"
```

---

## 🛡️ Étape 3 : Implémentation du Modèle AGDLP

### 1. Création des Groupes de Domaine Local (DL)
Dans l'OU `Société ASR/Groupes` :
* `DL-Documentation-Lecture`
* `DL-Documentation-ControleTotal`
* `DL-Compta-Modification`
* `DL-Compta-ControleTotal`
* `DL-Info-ControleTotal`

### 2. Intégration des Membres (G dans DL)
* `G-Informatique` est ajouté dans tous les groupes `DL-*-ControleTotal`.
* `G-Comptabilité` est ajouté dans `DL-Compta-Modification`.
* `Utilisateurs du domaine` est ajouté dans `DL-Documentation-Lecture` (*à l'exception de `G-Intérimaires` qui fait l'objet d'un refus d'accès*).

### 3. Commandes PowerShell de Création des Partages
```powershell
# Partage Documentation publié dans l'annuaire
New-SmbShare -Name "Documentation" `
             -Path "E:\DATA\Documentation" `
             -FullAccess "domNN\DL-Documentation-ControleTotal" `
             -ReadAccess "domNN\DL-Documentation-Lecture"

# Partage Comptabilité
New-SmbShare -Name "Compta" `
             -Path "E:\DATA\Services\Comptabilité" `
             -FullAccess "domNN\DL-Compta-ControleTotal" `
             -ChangeAccess "domNN\DL-Compta-Modification"

# Partage administratif masqué
New-SmbShare -Name "Info$" `
             -Path "E:\DATA\Informatique" `
             -FullAccess "domNN\DL-Info-ControleTotal"
```

---

## 🔑 Étape 4 : Délégations Administratives dans l'Annuaire (`dsa.msc`)

1. **David Grenier (Comptabilité)** :
   * Clic droit sur l'OU `Société ASR/Utilisateurs/Comptabilité` > **Déléguer le contrôle**.
   * Sélectionner `David Grenier`.
   * Tâches déléguées : *Réinitialiser les mots de passe des utilisateurs*, *Créer, supprimer et gérer les comptes d'utilisateurs*.
2. **Ivan Tard (Support Technique)** :
   * Clic droit sur l'OU `Société ASR/Utilisateurs` > **Déléguer le contrôle**.
   * Sélectionner le groupe `G-Support technique`.
   * Tâches déléguées : *Gestion complète des comptes d'utilisateurs* (en prenant soin d'exclure l'OU Informatique des droits hérités).
3. **Isabelle Védère (Admin Système)** :
   * Intégrée au groupe prédéfini `Administrateurs du domaine`.

---

## 🖨️ Étape 5 : Serveur d'Impression Réseau sur SRV1

1. Installer le rôle **Serveur d'impression et de numérisation de document**.
2. Dans la console `printmanagement.msc`, ajouter une nouvelle imprimante :
   * Type : Imprimante réseau TCP/IP.
   * Adresse IP du port : `192.168.21.30`.
   * Modèle : *Dell 5210CN* (Pilote Dell Open Print PCL 5).
   * Installer les pilotes supplémentaires **32 bits (x86)** et **64 bits (x64)**.
   * Nom de partage : `Dell 5210CN`, cocher **Publier dans Active Directory**.
3. **Configuration des autorisations et des horaires (Propriétés > Onglet Avancé)** :
   * `Utilisateurs du domaine` : Droit **Imprimer**.
   * `G-Comptabilité` : Impression restreinte aux heures creuses : **de 20h00 à 06h00**.
   * `David Grenier` : Droit **Gérer les documents** (permet de mettre en pause, supprimer ou redémarrer les impressions des utilisateurs de son service).
   * `G-Informatique` : Droit **Gérer les imprimantes** (Contrôle total sur l'imprimante).
