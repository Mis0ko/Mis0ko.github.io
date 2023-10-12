---
title: "7.Vulnérabilités du contrôle d'accès et élévation des privilèges"
category: "Serveur"
tag: "Web"
---
## CheckList en pratique



- <u>Trouver la page d'administration du site en essayant :</u>
    - `https://insecure-website.com/admin`
    - `https://insecure-website.com/robots.txt`
    - dirbuster
    - de regarder les scripts JS du site
- Contrôle d'accès basées sur des paramètres
    - Être attentif aux URL laissant entendre qu'un contrôle d'accès est opéré, tels que `https://insecure-website.com/login/home.jsp?admin=true&role=1`  
    Penser aux :
        - Valeurs prédictibles (entier comme identifiant, ...)
        - Valeurs non-prédictibles utilisées de manière similaire (**GUIDs**, ...), récupérables ailleurs sur le site.
        - Redirect de ressources vers une page de connection 
        &rarr; regarder la réponse de la requête (body) qui peut contenir des informations sensitives.

    - Regarder les paramètres QS/POST ainsi que la réponse de la requête qui peut donner un indice sur la façon dont sont utilisés ces paramètres.
- Broken access control résultant d'une mauvaise configuration de la plate-forme.   
Des applications limitent des méthodes HTTP à des utilisateurs sur des liens précis comme `DENY: POST, /admin/deleteUser, managers`
    - Utiliser `X-Original-URL` ou `X-Rewrite-URL` pour remplacer l'URL de la requête d'origine
    - Changer le type de méthode HTTP :   
    click droit &rarr; Change request method (ne pas oublier le cookie de session).
- Broken access control résultant de divergences de correspondance d'URL  
    - la casse des caractères du path
    - les extensions de fichiers (`/admin/deleteUser` et `/deleteUser.anything`)
    - différenciation de endpoint du `/`, entre `/admin/deleteUser/` et `/admin/deleteUser` par exemple.
