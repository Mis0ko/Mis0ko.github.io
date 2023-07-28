---
title: "Business Logic vulnerabilities Examples"
category: "Serveur"
tag: "Web"
---

## CheckList en pratique
- Vérifier que les contrôles côté serveur existent (par ex : changer le prix d'un achat), 2FA qu'on ne puisse pas bruteforce.
- Vérifier la prise en compte des données non conventionnelles (par ex: limite des entiers)
- Penser au troncage d'adresse mail si des vérifications sont faites dessus.
- Données obligatoires non fournies (paramètres `GET`,`POST` & `cookies`):
    - Supprimer un seul paramètre à la fois afin de s'assurer que tous les chemins de code pertinents sont atteints. 
    - Essayer de supprimer le nom du paramètre ainsi que sa valeur. Le serveur traitera généralement les deux cas différemment.
    - Suivre les processus en plusieurs étapes jusqu'au bout. Il arrive que la modification d'un paramètre à une étape ait un effet sur une autre étape plus loin dans le processus.
- Vérifier que les séquences d'étapes prennent en compte qu'on n'arrive pas jusqu'au bout (authentification 2FA se fait avant la vérification du code par exemple).
- Vérifier que chaque étape est requise et contrôlée lors d'un flux de travail :
    - En sautant certaines étapes.
    - En accédant à une étape plus d'une fois.
    - En revenant à une étape précédente.
On peut prendre comme exemple le fait de jouer une requête POST "validation de panier", rejouer sans faire la requête qui vérifie la valeur du panier.

## Confiance excessive dans les contrôles côté client
Une hypothèse fondamentalement érronée est le fait de croire que l'utilisateur va intéragir avec l'application seulement via l'interface web fournie (et donc faire confiance aux validation côté client).
Cependant, un attaquant peut simplement utiliser des outils comme Burp Proxy pour altérer les données après qu'elles soient envoyées par le navigateur mais avant qu'elles soient passées par la logique du côté serveur. L'attaquant peut ainsi faire des dommages de manière relativement facile.

**Recommandation** : Toujours vérifier l'intégrité des données et faire des validations de données côté serveur.

## Ne pas traiter les données non conventionnelles

Lors de l'audit d'une application, utiliser Burp Proxy et Repeater pour essayer de soumettre des valeurs non conventionnelles. En particulier, essayer de saisir des données dans des fourchettes que les utilisateurs légitimes ne saisiront probablement jamais.

Exemples :
- Des entrées numériques exceptionnellement élevées ou basses. 
- Des chaînes de caractères anormalement longues pour les champs textuels.
- Des types de données inattendus. 

En observant la réponse de l'application, essayer de répondre aux questions suivantes :

- Y a-t-il des limites imposées aux données ?
- Que se passe-t-il lorsqu'on atteint ces limites ?
- Une transformation ou une normalisation est-elle effectuée sur les données d'entrées?

Garder à l'esprit que si on trouve un formulaire sur le site web cible qui ne parvient pas à gérer en toute sécurité les entrées non conventionnelles, il est probable que d'autres formulaires présenteront les mêmes problèmes.

## Faire des hypothèses erronées sur le comportement des utilisateurs

### Les utilisateurs de confiance ne restent pas toujours dignes de confiance
Les applications peuvent sembler sûres parce qu'elles mettent en œuvre des mesures apparemment robustes pour appliquer les règles de l'entreprise.

Malheureusement, certaines applications commettent l'erreur de supposer qu'après avoir passé ces contrôles stricts au départ, l'utilisateur et ses données sont indéfiniment dignes de confiance. Il peut en résulter une application relativement laxiste des mêmes contrôles à partir de ce moment-là.

## Les utilisateurs ne fournissent pas toujours les données obligatoires
Comme on en a parlé précédemment, ce cas ne concerne pas les utilisateurs ordinaires mais les attaquants.
Ce problème se pose lorsqu'il existe plusieurs signatures d'un même nom de fonction. Du côté serveur, la présence ou absence d'un paramètre particulier peut déterminer quel code est exécuté.

Lorsqu'on recherche des défauts de logique, on doit supprimer les paramètres à tour de rôle et observer l'effet sur la réponse.
Pour cela (paramètres `GET`,`POST` & `cookies`):
- Supprimer un seul paramètre à la fois afin de s'assurer que tous les chemins de code pertinents sont atteints.
- Essayer de supprimer le nom du paramètre ainsi que sa valeur. Le serveur traitera généralement les deux cas différemment.
- Suivre les processus en plusieurs étapes jusqu'au bout. Il arrive que la modification d'un paramètre à une étape ait un effet sur une autre étape plus loin dans le processus.

<u>Exemple à penser :</u>
- Non vérification d'un mdp pour le changement de ce dernier une fois authentifié.
- Fonctionnalité mot de passe oublié.

## Les utilisateurs ne suivent pas toujours la séquence prévue

De nombreuses transactions s'appuient surs des flux de travail prédéfinis consistant en une séquence d'étape. En principe, l'interface web guide l'utilisateur du début à la fin de ce processus, mais ce n'est pas le cas pour les attaquants qui ne respecteront pas la séquence prévue.

<u>Exemple :</u>
De nombreux sites qui mettent en oeuvre l'authentification à 2 facteurs (2FA) exigent que les utilisateurs se connectent à une page avant d'entrer un code de vérification sur une autre page.
Si les développeurs supposent que les utilisateurs suivront toujours ce processus en ne vérifiant pas ce qu'ils font, les attaquants peuvent contourner l'étape d'authentification à 2 facteurs.

<u>Ce qu'il faut retenir :</u>
Faire des hypothèses sur la séquence des évènements peut conduire à un large éventail de problèmes, même au sein d'un même flux de travail ou d'une même fonctionnalité.

Pour identifier ce type de défaults, il faut forcer le navigateur à soumettre des requête dans une séquence inattendu.
Par exemple :
- En sautant certaines étapes
- accéder à une étape plus d'une fois
- revenir à une étape précédente

Il est parfois possible d'accéder à des étapes en soumettant différents ensembles de paramètres à la même URL.

Comme pour toutes les failles logiques, il faut essayer d'identifier les hypothèses émises par les développeurs et la surface d'attaque. 

## Défauts spécifiques à un domaine (au sens business, pas DNS)
On retrouve de nombreux défauts de logique spécifique à un domaine commercial ou au but du site.
La fonctionnalité de réduction d'un site web en est un bon exemple :

Prenons l'exemple d'une boutique en ligne qui offre une remise de 10 % sur les commandes supérieures à 1 000 dollars. Cette boutique pourrait être vulnérable aux abus si la logique commerciale ne vérifie pas si la commande a été modifiée après l'application de la réduction.

Dans ce cas, un attaquant pourrait simplement ajouter des articles à son panier jusqu'à ce qu'il atteigne le seuil de 1 000 dollars, puis supprimer les articles qu'il ne souhaite pas avant de passer la commande. Il bénéficierait alors de la remise sur sa commande, même si celle-ci ne répond plus aux critères prévus.


<u>**A faire attention :**</u> identifier les situations où les prix (ou autres valeurs sensibles) sont ajustées basé sur un critère déterminé par les actions de l'utilisateur.





