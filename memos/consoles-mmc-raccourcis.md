[← Retour aux Mémos](README.md) | [Accueil](../README.md)

# Consoles MMC, Applets & Raccourcis Clavier

Ce mémento rassemble l'ensemble des commandes d'exécution directe (`Windows + R`) pour ouvrir instantanément les consoles MMC et applets de configuration, ainsi que les raccourcis clavier indispensables sous Windows Server et en session Bureau à distance (RDP).

---

## 1. Consoles d'Administration MMC (`.msc`)

| Commande | Rôle / Console ouverte | Usage courant |
| :--- | :--- | :--- |
| **`dsa.msc`** | Utilisateurs et ordinateurs Active Directory | Gestion des comptes, groupes, OU, réinitialisation de mot de passe, délégation de contrôle. |
| **`dssite.msc`** | Sites et services Active Directory | Gestion des sites physiques, sous-réseaux IP et planification de la réplication inter-sites. |
| **`domain.msc`** | Domaines et approbations Active Directory | Niveaux fonctionnels de domaine/forêt et gestion des approbations inter-domaines. |
| **`adsiedit.msc`** | Éditeur ADSI (ADSI Edit) | Consultation et modification bas niveau des attributs LDAP bruts de l'annuaire. |
| **`gpmc.msc`** | Gestion des stratégies de groupe | Conception, liaison, modélisation, filtrage WMI et sécurité des GPO. |
| **`gpedit.msc`** | Éditeur de stratégie de groupe locale | Configuration des politiques de sécurité locales spécifiques à la machine locale. |
| **`secpol.msc`** | Stratégie de sécurité locale | Droits d'attribution des utilisateurs et politiques d'audit locales. |
| **`dnsmgmt.msc`** | Gestionnaire DNS | Zones de recherche directes/inverses, enregistrements A, PTR, CNAME, SRV. |
| **`dhcpmgmt.msc`** | Gestionnaire DHCP | Gestion des étendues IPv4/IPv6, exclusions, baux et réservations MAC. |
| **`compmgmt.msc`** | Gestion de l'ordinateur | Console regroupant disques, dossiers partagés, observateur d'événements et périphériques. |
| **`diskmgmt.msc`** | Gestion des disques | Initialisation des disques, partitionnement GPT/MBR et formatage NTFS/ReFS. |
| **`eventvwr.msc`** | Observateur d’événements | Analyse et filtrage des journaux Système, Sécurité et Application. |
| **`perfmon.msc`** | Analyseur de performances | Analyse en temps réel et création d'ensembles de collecteurs de données planifiés. |
| **`services.msc`** | Services Windows | Démarrage, arrêt, redémarrage et type de démarrage des services système. |
| **`taskschd.msc`** | Planificateur de tâches | Automatisation de scripts PowerShell/Batch et déclenchement d'actions planifiées. |
| **`wf.msc`** | Pare-feu Windows avec fonctions avancées | Règles de filtrage du trafic entrant et sortant par port/protocole. |
| **`certmgr.msc`** | Certificats (Utilisateur actuel) | Gestion des magasins de certificats personnels. |
| **`certlm.msc`** | Certificats (Ordinateur local) | Gestion des certificats machine, autorités racines de confiance et clés privées. |

---

## 2. Applets du Panneau de Configuration (`.cpl`)

| Commande | Applet Ouverte | Usage Rapide |
| :--- | :--- | :--- |
| **`ncpa.cpl`** | Connexions réseau | Affichage et configuration des cartes réseau, adresses IPv4/IPv6, passerelles et serveurs DNS. |
| **`sysdm.cpl`** | Propriétés système | Modification du nom d'hôte de la machine et jonction au domaine Active Directory. |
| **`firewall.cpl`** | Pare-feu Windows Defender | Activation/désactivation rapide du pare-feu par profil de réseau. |
| **`appwiz.cpl`** | Programmes et fonctionnalités | Désinstallation de logiciels et visualisation des mises à jour Windows installées. |

---

## 3. Raccourcis Clavier de l'Administrateur

### Navigation dans les Consoles MMC & l'Annuaire
* <kbd>F5</kbd> : Actualiser immédiatement l'arborescence ou la vue courante.
* <kbd>F2</kbd> : Renommer l'objet sélectionné (OU, compte, groupe).
* <kbd>Maj</kbd> + <kbd>F10</kbd> : Ouvrir le menu contextuel (équivalent du clic droit pour déléguer ou réinitialiser un mot de passe).
* <kbd>Alt</kbd> + <kbd>Entrée</kbd> : Afficher la boîte de propriétés de l'élément sélectionné.
* <kbd>Suppr</kbd> : Supprimer l'élément sélectionné.

### Gestionnaire de Serveur (Server Manager)
* <kbd>Alt</kbd> + <kbd>P</kbd> : Accéder à la tuile Propriétés du serveur local.
* <kbd>Alt</kbd> + <kbd>E</kbd> : Accéder à la tuile Événements.
* <kbd>Alt</kbd> + <kbd>R</kbd> : Accéder à la tuile Services.
* <kbd>Alt</kbd> + <kbd>B</kbd> : Lancer l'analyseur de meilleures pratiques (BPA).
* <kbd>Alt</kbd> + <kbd>O</kbd> : Consulter la tuile Performances.
* <kbd>Alt</kbd> + <kbd>A</kbd> : Consulter la tuile Rôles et fonctionnalités.

### Sessions Bureau à Distance (RDP / `mstsc`)
* <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>Fin</kbd> : Équivalent de `Ctrl + Alt + Suppr` dans la machine distante (changement de mot de passe, verrouillage, gestionnaire des tâches).
* <kbd>Alt</kbd> + <kbd>Début</kbd> : Ouvrir le menu Démarrer de la machine distante.
* <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>Pause</kbd> : Basculer la fenêtre Bureau à distance entre le mode plein écran et le mode fenêtré.
