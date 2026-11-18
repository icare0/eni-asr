# 🔍 Commandes Réseau, Active Directory & Diagnostic

En cas d'incident en entreprise (lenteur d'authentification, échec d'accès à un partage, problème de réplication), voici la boîte à outils des commandes de diagnostic indispensables.

---

## 🌐 1. Diagnostic Réseau & Résolution DNS

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
Le couteau suisse moderne pour tester la connectivité IP et la disponibilité d'un port TCP précis (remplace avantageusement `telnet`) :
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

## 🏛️ 2. Diagnostic Active Directory & Contrôleurs

### `nltest` (Localisation & Sécurisation de Canal)
```cmd
:: Vérifier à quel site physique la machine est rattachée
nltest /dsgetsite

:: Identifier quel contrôleur de domaine a authentifié le poste client
nltest /dsgetdc:domNN.ad

:: Vérifier l'état du canal sécurisé (Secure Channel) entre le client et le domaine
nltest /sc_query:domNN.ad
```

### `netdom` (Gestion de Domaine)
```cmd
:: Connaître les détenteurs des 5 rôles FSMO
netdom query fsmo

:: Vérifier l'état de confiance d'un ordinateur membre
netdom verify Win-CL1 /domain:domNN.ad
```

### `dcdiag` (Diagnostic de Santé des Contrôleurs)
Exécute une batterie complète de tests d'intégrité sur les contrôleurs de domaine :
```cmd
dcdiag /v /c /d /e
```

### `repadmin` (Audit & Synchronisation de la Réplication)
```cmd
:: Afficher l'état de réplication entre CD1 et CD2
repadmin /showrepl

:: Forcer une réplication immédiate sur tous les partenaires
repadmin /syncall /AePd
```

---

## 🔌 3. Diagnostic des Ports & Connexions Locales

### `netstat`
```cmd
:: Lister tous les ports en écoute avec le PID du processus associé
netstat -ano | findstr "LISTENING"

:: Trouver quel programme utilise le port 443
netstat -ano | findstr ":443"
```
*(Le PID renvoyé en dernière colonne peut ensuite être identifié dans le Gestionnaire des tâches ou via `Get-Process -Id <PID>`)*.
