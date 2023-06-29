---
title: "OS command injection"
category: "Command Injection"
tag: "Web : Serveur"
---

## CheckList en pratique


## Explication de la vulnérabilité
Cette vulnérabilité web permet à un attaquant d'exécuter une commande de système d'exploitation (OS) sur le serveur sur lequel l'application est exécutée, et compromettre toute l'application et ses données. Souvent, cette attaque amène à une exploitation de l'infrastructure hôte en pivotant sur d'autres systèmes de l'organisation.

## Exécution de commandes arbitraires
Considérons un site de magasin qui vérifie si un article est en stock de la manière suivante :
`https://insecure-website.com/stockStatus?productID=381&storeID=29`
Pour fournir l'information et pour des raisons historiques, la fonctionnalité va appelé une commande shell avec le produit et son ID de la manière suivante :
`stockreport.pl 381 29`, et le retour est renvoyé à l'utilisateur.

Sans protection, l'attaquant peut injection la commande suivante `& echo toto &` ce qui donnera `stockreport.pl & echo toto & 29`

## Commandes utiles
|Purpose of command|Linux|Windows|
|:----|:----|:----|
|Nom de l'utilisateur courant|whoami|whoami|
|système d'exploitation|uname -a|ver|
|configuration réseau|ifconfig|ipconfig /all|
|connections réseau|netstat -an|netstat -an|
|processus courants|ps -ef|tasklist|

## Blind Injection de commandes OS
De nombreuses injections de commandes sont faites en aveugle, ce qui veut dire que l'application ne renvoie pas de sortie de la commande dans la réponse HTTP. On peut exploiter les vulnérabilités en aveugle, mais de différentes manières.

pour illustrer cela, on peut prendre le cas d'une application qui permet de donner un feedback sur un site en donnant son adresse mail et le message de feedback. L'application du côté serveur génère un email vers l'administrateur du site contenant le feedback. 

Pour cela, il peut utiliser un programme `mail` avec les détails, par exemple :
`mail -s "This site is great" -aFrom:peter@normal-user.net feedback@vulnerable-website.com`
### Détecter les Blind injections de commandes OS
On peut utiliser une injection de commande qui va ajouter un temps de délai, nous permettant de voir que la commande a bien été injecté.
On peut faire ça avec `ping` par exemple, pour allonger le temps de traitement  :
`& ping -c 10 127.0.0.1 &`
Cette commande va ping son adresse réseau de loopback pendant 10sec.

