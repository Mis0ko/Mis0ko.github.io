---
title:  "Introduction"
category: "Injections SQL"
tag: "Principes de base de l'injection SQL"
---

Les injections SQL peuvent être classées en fonction d'où et comment on récupère leurs résultats.
<center><img src="/assets/images/htbAcademy/SQLInjectionFundamentals/typesOfSQLInjections.png" alt="Alt text"></center>

### SQL Injections In-band
Dans des cas simples, le résultat de la requête prévue et de la nouvelle requête peut être affichées directement sur le site front-end.

C'est ce que l'on appelle l'injection SQL **In-band** qui se présente sous deux formes : 
- L'injection SQL **Union-Based**
- L'injection SQL **Error based**

Avec l'injection SQL **Union-Based**, nous allons avoir à spécifier la colonne exacte que nous voulons lire.
Quant à l'injection SQL **Error-based**, elle est utilisée lorsque nous pouvons obtenir des erreurs PHP ou SQL dans le front-end, et nous pouvons donc intentionnellement provoquer une erreur SQL qui renvoie la sortie de notre requête.

### Blind Injections
Dans des cas plus compliqués, il se peut que la sortie ne soit pas affichée, et nous pouvons donc utiliser la logique SQL pour récupérer la sortie caractère par caractère. 

C'est ce que l'on appelle les **Bind Injections**, qui se déclinent également en deux types : L'injection **Boolean Based** et **Time Based**.

Avec l'injection **Boolean Based**, nous pouvons utiliser des instructions SQL conditionnelles pour contrôler si la page renvoie une sortie, c'est-à-dire la réponse à la requête originale, si notre instruction conditionnelle renvoie un résultat positif.

Concernant les injections **Time based**, nous utilisons des instructions conditionnelles SQL qui retardent la réponse de la page si l'instruction conditionnelle se vérifie à l'aide de la fonction **Sleep().**