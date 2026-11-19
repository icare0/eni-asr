[← Retour au Sommaire des Cours](README.md) | [Accueil](../../README.md)

# 4. Stratégies de Groupe (GPO) & Sécurité du Domaine (PSO)

Ce cours analyse le moteur des Stratégies de Groupe (*Group Policy Objects* - GPO), l'ordonnancement de leur traitement hiérarchique, le déploiement du magasin central SYSVOL et la mise en œuvre des politiques de mot de passe à granularité fine (**PSO**).

---

## 1. Rôle et Moteur des Stratégies de Groupe (GPO)

Les Stratégies de Groupe constituent l'outil central de gouvernance sous Windows Server. Elles permettent de configurer, restreindre et maintenir en conformité des milliers de machines et de sessions utilisateurs depuis une console unique (`gpmc.msc`).

Une GPO est composée de deux parties :
* **Un conteneur dans l'annuaire (GPC - *Group Policy Container*)** : Stocké dans la base LDAP, il contient les attributs, le numéro de version et l'état de la stratégie.
* **Un ensemble de fichiers (GPT - *Group Policy Template*)** : Stocké dans le partage répliqué `SYSVOL` (`\\domaine\SYSVOL\domaine\Policies\{GUID}`), il contient les scripts, modèles administratifs et fichiers de paramétrage réels.

---

## 2. Ordre de Traitement Hiérarchique (Règle LSDOU)

Lors du démarrage d'un ordinateur ou de l'ouverture de session d'un utilisateur, l'agent Windows évalue et applique les stratégies selon un ordre séquentiel strict, désigné par l'acronyme **LSDOU** :

```
1. Local (Stratégie de sécurité locale configurée sur la machine)
      │
      ▼
2. Site (GPO liée au site physique Active Directory du client)
      │
      ▼
3. Domaine (GPO liée à la racine du domaine d'appartenance)
      │
      ▼
4. Unité d'Organisation (OU Parente puis sous-UO de manière descendante)
```

### Règles de Résolution des Conflits
* **Règle générale (*Last-Writer-Wins*)** : Si deux stratégies définissent des paramètres divergents, **la dernière stratégie appliquée l'emporte toujours**. Une GPO liée à une Unité d'Organisation fine écrase donc un paramètre défini à la racine du domaine.
* **Option « Appliquer » (*Enforced*)** : Activée sur un lien de GPO supérieur (ex: au niveau du domaine), elle force la priorité absolue de cette GPO, empêchant toute UO inférieure de la surcharger.
* **Option « Bloquer l'héritage » (*Block Inheritance*)** : Positionnée sur une UO, elle empêche les GPO des niveaux supérieurs de s'appliquer aux objets de cette UO (sauf pour les GPO marquées comme *Enforced*).

---

## 3. Techniques de Filtrage et Portée d'Application

Pour appliquer des réglages chirurgicaux sans multiplier inutilement les Unités d'Organisation, l'administrateur dispose de trois mécanismes de ciblage :

### 1. Le Filtrage de Sécurité
Par défaut, une GPO s'applique à l'ensemble du groupe `Utilisateurs authentifiés` (tous les utilisateurs et ordinateurs du domaine).  
En retirant ce groupe et en assignant un groupe de sécurité spécifique (ex: `G-Equipe-Support`), seuls les membres de ce groupe recevront les paramètres.  
*Condition requise* : Le groupe doit posséder les permissions **Lire** et **Appliquer la stratégie de groupe** sur l'onglet Sécurité de la GPO.

