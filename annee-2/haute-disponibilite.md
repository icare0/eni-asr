# 2. Haute Disponibilité, Résilience & PCA/PRA

Dans une infrastructure d'entreprise, une indisponibilité non planifiée d'un service critique (messagerie, ERP, fichiers) se traduit immédiatement par des pertes financières et d'exploitation.

---

## ⏱️ 1. Les Métriques Stratégiques : RTO et RPO

Avant toute conception technique, l'administrateur définit avec la direction les deux indicateurs clés de reprise :

```
Temps (t) ──────► [ Événement de Panne / Crash ] ──────► [ Reprise normale ]
      │<──────────────── RPO ───────────────>│<────────────── RTO ─────────────>│
        (Perte de données maximale admise)       (Temps d'interruption maximal)
```

* **RTO (Recovery Time Objective)** : Durée maximale admissible d'interruption d'un service. Exemple : "Le serveur de fichiers doit être de retour en ligne en moins de 15 minutes".
* **RPO (Recovery Point Objective)** : Âge maximal des données perdues lors du sinistre. Exemple : "La base de données ne doit pas perdre plus de 2 heures d'enregistrements".
* **PCA (Plan de Continuité d'Activité)** : Mesures permettant de maintenir l'activité sans interruption perceptible (clustering, redondance).
* **PRA (Plan de Reprise d'Activité)** : Procédures permettant de reconstruire l'infrastructure sur un site de secours après un désastre majeur (incendie, cyberattaque généralisée).

---

## 🏗️ 2. Clusters de Basculement Windows Server (WSFC)

La technologie **Windows Server Failover Clustering (WSFC)** regroupe plusieurs serveurs physiques ou virtuels (*nœuds*) pour fournir une haute disponibilité aux rôles hébergés.

```
       [ Client Réseau ]
              │
              ▼
    [ Nom Virtuel du Cluster ]
    [  Adresse IP du Cluster  ]
              │
       ┌──────┴──────┐
       ▼             ▼
  ┌─────────┐   ┌─────────┐
  │ Nœud 1  │   │ Nœud 2  │
  │ (Actif) │   │(Passif) │
  └────┬────┘   └────┬────┘
       │             │
       └──────┬──────┘
              ▼
   [ Stockage Partagé / Quorum ]
```

### Le Quorum et les Témoins (*Witness*)
Pour éviter le syndrome de scission (*Split-Brain* où deux serveurs pensent que l'autre est mort et tentent d'écrire en même temps sur les disques), le cluster utilise un mécanisme de vote à la majorité :
* **Témoin de partage de fichiers (*File Share Witness*)** : Un partage SMB sur un serveur tiers détient un vote.
* **Témoin Cloud (*Cloud Witness*)** : Un conteneur Azure Blob Storage sert d'arbitre impartial.

---

## 🌐 3. Système de Fichiers Distribué (DFS)

### DFS-N (Espaces de noms)
Fournit un chemin d'accès UNC virtuel unifié et indépendant de l'emplacement physique des serveurs :
* L'utilisateur accède à `\\domNN.ad\Public\Docs`.
* En arrière-plan, le système le redirige de façon transparente vers `\\SRV1\Docs` ou `\\SRV2\Docs`.

### DFS-R (Réplication)
Synchronise les dossiers entre plusieurs serveurs distants à travers le réseau WAN grâce à l'algorithme **RDC** (*Remote Differential Compression*) qui n'envoie que les octets modifiés au sein d'un fichier.

---

## 💾 4. La Règle d'Or de la Sauvegarde : La Stratégie 3-2-1

Pour garantir la survie des données d'entreprise contre les pannes matérielles et les attaques de type **Ransomware** :

* **3** copies de vos données (1 copie originale de production + 2 sauvegardes distinctes).
* **2** supports technologiques différents (ex: Baie de disques NAS + Bandes magnétiques LTO ou stockage objet).
* **1** copie conservée **hors site** (*Off-site*), idéalement **immuable** (WORM : *Write Once, Read Many* empêchant toute suppression ou chiffrement malveillant pendant une période définie).
