# 3. Gouvernance d'Annuaire, Unités d'Organisation & Identités d'Entreprise

Une infrastructure Active Directory sans gouvernance stricte devient en quelques mois un labyrinthe ingérable, vulnérable aux attaques par escalade de privilèges et impossible à auditer. Ce dossier technique présente la méthodologie d'organisation de référence appliquée au cas de la **Société ASR**.

---

## 🏢 1. Méthodologie de Conception des Unités d'Organisation (OU)

Une Unité d'Organisation (OU) est un conteneur d'annuaire qui répond à **deux impératifs stricts** :
1. **Définir la portée d'application des Stratégies de Groupe (GPO)**.
2. **Définir les frontières de Délégation Administrative** (permettre à un collaborateur de gérer son service sans être administrateur du domaine).

### Arborescence Hiérarchique de Référence (3 Niveaux)
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
    │   └── Clients (ex: Win-CL1)
    └── Serveurs
        ├── Contrôleurs de domaine (Contient CD1, CD2)
        └── Serveurs membres (Contient SRV1)
```

> [!IMPORTANT]
> **Pourquoi séparer physiquement les Utilisateurs et les Ordinateurs ?**  
> Une GPO possède deux branches distinctes : *Configuration Ordinateur* et *Configuration Utilisateur*. Séparer les comptes des machines évite le traitement inutile de politiques de sécurité ordinateur sur des utilisateurs, ce qui accélère le temps de démarrage et d'ouverture de session des postes.

---

## 👤 2. Gouvernance des Comptes : Les Modèles (Templates)

Pour éviter les erreurs manuelles et garantir que chaque collaborateur dispose dès son arrivée des bons attributs (société, ville, code postal, horaires de travail autorisés, script de session), on crée des **comptes modèles désactivés**.

### Spécifications des Modèles de la Société ASR
* **Modèle Direction** : Ville = `Nantes`, CP = `44000`, Société = `Société ASR`, Service = `Direction`.
* **Modèle Informatique** : Société = `Société ASR`, Service = `Informatique`.
* **Modèle Comptabilité** : Société = `Société ASR`, Service = `Comptabilité`, **Horaires de connexion restreints du lundi au samedi de 07h00 à 21h00** (verrouille toute connexion nocturne ou le dimanche).

### Création d'Utilisateurs par Duplication
Lorsqu'un nouveau collaborateur rejoint l'entreprise, l'administrateur clique droit sur le modèle du service > **Copier** :
* **David Grenier** (Direction) : Directeur Comptabilité Finances, Téléphone 504.
* **Isabelle Védère** (Informatique) : Administratrice systèmes et réseaux, Téléphone 666.
* **Ivan Tard** (Informatique) : Technicien Support technique, Téléphone 102.
* **Christelle Urique** (Comptabilité) : Comptable CDI.
* **Christophe Talmie** (Comptabilité) : Comptable intérimaire avec **date d'expiration de compte obligatoire** fixée au 31 décembre.

---

## 👥 3. Stratégie des Groupes de Sécurité & Imbrication

Dans Active Directory, un groupe possède un **type** (Sécurité ou Distribution) et une **étendue** :

| Étendue | Rôle dans l'Architecture | Où peut-il être membre ? | Où peut-il recevoir des droits ? |
| :--- | :--- | :--- | :--- |
| **Global (G-)** | Représente une **fonction ou un service métier** (ex: `G-Comptabilite`, `G-Direction`). | Dans d'autres groupes globaux, ou dans des groupes de domaine local. | Dans **n'importe quel domaine** de la forêt. |
| **Domaine Local (DL-)** | Représente une **autorisation sur une ressource** (ex: `DL-Compta-Modif`, `DL-Docs-Lecture`). | Ne peut pas être membre de groupes globaux. | Uniquement dans le **domaine local**. |
| **Universel** | Utilisé pour regrouper des identités à travers plusieurs domaines d'une même forêt. | Dans des groupes universels ou de domaine local. | Dans toute la forêt (enregistré dans le Catalogue Global). |

### Création et Imbrication en PowerShell
```powershell
# Création d'un groupe global pour les salariés intérimaires
New-ADGroup -Name "G-Intérimaires" `
            -GroupScope Global `
            -GroupCategory Security `
            -Path "OU=Comptabilité,OU=Groupes,OU=Société ASR,DC=domNN,DC=ad"

# Affectation du compte intérimaire au groupe
Add-ADGroupMember -Identity "G-Intérimaires" -Members "Christophe Talmie"
```

---

## 🌐 4. Topologie Physique : Sites et Services (`dssite.msc`)

Alors que les domaines et les Unités d'Organisation représentent la structure **logique**, la console **Sites et services Active Directory** modélise l'architecture **physique** du réseau d'entreprise.

```
[ Site Physique : SIEGE ]                        [ Site Physique : AGENCE ]
Sous-réseau : 192.168.0.0/16                     Sous-réseau : 172.16.0.0/16
Contrôleurs : CD1, CD2                           Client : Win-CL1
         │                                                │
         └══════════════ Lien WAN IP (Réplication) ═══════┘
```

### Pourquoi Définir des Sites Physiques ?
1. **Optimisation de l'Authentification** : Le poste client s'authentifie en priorité auprès du contrôleur de domaine local situé dans le même site physique, évitant de saturer les liaisons WAN distantes.
2. **Contrôle de la Réplication** : Planifier les synchronisations d'annuaire aux heures creuses (la nuit ou toutes les 3 heures) pour préserver la bande passante inter-sites.

### Commande de Contrôle sur le Poste Client
```cmd
nltest /dsgetsite
```
*Cette commande certifie à quel site physique la machine est actuellement rattachée sur la base de son adresse IP.*

---

## 🔍 5. Audit d'Annuaire & Filtres LDAP Avancés

Dans la console `dsa.msc`, les **Requêtes enregistrées** permettent de surveiller l'état de santé des comptes grâce aux filtres LDAP.

### Détection des Comptes Inactifs ou Désactivés
Pour lister immédiatement tous les comptes utilisateurs désactivés dans l'annuaire, on utilise l'opérateur bitwise matching (`1.2.840.113556.1.4.803:=2` teste le bit 2 du champ `userAccountControl`) :
```ldap
(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))
```
*Ce filtre isole en un clic les comptes modèles ainsi que les anciens collaborateurs dont les comptes n'ont pas encore été purgés.*
