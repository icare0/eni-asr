[← Retour à l'Accueil](../README.md)

# Module — Administration Windows Server & Active Directory

* **Période :** Du 16/11/2026 au 20/11/2026 (35 heures)
* **Centre :** ENI École Informatique (Saint-Herblain) — Cursus ASR
* **Périmètre technique :** Windows Server 2022, Active Directory Domain Services (AD DS), DNS, Storage Spaces, Modèle AGDLP, Stratégies de Groupe (GPO), Sécurité des identités (PSO).

Ce module d'infrastructure est articulé en deux composantes complémentaires :
1. **La partie Cours (Théorie & Architecture)** : Présentation approfondie des concepts, des normes Microsoft, du fonctionnement interne des protocoles et du dimensionnement d'infrastructure.
2. **Le Coin TP (Synthèse des Réalisations)** : Synthèse exécutive et valorisante des ateliers pratiques menés sur la maquette virtualisée VMware (CD1, CD2, SRV1, Win-CL1), sans tutoriel verbeux de clics de souris.

---

## 1. Cours Théoriques & Fondations d'Architecture

Ces dossiers techniques détaillent les mécanismes internes et les règles de conception pour un environnement de production d'entreprise :

| Chapitre | Thématiques & Notions Fondamentales | Accès au cours |
| :--- | :--- | :--- |
| **1. Métrologie & Supervision** | Qualification de la charge serveur, compteurs critiques CPU / RAM / Disque sous `perfmon`, collecte programmée, architecture sans agent de Windows Admin Center et audit d'événements `MsiInstaller`. | [Consulter le cours](cours/01-metrologie-supervision.md) |
| **2. Active Directory & Rôles FSMO** | Architecture logique et physique de l'annuaire, dépendances DNS, réplication multi-maîtres, résilience par second DC et répartition stratégique des 5 rôles FSMO. | [Consulter le cours](cours/02-active-directory-fsmo.md) |
| **3. Gouvernance d'Annuaire & UO** | Règles de conception des Unités d'Organisation, gestion du cycle de vie des identités via des comptes modèles, étendues de groupes et topologie multi-sites Active Directory. | [Consulter le cours](cours/03-gouvernance-annuaire.md) |
| **4. Stockage d'Entreprise & AGDLP** | Espaces de stockage logiciels résilients (*Storage Spaces*), cumul Partages SMB / ACL NTFS, implémentation stricte du modèle AGDLP et impact sur les jetons Kerberos (*Token Bloat*). | [Consulter le cours](cours/04-stockage-agdlp.md) |
| **5. Stratégies de Groupe & PSO** | Moteur de traitement LSDOU, gestion des conflits, magasin central SYSVOL `PolicyDefinitions`, et politiques de mot de passe à granularité fine (*Password Settings Objects* - PSO). | [Consulter le cours](cours/05-gpo-securite-pso.md) |

---

## 2. Coin TP — Synthèse des Réalisations en Laboratoire

Ces fiches récapitulent les architectures déployées, les choix de configuration et les commandes de test validées en atelier pratique :

* **[Présentation de la Maquette & Topologie de Lab](tp/README.md)** : Réseau virtuel privé Host-Only `192.168.0.0/16`, rôles et dimensionnement des 4 machines virtuelles (CD1, CD2, SRV1, Win-CL1).
* **[TP 01 : Infrastructure & Métrologie Système](tp/tp01-infrastructure-metrologie.md)** : Déploiement du banc d'essai, collecte programmée `CompteurBase-AAMMJJ`, passerelle Windows Admin Center et audit d'événements.
* **[TP 02 : Contrôleurs de Domaine & FSMO](tp/tp02-domaine-fsmo.md)** : Promotion du domaine racine `domNN.ad`, haute disponibilité avec le second contrôleur réplica CD2, RSAT sur Windows 11 et transfert des rôles RID / PDC.
* **[TP 03 : Organisation de l'Annuaire Société ASR](tp/tp03-annuaire-societe-asr.md)** : Arborescence UO à 3 niveaux, modèles de comptes avec restrictions d'horaires, groupes de sécurité et sites Siège / Agence.
* **[TP 04 : Stockage Résilient & Partages AGDLP](tp/tp04-stockage-partages-agdlp.md)** : Agrégation de 3 disques SCSI en pool miroir sur SRV1, partages Documentation / Compta / Info$, matrice de droits AGDLP et serveur d'impression Dell 5210CN.
* **[TP 05 : Déploiement de GPO & Sécurité PSO](tp/tp05-gpo-securite-pso.md)** : Magasin central ADMX dans SYSVOL, GPO de sécurité et restriction, stratégie affinée PSO pour l'informatique et validation par `gpresult`.

---

## 3. Mémos Techniques & Référentiels

* [Consoles MMC et raccourcis d'administration](../memos/consoles-mmc-raccourcis.md)
* [Cheatsheet PowerShell Active Directory & Système](../memos/powershell-cheatsheet.md)
* [Commandes de diagnostic réseau et annuaire](../memos/commandes-reseau-diagnostic.md)
