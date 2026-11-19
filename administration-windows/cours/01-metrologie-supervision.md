[← Retour aux Cours](README.md) | [Accueil](../../README.md)

# 1. Métrologie Système, Analyse de Performances & Supervision

Ce chapitre traite de la qualification technique d'un serveur d'entreprise : dimensionnement matériel, métrologie continue des sous-systèmes critiques, administration web moderne sans agent via Windows Admin Center et audit des journaux d'événements.

---

## 1. Principes de la Métrologie Système

La métrologie consiste à mesurer, analyser et corréler l'utilisation des ressources physiques d'un système informatique. En environnement d'infrastructure, elle répond à trois objectifs fondamentaux :

* **Établir une ligne de base (*Baseline*)** : Déterminer la charge normale du serveur en conditions d'exploitation régulières afin d'identifier toute dérive future.
* **Diagnostiquer les goulots d'étranglement (*Bottlenecks*)** : Localiser le sous-système matériel saturé (processeur, mémoire vive, sous-système disque ou interface réseau) à l'origine d'un ralentissement applicatif.
* **Anticiper le dimensionnement (*Capacity Planning*)** : Prévoir l'augmentation de la consommation de ressources et justifier les investissements d'évolution d'infrastructure avant l'incident bloquant.

---

## 2. Analyseur de Performances (`perfmon.msc`)

L'Analyseur de performances (*Performance Monitor*) est l'outil natif de métrologie de Windows Server. Il interroge les compteurs exposés par les pilotes, les composants du noyau Windows et les services applicatifs.

### Les Quatre Sous-Systèmes Critiques et leurs Compteurs

Le tableau ci-dessous synthétise les métriques de référence à auditer en priorité sur un serveur de production :

