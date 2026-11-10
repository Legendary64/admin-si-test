# État du SI au <date et heure>

## SRV-DC01 · 10.10.0.10 · Windows Server
- Rôles : AD DS (pme.lan), DNS, DHCP, FSRM
- Partage Commun avec quota, GPO_MAJ_Windows, windows_exporter
## SRV-WEB01 · 10.10.0.20 · Debian 13
- Joint au domaine, nginx, SSH par clé, ufw, fail2ban
- vg_data (/var, /home), RAID 1, sauvegarde planifiée, comptes ansible et audit
## SRV-ADM01 · 10.10.0.30 · Debian 13
- Ansible, apt-cacher-ng (3142), rsyslog (514), Prometheus, Grafana, compte audit

## Ce qui marche
- Playbook site.yml (commun, ssh, durcissement) : rejeu à changed=0 sur les deux serveurs Linux
- Compte audit avec la clé de l'intervenant et sudo sans mot de passe
- Pipeline GitHub Actions (ansible-lint et --syntax-check) : <vert ou rouge, à vérifier>
- Lynis : indice <avant> avant durcissement, <après> après
- <supervision : cibles Prometheus UP, alerte testée, à confirmer>
## Ce qui ne marche pas
- <à compléter honnêtement, ex. sssd-ssh.service en échec sur SRV-WEB01>
## Ce qui reste fait à la main
- Installation des serveurs, jonction au domaine, Prometheus, Grafana
- <autres, ex. exécution des scripts PowerShell sur SRV-DC01>
## Incidents et leçons
- Faute port"514" dans rsyslog, corrigée avec rsyslogd -N1
- Ansible lancé en root sans clé déployée, corrigé avec sysadmin
- sudo absent de SRV-ADM01 et python3-apt absent de SRV-WEB01
- Script test-tp3-adm01.sh : 2 échecs alors que les commandes manuelles passent
- Compte audit refusé en SSH par clé : sshd le rejetait car verrouillé (journalctl : "account is locked"), corrigé avec password: "*" et password_lock: false dans le rôle commun
- Push GitHub refusé : le jeton n'avait pas le droit workflow, corrigé en recréant le jeton
## Points ouverts
- <à compléter>
