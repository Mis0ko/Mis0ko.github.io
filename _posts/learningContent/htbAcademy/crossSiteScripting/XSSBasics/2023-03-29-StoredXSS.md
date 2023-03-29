---
title:  "XSS Stockée"
category: "Notions de base sur les XSS"
tag: "Cross-Site Scripting (XSS)"
---
L'**XSS stockée** ou **XSS persistant** est la plus critiques des XSS.\
Si notre payload XSS injecté se retrouve stocké dans une base de données d'un serveur back-end,
et récupéré lors d'une visite de page, cela signifie que notre attaque XSS est persistante
et peut affecter n'importe quel utilisateur qui visite la page.

Elle est critique dans le sens où elle touche le plus d'audience et n'est pas facilement retirable.

# Payload pour tester les XSS

On peut tester de manière simple si une page est vulnérable au XSS avec le payload suivant :
```html
<script>alert(window.origin)</script>
```
Si on suppose que la page ne sanitize aucune entrées utilisateur.




