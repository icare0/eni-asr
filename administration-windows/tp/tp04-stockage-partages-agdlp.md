[← Retour aux TP](README.md) | [Accueil](../../README.md)

# TP 04 — Espaces de Stockage Résilients & Modèle AGDLP

Fiche de synthèse de l'atelier pratique n°4 : agrégation de disques physiques en pool Storage Spaces résilient, sécurisation des partages SMB par le modèle AGDLP, délégations d'annuaire et gestion d'un serveur d'impression.

---

## 1. Contexte & Périmètre de l'Atelier

* **Machines mobilisées** :
  * `SRV1` : Serveur membre de fichiers et d'impression (`192.168.21.10/16`).
  * `CD1` et `CD2` : Contrôleurs de domaine `domNN.ad`.
  * `Win-CL1` : Poste client de test et de validation (`192.168.21.20/16`).
* **Périphérique d'impression** : Imprimante réseau Dell 5210CN (`192.168.21.30/16`, pilote Dell Open Print PCL 5).
* **Objectif global** : Construire une infrastructure de stockage logiciel tolérante aux pannes, mettre en œuvre la matrice d'habilitations AGDLP sans faille et déléguer des privilèges administratifs ciblés.

---

## 2. Réalisations Techniques Clés

### Pool de Stockage Résilient (*Storage Spaces*) sur SRV1
* Ajout de **3 disques virtuels SCSI de 20 Go** sur le serveur `SRV1`.
* Agrégation des 3 disques au sein d'un **Pool de stockage** (*Storage Pool*) unifié.
* Création de deux disques virtuels en mode de **résilience Miroir (*Mirror*)** tolérant la panne d'un disque physique à chaud :
  * **Disque Virtuel DATA** : Capacité 10 Go, formaté en NTFS, présenté sous la lettre de lecteur `E:\`.
  * **Disque Virtuel USERS** : Capacité 5 Go, formaté en NTFS, rattaché via un **point de montage dans un dossier NTFS** (`C:\Base`) sans consommer de lettre de lecteur.

### Arborescence de Partages SMB & Modèle AGDLP
Création de trois répertoires partagés sous `E:\DATA` respectant strictement la norme **AGDLP** (Comptes → Groupes Globaux → Groupes Domaine Local → Droits NTFS) :

| Partage SMB | Chemin local | Caractéristiques du partage | Groupes Domaine Local (DL) associés | Accès effectif accordé |
| :--- | :--- | :--- | :--- | :--- |
| **Documentation** | `E:\DATA\Documentation` | Partage public publié dans l'annuaire Active Directory | `DL-Documentation-Lecture`<br>`DL-Documentation-ControleTotal` | Lecture pour l'ensemble des utilisateurs sauf intérimaires ; Contrôle total pour l'Informatique. |
| **Compta** | `E:\DATA\Compta` | Partage métier restreint | `DL-Compta-Modification`<br>`DL-Compta-ControleTotal` | Modification pour la Comptabilité ; Contrôle total pour l'Informatique. |
| **Info$** | `E:\DATA\Info$` | Partage administratif masqué (invisible en énumération) | `DL-Info-ControleTotal` | Contrôle total exclusif pour le groupe `G-Informatique`. |

### Délégations Administratives dans l'Annuaire
Application du principe de moindre privilège sans conférer inutilement les droits d'Administrateur du domaine :
* **David Grenier** : Délégation du contrôle total sur l'OU `Comptabilité` (gestion des comptes, réinitialisation de mots de passe, modification des groupes du service).
* **Ivan Tard** (`G-Support technique`) : Délégation des opérations d'assistance (réinitialisation de mot de passe, déverrouillage de compte) sur l'ensemble de l'OU `Utilisateurs`, avec exclusion explicite de l'OU `Informatique`.
* **Isabelle Védère** : Intégration au groupe privilégié des `Administrateurs du domaine`.

### Serveur d'Impression Réseau d'Entreprise
* Installation du rôle Serveur d'impression sur `SRV1` et création du port standard TCP/IP `192.168.21.30`.
* Installation des pilotes d'impression Dell PCL 5 en versions 64 bits et 32 bits pour garantir l'interopérabilité multi-clients.
* Publication de l'imprimante dans Active Directory pour découverte automatique par les clients.
* **Restrictions d'utilisation** : Autorisation d'imprimer restreinte de 20h00 à 06h00 pour l'équipe Comptabilité (protection des consommables).
* **Délégation de gestion** : Affectation des autorisations de gestion des documents de la file d'attente à David Grenier.

---

## 3. Commandes de Validation & Contrôle

```powershell
# Vérifier l'état de santé du pool de stockage et des disques physiques sur SRV1
Get-StoragePool -FriendlyName "*Pool*" | Select-Object FriendlyName, OperationalStatus, HealthStatus
Get-PhysicalDisk | Select-Object FriendlyName, MediaType, Size, OperationalStatus

# Contrôler les volumes virtuels créés et leur mode de résilience
Get-VirtualDisk | Select-Object FriendlyName, ResiliencySettingName, NumberOfDataCopies, OperationalStatus

# Lister les partages SMB actifs sur le serveur de fichiers
Get-SmbShare | Where-Object { -not $_.Name.EndsWith('$') -or $_.Name -eq 'Info$' }

# Auditer les permissions NTFS effectives sur le dossier Compta
Get-Acl -Path "E:\DATA\Compta" | Select-Object -ExpandProperty Access | 
    Select-Object IdentityReference, FileSystemRights, AccessControlType

# Tester la connexion au partage masqué depuis le client d'administration
Test-Path "\\SRV1\Info$"
```

---

[← TP précédent : TP 03](tp03-annuaire-societe-asr.md) | [Cours théorique associé : Stockage & AGDLP](../cours/04-stockage-agdlp.md) | [TP suivant : TP 05 →](tp05-gpo-securite-pso.md)

