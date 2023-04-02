---
title:  "Découverte d'XSS"
category: "Notions de base sur les XSS"
tag: "Cross-Site Scripting (XSS)"
---
Maintenant que nous avons vu les différents types d'XSS, nous allons pouvoir voir ici la manière de les détecter. Cette partie peut être aussi chronophage que l'exploitation.

# Découverte automatisée

Tous les Scanners de vulnérabilité d'application webs (comme [Nesses](https://www.tenable.com/products/nessus), [Burp Pro](https://portswigger.net/burp/pro), [ZAP](https://owasp.org/www-project-zap/)) permettent de détectés les 3 types de XSS avec plus ou moins de précisions.
Ces outils font généralement deux types de scans :
- une analyse passive, qui examine le code côté client à la recherche de vulnérabilités potentielles de types DOM.
- un scan actif, qui envoie différents payloads dans le but de provoquer une injection XSS.

Les scans actifs fonctionnent sur le principe d'envoyer une payload et de comparer le code / rendu visuel affiché dans le but de voir si le payload est contenu dans le code affiché.\
Bien que ces outils permettent de faire une première détection, il est important de vérifier manuellement si la vulnérabilité est avérée car l'exécution n'est pas forcément concluante pour diverses raisons.

Parmi les outils opensource pour assister aux découvertes de XSS, il existe [XSSStrike](https://github.com/s0md3v/XSStrike), [BruteXSS](https://github.com/rajeshmajumdar/BruteXSS) et [xsser](https://github.com/epsylon/xsser).

On peut utiliser **XSS Strike** :
```console
Misoko@htb[/htb]$ git clone https://github.com/s0md3v/XSStrike.git
Misoko@htb[/htb]$ cd XSStrike
Misoko@htb[/htb]$ pip install -r requirements.txt
Misoko@htb[/htb]$ python xsstrike.py                
Misoko@htb[/htb]$ python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test" 
```


# Découverte Manuelle
## XSS Payloads

La méthode la plus classique pour découvrir des vulnérabilités XSS est de tester divers payloads dans les champs d'entrées d'un site web.
On peut trouver de nombreuses listes de XSS en ligne comme celles de [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/README.md) ou [PayloadBox](https://github.com/payloadbox/xss-payload-list).


> Pour information:  
> Les XSS peuvent être injectés dans n'importe quelles entrées de la page HTML, ce qui n'est pas exclusif aux champs de saisies HTML. On peut également faire des XSS sur les en-têtes HTTP comme les Cookies ou les User-Agent ( quand leurs valeurs sont affichées dans la page).

On peut constater que la majorité des paylaods fournis dans les exemples précédents ne permettent pas de faire une XSS simple car ils sont spécifiques pour contourner des **échappements**.

C'est pourquoi il n'est pas très efficace de copier/coller manuellement des payloads XSS, car même si une application web est vulnérable, cela peut prendre un certain temps pour identifier la vulnérabilité, surtout si nous avons de nombreux champs d'entrée à tester.

Il peut être plus efficace d'écrire notre propre script Python pour automatiser l'envoi de ces payloads et de comparer ensuite la source de la page pour voir comment nos payloads ont été rendues.


## Revue de code

La revue de code manuelle est  la méthode la plus efficace poour détecter une vulnérabilité XSS. Si nous comprenons précisément comment la donnée est gérée à la fois du côté back-end et front-end, nous avons une grande chance d'élaborer un payload personnalisé qui devrait fonctionner.