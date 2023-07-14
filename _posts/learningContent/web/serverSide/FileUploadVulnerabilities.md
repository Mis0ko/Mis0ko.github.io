---
title: "File Upload Vulnerabilities"
tag: "Web : Serveur"
---



## CheckList en pratique
- Lors d'une requête d'un fichier, si la réponse contient l'en tête `Content-Type` contient la correspondance entre l'extension du fichier et le type MIME, il n'a peut être pas été explicitement défini (donc changer une extension d'un exe peut le bypass).

## Explication de la vulnérabilité
Les vulnérabilités d'upload de fichiers consistent en la possibilité d'upload des fichiers dans le système de fichier du serveur web à l'origine de cette fonctionnalité. 

Si l'on n'applique pas correctement des restrictions suffisantes, même une simple fonction de téléchargement d'images peut être utilisée pour upload des fichiers arbitraires et potentiellement dangereux. Cela peut être des scripts côté serveur qui permettent de faire des RCE.

Dans certains cas, le fait de télécharger le fichier suffit à lui seul à causer des dommages. D'autres attaques peuvent impliquer une requête HTTP de suivi pour le fichier, généralement pour déclencher son exécution par le serveur.

## Quels sont les impacts des vulnérabilités d'upload de fichiers?

L'impact dépend généralement de 2 facteurs :
- Quel aspect du fichier le site web ne parvient pasà valider correctement ( taille, type, contenu, etc)
- Quelles restrictions sont imposés sur le fichier une fois qu'il a été upload.

1. Dans le pire des scénarios, **le type de fichier n'est pas valide** et la configuration de serveur permet à certain types de fichiers (`.php` ou `.jsp`) d'executer du code. Dans ce cas, l'attaquant peut upload un fichier de code côté serveur qui focntionne comme `webshell`, permettant d'avoir total accès au serveur web.

2. Si **le nom de fichier n'est pas vérifié**, cela permettrai à un attaquant d'écraser des fichiers techniques simplement en téléversant un fichier portant le même nom. Couplé au **directory transversal**, cela permettrait à l'attaquant d'upload des ficheirs dans des localisations non-prévues.

