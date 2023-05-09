---
title: "Vulnérabilités d'authentification"
category: "Authentification"
tag: "Web : Serveur"
---
# Definition
Authentification = procédé pour vérifier l'identité de quelqu'un.

Il existe 3 types d'authentification, en fonction de :
- ce que tu sais : mot de passe, clé rsa, ..
- ce que tu as : portable, token, ..
- ce que tu es : pattern de comportement, biometrique, ..

### Différence entre autorisation et authentification
L'authentification est le procédé de vérifier si un utilisateur est vraiment ce qu'il prétend d'être, alors que l'autorisation implique de vérifier qu'un utilisateur est autoriser à faire quelquechose.

### De quelle manière subvienne les vulnérabilités d'authentification ?

Il existe 2 manières principales :
- Le mécanisme d'authentification est faible car il écoue à se protéger des attaques brutes-forces
- Des fauts logiques ou une implémentation défaillante permet un contournement du mécanisme d'authentification. C'est ce qu'on appelle "broken authentification".


