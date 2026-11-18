# 1. Virtualisation d'Infrastructure, Métrologie & Supervision Système

Ce dossier technique constitue la base opérationnelle de l'administrateur : savoir concevoir un laboratoire d'infrastructure virtualisé, superviser les performances en continu et auditer les événements du système d'exploitation.

---

## 🖥️ 1. Architecture de Laboratoire & Virtualisation (VMware)

Pour simuler fidèlement une infrastructure d'entreprise (contrôleurs de domaine, serveurs membres, postes clients) sans impacter les équipements de production ni risquer des conflits de baux DHCP sur le réseau physique, la virtualisation est incontournable.

```
                     [ Réseau Virtuel Privé : Host-Only ]
                           (Plage IP : 192.168.0.0/16)
                                        │
    ┌───────────────────┬───────────────┴───────────────┬───────────────────┐
    │                   │                               │                   │
    ▼                   ▼                               ▼                   ▼
┌──────────────┐ ┌──────────────┐                ┌──────────────┐    ┌──────────────┐
│     CD1      │ │     CD2      │                │     SRV1     │    │   Win-CL1    │
│ Contrôleur 1 │ │ Contrôleur 2 │                │Serveur Membre│    │ Client Admin │
│ 192.168.21.1 │ │ 192.168.21.2 │                │192.168.21.10 │    │192.168.21.20 │
└──────────────┘ └──────────────┘                └──────────────┘    └──────────────┘
```

### Modes de Mise en Réseau VMware Workstation
* **Host-Only (Réseau privé hôte)** : Les machines virtuelles sont connectées sur un commutateur virtuel isolé (`VMnet1`). Elles communiquent exclusivement entre elles et avec la machine hôte. Aucun paquet n'est routé vers Internet, ce qui garantit une isolation totale.
* **NAT (Network Address Translation)** : Les VMs partagent l'adresse IP de l'hôte pour accéder à Internet tout en étant invisibles depuis l'extérieur.
* **Bridged (Pont)** : La VM est branchée directement sur la carte réseau physique de l'hôte et obtient une adresse IP sur le même sous-réseau physique.

---

## 📊 2. Métrologie & Analyse de Performances (`perfmon.msc`)

En entreprise, l'analyseur de performances (*Performance Monitor*) permet de qualifier la charge d'un serveur avant et après la mise en production, d'identifier les goulots d'étranglement et de dimensionner les ressources matérielles.

### Les Compteurs Essentiels à Auditer

| Sous-Système | Compteur de Performance | Rôle & Seuil d'Alerte en Production |
| :--- | :--- | :--- |
| **Processeur** | `% temps processeur` (*% Processor Time*) | Mesure le temps pendant lequel le processeur exécute des threads actifs. **Seuil critique si > 80%** en moyenne prolongée. |
| **Processeur** | `% temps d’inactivité` (*% Idle Time*) | Pourcentage de disponibilité restante pour absorber une surcharge ponctuelle. |
| **Mémoire** | `Mémoire disponible (Mo)` (*Available MBytes*) | Quantité de mémoire vive physique immédiatement disponible. Doit rester supérieure à 10-15% de la RAM totale. |
| **Mémoire** | `Pages/s` (*Memory Pages/sec*) | Nombre de pages lues ou écrites sur le disque pour résoudre les défauts de page matériels (échanges RAM ↔ swap). Une valeur élevée persistante (> 20-50 pages/s) indique un manque critique de RAM physique. |
| **Disque Physique** | `Temps moyen de lecture / écriture (s)` | Latence d'accès aux blocs disque. Doit impérativement rester **inférieure à 15-20 ms**. Au-delà, les applications ralentissent fortement. |
| **Disque Physique** | `% temps disque` (*% Disk Time*) | Pourcentage de temps pendant lequel le disque est sollicité par des I/O. |

### Automatisation d'une Collecte Programmée de Performances
Plutôt que d'observer les compteurs en temps réel, l'administrateur configure un **Ensemble de collecteurs de données défini par l'utilisateur** :
1. **Emplacement de stockage** : Racine dédiée, par exemple `C:\PerfLogs`.
2. **Nommage dynamique automatique** : `CompteurBase-AAMMJJ` (permet de générer un sous-dossier horodaté par jour sans risque d'écrasement).
3. **Planification et échantillonnage** :
   * Lancement différé (ex: début de journée à 08h30).
   * Intervalle d'échantillonnage : 15 secondes.
   * Durée de collecte : 10 minutes pendant les pics d'activité.
   * Répétition programmée du lundi au vendredi.

---

## 🌐 3. Administration Moderne avec Windows Admin Center (WAC)

**Windows Admin Center** est l'interface web de gestion de nouvelle génération développée par Microsoft pour administrer des serveurs Windows Server et des clusters sans recourir à des sessions RDP lourdes.

### Caractéristiques d'Architecture
* **Passerelle Web légère** : S'installe sous forme d'un service Windows écoutant sur le port sécurisé **HTTPS 443** (ou port personnalisé 6516).
* **Communication sans agent** : Pilote les serveurs distants via les protocoles standards **PowerShell Remoting** et **WMI over WinRM** (Windows Remote Management).
* **Sécurité & Accès** : Validation des sessions via des certificats SSL/TLS et prise en charge de l'authentification multifacteur (MFA).

### Cas d'Usage Opérationnels
* **Gestion du Bureau à distance (RDP)** : Activation ou désactivation du service RDP à distance en un clic.
* **Supervision multi-serveurs** : Ajout centralisé de l'ensemble des contrôleurs de domaine (CD1, CD2) et serveurs membres (SRV1) dans une console unique.
* **Gestion des pare-feu et mises à jour** : Visualisation instantanée de l'état du pare-feu Windows Defender et application des correctifs Windows Update.

---

## 🔍 4. Audit & Diagnostic Système via l'Observateur d'Événements

Accessible via la commande `eventvwr.msc`, l'Observateur d'événements consigne l'ensemble des activités système, applicatives et de sécurité.

### Structure des Journaux Windows Principaux
* **Journal Application** : Rapports émis par les applications installées et les services métiers.
* **Journal Sécurité** : Traces des ouvertures de session (succès code `4624`, échecs code `4625`), modifications d'annuaire et élévations de privilèges.
* **Journal Système** : Événements générés par le système d'exploitation Windows, le noyau et les pilotes de périphériques.

### Cas Pratique : Auditer un Déploiement Logiciel
Pour vérifier la conformité d'une installation logicielle déployée via un package Windows Installer (`.msi`) :
1. Ouvrir le journal **Application**.
2. Appliquer un filtre d'affichage sur la source d'événement **`MsiInstaller`**.
3. Les événements de code `1033` ou `11707` confirment le succès de l'installation et précisent :
   * La date et l'heure exactes de l'opération.
   * Le code GUID unique du produit.
   * Le numéro de version précis déployé.
