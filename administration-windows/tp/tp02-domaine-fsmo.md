# TP 02 — Création du Domaine Active Directory & Rôles FSMO

Fiche de synthèse de l'atelier pratique n°2 : promotion du premier contrôleur de domaine, intégration de la haute disponibilité avec un contrôleur réplica, jonction des clients et répartition des rôles FSMO.

---

## 1. Contexte & Périmètre de l'Atelier

* **Machines mobilisées** :
  * `CD1` : Contrôleur de domaine racine (`192.168.21.1/16`)
  * `CD2` : Second contrôleur de domaine réplica (`192.168.21.2/16`)
  * `SRV1` : Serveur membre (`192.168.21.10/16`)
  * `Win-CL1` : Poste client Windows 11 (`192.168.21.20/16`)
* **Nom de domaine Active Directory** : `domNN.ad` (avec zone DNS intégrée).
* **Objectif global** : Construire l'annuaire d'entreprise, assurer sa tolérance aux pannes via deux contrôleurs et optimiser la répartition des charges FSMO.

---

## 2. Réalisations Techniques Clés

### Promotion du Contrôleur Racine (CD1)
* Simulation préalable des prérequis avec `Test-ADDSForestInstallation`.
* Promotion de `CD1` en contrôleur de domaine racine de forêt `domNN.ad`.
* Configuration automatique de la zone DNS intégrée à Active Directory et création des partitions d'annuaire.

### Haute Disponibilité : Déploiement de CD2
* Déploiement de la machine virtuelle `CD2` (`192.168.21.2/16`) avec DNS pointant prioritairement vers `CD1`.
* Jonction au domaine `domNN.ad` puis promotion en second contrôleur de domaine réplica.
* Validation de la réplication multi-maîtres bidirectionnelle entre `CD1` et `CD2` (base d'annuaire et partage SYSVOL).

### Intégration des Membres & Console RSAT
* Jonction au domaine du serveur de stockage `SRV1` et du poste de travail client `Win-CL1`.
* Installation des outils d'administration de serveur distant (**RSAT**) sur le poste `Win-CL1` :
  * Activation de la console *Centre d'administration Active Directory* (ADAC) et de la console *Utilisateurs et ordinateurs Active Directory*.
  * Validation de l'administration déportée depuis le poste client avec un compte de domaine habilité.

### Répartition Stratégique des 5 Rôles FSMO
* À l'origine, `CD1` hébergeait l'ensemble des 5 rôles FSMO.
* Transfert gracieux des deux rôles les plus sollicités en production depuis `CD1` vers `CD2` :
  * **Maître RID** (*Relative Identifier*) transféré vers `CD2`.
  * **Émulateur PDC** (*Primary Domain Controller*) transféré vers `CD2`.
* `CD1` conserve les rôles de forêt (**Schéma**, **Attribution de noms**) et le rôle **Infrastructure**.

---

## 3. Commandes de Validation & Contrôle

```powershell
# Vérifier la répartition des 5 rôles FSMO sur les deux contrôleurs
Get-ADForest "domNN.ad" | Select-Object SchemaMaster, DomainNamingMaster
Get-ADDomain "domNN.ad" | Select-Object InfrastructureMaster, PDCEmulator, RIDMaster

# Contrôler l'état de santé de la réplication Active Directory
repadmin /replsummary
repadmin /showrepl

# Vérifier la résolution des enregistrements de service SRV indispensables
nslookup -type=srv _ldap._tcp.dc._msdcs.domNN.ad
nslookup -type=srv _kerberos._tcp.dc._msdcs.domNN.ad

# Vérifier la présence des contrôleurs de domaine dans l'annuaire
Get-ADDomainController -Filter * | Select-Object Name, IPv4Address, Site, IsGlobalCatalog
```
