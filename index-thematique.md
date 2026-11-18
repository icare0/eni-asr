# Index des Notions & Commandes

Ce tableau recense les notions clés, rôles et commandes abordés dans le module actuel d'Administration Windows, avec un accès direct à la fiche correspondante.

| Notion / Commande | Catégorie | Description technique | Accès direct |
| :--- | :--- | :--- | :--- |
| **Active Directory (AD DS)** | Identité | Forêt racine, contrôleurs de domaine CD1 et CD2, DNS intégré | [Active Directory et rôles FSMO](administration-windows/active-directory-fsmo.md) |
| **AGDLP (Méthode de droits)** | Sécurité | Modèle d'attribution : Comptes → Groupes Globaux → Domaine Local → Permissions NTFS | [Stockage SMB et modèle AGDLP](administration-windows/stockage-agdlp.md) |
| **Comptes modèles (Gabarits)** | Annuaire | Modèles Direction, Informatique, Comptabilité avec restrictions d'horaires et duplication | [Gouvernance d'annuaire et OU](administration-windows/gouvernance-annuaire.md) |
| **Consoles MMC (.msc)** | Outils | `dsa.msc`, `gpmc.msc`, `dssite.msc`, `eventvwr.msc`, `perfmon.msc` | [Consoles MMC et raccourcis](memos/consoles-mmc-raccourcis.md) |
| **Délégations de contrôle** | Sécurité AD | Délégation de réinitialisation de mot de passe et gestion d'OU sans privilège Domain Admin | [Stockage SMB et modèle AGDLP](administration-windows/stockage-agdlp.md) |
| **DNS d'entreprise & SRV** | Réseau IP | Zones intégrées Active Directory, enregistrements de service `_ldap._tcp` et `_kerberos._tcp` | [Active Directory et rôles FSMO](administration-windows/active-directory-fsmo.md) |
| **FSMO (5 rôles de maîtres)** | Active Directory | Schéma, Naming, RID, PDC Emulator, Infrastructure, transfert `netdom` et PowerShell | [Active Directory et rôles FSMO](administration-windows/active-directory-fsmo.md) |
| **GPO (Stratégies de groupe)** | Administration | Ordre d'application LSDOU, filtrage de sécurité, préférences d'imprimante, `gpupdate /force` | [Stratégies GPO et sécurité PSO](administration-windows/gpo-securite-pso.md) |
| **Imprimantes partagées** | Périphériques | Port TCP/IP, pilotes 32/64 bits, planification horaire d'impression, déploiement GPO | [Stockage SMB et modèle AGDLP](administration-windows/stockage-agdlp.md) |
| **LDAP (Filtres de recherche)** | Annuaire | Requêtes bitwise pour comptes inactifs (`userAccountControl:1.2.840.113556.1.4.803:=2`) | [Gouvernance d'annuaire et OU](administration-windows/gouvernance-annuaire.md) |
| **Métrologie & Perfmon** | Supervision | Compteurs processeur (% temps CPU), mémoire (pages/s) et disque (latence), seuils d'alerte | [Métrologie et supervision](administration-windows/metrologie-supervision.md) |
| **Partages SMB & Droits NTFS** | Fichiers | Cumul des autorisations, partages masqués (`$`), droits de lecture/modification/contrôle total | [Stockage SMB et modèle AGDLP](administration-windows/stockage-agdlp.md) |
| **PowerShell Administrateur** | Scripting | Cmdlets `ActiveDirectory`, `Storage`, `SmbShare`, gestion d'erreurs `try/catch` | [Cheatsheet PowerShell](memos/powershell-cheatsheet.md) |
| **PSO (Password Settings Objects)** | Sécurité AD | Politiques de mots de passe affinées pour groupes spécifiques (`New-ADFineGrainedPasswordPolicy`) | [Stratégies GPO et sécurité PSO](administration-windows/gpo-securite-pso.md) |
| **Réseau Host-Only** | Virtualisation | Commutateur virtuel isolé VMware pour simuler une infrastructure sans fuite de trafic | [Métrologie et supervision](administration-windows/metrologie-supervision.md) |
| **Sites et sous-réseaux AD** | Topologie | `dssite.msc`, optimisation du trafic WAN, rattachement de poste `nltest /dsgetsite` | [Gouvernance d'annuaire et OU](administration-windows/gouvernance-annuaire.md) |
| **Storage Spaces (Stockage)** | Stockage | Pools de disques virtuels en miroir, points de montage NTFS dans un dossier (`C:\Base`) | [Stockage SMB et modèle AGDLP](administration-windows/stockage-agdlp.md) |
| **SYSVOL (Magasin central)** | GPO | Répertoire `PolicyDefinitions`, modèles d'administration `.admx` et fichiers de langue `.adml` | [Stratégies GPO et sécurité PSO](administration-windows/gpo-securite-pso.md) |
| **Unités d'Organisation (OU)** | Annuaire | Arborescence 3 niveaux Société ASR, séparation Utilisateurs / Ordinateurs | [Gouvernance d'annuaire et OU](administration-windows/gouvernance-annuaire.md) |
| **Windows Admin Center (WAC)** | Outils | Passerelle web HTTPS sur port 443, administration distante WinRM/PowerShell | [Métrologie et supervision](administration-windows/metrologie-supervision.md) |
