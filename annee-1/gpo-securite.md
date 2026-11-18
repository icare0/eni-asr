# 5. Stratégies de Groupe (GPO), Sécurité des Mots de Passe (PSO) & SYSVOL

Les Stratégies de Groupe (*Group Policy Objects* - GPO) constituent le mécanisme de gouvernance et de durcissement centralisé de Windows Server. Elles permettent de configurer, verrouiller et maintenir en conformité des milliers de postes clients et serveurs membres à travers l'annuaire Active Directory.

---

## ⚙️ 1. Mécanisme de Traitement Hiérarchique (Ordre LSDOU)

Lors du démarrage d'une machine ou de l'ouverture de session d'un utilisateur, Windows applique les stratégies selon un ordre hiérarchique invariable :

```
1. Local (Stratégie de sécurité locale de la machine)
      │
      ▼
2. Site (GPO liée au site physique Active Directory)
      │
      ▼
3. Domaine (GPO liée à la racine du domaine domNN.ad)
      │
      ▼
4. Unité d'Organisation (OU Parente puis OU Enfants)
```

> [!IMPORTANT]
> **Règle de Résolution des Conflits :**  
> Si deux GPO définissent un paramètre identique avec des valeurs opposées, **la dernière stratégie appliquée l'emporte toujours**. Une GPO liée à l'Unité d'Organisation finale écrasera donc un paramètre défini au niveau de la racine du domaine (sauf si l'option *Appliquer / Enforced* est cochée sur la GPO supérieure).

---

## 🔒 2. Politiques de Mots de Passe : Pourquoi une GPO liée à une OU échoue-t-elle ?

### L'Erreur Classique des Administrateurs Débutants
Beaucoup d'administrateurs tentent de créer une GPO liée à l'OU `Direction` ou `Informatique` pour imposer des mots de passe plus longs. **Ce paramétrage est strictement ignoré par Windows pour tous les comptes de domaine !**

* **Pourquoi ?** L'authentification Kerberos est gérée exclusivement par le contrôleur de domaine (KDC). Une seule politique de mot de passe de domaine peut être active par domaine, et elle **doit impérativement être liée à la racine du domaine**.

### La Solution : Les Stratégies de Mots de Passe Affinées (PSO)
Introduites pour répondre à ce besoin, les **PSO** (*Password Settings Objects*) permettent d'outrepasser la politique globale du domaine pour l'appliquer à des utilisateurs ou des groupes de sécurité ciblés.

```powershell
# Création d'une politique de mot de passe draconienne pour l'équipe informatique
New-ADFineGrainedPasswordPolicy -Name "PSO-Informatique" `
                                -Precedence 1 `
                                -ComplexityEnabled $true `
                                -MinPasswordLength 12 `
                                -PasswordHistoryCount 24 `
                                -LockoutThreshold 2 `
                                -LockoutDuration "00:00:00" `
                                -ResetLockoutCounterAfter "00:30:00"

# Liaison de la PSO au groupe des informaticiens
Add-ADFineGrainedPasswordPolicySubject -Identity "PSO-Informatique" `
                                      -Subjects "G-Informatique"
```
*Effet : Après seulement 2 échecs de mot de passe, le compte est verrouillé de manière permanente (`00:00:00`), obligeant un déverrouillage manuel par un confrère administrateur.*

---

## 🏛️ 3. Le Magasin Central des Modèles d'Administration (SYSVOL)

Par défaut, chaque poste d'administration lit ses modèles de stratégies (`.admx`) localement dans `C:\Windows\PolicyDefinitions`. Sur un parc hétérogène (Windows 10, Windows 11, Windows Server 2022), des modèles obsolètes ou manquants créent des erreurs d'affichage et des paramètres corrompus.

### Architecture du Magasin Central
On centralise tous les modèles dans le partage répliqué `SYSVOL` du domaine :
```
\\domNN.ad\SYSVOL\domNN.ad\Policies\PolicyDefinitions
    ├── *.admx (Modèles de définition universels)
    └── fr-FR\
         └── *.adml (Fichiers de localisation en langue française)
```

Dès que ce dossier existe et est alimenté, l'éditeur de stratégie de groupe de n'importe quel poste d'administration affiche la mention officielle :  
> *« Modèles d'administration : Récupérés à partir du magasin central »*.

---

## 🛡️ 4. Les 6 GPO Fondamentales de Durcissement en Entreprise

| GPO Déployée | Cible de Liaison | Paramètres Clés |
| :--- | :--- | :--- |
| **GPO-Postes-Commun-Securite** | OU `Postes de travail/Clients` | Masquer le dernier utilisateur connecté à l'écran de verrouillage, forcer l'activation du pare-feu Windows Defender sur les 3 profils (Domaine, Privé, Public). |
| **GPO-Utilisateurs-Direction-Restrictions** | OU `Utilisateurs/Direction` | Masquer les propriétés de "Ce PC", interdire l'ajout du composant MMC Certificats, bloquer l'accès à `regedit` et désactiver le Gestionnaire des tâches. |
| **GPO-Utilisateurs-Interimaires-Restrictions** | OU `Utilisateurs/Comptabilité` *(Filtrage de sécurité sur `G-Intérimaires`)* | Masquer la Corbeille sur le Bureau, désactiver les animations de bienvenue et bloquer l'accès aux lecteurs CD/DVD. |
| **GPO-Domaine-Politique-MotsDePasse** | **Racine du domaine** `domNN.ad` | Longueur minimale 10 caractères, complexité activée, historique 20 mots de passe, expiration maximale 30 jours. |
| **GPO-Utilisateurs-Deploiement-Imprimante** | OU `Utilisateurs` *(Préférences GPO)* | Déploiement automatique de l'imprimante réseau `\\SRV1\Dell 5210CN` avec ciblage par élément basé sur l'appartenance aux groupes. |
| **GPO-Serveurs-RDP-NLA** | OU `Serveurs/Serveurs membres` | Activation du Bureau à distance, exigence du protocole NLA (Network Level Authentication) et ajout du support technique au groupe local. |

---

## 📁 5. Redirection Centralisée des Dossiers Utilisateurs

Pour protéger les données contre le vol ou la perte d'un PC portable, on configure la redirection du dossier personnel *Documents* :
* Emplacement : `\\SRV1\USERS$\%USERNAME%`.
* La redirection s'effectue de manière totalement transparente pour l'utilisateur, et les données sont synchronisées sur le serveur de fichiers résilient.

---

## 🩺 6. Diagnostic & Validation des Stratégies Client

Sur le poste client `Win-CL1` :

```cmd
:: Forcer l'application immédiate des nouvelles stratégies
gpupdate /force

:: Afficher la liste synthétique des GPO appliquées
gpresult /r

:: Générer un rapport d'audit HTML exhaustif
gpresult /h C:\resultat-gpo.html
```
*Le rapport HTML détaille l'état de chaque extension côté client (CSE), les filtres WMI et les raisons d'un éventuel refus d'application.*
