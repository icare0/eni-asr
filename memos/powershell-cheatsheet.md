# Mémento PowerShell pour Administrateur Sysadmin

Ce mémento rassemble les cmdlets PowerShell les plus utilisées en administration d'infrastructure Windows Server et Active Directory.

---

## 1. Gestion des Utilisateurs Active Directory

```powershell
# Rechercher un utilisateur avec ses propriétés spécifiques
Get-ADUser -Identity "dgrenier" -Properties Department, Title, Mail, AccountExpirationDate

# Lister tous les utilisateurs d'un service précis
Get-ADUser -Filter 'Department -eq "Comptabilité"' -SearchBase "OU=Comptabilité,OU=Utilisateurs,OU=Société ASR,DC=domNN,DC=ad"

# Déverrouiller un compte verrouillé par la politique de sécurité
Unlock-ADAccount -Identity "dgrenier"

# Désactiver un compte lors du départ d'un collaborateur
Disable-ADAccount -Identity "ctalmie"

# Activer un compte utilisateur
Enable-ADAccount -Identity "ctalmie"

# Modifier un attribut spécifique
Set-ADUser -Identity "dgrenier" -Title "Directeur Administratif et Financier" -OfficePhone "504"

# Définir une date de fin de validité pour un compte temporaire/intérimaire
Set-ADAccountExpiration -Identity "ctalmie" -DateTime (Get-Date "31/12/2026")
```

---

## 2. Gestion des Groupes & Membres

```powershell
# Créer un groupe global de sécurité
New-ADGroup -Name "G-Comptabilite" `
            -GroupScope Global `
            -GroupCategory Security `
            -Path "OU=Groupes,OU=Société ASR,DC=domNN,DC=ad"

# Ajouter un membre dans un groupe
Add-ADGroupMember -Identity "G-Comptabilite" -Members "curique", "ctalmie"

# Lister tous les membres directs d'un groupe
Get-ADGroupMember -Identity "G-Informatique" | Select-Object Name, SamAccountName

# Retirer un utilisateur d'un groupe
Remove-ADGroupMember -Identity "G-Comptabilite" -Members "ctalmie" -Confirm:$false
```

---

## 3. Gestion du Stockage & Partages SMB

```powershell
# Lister les disques physiques bruts disponibles
Get-Disk | Where-Object OperationalStatus -eq 'Online'

# Initialiser un nouveau disque en table de partition GPT
Initialize-Disk -Number 1 -PartitionStyle GPT

# Créer une partition NTFS et lui assigner une lettre de lecteur
New-Partition -DiskNumber 1 -UseMaximumSize -AssignDriveLetter | Format-Volume -FileSystem NTFS -NewFileSystemLabel "DATA"

# Créer un partage SMB réseau sécurisé
New-SmbShare -Name "Compta" `
             -Path "E:\DATA\Compta" `
             -FullAccess "domNN\DL-Compta-ControleTotal" `
             -ChangeAccess "domNN\DL-Compta-Modification"

# Créer un partage administratif masqué (se terminant par $)
New-SmbShare -Name "Info$" `
             -Path "E:\DATA\Info$" `
             -FullAccess "domNN\DL-Info-ControleTotal"

# Lister les sessions SMB ouvertes actuellement sur le serveur de fichiers
Get-SmbSession | Select-Object ClientComputerName, ClientUserName, NumOpens
```

---

## 4. Diagnostics Système & Réseau

```powershell
# Tester la connectivité vers un port TCP précis (ex: port HTTPS de Windows Admin Center)
Test-NetConnection -ComputerName "SRV1" -Port 443

# Tester le port LDAP (389) ou Kerberos (88) sur un contrôleur de domaine
Test-NetConnection -ComputerName "CD1" -Port 389

# Redémarrer un service distant
Restart-Service -Name "Spooler" -Force

# Exécuter une commande à distance via PowerShell Remoting (WinRM)
Invoke-Command -ComputerName "SRV1" -ScriptBlock { Get-Service -Name "WAC*" }
```
