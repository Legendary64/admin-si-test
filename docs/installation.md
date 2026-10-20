# Procédure d'Installation et Configuration du SI - PME.LAN

Ce document détaille l'ensemble des étapes techniques, commandes exécutées, choix de configuration et résolutions d'incidents appliqués lors du déploiement des serveurs **SRV-DC01** et **SRV-WEB01**.

---

## Jour 1 — Déploiement du Contrôleur de Domaine et Installation de Debian

### Étape 0 : Configuration du réseau Hyperviseur (VMware)
- Désactivation du serveur DHCP local sur la carte NAT de VMware Workstation (`VMnet8`).
- Définition du sous-réseau `10.10.0.0/24` et de la passerelle `10.10.0.2`.

---

### Étape 1 & 2 : Configuration initiale de SRV-DC01 (Windows Server 2025)
1. Installation de Windows Server 2025 Standard (Desktop Experience).
2. Attribution de l'IP fixe et renommage du serveur via PowerShell :

```powershell
Get-NetAdapter
Set-NetIPInterface -InterfaceAlias "Ethernet0" -Dhcp Disabled
Remove-NetIPAddress -InterfaceAlias "Ethernet0" -Confirm:$false -ErrorAction SilentlyContinue
Remove-NetRoute -InterfaceAlias "Ethernet0" -DestinationPrefix 0.0.0.0/0 -Confirm:$false -ErrorAction SilentlyContinue
New-NetIPAddress -InterfaceAlias "Ethernet0" -IPAddress 10.10.0.10 -PrefixLength 24 -DefaultGateway 10.10.0.2
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 127.0.0.1,8.8.8.8
Rename-Computer -NewName SRV-DC01 -Restart

# Étape 3 : Promotion DC & DNS
Install-WindowsFeature AD-Domain-Services, DNS -IncludeManagementTools
Install-ADDSForest -DomainName "pme.lan" -DomainNetbiosName "PME" -InstallDns -SafeModeAdministratorPassword (Read-Host -AsSecureString "Mot de passe DSRM") -Force

# (Après redémarrage du DC) : Configuration des redirecteurs DNS
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 127.0.0.1
Add-DnsServerForwarder -IPAddress 8.8.8.8

# Étape 4 : Installation et configuration du DHCP
Install-WindowsFeature DHCP -IncludeManagementTools
Add-DhcpServerInDC -DnsName SRV-DC01.pme.lan -IPAddress 10.10.0.10
Add-DhcpServerv4Scope -Name "LAN-PME" -StartRange 10.10.0.100 -EndRange 10.10.0.200 -SubnetMask 255.255.255.0 -State Active
Set-DhcpServerv4OptionValue -ScopeId 10.10.0.0 -Router 10.10.0.2 -DnsServer 10.10.0.10 -DnsDomain pme.lan

# Mise à jour de l'enregistrement DNS A pour srv-web01
Get-DnsServerResourceRecord -ZoneName pme.lan -Name srv-web01 | Remove-DnsServerResourceRecord -ZoneName pme.lan -Force
Add-DnsServerResourceRecordA -ZoneName pme.lan -Name srv-web01 -IPv4Address 10.10.0.20

# Étape 6 : Configuration réseau statique (à adapter si nécessaire)
printf "search pme.lan\nnameserver 10.10.0.10\n" | sudo tee /etc/resolv.conf

# Étape 7 : Synchro NTP, Paquets et Jonction au domaine
sudo mkdir -p /etc/systemd/timesyncd.conf.d
printf "[Time]\nNTP=srv-dc01.pme.lan\n" | sudo tee /etc/systemd/timesyncd.conf.d/pme.conf
sudo systemctl restart systemd-timesyncd

sudo apt update
sudo apt install -y realmd sssd sssd-tools adcli libnss-sss libpam-sss krb5-user samba-common-bin packagekit winbind
sudo realm join -U Administrateur --computer-ou="OU=Serveurs,OU=PME,DC=pme,DC=lan" pme.lan

# Création des dossiers perso (mkhomedir) et droits sudo pour le groupe AD
sudo pam-auth-update --enable mkhomedir
echo '%gg_admins_linux@pme.lan ALL=(ALL:ALL) ALL' | sudo tee /etc/sudoers.d/admins-linux
sudo chmod 440 /etc/sudoers.d/admins-linux

# Étape 8 : Durcissement SSH
sudo mkdir -p /etc/ssh/sshd_config.d
printf "PermitRootLogin no\nPubkeyAuthentication yes\nPasswordAuthentication yes\n" | sudo tee /etc/ssh/sshd_config.d/durcissement.conf
sudo sshd -t && sudo systemctl reload ssh

# Étape 9 : Nginx et Pare-feu UFW
sudo apt install -y nginx ufw curl
echo "<h1>SRV-WEB01 - pme.lan</h1>" | sudo tee /var/www/html/index.html

sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx HTTP'
sudo ufw enable

