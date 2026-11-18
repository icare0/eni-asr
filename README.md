# Documentation ASR — Administrateur Systèmes & Réseaux

Ce portail rassemble l'ensemble des fiches techniques, cours d'architecture et commandes d'exploitation pour les deux années du cursus d'Administrateur Systèmes et Réseaux de l'ENI.

> **Recherche rapide :** Utilisez le champ de recherche dans la colonne de gauche ou consultez l'[Index des commandes et notions](index-thematique.md) pour accéder directement à une notion (`FSMO`, `AGDLP`, `PSO`, `WAC`, `VLAN`, `Storage Spaces`, etc.).

---

## 1ère Année : Administration Système & Réseau On-Premises

Les modules de première année se concentrent sur le déploiement, la gestion des accès et l'administration quotidienne des parcs Windows Server et Linux.

| Module | Thématiques abordées | Fiche technique |
| :--- | :--- | :--- |
| **01. Métrologie & Supervision** | Virtualisation VMware en Host-Only, métrologie Perfmon (compteurs CPU/RAM/Disque), Windows Admin Center (HTTPS:443), analyse des journaux d'événements. | [Consulter le cours](annee-1/environnement-outils.md) |
| **02. Active Directory & Rôles FSMO** | Déploiement en PowerShell (`Install-ADDSForest`), tolérance aux pannes avec CD2, gestion du DNS intégré et maîtrise des 5 rôles FSMO (`netdom query fsmo`). | [Consulter le cours](annee-1/windows-server-ad.md) |
| **03. Gouvernance d'Annuaire & OU** | Arborescence d'Unités d'Organisation en 3 niveaux, gabarits de comptes modèles, filtres LDAP de contrôle, topologie des sites et sous-réseaux IP (`dssite.msc`). | [Consulter le cours](annee-1/gestion-annuaire-asr.md) |
| **04. Stockage SMB & Modèle AGDLP** | Pools résilients Storage Spaces (miroir), cumul des autorisations NTFS et partage, matrice de droits AGDLP, délégations administratives, serveur d'impression. | [Consulter le cours](annee-1/stockage-partages-agdlp.md) |
| **05. Stratégies de Groupe (GPO)** | Ordre d'application LSDOU, politiques de mots de passe affinées (PSO en PowerShell), magasin central PolicyDefinitions dans SYSVOL et diagnostic client (`gpresult`). | [Consulter le cours](annee-1/gpo-securite.md) |
| **06. Fondations Réseau & Services IP** | Modèles OSI et TCP/IP, calculs de sous-réseaux IPv4/IPv6 (CIDR/VLSM), processus d'adressage DHCP (DORA) et enregistrements de service DNS (SRV). | [Consulter le cours](annee-1/reseau-fondations.md) |
| **07. Administration Linux & Bash** | Arborescence standard FHS, permissions POSIX octales (`chmod`), gestionnaire de services `systemctl`, automatisation par scripts Bash et accès sécurisé SSH. | [Consulter le cours](annee-1/linux-fondations.md) |

---

## 2ème Année : Ingénierie, Sécurité & Cloud

Les modules de seconde année traitent de la haute disponibilité, du durcissement d'infrastructure face aux attaques, de l'automatisation et de l'hybridation vers le Cloud.

| Module | Thématiques abordées | Fiche technique |
| :--- | :--- | :--- |
| **01. Commutation & Routage Cisco** | Segmentation par VLANs 802.1Q, liens Trunk, routage inter-VLAN, protocole anti-boucle Spanning Tree (RSTP), routage dynamique OSPF et listes ACL. | [Consulter le cours](annee-2/reseaux-avances-cisco.md) |
| **02. Haute Disponibilité (HA / PRA)** | Clusters de basculement Windows Server (WSFC), gestion du quorum et des témoins, réplication de fichiers DFS (DFS-N / DFS-R) et stratégie de sauvegarde 3-2-1. | [Consulter le cours](annee-2/haute-disponibilite.md) |
| **03. Sécurité SI, Tiering & PKI** | Modèle de séparation administrative en couches (Tier 0/1/2 ANSSI), solution LAPS pour comptes locaux et autorité de certification d'entreprise (AD CS). | [Consulter le cours](annee-2/securite-durcissement-pki.md) |
| **04. Virtualisation Datacenter & Azure** | Hyperviseurs Type 1 (VMware ESXi / vCenter, vMotion, DRS, HA), Proxmox VE et synchronisation d'identités hybrides avec Microsoft Entra Connect. | [Consulter le cours](annee-2/virtualisation-cloud.md) |
| **05. Automatisation & DevOps** | Scripting PowerShell orienté objet avec gestion d'erreurs (`try/catch`), gestion de configuration sans agent avec Ansible, conteneurs Docker et Git pour l'infrastructure. | [Consulter le cours](annee-2/automatisation-devops.md) |
| **06. Titre Professionnel ASR** | Référentiel des compétences AT1 et AT2, constitution du Dossier Professionnel (DP) et préparation de la soutenance orale devant le jury professionnel. | [Consulter le cours](annee-2/titre-professionnel-projet.md) |

---

## Référentiels & Mémos d'Exploitation

* [Consoles MMC et raccourcis d'administration](memos/consoles-mmc-raccourcis.md) : Liste des fichiers `.msc`, applets `.cpl` et raccourcis clavier pour Server Manager et RDP.
* [Cheatsheet PowerShell Administrateur](memos/powershell-cheatsheet.md) : Commandes d'administration pour Active Directory, le stockage, les partages et PowerShell Remoting.
* [Commandes de diagnostic réseau et Active Directory](memos/commandes-reseau-diagnostic.md) : Outils de dépannage (`netdom`, `nltest`, `Test-NetConnection`, `repadmin`, `dcdiag`).
