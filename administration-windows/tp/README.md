# Coin TP — Synthèse des Ateliers Pratiques en Laboratoire

Cette section documente les réalisations concrètes effectuées en travaux pratiques (TP) sur l'infrastructure de virtualisation VMware Workstation durant le module d'administration Windows Server.

Plutôt qu'un guide pas-à-pas verbeux détaillant chaque clic de souris dans les menus graphiques, ces fiches de synthèse présentent de manière valorisante :
* Les **objectifs opérationnels** visés par l'atelier.
* Les **machines et paramètres réseau** mobilisés.
* Les **configurations et livrables techniques** effectivement mis en place.
* Les **commandes de test et de validation** confirmant le bon fonctionnement.

---

## Architecture de la Maquette de Laboratoire (VMware)

L'ensemble des travaux pratiques est déployé sur un réseau virtuel privé isolé (**Host-Only / VMnet1**) sur le plan d'adressage `192.168.0.0/16`. Cette isolation garantit l'absence totale de perturbation sur le réseau physique de formation.

```
                           [ Réseau Virtuel VMware : Host-Only ]
                                (Plage IP : 192.168.0.0/16)
                                             │
        ┌────────────────────────┬───────────┴───────────┬────────────────────────┐
        │                        │                       │                        │
        ▼                        ▼                       ▼                        ▼
┌────────────────┐      ┌────────────────┐      ┌────────────────┐      ┌────────────────┐
│      CD1       │      │      CD2       │      │      SRV1      │      │    Win-CL1     │
│ Windows Server │      │ Windows Server │      │ Windows Server │      │   Windows 11   │
│ Contrôleur AD  │      │ Second DC      │      │ Serveur Membre │      │ Poste Client   │
│ 192.168.21.1   │      │ 192.168.21.2   │      │ 192.168.21.10  │      │ 192.168.21.20  │
└────────────────┘      └────────────────┘      └────────────────┘      └────────────────┘
```

### Inventaire des Machines Virtuelles

| Machine | Système d'exploitation | Rôle dans l'infrastructure | Adresse IP | Ressources allouées |
| :--- | :--- | :--- | :--- | :--- |
| **CD1** | Windows Server 2022 | Premier contrôleur de domaine racine (`domNN.ad`), DNS intégré, rôles FSMO forêt & infrastructure | `192.168.21.1/16` | 1 vCPU, 4 Go RAM, disque 40 Go |
| **CD2** | Windows Server 2022 | Second contrôleur de domaine réplica, DNS intégré, détenteur des rôles FSMO RID et PDC | `192.168.21.2/16` | 1 vCPU, 4 Go RAM, disque 40 Go |
| **SRV1** | Windows Server 2022 | Serveur membre du domaine, serveur de fichiers et de stockage (*Storage Spaces*), serveur d'impression | `192.168.21.10/16` | 1 vCPU, 2 Go RAM, disque système 40 Go + 3 disques SCSI 20 Go |
| **Win-CL1** | Windows 11 Enterprise | Poste client membre du domaine, station de travail d'administration avec outils RSAT | `192.168.21.20/16` | 1 vCPU, 4 Go RAM, disque 40 Go |

---

## Sommaire des Ateliers Pratiques Réalisés

* **[TP 01 : Création de l'infrastructure & Métrologie](tp01-infrastructure-metrologie.md)**
  * Déploiement des machines virtuelles et configuration réseau IP.
  * Audit des compteurs critiques sous `perfmon` et collecte automatisée programmée.
  * Déploiement de Windows Admin Center sur SRV1 et audit `MsiInstaller`.

* **[TP 02 : Création du domaine Active Directory & FSMO](tp02-domaine-fsmo.md)**
  * Promotion du contrôleur racine `domNN.ad` et DNS intégré.
  * Tolérance aux pannes via la promotion du second DC `CD2`.
  * Jonction au domaine des membres et déploiement RSAT sur Windows 11.
  * Transfert gracieux des rôles FSMO RID et PDC vers CD2.

* **[TP 03 : Structuration de l'annuaire Société ASR](tp03-annuaire-societe-asr.md)**
  * Arborescence hiérarchique d'Unités d'Organisation à 3 niveaux.
  * Création de comptes modèles (Direction, Informatique, Comptabilité avec plages horaires).
  * Population des utilisateurs réels et groupes globaux de sécurité.
  * Topologie de sites physiques (Siège et Agence) et requêtes LDAP enregistrées.

* **[TP 04 : Espaces de stockage résilients & Modèle AGDLP](tp04-stockage-partages-agdlp.md)**
  * Pool de stockage logiciel Storage Spaces avec résilience en miroir sur SRV1.
  * Volumes DATA (E:) et USERS (monté dans C:\Base).
  * Partages SMB (Documentation, Compta, Info$) et implémentation rigoureuse du modèle AGDLP.
  * Serveur d'impression Dell 5210CN et délégations administratives d'annuaire.

* **[TP 05 : Stratégies de groupe (GPO) & Sécurité PSO](tp05-gpo-securite-pso.md)**
  * Déploiement du Magasin Central `PolicyDefinitions` dans SYSVOL.
  * GPO de sécurité pour postes clients, restrictions utilisateurs et déploiement d'imprimante.
  * Politique de mot de passe globale du domaine et stratégie affinée PSO pour l'informatique.
  * Activation du Bureau à distance NLA et validation méthodique avec `gpresult`.
