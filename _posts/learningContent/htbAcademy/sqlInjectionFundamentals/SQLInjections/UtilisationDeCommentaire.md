---
title:  "Utilisation de commentaire"
category: "Injections SQL"
tag: "Principes de base de l'injection SQL"
---

# Commentaires

Comme tout langage, le SQL permet l'utilisation de commentaires. On retrouve **-- # et /\*\*/**.
On utilisera principalement -- et # pour les injections sql.

> En SQL, utiliser deux tirets n'est pas suffisant pour mettre en commentaire, il est nécessaire d'avoir un espace après. On utilisera donc (-- )
> De plus, cette combinaison est souvent encodé dans l'URL en **(--+)**, comme les espaces sont encodées en (+). Pour être clair, on utilisera (-- -).

Si on utilise le payload dans l'URL du navigateur, le (#) est souvent utilisé comme tag, et ne devrait pas passé dans l'URL.

Pour utilisé le signe (#) comme commentaire, on peut utiliser sa forme URL encodée avec **'%23'**.