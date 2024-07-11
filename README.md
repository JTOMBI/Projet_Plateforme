# Projet_Plateforme
Projet Firewall


Installation du machine virtuelle (pour ma part sur VirtualBox) 
Lors de l'installation de Debian 12 quand vous serait à "sélection des logiciel" cocher serveur web et ssh

![Screenshot_4](https://github.com/JTOMBI/Projet_Plateforme/assets/171478594/4592c1b1-cc9c-4f4e-b9cc-6f6f2911d44e)

sinon installer le manuellement a l'aide du terminal

https://friendhosting.net/fr/blog/install-apache-on-debian-11.php
https://www.it-connect.fr/chapitres/installation-dun-serveur-ssh-et-premiere-connexion/

ensuite tester si votre site ou le ssh fonctionne
pour le site web aller dans votre navigateur et ecriver 127.0.0.1 pour trouver votre site
et pour le ssh ecriver sur un terminal = apt-cache policy openssh-server et regarder si il est actif et tester le ssh sur un autre ordi avec par exemple putty

une fois Debian installé aller a votre terminal taper ces commande pour supprimer les regle du firwall

![Screenshot_3](https://github.com/JTOMBI/Projet_Plateforme/assets/171478594/e02a4b09-d44d-4886-bef2-3b8be6544812)

et ensuite vérifier que la commande  iptables -nvL ne renvoi rien

![Screenshot_5](https://github.com/JTOMBI/Projet_Plateforme/assets/171478594/37c4c97b-7aee-43b1-839c-633f86387ac5)

Maintenant, nous allons recréez chacune des règles nécessaires pour que le serveur fonctionne (connexion ssh et serveur web).

![Screenshot_6](https://github.com/JTOMBI/Projet_Plateforme/assets/171478594/03fc45eb-0938-47ea-a370-f56fc66826d8)

nous autorisont les entrées / sortie des ports 80 - 8443 - 443 pour le serveur web

![Screenshot_7](https://github.com/JTOMBI/Projet_Plateforme/assets/171478594/bad7064d-84e8-43a9-b073-cfbe31c51d5c)

nous autorisont les entrées / sortie du port 22 pour le serveur ssh

et on verifie avec la commande iptables -nvL

![Screenshot_9](https://github.com/JTOMBI/Projet_Plateforme/assets/171478594/026b5ca9-3f1f-4ccf-8fd3-0ec43b0e92bd)

puis on verifie si le site web et les connexions ssh fonctionne

pour vérifier nos log on va installer logwatch
Logwatch est un système configurable d'analyse de fichiers journaux ( log ) distribué sous licence MIT. Il va parcourir et analyser les fichiers journaux, et envoyer un rapport par courriel

taper 'apt-get install logwatch'
modifier les lignes dans le fichier /usr/share/logwatch/default.conf/logwatch.conf

LogDir = /var/log
MailTo = votre_adresse_email@fai.com
Detail = High
Service = All
mailer = "/usr/sbin/sendmail -t" (si vous utiliser postfix)
Ouput = mail

modifier les lignes de postfix dans le fichier /etc/postfix/main.cf

mydestination = votre_adresse_email@fai.com, localhost, localhost.localdomain
mydomain = votre_adresse_email@fai.com
myhostname = votre_adresse_email@fai.com
mynetworks = 127.0.0.0/8 111.222.111.222

pour tester l'envoie de mail taper logwatch --range=Today --output mail et normalement vous recever les logs dans votre boite mail

Maintenant on va s'occuper de fail2ban

fail2ban est une application qui analyse les logs de divers services (SSH, Apache, FTP ...) en cherchant des correspondances entre des motifs définis dans ses filtres et les entrées des logs. Lorsqu'une correspondance est trouvée une ou plusieurs actions sont exécutées

tout d'abord installé fail2ban en executant apt-get install fail2ban
ensuite dans le fichier /etc/fail2ban/fail2ban.conf ajouter les modifications suivante apres "Definition"

![Screenshot_11](https://github.com/JTOMBI/Projet_Plateforme/assets/171478594/722725fa-355f-4afb-ab93-bf6558e8cc5e)

puis dans /etc/fail2ban/jail.conf modifier dans la partie ssh

![Screenshot_12](https://github.com/JTOMBI/Projet_Plateforme/assets/171478594/d632f9d5-3884-46aa-9bd1-ebb33ab17fc7)

et modifier la partie ftp

![Screenshot_15](https://github.com/JTOMBI/Projet_Plateforme/assets/171478594/c9180f0c-b5be-4c85-bb43-a2da04e659b8)

on teste la connexion ssh en essayent de se connecter 5 fois 
pour rerifier les log taper la commande 'cat /var/log/auth.log

![Screenshot_14](https://github.com/JTOMBI/Projet_Plateforme/assets/171478594/601373d4-cf21-43a9-ba0e-24865ae85491)

et regerdé avec la commande 'fail2ban-client status sshd' si une ip a été ban

![Screenshot_13](https://github.com/JTOMBI/Projet_Plateforme/assets/171478594/498da798-e49f-4441-92b6-5bc3643c5cca)

pour la connexion ftp il nous faut un serveur ftp

https://www.it-connect.fr/debian-9-configurer-un-serveur-ftp-avec-proftpd/
n'oublier pas d'ajouter les regles iptable pour le ftp

une fois installer on teste le serveur avec plus de 10 requete ip
et 20 tentative de connexion avec une autre machine et normalement il fait comme avec le ssh a un moment il vont ban le personne qui essaye de brute force

