---
title: "File Upload Vulnerabilities"
category : "Serveur"
tag: "Web"
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
```http
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
```http
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

### Liste noire des types de fichiers dangereux insuffisante
Une des manières de se prémunir des scripts malicieux est de créer une blacklist des extensions de fichiers potentiellement dangeureux comme `.php`.
Une blacklist est cependant une mauvaise idée car on peut contourner ce mécanisme en utilisant des extensions qui sort des sentiers battus comme `.php5` ou `.shtml`, etc.

#### Remplacer la configuration du serveur
Les serveurs ne vont exécuter les types de fichiers que s'ils ont été configurés pour cela.
Par exemple, un serveur Apache va exécuter des fichiers `PHP`de la part du client si les développeurs ont ajoutés les directives suivantes dans le fichier `/etc/apache2/apache2.conf`:
```
LoadModule php_module /usr/lib/apache2/modules/libphp.so
AddType application/x-httpd-php .php
```

De nombreux serveurs permettent de créer des configurations propre à un dossier afin de remplacer des configurations globales à l'application.
- Par exemple, les serveurs `Apache` chargeront une configuration spécifique au répertoire à partir du fichier `.htaccess` s'il existe.
-  De manière équivalente, les développeurs peuvent configurer des répertoires spécifiques sur les serveurs `IIS` à l'aide d'un fichier `web.config`. 
Voici un exemple de directive pour permettre de fournir des fichiers JSON aux utilisateurs. 
```
<staticContent>
    <mimeMap fileExtension=".json" mimeType="application/json" />
</staticContent>
```

Les serveurs web utilisent ce type de fichiers de configuration mais il n'est normalement pas possible d'y avoir accès avec des requêtes HTTP. Cependant, il peut arriver qu'un serveur ne parvienne pas à nous empêcher d'upload notre fichier de configuration malicieux. Dans ce cas, on pourra changer la configuration du serveur et autoriser l'upload de fichiers avec des extensions qui ne devraient pas l'être.


