# État du projet

Dernière mise à jour : 29/10/2026

## Ce qui marche
- SRV-ADM01 : Ansible installé, `ansible all -m ping` répond sur SRV-ADM01 et SRV-WEB01
- Authentification SSH par clé de sysadmin vers `ansible@10.10.0.20`
- Inventaire `inventaire/hosts.ini` et `ansible.cfg` en place
- Playbook `maj.yml` : syntaxe valide (`--syntax-check`)
- Playbook `proxy-apt.yml` : cache apt configuré
- apt-cacher-ng installé, actif, écoute sur le port 3142
- Pare-feu ufw actif (ports 22, 3142, 514 autorisés)
- Collecte des journaux : rsyslog reçoit les logs de SRV-WEB01 sur le port 514/tcp, rotation avec logrotate
- Dépôt Git du projet initialisé

## Ce qui ne marche pas
- Heure non synchronisée sur SRV-ADM01 (NTP vers srv-dc01.pme.lan)
- Le script de test du TP 3 affiche encore 2 échecs (`ansible all -m ping` : 0/0 machine, `maj.yml : syntaxe valide`) alors que les mêmes commandes lancées à la main réussissent : probablement un défaut du script
- Rôles `ssh` et `durcissement` : pas encore écrits
- Pipeline GitHub Actions : pas encore en place

## Ce qui reste fait à la main
- Installation de SRV-ADM01 et des paquets (ansible, git, apt-cacher-ng, rsyslog)
- Création de la clé SSH de sysadmin et `ssh-copy-id` vers SRV-WEB01
- Configuration rsyslog (réception et envoi) et logrotate
- Règles ufw
- GPO de mise à jour Windows (bloquée par un disque plein sur SRV-DC01 lors du test)

## Incidents et leçons
- Faute de frappe `port"514"` (signe `=` oublié) dans deux fichiers rsyslog : détectée avec `rsyslogd -N1`
- Ansible lancé en root : la clé SSH de root n'était pas déployée sur SRV-WEB01 (Permission denied). Corrigé en travaillant avec sysadmin
- Projet copié dans `/root` alors que le script de test le cherchait sous `/home` : déplacé dans `/home/sysadmin/admin-si-test`
- `sudo` absent de SRV-ADM01 (Debian minimale) : installé en root
- `python3-apt` absent de SRV-WEB01 : bloquait `--check`
- `git push` refusé avec le mot de passe : GitHub exige un jeton d'accès personnel
- Droits `0664` sur `/etc/logrotate.d/distants` : logrotate ignorait le fichier, corrigé en `0644`
- `New-GPO` en échec (`0x80070070`, disque plein) sur SRV-DC01

## Points ouverts
- Écrire les rôles `commun` (avec le compte audit), `ssh`, `durcissement` et `site.yml`
- Mettre en place le pipeline CI
- Prouver la sauvegarde et la restauration, supervision, Lynis avant/après
- Rédiger le README et la procédure de restauration
