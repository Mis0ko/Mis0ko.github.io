---
title:  "Subvertir la logique des requêtes"
category: "Injections SQL"
tag: "Principes de base de l'injection SQL"
---
# SQLI Injection
Pour découvrir des injections SQL, une bonne façon de faire est de tester les caractères suivants dans les champs cibles.

| Payload | URL Encodé |
|---------|-------------|
| '       | %27         |
| "       | %22         |
| #       | %23         |
| ;       | %3B         |
| )       | %29         |

***Dans certains cas, il peut s'avérer nécessaire d'utiliser la version codée de l'URL du payload. C'est le cas, par exemple, lorsque nous plaçons notre charge utile directement dans l'URL, c'est-à-dire dans une requête HTTP GET.***

Considérons une requête classique qui prend les valeurs de deux champs d'un formulaire d'authentification : 
```sql
SELECT * FROM logins WHERE username='admin' AND password = 'p@ssw0rd';
```

Le fait de passer une quote dans un champ (le username ici) va transformer la requête en la suivante :

```sql
SELECT * FROM logins WHERE username=''' AND password = 'something';
```

L'insertion de la quote implique qu'il y a maintenant un nombre impair de quotes, provoquant une erreur.

# Injection OR

Pour contourner l'authentification, il faut que la requête renvoie toujours **true**, quels que soient le nom d'utilisateur et le mot de passe saisis. Pour ce faire, nous pouvons abuser de l'opérateur **OR** dans notre injection SQL.

L'opérateur **AND** est évalué avant l'opérateur **OR**. Cela signifie que s'il y a au moins une condition **TRUE** dans la requête entière avec un opérateur **OR**, la requête entière sera évaluée à **TRUE** puisque l'opérateur **OR** renvoie **TRUE** si l'un de ses opérandes est **true**.

L'instruction **'1'='1'** est un exemple fonctionnel. Cependant, pour conserver un nombre pair de quotes lorsqu'on injecte l'exemple dans la requête, on va enlever une quote, et on va plutôt utiliser des exemples comme le suivant (pour le username dans notre cas )
```sql
admin' or '1'='1
```
Ce qui donne une requête finale de :
```sql
SELECT * FROM logins WHERE username='admin' or '1'='1' AND password = 'something';
```

Ce qui signifie :
- Si le nom d'utilisateur est admin
OU
- Si 1=1 renvoie true (c'est toujours le cas)
ET
- Si le mot de passe est 'something'

<center><img src="/assets/images/htbAcademy/SQLInjectionFundamentals/ORSQLInjection.png" alt="Alt text"></center>

L'opérateur **AND** sera évalué en premier et renverra un résultat **False**. Ensuite, l'opérateur **OR** sera évalué, et si l'une des affirmations est vraie, il renverra **true**. Puisque 1=1 renvoie toujours vrai, cette requête renverra vrai et nous permettra d'accéder au site.

> Il est à noter que ce payload est un parmi beaucoup d'autres payloads pour contourner l'authentification, que l'on peut retrouver notamment dans [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass).


### Contourner sans connaissance du nom d'utilisateur

Finalement, même si l'on ne connait pas le nom d'utilisateur, dans notre cas nous pouvons simplement ajouter une condition OR de manière similaire dans le mot de passe ce qui provoquera une condition vrai dans le **WHERE**.

En se basant sur le même procédé avec également le mot de passe, on obtient la requête suivante :

<center><img src="/assets/images/htbAcademy/SQLInjectionFundamentals/ORSQLInjection2.png" alt="Alt text"></center>

Cette requête mène à l'authentification.