- `IDOR` (accès à des ressources à partir d'une donnée cliente).  
- Vulnérabilités de contrôle d'accès dans les processus en plusieurs étapes  
&rarr; Tester si toutes les étapes vérifient le contrôle d'accès, en testant avec un user priviligié d'abord (ne pas oublier le cookie de session).
- Contrôle d'accès basé sur les référents   
&rarr; Fournir le bon URL dans le header `Referer` pour faire un broken access control.
- Contrôle d'accès basé sur la localisation (banque, multi-média, ...)   
&rarr; Utiliser un VPN.

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
Par exemple :
- `https://insecure-website.com/login/home.jsp?admin=true`
- `https://insecure-website.com/login/home.jsp?role=1`

Cette approche est non sécurisée car un utilisateur peut simplement modifier la valeur et accéder à des fonctionnalités auxquelles il n'est pas autorisé.

>Penser à regarder les paramètres QS/POST, et regarder la réponse de la requête qui peut donner un indice sur la façon dont sont utilisés ces paramètres!

### Broken access control résultant d'une mauvaise configuration de la plate-forme

Certaines applications appliquent des contrôles d'accès au niveau de la plate-forme en limitant l'accès à des URL et des méthodes HTTP spécifiques en fonction du rôle de l'utilisateur.  
Par exemple, une application peut configurer des règles telles que les suivantes :
`DENY: POST, /admin/deleteUser, managers`

Cette règle refuse l'accès à la méthode `POST` sur l'URL `/admin/deleteUser`, pour les utilisateurs du groupe des managers. 

Ce genre de comportement laisse la possibilité de faire des contournements du contrôle d’accès.

1. Certains frameworks d'application prennent en charge divers en-têtes HTTP non standard qui peuvent être utilisés pour remplacer l'URL dans la requête d'origine, tels que `X-Original-URL` et `X-Rewrite-URL`.

Si un site Web utilise des contrôles front-end rigoureux pour restreindre l'accès en fonction de l'URL, mais que l'application permet de remplacer l'URL via un en-tête de requête, il peut alors être possible de contourner les contrôles d'accès à l'aide d'une requête comme celle-ci :

```http
POST /?username=carlos HTTP/1.1
X-Original-URL: /admin/deleteUser
...
```

>Lorsqu'il est nécessaire d'utiliser des paramètres dans la queryString, penser à les ajouter dans la requête original et pas le header HTTP.

2. Une autre possibilité pour tenter de vérifier le contrôle d'accès est de changer la requête HTTP.
Si un attaquant peut utiliser la méthode GET (ou une autre) pour effectuer des actions sur une URL restreinte, il peut alors contourner le contrôle d'accès mis en œuvre au niveau de la plate-forme.

Dans Burp-Repeater, faire un click droit -> Change request method, et modifier le cookie de session pour mettre une session d'un utilisateur n'ayant pas les droits pour l'action qu'il demande.


### Broken access control résultant de divergences de correspondance d'URL
Lors du routage des demandes entrantes, les sites Web peuvent faire correspondre différents chemins à un point de terminaison défini.
Par exemple, /ADMIN/DELETEUSER et /admin/deleteUser peuvent être mappée au même point de terminaison.

Ce n'est pas un problème si le mécanisme de contrôle d'accès s'applique aux 2 paths.Cependant, on voit que certains site web n'applique pas les mêmes restrictions aux différents paths.

Quelques exemples de divergences de correspondance d'URL : 
- la casse des caractères du path
- les extensions de fichiers (`/admin/deleteUser` et `/deleteUser.anything`)
- différenciation de endpoint du `/`, entre `/admin/deleteUser/` et `/admin/deleteUser` par exemple.

## Élévation horizontale des privilèges

L'élévation horizontale des privilèges se produit lorsqu'un utilisateur est en mesure d'accéder aux ressources appartenant à un autre utilisateur (du même groupe de droits), au lieu de ses propres ressources de ce type.

Les méthodes d'exploit peuvent être similaire à celles de l'élévation verticale. 
On peut voir différents exemples.
1. Accéder à un compte utilisateur en changeant un chiffre dans le lien `https://insecure-website.com/myaccount?id=123`.

2. Valeur non-prédictible utilisée de manière similaire, tels que les **GUIDs** (globally unique identifiers) partout sur le site, donc récupérable sur ce dernier à d'autres endroits que celui où on s'en sert pour exploiter(article, ...).

3. Lorsqu'un accès à une ressource redirige vers une page de connection, regarder la réponse de la requête (body) qui peut contenir des informations sensitives.

## Élévation des privilèges horizontale à verticale
En utilisant une élévation des privilèges horizontale, en ayant accès à un autre compte utilisateur par exemple, il est possible de faire une élévation verticale.
Ceci est possible à l'aide du reset/capture de mot de passe d'un autre utilisateur, ou d'un accès à une page d'administration.

## Insecure direct object references (IDOR)
Un IDOR se produit lorsqu'une application utilise une entrée fournie par l'utilisateur pour accéder directement aux objets et qu'un attaquant peut modifier l'entrée pour obtenir un accès non autorisé.

## Vulnérabilités de contrôle d'accès dans les processus en plusieurs étapes

Ici, il est sujet de fonctionnalités se faisant en plusieurs étapes comme mettre à jour des informations sur un utilisateurs.   
Par exemple :
1. Charger le formulaire contenant les détails d'un utilisateur spécifique.
2. Soumettre des changements.
3. Vérifiez les modifications et confirmez.

Par exemple pour un procédé en 3 étapes, un développeur peut supposer qu'un contrôle d'accès est nécessaire seulement pour la première étape et ne pas faire de contrôle sur les étapes 2 et 3.
Un hackeur peut reproduire les étapes 2 et 3 sans vérification.
(penser à avoir le bon cookie de session lors du test).

## Contrôle d'accès basé sur les référents
Certains sites web base le contrôle d'accès sur le header `Referer` soumis lors de la requête HTTP.
Ce header est ajouté aux requêtes des navigateurs pour indiquer la page à partir de laquelle une requête a été lancée.

Par exemple, prenons un site renforce le contrôle d'accès sur la page princiape d'administration à `/admin`, mais pas aux sous-pages comme `/admin/deleteUser` qui inspectent seulement le header `Referer`

Si le header `Referer` contient l'URL principal avec `/admin`, alors la requête est autorisée.

Dans cette situation, fournir le bon URL dans le header `Referer` permet un **broken access control**.
## Contrôle d'accès basé sur la localisation

Certains sites Web appliquent des contrôles d'accès aux ressources en fonction de la situation géographique de l'utilisateur. 

Par exemple, les applications bancaires / services multimédias où s'appliquent la législation nationale ou des restrictions commerciales.

Ces contrôles d'accès peuvent souvent être contournés par l'utilisation de proxys Web, de VPN ou par la manipulation de mécanismes de géolocalisation côté client.


## Comment prévenir les vulnérabilités du contrôle d'accès

- Ne comptez jamais uniquement sur l’obscurcissement pour le contrôle d’accès.
- Sauf si une ressource est destinée à être accessible au public, refusez l’accès par défaut.
- Dans la mesure du possible, utilisez un mécanisme unique à l’échelle de l’application pour appliquer les contrôles d’accès.
- Au niveau du code, obligez les développeurs à déclarer l'accès autorisé pour chaque ressource et à refuser l'accès par défaut.
- Auditez et testez minutieusement les contrôles d’accès pour vous assurer qu’ils fonctionnent comme prévu.