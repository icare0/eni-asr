# 5. Stratégies de Groupe (GPO), Sécurité & Magasin SYSVOL

Les Stratégies de Groupe (*Group Policy Objects* - GPO) permettent d'administrer et de sécuriser de façon centralisée des milliers de postes clients et serveurs membres au sein d'un domaine Active Directory.

---

## ⚙️ 1. Cycle de Traitement & Ordre d'Application (LSDOU)

Lors du démarrage d'une machine ou de l'ouverture de session d'un utilisateur, Windows applique les stratégies selon un ordre hiérarchique strict :

```
1. Local (Stratégie locale de la machine)
      │
      ▼
2. Site (GPO liée au site physique Active Directory)
      │
      ▼
3. Domaine (GPO liée à la racine du domaine domNN.ad)
      │
      ▼
4. Unité d'Organisation (OU parente puis OU enfants)
```

> [!IMPORTANT]
> **Règle de résolution des conflits :**  
> En cas de conflit sur un même paramètre, **la dernière stratégie appliquée gagne**. Une GPO liée à l'OU finale écrasera donc un paramètre défini au niveau du domaine, sauf si l'option *Appliquer* (*Enforced*) est cochée au niveau supérieur.

---

## 🔒 2. Politiques de Mots de Passe : Pourquoi une GPO sur une OU échoue-t-elle ?

### L'incompréhension classique
Beaucoup d'administrateurs débutants créent une GPO liée à une OU spécifique (ex. `OU=Direction`) pour exiger un mot de passe de 14 caractères. **Cela ne fonctionne jamais pour des comptes de domaine !**

* Pourquoi ? L'authentification Kerberos est gérée exclusivement par le contrôleur de domaine (KDC). Une seule politique de mot de passe de domaine peut être active par domaine, et elle **doit être impérativement liée à la racine du domaine**.

### La solution : Les Stratégies de Mots de Passe Affinées (PSO)
Introduites avec Windows Server, les **PSO** (*Password Settings Objects*) permettent d'imposer des règles de complexité ou de longueur plus strictes à des utilisateurs ou des groupes de sécurité particuliers.

```powershell
# Création d'une PSO stricte pour le service Informatique
New-ADFineGrainedPasswordPolicy -Name "PSO-Informatique" `
                                -Precedence 1 `
                                -ComplexityEnabled $true `
                                -MinPasswordLength 12 `
                                -PasswordHistoryCount 24 `
                                -LockoutThreshold 2 `
                                -LockoutDuration "00:00:00" `
                                -ResetLockoutCounterAfter "00:30:00"

# Application de la PSO au groupe des informaticiens
Add-ADFineGrainedPasswordPolicySubject -Identity "PSO-Informatique" `
                                      -Subjects "G-Informatique"
```

---

## 📂 3. Le Magasin Central des Modèles d'Administration (SYSVOL)

Par défaut, chaque poste d'administration lit ses modèles de GPO (`.admx`) localement dans `C:\Windows\PolicyDefinitions`. En cas de parc hétérogène (Windows 10, Windows 11, Windows Server), des incohérences apparaissent.

### Création du Magasin Central
On centralise tous les modèles dans le partage répliqué `SYSVOL` du domaine :
```
\\domNN.ad\SYSVOL\domNN.ad\Policies\PolicyDefinitions
    ├── *.admx (Fichiers de définition indépendants de la langue)
    └── fr-FR\
         └── *.adml (Fichiers de localisation en français)
```

Dès que ce dossier est alimenté, l'Éditeur de stratégie de groupe affiche la confirmation :  
> *« Modèles d'administration : Récupérés à partir du magasin central »*.

---

## 🛠️ 4. Stratégies vs Préférences de GPO

Dans la console `gpmc.msc`, chaque branche se subdivise en :
* **Stratégies (Policies)** : Paramètres imposés et verrouillés. L'utilisateur ne peut pas modifier la valeur (grisée dans l'interface Windows). Si la GPO est supprimée, la valeur revient à l'état par défaut (Tatouage évité).
* **Préférences (Preferences)** : Valeur initiale appliquée, mais modifiable par l'utilisateur (ex: montage d'un lecteur réseau, ajout d'une imprimante partagée, raccourci bureau). Intègre le **ciblage au niveau de l'élément** (*Item-level targeting*) pour filtrer selon des conditions très fines (appartenance à un groupe de sécurité, plage d'adresses IP, batterie, etc.).

---

## 🩺 5. Méthode d'Application et Diagnostic

### 1. Forcer l'application immédiate
```cmd
gpupdate /force
```

### 2. Vérifier les GPO appliquées dans le terminal
```cmd
gpresult /r
```

### 3. Générer un rapport HTML complet
```cmd
gpresult /h C:\rapport-gpo.html
```
Ce rapport détaille les GPO appliquées, les GPO refusées (avec la raison : filtrage de sécurité, WMI, désactivation), et les extensions côté client (CSE) ayant échoué.
