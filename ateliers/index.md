# 🔬 Guide de la Maquette de Laboratoire (Ateliers Pratiques)

Tous les ateliers pratiques (TP01 à TP05) s'appuient sur une maquette de machines virtuelles interconnectées au sein d'un environnement virtualisé sous **VMware Workstation**.

---

## 🗺️ Schéma & Topologie d'Adressage

```
                       [ Réseau Privé Virtuel VMware : Host-Only ]
                                 (Plage : 192.168.0.0/16)
                                            │
        ┌───────────────────────┬───────────┴───────────┬───────────────────────┐
        │                       │                       │                       │
        ▼                       ▼                       ▼                       ▼
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│       CD1        │    │       CD2        │    │       SRV1       │    │     Win-CL1      │
│  Contrôleur DC1  │    │  Contrôleur DC2  │    │  Serveur Membre  │    │  Client Windows  │
│  192.168.21.1/16 │    │  192.168.21.2/16 │    │ 192.168.21.10/16 │    │ 192.168.21.20/16 │
│  DNS : 127.0.0.1 │    │  DNS : .21.1     │    │ DNS : .21.1      │    │ DNS : .21.1      │
└──────────────────┘    └──────────────────┘    └──────────────────┘    └──────────────────┘
```

---

## 📋 Inventaire des Machines de la Maquette

| Nom d'Hôte | Système d'Exploitation | Rôle dans l'Infrastructure | Adresse IP / Masque | Serveur DNS |
| :--- | :--- | :--- | :--- | :--- |
| **CD1** | Windows Server 2022 | Contrôleur de domaine principal, Serveur DNS intégré, Maître FSMO racine. | `192.168.21.1 /16` | `127.0.0.1` |
| **CD2** | Windows Server 2022 | Contrôleur additionnel (tolérance aux pannes), Maître RID & Émulateur PDC. | `192.168.21.2 /16` | `192.168.21.1` |
| **SRV1** | Windows Server 2022 | Serveur membre du domaine, Pool de stockage résilient, Partages SMB, Serveur d'impression, Windows Admin Center. | `192.168.21.10 /16` | `192.168.21.1` |
| **Win-CL1** | Windows 11 Professionnel | Poste client d'administration distante équipé des outils RSAT. | `192.168.21.20 /16` | `192.168.21.1` |

---

## 📑 Récapitulatif des Ateliers

1. [**TP 01 : Infrastructure & Métrologie**](/ateliers/tp01-infrastructure-metrologie.md) : Déploiement initial, collectes de performances `perfmon`, WAC et audit des événements.
2. [**TP 02 : Création Domaine AD & Rôles FSMO**](/ateliers/tp02-domaine-ad-fsmo.md) : Promotion PowerShell de la forêt, intégration de CD2 et manipulation des rôles FSMO.
3. [**TP 03 : Société ASR, OU, Sites & LDAP**](/ateliers/tp03-gestion-societe-asr.md) : Création de l'arborescence, comptes modèles, groupes et topologie des sites WAN.
4. [**TP 04 : Stockage résilient & Modèle AGDLP**](/ateliers/tp04-ressources-agdlp.md) : Storage Spaces, partages SMB, matrice AGDLP, délégations et imprimante réseau.
5. [**TP 05 : GPO, Sécurité, PSO & SYSVOL**](/ateliers/tp05-gpo-securite-avancee.md) : Déploiement des 6 GPOs, politiques affinées PSO et magasin central ADMX.
