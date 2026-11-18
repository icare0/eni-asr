# Atelier 02 — Création d’un Domaine Active Directory & Rôles FSMO

Cet atelier est dédié à l'initialisation de l'annuaire d'entreprise, à la mise en place de la tolérance aux pannes et à la répartition stratégique des rôles FSMO.

---

## 🎯 Objectifs Opérationnels
1. Déployer la racine de forêt `domNN.ad` en PowerShell sur le contrôleur principal CD1.
2. Déployer un second contrôleur de domaine réplica (CD2) pour la haute disponibilité.
3. Intégrer les serveurs membres et le poste client d'administration RSAT au domaine.
4. Équilibrer les rôles FSMO entre les deux contrôleurs et certifier la configuration avec `netdom`.

---

## ⚡ Étape 1 : Promotion du Premier Contrôleur de Domaine (CD1)

Toutes les étapes s'exécutent dans une console PowerShell ouverte en tant qu'Administrateur sur **CD1** :

```powershell
# 1. Installation du rôle AD DS et des consoles d'administration
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# 2. Validation préalable des prérequis de la forêt
Test-ADDSForestInstallation -DomainName "domNN.ad"

# 3. Promotion effective de la forêt racine (avec DNS intégré)
Install-ADDSForest -DomainName "domNN.ad" -InstallDns:$true
```
*Le serveur redémarre automatiquement à l'issue de la promotion.*

---

## 🛡️ Étape 2 : Déploiement du Contrôleur Additionnel (CD2)

### Spécifications de la VM CD2
* OS : Windows Server 2022
* Ressources : 1 vCPU, 4 Go RAM, disque 40 Go
* Réseau : IP `192.168.21.2 /16`, Passerelle `192.168.0.1`, DNS primaire `192.168.21.1` (CD1)

### Commandes PowerShell sur CD2
```powershell
# 1. Jonction au domaine existant
Add-Computer -DomainName "domNN.ad" -Restart

# 2. Après redémarrage, installation du rôle AD DS
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# 3. Promotion en contrôleur de domaine additionnel (réplica)
Install-ADDSDomainController -DomainName "domNN.ad" -InstallDns:$true
```

---

## 💻 Étape 3 : Intégration des Membres & Administration RSAT

1. **Jonction au domaine de SRV1 et Win-CL1** :
   ```powershell
   Add-Computer -DomainName "domNN.ad" -Restart
   ```
2. **Poste d'administration Win-CL1** :
   * Activer les fonctionnalités RSAT (*Outils d'administration de serveur distant*) dans les fonctionnalités facultatives de Windows 11.
   * Ouvrir le **Centre d'administration Active Directory (ADAC)** directement depuis le poste client.
   * Créer un premier compte utilisateur de test pour valider que l'administration distante opère sans avoir à ouvrir de session sur les contrôleurs.

---

## 👑 Étape 4 : Répartition & Vérification des Rôles FSMO

Pour décharger CD1 et répartir la charge d'authentification et de création d'objets, nous transférons les rôles **Maître RID** et **Émulateur PDC** vers **CD2**.

### Commande de transfert PowerShell (exécutée depuis Win-CL1 ou CD1) :
```powershell
Move-ADDirectoryServerOperationMasterRole -Identity "CD2" `
                                        -OperationMasterRole RidMaster, PDCEmulator
```

### Certification des détenteurs de rôles :
```cmd
netdom query fsmo
```

### Résultat attendu :
```text
Maître de schéma               CD1.domNN.ad
Maître d'attribution de noms   CD1.domNN.ad
Émulateur PDC                  CD2.domNN.ad
Maître RID                     CD2.domNN.ad
Maître d'infrastructure        CD1.domNN.ad
La commande s'est terminée correctement.
```
