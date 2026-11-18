# Atelier 05 — Stratégies de Groupe (GPO), Sécurité & Magasin SYSVOL

Cet atelier pratique met en œuvre le durcissement de sécurité du parc informatique via les **Stratégies de Groupe (GPO)**, les **politiques de mots de passe affinées (PSO)** et la création du **magasin central SYSVOL**.

---

## 🎯 Objectifs Opérationnels
1. Déployer et alimenter le magasin central de modèles d'administration (`PolicyDefinitions`) dans `SYSVOL`.
2. Mettre en œuvre 6 GPO de durcissement (ordinateur, utilisateur, restrictions).
3. Configurer une politique de mots de passe affinée (**PSO**) pour les comptes informatiques en PowerShell.
4. Déployer une imprimante partagée via les Préférences de GPO avec ciblage par élément.
5. Configurer la redirection centralisée des dossiers utilisateurs (*Documents*).
6. Auditer l'application des stratégies avec `gpupdate` et `gpresult`.

---

## 🏛️ Étape 1 : Création du Magasin Central dans SYSVOL

1. Sur le contrôleur de domaine `CD1`, naviguer vers le partage :
   ```
   \\domNN.ad\SYSVOL\domNN.ad\Policies
   ```
2. Créer un sous-dossier nommé impérativement **`PolicyDefinitions`**.
3. Copier l'intégralité des fichiers modèles `.admx` depuis `C:\Windows\PolicyDefinitions` vers ce nouveau dossier.
4. Créer le sous-répertoire de langue `PolicyDefinitions\fr-FR` et y copier tous les fichiers `.adml` correspondants.
5. **Vérification** : Ouvrir `gpmc.msc` sur le client `Win-CL1`. Dans l'éditeur de stratégie, vérifier que la mention *« Modèles d'administration : Récupérés à partir du magasin central »* apparaît.

---

## 🛡️ Étape 2 : Déploiement des Stratégies de Groupe

### 1. `GPO-Postes-Commun-Securite`
* **Liaison** : Liée à l'OU `Société ASR/Postes de travail/Clients`.
* **Paramètres** :
  * *Options de sécurité* : Masquer le dernier utilisateur connecté (`Activé`).
  * *Pare-feu Windows Defender* : Forcer l'activation du pare-feu sur les 3 profils (Domaine, Privé, Public).

### 2. `GPO-Utilisateurs-Direction-Restrictions`
* **Liaison** : Liée à l'OU `Société ASR/Utilisateurs/Direction`.
* **Paramètres (Configuration utilisateur > Modèles d'administration)** :
  * Masquer l'accès aux propriétés de *Ce PC*.
  * Empêcher l'ajout du composant logiciel enfichable de la console MMC *Certificats*.
  * Désactiver les outils de modification du Registre (`regedit`).
  * Désactiver le Gestionnaire des tâches (`Ctrl + Maj + Échap`).

### 3. `GPO-Utilisateurs-Interimaires-Restrictions`
* **Liaison** : Liée à l'OU `Société ASR/Utilisateurs/Comptabilité`.
* **Filtrage de sécurité** : Supprimer `Utilisateurs authentifiés` du filtrage de sécurité et ajouter uniquement le groupe global **`G-Intérimaires`** (avec droits *Lecture* et *Appliquer la stratégie de groupe*).
* **Paramètres** :
  * Masquer la Corbeille sur le Bureau.
  * Désactiver l'animation de premier démarrage de Windows.
  * Bloquer l'accès en lecture/écriture au lecteur de CD/DVD.

### 4. `GPO-Domaine-Politique-MotsDePasse`
* **Liaison** : **Liée impérativement à la racine du domaine `domNN.ad`**.
* **Paramètres (Configuration ordinateur > Paramètres de sécurité > Stratégies de compte)** :
  * Longueur minimale : **10 caractères**.
  * Exigences de complexité : **Activées**.
  * Historique : **20 mots de passe mémorisés**.
  * Durée de vie maximale : **30 jours**.

---

## 🔒 Étape 3 : Stratégie de Mot de Passe Affinée (PSO)

Pour imposer un verrouillage draconien aux informaticiens sans pénaliser les utilisateurs métiers :

```powershell
# 1. Création de l'objet PSO
New-ADFineGrainedPasswordPolicy -Name "PSO-Informatique" `
                                -Precedence 1 `
                                -ComplexityEnabled $true `
                                -MinPasswordLength 12 `
                                -PasswordHistoryCount 24 `
                                -LockoutThreshold 2 `
                                -LockoutDuration "00:00:00" `
                                -ResetLockoutCounterAfter "00:30:00"

# 2. Association de la PSO au groupe G-Informatique
Add-ADFineGrainedPasswordPolicySubject -Identity "PSO-Informatique" `
                                      -Subjects "G-Informatique"
```
*Effet : Après seulement 2 tentatives infructueuses, le compte informatique est verrouillé indéfiniment jusqu'à déverrouillage manuel par un administrateur.*

---

## 🖨️ Étape 4 : Déploiement d'Imprimante par Préférences de GPO

1. Créer la GPO `GPO-Utilisateurs-Deploiement-Imprimante`.
2. Éditer sous **Configuration utilisateur** > **Préférences** > **Paramètres du Panneau de configuration** > **Imprimantes**.
3. Clic droit > **Nouveau** > **Imprimante partagée** :
   * Action : `Mettre à jour`.
   * Chemin : `\\SRV1\Dell 5210CN`.
   * Cocher **Définir comme imprimante par défaut**.
4. Dans l'onglet **Commun**, cocher **Ciblage au niveau de l'élément** > Cibler selon l'appartenance au groupe d'utilisateurs autorisés.

---

## 📁 Étape 5 : Redirection du Dossier Documents

1. Dans la GPO de base des utilisateurs, naviguer vers :
   * **Configuration utilisateur** > **Stratégies** > **Paramètres Windows** > **Redirection de dossiers** > **Documents**.
2. Paramètres :
   * Définition : *De base - Rediriger les dossiers de tout le monde vers le même emplacement*.
   * Emplacement : *Créer un dossier pour chaque utilisateur sous le chemin racine*.
   * Chemin d'accès racine : `\\SRV1\USERS$\%USERNAME%`.
   * Exclure les intérimaires via le filtrage de sécurité.

---

## 🩺 Étape 6 : Validation & Diagnostic Client

Sur le poste client `Win-CL1` :

```cmd
:: Forcer la mise à jour immédiate des stratégies
gpupdate /force

:: Afficher la liste des GPO appliquées
gpresult /r

:: Générer le rapport d'analyse HTML complet
gpresult /h C:\resultat-gpo.html
```

Ouvrir `C:\resultat-gpo.html` dans Edge pour vérifier que les restrictions s'appliquent correctement selon l'utilisateur connecté !
