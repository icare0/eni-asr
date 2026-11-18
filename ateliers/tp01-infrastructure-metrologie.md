# Atelier 01 — Création de l’Infrastructure & Métrologie

Cet atelier pose les fondations matérielles et de supervision de notre maquette d'apprentissage.

---

## 🎯 Objectifs Opérationnels
1. Déployer et interconnecter 3 machines virtuelles sous VMware Workstation sur un réseau privé isolé.
2. Mettre en place une collecte programmée de compteurs de performances système sur CD1.
3. Déployer et exploiter la console d'administration moderne **Windows Admin Center** sur SRV1.
4. Auditer l'installation logicielle via les journaux d'événements Windows (`eventvwr.msc`).

---

## 🛠️ Étape 1 : Déploiement & Configuration Réseau (VMware)

1. Créer un réseau virtuel personnalisé dans VMware (Virtual Network Editor) configuré en **Host-Only** (sans DHCP automatique VMware) sur la plage `192.168.0.0/16`.
2. Déployer les 3 machines virtuelles :
   * **CD1** (Windows Server 2022) : IP fixe `192.168.21.1`, Masque `255.255.0.0`, DNS `127.0.0.1`.
   * **SRV1** (Windows Server 2022) : IP fixe `192.168.21.10`, Masque `255.255.0.0`, DNS `192.168.21.1`.
   * **Win-CL1** (Windows 11) : IP fixe `192.168.21.20`, Masque `255.255.0.0`, DNS `192.168.21.1`.
3. Valider la connectivité réseau par des tests `ping` croisés entre les machines (veiller à autoriser les paquets ICMPv4 dans le pare-feu local).

---

## 📊 Étape 2 : Collecte de Compteurs de Performances sur CD1 (`perfmon.msc`)

1. Lancer l'analyseur de performances via `Windows + R` > `perfmon.msc`.
2. Développer **Ensembles de collecteurs de données** > Clic droit sur **Défini par l'utilisateur** > **Nouveau** > **Ensemble de collecteurs de données**.
3. Nommer l'ensemble `CompteurBase`. Sélectionner **Créer manuellement (avancé)** > Cocher **Compteur de performance**.
4. Ajouter les compteurs de référence suivants :
   * **Processeur** : `% temps processeur`, `% temps d’inactivité`.
   * **Mémoire** : `Mémoire disponible (Mo)`, `Pages/s` (échanges RAM ↔ fichier d'échange).
   * **Disque physique** : `Temps moyen de lecture/écriture (s)`, `% temps disque`, `Lectures disque/s`, `Écritures disque/s`.
5. **Configuration du stockage et du nommage :**
   * Répertoire racine : `C:\PerfLogs`.
   * Format du nom du sous-répertoire : `CompteurBase-AAMMJJ` (date dynamique).
6. **Planification :**
   * Dans l'onglet **Planification**, configurer un démarrage différé de 30 minutes.
   * Dans l'onglet **Condition d'arrêt**, fixer la durée d'échantillonnage à **10 minutes**.
   * Définir une récurrence quotidienne du lundi au vendredi inclus.

---

## 🌐 Étape 3 : Déploiement de Windows Admin Center (WAC) sur SRV1

1. Copier le package d'installation `WindowsAdminCenter.msi` sur `SRV1`.
2. Exécuter l'installation avec les paramètres par défaut (port HTTPS 443, génération d'un certificat auto-signé valide 60 jours).
3. Depuis le client `Win-CL1`, ouvrir le navigateur web et accéder à `https://srv1:443`.
4. **Administration distante depuis l'interface web :**
   * Naviguer dans les paramètres de `SRV1` > Activer le **Bureau à distance** (RDP).
   * Valider la connexion RDP depuis `Win-CL1` en lançant `mstsc /v:srv1`.
   * Dans Windows Admin Center, cliquer sur **Ajouter** > Serveur > Ajouter `CD1` pour centraliser la supervision des deux serveurs.

---

## 🔍 Étape 4 : Audit de l'Installation dans l'Observateur d'Événements

1. Sur `SRV1`, ouvrir la console `eventvwr.msc`.
2. Développer **Journaux Windows** > **Application**.
3. Cliquer sur **Filtrer le journal actuel...** dans le volet d'actions de droite.
4. Sélectionner la source d'événement **`MsiInstaller`**.
5. Identifier l'événement d'installation réussie (ID 1033 ou 11707) pour extraire avec précision :
   * La date et l'heure exactes de fin d'installation.
   * Le code produit GUID et la version exacte de Windows Admin Center déployée.
