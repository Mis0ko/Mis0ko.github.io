---
title:  "Énumeration d'infrastructure"
category: "Introduction"
tag: "Empreinte"
---
# Enumeration d'infrastructure
## Information sur le domaine
L'information de base est le certificat SSL, qui associe le nom de domaine à une URL,
et permet d'établir avec certitude le lien entre site internet et son propriétaire.
Une base de données de ces certificats se trouve par l'IHM crt.sh.

On peut utiliser l'outil **shodan** pour récupérer des informations, ou des appareils (comme ceux d'IOT)
à partir d'une adresse IP de domaine par exemple, et de trouver des ports ou autre information.

## Ressources Cloud

### Google avec inurl: et intext:
intext:entreprise  inurl:amazonaws.com 
avec 

### Autres
domain.glas (découvre information comme celles dans le certificat SSL), GrayHatWarfare (permet de découvrir des serveurs de stockage cloud, ...).