| Sous-système | Compteur de performance | Unité | Seuil d'alerte en production | Interprétation technique |
| :--- | :--- | :--- | :--- | :--- |
| **Processeur** | `% temps processeur` (*% Processor Time*) | Pourcentage | Moyenne > 80% sur 10 min | Saturation de calcul. La file d'attente des threads processeur s'accumule. |
| **Processeur** | `% temps d’inactivité` (*% Idle Time*) | Pourcentage | Moyenne < 15% | Marge de manœuvre critique restante pour absorber les pics d'activité. |
| **Mémoire** | `Mémoire disponible (Mo)` (*Available MBytes*) | Mégaoctets | < 10% à 15% de la RAM totale | Pression mémoire excessive. Risque d'allocations rejetées ou d'hyper-pagination. |
| **Mémoire** | `Pages/s` (*Memory Pages/sec*) | Événements/s | > 20 à 50 pages/s en continu | Pagination matérielle intense (échanges RAM ↔ fichier d'échange `pagefile.sys`). Indique un déficit de RAM physique. |
| **Disque Physique** | `Temps moyen de lecture (s)` (*Avg. Disk sec/Read*) | Secondes | > 0,015 s (15 ms) | Latence de lecture disque anormale. Problème de saturation I/O ou contrôleur de stockage lent. |
| **Disque Physique** | `Temps moyen d'écriture (s)` (*Avg. Disk sec/Write*) | Secondes | > 0,020 s (20 ms) | Latence d'écriture disque critique. Risque de blocage sur les transactions de bases de données. |
| **Disque Physique** | `% temps disque` (*% Disk Time*) | Pourcentage | > 85% prolongé | Le disque passe la majorité de son temps à traiter des requêtes I/O. |
| **Interface Réseau** | `Octets totaux/s` (*Bytes Total/sec*) | Octets/s | > 70% de la bande passante | Saturation de la liaison réseau physique ou saturation du commutateur virtuel. |

### Automatisation de la Collecte : Ensembles de Collecteurs

Surveiller des compteurs en temps réel via l'interface graphique consomme inutilement des ressources et empêche l'analyse historique. La bonne pratique consiste à configurer un **Ensemble de collecteurs de données défini par l'utilisateur** :

1. **Format des fichiers et emplacement** : Sauvegarde sous un répertoire dédié (ex: `C:\PerfLogs`) au format binaire `.blg` (journal de performances compact et indexable).
2. **Modèle de nommage automatique** : Emploi d'un masque dynamique (ex: `CompteurBase-AAMMJJ`) pour générer automatiquement un sous-dossier horodaté par jour sans écraser les métriques antérieures.
3. **Période d'échantillonnage** :
   * Analyse standard : un point de mesure toutes les 15 à 60 secondes.
   * Diagnostic de crise : un point toutes les 1 à 5 secondes sur une fenêtre restreinte.
4. **Planification et déclencheurs** : Définition d'une heure de démarrage programmée correspondant aux plages de production (ex: 08h30 à 18h30, du lundi au vendredi) avec arrêt automatique à l'issue de la fenêtre d'observation.

---

## 3. Administration Moderne : Windows Admin Center (WAC)

**Windows Admin Center** (WAC) représente l'évolution majeure des outils d'administration Microsoft, venant remplacer progressivement les consoles MMC historiques pour les opérations courantes.

```
┌────────────────────────────────────────────────────────┐
│ Navigateur Web Administrateur (HTTPS / Port 443)       │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ Passerelle Windows Admin Center (Service WAC)          │
│ • Certificat TLS / SSL                                 │
│ • Authentification NTLM / Kerberos                     │
└───────────────────────────┬────────────────────────────┘
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
┌───────────────────────────┐ ┌───────────────────────────┐
│ Contrôleurs AD (CD1, CD2) │ │ Serveur de Stockage SRV1  │
│ PowerShell Remoting       │ │ WMI over WinRM            │
│ Port TCP 5985 / 5986      │ │ Port TCP 5985 / 5986      │
└───────────────────────────┘ └───────────────────────────┘
```

### Principes d'Architecture
* **Architecture sans agent (*Agentless*)** : Aucun agent propriétaire n'est installé sur les serveurs cibles. La passerelle dialogue avec les serveurs via les protocoles natifs de gestion à distance : **PowerShell Remoting** et **WMI over WinRM** (Windows Remote Management, ports TCP 5985 en HTTP et TCP 5986 en HTTPS).
* **Sécurité des échanges** : Toutes les sessions d'administration passent par un canal HTTPS chiffré. La délégation d'authentification s'appuie sur le protocole Kerberos.
* **Centralisation des rôles** : Visualisation globale de l'état de santé des machines (CPU, mémoire, cartes réseau, état du pare-feu Defender, services Windows, mises à jour Windows Update et certificats).

---

## 4. Audit & Diagnostic via l'Observateur d'Événements

L'Observateur d'événements (`eventvwr.msc`) journalise l'ensemble des faits techniques intervenus sur le système d'exploitation et les applications.

### Structure des Journaux Système Windows

* **Journal Application** : Rapports émis par les applications tierces, les composants logiciels installés et les services applicatifs.
* **Journal Sécurité** : Traces relatives aux contrôles d'accès, ouvertures/fermetures de session, élévations de privilèges et accès aux objets audités.
  * *Code 4624* : Ouverture de session réussie (utile pour identifier le type d'ouverture : 2 = interactif local, 3 = réseau/partage SMB, 10 = Bureau à distance RDP).
  * *Code 4625* : Échec d'ouverture de session (analyse des tentatives d'intrusion ou des comptes bloqués).
* **Journal Système** : Événements générés par le système d'exploitation Windows, le chargeur de démarrage, les pilotes de périphériques et les services Windows natifs (arrêts imprévus, pannes de disques, conflits d'adresses IP).

### Méthode de Filtrage Ciblé

Dans un journal contenant plusieurs dizaines de milliers d'entrées, l'administrateur crée des **Vues personnalisées** ou applique des filtres stricts :
* **Filtrage par source** : Cibler précisément le service ou le composant (ex: `MsiInstaller` pour tracer les installations logicielles, `Service Control Manager` pour les changements d'état des services).
* **Filtrage par niveau de criticité** : Isoler uniquement les niveaux *Erreur* et *Critique*.
* **Codes d'événement d'installation Windows Installer (`MsiInstaller`)** :
  * *Code 1033* ou *11707* : Confirmation de fin d'installation réussie avec restitution du nom du produit, de sa version et de son code d'identification unique (GUID).
  * *Code 11708* : Échec d'installation avec motif d'interruption.

---

[← Sommaire des Cours](README.md) | [Atelier pratique associé : TP 01](../tp/tp01-infrastructure-metrologie.md) | [Chapitre suivant : 2. Active Directory & Rôles FSMO →](02-active-directory-fsmo.md)

