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