# 3. Gestion d'Annuaire, Identités & Cas « Société ASR »

La structuration rigoureuse de l'annuaire est la condition indispensable à une administration pérenne, automatisable et sécurisée. Ce chapitre illustre les bonnes pratiques à travers l'étude de cas de la **Société ASR**.

---

## 🏢 1. Architecture des Unités d'Organisation (OU)

Une Unité d'Organisation (OU) n'est pas qu'un simple dossier de rangement : elle définit **la portée d'application des stratégies de groupe (GPO)** et **les limites de délégation de contrôle**.

### Arborescence recommandée en 3 niveaux
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
        ├── Contrôleurs de domaine (CD1, CD2)
        └── Serveurs membres (SRV1)
```

> [!TIP]
> **Pourquoi séparer Utilisateurs et Ordinateurs ?**  
> Les GPO disposent de deux branches distinctes : *Configuration Ordinateur* et *Configuration Utilisateur*. Séparer ces objets dans des OU distinctes évite d'appliquer inutilement des extensions de configuration ordinateur sur des comptes utilisateurs et inversement.

---

## 👤 2. Gestion Efficace des Utilisateurs : Les Comptes Modèles

Pour garantir l'homogénéité des attributs (ville, code postal, société, horaires de connexion, script d'ouverture de session), l'administrateur crée des **comptes modèles** désactivés par service.

### Gabarits de la Société ASR
* **Modèle Direction** : Ville = Nantes, CP = 44000, Service = Direction, Société = Société ASR.
* **Modèle Informatique** : Service = Informatique.
* **Modèle Comptabilité** : Service = Comptabilité, Horaires de connexion restreints du lundi au samedi de 07h00 à 21h00.

Lors de l'arrivée d'un collaborateur (ex. *Christophe Talmie*, comptable intérimaire), l'administrateur clique droit sur le modèle correspondant > **Copier**. Il suffit de renseigner l'identité, l'identifiant, et le cas échéant une date de fin de compte.

---

## 👥 3. Groupes de Sécurité : Types et Étendues

Dans Active Directory, un groupe possède un **type** (Sécurité ou Distribution) et une **étendue** (Globale, Domaine Local, Universelle) :

| Étendue | Portée des membres | Où peut-il être utilisé pour des droits ? | Rôle dans l'architecture |
| :--- | :--- | :--- | :--- |
| **Global (G-)** | Comptes et groupes du même domaine uniquement. | Dans **n'importe quel domaine** de la forêt. | Représente le **rôle métier** d'un utilisateur (ex: `G-Comptabilite`, `G-Direction`). |
| **Domaine Local (DL-)** | Comptes de tout domaine, Groupes globaux de tout domaine. | Uniquement dans le **domaine local**. | Représente l'**autorisation sur une ressource** (ex: `DL-Compta-Modif`). |
| **Universel** | Comptes et groupes de toute la forêt. | Dans toute la forêt. | Utilisé dans les architectures multi-domaines complexes. |

### Création d'un groupe en PowerShell
```powershell
New-ADGroup -Name "G-Intérimaires" `
            -GroupScope Global `
            -GroupCategory Security `
            -Path "OU=Comptabilité,OU=Groupes,OU=Société ASR,DC=domNN,DC=ad"

Add-ADGroupMember -Identity "G-Intérimaires" -Members "Christophe Talmie"
```

---

## 🌐 4. Topologie Physique : Sites et Services (`dssite.msc`)

Tandis que les domaines et les OU représentent l'organisation **logique**, la console **Sites et services Active Directory** représente l'organisation **physique** du réseau.

### Pourquoi définir des sites physiques ?
1. **Localisation des contrôleurs** : Permet à un poste client de s'authentifier en priorité auprès du contrôleur de domaine situé dans son propre bâtiment (LAN), évitant d'encombrer le lien WAN.
2. **Contrôle de la réplication** : Définir des planifications de synchronisation inter-sites (ex: réplication toutes les 3 heures ou uniquement la nuit).

### Exemple d'implémentation
* Site principal : Renommage de `Default-First-Site-Name` en **`Siege`** (Sous-réseau `192.168.0.0/16`).
* Site distant : Création du site **`Agence`** (Sous-réseau `172.16.0.0/16`).

### Vérification du rattachement de site sur le client
```cmd
nltest /dsgetsite
```

---

## 🔍 5. Requêtes LDAP Enregistrées

Dans `dsa.msc`, les requêtes enregistrées permettent d'automatiser le filtrage des comptes selon leurs attributs LDAP.

### Trouver tous les comptes désactivés
Filtre LDAP utilisant l'opérateur bitwise matching (`1.2.840.113556.1.4.803:=2` teste le bit 2 du champ `userAccountControl`) :
```ldap
(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))
```
Permet d'auditer instantanément les comptes modèles et les comptes d'anciens collaborateurs non clôturés.
