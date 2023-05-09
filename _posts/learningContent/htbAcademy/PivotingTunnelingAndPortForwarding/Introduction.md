---
title:  "Bases"
category: "Introduction"
tag: "Pivoting, Tunneling, et Port Forwarding"
---
Durant un pentest, il arrive souvent de se retrouver dans une situation où nous avons déjà compromis des **credentials, clés ssg, hashes, etc** nécessaire afin de passer à un autre hôte, mais il se peut qu'il n'y ait pas d'autres hôte accessible directement à partir de notre hôte d'attaque.
On peut utiliser ce qu'on appelle un **hôte pivot** déjà compromis afin de passer à la prochaine cible.

Une des premières choses à faire lorsqu'on arrive sur un hôte est de vérifier **le niveau de privilège, les connections réseaux ainsi que les éventuels VPN et autre logiciel d'accès à distance**.  Si l'hôte à plusieurs cartes réseaux, nous pouvons nous déplacer dans différents segments réseaux. 

# Mouvement latéral, Pivoting & Tunneling

### Lateral Movement
Technique utilisée pour améliorer notre accès à **d'autres hôtes, applications et services** dans un environnement réseau. Cela permet également de gagner accès à **des ressources** du domaine pour **élever nos privilèges**.
> Exemple pratique : durant un pentest, on obtient un accès initiale à une cible et on obtient un contrôle d'un compte admin local. Après un scan, on voit 3 autres hôtes Windows dans le réseau.


# Validation d'entrées utilisateurs








```sql

```

<center><img src="/assets/images/htbAcademy/SQLInjectionFundamentals/readingFiles1.png" alt="Alt text"></center>
