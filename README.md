# Documentation ASR — Administrateur Système & Réseau

Portail de documentation et fiches techniques d'exploitation pour le cursus ASR de l'ENI (Saint-Herblain).

* **Stagiaire :** Antoine BERNARD
* **Formation :** Titre Professionnel Administrateur·rice Système et Réseau (Niveau 6, RNCP)
* **Période :** Du 16/11/2026 au 27/10/2028
* **Planning complet :** [Consulter le calendrier officiel de formation](calendrier.md)

---

## Module Actuel : Administration Windows (16/11 - 20/11/2026)

Ce premier module de 35 heures couvre l'architecture, le déploiement et la sécurisation d'une infrastructure d'entreprise sous Windows Server 2022 et Active Directory Domain Services (AD DS).

| Chapitre | Thématiques abordées | Fiche de cours |
| :--- | :--- | :--- |
| **01. Métrologie & Supervision** | Virtualisation VMware en Host-Only, analyse de performances Perfmon (compteurs CPU/RAM/Disque), Windows Admin Center sur port HTTPS:443 et audit d'événements. | [Consulter le cours](administration-windows/metrologie-supervision.md) |
| **02. Active Directory & Rôles FSMO** | Déploiement en PowerShell (`Install-ADDSForest`), contrôleur réplica CD2 pour la haute disponibilité, DNS intégré et gestion des 5 rôles FSMO (`netdom query fsmo`). | [Consulter le cours](administration-windows/active-directory-fsmo.md) |
| **03. Gouvernance d'Annuaire & OU** | Arborescence d'OU hiérarchique en 3 niveaux (Société ASR), gabarits de comptes modèles, filtres LDAP bitwise pour comptes inactifs et gestion des sites physiques dans `dssite.msc`. | [Consulter le cours](administration-windows/gouvernance-annuaire.md) |
| **04. Stockage SMB & Modèle AGDLP** | Pools de stockage résilients Storage Spaces (miroir), cumul des autorisations NTFS et partage, matrice de droits AGDLP, délégations administratives, serveur d'impression. | [Consulter le cours](administration-windows/stockage-agdlp.md) |
| **05. Stratégies GPO & Sécurité PSO** | Ordre d'application LSDOU, politiques de mots de passe affinées (PSO en PowerShell), magasin central PolicyDefinitions dans SYSVOL et audit client avec `gpresult`. | [Consulter le cours](administration-windows/gpo-securite-pso.md) |

---

## Mémos Pratiques & Commandes d'Exploitation

* [Consoles MMC et raccourcis d'administration](memos/consoles-mmc-raccourcis.md) : Fichiers `.msc`, applets `.cpl` et raccourcis clavier pour Server Manager et les sessions RDP.
* [Cheatsheet PowerShell Administrateur](memos/powershell-cheatsheet.md) : Commandes pour Active Directory, le stockage, les partages et l'exécution distante.
* [Commandes de diagnostic réseau et Active Directory](memos/commandes-reseau-diagnostic.md) : Outils de vérification et de santé (`netdom`, `nltest`, `Test-NetConnection`, `repadmin`, `dcdiag`).
* [Index des commandes et notions](index-thematique.md) : Accès rapide par mot-clé (`FSMO`, `AGDLP`, `PSO`, `WAC`, `Storage Spaces`, etc.).
