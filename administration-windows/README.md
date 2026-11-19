[← Retour à l'Accueil](../README.md)

# Module — Administration Windows Server & Active Directory

* **Période :** Du 16/11/2026 au 20/11/2026 (35 heures)
* **Centre :** ENI École Informatique (Saint-Herblain) — Cursus ASR
* **Périmètre technique :** Windows Server 2022, Active Directory Domain Services (AD DS), DNS, Storage Spaces, Modèle AGDLP, Stratégies de Groupe (GPO), Sécurité des identités (PSO).

Ce module d'infrastructure est articulé en deux composantes complémentaires :
1. **La partie Cours (Théorie, Mécanismes & Architecture)** : Présentation didactique et approfondie des concepts, du dimensionnement et des règles d'ingénierie applicables en entreprise.
2. **Le Coin TP (Synthèse des Réalisations en Laboratoire)** : Récapitulatif clair et valorisant des ateliers menés sur la maquette virtuelle VMware (machines CD1, CD2, SRV1, Win-CL1).

---

## 1. Cours Théoriques & Fondations d'Architecture

La progression logique des cours suit l'ordre naturel de mise en place d'une infrastructure :

| Chapitre | Thématiques & Notions Fondamentales | Accès au cours |
| :--- | :--- | :--- |
| **1. Active Directory & Rôles FSMO** | Annuaire d'entreprise vs Workgroup, dépendances DNS et Kerberos, tolérance aux pannes par second DC et répartition stratégique des 5 rôles FSMO. | [Consulter le cours](cours/01-active-directory-architecture-fsmo.md) |
| **2. Gouvernance d'Annuaire & UO** | Règles de conception des Unités d'Organisation, gestion du cycle de vie des identités via des comptes modèles, étendues de groupes et topologie multi-sites. | [Consulter le cours](cours/02-gouvernance-annuaire-identites.md) |
| **3. Stockage d'Entreprise & AGDLP** | Espaces de stockage logiciels résilients (*Storage Spaces*), cumul Partages SMB / ACL NTFS, et implémentation stricte du modèle d'habilitation AGDLP. | [Consulter le cours](cours/03-stockage-partages-agdlp.md) |
| **4. Stratégies de Groupe & PSO** | Moteur de traitement LSDOU, gestion des conflits, magasin central SYSVOL `PolicyDefinitions`, et politiques de mot de passe à granularité fine (*PSO*). | [Consulter le cours](cours/04-gpo-securite-pso.md) |
| **5. Métrologie Système & Supervision** | Qualification de la charge serveur, compteurs critiques CPU / RAM / Disque sous `perfmon`, passerelle sans agent Windows Admin Center et audit d'événements. | [Consulter le cours](cours/05-metrologie-supervision-evenements.md) |

---

## 2. Coin TP — Synthèse des Ateliers en Laboratoire

Ces fiches récapitulent les architectures déployées, les choix de configuration et les commandes de test validées en atelier pratique sur VMware :

* **[Présentation de la Maquette & Topologie de Lab](tp/README.md)** : Réseau virtuel privé Host-Only `192.168.0.0/16`, rôles et dimensionnement des 4 machines virtuelles (CD1, CD2, SRV1, Win-CL1).
* **[TP 01 : Infrastructure & Métrologie Système](tp/tp01-infrastructure-metrologie.md)** : Déploiement du banc d'essai, collecte programmée `CompteurBase-AAMMJJ`, passerelle Windows Admin Center et audit d'événements.
* **[TP 02 : Contrôleurs de Domaine & FSMO](tp/tp02-domaine-fsmo.md)** : Promotion du domaine `domNN.ad`, contrôleur réplica CD2, outils RSAT sur Windows 11 et transfert des rôles RID / PDC.
* **[TP 03 : Organisation de l'Annuaire Société ASR](tp/tp03-annuaire-societe-asr.md)** : Arborescence UO à 3 niveaux, modèles de comptes avec restrictions d'horaires, groupes de sécurité et sites Siège / Agence.
* **[TP 04 : Stockage Résilient & Partages AGDLP](tp/tp04-stockage-partages-agdlp.md)** : Agrégation de 3 disques SCSI en pool miroir sur SRV1, partages Documentation / Compta / Info$, matrice de droits AGDLP et serveur d'impression Dell 5210CN.
* **[TP 05 : Déploiement de GPO & Sécurité PSO](tp/tp05-gpo-securite-pso.md)** : Magasin central ADMX dans SYSVOL, GPO de sécurité et restriction, stratégie affinée PSO pour l'informatique et validation par `gpresult`.

---

## 3. Mémos Techniques & Référentiels

* [Consoles MMC et raccourcis d'administration](../memos/consoles-mmc-raccourcis.md)
* [Cheatsheet PowerShell Active Directory & Système](../memos/powershell-cheatsheet.md)
* [Commandes de diagnostic réseau et annuaire](../memos/commandes-reseau-diagnostic.md)
