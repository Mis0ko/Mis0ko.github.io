---
title:  "XPath Injection"
date:   2022-10-23
category: article
---
Cette article a pour but de décrire le principe d'injection XPath et d'en énumérer une variété de manière non exaustive. 
## Prérequis
Avant de décrire les injections XPath il est nécessaire d'introduire quelques connaissances.
Tout d'abord, les injections XPath s'utilisent sur le langage XML. 
XML est un langage pour décrire des données de façon rapide et facile.

En utilisant XML, nous avons besoin d'un mécanisme pour nous permettre d'utiliser ce genre de données.
Ce mécanisme est le langage XPath (XML path langage). Il permet de sélectionner des informations du document par référence à n'importe quel type d'élément du document.
Une des opérations du XPath est le chemin de localisation. On peut prendre l'exemple si dessous:
```
<?xml version=”1.0”?>
<database>
<user>
<username>John</username>
<password>Doeuf</password>
<email>omelette.champignon@mail.com</email>
</user>
<user>
<username>testi</username>
<password>cule</password>
<email>jean.luc@mail.com</email>
</user>
</database>
```
Un chemin de localisation aura la forme de /user/username.
Ce chemin sélectionne tous les éléments de type username, étant enfant de tout élément de type user.

| Requête XPath   | résultat       |
| :--------------- |:---------------| 
| /database  |    Le noeud root database est sélectionné    | 
| //user   | Tous les noeuds 'user' sont sélectionnés            |   
| /database/user[user='John'] | Le noeud 'user', enfant de database et ayant pour user 'John' est sélectionné.          |  
| //user[email='omelette.champignon@mail.com'] | Tous les noeuds de type user contenant comme adresse mail 'omelette.champignon@mail.com' sont sélectionnés, indépendamment de leur position dans l'arbre| 
| /database/child::node()  |    Tous les noeuds enfants du neoud database    | 
| //user[position()=2]  |    Selectionne le noeud à la position indiquée   | 

# XPath Injection
Les attaques par XPath injection ressemble beaucoup à celle d'injection SQL car elles sont
basés sur le même principe : un non-échappement des caractères spéciaux.

## Bind XPath Injection
Pour ce type d'injection, il est bien de tester des ' ou " dans les champs d'entrée, on obtient dans le meilleur des
cas un message d'erreur comme :
- **Warning: SimpleXMLElement::xpath(): Invalid predicate in /challenge/[..]/index.php**
- **Warning: SimpleXMLElement::xpath(): xmlXPathEval: evaluation failed in /[..]/index.php on line 56**

L'avantage contrairement aux injections sql, c'est qu'il n'y a pas de contrôle d'accès.

Par exemple, une requête XPath pour une authentification pourrait ressembler à ceci : 
```
xpath("/accounts/user[username=' " . $_POST["username"] . " ' and password=' " . $_POST["password"] . " ' ]");
``` 
Le problème est que contrairement aux injections sql, on ne peut pas choisir de mettre en commentaire tout le reste de la requête.

Nous allons devoir faire en sorte que la condition soit vraie. L'exemple classique à injecter est  
**' or 1=1 or ''='**, ce qui donnera : 
```
xpath("/accounts/user[username='' or 1=1 or ''='' and password='blabla' ]");
```
Le fait d'avoir deux conditions vraies assure que la condition générale le soit également.
Cela renverra le premier élément du document xml.

## XPath Injection : String extraction
On a aussi le cas ou la sortie contient des chaînes de caractères et l'utilisateur peut manipuler les valeurs à rechercher à l'aide de la fonction "contains".
```
/user/username[contains(., '+VALUE+')]
```

On peut trouver ce genre d'erreur toujours avec des caractères spéciaux comme \' et avoir un message d'erreur:
- **Error during search, invalid XPath syntax : //user/username[contains(., ''')]**

On peut utiliser diverses payload comme **')] | //node()[('')=('** ou **') or 1=1 or ('**.  

(voir références pour une liste plus complète).


## Références
- https://repository.root-me.org/Exploitation%20-%20Web/EN%20-%20Blind%20Xpath%20injection.pdf?_gl=1*1qfi55x*_ga*MTgyMzg3NjQ3My4xNjY2MTIwNjM2*_ga_SRYSKX09J7*MTY2NjI5OTcwMS42LjEuMTY2NjI5OTk1OS4wLjAuMA
- https://book.hacktricks.xyz/pentesting-web/xpath-injection
- https://www.scip.ch/en/?labs.20180802

