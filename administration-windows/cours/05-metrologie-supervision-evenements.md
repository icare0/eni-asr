[← Retour au Sommaire des Cours](README.md) | [Accueil](../../README.md)

# 5. Métrologie Système, Analyse de Performances & Supervision

Ce cours aborde la gestion opérationnelle et le maintien en conditions opérationnelles d'une infrastructure Windows Server : mesurer la santé des sous-systèmes matériels, programmer des collectes de métrologie, superviser à distance via Windows Admin Center et auditer les événements critiques du système d'exploitation.

---

## 1. Principes de la Métrologie en Infrastructure

Une fois les services d'infrastructure (Active Directory, serveurs de fichiers, DNS) mis en production, l'administrateur doit garantir leur disponibilité et leurs performances.

La métrologie répond à trois besoins opérationnels :
* **Définir la ligne de base (*Baseline*)** : Connaître le niveau d'utilisation normal des ressources (CPU, RAM, Disque, Réseau) en heures ouvrées normales, afin d'identifier toute anomalie ou dérive future.
* **Isoler les goulots d'étranglement (*Bottlenecks*)** : Lorsqu'une application ou un partage ralentit, identifier avec certitude le composant matériel saturé (manque de mémoire vive, disques trop lents ou processeur surchargé).
* **Anticiper les investissements (*Capacity Planning*)** : Prévoir l'augmentation des besoins avant que le serveur n'atteigne le point de rupture.

---

## 2. Analyseur de Performances (`perfmon.msc`)

L'Analyseur de performances (*Performance Monitor*) est l'outil natif de Windows Server pour interroger les compteurs d'exploitation exposés par le noyau Windows, le matériel et les services.

### Les Quatre Sous-Systèmes Critiques et leurs Compteurs

