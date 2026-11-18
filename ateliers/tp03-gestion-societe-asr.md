# Atelier 03 — Gestion de l’Annuaire Active Directory (Société ASR)

Cet atelier met en œuvre les bonnes pratiques d'organisation logique et physique de l'annuaire d'entreprise pour le cas d'étude **Société ASR**.

---

## 🎯 Objectifs Opérationnels
1. Créer une arborescence d'Unités d'Organisation (OU) hiérarchique en 3 niveaux.
2. Définir des comptes modèles désactivés avec contraintes d'horaires et attributs d'entreprise.
3. Créer les comptes utilisateurs réels par duplication des modèles.
4. Structurer les groupes globaux de sécurité et tester l'imbrication de groupes.
5. Créer une requête LDAP enregistrée pour auditer les comptes inactifs.
6. Modéliser la topologie des sites physiques et des sous-réseaux sous `dssite.msc`.

---

## 🏢 Étape 1 : Architecture des Unités d'Organisation (3 Niveaux)

Déployer l'arborescence suivante sous la racine `DC=domNN,DC=ad` :

```
domNN.ad
└── Société ASR
    ├── Utilisateurs
    │   ├── Direction
    │   ├── Informatique
    │   └── Comptabilité
    ├── Groupes
    │   ├── Direction
    │   ├── Informatique
    │   └── Comptabilité
    ├── Postes de travail
    │   └── Clients (Contient Win-CL1)
    └── Serveurs
        ├── Contrôleurs de domaine (Contient CD1, CD2)
        └── Serveurs membres (Contient SRV1)
```

---

## 👤 Étape 2 : Comptes Modèles & Comptes Utilisateurs

### 1. Création des 3 Comptes Modèles (Désactivés)
* **Modèle Direction** : Ville = `Nantes`, Code postal = `44000`, Service = `Direction`, Société = `Société ASR`.
* **Modèle Informatique** : Service = `Informatique`, Société = `Société ASR`.
* **Modèle Comptabilité** : Service = `Comptabilité`, Horaires de connexion restreints du **lundi au samedi de 07h00 à 21h00**.

### 2. Création des Utilisateurs par Copie
Depuis `dsa.msc`, clic droit sur le modèle > **Copier** :

| Nom complet | Identifiant | Service | Fonction | Téléphone | Remarques |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **David Grenier** | `dgrenier` | Direction | Directeur Comptabilité Finances | `504` | Chef de service |
| **Isabelle Védère** | `ivedere` | Informatique | Administratrice systèmes et réseaux | `666` | Admin ASR |
| **Ivan Tard** | `itard` | Informatique | Support technique | `102` | Technicien support |
| **Christelle Urique** | `curique` | Comptabilité | Comptable | `201` | Salariée CDI |
| **Christophe Talmie** | `ctalmie` | Comptabilité | Comptable intérimaire | `202` | **Fin de compte au 31/12** |

---

## 👥 Étape 3 : Groupes Globaux & Imbrication

1. **Création graphique dans `dsa.msc`** :
   * `G-Direction` : Membres = *David Grenier*, *Modèle Direction* (Email: `direction@domNN.fr`).
   * `G-Support technique` : Membres = *Ivan Tard*.
   * `G-Informatique` : Membres = *Isabelle Védère*, *Modèle Informatique*, **groupe `G-Support technique` imbriqué**.
   * `G-Comptabilité` : Membres = *Christelle Urique*, *Christophe Talmie*.

2. **Création du groupe des intérimaires en PowerShell** :
   ```powershell
   New-ADGroup -Name "G-Intérimaires" `
               -GroupScope Global `
               -GroupCategory Security `
               -Path "OU=Comptabilité,OU=Groupes,OU=Société ASR,DC=domNN,DC=ad"

   Add-ADGroupMember -Identity "G-Intérimaires" -Members "Christophe Talmie"
   ```

---

## 🔍 Étape 4 : Requête Enregistrée des Comptes Désactivés

1. Dans la console `dsa.msc`, clic droit sur **Requêtes enregistrées** > **Nouveau** > **Requête**.
2. Nom : `Comptes désactivés`.
3. Racine de recherche : `OU=Société ASR,DC=domNN,DC=ad`.
4. Définir une requête personnalisée avec le filtre LDAP bitwise :
   ```ldap
   (&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))
   ```
5. Valider : la requête remonte immédiatement les 3 comptes modèles ainsi que les comptes inactifs.

---

## 🌐 Étape 5 : Topologie des Sites Active Directory (`dssite.msc`)

1. Ouvrir la console **Sites et services Active Directory** (`dssite.msc`).
2. Renommer le site par défaut `Default-First-Site-Name` en **`Siege`**.
3. Clic droit sur **Sites** > **Nouveau site** > Nommer le site **`Agence`** (sélectionner le lien de site `DEFAULTIPSITELINK`).
4. Dans le dossier **Subnets** (Sous-réseaux), créer un nouveau sous-réseau IPv4 :
   * Préfixe : `172.16.0.0/16`.
   * Associer ce sous-réseau au site **`Agence`**.
5. **Vérification sur le poste client Win-CL1** :
   ```cmd
   nltest /dsgetsite
   ```
   *Renvoie le site physique `Siege` auquel la machine est actuellement rattachée.*

---

## 📋 Étape 6 : Scripts d'Export et Contrôle (Livrables)

```powershell
# 1. Export des membres de tous les groupes globaux de l'annuaire
Get-ADGroup -Filter 'Name -like "G-*"' | ForEach-Object {
    [PSCustomObject]@{
        Groupe  = $_.Name
        Membres = (Get-ADGroupMember -Identity $_.DistinguishedName | Select-Object -ExpandProperty Name) -join ", "
    }
} | Format-Table -AutoSize | Out-File "C:\livrables\ctrl_membres_groupes.txt"

# 2. Export des attributs complets du compte David Grenier
Get-ADUser -Identity "David Grenier" -Properties * | Out-File "C:\livrables\detail_user_David.txt"
```
