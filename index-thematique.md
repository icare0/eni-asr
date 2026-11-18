# Index des Notions & Commandes

Ce tableau recense les notions clés, protocoles et commandes abordés dans les cours, avec un accès direct à la fiche technique correspondante.

| Notion / Commande | Domaine | Description technique | Accès |
| :--- | :--- | :--- | :--- |
| **Active Directory (AD DS)** | Identité | Forêt, domaines, promotion de contrôleurs, réplication multimaître | [Active Directory et rôles FSMO](annee-1/windows-server-ad.md) |
| **AGDLP (Méthode de droits)** | Sécurité | Modèle d'attribution : Comptes → Groupes Globaux → Domaine Local → Permissions NTFS | [Stockage SMB et modèle AGDLP](annee-1/stockage-partages-agdlp.md) |
| **Ansible** | Automatisation | Gestion de parc sans agent (SSH/WinRM), playbooks YAML, principe d'idempotence | [Automatisation Ansible et Docker](annee-2/automatisation-devops.md) |
| **Azure & Entra ID** | Cloud | Synchronisation d'identités locales vers le Cloud via Microsoft Entra Connect (PHS, SSO) | [Virtualisation Datacenter et Azure](annee-2/virtualisation-cloud.md) |
| **Cisco IOS (VLAN & Trunk)** | Réseaux | 802.1Q, sous-interfaces Router-on-a-Stick, Rapid-STP, routage OSPF, ACLs | [Commutation et routage Cisco](annee-2/reseaux-avances-cisco.md) |
| **Comptes modèles (Gabarits)** | Annuaire | Modèles Direction, Informatique, Comptabilité avec restrictions d'horaires et copie | [Gouvernance d'annuaire et OU](annee-1/gestion-annuaire-asr.md) |
| **Consoles MMC (.msc)** | Outils | `dsa.msc`, `gpmc.msc`, `dssite.msc`, `eventvwr.msc`, `perfmon.msc` | [Consoles MMC et raccourcis](memos/consoles-mmc-raccourcis.md) |
| **Délégations de contrôle** | Sécurité AD | Délégation de réinitialisation de mot de passe et gestion d'OU sans privilège Domain Admin | [Stockage SMB et modèle AGDLP](annee-1/stockage-partages-agdlp.md) |
| **DFS (Espaces de noms & DFS-R)** | Résilience | DFS-N chemin UNC unifié virtuel, DFS-R réplication WAN par bloc différentiel (RDC) | [Haute disponibilité et clusters](annee-2/haute-disponibilite.md) |
| **DHCP (Attribution dynamique)** | Réseau IP | Négociation DORA (Discover, Offer, Request, Acknowledge), étendues, baux et réservations MAC | [Fondations réseau et services IP](annee-1/reseau-fondations.md) |
| **Diagnostic Active Directory** | Dépannage | Commandes d'audit et santé : `netdom`, `nltest`, `dcdiag`, `repadmin` | [Diagnostic réseau et Active Directory](memos/commandes-reseau-diagnostic.md) |
| **DNS d'entreprise** | Réseau IP | Zones intégrées Active Directory, enregistrements A, PTR, CNAME et `SRV` (`_ldap._tcp`) | [Fondations réseau et services IP](annee-1/reseau-fondations.md) |
| **Docker (Conteneurisation)** | DevOps | Différence conteneur vs machine virtuelle, Dockerfile, Docker Compose | [Automatisation Ansible et Docker](annee-2/automatisation-devops.md) |
| **Dossier Professionnel (DP)** | Examen | Rédaction des fiches de pratiques réelles pour le Titre Professionnel ASR | [Titre professionnel ASR](annee-2/titre-professionnel-projet.md) |
| **Failover Cluster (WSFC)** | Résilience | Nœuds de cluster Windows Server, quorum, témoin de partage et témoin Cloud | [Haute disponibilité et clusters](annee-2/haute-disponibilite.md) |
| **FSMO (5 rôles de maîtres)** | Active Directory | Schéma, Naming, RID, PDC Emulator, Infrastructure, transfert `netdom` / PowerShell | [Active Directory et rôles FSMO](annee-1/windows-server-ad.md) |
| **GPO (Stratégies de groupe)** | Administration | Ordre LSDOU, filtrage de sécurité, préférences avec ciblage d'élément, `gpupdate /force` | [Stratégies GPO et sécurité PSO](annee-1/gpo-securite.md) |
| **Imprimantes partagées** | Périphériques | Port TCP/IP, pilotes 32/64 bits, planification horaire d'impression, déploiement GPO | [Stockage SMB et modèle AGDLP](annee-1/stockage-partages-agdlp.md) |
| **LAPS (Windows LAPS)** | Sécurité SI | Mot de passe aléatoire unique pour le compte administrateur local de chaque poste | [Sécurité des SI, Tiering et PKI](annee-2/securite-durcissement-pki.md) |
| **LDAP (Filtres de recherche)** | Annuaire | Requêtes bitwise pour comptes inactifs (`userAccountControl:1.2.840.113556.1.4.803:=2`) | [Gouvernance d'annuaire et OU](annee-1/gestion-annuaire-asr.md) |
| **Linux (CLI et administration)** | Systèmes | Arborescence FHS, droits octaux `chmod`, gestionnaire `systemctl`, Bash et SSH | [Administration Linux et Bash](annee-1/linux-fondations.md) |
| **Métrologie & Perfmon** | Supervision | Compteurs processeur, mémoire (pages/s) et disque (latence), seuils d'alerte | [Virtualisation et métrologie](annee-1/environnement-outils.md) |
| **OSPF (Routage dynamique)** | Réseaux | Algorithme Dijkstra, état de liens, zones (Area 0), métrique coût | [Commutation et routage Cisco](annee-2/reseaux-avances-cisco.md) |
| **Partages SMB & Droits NTFS** | Fichiers | Cumul des autorisations, partages masqués (`$`), droits de lecture/modification/contrôle total | [Stockage SMB et modèle AGDLP](annee-1/stockage-partages-agdlp.md) |
| **PKI / AD CS (Certificats)** | Sécurité SI | Autorité racine hors-ligne, autorité subordonnée d'entreprise, auto-enrollment GPO | [Sécurité des SI, Tiering et PKI](annee-2/securite-durcissement-pki.md) |
| **PowerShell Administrateur** | Scripting | Modules `ActiveDirectory`, `Storage`, `SmbShare`, gestion d'erreurs `try/catch` | [Cheatsheet PowerShell](memos/powershell-cheatsheet.md) |
| **PSO (Password Settings Objects)** | Sécurité AD | Politiques de mots de passe affinées pour groupes spécifiques (`New-ADFineGrainedPasswordPolicy`) | [Stratégies GPO et sécurité PSO](annee-1/gpo-securite.md) |
| **RTO & RPO (PCA / PRA)** | Continuité | Objectif de temps de reprise (RTO) et perte de données maximale admissible (RPO) | [Haute disponibilité et clusters](annee-2/haute-disponibilite.md) |
| **Sauvegarde 3-2-1** | Résilience | 3 copies des données, 2 supports différents, 1 copie hors site immuable | [Haute disponibilité et clusters](annee-2/haute-disponibilite.md) |
| **Sites et sous-réseaux AD** | Topologie | `dssite.msc`, optimisation du trafic WAN, rattachement de poste `nltest /dsgetsite` | [Gouvernance d'annuaire et OU](annee-1/gestion-annuaire-asr.md) |
| **Spanning Tree (STP / RSTP)** | Réseaux | Prévention des tempêtes de broadcast en boucle commutée (802.1w) | [Commutation et routage Cisco](annee-2/reseaux-avances-cisco.md) |
| **Storage Spaces (Stockage)** | Stockage | Pools de disques virtuels en miroir, points de montage NTFS dans un dossier | [Stockage SMB et modèle AGDLP](annee-1/stockage-partages-agdlp.md) |
| **SYSVOL (Magasin central)** | GPO | Répertoire `PolicyDefinitions`, modèles d'administration `.admx` et fichiers de langue `.adml` | [Stratégies GPO et sécurité PSO](annee-1/gpo-securite.md) |
| **Tiering Model Active Directory** | Sécurité SI | Cloisonnement strict Tier 0 (Contrôleurs), Tier 1 (Serveurs), Tier 2 (Postes clients) | [Sécurité des SI, Tiering et PKI](annee-2/securite-durcissement-pki.md) |
| **VLANs & 802.1Q** | Réseaux | Segmentation logique niveau 2, ports d'accès, liens Trunk, routage inter-VLAN | [Commutation et routage Cisco](annee-2/reseaux-avances-cisco.md) |
| **Virtualisation vSphere / Proxmox** | Datacenter | Hyperviseurs Type 1 bare-metal, vMotion à chaud, DRS, vSphere HA | [Virtualisation Datacenter et Azure](annee-2/virtualisation-cloud.md) |
| **Windows Admin Center (WAC)** | Outils | Passerelle web HTTPS sur port 443, administration distante WinRM/PowerShell | [Virtualisation et métrologie](annee-1/environnement-outils.md) |