| Sous-système | Compteur de référence | Unité | Seuil d'alerte en production | Interprétation technique |
| :--- | :--- | :--- | :--- | :--- |
| **Processeur** | `% temps processeur` (*% Processor Time*) | Pourcentage | Moyenne > 80% sur 10 min | Saturation de calcul. La file d'attente des threads processeur s'accumule. |
| **Processeur** | `% temps d’inactivité` (*% Idle Time*) | Pourcentage | Moyenne < 15% | Marge de manœuvre critique restante pour absorber les pics d'activité imprévus. |
| **Mémoire** | `Mémoire disponible (Mo)` (*Available MBytes*) | Mégaoctets | < 10% à 15% de la RAM totale | Pression mémoire excessive. Risque de ralentissement généralisé ou de rejets d'allocation. |
| **Mémoire** | `Pages/s` (*Memory Pages/sec*) | Événements/s | > 20 à 50 pages/s en continu | Pagination matérielle intense (échanges excessifs entre la RAM et le fichier d'échange `pagefile.sys`). Indique un déficit de RAM physique. |
| **Disque Physique** | `Temps moyen de lecture (s)` (*Avg. Disk sec/Read*) | Secondes | > 0,015 s (15 ms) | Latence de lecture disque anormale. Problème de saturation I/O ou contrôleur de stockage lent. |
| **Disque Physique** | `Temps moyen d'écriture (s)` (*Avg. Disk sec/Write*) | Secondes | > 0,020 s (20 ms) | Latence d'écriture disque critique. Risque de blocage sur les transactions de bases de données ou les écritures de fichiers. |
| **Disque Physique** | `% temps disque` (*% Disk Time*) | Pourcentage | > 85% prolongé | Le disque passe la majorité de son temps à traiter des requêtes I/O au lieu d'être disponible. |
| **Interface Réseau** | `Octets totaux/s` (*Bytes Total/sec*) | Octets/s | > 70% de la bande passante | Saturation de la carte réseau physique ou du commutateur virtuel (vSwitch). |

---

## 3. Automatisation de la Métrologie : Ensembles de Collecteurs

Observer des compteurs en temps réel sur un écran ne permet pas d'analyser l'historique d'un incident intervenu la nuit ou pendant un pic d'activité. La bonne pratique consiste à configurer un **Ensemble de collecteurs de données défini par l'utilisateur** :

1. **Format des données** : Sauvegarde sous un répertoire dédié (ex: `C:\PerfLogs`) au format binaire indexé `.blg`.
2. **Nommage dynamique automatique** : Utilisation d'un masque dynamique (ex: `CompteurBase-AAMMJJ`) pour générer automatiquement un sous-dossier horodaté par jour sans écraser les mesures antérieures.
3. **Fréquence d'échantillonnage** :
   * En exploitation normale : un relevé toutes les 15 à 60 secondes.
   * En diagnostic de crise : un relevé toutes les 1 à 5 secondes.
4. **Planification** : Lancement programmé aux heures d'ouverture de l'entreprise (ex: de 08h00 à 19h00, du lundi au vendredi) avec arrêt automatique à l'issue de la journée.

```powershell
# Démarrer manuellement une collecte programmée
logman start "CompteurBase"

# Consulter l'état des collecteurs actifs
logman query

# Arrêter la collecte
logman stop "CompteurBase"
```

---

## 4. Administration Moderne : Windows Admin Center (WAC)

**Windows Admin Center** (WAC) est la console web moderne développée par Microsoft pour administrer des serveurs Windows Server et des clusters sans recourir à des sessions Bureau à distance (RDP) lourdes.

```
┌────────────────────────────────────────────────────────┐
│ Navigateur Web Administrateur (HTTPS / Port 443)       │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ Passerelle Windows Admin Center (Service WAC)          │
│ • Certificat TLS / SSL                                 │
│ • Authentification sécurisée NTLM / Kerberos           │
└───────────────────────────┬────────────────────────────┘
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
┌───────────────────────────┐ ┌───────────────────────────┐
│ Contrôleurs Active Direct.│ │ Serveurs de Fichiers      │
│ PowerShell Remoting       │ │ WMI over WinRM            │
│ Port TCP 5985 / 5986      │ │ Port TCP 5985 / 5986      │
└───────────────────────────┘ └───────────────────────────┘
```

### Caractéristiques d'Architecture
* **Architecture sans agent (*Agentless*)** : Aucun agent logiciel n'est installé sur les serveurs administrés. La passerelle communique avec les serveurs distants via les protocoles de gestion standards : **PowerShell Remoting** et **WMI over WinRM** (Windows Remote Management, ports TCP 5985 en HTTP et TCP 5986 en HTTPS).
* **Supervision multi-serveurs** : Visualisation centralisée de l'ensemble des serveurs dans une interface unifiée (état CPU, mémoire, pare-feu, services en cours, mises à jour Windows Update et certificats).

---

## 5. Audit & Diagnostic via l'Observateur d'Événements

Accessible via la commande `eventvwr.msc`, l'Observateur d'événements consigne l'ensemble des faits techniques intervenus sur le système et les applications.

### Les Trois Journaux Système Principaux
* **Journal Application** : Rapports émis par les logiciels tiers, les composants logiciels installés et les services métiers.
* **Journal Sécurité** : Traces relatives aux contrôles d'accès, ouvertures/fermetures de session, élévations de privilèges et accès aux fichiers audités.
  * *Code 4624* : Ouverture de session réussie (avec type d'ouverture : 2 = interactif local, 3 = réseau/partage SMB, 10 = RDP).
  * *Code 4625* : Échec d'ouverture de session (analyse des tentatives d'intrusion ou des comptes verrouillés).
* **Journal Système** : Événements générés par le système d'exploitation Windows, le noyau, les pilotes et les services Windows (pannes matérielles, arrêts imprévus, coupures réseau).

### Méthode de Filtrage Ciblé
Dans un journal contenant des dizaines de milliers d'entrées, l'administrateur crée des **Vues personnalisées** :
* **Filtrage par niveau** : Sélectionner uniquement les niveaux *Avertissement*, *Erreur* et *Critique*.
* **Filtrage par source** : Cibler le composant concerné (ex: `MsiInstaller` pour tracer le déploiement de packages logiciels, ou `Service Control Manager` pour surveiller les plantages de services système).

---

[← Chapitre précédent : 4. Stratégies de Groupe (GPO) & Sécurité PSO](04-gpo-securite-pso.md) | [Atelier pratique associé : TP 01](../tp/tp01-infrastructure-metrologie.md) | [Sommaire général des Cours](README.md)
