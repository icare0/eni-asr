# Administration Windows

* **Dates :** Du 16/11/2026 au 20/11/2026 (35 heures)
* **Lieu :** ENI École Informatique (Saint-Herblain) — Présentiel
* **Formateur / Source :** Support pédagogique ASR ENI (Maha Naceur)

Ce module traite du cycle de vie complet d'une infrastructure d'entreprise sous Windows Server 2022 et Active Directory Domain Services (AD DS) : maquettage virtuel, métrologie, promotion de contrôleurs, rôles FSMO, modèle AGDLP et durcissement par stratégies de groupe (GPO).

---

## Sommaire du Module

| Chapitre | Thématiques & Notions Clés | Fiche de cours |
| :--- | :--- | :--- |
| **01. Métrologie & Supervision** | Virtualisation VMware en Host-Only, compteurs Perfmon (CPU, RAM, Disque), seuils d'alerte, Windows Admin Center sur port HTTPS:443 et audit des événements `MsiInstaller`. | [Consulter le cours](metrologie-supervision.md) |
| **02. Active Directory & FSMO** | Promotion de CD1 en PowerShell (`Install-ADDSForest`), ajout du réplica CD2 pour tolérance aux pannes, DNS intégré et gestion des 5 rôles FSMO (`netdom query fsmo`). | [Consulter le cours](active-directory-fsmo.md) |
| **03. Gouvernance d'Annuaire & OU** | Arborescence en 3 niveaux (Société ASR), gabarits de comptes modèles, filtres LDAP bitwise pour comptes inactifs et gestion des sites physiques dans `dssite.msc`. | [Consulter le cours](gouvernance-annuaire.md) |
| **04. Stockage SMB & Modèle AGDLP** | Pools résilients Storage Spaces (miroir), cumul des autorisations NTFS et partage, matrice complète AGDLP, délégations administratives et serveur d'impression réseau. | [Consulter le cours](stockage-agdlp.md) |
| **05. Stratégies GPO & Sécurité PSO** | Ordre d'application LSDOU, politiques de mots de passe affinées (PSO en PowerShell), magasin central PolicyDefinitions dans SYSVOL et audit `gpresult`. | [Consulter le cours](gpo-securite-pso.md) |

---

## Fiches Pratiques Associées

* [Consoles MMC et raccourcis d'administration](/memos/consoles-mmc-raccourcis.md)
* [Cheatsheet PowerShell Administrateur](/memos/powershell-cheatsheet.md)
* [Commandes de diagnostic réseau et Active Directory](/memos/commandes-reseau-diagnostic.md)
