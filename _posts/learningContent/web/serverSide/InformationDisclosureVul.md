---
title: "Information Disclosure Vulnerable"
tag: "Web : Serveur"
---

## CheckList en pratique


## Explication de la vulnérabilité

La divulgation d'informations (information disclosure), également appelée fuite d'informations(information leakage), se produit lorsqu'un site web révèle involontairement des informations sensibles à ses utilisateurs. 
Les sites web peuvent divulguer toutes sortes d'informations à un attaquant potentiel, notamment :
- Des données sur des utilisateurs (username/password)
- Des données sensibles commerciales
- Des détails technique sur le site web ou son infrastructure

Quelques exemples d'**information disclosure** :
- Révéler les noms des répertoires cachés, leur structure et leur contenu par le biais d'un fichier `robots.txt` ou d'une liste de répertoires.
- Permettre l'accès aux fichiers de code source par le biais de backups.
- Mentionner explicitement les noms des tables ou des colonnes de la base de données dans les messages d'erreur.
- Exposer inutilement des informations très sensibles, telles que les détails d'une carte de crédit.
- Coder en dur dans le code source des clés API, des adresses IP, des identifiants de base de données, etc.
- Suggérer l'existence ou l'absence de ressources, de noms d'utilisateur, etc, par le biais de différences subtiles dans le comportement de l'application.

## Exploiter la divulgation d'informations
Voir [ici](http://misoko.github.io)

## Comment prévenir les vulnérabilités liées à la divulgation d'informations