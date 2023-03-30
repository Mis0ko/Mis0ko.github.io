---
title:  "XSS Stockée"
category: "Notions de base sur les XSS"
tag: "Cross-Site Scripting (XSS)"
---
L'**XSS stockée** ou **XSS persistante** est la plus critique des XSS.\
Si notre payload d'XSS injectée se retrouve stockée dans une base de données d'un serveur web,
et récupérée lors d'une visite de page, cela signifie que notre attaque XSS est persistante
et peut affecter n'importe quel utilisateur qui visite la page.

Elle est critique dans le sens où elle touche le plus d'audience et n'est pas facilement retirable.

# Payload pour tester les XSS

On peut tester de manière simple si une page est vulnérable au XSS avec le payload suivant :
```html
<script>alert(window.origin)</script>
```
Si on suppose que la page ne contrôle aucune entrée utilisateur.
On peut voir le résultat dans la page web en faisant un **CTRL+U**.

De nombreuses applications web modernes utilises des iframes cross-domain pour gérer les entrées utilisateurs,
de sorte à ce que même si le formulaire est vulnérable au xss, cela ne serait pas une vulnérabilité pour 
l'application web principale.

On affiche **window.origin** pour cela, afin de voir l'url sur lequel le script est executé.

Le payload **alert()** peut être bloqué par les navigateurs modernes.
De ce fait, il peut être intéressant de connaître quelques autres payloads de bases:
```html
<plaintext>
<script>print()</script>
```

Pour savoir s'il s'agit d'une XSS stockée, il suffit de rafraîchir la page. Si le script s'exécute de nouveau,
cela signifie que chaque visiteur va déclencher l'alerte à l'affichage de la page.