[← Retour à l'Accueil](README.md)

# Index des Notions & Commandes

Ce tableau recense les notions clés, protocoles, architectures et commandes abordés dans le module d'Administration Windows, avec un accès direct vers le cours théorique ou la fiche de TP correspondante.

| Notion / Commande | Catégorie | Description technique | Accès direct |
| :--- | :--- | :--- | :--- |
| **Active Directory (AD DS)** | Identité | Forêt racine, contrôleurs de domaine CD1 et CD2, DNS intégré | [Cours Active Directory](administration-windows/cours/02-active-directory-fsmo.md) |
| **AGDLP (Méthode d'accès)** | Sécurité | Modèle d'attribution : Comptes → Groupes Globaux → Domaine Local → Permissions NTFS | [Cours Modèle AGDLP](administration-windows/cours/04-stockage-agdlp.md) / [TP 04](administration-windows/tp/tp04-stockage-partages-agdlp.md) |
| **Comptes modèles (Gabarits)** | Annuaire | Modèles Direction, Informatique, Comptabilité avec restrictions d'horaires et duplication | [Cours UO & Identités](administration-windows/cours/03-gouvernance-annuaire.md) / [TP 03](administration-windows/tp/tp03-annuaire-societe-asr.md) |
| **Consoles MMC (.msc)** | Outils | `dsa.msc`, `gpmc.msc`, `dssite.msc`, `eventvwr.msc`, `perfmon.msc` | [Consoles MMC et raccourcis](memos/consoles-mmc-raccourcis.md) |
| **Délégations de contrôle** | Sécurité AD | Délégation de réinitialisation de mot de passe et gestion d'OU sans privilège Domain Admin | [Cours Stockage & Droits](administration-windows/cours/04-stockage-agdlp.md) / [TP 04](administration-windows/tp/tp04-stockage-partages-agdlp.md) |
| **DNS d'entreprise & SRV** | Réseau IP | Zones intégrées Active Directory, enregistrements de service `_ldap._tcp` et `_kerberos._tcp` | [Cours Active Directory](administration-windows/cours/02-active-directory-fsmo.md) |
| **FSMO (5 rôles de maîtres)** | Active Directory | Schéma, Naming, RID, PDC Emulator, Infrastructure, transfert PowerShell | [Cours Rôles FSMO](administration-windows/cours/02-active-directory-fsmo.md) / [TP 02](administration-windows/tp/tp02-domaine-fsmo.md) |
| **GPO (Stratégies de groupe)** | Administration | Ordre d'application LSDOU, filtrage de sécurité, préférences d'imprimante, `gpupdate /force` | [Cours Stratégies GPO](administration-windows/cours/05-gpo-securite-pso.md) / [TP 05](administration-windows/tp/tp05-gpo-securite-pso.md) |
| **Imprimantes partagées** | Périphériques | Port TCP/IP, pilotes 32/64 bits, planification horaire d'impression, déploiement GPO | [TP 04 Stockage & Imprimante](administration-windows/tp/tp04-stockage-partages-agdlp.md) |
| **Métrologie & Perfmon** | Supervision | Compteurs processeur (% temps CPU), mémoire (pages/s) et disque (latence), collecteurs automatiques | [Cours Métrologie](administration-windows/cours/01-metrologie-supervision.md) / [TP 01](administration-windows/tp/tp01-infrastructure-metrologie.md) |
| **Partages SMB & Droits NTFS** | Fichiers | Cumul des autorisations, partages masqués (`$`), droits de lecture/modification/contrôle total | [Cours Stockage & SMB](administration-windows/cours/04-stockage-agdlp.md) / [TP 04](administration-windows/tp/tp04-stockage-partages-agdlp.md) |
| **PowerShell Administrateur** | Scripting | Cmdlets `ActiveDirectory`, `Storage`, `SmbShare`, diagnostics système | [Cheatsheet PowerShell](memos/powershell-cheatsheet.md) |
| **PSO (Password Settings Objects)** | Sécurité AD | Politiques de mots de passe affinées pour groupes spécifiques (`New-ADFineGrainedPasswordPolicy`) | [Cours Sécurité PSO](administration-windows/cours/05-gpo-securite-pso.md) / [TP 05](administration-windows/tp/tp05-gpo-securite-pso.md) |
| **Réseau Host-Only** | Virtualisation | Commutateur virtuel isolé VMware pour simuler une infrastructure sans fuite de trafic | [Architecture Lab](administration-windows/tp/README.md) |
| **Sites et sous-réseaux AD** | Topologie | `dssite.msc`, optimisation du trafic WAN, rattachement de poste `nltest /dsgetsite` | [Cours Topologie Sites](administration-windows/cours/03-gouvernance-annuaire.md) / [TP 03](administration-windows/tp/tp03-annuaire-societe-asr.md) |
| **Storage Spaces (Stockage)** | Stockage | Pools de disques virtuels en miroir, points de montage NTFS dans un dossier (`C:\Base`) | [Cours Stockage Résilient](administration-windows/cours/04-stockage-agdlp.md) / [TP 04](administration-windows/tp/tp04-stockage-partages-agdlp.md) |
| **SYSVOL (Magasin central)** | GPO | Répertoire `PolicyDefinitions`, modèles d'administration `.admx` et fichiers de langue `.adml` | [Cours SYSVOL & ADMX](administration-windows/cours/05-gpo-securite-pso.md) / [TP 05](administration-windows/tp/tp05-gpo-securite-pso.md) |
| **Unités d'Organisation (OU)** | Annuaire | Arborescence 3 niveaux Société ASR, séparation Utilisateurs / Ordinateurs | [Cours UO & Annuaire](administration-windows/cours/03-gouvernance-annuaire.md) / [TP 03](administration-windows/tp/tp03-annuaire-societe-asr.md) |
| **Windows Admin Center (WAC)** | Outils | Passerelle web HTTPS sur port 443, administration distante WinRM/PowerShell | [Cours WAC](administration-windows/cours/01-metrologie-supervision.md) / [TP 01](administration-windows/tp/tp01-infrastructure-metrologie.md) |
