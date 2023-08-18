---
title: "2.Authentification"
category: "Serveur"
tag: "Web"
---
# Definition
Authentification = procédé pour vérifier l'identité de quelqu'un.

Il existe 3 types d'authentification, en fonction de :
- ce que tu sais : mot de passe, clé rsa, ..
- ce que tu as : portable, token, ..
- ce que tu es : pattern de comportement, biometrique, ..

### Différence entre autorisation et authentification
L'authentification est le procédé de vérifier si un utilisateur est vraiment ce qu'il prétend d'être, alors que l'autorisation implique de vérifier qu'un utilisateur est autorisé à faire quelquechose.

### De quelle manière subvienne les vulnérabilités d'authentification ?

Il existe 2 manières principales :
- Le mécanisme d'authentification est faible car il échoue à se protéger des attaques brutes-forces
- Défauts logiques ou une implémentation défaillante permet un contournement du mécanisme d'authentification. C'est ce qu'on appelle `broken authentification`.

## Enumération de noms d'utilisateurs
L'énumération de noms d'utilisateurs est possible lorsque le comportement du site web va changer dans le but d'identifier un nom d'utilisateur valide. On peut voir 3 différences :
- **Code HTTP** : durant un bruteforce, les  code http seront globalement les mêmes pour les différentes tentatives, à part lors d'une tentative positive d'authentification.
- **Messages d'erreurs** : le message d'erreur peut être différent selon l'existence ou non d'un nom d'utilisateur.
- **Temps de réponse**: si le temps de réponse d'une requête est plus long que la moyenne, cela peut indiquer qu'un nom d'utilisateur est correct, par exemple si un site contrôle que le mot de passe est valide.

### Technique d'exploitation
Il existe plusieurs cas de figures pour énumérer des usernames :
- Vérifier si le site renvoie un réponse de taille significativement différente si le username est valide ou non (en dehors du password), avec une liste de username en payload.
- Vérifier si le message d'erreur est toujours le même durant le bruteforce durant l'énumération de username. Pour le sélectionner **(Sniper), Options-> Grep-Extract->Add et sélectionner le message.**
- **Utilisation du temps de réponse** : cela fonctionne bien si on a déjà des credentials et qu'on peut essayer avec un long password invalide si le temps de réponse est plus grand. Après une attaque d'énumération de username faite avec un long password invalide fixe, cliquer sur **Columns et sélectionné Response received et Response completed**.

Pour contrer la protection bruteforce:
- **Blocage d'IP** : utiliser le **header X-Forwarded-For** s'il est supporté (permet d'usurper une adresse IP). Le faire varier à chaque requête avec **Pitchfork**.
- de **faille logique** (blocage après un nombre de tentative mais réinitialisé après connection établie) : si on connait un username/password et qu'on veut bruteforce le password d'un compte dont on a l'identifiant, on fait une liste de usernames (en répétant par intervalle régulier l'identifiant du compte connu), une liste de password (idem avec le password connu). Puis on exécute une attaque par **Pitchfork**.

## Authentification HTTP Basique
Cette authentification est ancienne mais peut toujours exister. Dans ce cadre là, le client reçoit un token du serveur, qui est construit par concaténation du nom d'utilisateur et du mot de passe, encoder en base64. 

Le token est stocké et géré par le navigateur, qui l'ajoute automatiquement dans le header ***Authorization*** à chaque requête.
```http
Authorization: Basic base64(username:password)
```

Cette authentification n'est pas considéré comme sécurisé car on envoie de manière régulière les identifiants de l'utilisateur dans la requête.

# Momentum sur HTTPS
HTTPS (HTTP Secure) est la version sécurisée de HTTP. Il utilise une couche de chiffrement pour protéger les données transitant entre le navigateur et le serveur. HTTPS utilise un protocole appelé SSL ou TLS pour établir une connexion sécurisée. Lorsqu'une connexion HTTPS est établie, le trafic est chiffré, ce qui rend les données illisibles pour les personnes non autorisées qui tenteraient d'intercepter ou d'espionner la communication.

On peut catégoriser les éléments chiffrés :
- **En-têtes (Headers) :**
Tous les en-têtes HTTP, tels que User-Agent, Accept, Cookie, etc., sont chiffrés avec HTTPS. Cela inclut toutes les informations envoyées dans les en-têtes de la requête et de la réponse.
- **Corps de la requête (Request Body) :**
Le corps (payload) de la requête, qui peut contenir des données sensibles telles que des formulaires, des fichiers téléchargés, etc., est chiffré avec HTTPS.
- **Paramètres d'URL (Query Parameters) :**
Les paramètres d'URL envoyés dans les requêtes GET sont chiffrés avec HTTPS. Par exemple, si vous avez une URL comme https://www.example.com/page?param1=value1&param2=value2, les valeurs de paramètres (value1, value2) seront chiffrées.
- **Cookies :**
Les cookies envoyés dans les requêtes et les réponses HTTP sont chiffrés avec HTTPS. Les cookies sont utilisés pour stocker des informations d'authentification et de session, et leur chiffrement garantit leur confidentialité.
- **Méthode de requête (Request Method) :**
La méthode de requête HTTP, telle que GET, POST, PUT, DELETE, etc., n'est pas spécifiquement chiffrée, car elle fait partie de la ligne de requête HTTP. Cependant, la méthode de requête est généralement envoyée dans l'en-tête de la requête, qui lui est chiffré avec HTTPS.