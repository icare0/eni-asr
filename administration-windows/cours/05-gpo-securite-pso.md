[← Retour aux Cours](README.md) | [Accueil](../../README.md)

# 5. Stratégies de Groupe (GPO), Sécurité des Mots de Passe (PSO) & SYSVOL

Ce chapitre analyse le fonctionnement du moteur des Stratégies de Groupe (*Group Policy Objects* - GPO), l'ordonnancement de leur traitement, le déploiement du magasin central SYSVOL et la mise en œuvre des politiques de mot de passe à granularité fine (**PSO**).

---

## 1. Moteur de Traitement et Ordre d'Application (LSDOU)

Les Stratégies de Groupe permettent de centraliser la configuration, la restriction et le durcissement de l'ensemble des postes de travail et serveurs d'un domaine Active Directory.

Lors du démarrage d'une machine ou de l'ouverture de session d'un utilisateur, l'agent Windows évalue et applique les GPO selon un ordre hiérarchique invariable, désigné par l'acronyme **LSDOU** :

```
1. Local (Stratégie de sécurité locale définie sur le poste)
      │
      ▼
2. Site (GPO liée au site physique Active Directory)
      │
      ▼
3. Domaine (GPO liée à la racine du domaine Active Directory)
      │
      ▼
4. Unité d'Organisation (OU Parente puis sous-UO de manière descendante)
```

### Règles de Résolution des Conflits de Paramètres
* **Règle générale (*Last-Writer-Wins*)** : Si deux stratégies appliquent une valeur divergente pour un paramètre identique, **la dernière stratégie traitée l'emporte toujours**. Une GPO liée à l'Unité d'Organisation finale surcharge donc les paramètres définis à la racine du domaine.
* **Option « Appliquer » (*Enforced*)** : Placée sur un lien de GPO supérieur (ex: au niveau du domaine), elle neutralise l'écrasement par les niveaux inférieurs. Le paramètre devient prioritaire sur toutes les UO enfants.
* **Option « Bloquer l'héritage » (*Block Inheritance*)** : Positionnée sur une UO, elle interrompt la transmission descendante des GPO des niveaux supérieurs (sauf pour celles marquées comme *Enforced*).

---

## 2. Techniques de Filtrage et Portée d'Application

Une GPO ne doit s'appliquer qu'aux cibles appropriées. Pour ajuster sa portée au-delà de la simple liaison à une UO, l'administrateur dispose de trois leviers :

### 1. Le Filtrage de Sécurité
Par défaut, une GPO s'applique au groupe `Utilisateurs authentifiés` (qui englobe tous les comptes utilisateurs et ordinateurs du domaine).
* En supprimant `Utilisateurs authentifiés` et en ajoutant un groupe de sécurité spécifique (ex: `G-Intérimaires` ou `G-Informatique`), seuls les membres de ce groupe recevront les paramètres de la GPO.
* *Condition requise* : Le groupe cible doit posséder les permissions **Lire** et **Appliquer la stratégie de groupe** sur l'onglet Sécurité de la GPO.

