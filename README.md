# Documentation ASR — Administrateur Système & Réseau

Portail de référence technique et suivi des travaux pratiques pour le cursus ASR de l'ENI École Informatique (Saint-Herblain).

* **Stagiaire :** Antoine BERNARD
* **Formation :** Titre Professionnel Administrateur·rice Système et Réseau (Niveau 6 / Bac+3)
* **Période :** Du 16/11/2026 au 27/10/2028 (1 092 heures en centre)
* **Planning complet :** [Consulter le calendrier officiel de formation](calendrier.md)
* **Index alphabétique :** [Index des notions et commandes](index-thematique.md)

---

## Module Actuel : Administration Windows Server (16/11 - 20/11/2026)

Ce premier module de 35 heures traite de l'architecture, du déploiement et de la sécurisation d'une infrastructure d'entreprise sous Windows Server 2022 et Active Directory Domain Services (AD DS).

```
eni-asr/
├── administration-windows/
│   ├── cours/    <-- Concepts généraux, théories, protocoles et architectures
│   └── tp/       <-- Synthèse valorisante des 5 ateliers réalisés en Lab
├── memos/        <-- Consoles MMC, Cheatsheet PowerShell, Commandes diagnostic
├── calendrier.md <-- Planning officiel 2026-2028
└── index-thematique.md
```

---

### 1. Cours Théoriques & Fondations Techniques

Un corpus structuré expliquant les mécanismes internes et les règles d'ingénierie système, selon une progression logique :

| Chapitre | Concepts & Notions Clés | Fiche de cours |
| :--- | :--- | :--- |
| **01. Active Directory & Rôles FSMO** | Architecture d'annuaire d'entreprise vs Workgroup, dépendances DNS/Kerberos, réplication multi-maîtres et gouvernance des 5 rôles FSMO. | [Lire le cours](administration-windows/cours/01-active-directory-architecture-fsmo.md) |
| **02. Gouvernance d'Annuaire & UO** | Conception d'arborescences d'UO à 3 niveaux, gestion du cycle de vie des identités par modèles avec restrictions horaires, stratégie de groupes et topologie multi-sites. | [Lire le cours](administration-windows/cours/02-gouvernance-annuaire-identites.md) |
| **03. Stockage d'Entreprise & AGDLP** | Espaces de stockage logiciels résilients (*Storage Spaces*), sécurité cumulée SMB/NTFS, et implémentation stricte du modèle d'habilitation AGDLP. | [Lire le cours](administration-windows/cours/03-stockage-partages-agdlp.md) |
| **04. Stratégies GPO & Sécurité PSO** | Moteur de traitement LSDOU, gestion des conflits, magasin central SYSVOL `PolicyDefinitions` et politiques de mot de passe à granularité fine (*PSO*). | [Lire le cours](administration-windows/cours/04-gpo-securite-pso.md) |
| **05. Métrologie Système & Supervision** | Qualification des charges serveur, compteurs critiques (CPU, RAM, Disque, Réseau) sous `perfmon`, collecteurs automatisés, Windows Admin Center et audit d'événements. | [Lire le cours](administration-windows/cours/05-metrologie-supervision-evenements.md) |

---

### 2. Coin TP — Synthèse des Ateliers en Laboratoire

Une synthèse des architectures déployées et des livrables validés en atelier sur la maquette virtuelle VMware (machines `CD1`, `CD2`, `SRV1`, `Win-CL1`) :

* **[Présentation de la Maquette & Topologie de Lab](administration-windows/tp/README.md)** : Réseau Host-Only `192.168.0.0/16`, dimensionnement et rôles des 4 machines virtuelles.
* **[TP 01 : Création de l'infrastructure & Métrologie](administration-windows/tp/tp01-infrastructure-metrologie.md)** : Configuration IP, collecteur automatisé `CompteurBase-AAMMJJ`, passerelle WAC et audit d'installation.
* **[TP 02 : Création du domaine Active Directory & FSMO](administration-windows/tp/tp02-domaine-fsmo.md)** : Promotion du domaine `domNN.ad`, contrôleur réplica CD2, outils RSAT sur Windows 11 et transfert des rôles RID/PDC.
* **[TP 03 : Structuration de l'annuaire Société ASR](administration-windows/tp/tp03-annuaire-societe-asr.md)** : Arborescence d'UO à 3 niveaux, modèles de comptes, groupes globaux imbriqués et topologie Siège / Agence (`nltest`).
* **[TP 04 : Espaces de stockage résilients & Modèle AGDLP](administration-windows/tp/tp04-stockage-partages-agdlp.md)** : Pool miroir de 3 disques SCSI sur SRV1, partages Documentation / Compta / Info$, matrice de droits AGDLP et imprimante Dell 5210CN.
* **[TP 05 : Stratégies de groupe (GPO) & Durcissement PSO](administration-windows/tp/tp05-gpo-securite-pso.md)** : Magasin central ADMX dans SYSVOL, GPO de restriction, objet PSO pour l'informatique et audit `gpresult`.

---

### 3. Mémos Pratiques & Référentiels d'Exploitation

* **[Consoles MMC et raccourcis d'administration](memos/consoles-mmc-raccourcis.md)** : Fichiers `.msc`, applets `.cpl` et raccourcis clavier pour Server Manager et les sessions RDP.
* **[Cheatsheet PowerShell Administrateur](memos/powershell-cheatsheet.md)** : Commandes de gestion pour Active Directory, le stockage, les partages et l'administration distante.
* **[Commandes de diagnostic réseau et Active Directory](memos/commandes-reseau-diagnostic.md)** : Outils de vérification et de santé (`netdom`, `nltest`, `Test-NetConnection`, `repadmin`, `dcdiag`, `logman`).
