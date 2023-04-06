---
title:  "Prévention des XSS"
category: "Attaque XSS"
tag: "Cross-Site Scripting (XSS)"
---
Les vulnérabilités sont principalement liés à deux parties d'une application web. Un **Source** comme un champ d'entrée utilisateur et un **Sink** qui affiche les données d'entrée.

Dans le but de se prévenir des vulnérabilités XSS, un assainissement et une validation des entrées sont nécessaires, à la fois dans le front et back-end.
# Front end 

## Validation des entrées
La validation des données s'opère principalement à partir d'un typage de données ou d'expressions régulières adéquates (format email par exemple).

## Assainissement des entrées

En plus de la validation des données, il est important d'en faire un assainissement comme par exemple échapper les caractères spéciaux avec des backslash **\\**.

## Entrée directe

Enfin, il est important de s'assurer de ne jamais utiliser d'entrées utilisateurs directement dans certains tags HTML, comme :

- code JavaScript : \<script></script>
- Code CSS Style : \<style></style>
- Champs de Tag/Attributs : \<div name='INPUT'></div>
- Commentaire HTML : \<!-- -->

Si une donnée utilisateur arrive dans une des balises précédentes, du code Javascript peut être inséré ce qui pourrait mener à une injection XSS.

Pour le code Javascript, il est important d'éviter également les fonctions JS suivantes si aucune vérification de données sont faites :
- **DOM.innerHTML**
- **DOM.outerHTML**
- **document.write()**
- **document.writeln()**
- **document.domain**

Et les fonctions jQuery suivantes :

- **html()**
- **parseHTML()**
- **add()**
- **append()**
- **prepend()**
- **after()**
- **insertAfter()**
- **before()**
- **insertBefore()**
- **replaceAll()**
- **replaceWith()**

Toutes ces fonctions écrivent du texte brut dans du code HTML.

# Back-end 

Il est aussi important de réfléchir à comment prévenir les vulnérabilités XSS avec des mesures back-end pour empêcher l'usage des XSS stockées et Réfléchies.

## Validation des entrées

La validation des entrées du côté Back-end est similaire à celle du Front-end. On peut donc s'y référer pour savoir quoi mettre en place.

## Assainissement des entrées

Idem que pour le Front-end

## Encodage HTML des données de sortie

Un autre aspect important du côté back-end est **l'Output Encoding**.
Cela signifie d'encoder tous les caractères spéciaux dans leur **codes HTML**.
Cela permet d'afficher l'entrée utilisateur sans risquer d'introduire une XSS. On retrouvera des caractères comme **<** remplacé par **&lt** de sorte à ce que le navigateur les affichent correctement sans risquer d'XSS.

En php, on retrouve les fonctions comme **htmlspecialchars ou htmlentities** pour cela.

## Configuration de serveurs

En plus des recommandations précédentes, il existe des configurations de serveurs web pouvant être mises en place pour prévenir les attaques XSS, comme :

- Utiliser HTTPS pour l'ensemble du domaine
- Utiliser des en-têtes de prévention XSS, comme **X-XSS-Protection.**
- Utilisation du Content-Type approprié pour la page, comme  **X-Content-Type-Options=nosniff.**
- L'utilisation d'options de politique de sécurité du contenu, comme **script-src 'self'**, qui n'autorise que les scripts hébergés localement.
- Utiliser les flags de cookie **HttpOnly et Secure** pour empêcher JavaScript de lire les cookies et ne les transporter que via HTTPS.

En plus des recommandations précédentes, avoir un bon **WAF (Web Application Firewall)** peut réduire significativement les chances d'exploitations d'XSS.