### 2. Le Ciblage par Élément (*Item-Level Targeting*)
Disponible dans les **Préférences de GPO** (ex: mappage de lecteurs réseau, déploiement d'imprimantes, création de raccourcis), le ciblage par élément permet d'évaluer des conditions dynamiques très riches sans multiplier les GPO :
* Appartenance à un groupe de sécurité.
* Plage d'adresses IP du client.
* Présence d'un fichier spécifique ou clé de registre.
* Type de système d'exploitation ou quantité de mémoire vive.

### 3. Les Filtres WMI (*Windows Management Instrumentation*)
Un filtre WMI soumet l'application de la GPO à une requête WQL exécutée sur la machine cible :
```sql
-- Exemple : Appliquer la GPO uniquement aux machines sous Windows 11
SELECT * FROM Win32_OperatingSystem WHERE Version LIKE "10.0.22%" AND ProductType = "1"
```

---

## 3. Le Magasin Central des Modèles d'Administration (SYSVOL)

Les paramètres des GPO reposent sur des fichiers de modèles d'administration :
* **Fichiers `.admx`** : Définissent la structure et la logique des paramètres indépendamment de la langue (fichier XML neutre).
* **Fichiers `.adml`** : Contiennent les descriptions et traductions textuelles dans une langue donnée (ex: sous-dossier `fr-FR` ou `en-US`).

### Pourquoi Déployer le Magasin Central (*Central Store*) ?
Par défaut, les modèles sont lus localement depuis le poste d'administration (`C:\Windows\PolicyDefinitions`). Si des administrateurs travaillent avec des versions de Windows différentes, des erreurs d'incompatibilité et d'écrasement de modèles surviennent.

* **Solution d'architecture** : Créer le dossier central dans le partage SYSVOL répliqué :
  `\\<domaine.ad>\SYSVOL\<domaine.ad>\Policies\PolicyDefinitions`
* **Bénéfice** : Dès lors que ce dossier existe, toutes les consoles GPMC du domaine chargent automatiquement la même version de référence des modèles d'administration, répliquée sur l'ensemble des contrôleurs de domaine.

---

## 4. Sécurité des Mots de Passe : Pourquoi la GPO d'OU Échoue-t-elle ?

### La Limitation Architecturale de Windows Server
Beaucoup d'administrateurs commettent l'erreur de lier une GPO sur une Unité d'Organisation (ex: `OU=Direction`) pour exiger des mots de passe plus longs pour les cadres ou les techniciens. **Cette stratégie est totalement ignorée par Windows pour tous les comptes de domaine !**

* **Explication technique** : L'authentification des comptes de domaine est assurée exclusivement par le composant KDC (*Kerberos Key Distribution Center*) sur les contrôleurs de domaine.
* Il ne peut exister qu'**une seule politique de mot de passe globale par domaine**, et celle-ci **doit impérativement être liée à la racine du domaine** (habituellement la stratégie par défaut `Default Domain Policy`).

### La Solution d'Entreprise : Les Stratégies Affinées (PSO)

Pour imposer des règles de complexité différentes selon les profils (ex: 12 caractères pour l'équipe informatique contre 10 pour les utilisateurs standard), Microsoft a introduit les **PSO** (*Password Settings Objects*).

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
│ STRATÉGIE AFFINÉE PSO (Cible : G-Informatique / Admin)    │
│ • Précédence : 1 (Priorité absolue)                        │
│ • Longueur min : 12 caractères                             │
│ • Historique : 24 mots de passe                            │
│ • Verrouillage : 2 échecs seulement                        │
│ • Déverrouillage : Obligatoirement manuel par administrateur│
└────────────────────────────────────────────────────────────┘
```

```powershell
# Création d'une politique PSO renforcée pour les comptes à privilèges
New-ADFineGrainedPasswordPolicy -Name "PSO-Informatique" `
                                -Precedence 1 `
                                -ComplexityEnabled $true `
                                -MinPasswordLength 12 `
                                -PasswordHistoryCount 24 `
                                -LockoutThreshold 2 `
                                -LockoutDuration "00:00:00" `
                                -ResetLockoutCounterAfter "00:30:00"

# Affectation de la PSO au groupe des administrateurs et informaticiens
Add-ADFineGrainedPasswordPolicySubject -Identity "PSO-Informatique" `
                                      -Subjects "G-Informatique"
```

---

## 5. Diagnostic et Audit des GPO sur les Clients

Lorsqu'une stratégie de groupe ne produit pas le comportement attendu sur un poste client, l'administrateur procède par étapes méthodiques :

```powershell
# 1. Forcer la réapplication immédiate des stratégies sans attendre le cycle normal (90 min)
gpupdate /force

# 2. Afficher la liste des GPO appliquées et rejetées pour l'ordinateur et l'utilisateur connecté
gpresult /r

# 3. Générer un rapport HTML exhaustif visualisable dans le navigateur
gpresult /h C:\resultat-gpo.html
```

Le rapport HTML détaille pour chaque paramètre la GPO victorieuse, les GPO écrasées et les motifs de rejet (ex: *Refusé par filtrage de sécurité*, *Filtre WMI non satisfait*, ou *Héritage bloqué*).

---

[← Chapitre précédent : 4. Stockage d'Entreprise & AGDLP](04-stockage-agdlp.md) | [Atelier pratique associé : TP 05](../tp/tp05-gpo-securite-pso.md) | [Sommaire général des Cours](README.md)

