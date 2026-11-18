# 🎓 ENI ASR — Cursus Administrateur Systèmes & Réseaux

> **Portail de référence et base de connaissances sur 2 ans** pour le cursus d'Administrateur Systèmes et Réseaux (ASR) de l'ENI.  
> Regroupe les cours théoriques, les démarches pas à pas des ateliers pratiques (TP), les scripts PowerShell, les architectures cibles et les mémos d'examen.

---

<span class="badge badge-blue">Titre Pro RNCP Niveau 6 (Bac+2 / Bac+3)</span>
<span class="badge badge-green">Cursus 2 Ans ENI</span>
<span class="badge badge-purple">Windows Server • Linux • Cisco • Cloud</span>
<span class="badge badge-amber">Année 1 en cours</span>

---

## 🗺️ Vision Globale du Cursus sur 2 Ans

Plutôt qu'un simple découpage par modules isolés, ce portail est structuré selon la progression chronologique et les compétences métier acquises sur les **deux années de formation** à l'ENI.

```
                  ┌──────────────────────────────────────────────────────────────┐
                  │          CURSUS ENI — ADMINISTRATEUR SYSTÈMES & RÉSEAUX      │
                  └──────────────────────────────┬───────────────────────────────┘
                                                 │
            ┌────────────────────────────────────┴────────────────────────────────────┐
            ▼                                                                         ▼
┌───────────────────────────────────────┐                 ┌───────────────────────────────────────┐
│     ANNÉE 1 : SOCLE FONDAMENTAL       │                 │   ANNÉE 2 : INGÉNIERIE & HYBRIDE      │
│  Administration & Exploitation On-Prem│                 │   Sécurité, HA, DevOps & Cloud        │
├───────────────────────────────────────┤                 ├───────────────────────────────────────┤
│ • Virtualisation, Outils & Métrologie │                 │ • Réseaux Avancés (VLANs, Routage)    │
│ • Architecture Active Directory (AD)  │                 │ • Haute Disponibilité (WSFC, DFS, PRA)│
│ • Gestion des Comptes & Société ASR   │                 │ • Sécurité SI, Tiering & PKI (AD CS)  │
│ • Stockage Résilient & Modèle AGDLP   │                 │ • Virtualisation Datacenter & Cloud   │
│ • Stratégies de Groupe (GPO) & PSO    │                 │ • Automatisation (PowerShell, Ansible)│
│ • Fondations Réseau (IPv4, DNS, DHCP) │                 │ • Préparation Titre Professionnel     │
│ • Administration Linux (CLI & Bash)   │                 │ • Dossier Projet & Soutenance Jury    │
└───────────────────┬───────────────────┘                 └───────────────────┬───────────────────┘
                    │                                                         │
                    ▼                                                         ▼
       [Ateliers TP 01 à 05 Intégrés]                             [Projet d'Infrastructure Global]
```

---

## 📍 Où en sommes-nous ? (Début de 1ère Année)

Nous sommes actuellement sur les **fondations indispensables de l'administration Windows Server et de l'annuaire Active Directory Domain Services (AD DS)**.

Toutes les sources de notre carnet de recherche pédagogique NotebookLM ont été intégrées, structurées et enrichies :

| Atelier / Module | Objectif Opérationnel | Statut |
| :--- | :--- | :--- |
| [**TP 01 : Infrastructure & Métrologie**](/ateliers/tp01-infrastructure-metrologie.md) | Déploiement de la maquette VMware, compteurs Perfmon et Windows Admin Center | <span class="badge badge-green">Terminé & Documenté</span> |
| [**TP 02 : Création Domaine AD & FSMO**](/ateliers/tp02-domaine-ad-fsmo.md) | Promotion CD1, réplica CD2, RSAT sur Win-CL1 et transfert rôles FSMO | <span class="badge badge-green">Terminé & Documenté</span> |
| [**TP 03 : Société ASR & Annuaire**](/ateliers/tp03-gestion-societe-asr.md) | Unités d'organisation hiérarchiques, comptes modèles, groupes globaux, sites AD | <span class="badge badge-green">Terminé & Documenté</span> |
| [**TP 04 : Stockage & Modèle AGDLP**](/ateliers/tp04-ressources-agdlp.md) | Espace de stockage résilient, partages SMB, matrice AGDLP, délégation & imprimante | <span class="badge badge-green">Terminé & Documenté</span> |
| [**TP 05 : Sécurité GPO & SYSVOL**](/ateliers/tp05-gpo-securite-avancee.md) | Restrictions GPO, stratégie de mot de passe affinée (PSO), modèles ADMX | <span class="badge badge-green">Terminé & Documenté</span> |

---

## 🚀 Navigation Rapide

* 📘 [**Accéder au programme complet de l'Année 1**](/annee-1/) : Concepts fondamentaux, architectures, guides théoriques.
* 🔬 [**Consulter les Ateliers Pratiques (TP)**](/ateliers/) : Démarches pas à pas avec scripts PowerShell testés et validés.
* 🚀 [**Découvrir la feuille de route de l'Année 2**](/annee-2/) : Les compétences avancées, l'ingénierie réseau, la sécurité et le cloud.
* 🛠️ [**Ouvrir la Boîte à Outils Administrateur**](/memos/consoles-mmc-raccourcis.md) : Consoles MMC (`.msc`), raccourcis clavier, cheatsheet PowerShell.

---

> [!TIP]
> **Recherche rapide** : Utilisez la barre de recherche en haut à gauche (ou la touche `/`) pour trouver instantanément une commande PowerShell, une console MMC ou une explication de cours dans l'ensemble du site.