#### Obfusquer les entensions de fichiers
Comme on a pu le voir précédemment, une blacklist est une mauvaise idée. Dans cette partie nous verrons quelques idées en plus à l'aide de l'obfuscation pour contourner les blacklists d'extension de fichier.
Il faut garder en tête que le **code de validation (d'extension dans notre cas), est indépendant du code qui mappe l'extension de fichier** au type `MIME`.
A partir de cette hypothèse on peut déduire plusieurs techniques de contournement du code de validation.
- Si le code de validation est sensible à la casse mais pas le code de mappage, alors utiliser une extension comme `exploit.pHp` pourrait contourner la validation.
- Fournir plusieurs extensions. En fonction de la méthode de parsing du nom de fichier, le fichier suivant pourrait être interprété comme une image où un  script : `exploit.php.jpg`
- Utiliser l'encodage url (ou double encodage) pour les `.` `\` ou `/`. Si la valeur n'est pas décodé lors de la validation mais plus tard du côté serveur, cela permettrait d'upload des fichiers malicieux qui seraient bloqués autrement : `exploit%2Ephp`
- Ajouter des `;` ou `%00` (octet nul url-encodé) avant l'extension. Si la validation est faite dans un langage haut niveau **(PHP ou java)**, mais le serveur traite le fichier avec des fonctions bas-niveaux **(C/C++)**, cela peut créer des divergences dans ce qui est traité à la fin du nom de fichier. `exploit.asp;.jpg` ou `exploit.asp%00.jpg`.
- Essayer l'utilisation des **caractères unicode multioctets**, qui peuvent être convertis en octets nuls et en points après la conversion ou la normalisation unicode. Des séquences telles que `xC0 x2E`, `xC4 xAE` ou `xC0 xAE` peuvent être traduites en `x2E` si le nom de fichier est analysé comme une **chaîne UTF-8**, puis converties en **caractères ASCII** avant d'être utilisées dans un chemin d'accès.
- Si une défenses consistant à supprimer ou à remplacer les extensions dangereuses n'est pas implémenté de manière récursive, alors elle peut être contournée : `exploit.p.phphp`


### Validation défectueuse de contenu du fichier

Les serveurs plus sécurisés peuvent essayer de vérifier le contenu d'un fichier plutôt que de faire confiance au `Content-Type` fourni par la requête HTML.

Ce type de validation se fait en vérifiant des caractéristiques intrinsèques au type de fichier uploadé. Par exemple, une validation d'image pourrait consister à vérifier sa dimension (qu'un script php n'aurait pas).

Également, certains types de fichiers contiennent une séquence d'octet spécifique dans leurs headers ou footers, qui peut être utiliser comme signature d'un type de fichier.
Par exemple, les JPEG commencent toujours par les octets `FF D8 FF`.

Des outils comme `ExifTool` peuvent être utilisés pour créer des fichiers polyglotte contenant du code malveillant dans ses métadonnées.

#### Méthode pour créer un script dans une image 
```console
exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" <YOUR-INPUT-IMAGE>.jpg -o polyglot.php
```
Puis faire une recherche sur `START` dans la réponse à la requête HTTP.

### Exploiter "race-conditions" (situation de compétition) d'upload de fichiers

Les frameworks modernes sont mieux armés contre les types d'attaques que nous avons vu précédemment. Ils n'uplaod pas les fichiers directements vers leur destination prévue dans le système de fichier. Au lieu de cela, ils upload les fichiers dans un répertoire temporaire "bac à sable" d'abord et randomise leurs noms pour éviter d'écraser des fichiers existants. La plupart du temps, un antivirus va scanner le fichier à la recherche de malware.

Ils procèdent ensuite à la validation du fichier temporaire et ne le transfère à sa destination qu'une fois qu'il est jugé sûr.


Cependant, les développeurs implémentent parfois leurs propre processus d'upload de fichiers de manière indépendante du framework, ce qui peut mener à des situations de **race conditions**. 
Par exemple, des sites upload leurs fichier directement sur le système de fichier principale puis le supprime s'il ne passe pas le test de validation. Ce type de comportement laisse un court moment pour exécuter le fichier avant qu'il ne soit supprimé. 

## Exploitation des vulnérabilités d'upload de fichiers sans RCE
La conséquence la plus sérieuse d'un upload de fichier non sécurisé en terme de vulnérabilité est le RCE. 
Cependant, d'autres vulnérabilités peuvent être exploitées.

### Upload des scripts malicieux côté client
Si le fichier upload n'est pas exécuté côté serveur, il peut très bien l'être côté client.
Par exemple, s'il est possible d'upload un fichier **HTML** ou une image **SVG**, il est possible d'utiliser une balise `<script>` pour créer des payloads d'**XSS stockées**.

Si le fichier upload apparaît sur une page visitée par d'autres utilisateurs, leurs navigateurs exécutera le script quand il essayera d'afficher la page. 
***A noter qu'en raison des restrictions de `same-origin policy`, ce type d'attaques fonctionnent seulement si le fichier uploadé est fourni avec la même origine que celle vers laquelle elle a été upload***

### Exploitation de vulnérabilités dans le parsing des fichiers uploadés

Si le fichier téléchargé semble être à la fois stocké et servi de manière sécurisée, le dernier recours est d'essayer d'exploiter les vulnérabilités spécifiques au parsing ou au traitement des différents formats de fichiers. 

Par exemple, si on sait que le serveur analyse les fichiers basés sur **XML**, tels que les fichiers **.doc** ou **.xls de Microsoft Office**, il peut s'agir d'un vecteur potentiel pour les attaques par injection XXE.

## Upload des fichiers en utilisant PUT

Il est bien de savoir que certains serveurs web peuvent être configurés pour prendre en charge les requêtes PUT. S'il n'y a pas de défenses convenables en place, il peut être possible d'upload des fichiers malveillants même sans interface web disponible.


```http
PUT /images/exploit.php HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-httpd-php
Content-Length: 49

<?php echo file_get_contents('/path/to/file'); ?>
```

Il est peut être intéressant d'essayer d'envoyer des requêtes `OPTIONS` à différents points de terminaison pour tester ceux qui annoncent la prise en charge de la méthode `PUT`.

## Comment prévenir les vulnérabilités liées à l'upload de fichiers

- Vérifier l'extension du fichier dans une liste blanche d'extensions autorisées plutôt que dans une liste noire d'extensions interdites. Il est beaucoup plus facile de deviner les extensions que l'on souhaite autoriser que celles qu'un attaquant pourrait tenter de télécharger.
- S'assurer que le nom du fichier ne contient pas de sous-chaînes pouvant être interprétées comme un répertoire ou une séquence de traversée (`../`).
- Renommer les fichiers téléchargés afin d'éviter les collisions qui pourraient entraîner l'écrasement de fichiers existants.
- Ne pas télécharger de fichiers dans le système de fichiers permanent du serveur avant qu'ils n'aient été entièrement validés.
- Dans la mesure du possible, utiliser un cadre établi pour le prétraitement des fichiers téléchargés plutôt que d'essayer d'écrire ses propres mécanismes de validation.





