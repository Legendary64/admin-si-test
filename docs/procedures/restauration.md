# Restauration d'un fichier

## Ce qui est sauvegardé
- Serveur : SRV-WEB01
- Source : <dossiers sauvegardés, ex. /home, /var/www>
- Destination : <dossier ou disque de sauvegarde>
- Outil : <rsync, restic, borg, tar...>
- Planification : <cron ou timer, jour et heure>

## Vérifier que la sauvegarde existe
<commande pour lister les sauvegardes disponibles>

## Restaurer un fichier
1. Identifier la sauvegarde contenant la bonne version : <commande>
2. Restaurer le fichier : <commande avec chemins réels>
3. Vérifier le contenu et les droits : `ls -l <fichier>`

## Test de restauration réalisé
- Fichier supprimé : <lynis-avant>,
- Commande de restauration : <commande>
- Résultat : <lynis-apres>
