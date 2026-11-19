[← Retour au Sommaire des Cours](README.md) | [Accueil](../../README.md)

# 2. Gouvernance d'Annuaire, Unités d'Organisation & Identités

Ce cours aborde les règles d'architecture indispensables pour concevoir un annuaire d'entreprise pérenne : structurer les Unités d'Organisation (UO), industrialiser la gestion du cycle de vie des collaborateurs par modèles et maîtriser les groupes de sécurité et la topologie physique des sites.

---

## 1. Pourquoi et Comment Structurer un Annuaire ?

Lors de l'installation initiale d'Active Directory, les objets sont déposés en vrac dans deux conteneurs par défaut : `CN=Users` et `CN=Computers`. Laisser une entreprise dans cette configuration est une erreur d'exploitation majeure.

Une infrastructure d'entreprise repose sur la création d'**Unités d'Organisation** (*Organizational Units* - OU).

### Les Deux Seules Raisons Valables de Créer une UO
Dans la théorie Microsoft, on ne crée pas une UO simplement pour copier l'organigramme RH. On crée une UO pour répondre à **deux objectifs techniques stricts** :

1. **Définir la portée d'application des Stratégies de Groupe (GPO)** : Une GPO ne peut pas être liée aux conteneurs par défaut (`CN=Users` ou `CN=Computers`). Elle ne peut être liée qu'à la racine du domaine, à un site physique ou à une **Unité d'Organisation**.
2. **Définir les frontières de Délégation Administrative** : Permettre à un technicien support de réinitialiser les mots de passe ou de déverrouiller des comptes sur un périmètre restreint sans lui donner les pleins pouvoirs d'administrateur du domaine.

| Caractéristique | Conteneurs par défaut (`CN=Users`, `CN=Computers`) | Unités d'Organisation (`OU=Direction,OU=Utilisateurs...`) |
| :--- | :--- | :--- |
| **Liaison de GPO** | **Impossible**. Les conteneurs `CN=` ne supportent aucune stratégie de groupe. | **Native**. Chaque UO peut hériter ou posséder ses propres liaisons de GPO. |
| **Délégation de contrôle** | Risquée et déconseillée. | **Chirurgicale**. L'assistant de délégation permet d'assigner des droits fins par UO. |
| **Arborescence hiérarchique** | Impossible (structure plate non imbriquable). | **Arborescente**. Supporte plusieurs niveaux d'imbrication hiérarchique. |

---

## 2. Méthodologie d'Arborescence : Le Modèle à 3 Niveaux

Pour garantir la lisibilité et la maintenance à long terme d'un annuaire d'entreprise, la méthodologie de référence structure l'annuaire en **3 niveaux hiérarchiques** :

```
domaine.local
└── Entreprise (Racine d'organisation)
    ├── Catégorie : Utilisateurs
    │   ├── Service : Direction
    │   ├── Service : Informatique
    │   └── Service : Comptabilité
    ├── Catégorie : Groupes
    │   ├── Groupes de Sécurité Métiers
    │   └── Groupes de Ressources
    ├── Catégorie : Postes de travail
    │   ├── Ordinateurs Fixes
    │   └── Ordinateurs Portables
    └── Catégorie : Serveurs
        ├── Contrôleurs de domaine
        └── Serveurs membres
```

> [!IMPORTANT]
> **Pourquoi séparer strictement les Utilisateurs et les Ordinateurs ?**  
> Une GPO contient deux branches indépendantes : la configuration **Ordinateur** et la configuration **Utilisateur**. Placer les comptes d'ordinateurs dans une UO dédiée et les utilisateurs dans une autre évite que le moteur Windows n'évalue inutilement des politiques machine sur des sessions utilisateurs, ce qui accélère le temps d'ouverture de session et supprime les effets de bord.

---

## 3. Gestion Industrielle des Comptes Utilisateurs

### Utilisation de Comptes Modèles (*Templates*)
Pour éviter les erreurs manuelles lors de l'arrivée d'un nouveau collaborateur (oubli d'un groupe, faute dans le nom du service, omission d'un droit), l'administrateur prépare des **comptes modèles** par métier.

* **État impératif du compte modèle** : Le compte modèle doit impérativement rester **désactivé**. Cela interdit formellement à quiconque d'ouvrir une session avec ce compte générique.
* **Attributs types configurés dans le modèle** :
  * Informations de société, service, ville, code postal.
  * Appartenance aux groupes de sécurité par défaut du service.
  * **Restrictions d'horaires de connexion (*Logon Hours*)** : Définition des plages de travail autorisées (par exemple, interdire l'ouverture de session la nuit et le week-end pour limiter les risques de compromission en dehors des heures de bureau).
