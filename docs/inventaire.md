# Inventaire de l'Infrastructure Système - Projet PME.LAN

Ce document dresse l'inventaire complet des équipements, logiciels, comptes et ressources déployés dans le cadre de l'infrastructure fil rouge de l'entreprise `pme.lan`.

---

## 1. Informations Générales du Domaine

| Paramètre | Valeur |
| :--- | :--- |
| **Nom de domaine FQDN** | `pme.lan` |
| **Nom NetBIOS** | `PME` |
| **Plage réseau global** | `10.10.0.0/24` |
| **Masque de sous-réseau** | `255.255.255.0` |
| **Passerelle par défaut** | `10.10.0.2` (VMware Workstation) |
| **Serveur DNS principal** | `10.10.0.10` (SRV-DC01) |
| **Plage DHCP** | `10.10.0.100` à `10.10.0.200` (distribuée par SRV-DC01) |

---

## 2. Inventaire des Serveurs

| Nom de Hôte | Système d'Exploitation | Adresse IP | Rôles / Services | Ressources (RAM / vCPU / Disk) |
| :--- | :--- | :--- | :--- | :--- |
| **SRV-DC01** | Windows Server 2025 Standard (Desktop Exp.) | `10.10.0.10` (Fixe) | AD DS, DNS, DHCP | 4 Go RAM, 2 vCPU, 40 Go |
| **SRV-WEB01** | Debian 13 (Installation minimale) | `10.10.0.20` (Fixe) | Serveur Web, membre du domaine AD | 1 Go RAM, 1 vCPU, 20 Go |
| **SRV-ADM01** | Debian 13 | `10.10.0.30` | *(Prévu - Ansible, Supervision, Logs)* | À venir |

---

## 3. Annuaire Active Directory (OU, Groupes, Comptes)

### Structure des Unités d'Organisation (OU)
* `OU=PME,DC=pme,DC=lan`
  * `OU=Utilisateurs`
  * `OU=Groupes`
  * `OU=Serveurs`

### Groupes de Sécurité (Dans `OU=Groupes,OU=PME,DC=pme,DC=lan`)
* **`GG_Admins_Linux`** : Groupe des administrateurs ayant les privilèges `sudo` sur les serveurs Linux.
* **`GG_Utilisateurs`** : Groupe général des utilisateurs du domaine.

### Comptes Utilisateurs
| Identifiant (sAMAccountName) | Nom Complet | Rôle / Droits Linux | Membre des groupes |
| :--- | :--- | :--- | :--- |
| **a.martin** | Alice Martin | Administratrice Linux (`sudo` autorisé) | `GG_Admins_Linux`, `GG_Utilisateurs` |
| **b.durand** | Bernard Durand | Utilisateur standard (Accès restreint) | `GG_Utilisateurs` |

---

## 4. Services et Configurations Sécurité (SRV-WEB01)

* **SSH (OpenSSH)** : Port 22, connexion par clé publique activée (`PubkeyAuthentication yes`), connexion directe `root` interdite (`PermitRootLogin no`).
* **Serveur Web (Nginx)** : Port 80 HTTP, page d'accueil personnalisée `/var/www/html/index.html`.
* **Pare-feu (UFW)** : Actif. Politique par défaut : `deny incoming`, `allow outgoing`. Règles autorisées : Ports 22 (`OpenSSH`) et 80 (`Nginx HTTP`).
