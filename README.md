![Licence GPL-3.0](https://img.shields.io/badge/Licence-GPL_3.0-red)


# Configuration-serveur-Ubuntu-20.04-Linux-Domain-Controller-Active-Directory
Configuration d’un serveur Ubuntu 20.04 Linux  en tant que  ' Domain Controller Active Directory Samba 4 '


 Samba est un logiciel gratuit à code source ouvert offrant une interopérabilité standard entre les systèmes d’exploitation Windows et Linux/Unix .

Samba peut fonctionner en tant que fichier et serveur d’impression autonome pour les clients Windows et Linux via la suite de protocoles SMB/CIFS
ou peut servir de contrôleur de domaine Active Directory ou être intégré à un < strong> le royaume en tant que membre du domaine .
Le niveau AD DC et le niveau de forêt les plus élevés que Samba4 peut actuellement émuler est Windows 2008 R2 .



## Pré-requis

  * Avoir des bases solides en Linux;
  * Un serveur Ubuntu 20.04;
  * Disposer des droits 'root'sur le serveur Linux;
  * Un ordinateur Windows 10.


## Configuration du nom d'hôte du serveur

Commencez en attribuant une adresse IP statique à votre serveur.
Ubuntu Server utilise « netplan » pour la gestion du réseau.
Votre configuration réseau ressemblera à ceci :

### ~$ sudo nano /etc/netplan/00-installer-config.yaml

![r](https://user-images.githubusercontent.com/46109209/179738782-df28e974-1b7f-471b-918d-3a5bce111845.png)

N.B. Ne pas utiliser de tabulations dans « netplan » mais des espaces !


Appliquer la configuration avec la commande :
### ~$ sudo netplan apply


Vérifiez si l'heure de votre serveur synchronise avec un serveur Internet.
### ~$ sudo timedatectl


Procédez à la mise à jour du système, en vous plaçant dans le terminal et frappez : 
### ~$ sudo apt –y update && apt –y upgrade


Modifiez le nom d'hôte (hostname), mettez à jour le fichier « hosts » et identifiez dans « resolv.conf » le serveur DNS à utiliser pour résoudre le nom de domaine
### ~$ sudo nano /etc/hostname
![image](https://user-images.githubusercontent.com/46109209/179642569-1e63f582-9758-4383-a605-75e538c4ff4a.png)

### ~$ sudo nano /etc/hosts
![image](https://user-images.githubusercontent.com/46109209/179642970-48900663-aa5e-441c-8d78-a2a75feb7986.png)

 
### ~$ sudo nano /etc/resolv.conf
![image](https://user-images.githubusercontent.com/46109209/179642997-5615dd45-c704-4ae2-b6d1-192eb9fc494b.png)


À présent, il faut activer l’ « IP forwarding ». Le « forwarding » est la capacité pour un système d'exploitation d'accepter les paquets réseau entrants sur une interface, de reconnaître qu'ils ne sont pas destinés au système lui-même, mais qu'ils doivent être transmis à un autre réseau, puis de les transférer en conséquence. 
### ~$ sudo nano /etc/sysctl.conf
![image](https://user-images.githubusercontent.com/46109209/179643061-02ab35c0-8519-41c3-945d-f97eb7390367.png)
 

Téléchargez et installez ensuite le paquet « netfilter-pesistent », « iptables-persistent » et « ipables » avec les commandes suivantes :
### ~$ sudo apt-get install netfilter-persistent
### ~$ sudo apt-get install iptables-persistent
### ~$ sudo apt-get install iptables

![image](https://user-images.githubusercontent.com/46109209/179643292-6f26abb2-ed40-4c93-9c43-6d77237692aa.png)

![image](https://user-images.githubusercontent.com/46109209/179643339-edfc2859-f461-4bfc-bbb9-542eecb08bbb.png)


Activez « netfilter-persistent » avec :
### ~$ sudo service netfilter-persistent start


Vous pourrez à présent établir vos règles de routage. Voici la règle routage NAT que nous mettons en place sur les interfaces « enp0s3 » avec la commande suivante :
### ~$ sudo iptables –t nat –A POSTROUTING –o enp0s3 –j MASQUERADE

![image](https://user-images.githubusercontent.com/46109209/179643583-77709aba-da6e-4020-8ed6-fbd749e78e7c.png)


Enregistrez la règle avec :
### ~$ sudo iptables-save


La règle NAT est maintenant enregistrée de manière temporaire. Rendez-la permanente avec la commande : 
### ~$ sudo iptables-save > /etc/iptables/rules.v4


On pourra constater la présence de la règle de NAT avec la commande :
### ~$ sudo iptables –t nat –L –n -v
 

Effectuez un redémarrage du système pour prendre en compte les changements effectués.
### ~$ sudo reboot


Après le redémarrage, installez les packages « SAMBA 4 Active Directory » :
### ~$ sudo apt -y install samba krb5-config winbind smbclient
---
Kerberos Realm: it.pro

Kerberos servers for your realm: ad1.it.pro

Administrative server for your Kerberos realm: ad1.it.pro

![image](https://user-images.githubusercontent.com/46109209/179643938-5b4ea2ae-a351-4d25-ad50-743f97fd1733.png)
![image](https://user-images.githubusercontent.com/46109209/179643989-cfc4667f-27bf-438a-85ce-6655f89b9f87.png)
![image](https://user-images.githubusercontent.com/46109209/179644018-de38fd60-8901-4f01-8e66-866804f168ae.png)


Procédez à l’édition du fichier « smb.conf ». Il est conseillé de faire une sauvegarde du fichier « smb.conf ».
### ~$ sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.backup


Lancement du « Domain Controller »
### ~$ sudo samba-tool domain provision
---
Realm [IT.PRO]:                                                                                        "A"

Domain [IT]:		                                                                                         "B"

Server Role (dc, member, standalone) [dc]:		                                                           "C"

DNS backend (SAMBA_INTERNAL, BIND9_FLATFILE, BIND9_DLZ, NONE) [SAMBA_INTERNAL]:	                       "D"

DNS forwarder IP address (write 'none' to disable forwarding)  [127.0.0.53]:	8.8.8.8

Administrator password: Entrer le mot de passe « administrateur » du « Domain Controller »

Retype password: Confirmer le mot de passe « administrateur » du « Domain Controller »
 
Du point ‘A à D’, faites « Enter » pour passer au point suivant.
Bien qu’ayant opté pour le DNS du serveur [192.168.250.17], il est conseillé de choisir le DNS de Google [8.8.8.8] car lorsqu'un utilisateur demande un nom de domaine mais que le serveur DNS de l'utilisateur ne peut pas trouver l'adresse IP correspondante dans son cache DNS ou dans ses zones d'autorité, le DNS de Google pourra le résoudre.

![image](https://user-images.githubusercontent.com/46109209/179644719-2894ace2-4339-44a0-8073-2f5eacd045f7.png)


Autorisez « SAMBA » à travers votre pare-feu :
### ~$ sudo apt-get –y install ufw
### ~$ sudo ufw allow ‘Samba’
 
![image](https://user-images.githubusercontent.com/46109209/179644775-e4d4e43c-c1cc-4677-b9b2-cf067729c5f1.png)


Copiez le fichier de configuration Kerberos :
### ~$ sudo cp /var/lib/samba/private/krb5.conf /etc/


Exécutez la commande suivante pour arrêter et désactiver les services dont le serveur Samba Active Directory n'a pas besoin smbd, nmbd et winbind. Le serveur n'a besoin que de samba-ac-dc pour servir d'Active Directory et de contrôleur de domaine :
### ~$ sudo systemctl disable --now smbd nmbd winbind systemd-resolved
### ~$ sudo systemctl enable --now samba-ad-dc


La commande suivante permet de vérifier le niveau AD DC et le niveau de forêt du serveur « SAMBA » :
### ~$ sudo samba-tool domain level show

![image](https://user-images.githubusercontent.com/46109209/179645341-85df2d1b-3bdd-4e46-8f52-f426011c6329.png)

« Active Directory » est maintenant prêt et fonctionnel! 


Cependant, avant de pouvoir joindre un PC Windows 10 à votre domaine AD, il faut créer des utilisateurs de l’AD.

Utiliser la commande samba-tool pour administrer « Active Directory » à partir du serveur. Créez l’utilisateur ‘ jadam ’

### ~$ sudo samba-tool user create jadam
![image](https://user-images.githubusercontent.com/46109209/179756015-d5ee731f-a960-41fa-8a60-6f1030874d34.png)


Joignez un ordinateur au domaine et connectez-vous à l'utilisateur du domaine

Configurez comme suit l'interface réseau de votre PC Windows
![image](https://user-images.githubusercontent.com/46109209/179756143-b2b5c865-5c75-4bf9-a35e-482f2912cc5c.png)
![image](https://user-images.githubusercontent.com/46109209/179645537-c5790667-a72b-424d-abea-e6db51772f0e.png)
![image](https://user-images.githubusercontent.com/46109209/179756451-6351f2a3-c03d-42f9-86ab-86fa3d9314e2.png)

![image](https://user-images.githubusercontent.com/46109209/179756574-3df64aff-9fa1-404f-ac78-c4aafa5e0c61.png)
![image](https://user-images.githubusercontent.com/46109209/179756601-656464a3-49d3-44c0-a307-44b7d51d506c.png)