* **Industrialisation par duplication** : À l'arrivée d'un collaborateur, l'administrateur clique droit sur le modèle correspondant > **Copier**. Le nouveau compte hérite automatiquement de tous les attributs prédéfinis ; il ne reste plus qu'à renseigner le nom, le prénom, le login et le mot de passe initial.

### Comptes Temporaires et Intérimaires
Pour tout profil externe, stagiaire ou prestataire, la bonne pratique de sécurité impose de configurer une **date d'expiration de compte** (*Account Expiration Date*) dès sa création. À l'échéance programmée, Active Directory verrouille automatiquement le compte sans nécessiter d'intervention humaine.

---

## 4. Stratégie des Groupes de Sécurité

Dans Active Directory, un groupe est caractérisé par deux propriétés : son **type** et son **étendue**.

```
┌────────────────────────────────────────────────────────────────────────┐
│ TYPES DE GROUPES                                                       │
├───────────────────────────────────┬────────────────────────────────────┤
│ Groupes de Sécurité               │ Groupes de Distribution            │
│ • Possèdent un identifiant SID    │ • Dépourvus d'identifiant SID      │
│ • Peuvent recevoir des droits     │ • Réservés aux listes de diffusion │
│ • Utilisables dans les ACL NTFS   │   de messagerie (ex: Exchange)     │
└───────────────────────────────────┴────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│ ÉTENDUES DE GROUPES (*Group Scopes*)                                   │
├───────────────────┬────────────────────┬───────────────────────────────┤
│ Étendue Globale   │ Domaine Local      │ Étendue Universelle           │
│ (Global Group)    │ (Domain Local)     │ (Universal Group)             │
│ • Regroupe les    │ • Affecte les      │ • Valable pour toute la forêt │
│   utilisateurs par│   droits sur une   │ • Répliqué dans le Catalogue  │
│   métier / fonction│   ressource locale │   Global (gourmand en bande)  │
└───────────────────┴────────────────────┴───────────────────────────────┘
```

### Le Principe d'Imbrication des Groupes
Active Directory permet d'imbriquer des groupes pour simplifier l'administration.  
*Exemple :* Si une entreprise dispose d'une équipe `Support-Technique` et d'une équipe `Administrateurs-Systèmes`, ces deux groupes peuvent être imbriqués au sein d'un groupe parent `Departement-Informatique`. Toute ressource autorisée pour le département informatique sera automatiquement accessible par les membres des deux sous-équipes.

---

## 5. Topologie Physique : Les Sites Active Directory

Alors que les domaines et les Unités d'Organisation constituent la structure **logique**, les **Sites Active Directory** décrivent la réalité **physique** du réseau d'entreprise.

### Concepts Physiques
* **Site Active Directory** : Représente une zone géographique connectée par un réseau local rapide et fiable (LAN). Par exemple : le site *Siège*, le site *Usine* ou le site *Agence-Paris*.
* **Sous-réseaux (*Subnets*)** : Plages d'adresses IP associées à chaque site physique (ex: `192.168.1.0/24` pour le Siège, `10.10.0.0/24` pour l'Agence).
* **Liens de sites (*Site Links*)** : Décrivent les liaisons WAN interconnectant les sites distants, avec un **coût** métrique et une fréquence de réplication planifiée (ex: réplication toutes les 3 heures pour ne pas encombrer les liens WAN en journée).

```
[ Site Siège : 192.168.1.0/24 ]                     [ Site Agence : 10.10.0.0/24 ]
┌─────────────────────────────┐                     ┌─────────────────────────────┐
│ • Contrôleurs de domaine    │                     │ • Postes clients locaux     │
│ • Serveurs d'infrastructure │ ◄─────────────────► │ • Serveur de fichiers local │
│ • Connexion LAN 10 Gbps     │   Lien WAN (Coût 100)│ • Sans contrôleur dédié     │
└─────────────────────────────┘                     └─────────────────────────────┘
```

### Pourquoi Définir des Sites ?
1. **Authentification locale prioritaire** : Lorsqu'un poste client démarre, il utilise le DNS pour identifier le contrôleur de domaine situé dans son propre site géographique (`nltest /dsgetsite`). Cela évite de faire transiter les ouvertures de sessions à travers des liaisons distantes lentes.
2. **Optimisation de la réplication** :
   * À l'intérieur d'un même site (LAN) : Réplication quasi instantanée par notification (délai de 15 secondes), non compressée.
   * Entre sites distants (WAN) : Réplication compressée et planifiée sur les heures creuses.

---

[← Chapitre précédent : 1. Architecture Active Directory & Rôles FSMO](01-active-directory-architecture-fsmo.md) | [Atelier pratique associé : TP 03](../tp/tp03-annuaire-societe-asr.md) | [Chapitre suivant : 3. Stockage d'Entreprise & Modèle AGDLP →](03-stockage-partages-agdlp.md)
