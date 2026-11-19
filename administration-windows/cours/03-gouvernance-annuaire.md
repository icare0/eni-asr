[← Retour aux Cours](README.md) | [Accueil](../../README.md)

# 3. Gouvernance d'Annuaire, Unités d'Organisation & Identités

Ce chapitre développe les règles d'architecture nécessaires pour concevoir une arborescence Active Directory pérenne, industrialiser la création des comptes utilisateurs par modèles et maîtriser la topologie des sites physiques et des groupes de sécurité.

---

## 1. Méthodologie de Conception des Unités d'Organisation (UO)

Une Unité d'Organisation (*Organizational Unit* - OU) est un conteneur logique d'annuaire répondant à **deux objectifs techniques stricts** :

1. **Délimiter la portée d'application des Stratégies de Groupe (GPO)** : Une GPO ne peut être liée qu'à la racine du domaine, à un site physique ou à une UO (elle ne peut pas être liée aux conteneurs par défaut `CN=Users` ou `CN=Computers`).
2. **Définir les frontières de délégation administrative** : Confier la gestion opérationnelle d'un périmètre d'objets (ex: réinitialiser les mots de passe, ajouter des membres à un groupe) à un collaborateur ou une équipe support sans lui attribuer les droits d'administration du domaine.

### Conteneurs par Défaut vs Unités d'Organisation

| Caractéristique | Conteneurs par défaut (`CN=Users`, `CN=Computers`) | Unités d'Organisation (`OU=Direction,OU=Utilisateurs...`) |
| :--- | :--- | :--- |
| **Liaison de GPO** | **Impossible**. Les conteneurs `CN=` ne supportent aucun lien de stratégie de groupe. | **Native**. Chaque UO peut hériter ou posséder ses propres liaisons de GPO. |
| **Délégation de contrôle** | Complexe et déconseillée. | **Simple**. L'assistant de délégation permet d'assigner des droits ciblés avec précision. |
| **Création de sous-niveaux** | Impossible (structure plate). | **Arborescente**. Supporte une hiérarchie imbriquée à plusieurs niveaux. |

### Architecture Hiérarchique Recommandée (Modèle à 3 Niveaux)

La conception d'une arborescence d'UO doit refléter les besoins de gestion technique et non l'organigramme RH fluctuant de l'entreprise :

```
domaine.ad
└── Société / Entreprise (Racine d'organisation)
    ├── Catégorie : Utilisateurs
    │   ├── Service : Direction
    │   ├── Service : Informatique
    │   └── Service : Comptabilité
    ├── Catégorie : Groupes
    │   ├── Groupes de Sécurité Métiers
    │   └── Groupes de Ressources
    ├── Catégorie : Postes de travail
    │   ├── Ordinateurs Portables
    │   └── Ordinateurs Fixes
    └── Catégorie : Serveurs
        ├── Contrôleurs de domaine
        └── Serveurs membres
```

> [!IMPORTANT]
> **Pourquoi séparer physiquement les Utilisateurs et les Ordinateurs ?**  
> Une GPO contient deux branches : *Configuration Ordinateur* et *Configuration Utilisateur*. Regrouper les comptes d'ordinateurs dans une UO dédiée évite d'évaluer inutilement des politiques machine sur des comptes d'utilisateurs, optimisant ainsi le temps d'ouverture de session et réduisant les risques d'effets de bord.

---

## 2. Gestion et Cycle de Vie des Identités

### Utilisation de Comptes Modèles (*Templates*)

Afin de garantir l'homogénéité des attributs d'annuaire et d'éviter les omissions manuelles, l'administrateur crée des **comptes modèles** spécifiques à chaque métier.

* **État du compte modèle** : Le compte modèle doit impérativement être **désactivé** pour interdire toute ouverture de session frauduleuse avec ce modèle.
* **Attributs pré-paramétrés** :
  * Informations d'entreprise : Société, Département, Ville, Code Postal.
  * Appartenances aux groupes de sécurité par défaut du métier.
  * Script d'ouverture de session ou chemin vers le profil itinérant.
  * **Restrictions d'horaires de connexion (*Logon Hours*)** : Limitation stricte des ouvertures de session aux plages de travail autorisées (ex: interdiction formelle le dimanche et de nuit pour les services administratifs).
* **Industrialisation par duplication** : Pour créer un nouvel utilisateur, il suffit de copier le modèle du service concerné. Tous les attributs sont hérités, l'administrateur renseignant uniquement les informations individuelles (Nom, Prénom, Téléphone, Login, Mot de passe).

