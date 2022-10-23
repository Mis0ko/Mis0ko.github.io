---
title:  "Injection SQL"
date:   2022-05-07
category: article
---
Cette article a pour but de décrire le principe d'injection SQL et d'en énumérer une variété de manière non exaustive. 
# Injection SQL
Une attaque par injection sql consiste à insérer une requête SQL à travers une donnée d'entrée à partir de l'application client. Cela a pour conséquence de pouvoir accéder ou altérer la base de données ciblée.  
Si on considère la requête sql suivante nous permettant d'obtenir les lignes de la table admin dont les champs login et password sont respectivement égaux à $login et $password:
```
SELECT * FROM admins WHERE login='$login' AND password='$password'
```
Un exemple de requête sql serait d'insérer à la place de $login :
```
' OR 1=1 --
```
avec - - indiquant de mettre en commentaire ce qui se trouve à droite, on obtient la requête suivante :
```
SELECT * FROM admins WHERE login='' OR 1=1 --$login' AND password='$password'
```
La requête ci-dessus va donc récupérer les lignes de la base de données admins qui ont soit un pseudo nul, soit tel que '1=1', c'est à dire toujours.
Nous obtiendrons donc toutes les lignes de la table admins.  

Une autre possibilité aurait été de mettre comme pseudo :
```
' or 1=1 AND username='admin'--
```
dans une condition WHERE sur la valeur d'un pseudo, permettant ainsi de sélectionner les lignes de l'admin,
utile pour une connection par exemple.

Il est important de notifier que la mise en commentaire dépend du serveur et que donc on peut retrouver
\- - comme # ou /*, etc.

# Comment s'en prémunir
Afin d'éviter d'être soumis aux injections SQL, le principe de base est "d'échapper" des caractères, principalement
les quotes et doubles quotes. Cela veut dire que l'on va annuler l'interprétation du caractère en question pour qu'il soit simplement afficher.
On peut prendre l'exemple de l'échappement de la quote dans printf en C :
```
printf('je m'échappe'); // provoquera une erreur car la chaine de caractère se termine après le premier m 
```
En C, on échappe des caractères avec le caractère d'échappement '\' :

```
printf('je m\'échappe'); # affichera "je m'échappe" 
```
Ici on a échappé la quote entre le m et le é.
Le but est de faire de même avec les quotes et doubles quotes des conditions de chaines de caractères dans les WHERE.
Il existe d'autres méthodes de préventions comme le typage des données d'entrées mais nous n'entrerons pas dans le détail ici.

# Autres Injections SQL 
## Injection SQL - GBK
Cette Injection sql très connue se base sur l'interférence des charsets.
Avant de parler d'aller plus loin il est nécessaire d'expliquer quelques notions. 

### Notion d'encodage
**Un charset est une transcription avec représentation numérique des caractères des langues naturelles.**  
De manière plus simple, les caractères sont regroupés dans un registre de caractères, appelé ensuite "registre de caractères codés" lorsqu’un chiffre précis est attribué à chaque caractère, nommé "point de code". 

L'encodage est ce qui structure les points de code en octets dans la mémoire de l'ordinateur, puis lits les octets à nouveau en points de code.  

Pour faire simple, on passe des caractères à des chiffres d'une certaine manière en fonction du type d'encodage.
Quand on parle de charset, on parle donc d'encodage.

### Explication Injection SQL - GBK
Quand un serveur s'attend à recevoir une requête sql dans un type d'encodage/charset (unicode par exemple),
il va donc traduire les valeurs d'octets en caractères.

Par exemple 0x5c devient \ dans la majeure partie des charset.

Souvent, afin d'éviter des injections sql, nous échappons les caractères spéciaux afin qu'ils ne soient pas 
interprétés par le serveur sql.
Nous en arrivons à un exemple de charset qui est le jeu de caractères GBK (jeu de caractères chinois).

Les caractères spéciaux de GBK commencent par 0xBF, puis forment différents caractères selon les valeurs des bytes suivants. On peut citer notamment 0xBF5C qui est un caractère chinois, or 0x5c représente l'antislash "\\" dans la plupart des charset occidentaux.

Ainsi, si on imagine une requête SQL du type :
```
WHERE login='  0xbf'  ' AND BLABLA
```
et que **0xbf'** est nettoyée pour se prémunir des injections sql, typiquement avec la fonction [addslashes](https://www.php.net/manual/en/function.addslashes.php), on obtiendra **0xbf\\'** .
Si le serveur Mysql s'attend à recevoir du GBK, il traitera **0xbf\'** (**0xbf0x5c0x27**) comme un caractère chinois
(car **0xbf0x5c0x27** n'est pas valide en chinois, et devient **0xbf5c0x27**).

Ainsi, la requête initialement demandée
```
WHERE login='  0xbf'  ' AND BLABLA
```
deviendra, une fois décodée par le charset (et donc interprété par le serveur Mysql) :
```
WHERE login='  [caractère chinois]'  ' AND BLABLA
```
Au lieu d'avoir la quote après bf échappée par un antislash.

Si on reprend l'exemple classique de l'injection sql **' OR 1=1 #**, il faudra le changer en **0xbf' OR 1=1#**.

## Références
- https://www.php.net/manual/en/function.addslashes.php
- https://www.bases-hacking.org/injections-sql-avancees.html
- https://www.w3.org/International/questions/qa-what-is-encoding.fr

