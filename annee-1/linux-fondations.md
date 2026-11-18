# 7. Découverte & Administration Linux

Bien que Windows domine les postes de travail et l'annuaire d'entreprise, Linux propulse l'immense majorité des serveurs web, bases de données, appliances réseau et infrastructures Cloud. L'administrateur ASR se doit d'être polyvalent et à l'aise sur les deux environnements.

---

## 🐧 1. Distributions d'Entreprise & Arborescence FHS

En milieu professionnel, deux grandes familles dominent :
* **Famille Debian / Ubuntu** : Gestionnaire de paquets `apt` (`.deb`).
* **Famille Red Hat (RHEL / Rocky Linux / AlmaLinux)** : Gestionnaire de paquets `dnf` / `rpm`.

### L'Arborescence Standard FHS (Filesystem Hierarchy Standard)
```
/
├── bin & sbin  -> Commandes exécutables essentielles (administrateur et utilisateurs)
├── boot        -> Fichiers de démarrage du noyau Linux (vmlinuz, initramfs, grub)
├── etc         -> Fichiers de configuration globale du système et des services
├── home        -> Répertoires personnels des utilisateurs standards
├── root        -> Répertoire personnel du super-utilisateur (root)
├── var         -> Données dynamiques (journaux dans /var/log, spool, caches)
├── tmp         -> Fichiers temporaires nettoyés au redémarrage
└── opt & srv   -> Applications tierces et données hébergées
```

---

## 🔒 2. Permissions & Droits Fichiers (POSIX)

Chaque fichier ou répertoire sous Linux possède un **propriétaire (u)**, un **groupe propriétaire (g)** et s'applique aux **autres (o)** :

| Droit | Valeur Symbolique | Valeur Octale | Effet sur un fichier | Effet sur un répertoire |
| :--- | :--- | :--- | :--- | :--- |
| **Lecture** | `r` | `4` | Lire le contenu | Lister les fichiers du dossier (`ls`) |
| **Écriture** | `w` | `2` | Modifier/écraser le fichier | Créer ou supprimer des fichiers dans le dossier |
| **Exécution** | `x` | `1` | Lancer le script/binaire | Traverser le dossier (`cd`) |

### Exemples pratiques de commandes :
```bash
# Attribuer les droits rwxr-xr-- (754) à un script
chmod 754 script_backup.sh

# Rendre propriétaire l'utilisateur 'admin-sys' et le groupe 'techniciens'
chown admin-sys:techniciens /opt/donnees/
```

---

## ⚙️ 3. Gestion des Services avec Systemd (`systemctl`)

Sur les distributions modernes, `systemd` est le gestionnaire de système et de session standard :

```bash
# Vérifier l'état d'un service (ex: serveur SSH ou Apache)
sudo systemctl status sshd

# Démarrer ou redémarrer un service
sudo systemctl restart apache2

# Activer un service pour qu'il se lance au démarrage de la machine
sudo systemctl enable nginx
```

---

## 📜 4. Scripting Bash pour l'Automatisation

Un administrateur système ne répète jamais manuellement une tâche récurrente :

```bash
#!/bin/bash
# Script de sauvegarde simple avec contrôle d'erreur

SOURCE="/etc/nginx"
DEST="/var/backups/nginx_$(date +%Y%m%d).tar.gz"

echo "[*] Début de la sauvegarde de $SOURCE..."
tar -czf "$DEST" "$SOURCE" 2>/dev/null

if [ $? -eq 0 ]; then
    echo "[+] Sauvegarde réussie dans : $DEST"
else
    echo "[-] Erreur critique lors de la création de l'archive !" >&2
    exit 1
fi
```

---

## 🔑 5. Administration à Distance Sécurisée : SSH

L'accès distant à un serveur Linux s'effectue exclusivement via le protocole chiffré **SSH** (port TCP 22).

### Authentification par clé asymétrique (Recommandé) :
1. Générer une paire de clés Ed25519 sur le poste client :
   ```bash
   ssh-keygen -t ed25519 -C "admin@domNN.ad"
   ```
2. Déployer la clé publique sur le serveur distant :
   ```bash
   ssh-copy-id -i ~/.ssh/id_ed25519.pub utilisateur@192.168.21.10
   ```
3. Connexion instantanée et sécurisée sans mot de passe :
   ```bash
   ssh utilisateur@192.168.21.10
   ```