### Comptes Temporaires et Intérimaires
Pour les collaborateurs externes, stagiaires ou intérimaires, la bonne pratique de sécurité impose de configurer une **date d'expiration de compte obligatoire** dès la création. À la date et heure définies, le compte est verrouillé automatiquement par Active Directory sans action manuelle requise.

---

## 3. Typologie et Étendues des Groupes de Sécurité

Dans Active Directory, un groupe est défini par son **type** et son **étendue** :

```
┌────────────────────────────────────────────────────────────────────────┐
│ TYPES DE GROUPES                                                       │
├───────────────────────────────────┬────────────────────────────────────┤
│ Groupes de Sécurité               │ Groupes de Distribution            │
│ • Peuvent recevoir des droits     │ • Réservés aux listes de diffusion │
│ • Possèdent un SID                │ • Dépourvus de SID                 │
│ • Utilisables dans les ACL NTFS   │ • Inutilisables pour la sécurité   │
└───────────────────────────────────┴────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│ ÉTENDUES DE GROUPES                                                    │
├───────────────────┬────────────────────┬───────────────────────────────┤
│ Étendue Globale   │ Domaine Local      │ Étendue Universelle           │
│ (Global Group)    │ (Domain Local)     │ (Universal Group)             │
│ • Regroupe les    │ • Porte les droits │ • Valable pour toute la forêt │
│   utilisateurs par│   d'accès sur une  │ • Répliqué dans le Catalogue   │
│   métier / fonction│   ressource locale│   Global (surcoût réseau)     │
└───────────────────┴────────────────────┴───────────────────────────────┘
```

### Mécanisme d'Imbrication des Groupes
Active Directory autorise l'imbrication de groupes. Par exemple, un groupe global `G-Support-Technique` peut être imbriqué comme membre du groupe global `G-Informatique`, conférant automatiquement à l'équipe support tous les privilèges attribués à l'ensemble du département informatique.

---

## 4. Topologie Physique des Sites Active Directory

Alors que l'arborescence des domaines et des UO constitue la structure **logique**, les **Sites Active Directory** représentent la topologie **physique** du réseau d'entreprise.

### Concepts Physiques Fondamentaux
* **Site Active Directory** : Ensemble d'un ou plusieurs sous-réseaux IP connectés par des liaisons réseau fiables et à haut débit (LAN). Typiquement, chaque bâtiment ou implantation géographique forme un site.
* **Sous-réseaux (*Subnets*)** : Plages réseau (ex: `192.168.0.0/16` pour le Siège, `172.16.0.0/16` pour une Agence distante). Chaque sous-réseau est rattaché à un site unique.
* **Liens de sites (*Site Links*)** : Définissent les connexions WAN reliant les sites entre eux, associés à un **coût** métrique et à une fréquence de réplication planifiée (ex: réplication toutes les 180 minutes en heures ouvrées ou uniquement la nuit pour préserver la bande passante WAN).

```
[ Site Siège : 192.168.0.0/16 ]                     [ Site Agence : 172.16.0.0/16 ]
┌─────────────────────────────┐                     ┌─────────────────────────────┐
│ • Contrôleur CD1            │                     │ • Postes clients agence     │
│ • Contrôleur CD2            │ ◄─────────────────► │ • Serveur local             │
│ • Serveurs d'infrastructure │   Lien WAN (Coût 100)│ • Sans contrôleur dédié     │
└─────────────────────────────┘                     └─────────────────────────────┘
```

### Bénéfices Opérationnels des Sites
1. **Localisation du contrôleur le plus proche** : Lorsqu'un utilisateur ouvre sa session, le résolveur interroge DNS pour trouver un contrôleur de domaine situé dans son propre site physique (`nltest /dsgetsite`), évitant de saturer la liaison WAN distante.
2. **Optimisation de la réplication** :
   * **Intra-site** : Réplication rapide, déclenchée par notification immédiate (délai de 15 secondes), données non compressées.
   * **Inter-sites** : Réplication planifiée, données fortement compressées pour économiser la bande passante inter-sites.

---

[← Chapitre précédent : 2. Active Directory & Rôles FSMO](02-active-directory-fsmo.md) | [Atelier pratique associé : TP 03](../tp/tp03-annuaire-societe-asr.md) | [Chapitre suivant : 4. Stockage d'Entreprise & AGDLP →](04-stockage-agdlp.md)

