---
title:  "Mutations de mots de passe"
category: "Attaques par mot de passe à distance"
tag: "Attaques de mots de passe"
---

Hashcat permet de créer des mutations de mots de passe basés sur une liste donnée et des règles implémentées dans un fichier qui ont la forme :

| Fonction | Description                                                        |
|----------|--------------------------------------------------------------------|
| :        | Ne faites rien.                                                    |
| l        | Toutes les lettres minuscules.                                     |
| u        | Mettre toutes les lettres en majuscules.                           |
| c        | Mettre en majuscule la première lettre et en minuscule les autres. |
| sXY      | Remplacer toutes les occurrences de X par Y.                       |
| $!       | Ajouter le caractère d'exclamation à la fin.                       |

Chaque règle est écrite sur une nouvelle ligne et décrit comment les mots doivent être mutés.
### Génération de la liste de mots à partir d'une règle
```console
Misoko@htb[/htb]$ hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```
**Hashcat** et **John** possèdent des listes de règles prédéfinies dont **best64.rule** qui donne de bons résultats.

### Règles Hashcat existantes
```console
Misoko@htb[/htb]$ ls /usr/share/hashcat/rules/
```

# Réutilisation de MDP / MDP par défaut.

Des mots de passes par défaut sont souvent utilisés.
Voir [DefaultCreds-Cheat-Sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet) pour avoir une liste.

Le fait d'attaquer ces services avec les credentials par défaut s'appelle le **Credential Stuffing**.

## Credential Stuffing - Syntaxe de Hydra
```console
Misoko@htb[/htb]$ hydra -C <user_pass.list> <protocol>://<IP>
```
Il existe également des listes pour les routeurs comme [celle-ci](https://www.softwaretestinghelp.com/default-router-username-and-password-list/).