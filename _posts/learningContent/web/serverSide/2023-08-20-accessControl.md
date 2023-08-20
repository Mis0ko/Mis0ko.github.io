---
title: "7.Vulnérabilités du contrôle d'accès et élévation des privilèges"
category: "Serveur"
tag: "Web"
---
## CheckList en pratique


## Explication de la vulnérabilité

Le contrôle d'accès (ou d'autorisation) est l'application de contraintes sur qui (ou quoi) peut effectuer des tentatives d'actions ou accéder aux ressources qu'ils ont demandées. Dans le cadre des applications web, le contrôle d'accès est dépendant de **l'authentification** et de la **gestion des sessions** :

- `L'authentification` identifie l'utilisateur et confirme qu'il est bien celui qu'il prétend être.
- `La gestion de session` identifie les requêtes HTTP suivantes effectuées par ce même utilisateur.
- `Le contrôle d'accès` détermine si l'utilisateur est autorisé à effectuer l'action qu'il tente d'effectuer.

Les **"broken access control"** sont une vulnérabilité de sécurité courante et souvent critique.
On peut définir les contrôles d'accès selon 3 catégories.
### Contrôle d'accès verticaux 
Les contrôles d'accès verticaux sont des mécanismes qui restreignent l'accès à des fonctionnalités sensibles qui ne sont pas disponibles pour d'autres types d'utilisateurs.  

Différents types d'utilisateurs ont différents types de fonctionnalités.
Par exemple un administrateur peut modifier ou supprimer des comptes d'utilisateur, là où un utilisateur classique n'aurait pas d'accès à ces fonctionnalités.

### Contrôle d'accès horizontaux
Les contrôles d'accès horizontaux sont des mécanismes qui limitent l'accès aux ressources des utilisateurs spécifiquement autorisés à accéder à ces ressources.

Avec les contrôles d'accès horizontaux, différents utilisateurs ont accès à un sous-ensemble de ressources du même type. 
Par exemple, un client d'une banque peut visualiser le contenu de son compte courant mais pas celui du voisin.


### Contrôles d'accès dépendant du contexte

Les contrôles d'accès dépendant du contexte restreignent l'accès aux fonctionnalités et aux ressources en fonction de l'état de l'application ou de l'interaction de l'utilisateur avec celle-ci.

Les contrôles d'accès dépendant du contexte empêchent un utilisateur d'effectuer des actions dans le mauvais ordre. 
Par exemple, un site Web de vente au détail peut empêcher les utilisateurs de modifier le contenu de leur panier après avoir effectué le paiement.

# Exemples de "Broken access control"

Des vulnérabilités de "broken access control" existent lorsqu'un utilisateur peut  accéder à une ressource ou effectuer une action à laquelle il n'est pas censé pouvoir accéder.

## Élévation verticale des privilèges
Si un utilisateur peut accéder à une fonctionnalité à laquelle il n'est pas autorisé à accéder, il s'agit d'une élévation verticale des privilèges.

### Fonctionnalité non protégée
L'élévation verticale des privilèges survient lorsqu'une application n'applique aucune protection sur les fonctionnalités sensibles. Par exemple, une page administrative liée à une page de bienvenue pour les utilisateurs admins, mais qui est disponible à tout utilisateur ayant le lien à cette fonctionnalités.

On retrouve deux exemples classiques :
- `https://insecure-website.com/admin`
- `https://insecure-website.com/robots.txt`

Parfois, l'URL de ma page d'administration peut être divulguée à d'autres emplacements, tels que le fichier `robots.txt`.
D'autres localisations peuvent être sensibles, et révélées par l'attaquant à l'aide d'outils d'énumération comme dirbuster.

  
Une technique de protection utilisée par des développeurs (à tort), est d'utiliser un lien non conventionnel pour la page d'administration, comme par exemple :
` https://insecure-website.com/administrator-panel-yb556`
En tant qu'attaquant, il peut être intéressant de regarder les scripts JS pour voir s'il n'y a pas de traces de ces pages (par exemple, un affichage en fonction des droits de l'utilisateur).

### Méthodes de contrôle d'accès basées sur des paramètres

Certaines applications déterminent les droits d'accès lors de la connexion, puis stockent ces informations dans un emplacement contrôlable par l'utilisateur, tel qu'un champ masqué, un cookie ou un paramètre de chaîne de requête prédéfini. 

L'application prend des décisions de contrôle d'accès ultérieures en fonction de la valeur soumise.
par exemple :
- `https://insecure-website.com/login/home.jsp?admin=true`
- `https://insecure-website.com/login/home.jsp?role=1`

Cette approche est non sécurisée car un utilisateur peut simplement modifier la valeur et accéder à des fonctionnalités auxquelles il n'est pas autorisé.

### Contrôle d'accès brisé résultant d'une mauvaise configuration de la plate-forme
### Contrôle d'accès brisé résultant de divergences de correspondance d'URL



## Élévation horizontale des privilèges



## Élévation des privilèges horizontale à verticale


## Insecure direct object references (IDOR)


## Vulnérabilités de contrôle d'accès dans les processus en plusieurs étapes



## Contrôle d'accès basé sur les référents


## Location-based access control

