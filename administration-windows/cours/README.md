[← Administration Windows](../README.md) | [Accueil](../../README.md)

# Cours — Administration Windows Server & Active Directory

Ce dossier regroupe les fondations théoriques, conceptuelles et architecturales du module **Administration système Windows** du cursus ASR (Administrateur Systèmes et Réseaux) de l'ENI.

---

## Objectifs Pédagogiques du Module

L'objectif de ce module est de maîtriser les composants fondamentaux d'une infrastructure d'entreprise sous Windows Server :

* **Supervision et métrologie** : Comprendre le fonctionnement des sous-systèmes matériels (CPU, RAM, Disque, Réseau), savoir qualifier une charge système et auditer les événements Windows.
* **Services d'annuaire (AD DS)** : Maîtriser l'architecture logique (forêt, domaine, catalogue global) et physique (sites, sous-réseaux) d'Active Directory, les mécanismes de réplication et la tolérance aux pannes via la répartition des rôles FSMO.
* **Gouvernance et identités** : Savoir concevoir une arborescence d'Unités d'Organisation (UO) maintenable, gérer le cycle de vie des comptes et implémenter une stratégie de groupes cohérente.
* **Stockage et partages de fichiers** : Déployer des espaces de stockage résilients logiciels (*Storage Spaces*), configurer des partages SMB et appliquer sans faille le modèle de droits d'accès **AGDLP**.
* **Stratégies de groupe (GPO) et durcissement** : Automatiser la configuration et la sécurité des postes et serveurs via le moteur GPO (ordre LSDOU, filtrage WMI) et implémenter des stratégies de mot de passe affinées (**PSO**).

---

## Sommaire des Chapitres

1. **[1. Métrologie, analyse de performances et Windows Admin Center](01-metrologie-supervision.md)**
   * Sous-systèmes matériels et compteurs clés sous `perfmon`.
   * Ensembles de collecteurs programmés et métrologie préventive.
   * Architecture sans agent de Windows Admin Center (WAC).
   * Audit et journaux d'événements Windows (`eventvwr`).

2. **[2. Active Directory Domain Services (AD DS) et rôles FSMO](02-active-directory-fsmo.md)**
   * Architecture logique (forêt, domaine, arbre) et dépendances DNS.
   * Déploiement et promotion d'un contrôleur de domaine en PowerShell.
   * Contrôleur réplica et résilience de l'annuaire.
   * Les 5 rôles FSMO : fonctionnement, répartition stratégique et transfert/saisie (*seize*).

3. **[3. Gouvernance d'annuaire, Unités d'Organisation et identités](03-gouvernance-annuaire.md)**
   * Principes de conception des UO (portée GPO et délégation de droits).
   * Gestion des comptes utilisateurs par modèles (*templates*) et restrictions d'horaires.
   * Typologie et étendues des groupes (Global, Domaine Local, Universel).
   * Topologie physique des sites Active Directory et réplication inter-sites.

4. **[4. Espaces de stockage, partages SMB et modèle AGDLP](04-stockage-agdlp.md)**
   * Agrégation de disques et résilience logicielle (*Storage Spaces*).
   * Points de montage NTFS vs lettres de lecteur.
   * Barrières de sécurité : cumul Partages SMB et ACL NTFS.
   * Mécanisme AGDLP : jetons d'accès Kerberos et simplification de maintenance.

5. **[5. Stratégies de groupe (GPO), SYSVOL et politiques PSO](05-gpo-securite-pso.md)**
   * Ordre de traitement des stratégies (LSDOU) et résolution des conflits.
   * Magasin central des modèles d'administration (`PolicyDefinitions` dans SYSVOL).
   * Filtrage de sécurité, ciblage par éléments et filtres WMI.
   * Politiques de mot de passe du domaine vs stratégies affinées (*Password Settings Objects* - PSO).

---

> [!NOTE]
> Pour consulter la synthèse des ateliers pratiques réalisés en environnement virtuel VMware (machines CD1, CD2, SRV1, Win-CL1), reportez-vous à la section **[Coin TP — Réalisations en Lab](../tp/README.md)**.