### 2. Le Ciblage par Élément (*Item-Level Targeting*)
Disponible dans les **Préférences de GPO** (ex: mappage automatique de lecteurs réseau, imprimantes, raccourcis), ce ciblage évalue des conditions logiques riches :
* Plage d'adresses IP ou sous-réseau du poste.
* Type de matériel ou batterie présente (ex: appliquer un réglage d'énergie uniquement aux ordinateurs portables).
* Présence d'un fichier, d'une clé de registre ou d'une version de système d'exploitation.

### 3. Les Filtres WMI (*Windows Management Instrumentation*)
Un filtre WMI conditionne l'exécution de la GPO à une requête WQL évaluée directement sur le système d'exploitation client :
```sql
-- Exemple : Appliquer la stratégie uniquement aux machines sous Windows 11
SELECT * FROM Win32_OperatingSystem WHERE Version LIKE "10.0.22%" AND ProductType = "1"
```

---

## 4. Le Magasin Central des Modèles d'Administration (SYSVOL)

Les paramètres de GPO reposent sur des modèles d'administration :
* **Fichiers `.admx`** : Structure et logique des paramètres en XML neutre.
* **Fichiers `.adml`** : Traductions textuelles dans une langue donnée (ex: sous-dossier `fr-FR`).

### Pourquoi Créer le Magasin Central (*Central Store*) ?
Par défaut, la console GPMC charge les modèles locaux situés sur le poste de l'administrateur (`C:\Windows\PolicyDefinitions`). Si des administrateurs utilisent des versions de Windows différentes, des erreurs d'incompatibilité de modèles surviennent.

* **La bonne pratique d'architecture** : Créer le dossier central au sein du partage répliqué Active Directory :  
  `\\domaine\SYSVOL\domaine\Policies\PolicyDefinitions`
* **Résultat** : Toutes les consoles GPMC du domaine chargent automatiquement la même version de référence des modèles, répliquée sur tous les contrôleurs de domaine.

---

## 5. Sécurité des Mots de Passe : Pourquoi les GPO d'UO Échouent ?

### L'Erreur Fréquente en Entreprise
Beaucoup d'administrateurs tentent de lier une GPO sur une UO (ex: `OU=Direction` ou `OU=Informatique`) pour exiger des mots de passe de 14 caractères au lieu de 10. **Ce paramétrage est strictement ignoré par Windows pour tous les comptes de domaine !**

* **Pourquoi ?** L'authentification des comptes de domaine est assurée exclusivement par le KDC Kerberos sur les contrôleurs de domaine.  
* Il ne peut exister qu'**une seule politique de mot de passe globale par domaine**, et elle **doit impérativement être liée à la racine du domaine** (la stratégie par défaut `Default Domain Policy`).

### La Solution d'Entreprise : Les Stratégies Affinées (PSO)
Pour appliquer des politiques renforcées à des profils sensibles (administrateurs, informaticiens, comptables), Microsoft a introduit les **PSO** (*Password Settings Objects*) :

```
┌────────────────────────────────────────────────────────────┐
│ POLITIQUE DU DOMAINE (Liaison Racine)                      │
│ • Longueur min : 10 caractères                             │
│ • Verrouillage : 5 échecs                                  │
│ • S'applique à 100% des comptes par défaut                 │
└─────────────────────────────┬──────────────────────────────┘
                              │
                              ▼ Remplacée prioritairement par
┌────────────────────────────────────────────────────────────┐
│ STRATÉGIE AFFINÉE PSO (Cible : Groupe Administrateurs)    │
│ • Précédence : 1 (Priorité absolue)                        │
│ • Longueur min : 14 caractères                             │
│ • Verrouillage : 2 tentatives infructueuses seulement      │
│ • Déverrouillage : Exige l'action manuelle d'un confrère   │
└────────────────────────────────────────────────────────────┘
```

```powershell
# Créer une stratégie de mot de passe affinée (PSO) en PowerShell
New-ADFineGrainedPasswordPolicy -Name "PSO-Comptes-Sensibles" `
                                -Precedence 1 `
                                -ComplexityEnabled $true `
                                -MinPasswordLength 14 `
                                -PasswordHistoryCount 24 `
                                -LockoutThreshold 2 `
                                -LockoutDuration "00:00:00" `
                                -ResetLockoutCounterAfter "00:30:00"

# Lier la PSO au groupe de sécurité des comptes à privilèges
Add-ADFineGrainedPasswordPolicySubject -Identity "PSO-Comptes-Sensibles" `
                                      -Subjects "G-Administrateurs"
```

---

## 6. Diagnostic et Audit des GPO sur un Poste Client

Lorsqu'une stratégie ne s'applique pas comme prévu, l'administrateur utilise la boîte à outils native :

```powershell
# Forcer la réapplication immédiate des GPO sans attendre le cycle automatique (90 min)
gpupdate /force

# Visualiser en console les GPO appliquées et rejetées pour l'ordinateur et l'utilisateur
gpresult /r

# Générer un rapport d'audit exhaustif au format HTML
gpresult /h C:\audit-gpo.html
```

---

[← Chapitre précédent : 3. Stockage d'Entreprise & Modèle AGDLP](03-stockage-partages-agdlp.md) | [Atelier pratique associé : TP 05](../tp/tp05-gpo-securite-pso.md) | [Chapitre suivant : 5. Métrologie, Supervision & Événements →](05-metrologie-supervision-evenements.md)
