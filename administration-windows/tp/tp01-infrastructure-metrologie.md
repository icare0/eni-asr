[← Retour aux TP](README.md) | [Accueil](../../README.md)

# TP 01 — Création de l'Infrastructure & Métrologie Système

Fiche de synthèse de l'atelier pratique n°1 : déploiement de l'environnement virtuel de laboratoire, qualification des performances système et mise en place de la supervision.

---

## 1. Contexte & Périmètre de l'Atelier

* **Machines mobilisées** :
  * `CD1` : Windows Server 2022 (`192.168.21.1/16`)
  * `SRV1` : Windows Server 2022 (`192.168.21.10/16`)
  * `Win-CL1` : Windows 11 Enterprise (`192.168.21.20/16`)
* **Interconnexion** : Réseau virtuel privé isolé VMware Workstation en mode Host-Only (`VMnet1`, sous-réseau `192.168.0.0/16`).
* **Objectif global** : Construire le socle réseau du laboratoire, auditer les ressources matérielles par métrologie et déployer la console web d'administration Windows Admin Center.

---

## 2. Réalisations Techniques Clés

### Métrologie & Collecteur de Performances sur CD1
Configuration d'un **Ensemble de collecteurs de données défini par l'utilisateur** dans l'Analyseur de performances (`perfmon.msc`) :
* **Compteurs audités** :
  * Processeur : `% temps processeur` et `% temps d’inactivité`.
  * Mémoire vive : `Mémoire disponible (Mo)` et `Pages/s` (taux d'échange avec le swap).
  * Disque physique : `Temps moyen de lecture/écriture (s)` et `% temps disque`.
* **Automatisation de la collecte** :
  * Répertoire de destination : `C:\PerfLogs`.
  * Règle de nommage dynamique : `CompteurBase-AAMMJJ` (génère un sous-dossier horodaté par jour sans écrasement).
  * Déclenchement planifié : Lancement différé de 30 minutes, échantillonnage toutes les 15 secondes, durée totale de mesure de 10 minutes, répétition quotidienne du lundi au vendredi.

### Déploiement de Windows Admin Center sur SRV1
* Installation du package d'administration web **Windows Admin Center** (WAC) sur le serveur membre `SRV1` sur le port sécurisé standard HTTPS 443.
* Validation de la passerelle web et configuration de la gestion multi-serveurs à distance sans agent via **WinRM** (Windows Remote Management).

### Audit d'Installation dans l'Observateur d'Événements
* Exploitation de la console `eventvwr.msc` pour tracer et valider l'installation de WAC.
* Journal audité : **Application**.
* Filtre appliqué sur la source **`MsiInstaller`** : Détection des événements de code **1033** et **11707**, confirmant le succès du déploiement avec horodatage, GUID du package et numéro de version exact.

---

## 3. Commandes de Validation & Contrôle

```powershell
# Vérification de l'adressage IP et de la passerelle sur les serveurs
Get-NetIPAddress -InterfaceAlias "Ethernet0" -AddressFamily IPv4 | Select-Object IPAddress, PrefixLength

# Démarrer manuellement l'ensemble de collecteurs de données
logman start "CompteurBase"

# Lister l'état des collecteurs configurés
logman query

# Arrêter la collecte de données
logman stop "CompteurBase"

# Vérifier la présence des fichiers journaux binaires de métrologie (.blg)
Get-ChildItem -Path "C:\PerfLogs" -Recurse -Filter "*.blg"

# Extraire les événements récents d'installation logicielle via PowerShell
Get-WinEvent -FilterHashtable @{LogName='Application'; ProviderName='MsiInstaller'} -MaxEvents 5 | 
    Select-Object TimeCreated, Id, Message | Format-List
```

---

[← Sommaire des TP](README.md) | [Cours théorique associé : Métrologie](../cours/01-metrologie-supervision.md) | [TP suivant : TP 02 →](tp02-domaine-fsmo.md)

