[← Retour aux TP](README.md) | [Accueil](../../README.md)

# TP 03 — Structuration de l'Annuaire & Société ASR

Fiche de synthèse de l'atelier pratique n°3 : organisation de l'arborescence d'Unités d'Organisation, gestion industrielle des identités par modèles, stratégie de groupes globaux et modélisation multi-sites.

---

## 1. Contexte & Périmètre de l'Atelier

* **Entreprise simulée** : Société ASR.
* **Machines mobilisées** :
  * Contrôleurs de domaine : `CD1` et `CD2`.
  * Serveur membre : `SRV1`.
  * Poste d'administration : `Win-CL1`.
* **Topologie physique** : Deux sites géographiques (Siège à Nantes et Agence distante).
* **Objectif global** : Construire une structure d'annuaire normalisée, déployer des modèles de comptes avec restrictions d'accès horaires et configurer les sites Active Directory.

---

## 2. Réalisations Techniques Clés

### Arborescence d'Unités d'Organisation (3 Niveaux)
Mise en place de la hiérarchie d'UO sous la racine `Société ASR` pour séparer strictement les périmètres de GPO et de délégation :
* **Niveau 1** : `Société ASR`.
* **Niveau 2 (Catégories)** : `Utilisateurs`, `Groupes`, `Postes de travail`, `Serveurs`.
* **Niveau 3 (Services métiers et typologies)** :
  * Sous `Utilisateurs` et `Groupes` : `Direction`, `Informatique`, `Comptabilité`.
  * Sous `Postes de travail` : `Clients` (héberge `Win-CL1`).
  * Sous `Serveurs` : `Contrôleurs de domaine` (héberge `CD1`, `CD2`) et `Serveurs membres` (héberge `SRV1`).

### Comptes Modèles (*Templates*) Désactivés
Création de 3 comptes modèles désactivés pour industrialiser la création de collaborateurs sans omission d'attributs :
* **Modèle Direction** : Pré-rempli avec Société = `Société ASR`, Ville = `Nantes`, CP = `44000`.
* **Modèle Informatique** : Pré-rempli avec Société = `Société ASR`, Département = `Informatique`.
* **Modèle Comptabilité** : Pré-rempli avec **restrictions strictes d'horaires de connexion (*Logon Hours*)** : ouvertures de session autorisées uniquement du lundi au samedi de 07h00 à 21h00 (verrouillage systématique la nuit et le dimanche).

### Population des Collaborateurs Réels
Création des comptes d'entreprise par duplication des modèles :
* Direction : **David Grenier** (Directeur Administratif et Financier).
* Informatique : **Isabelle Védère** (Administratrice Systèmes) et **Ivan Tard** (Technicien Support).
* Comptabilité : **Christelle Urique** (Comptable) et **Christophe Talmie** (Comptable intérimaire, configuré avec une **date d'expiration de compte obligatoire** fixée au 31 décembre).

### Stratégie de Groupes Globaux de Sécurité
* Création des groupes métiers : `G-Direction` (avec adresse e-mail `direction@domNN.fr`), `G-Comptabilité`, `G-Support technique`, `G-Informatique`.
* **Imbrication de groupes** : Intégration du groupe `G-Support technique` comme membre de `G-Informatique`.
* **Création par script PowerShell** : Automatisation de la création du groupe `G-Intérimaires` au sein de l'OU correspondante.

### Topologie de Sites & Requêtes Enregistrées
* Renommage du site par défaut en **Siege** et association au sous-réseau `192.168.0.0/16`.
* Création du site distant **Agence** et association au sous-réseau `172.16.0.0/16` (sans contrôleur local).
* Configuration d'une requête LDAP enregistrée dans la console MMC pour afficher instantanément tous les **Comptes désactivés** de l'annuaire.

---

## 3. Commandes de Validation & Contrôle

```powershell
# Vérifier l'appartenance d'un client au bon site physique Active Directory
nltest /dsgetsite

# Exporter l'inventaire complet des membres de tous les groupes d'entreprise
Get-ADGroup -Filter 'Name -like "G-*"' | ForEach-Object {
    $grp = $_.Name
    Get-ADGroupMember -Identity $grp | Select-Object @{N="Groupe";E={$grp}}, Name, SamAccountName
} | Export-Csv -Path "C:\ctrl_membres_groupes.csv" -NoTypeInformation -Encoding UTF8

# Auditer les attributs complets d'un utilisateur (plages horaires, expiration)
Get-ADUser -Identity "dgrenier" -Properties Department, City, AccountExpirationDate, LogonHours

# Rechercher tous les comptes d'utilisateurs désactivés dans l'arborescence
Search-ADAccount -AccountDisabled -UsersOnly | Select-Object Name, SamAccountName, DistinguishedName
```

---

[← TP précédent : TP 02](tp02-domaine-fsmo.md) | [Cours théorique associé : Gouvernance & UO](../cours/03-gouvernance-annuaire.md) | [TP suivant : TP 04 →](tp04-stockage-partages-agdlp.md)

