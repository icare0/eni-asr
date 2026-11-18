# 3. Sécurité des SI, Durcissement & PKI (AD CS)

La compromission d'un seul contrôleur de domaine entraîne la perte totale du système d'information. En 2ème année, l'administrateur met en œuvre des mécanismes de défense en profondeur pour empêcher les attaquants de progresser sur le réseau.

---

## 🏛️ 1. Le Modèle d'Administration en Couches (Tiering Model)

Le modèle de hiérarchisation administrative (*Tiering Model*) préconisé par l'ANSSI et Microsoft repose sur une règle absolue : **les privilèges de haut niveau ne doivent jamais être exposés sur des machines moins sécurisées**.

```
┌────────────────────────────────────────────────────────────────────────┐
│ TIER 0 : CŒUR DE SÉCURITÉ DE L'IDENTITÉ                                │
│ Contrôleurs de domaine (CD1, CD2), PKI racine, Comptes Domain Admins. │
│ ❌ Interdiction stricte de se connecter sur des postes clients ou serveurs T1/T2.│
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼────────────────────────────────────┐
│ TIER 1 : SERVEURS MEMBRES & APPLICATIONS                               │
│ Serveurs de fichiers (SRV1), SQL, Web, Windows Admin Center.           │
│ Administrés uniquement par des comptes de type 'Admin-SRV'.            │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼────────────────────────────────────┐
│ TIER 2 : POSTES DE TRAVAIL & PÉRIPHÉRIQUES UTILISATEURS                │
│ Postes clients Windows 11, PC portables, Imprimantes.                  │
│ Administrés par les comptes de support technique (ex: Ivan Tard).       │
└────────────────────────────────────────────────────────────────────────┘
```

> [!CAUTION]
> **Pourquoi cette séparation ?**  
> Si un administrateur du domaine ouvre une session sur un poste de travail infecté (Tier 2), ses empreintes d'authentification (tokens Kerberos / hashes NTLM) restent en mémoire dans le processus LSASS. Un pirate local peut alors les extraire (attaque *Pass-the-Hash*) et prendre le contrôle instantané de toute l'entreprise.

---

## 🔑 2. Neutraliser le Mouvement Latéral : LAPS

Sur un parc non sécurisé, tous les postes clients partagent souvent le même mot de passe pour le compte `Administrateur` local. La compromission d'un poste permet d'infecter l'ensemble du parc par rebond.

### La Solution : Windows LAPS
**Windows LAPS** (*Local Administrator Password Solution*) automatise la gestion des comptes locaux :
* Chaque poste génère un mot de passe local unique, complexe et aléatoire.
* Ce mot de passe est renouvelé automatiquement tous les 30 jours et chiffré dans l'Active Directory.
* Seuls les administrateurs autorisés peuvent lire le mot de passe du poste cible en cas de dépannage hors ligne.

---

## 📜 3. Infrastructure à Clés Publiques (PKI) avec AD CS

Le rôle **Active Directory Certificate Services (AD CS)** transforme votre Windows Server en autorité de certification d'entreprise intégrée à l'annuaire.

```
       [ Autorité Racine Hors Ligne (Root CA Offline) ]
                              │
                              ▼
        [ Autorité Subordonnée d'Entreprise (Issuing CA) ]
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
[ Certificats Web HTTPS ] [ Authentification 802.1X ] [ Chiffrement VPN IPsec ]
```

### Bénéfices opérationnels :
* **Déploiement automatique (Auto-enrollment)** : Les postes clients et serveurs membres reçoivent automatiquement leurs certificats SSL/TLS via GPO sans aucune manipulation manuelle.
* **Sécurisation des flux internes** : Chiffrement du trafic Web applicatif, validation des connexions Wi-Fi d'entreprise (WPA2/WPA3 Enterprise via RADIUS) et authentification par carte à puce.
