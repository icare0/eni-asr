# 1. Environnement, Virtualisation & Outils de l'Administrateur

L'efficacité d'un administrateur systèmes et réseaux repose sur la maîtrise de ses outils d'administration, de son environnement de maquettage et de ses instruments de métrologie.

---

## 🖥️ 1. Virtualisation & Maquette sous VMware

Pour reproduire fidèlement une architecture d'entreprise sans impacter la production, la virtualisation est incontournable.

### Les modes de mise en réseau VMware Workstation
* **Host-Only (Réseau privé hôte)** : Les machines virtuelles communiquent exclusivement entre elles et avec l'hôte physique. Aucun accès Internet direct n'est autorisé. C'est le mode privilégié pour maqueter un domaine Active Directory sans risque d'interférences avec le réseau local physique (ex: conflits de baux DHCP).
* **NAT (Network Address Translation)** : Les VMs partagent l'adresse IP de l'hôte pour accéder à Internet tout en étant invisibles depuis l'extérieur.
* **Bridged (Pont)** : La VM est branchée directement sur la carte réseau physique de l'hôte et obtient une adresse IP sur le même sous-réseau physique.

---

## 🎛️ 2. Les Outils d'Administration Windows

### Le Gestionnaire de Serveur (Server Manager)
Le tableau de bord central sous Windows Server permet d'ajouter des rôles et fonctionnalités, de surveiller l'état des services et de lancer les outils d'administration.

### Les Consoles MMC (Microsoft Management Console)
Les consoles `.msc` sont les briques modulaires historiques de Windows :
* `compmgmt.msc` : Gestion de l'ordinateur (Disques, Dossiers partagés, Utilisateurs locaux).
* `eventvwr.msc` : Observateur d'événements.
* `perfmon.msc` : Analyseur de performances.
* `services.msc` : Gestionnaire des services système.

### Windows Admin Center (WAC)
Solution web moderne et légère développée par Microsoft pour administrer des serveurs Windows Server et clusters depuis un simple navigateur :
* Fonctionne en architecture passerelle (écoute sur le port HTTPS 443 ou 6516).
* Permet de gérer les machines distantes via PowerShell Remoting et WMI over WinRM.
* Fournit une interface unifiée pour le monitoring, le pare-feu, les mises à jour Windows Update, le stockage et les fichiers sans nécessiter de session RDP complète.

---

## 📊 3. Métrologie & Analyse de Performances (Perfmon)

L'analyseur de performances (`perfmon.msc`) permet de diagnostiquer les goulots d'étranglement matériels et logiciels.

### Les Compteurs Clés à Surveiller

| Sous-système | Compteur | Signification & Seuil d'alerte |
| :--- | :--- | :--- |
| **Processeur** | `% temps processeur` | Taux d'occupation global du CPU. Alerte si > 80% sur une période prolongée. |
| **Processeur** | `% temps d’inactivité` | Temps disponible restant du processeur. |
| **Mémoire** | `Mémoire disponible (Mo)` | Quantité de RAM immédiatement disponible pour les processus. |
| **Mémoire** | `Pages/s` | Échanges entre la RAM physique et le fichier d'échange (swap). Une valeur élevée persistante indique un manque critique de mémoire vive. |
| **Disque physique** | `Temps moyen de lecture / écriture (s)` | Latence des I/O disque. Doit rester inférieure à 15-20 ms. |
| **Disque physique** | `% temps disque` | Pourcentage de temps où le disque est occupé à traiter des requêtes. |

### Les Ensembles de Collecteurs de Données
Plutôt qu'une observation ponctuelle à l'écran, l'administrateur crée un ensemble de collecteurs :
* **Planification différée** : Déclencher la collecte lors des pics d'activité (ex: 08h30 - 11h30).
* **Nommage dynamique** : Utiliser les variables d'horodatage (`CompteurBase-AAMMJJ`) pour archiver automatiquement les fichiers de traces (`.blg` ou `.csv`) dans `C:\PerfLogs`.

---

## 🔍 4. Audit & Observateur d'Événements (Event Viewer)

Accessible via `eventvwr.msc`, il consigne l'ensemble des journaux d'activité :
* **Journal Application** : Rapports des logiciels installés, services métiers et installateurs (ex. source `MsiInstaller` pour tracer la date, l'heure et la version exacte d'un package logiciel déployé).
* **Journal Sécurité** : Tentatives de connexion (succès/échecs 4624/4625), modifications d'annuaire et élévations de privilèges.
* **Journal Système** : Événements générés par le noyau Windows, pilotes matériels et services de base du système d'exploitation.
