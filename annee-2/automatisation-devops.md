# 5. Automatisation, Scripting & Culture DevOps pour l'Administrateur

L'administrateur système moderne ne clique plus 50 fois dans une interface graphique pour créer 50 comptes utilisateurs ou configurer 20 serveurs. Il utilise l'**automatisation en code** pour éliminer les erreurs humaines, standardiser les déploiements et gagner un temps précieux.

---

## ⚡ 1. PowerShell Avancé : La Puissance des Objets

Contrairement aux shells Unix historiques qui manipulent de simples flux de texte brut, **PowerShell manipule des objets .NET complets**.

```
[ Commande 1 : Get-Process ]
            │
            ▼ (Transmet un tableau d'objets System.Diagnostics.Process)
[ Pipeline | ]
            │
            ▼
[ Commande 2 : Where-Object {$_.CPU -gt 50} ]
            │
            ▼
[ Commande 3 : Stop-Process -Confirm ]
```

### Script robuste avec gestion des exceptions (`try / catch`)
```powershell
[CmdletBinding()]
param (
    [Parameter(Mandatory=$true)]
    [string]$NomUtilisateur
)

try {
    Write-Verbose "Recherche de l'utilisateur $NomUtilisateur dans l'annuaire..."
    $user = Get-ADUser -Identity $NomUtilisateur -Properties AccountExpirationDate -ErrorAction Stop
    
    # Prolongation du compte de 6 mois
    $nouvelleDate = (Get-Date).AddMonths(6)
    Set-ADAccountExpiration -Identity $user -DateTime $nouvelleDate
    
    Write-Host "[+] Compte $NomUtilisateur prolongé avec succès jusqu'au $nouvelleDate" -ForegroundColor Green
}
catch [Microsoft.ActiveDirectory.Management.ADIdentityNotFoundException] {
    Write-Warning "[-] L'utilisateur $NomUtilisateur n'existe pas dans le domaine !"
}
catch {
    Write-Error "[-] Erreur critique inattendue : $_"
}
```

---

## 🤖 2. Gestion de Configuration avec Ansible

**Ansible** est l'outil d'automatisation de référence pour configurer des parcs hétérogènes (Linux et Windows) sans installer le moindre agent logiciel lourd :
* Utilise **SSH** pour piloter les machines Linux.
* Utilise **WinRM** (ou OpenSSH) pour piloter les serveurs Windows.
* Fonctionne sur le principe de l'**idempotence** : exécuter le script 10 fois de suite produit exactement le même résultat sans altérer l'existant.

### Exemple de Playbook Ansible (`deploy-web.yml`) :
```yaml
---
- name: Déploiement standardisé du serveur Web d'entreprise
  hosts: serveurs_web
  become: yes

  tasks:
    - name: Installation du serveur Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Déploiement de la configuration de sécurité Nginx
      copy:
        src: files/nginx.conf
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: '0644'
      notify: Redémarrer Nginx

  handlers:
    - name: Redémarrer Nginx
      service:
        name: nginx
        state: restarted
```

---

## 🐳 3. Conteneurisation avec Docker

Alors qu'une machine virtuelle virtualise l'ensemble du matériel (matériel virtuel + OS complet de 15 Go), un conteneur **Docker** isole uniquement le processus applicatif en partageant le noyau du système hôte :

```
┌─────────────────────────────────┐       ┌─────────────────────────────────┐
│     MACHINE VIRTUELLE (VM)      │       │        CONTENEUR DOCKER         │
├─────────────────────────────────┤       ├─────────────────────────────────┤
│ Application + Bibliothèques     │       │ Application + Bibliothèques     │
│ Système d'Exploitation Invité   │       │ Moteur de Conteneurs Docker     │
│ Hyperviseur (Type 1 ou 2)       │       │ Système d'Exploitation Hôte     │
│ Matériel Physique               │       │ Matériel Physique               │
└─────────────────────────────────┘       └─────────────────────────────────┘
  (Lourd : 10-40 Go, démarrage 1 min)       (Ultra-léger : 50 Mo, démarrage 1 s)
```

---

## 🐙 4. Git pour les Administrateurs Systèmes (IaC)

Pourquoi un administrateur système utilise-t-il Git ?
* **Historique des modifications** : Savoir exactement qui a modifié telle règle de pare-feu ou tel script de sauvegarde il y a 3 semaines.
* **Rollback instantané** : Revenir en arrière en 1 seconde en cas de bogue dans un script déployé.
* **Collaboration d'équipe** : Travailler à plusieurs sur les configurations réseau et d'infrastructure sans risque d'écraser le travail de ses collègues.
