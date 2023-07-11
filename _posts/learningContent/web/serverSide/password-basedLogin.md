---
title: "Authentification with passwords"
category: "Authentification"
tag: "Web : Serveur"
---

# Attaque Brute-force

On parle d'attaque par force brute lorsqu'un attaquant utilise un système d'essais et d'erreurs pour tenter de deviner les informations d'identification valides de l'utilisateur,  de manière automatisée en utilisent des listes de mots de passe et de noms d'utilisateurs. 



## Énumération de noms d'utilisateurs
La réponse à la tentative d'authentification peut réduire le temps d'attaque de brute force ("username already taken", ...).


## Checklist
### Récupération d'information pour le nom d'utilisateur
- réponse HTTP (peut contenir des adresses email, login, ..)
- adresses mails d'utilisateur
- nom d'utilisateurs courant (admin, administrator, ...)
- Changement de Code de statut HTTP : toujours le même lorsque c'est erroné.
- Message d'erreur : peut être différent en fonction des combinaisons (login + password incorrect, seulement login, seulement password)
- Temps de réponse : Si la plupart des requêtes prennent le même temps et qu'une dure un peu plus longtemps, cela peut être un traitement différent en fonction du login (par exemple si login existe, alors on regarde seulement à ce moment là les passwords).

Note : faire les lab




