---
title: "Directory/file path traversal"
category: "Directory traversal"
tag: "Web : Serveur"
---

## CheckList en pratique
- cas classique :  `../../../etc/passwd`
- chemin relatif à un répertoire de travail par défault : `/etc/passwd`
- App bloque ../ : `..../..../..../etc/passwd`
- Url encode : `..%2F..%2F..%2Fetc%2Fpasswd`
- Double Url encode : `..%252F..%252F..%252Fetc%252Fpasswd`
- validation du début du chemin : `/var/www/images/../../../etc/passwd`
- Validation de l'extension de fichier avec l'octet nul : `../../../etc/passwd%00.jpg`

## Explication de la vulnérabilité
Cette vulnérabilité web permet à un attaquant de lire des fichiers arbitraires sur le serveur qui exécute l'application. Cela inclut du code, des données de l'application, credentials, fichiers sensibles pour l'OS, etc. Dans certains cas, l'attaquant peut écrire des fichiers sur le serveur, ce qui lui permet de modifier des données / comportement de l'application et donc prendre son contrôle.

## Lecture arbitraire de fichiers par Directory traversal
Un cas classique est celui d'images uploadées par du HTML de la manière suivante :
```html
<img src="/loadImage?filename=218.png">
```
Si les images du serveurs sont stockées dans ***/var/www/images***, l'URL ***https://insecure-website.com/loadImage?filename=218.png*** accédera au fichier suivante :
```console
/var/www/images/218.png
```

L'attaque de base consiste à utiliser la séquence ../ ou ..\ (windows) pour remonter l'arborescence.

```
https://insecure-website.com/loadImage?filename=../../../etc/passwd
https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini
```
## Obstacles communs pour exploiter le Directory traversal
Les applications peuvent supprimer ou bloquer les séquances de traversées (../) de répertoire à partir du nom du fichier.

- Pour cela, il est possible d'utiliser un chemin absolu à partir de la racine du système de fichier  `/etc/passwd`.
</br>
- Il est également possible d'utiliser des séquences de traversées imbriquées, telles que `....//` ou `....\/`, qui redeviendront des séquences de traversée simples lorsque la séquence interne sera supprimée.
</br>
- Dans certains contextes, comme le chemin d'accès à une URL ou le paramètre 'un  nom de fichier d'une requête `multipart/form-data`, les serveurs web peuvent supprimer toutes les séquences de traversée de répertoire avant de transmettre votre entrée à l'application. Il est possible de contourner ce mécanisme en encodant les caractères `../` dans l'URL, ou en les encodant deux fois, ce qui donne respectivement `%2e%2e%2f` et `%252e%252e%252f`. 
Il existe d'autres encodages non-standars qui peuvent passer, comme `..%c0%af` ou `..%ef%bc%8f`
</br>
- Si une application exige que le nom de fichier fourni par l'utilisateur commence par le dossier de base prévu (`/var/www/images/` par exemple), on peut fournir un chemin commençant par cette base suivi d'une séquence de traversée:
`filename=/var/www/images/../../../etc/passwd`
</br>
- Si l'application exige que le nom de fichier fourni se termine apr une extension attendue, telle que `.png`, Alors il est possible d'utiliser un octet nul pour terminer le chemin du fichier avant l'extension requise
`filename=../../../etc/passwd%00.png`


## Comment s'en prémunir
Le meilleur moyen de s'en prévenir est d'éviter de passer des entrées utilisateur dans une API de système de fichiers.

S'il est nécessaire de fournir une entrée utilisateur à une API de système de fichiers, alors :
- L'application doit vérifier l'entrée utilisateur avant de la traiter. Idéalement, la validation vérifie l'input parmi une liste blanche de valeurs. Sinon, que l'entrée contaigne seulement des éléments permi, tel que des alphanumériques.
- Après la validation de l'entrée utilisateur, l'application doit s'ajouter à la suite d'un dossier de base, et utilise une API du système de fichiers pour canoniser le chemin d'accès. La forme canonisée doit commencer par le dossier de base.

