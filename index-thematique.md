# 🔍 Index Thématique & Moteur de Recherche Rapide

> **Trouvez instantanément ce que vous cherchez** parmi tous les cours, travaux pratiques, scripts PowerShell et commandes de vos 2 années d'ASR à l'ENI.

---

## ⚡ Index Alphabétique des Notions Clés

| Notion / Mot-clé | Domaine | Où le trouver dans le cursus ? | Accès Direct |
| :--- | :--- | :--- | :--- |
| **Active Directory (AD DS)** | Identité | Architecture forêt/domaine, contrôleurs CD1/CD2, réplication | [Cours 2 : Architecture AD](/annee-1/windows-server-ad.md) • [TP 02](/ateliers/tp02-domaine-ad-fsmo.md) |
| **AGDLP (Méthode de droits)** | Sécurité & Fichiers | Comptes → Groupes Globaux → Domaine Local → Permissions NTFS | [Cours 4 : Modèle AGDLP](/annee-1/stockage-partages-agdlp.md) • [TP 04](/ateliers/tp04-ressources-agdlp.md) |
| **Ansible** | DevOps | Automatisation de parc hétérogène (Linux/Windows), playbooks YAML | [Année 2 : Automatisation & DevOps](/annee-2/automatisation-devops.md) |
| **Azure & Entra ID** | Cloud | Synchronisation d'annuaire hybride via Entra Connect (PHS, SSO) | [Année 2 : Cloud Hybride](/annee-2/virtualisation-cloud.md) |
| **Cisco (VLAN & Routage)** | Réseaux | 802.1Q Trunk, Router-on-a-Stick, Rapid-STP, OSPF, ACLs | [Année 2 : Réseaux Avancés Cisco](/annee-2/reseaux-avances-cisco.md) |
| **Comptes Modèles & Gabarits** | Annuaire | Modèles Direction, Info, Compta, contraintes horaires, duplication | [Cours 3 : Société ASR](/annee-1/gestion-annuaire-asr.md) • [TP 03](/ateliers/tp03-gestion-societe-asr.md) |
| **Consoles MMC (.msc)** | Outils | `dsa.msc`, `dssite.msc`, `gpmc.msc`, `eventvwr`, raccourcis RDP | [Mémo Consoles MMC & Raccourcis](/memos/consoles-mmc-raccourcis.md) |
| **Délégations de Contrôle** | Administration | Déléguer la réinitialisation de mot de passe sans être Domain Admin | [Cours 4 : Stockage & Partages](/annee-1/stockage-partages-agdlp.md) • [TP 04](/ateliers/tp04-ressources-agdlp.md) |
| **DFS (Espaces de noms & Réplication)** | Résilience | DFS-N chemin UNC virtuel unifié, DFS-R réplication WAN par bloc RDC | [Année 2 : Haute Disponibilité](/annee-2/haute-disponibilite.md) |
| **DHCP (Service d'adressage)** | Réseau IP | Négociation DORA, étendues, réservations MAC, options 003/006/015 | [Cours 6 : Fondations Réseau](/annee-1/reseau-fondations.md) |
| **Diagnostic Réseau & AD** | Dépannage | `netdom query fsmo`, `nltest /dsgetsite`, `repadmin`, `Test-NetConnection` | [Mémo Diagnostic Réseau & AD](/memos/commandes-reseau-diagnostic.md) |
| **DNS d'Entreprise** | Réseau IP | Enregistrements A, PTR, CNAME et surtout `SRV` (`_ldap._tcp`) pour AD | [Cours 6 : Fondations Réseau](/annee-1/reseau-fondations.md) • [TP 02](/ateliers/tp02-domaine-ad-fsmo.md) |
| **Docker (Conteneurisation)** | DevOps | Conteneurs vs Machines Virtuelles, Dockerfile, Docker Compose | [Année 2 : Automatisation & DevOps](/annee-2/automatisation-devops.md) |
| **Dossier Professionnel (DP)** | Examen | Rédaction des exemples de pratique professionnelle pour le titre ASR | [Année 2 : Préparation Titre Pro](/annee-2/titre-professionnel-projet.md) |
| **Failover Cluster (WSFC)** | Haute Dispo | Nœuds de cluster, gestion du quorum, témoins de partage et Cloud | [Année 2 : Haute Disponibilité](/annee-2/haute-disponibilite.md) |
| **FSMO (5 Rôles de Maîtres)** | Active Directory | Schéma, Naming, RID, PDC, Infrastructure, transfert `netdom` / PowerShell | [Cours 2 : Rôles FSMO](/annee-1/windows-server-ad.md) • [TP 02](/ateliers/tp02-domaine-ad-fsmo.md) |
| **GPO (Stratégies de Groupe)** | Sécurité | Ordre LSDOU, filtrage sécurité, ciblage par élément, `gpupdate /force` | [Cours 5 : GPO & Sécurité](/annee-1/gpo-securite.md) • [TP 05](/ateliers/tp05-gpo-securite-avancee.md) |
| **Imprimantes Réseau** | Périphériques | Port TCP/IP, pilotes 32/64 bits, planification horaire, déploiement GPO | [Cours 4 : Partages & Impression](/annee-1/stockage-partages-agdlp.md) • [TP 04](/ateliers/tp04-ressources-agdlp.md) |
| **LAPS (Windows LAPS)** | Sécurité SI | Mot de passe aléatoire unique pour chaque administrateur local de poste | [Année 2 : Sécurité des SI & PKI](/annee-2/securite-durcissement-pki.md) |
| **LDAP (Filtres de recherche)** | Annuaire | Requêtes bitwise `userAccountControl:1.2.840.113556.1.4.803:=2` | [Cours 3 : Société ASR](/annee-1/gestion-annuaire-asr.md) • [TP 03](/ateliers/tp03-gestion-societe-asr.md) |
| **Linux (Administration & CLI)** | Systèmes | Arborescence FHS, droits octaux `chmod`, services `systemctl`, Bash, SSH | [Cours 7 : Découverte Linux](/annee-1/linux-fondations.md) |
| **Métrologie & Performances** | Supervision | Compteurs Perfmon CPU/RAM/Disque, planification, seuils d'alerte | [Cours 1 : Outils & Métrologie](/annee-1/environnement-outils.md) • [TP 01](/ateliers/tp01-infrastructure-metrologie.md) |
| **Observateur d'Événements** | Audit | Journaux Système/Application/Sécurité, filtre source `MsiInstaller` | [Cours 1 : Outils & Métrologie](/annee-1/environnement-outils.md) • [TP 01](/ateliers/tp01-infrastructure-metrologie.md) |
| **OSPF (Routage dynamique)** | Réseaux | Algorithme Dijkstra, état de liens, zones (Area 0), métrique coût | [Année 2 : Réseaux Avancés Cisco](/annee-2/reseaux-avances-cisco.md) |
| **Partages SMB & Droits NTFS** | Stockage | Règle du cumul le plus restrictif, partages administratifs masqués `$` | [Cours 4 : Stockage & Partages](/annee-1/stockage-partages-agdlp.md) • [TP 04](/ateliers/tp04-ressources-agdlp.md) |
| **PKI / AD CS (Certificats)** | Sécurité SI | Autorité racine hors-ligne, autorité subordonnée, auto-enrollment GPO | [Année 2 : Sécurité des SI & PKI](/annee-2/securite-durcissement-pki.md) |
| **PowerShell Administrateur** | Scripting | Cmdlets `ActiveDirectory`, `Storage`, `SmbShare`, `Invoke-Command` | [Mémo PowerShell Sysadmin](/memos/powershell-cheatsheet.md) |
| **PSO (Password Settings Objects)** | Sécurité AD | Politiques de mots de passe affinées pour groupes spécifiques (`New-ADFineGrainedPasswordPolicy`) | [Cours 5 : GPO & Sécurité](/annee-1/gpo-securite.md) • [TP 05](/ateliers/tp05-gpo-securite-avancee.md) |
| **RTO & RPO (PCA / PRA)** | Résilience | Objectifs de temps de reprise (RTO) et de perte de données admissible (RPO) | [Année 2 : Haute Disponibilité](/annee-2/haute-disponibilite.md) |
| **Sauvegarde 3-2-1** | Résilience | 3 copies, 2 supports différents, 1 copie hors-site immuable (WORM) | [Année 2 : Haute Disponibilité](/annee-2/haute-disponibilite.md) |
| **Sites & Sous-réseaux AD** | Topologie | `dssite.msc`, optimisation du trafic WAN, rattachement `nltest /dsgetsite` | [Cours 3 : Société ASR](/annee-1/gestion-annuaire-asr.md) • [TP 03](/ateliers/tp03-gestion-societe-asr.md) |
| **Spanning Tree (STP / RSTP)** | Réseaux | Prévention des tempêtes de broadcast en boucle commutée (802.1w) | [Année 2 : Réseaux Avancés Cisco](/annee-2/reseaux-avances-cisco.md) |
| **Storage Spaces (Espaces de stockage)** | Stockage | Pools de stockage résilients en Miroir/Parité, disques virtuels | [Cours 4 : Stockage & Partages](/annee-1/stockage-partages-agdlp.md) • [TP 04](/ateliers/tp04-ressources-agdlp.md) |
| **SYSVOL (Magasin Central)** | GPO | Répertoire `PolicyDefinitions`, modèles `.admx` et localisations `fr-FR\.adml` | [Cours 5 : GPO & Sécurité](/annee-1/gpo-securite.md) • [TP 05](/ateliers/tp05-gpo-securite-avancee.md) |
| **Tiering Model Active Directory** | Sécurité SI | Séparation stricte Tier 0 (Contrôleurs), Tier 1 (Serveurs), Tier 2 (Clients) | [Année 2 : Sécurité des SI & PKI](/annee-2/securite-durcissement-pki.md) |
| **VLANs & 802.1Q** | Réseaux | Segmentation logique L2, ports Access, ports Trunk, Router-on-a-Stick | [Année 2 : Réseaux Avancés Cisco](/annee-2/reseaux-avances-cisco.md) |
| **VMware vSphere & Proxmox** | Virtualisation | Hyperviseurs Type 1 Bare-Metal, vCenter, vMotion à chaud, DRS, HA | [Année 2 : Virtualisation Datacenter](/annee-2/virtualisation-cloud.md) |
| **Windows Admin Center (WAC)** | Outils | Passerelle web HTTPS port 443, administration distante WinRM/PowerShell | [Cours 1 : Outils & Métrologie](/annee-1/environnement-outils.md) • [TP 01](/ateliers/tp01-infrastructure-metrologie.md) |
