# 📘 Année 1 : Fondations Systèmes, Réseaux & Infrastructure On-Premises

Bienvenue dans la section consacrée à la **première année du cursus Administrateur Systèmes et Réseaux de l'ENI**.

L'objectif de cette première année est de construire un socle technique solide et opérationnel. En tant qu'administrateur, vous devez être capable de déployer, configurer, sécuriser et superviser une infrastructure d'entreprise locale complète basée sur **Windows Server 2022**, **Active Directory Domain Services (AD DS)**, les services réseau de base et une première couche **Linux**.

---

## 🎯 Compétences Cibles de l'Année 1

* **Virtualisation & Maquettage** : Savoir concevoir un réseau privé virtuel sous VMware Workstation / ESXi pour simuler une PME ou une filiale.
* **Architecture d'Annuaire** : Déployer une forêt, des domaines, configurer la réplication inter-sites et comprendre les rôles FSMO.
* **Gestion des Identités** : Organiser une arborescence d'Unités d'Organisation (OU), gérer le cycle de vie des utilisateurs et des groupes de sécurité.
* **Sécurisation de l'Accès aux Données** : Mettre en œuvre le modèle de référence **AGDLP**, configurer des pools de stockage résilients et des partages SMB sécurisés par des listes de contrôle d'accès NTFS.
* **Automatisation & Politiques (GPO)** : Standardiser les postes clients et serveurs via les stratégies de groupe, gérer les politiques de mots de passe de domaine et affinées (PSO), et centraliser les modèles ADMX dans SYSVOL.
* **Fondamentaux Réseau** : Maîtriser le plan d'adressage IPv4/IPv6, le découpage en sous-réseaux (VLSM/CIDR), et les services d'infrastructure DHCP et DNS.
* **Systèmes Linux** : Prendre en main la ligne de commande, l'arborescence standard FHS, la gestion des permissions et le scripting Bash.

---

## 📚 Sommaire des Modules d'Année 1

```
Année 1 : Le Socle Fondamental
│
├── 1. Environnement, Virtualisation & Outils
│    └── VMware, MMC, Windows Admin Center, Perfmon & Journaux d'événements
│
├── 2. Windows Server & Active Directory (AD DS)
│    └── Forêt, Domaines, Contrôleurs CD1/CD2, Rôles FSMO, Catalogue Global
│
├── 3. Gestion d'Annuaire & Société ASR
│    └── Comptes modèles, Groupes globaux, Arborescence OU 3 niveaux, Requêtes LDAP
│
├── 4. Stockage, Partages SMB & Modèle AGDLP
│    └── Storage Spaces, Partages SMB, Droits NTFS, Matrice AGDLP, Impression
│
├── 5. Stratégies de Groupe (GPO) & Sécurité
│    └── GPO utilisateurs/ordinateurs, PSO mots de passe, Magasin central SYSVOL
│
├── 6. Fondations Réseau & Services IP
│    └── Modèle OSI, IPv4/IPv6, Sous-réseaux CIDR, DNS & DHCP d'entreprise
│
└── 7. Découverte & Administration Linux
     └── CLI, Arborescence FHS, Droits chmod/chown, Bash scripting, Accès SSH
```

---

## 🔬 Les Ateliers Pratiques Associés

Chaque concept théorique est directement validé par un atelier pratique :

* [🧪 **TP 01 : Infrastructure & Métrologie**](/ateliers/tp01-infrastructure-metrologie.md)
* [🧪 **TP 02 : Création Domaine AD & Rôles FSMO**](/ateliers/tp02-domaine-ad-fsmo.md)
* [🧪 **TP 03 : Gestion d'Annuaire & Société ASR**](/ateliers/tp03-gestion-societe-asr.md)
* [🧪 **TP 04 : Stockage résilient & Modèle AGDLP**](/ateliers/tp04-ressources-agdlp.md)
* [🧪 **TP 05 : Stratégies de Groupe (GPO) & Sécurité**](/ateliers/tp05-gpo-securite-avancee.md)
