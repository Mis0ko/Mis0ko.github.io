---
title:  "Introduction"
category: "MySQL"
tag: "Principes de base de l'injection SQL"
---

Les injections SQL peuvent être classées en fonction d'où et comment on récupère leurs résultats.
<center><img src="/assets/images/htbAcademy/SQLInjectionFundamentals/typesOfSQLInjections.png" alt="Alt text"></center>

Dans des cas simples, le résultat de la requête prévue et de la nouvelle requête peut être affichées directement sur le site front-end.

C'est ce que l'on appelle l'injection SQL **In-band** qui se présente sous deux formes : 
- L'injection SQL **Union-Based**
- et l'injection SQL basée sur l'erreur.