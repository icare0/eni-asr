# Commandes Réseau, Active Directory & Diagnostic

En cas d'incident en entreprise (lenteur d'authentification, échec d'accès à un partage, problème de réplication), voici la boîte à outils des commandes de diagnostic indispensables.

---

## 1. Diagnostic Réseau & Résolution DNS

### `ipconfig`
```cmd
:: Afficher tous les détails réseau (MAC, IP, Passerelle, Serveurs DNS)
ipconfig /all

:: Vider le cache du résolveur DNS local
ipconfig /flushdns

:: Renouveler le bail DHCP
ipconfig /release
ipconfig /renew
```

### `Test-NetConnection` (PowerShell)
Le couteau suisse moderne pour tester la connectivité IP et la disponibilité d'un port TCP précis :
```powershell
# Tester si le serveur SRV1 écoute bien sur le port HTTPS 443
Test-NetConnection -ComputerName "SRV1" -Port 443

# Tester si le port LDAP (389) ou Kerberos (88) répond sur le contrôleur CD1
Test-NetConnection -ComputerName "CD1" -Port 389
```

### `nslookup` (Interrogation DNS avancée)
```cmd
:: Interroger le serveur DNS par défaut
nslookup srv1.domNN.ad

:: Interroger un serveur DNS spécifique
nslookup srv1.domNN.ad 192.168.21.1

:: Vérifier la présence d'un enregistrement de service SRV pour Active Directory
nslookup
> set type=srv
> _ldap._tcp.dc._msdcs.domNN.ad
```

---

## 2. Diagnostic Active Directory & Contrôleurs

### `nltest` (Localisation & Sécurisation de Canal)
```cmd
:: Vérifier à quel site physique la machine est rattachée
nltest /dsgetsite

:: Identifier quel contrôleur de domaine a authentifié le poste client
nltest /dsgetdc:domNN.ad

:: Vérifier l'état du canal sécurisé (Secure Channel) entre le client et le domaine
nltest /sc_query:domNN.ad
```

### `repadmin` (Diagnostic de Réplication Multi-Maîtres)
```cmd
:: Résumé rapide de la réplication sur tous les contrôleurs
repadmin /replsummary

:: Rapport détaillé des liaisons de réplication entrantes et sortantes
repadmin /showrepl

:: Forcer la réplication immédiate de toutes les partitions
repadmin /syncall /AdeP
```

### `dcdiag` (Analyseur de Santé du Contrôleur)
```cmd
:: Audit complet du contrôleur local (DNS, SYSVOL, Netlogon, FSMO)
dcdiag /v

:: Tester spécifiquement la santé du service DNS intégré
dcdiag /test:DNS
```

---

## 3. Stratégies de Groupe (GPO)

```cmd
:: Forcer l'application immédiate des GPO sans attendre le cycle de rafraîchissement
gpupdate /force

:: Afficher la liste des GPO appliquées et rejetées pour l'utilisateur et l'ordinateur
gpresult /r

:: Générer un rapport d'audit exhaustif au format HTML
gpresult /h C:\resultat-gpo.html
```

---

## 4. Métrologie & Collecteurs (`logman`)

```cmd
:: Démarrer manuellement l'ensemble de collecteurs de performances
logman start "CompteurBase"

:: Afficher la liste et l'état des collecteurs configurés
logman query

:: Stopper la collecte
logman stop "CompteurBase"
```
