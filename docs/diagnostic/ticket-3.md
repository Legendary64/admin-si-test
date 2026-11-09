# Diagnostic et Résolution - Ticket #3

## 1. Informations générales
- **Détection** : Alerte Prometheus `HostDown` / `SSHServiceDown`, heure de la panne : 14:15, heure du constat : 14:25.
- **Serveur concerné** : `srv-web01` (`10.10.0.20`)
- **Service impacté** : `ssh.service`

## 2. Symptôme constaté
- Impossible d'établir une nouvelle connexion SSH vers le serveur (`ssh sysadmin@10.10.0.20`).
- La tentative de connexion renvoie une erreur de type `Connection refused`.

## 3. Diagnostic des journaux et analyse
depuis la console locale du serveur / session ouverte :

1. **Statut du service** :
   `systemctl status ssh` indique un état `failed` (code d'erreur au démarrage).

2. **Journaux du service** :
   `journalctl -u ssh -b -n 20` confirme l'arrêt du démon SSH suite à une erreur dans la configuration.

3. **Validation de la configuration** :
   La commande `sudo sshd -t` met en évidence une erreur de syntaxe dans le fichier `/etc/ssh/sshd_config.d/50-redhat.conf` (ou nom du fichier récent) concernant la directive `PermitRootLogin`.
   
   *Cause racine* : La valeur renseignée pour `PermitRootLogin` était invalide (ex: mauvaise syntaxe ou faute de frappe), empêchant le service SSH de lire le fichier de configuration au démarrage.

## 4. Action corrective (Durcissement SSH maintenu)
L'intention initiale était de sécuriser l'accès `root` au serveur. Le durcissement a été conservé en appliquant une valeur conforme aux recommandations de sécurité OpenSSH :

1. Modification du fichier de configuration dans `/etc/ssh/sshd_config.d/` :
   - Remplacement de la valeur erronée par `PermitRootLogin prohibit-password` (ou `no`).

2. Vérification de la configuration :
   `sudo sshd -t` (aucun retour d'erreur).

3. Redémarrage du service :
   `sudo systemctl restart ssh`

## 5. Validation et clôture
- Vérification du statut : `systemctl status ssh` indique `active (running)`.
- Validation de l'accès : La connexion SSH `sysadmin@10.10.0.20` fonctionne à nouveau.
- Résolution Prometheus : L'alerte s'est automatiquement acquittée sur le tableau de bord de supervision.
