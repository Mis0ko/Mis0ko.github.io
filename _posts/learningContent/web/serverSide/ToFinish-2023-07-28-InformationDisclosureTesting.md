---
title: "Information Disclosure Testing"
category: "Serveur"
tag: "Web"
---

# Comment trouver et exploiter les vulnérabilités liées à la divulgation d'informations

## CheckList en pratique

A FAIRE

## Comment tester les vulnérabilités en matière de divulgation d'informations
Des données sensibles peuvent être divulguées dans toute sorte d'endroits. La plupart du temps elles sont découvertes lors de test d'autres vulnérabilités, donc il est important de ne pas avoir une "vision étroite" lors des tests.
// https://mis0ko.github.io/serveur/InformationDisclosureTesting/

<u>Quelques outils qui peuvent être utiles :</u>
- Fuzzing
- Burp Scanner
- Outils d'engagement de Burp
- Réponses informatives en matière d'ingénierie

### Fuzzing 

En trouvant des paramètres intéressants, il peut être bien d'utiliser le Fuzzing, de sorte à soumettre de nombreux types de données pour provoquer un comportement inattendu.

- Ajouter des payloads aux paramètres et utiliser des listes de mots prédéfinies pour tester un grand nombre d'entrées différentes.
- Identifier facilement les différences dans les réponses en comparant les codes HTTP, les temps de réponse, les longueurs de réponses, etc.
- Utiliser les règles de correspondance grep pour identifier rapidement les occurrences de mots-clés, tels que `error`, `invalid`, `SELECT`, `SQL`, etc.
- Appliquer des règles d'extraction grep pour extraire et comparer le contenu d'éléments intéressants dans les réponses.

Extension burp à regarder : [Logger++](https://portswigger.net/bappstore/470b7057b86f41c396a97903377f3d81)

### Outils d'engagement de Burp

On peut trouver des informations intéressante sur un site de manière simple à partir des outils d'engagement de Burp.
Pour y accéder à partir du menu contextuel :  
&rarr;  Click droit sur n'importe quel message HTTP &rarr; Burp Proxy  &rarr; "Engagement tools"

#### Search

Outil pour rechercher une expression quelconque dans l'élément sélectionné. Diverses options de recherche avancée, telles que la recherche par expressions régulière ou la recherche négative, pour trouver l'occurence (ou non occurence) d'un élément.

#### Find comments

Outil pour extraire rapidement tous les commentaires de développeurs trouvés dans l'élément sélectionné.  
Il fournit également des onglets permettant d'accéder instantanément au cycle requête/réponse HTTP dans lequel chaque commentaire a été trouvé.

#### Découvrir le contenu

Outil pour identifier du contenu et des fonctionnalités supplémentaires qui ne sont pas liés au contenu visible du site web (par exemple répertoires et fichiers qui n'apparaîtront pas dans le plan du site).

#### Engineering informative responses

A voir si on s'en sert dans les labs car pas compris l'utilité.

## Sources communes de divulgation d'informations

### Fichiers pour les robots d'indexation
De nombreux sites fournissent les fichiers `/robots.txt` et `/sitemap.xml` pour aider les robots d'indexation (crawlers) à indexer leur site. Ces fichiers peuvent contenir des informations sensibles que les crawlers devraient passer.

### Listage de répertoires
Les serveur web peuvent être configurés pour automatiquement lister le contenu des dossiers n'ayant pas de page d'index. Cela permet à un attaquant d'identifier rapidement les ressources à certaines adresses du site.

Un site sensible au listage de répertoire ne possède pas forcément de vulnérabilité, mais s'il échoue à implémenter un accès de contrôle convenable, cela peut permettre une fuite de données potentiellement sensibles.


### Commentaires des développeurs 

Des commentaires oubliés par les développeurs seraient directement disponible dans le code sans être visible par l'affichage du site. Cela vaut le coup de regarder rapidement le code html.

### Messages d'erreur 

Les messages verbeux sont un bon indice pour savoir s'il est pertinent d'utiliser du temps sur une fonctionnalité d'un site web ou non.  
Ils peuvent indiquer des noms de technologies, moteurs de template, type de base de données, etc. Il est possible de vérifier également la version de ces outils si on les découvre avec des messages d'erreur disponible dans la documentation officielle.

### Données de débogage 

Pour des raisons de debug, des messages d'erreurs customisés et des logs sont parfois fournis par des sites web.
Ces informations peuvent contenir des informations vitales pour un attaquant telles que :
- Des valeurs de variables de clés de session qui peuvent être manipulées par l'entrée utilisateur.
- Des noms d'hôtes et credentials de composants côté serveur
- Des noms de fichiers et dossier du serveur
- Des clés utilisées pour chiffrées des données transmises au client.

Parfois, les informations de debogage sont stockés dans un fichier qui peut servir à comprendre le comportement d'un site en fonction des entrées fournies.

### Pages des comptes d'utilisateurs 

Les pages de profil des utilisateurs sont par natures remplies de données sensibles (adresse mail, num de téléphone, clés API, etc). Lorsqu'un utilisateur accède à sa propre page, pas de problèmes de sécurité de ce point de vue là.    

Cependant, certains sites ayant des **défauts de logique** peuvent permettre à un attaquant d'accéder à ces pages là facilement sans être forcément passé par les étapes requises.  

Par exemple, un site qui récupère une page de profil d'un compte en se basant sur un paramètre `user` d'une requête `GET`, tant que l'utilisateur courant est connecté avec une adresse mail serait une vulnérabilité.  
`GET /user/personal-info?user=carlos`

### Fichiers de sauvegarde 

Les fichiers de sauvegarde sont un bon moyen pour un attaquant de comprendre le comportement d'un site web et accéder à des données sensibles qui pourraient être hardcodés telles que les **API ekys** ou **credentials** pour accéder à des composants côté serveur.

Identifier une technologie opensource utilisée par un site peut permettre d'avoir un accès à une partie limité du code source.

Parfois, il est possible que le site expose son propre code source, qui est référencé explicitement.  
Malheureusement, requêter du code ne le révèle pas forcément. Quand un server web 
traite un fichier avec une extension particulière comme `.php`, il va exécuter le code plutôt que simplement l'envoyer au client comme du texte.

Dans certaines situations, il est possible de piéger le site web pour qu'il nous renvoie des contenus de fichier à la place. Par exemple, les éditeurs de texte génère souvent des backup de fichiers temporaire du fichier original qui est en train d'être édité.  
Ces fichiers sont souvent indiqués en ayant un `~` dans leurs noms de fichiers, ou en ajoutant une `différente extension au fichier`.
Requêter un fichier de code en utilisant l'extension de fichier de sauvegarde peut permettre d'y accéder.

### Configuration non sécurisée 

Les sites web sont parfois vulnérables à une mauvaise configuration. Cela est principalement dû à des composants tierces dont les options de configuration sont vastes et souvent mal comprises des personnes les implémentants.

Dans d'autres cas, les développeurs peuvent parfois oublier de désactiver les options de debug en environnement de production.

Par exemple, La méthode HTTP `TRACE` est conçue à des fins de diagnostic. Si elle est activée, le serveur web répondra aux requêtes qui utilisent la méthode TRACE en faisant écho dans sa réponse à la requête exacte qui a été reçue.

Ce comportement est souvent inoffensif, mais il conduit parfois à la divulgation d'informations sensibles telles que les en-têtes d'authentification internes ajoutés par les proxys inversés. 


### Historique du contrôle de version 


test