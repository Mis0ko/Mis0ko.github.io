---
title: "Directory/file path traversal"
category: "Directory traversal"
tag: "Web : Serveur"
---
Cette vulnérabilité web permet à un attaquant de lire des fichiers arbitraires sur le serveur qui exécute l'application. Cela inclut du code, des données de l'application, credentials, fichiers sensibles pour l'OS, etc. Dans certains cas, l'attaquant peut écrire des fichiers sur le serveur, ce qui lui permet de modifier des données / comportement de l'application et donc prendre son contrôle.

## Lecture arbitraire de fichiers par Directory traversal
Un cas classique est celui d'images uploadés par du HTML de la manière suivante :
```html
<img src="/loadImage?filename=218.png">
```
Si les images du serveurs sont stockées dans ***/var/www/images***, l'URL ***https://insecure-website.com/loadImage?filename=218.png*** accédera au fichier suivante :
```console
/var/www/images/218.png
```

L'attaque de base consiste à utiliser la séquence ../ ou ..\ (en fonction de l'OS) pour remonter l'arborescence.

```
https://insecure-website.com/loadImage?filename=../../../etc/passwd
```
