[← Administration Windows](../README.md) | [Accueil](../../README.md)

# Cours — Administration Windows Server & Active Directory

Ce dossier regroupe les fondations théoriques, conceptuelles et architecturales du module **Administration système Windows** du cursus ASR (Administrateur Systèmes et Réseaux) de l'ENI.

Contrairement aux fiches de travaux pratiques qui décrivent des manipulations de laboratoire spécifiques, ces cours développent les principes universels d'ingénierie et d'administration d'une infrastructure d'entreprise.

---

## Sommaire Pédagogique des Chapitres

La progression suit l'ordre logique d'ingénierie d'une infrastructure :

1. **[1. Architecture Active Directory (AD DS) & Rôles FSMO](01-active-directory-architecture-fsmo.md)**
   * Pourquoi un annuaire centralisé ? (Limites du Workgroup vs Domaine).
   * Architecture logique (Forêt, Domaine, Arbre, Catalogue Global).
   * Dépendances DNS intégrées (enregistrements SRV) et protocole d'authentification Kerberos.
   * Haute disponibilité par réplication multi-maîtres et gestion des 5 rôles FSMO.

2. **[2. Gouvernance d'Annuaire, Unités d'Organisation & Identités](02-gouvernance-annuaire-identites.md)**
   * Les deux impératifs de conception d'une UO (portée GPO et frontière de délégation).
   * Méthodologie d'arborescence à 3 niveaux (séparation Utilisateurs / Ordinateurs).
   * Industrialisation par comptes modèles désactivés et restrictions d'horaires.
   * Stratégie des groupes de sécurité (étendues Globale, Domaine Local, Universelle) et sites physiques.

3. **[3. Stockage d'Entreprise, Partages SMB & Modèle AGDLP](03-stockage-partages-agdlp.md)**
   * Virtualisation du stockage avec les Espaces de stockage logiciels (*Storage Spaces*).
   * Types de résilience (Miroir, Parité) et points de montage NTFS dans un dossier.
   * La règle d'or du cumul : Partages SMB vs Permissions NTFS.
   * Le modèle universel d'habilitation AGDLP (prévention du gonflement de jeton Kerberos).

4. **[4. Stratégies de Groupe (GPO) & Sécurité du Domaine (PSO)](04-gpo-securite-pso.md)**
   * Fonctionnement du moteur de GPO et ordre hiérarchique LSDOU.
   * Résolution des conflits (options Appliquer et Bloquer l'héritage).
   * Magasin central des modèles d'administration dans SYSVOL (`PolicyDefinitions`).
   * Pourquoi les GPO de mot de passe liées aux UO échouent et comment les PSO résolvent ce besoin.

5. **[5. Métrologie Système, Analyse de Performances & Supervision](05-metrologie-supervision-evenements.md)**
   * Qualification des performances et définition de la ligne de base (*Baseline*).
   * Les 4 sous-systèmes critiques sous `perfmon` (CPU, RAM, Disque, Réseau) et seuils d'alerte.
   * Ensembles de collecteurs de données programmés.
   * Supervision web moderne sans agent via Windows Admin Center (WAC).
   * Audit et diagnostic dans l'Observateur d'événements (`eventvwr.msc`).

---

> [!NOTE]
> Pour consulter la synthèse des ateliers pratiques réalisés en environnement virtuel VMware (machines CD1, CD2, SRV1, Win-CL1), reportez-vous à la section **[Coin TP — Réalisations en Lab](../tp/README.md)**.
