# TP 05 — Stratégies de Groupe (GPO) & Durcissement PSO

Fiche de synthèse de l'atelier pratique n°5 : déploiement du magasin central SYSVOL, industrialisation de politiques de durcissement par GPO, stratégies de mot de passe affinées (PSO) et diagnostic d'application.

---

## 1. Contexte & Périmètre de l'Atelier

* **Machines mobilisées** :
  * Contrôleurs de domaine : `CD1` et `CD2` (hébergement et réplication SYSVOL).
  * Serveur de fichiers : `SRV1` (partage de redirection de dossiers `\\SRV1\Users$` et imprimante).
  * Poste client d'évaluation : `Win-CL1` (validation des effets GPO utilisateur et machine).
* **Objectif global** : Maîtriser le déploiement centralisé de politiques de sécurité, durcir l'environnement de travail des collaborateurs, implémenter une politique de mot de passe renforcée pour les comptes à hauts privilèges et valider les résultats par audit.

---

## 2. Réalisations Techniques Clés

### Déploiement du Magasin Central (*Central Store*) dans SYSVOL
* Création du répertoire de référence `PolicyDefinitions` au sein du partage répliqué Active Directory :
  `\\domNN.ad\SYSVOL\domNN.ad\Policies\PolicyDefinitions`
* Copie des modèles d'administration `.admx` neutres et des fichiers de langue `.adml` dans le sous-dossier `fr-FR`.
* **Validation** : Dès lors, toutes les consoles GPMC du domaine indiquent la mention *« Modèles d'administration (ADMX) récupérés à partir du magasin central »*.

### Matrice des Stratégies de Groupe (GPO) Déployées

| Stratégie de groupe (GPO) | Niveau de liaison | Filtrage / Cible | Paramètres & Restrictions appliqués |
| :--- | :--- | :--- | :--- |
| **GPO-Postes-Commun-Securite** | OU `Postes de travail / Clients` | Tous les postes clients | • Masquage du dernier utilisateur connecté à l'ouverture de session.<br>• Activation stricte et verrouillage du Pare-feu Windows Defender sur tous les profils (Domaine, Privé, Public). |
| **GPO-Utilisateurs-Direction-Restrictions** | OU `Utilisateurs / Direction` | Collaborateurs de la Direction | • Blocage de l'éditeur de registre (`regedit.exe`).<br>• Désactivation du Gestionnaire des tâches.<br>• Masquage des propriétés système.<br>• Restriction d'exécution de la console MMC Certificats. |
| **GPO-Utilisateurs-Interimaires-Restrictions** | OU `Utilisateurs` | **Filtrage de sécurité** : Cible exclusivement le groupe `G-Intérimaires` | • Masquage de l'icône de la Corbeille sur le Bureau.<br>• Suppression de l'animation de premier démarrage de Windows.<br>• Blocage de l'accès et de l'exécution sur les lecteurs optiques CD/DVD. |
| **GPO-Serveurs-RDP-NLA** | OU `Serveurs / Serveurs membres` | Serveur `SRV1` | • Activation du service Bureau à distance (RDP).<br>• Exigence obligatoire de l'authentification au niveau du réseau (**NLA**).<br>• Intégration automatique du groupe `G-Support technique` au groupe local *Utilisateurs du Bureau à distance*. |
| **GPO-Utilisateurs-Deploiement-Imprimante** | OU `Utilisateurs` | Préférences GPO avec ciblage par élément | • Connexion et configuration automatique par défaut de l'imprimante réseau partagée `\\SRV1\Dell5210CN` lors de l'ouverture de session. |

### Redirection Sécurisée de Dossiers
* Déploiement par GPO de la redirection du dossier **Documents** vers le partage réseau sécurisé du serveur de fichiers : `\\SRV1\Users$\%USERNAME%`.
* Octroi exclusif des droits d'accès à l'utilisateur propriétaire et déplacement automatique des documents existants sans intervention utilisateur.
* Exclusion des intérimaires de la politique de redirection.

### Stratégie de Mots de Passe Globale vs PSO Informatique
1. **Politique globale du domaine** (liée à la racine de `domNN.ad`) :
   * Longueur minimale : 10 caractères.
   * Complexité activée (majuscules, minuscules, chiffres, caractères spéciaux).
   * Historique : mémorisation des 20 derniers mots de passe.
   * Durée de vie maximale : 30 jours.
2. **Stratégie affinée (*PSO*) pour les comptes sensibles** :
   * Création de l'objet **`PSO-Informatique`** avec précédence de niveau 1 (priorité absolue).
   * Longueur minimale portée à **12 caractères**.
   * Seuil de verrouillage : **2 tentatives infructueuses seulement**.
   * Durée de verrouillage fixée à `00:00:00` : le compte reste verrouillé de manière permanente jusqu'à **déverrouillage manuel obligatoire par un confrère administrateur**.
   * Affectation directe au groupe de sécurité `G-Informatique`.

---

## 3. Commandes de Validation & Contrôle

```powershell
# Forcer la réévaluation immédiate de toutes les stratégies sur la machine cliente
gpupdate /force

# Vérifier en console les GPO appliquées à l'ordinateur et à la session utilisateur
gpresult /r

# Générer le rapport d'audit exhaustif des GPO au format HTML
gpresult /h C:\resultat-gpo.html

# Auditer la stratégie de mot de passe affinée (PSO) appliquée à un informaticien
Get-ADUserResultantPasswordPolicy -Identity "ivedere"

# Contrôler les paramètres de configuration de l'objet PSO
Get-ADFineGrainedPasswordPolicy -Identity "PSO-Informatique" | 
    Select-Object Name, Precedence, MinPasswordLength, LockoutThreshold, LockoutDuration

# Vérifier la présence des modèles d'administration dans le magasin central SYSVOL
Get-ChildItem -Path "\\domNN.ad\SYSVOL\domNN.ad\Policies\PolicyDefinitions" -Filter "*.admx" | 
    Select-Object -First 5 Name
```
