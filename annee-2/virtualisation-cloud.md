# 4. Virtualisation Datacenter & Cloud Hybride (Azure / Entra ID)

L'époque des salles serveurs remplies de machines physiques dédiées à une seule tâche est révolue. Aujourd'hui, l'administrateur gère des hyperviseurs de datacenter et connecte son infrastructure locale au Cloud public.

---

## 🏢 1. Virtualisation d'Entreprise : VMware vSphere & Proxmox

Tandis que VMware Workstation est un hyperviseur de type 2 (qui s'exécute par-dessus un système d'exploitation de bureau), la production repose sur des hyperviseurs **Bare-Metal (Type 1)** installés directement sur le matériel physique.

```
       [ Console Centralisée vCenter Server / Proxmox Web GUI ]
                                  │
         ┌────────────────────────┴────────────────────────┐
         ▼                                                 ▼
┌──────────────────┐                              ┌──────────────────┐
│  Hôte ESXi 01    │ ◄══════ vMotion (À chaud) ══► │   Hôte ESXi 02   │
│ (Type 1 Bare-Met)│                              │ (Type 1 Bare-Met)│
└────────┬─────────┘                              └────────┬─────────┘
         │                                                 │
         └────────────────────────┬────────────────────────┘
                                  ▼
                   [ Baie de Stockage SAN / iSCSI ]
```

### Les Fonctionnalités Essentielles de VMware vSphere
* **vMotion** : Déplacement d'une machine virtuelle en cours d'exécution d'un serveur physique à un autre **sans aucune seconde d'interruption de service** pour les utilisateurs.
* **vSphere HA (High Availability)** : Si l'hôte physique `ESXi 01` subit une panne de carte mère ou d'alimentation, les machines virtuelles sont redémarrées automatiquement sur l'hôte survivant `ESXi 02` en quelques dizaines de secondes.
* **vSphere DRS (Distributed Resource Scheduler)** : Répartition automatique de la charge : les machines virtuelles sont migrées dynamiquement pour équilibrer la consommation CPU et mémoire vive entre les hôtes.

---

## ☁️ 2. L'Infrastructure Hybride avec Microsoft Azure

Une infrastructure hybride combine les serveurs hébergés dans les locaux de l'entreprise avec les services illimités du Cloud public.

```
[ Datacenter Local (On-Premises) ]               [ Cloud Microsoft Azure ]
Contrôleurs de Domaine (CD1/CD2)                  Microsoft Entra ID (Cloud)
               │                                              ▲
               │════════ Synchronisation Sécurisée ═══════════│
               │         (Microsoft Entra Connect)            │
               ▼                                              ▼
   Utilisateurs & Mots de Passe ──────────────────► Comptes Microsoft 365
                                                    (Teams, Exchange Online)
```

### Le Rôle Clé de Microsoft Entra Connect
Pour éviter aux collaborateurs de devoir mémoriser deux mots de passe différents (un pour leur session Windows au bureau et un pour leur boîte mail Office 365), l'administrateur déploie l'outil de synchronisation **Microsoft Entra Connect** :

* **Synchronisation du hachage de mot de passe (PHS - Password Hash Sync)** : Une empreinte chiffrée du mot de passe local est synchronisée en continu vers le Cloud Azure.
* **Authentification Unique (Seamless SSO)** : L'utilisateur connecté à son PC de bureau accède instantanément à ses applications Cloud sans avoir à ressaisir son mot de passe.
