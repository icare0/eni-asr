# 1ère Année : Administration Système & Réseau On-Premises

Ce dossier regroupe les cours et fiches techniques de la première année du cursus Administrateur Systèmes et Réseaux de l'ENI.

L'objectif est d'acquérir les compétences nécessaires pour installer, configurer, sécuriser et superviser une infrastructure locale d'entreprise sous Windows Server 2022 et Linux.

---

## Sommaire des Cours

1. [Virtualisation et métrologie système](environnement-outils.md) : Maquettage réseau VMware en Host-Only, compteurs de performances processeur/mémoire/disque sous Perfmon, Windows Admin Center sur port 443 et audit d'événements.
2. [Active Directory (AD DS) et rôles FSMO](windows-server-ad.md) : Promotion de contrôleur de domaine en PowerShell (`Install-ADDSForest`), ajout d'un réplica CD2 pour la haute disponibilité, DNS intégré et transfert des 5 rôles FSMO.
3. [Gouvernance d'annuaire et Unités d'Organisation](gestion-annuaire-asr.md) : Conception d'une arborescence d'OU en 3 niveaux, gabarits de comptes modèles, filtres LDAP bitwise et gestion des sites physiques dans `dssite.msc`.
4. [Stockage résilient SMB et modèle AGDLP](stockage-partages-agdlp.md) : Pools de stockage Storage Spaces en miroir, cumul des autorisations NTFS et de partage, matrice des droits AGDLP, délégations de contrôle et serveur d'impression réseau.
5. [Stratégies de groupe (GPO) et sécurité PSO](gpo-securite.md) : Ordre d'application LSDOU, politiques de mots de passe affinées (PSO en PowerShell), magasin central PolicyDefinitions dans SYSVOL et audit client avec `gpresult`.
6. [Fondations réseau et services IP](reseau-fondations.md) : Modèles OSI et TCP/IP, calculs de sous-réseaux IPv4/IPv6 (CIDR/VLSM), attribution dynamique DHCP (DORA) et enregistrements de service DNS (SRV).
7. [Administration Linux et scripting Bash](linux-fondations.md) : Arborescence standard FHS, permissions POSIX octales (`chmod`), gestionnaire `systemctl`, scripts d'automatisation Bash et connexion chiffrée par clés SSH.
