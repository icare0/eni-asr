# 📜 Mémento PowerShell pour Administrateur Sysadmin

Ce mémento rassemble les cmdlets PowerShell les plus utilisées en administration d'infrastructure Windows Server et Active Directory.

---

## 👥 1. Gestion des Utilisateurs Active Directory

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

## 👥 2. Gestion des Groupes & Membres

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

## 📂 3. Gestion du Stockage & Partages SMB

```powershell
# Lister les disques physiques bruts disponibles
Get-Disk | Where-Object OperationalStatus -eq 'Online'

# Initialiser un disque neuf en table de partitionnement GPT
Initialize-Disk -Number 1 -PartitionStyle GPT

# Créer une partition occupant tout l'espace et assigner une lettre
New-Partition -DiskNumber 1 -UseMaximumSize -AssignDriveLetter | Format-Volume -FileSystem NTFS -NewFileSystemLabel "DATA"

# Créer un partage SMB avec droits d'accès initiaux
New-SmbShare -Name "Documentation" `
             -Path "E:\DATA\Documentation" `
             -FullAccess "domNN\DL-Documentation-ControleTotal" `
             -ReadAccess "domNN\DL-Documentation-Lecture"

# Lister tous les partages SMB actifs sur le serveur
Get-SmbShare | Where-Object Special -eq $false
```

---

## ⚙️ 4. Gestion des Services & Processus

```powershell
# Vérifier l'état d'un service (ex. Spouleur d'impression ou DNS)
Get-Service -Name "Spooler", "DNS"

# Redémarrer un service planté
Restart-Service -Name "Spooler" -Force

# Configurer le démarrage d'un service en Automatique
Set-Service -Name "W32Time" -StartupType Automatic
```

---

## 🌐 5. Administration à Distance (PowerShell Remoting)

```powershell
# Ouvrir une session interactive distante sur un serveur
Enter-PSSession -ComputerName "SRV1"

# Exécuter un script ou une commande en parallèle sur 20 serveurs
Invoke-Command -ComputerName "CD1", "CD2", "SRV1" -ScriptBlock {
    Get-Service -Name "W32Time" | Select-Object PSComputerName, Status, StartType
}
```
