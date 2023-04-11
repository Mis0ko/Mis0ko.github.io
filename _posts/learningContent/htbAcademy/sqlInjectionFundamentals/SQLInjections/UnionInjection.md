---
title:  "injection Union"
category: "Injections SQL"
tag: "Principes de base de l'injection SQL"
---

Dans cette section, plutôt que d'utiliser le mot clé **OR** ou les commentaires, nous allons voir comme injecter une requête SQL intégralement avec la clause SQL **UNION**.

# Union

Il est important de comprendre comment fonctionne la clause **UNION** avant de pouvoir l'utiliser pour faire des injections.

La clause **UNION** permet de combiner des résultats de la part de plusieurs instructions **SELECT**. Elle permet donc de pouvoir dump les données à travers tout un SGBD, de la part de toutes ses tables et bases de données.\
La syntaxe est la suivante :

```sql
mysql> SELECT * FROM ports UNION SELECT * FROM ships;
```

### Colonnes égales

L'instruction **UNION** peut s'opérer sur les instructions **SELECT** ayant un nombre égale de colonnes.
Si deux **SELECT** ont un nombre de colonnes différents, une erreur sera renvoyée.

Il est donc nécessaire de savoir combien de colonnes la requête de base va renvoyée afin de pouvoir faire un union avec une requête injectée.
Par exemple, si on suppose que la requête suviante renvoie 2 colonnes.

```sql
SELECT * FROM products WHERE product_id = 'user_input'
```

Alors on peut faire une injection de la manière suivante.
```sql
SELECT * from products where product_id = '1' UNION SELECT username, password from passwords-- '
```

### Colonnes inégales

La plupart du temps, nous allons nous retrouver à devoir adapter le nombre de colonnes de notre requête avec celle que nous voulons exécuter.
Pour cela, nous allons devoir insérer des **données inutiles** pour ajouter des colonnes, comme une string **"junk"** qui sera toujours renvoyer peu importe la requête. On peut également utiliser un chiffre.

> Il est important de se rappeler que le type de données des colonnes doit correspondre à celui de la colonne afin de ne pas provoquer d'erreur.

Par exemple, si la table **products** a 4 colonnes, la première étant de type string et les autres de type entier, on peut faire une union de la manière suivante :

```sql
mysql> SELECT * from products where product_id UNION SELECT username, 2, 3, 4 from passwords-- '

+-----------+-----------+-----------+-----------+
| product_1 | product_2 | product_3 | product_4 |
+-----------+-----------+-----------+-----------+
|   admin   |    2      |    3      |    4      |
+-----------+-----------+-----------+-----------+
```

# Injection UNION

## Detection du nombre de colonnes

Avant d'exploiter une requête avec un **UNION**, il faut déterminer le nombre de colonnes.
Pour cela il existe deux méthodes :
- En utilisant **ORDER BY**
- En utilisant **UNION**

### Utilisation de ORDER BY

Le principe est d'utiliser l'instruction **ORDER BY** afin de classer la requête en fonction du numéro de la colonne. On utilisera une injection du type :

```sql
' order by 1-- -
```

Ensuite, on itère (order by 2, 3, ...) jusqu'à obtenir une erreur. Si on obtient une erreur en faisant un order by à la colonne **n**, cela voudra dire que l'on a **n-1** colonnes.

Le principe de cette méthode est d'obtenir toujours un résultat positif jusqu'à atteindre une erreur.

### Utilisation de UNION

Contrairement à la méthode précédente, on va faire des tentatives de requêtes avec **UNION** jusqu'à qu'on obtienne un résultat positif.
On aura donc toujours une erreur sur nos tentatives jusqu'à obtenir un résultat.

On comment par exemple avec 2 colonnes, et on continue d'ajouter des colonnes jusqu'à obtenir un résultat positif.
```sql
cn' UNION select 1,2,3,4-- -
```

<center><img src="/assets/images/htbAcademy/SQLInjectionFundamentals/UNIONInjection1.png" alt="Alt text"></center>

### Localisation de l'injection

Bien que la requête SQL renvoie plusieurs colonnes, il est probable qu'un sous ensemble soit affichée sur la page du site web. On a pu voir cela dans la dernière requête **UNION** de la méthode précédente où étaient affichées seulement les colonnes 2,3 et 4.

On va donc utiliser des chiffres pour localiser facilement les colonnes impactées par l'injection **UNION** et ensuite récupérer les informations qui nous intéressent sur les colonnes précédemment identifiées.

Un exemple pour voir si on arrive à obtenir une information du serveur, serait de requêter la version de la base de données (**version**) comme test.

```sql
cn' UNION select 1,version,3,4-- -
```