3. La non-vérification de la taille du fichier (en dessous d'un certain seuil prévu) peut permettre une attaque par **déni de service (DOS)**, où l'attaquant rempli l'espace disque.

## Raisons pour lesquelles la vulnérabilité existe
- Usage de blacklist avec oubli d'un type de fichier
- Le site qui tente de vérifier le type de fichier en regarant les propriétés, ce qui peut être manipulé par l'attaquant avec Burp
- Une validation robuste qui est appliquée de manière incohérente dans le réseau d'hôtes et de dossier qui forme le site.

## Comment les serveurs web traitent-ils les demandes de fichiers statiques ?

Historiquement, les sites web étaient presque entièrement constitués de fichiers statiques qui étaient présentés aux utilisateurs lorsqu'ils en faisaient la demande, donc le path de chaque requête était mappé sur le path du système de fichier. 

De nos jours, des vérifications sont faites 
1. Le serveur web parse le path du fichier pour récupérer l'extension du fichier.
2. Il l'utilise pour déterminer le type de fichier requêté, typiquement en le comparant à une liste de correspondances préconfigurées entre les extensions et les types MIME. La suite dépend du type de fichier et de la configuration du serveur
3. - Si ce type de fichier **n'est pas exécutable,** comme une image ou une page HTML statique, le serveur peut simplement **envoyer le contenu du fichier au client** dans une réponse HTTP.  
    - Si le type de fichier est **exécutable**, comme un fichier PHP, et que le serveur est **configuré pour exécuter des fichiers** de ce type, il attribue des variables en fonction des en-têtes et des paramètres de la requête HTTP avant d'**exécuter le script**. Le résultat peut alors être **envoyé au client** dans une réponse HTTP.  
    - Si le type de fichier est **exécutable**, mais que le serveur n'est **pas configuré pour exécuter des fichiers** de ce type, il répondra généralement par une **erreur**. Toutefois, dans certains cas, le contenu du fichier peut toujours être transmis au client sous forme de texte brut. De telles erreurs de configuration peuvent parfois être exploitées pour faire fuir le code source et d'autres informations sensibles. Vous pouvez en voir un exemple dans notre matériel pédagogique sur la divulgation d'informations.

**<u>Important :</u>**
L'en-tête de réponse `Content-Type` peut fournir des indications sur le type de fichier que le serveur pense avoir servi. Si cet en-tête n'a pas été explicitement défini par le code de l'application, il contient normalement le résultat de la correspondance entre l'`extension du fichier et le type MIME`.

## Exploiter l'upload de fichier sans restriction pour déployer un webshell
On se trouve dans ce scénario lorsque le serveur web nous permet d'upload un fichier script exécutable, tel qu'un php, java ou python.

### Web shell
Un web shell est un script qui permet à un attaquant d'exécuter des commandes sur un serveur web à travers des requêtes HTTP.
Si on a accès à un web shell, on a accès à un serveur web entier, on peut faire de l'exfiltrartion de données, etc.

Exemple de one liner web shell : `<?php echo file_get_contents('/path/to/target/file'); ?>`

Un webshell plus exhaustif serait `<?php echo system($_GET['command']); ?>` qui permettrait d'exécuter des commandes en utilisant des requête GET :
 `GET /example/exploit.php?command=id HTTP/1.1`

## Exploitation d'une validation défectueuse de téléversement de fichiers

Dans cette section, nous examinerons certaines méthodes utilisées par les serveurs web pour valider et assainir l'upload de fichiers.
Puis la manière dont nous pouvons exploiter les failles de ces mécanismes pour obtenir un shell web permettant l'exécution de code à distance.

### Validation défectueuse du type de fichier


Lors d'une soumission d'un formulaire HTTP, le navigateur envoie généralement une requête `POST` contenant un **Content-type** avec la valeur `application/x-www-form-url-encoded `. Ce type est bien pour les données "simples" (nom, adresse, etc), mais pas adéquat pour des binaires/images/etc qui contiennent de nombreux caractères non-alphabétiques. On utilise dans ce cas le **Content-type** `multipart/form-data`. 
Voir [Ici](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST) pour plus de détails.

Prenons un exemple de requête `POST` qui soumets différents types de données, ce qui devrait ressembler à ceci :
```html
POST /images HTTP/1.1
Host: normal-website.com
Content-Length: 12345
Content-Type: multipart/form-data; boundary=---------------------------012345678901234567890123456

---------------------------012345678901234567890123456
Content-Disposition: form-data; name="image"; filename="example.jpg"
Content-Type: image/jpeg

[...binary content of example.jpg...]

---------------------------012345678901234567890123456
Content-Disposition: form-data; name="description"

This is an interesting description of my image.

---------------------------012345678901234567890123456
Content-Disposition: form-data; name="username"

wiener
---------------------------012345678901234567890123456--
```
On voit que chaque partie contient un header `Content-Disposition` qui fournit une information basique sur le champ d'entrée. Ces parties peuvent également avoir leur propre header `Content-Type`, qui indiquera au serveur le type `MIME`de la donnée soumise par l'utilisateur.

**<u>Validation des uploads de fichier par des sites web</u>**
Une des manières est de regarder si le header `Content-Type` correspond à un certain type MIME prédéfini par le serveur. Par exemple un serveur peut choisir de seulement autoriser les types **image/jpeg**.
Si le serveur fait confiance au `Content-Type` de la requête sans faire d'autres validations (vérifier que le contenu du fichier correspond au type MIME), alors une attaque peut être faite en modifiant la valeur du `Content-Type`.

### Empêcher l'exécution de fichiers dans des répertoires accessibles à l'utilisateur

En plus d'empêcher un fichier d'être upload, une défense consiste à empêcher le serveur d'exécuter des scripts qui proviennent d'une entrée utilisateur.

Comme précaution, les serveurs vont généralement exécuter les scripts dont les types MIME ont été renseignés dans un fichier de configuration. Dans le cas contraire, le serveur renverra un message d'erreur ou la valeur du script sans qu'il soit exécuté. 

**<u>Exemple</u>**
```html
GET /static/exploit.php?command=id HTTP/1.1
Host: normal-website.com


HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 39

<?php echo system($_GET['command']); ?>
```

Ce genre de configuration diffère souvent entre les répertoires. Il y a de grandes chances qu'un répertoire qui reçoit des fichiers uploadés sera configuré pour avoir des contrôles plus strictes qu'à d'autres localisations du système de fichiers qui seront considérés comme en dehors de la portée des fichiers fournies par l'utilisateurs.


**Tips**:
- Les serveurs web utilisent souvent le champ **filename** dans les requêtes `multipart/form-data` pour déterminer le nom et la localisation pour sauvegarder le fichier.
