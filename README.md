# Projet de synthèse : industrialiser le SI de la PME

Binôme : <Prénom Nom> et <Prénom Nom>
Dépôt : https://github.com/Legendary64/admin-si-test

## Le projet
La configuration des serveurs Linux est décrite en Ansible, celle de Windows
en PowerShell, et vérifiée par un pipeline GitHub Actions (ansible-lint et
--syntax-check à chaque push).

## Schéma du SI
| Serveur   | IP         | Système | Rôle                                        |
|-----------|------------|---------|---------------------------------------------|
| SRV-DC01  | 10.10.0.10 | Windows | AD DS, DNS, DHCP, partage Commun            |
| SRV-WEB01 | 10.10.0.20 | Debian  | nginx, sauvegarde, intranet                 |
| SRV-ADM01 | 10.10.0.30 | Debian  | Ansible, apt-cacher-ng, Prometheus, Grafana |

## Contenu du dépôt
- `ansible/` : site.yml, maj.yml, exporteurs.yml, proxy-apt.yml, inventaire, rôles commun, ssh, durcissement
- `windows/` : scripts PowerShell rejouables
- `supervision/` : configuration Prometheus (prometheus.yml, pme.yml)
- `docs/` : installation, inventaire, supervision, exploitation, procédures, sécurité (Lynis), journaux, diagnostic
- `.github/workflows/ci.yml` : ansible-lint et --syntax-check
- `ETAT.md` : état honnête du projet

## Rejouer la configuration, dans l'ordre
1. Prérequis : SRV-ADM01 avec Ansible, clé SSH de sysadmin déployée sur ansible@10.10.0.20, sudo et python3-apt présents sur SRV-WEB01
2. `git clone https://github.com/Legendary64/admin-si-test.git` puis `cd admin-si-test/ansible`
3. `ansible all -m ping`
4. `ansible-playbook site.yml -K --check --diff`
5. `ansible-playbook site.yml -K`
6. Rejouer : `changed=0`
7. Windows : exécuter les scripts de `windows/` sur SRV-DC01 dans l'ordre : <liste>

## Rôles Ansible
- `commun` : paquets de base, fuseau horaire, compte audit
- `ssh` : sshd durci (PermitRootLogin no, MaxAuthTries 3), validé par sshd -t
- `durcissement` : ufw (refus par défaut, ports dans vars.yml), fail2ban, mises à jour automatiques de sécurité

## Compte audit
Le rôle commun crée le compte `audit` (clé publique de l'intervenant, sudo sans
mot de passe). La clé est dans `ansible/group_vars/all/vars.yml`.
Le compte a `password: "*"` et `password_lock: false` : sans mot de passe
utilisable, mais non verrouillé, sinon sshd refuse la connexion par clé.

## Sécurité : Lynis avant et après
- Avant durcissement : <indice> (`docs/securite/lynis-avant.txt`)
- Après durcissement : <indice> (`docs/securite/lynis-apres.txt`)

## Secrets
Aucun secret en clair. <Si Vault : fichier chiffré, mot de passe hors dépôt.>

##Écarts constatés lors de l'audit
SRV-WEB01 absent de l'AD
Le serveur Web a été joint au domaine sous le nom « EWAN ». Le script d'audit attend la présence d'un ordinateur nommé « SRV-WEB01 », ce qui explique l'écart signalé.

GPO de mises à jour non détectée
Une GPO dédiée aux mises à jour est configurée sous le nom « GPO_MAJ_Windows ». Le script d'audit ne semble pas prendre en compte cette dénomination et ne l'identifie donc pas comme une GPO de gestion des mises à jour.

## Sources
<Code ou documentation repris d'ailleurs, avec le lien. Écrire "aucune" si rien.>
