---
title: "Business Logic"
category: "Serveur"
tag: "Web"
---

## CheckList en pratique

>Voir [Exemple](https://mis0ko.github.io/serveur/BusinessLogicVulnExamples/)

## Explication de la vulnérabilité
Les vulnérabilités de type "business logic" sont des failles dans la conception et la mise en oeuvre d'une application qui permet à un attaquant d'obtenir un comportement non souhaité.

La "business logic" a pour but de dicter comment l'application devrait se comporter selon un certain scénario.

**<u>Exemple de comportements non-attendu :</u>**
- Un attaquant peut être en mesure d'effectuer une transaction sans passer par le processus d'achat prévu.
- Une validation défaillante ou inexistante des données fournies par l'utilisateur peut permettre à ce dernier d'apporter des modifications arbitraires à des valeurs critiques pour la transaction ou de soumettre des données absurdes

## Comment s'en prémunir

- S'assurer que les développeurs et les testeurs comprennent le domaine auquel s'adresse l'application.
- Éviter de faire des suppositions implicites sur le comportement de l'utilisateur où d'autres parties de l'application.
- Vérifier que la donnée est valide avant de s'en servir du côté serveur.
- S'assurer que les développeurs et les testeurs sont en mesure de comprendre pleinement la manière dont l'application est censée réagir dans différents scénarios.

**<u>D'un point de vue bonnes pratiques des développeurs :</u>**
- Maintenir des documents de conception et des flux de données clairs pour toutes les transactions et tous les flux de travail, en notant les hypothèses.
- Rédiger le code aussi clairement que possible. Un code bien écrit ne nécessite pas de documentation pour être compris.
- Dans les cas inévitablement complexes, la production d'une documentation claire est cruciale pour s'assurer que les autres développeurs et testeurs savent quelles hypothèses sont faites et quel est exactement le comportement attendu.
- Notez toutes les références à d'autres codes qui utilisent chaque composant, et pensez aux effets secondaires des dépendances.













