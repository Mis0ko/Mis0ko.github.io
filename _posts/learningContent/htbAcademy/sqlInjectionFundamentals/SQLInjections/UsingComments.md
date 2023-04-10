---
title:  "Utilisation de commentaires"
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

# Contournement d'authentification avec des commentaires

Le principe est d'ajouter des commentaires dans les champs de saisies de sorte que la suite de la commande sql ne soit pas pris en compte. Cela peut être particulièrement utile lorsque la partie concernée est une condition pour s'authentifier.

Si on reprend l'exemple d'authentification classique, on peut insérer **admin'--** dans le username, ce qui annulera le reste de la requête qui exige le mot de passe.

```sql
SELECT * FROM logins WHERE username='admin'-- ' AND password = 'something';
```

