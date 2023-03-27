---
title:  "Connaissances basiques"
category: "Introduction"
tag: "Attaques de mots de passe"
---
# Introduction rapide

# Linux
Les mots de passe sont stockées chiffrés dans le fichier **/etc/shadow** sous forme de hash.
## Format /etc/shadow
Le format des lignes du fichier est de la forme suivante :

| nom d'utilisateur : | mot de passe chiffré : | jour de la dernière modification : | âge minimum : | âge maximum : | période d'avertissement : | période d'inactivité : | date dexpiration : | champ réservé |

Le format du type de chiffrement du mot de passe à la forme suivante :

| \$ id | \$ sel | \$ hash |

Autrefois, le nom d'utilisateur ainsi que le mot de passe étaient stockées dans **/etc/passwd** accessible en lecture à tous les utilisateurs.\
Le fichier **shadow** n'est lisible que par l'utilisateur root.

## Format /etc/passwd

| nom d'utilisateur : | mot de passe : | uid : | gid : | commentaire : | répertoire personnel : | cmd exécutée après louverture de session |

# Processus d'authentification Windows
Le LSA (Local Security Authority) est un sous-système protégé qui authentifie les utilisateurs et les connectent à l'ordinateur local. En outre, le LSA maintient des informations sur tous les aspects de la sécurité locale sur un ordinateur. Il fournit également des services de traduction de noms en IDs de sécurité (SIDs).

Le LSA garde la trace des politiques de sécurité et des comptes qui résident sur un système informatique (qui sont stockées dans l'Active Directory).

Il permet également de surveiller les accès aux objets, les permissions utilisateurs et générer des messages de monitoring.

## LSASS (Local Security Authority Subsystem Service)
C'est un ensemble de modules qui a accès à tous les processus d'authentification qui se trouvent dans **/SystemRoot/System32/Lsass.exe**.

Il est responsable de la politique de sécurité du système **local**, de l'authentification des utilisateurs et de l'envoie de logs au journal des évènements.

## Bases de données SAM
Le SAM (Security Account Manager) est une Base de données Windows qui **stocke les mots de passes des utilisateurs.** 

Les mots de passe des utilisateurs sont stockés sous forme de hachage dans une structure de registre, sous forme de hachage **LM** ou **NTLM**. 
Ce fichier est situé dans **%SystemRoot%/system32/config/SAM** et est monté sur **HKLM/SAM**. Des autorisations de niveau **SYSTEM** sont nécessaires pour le visualiser.

Les systèmes Windows peuvent être assignés à un groupe de travail ou domain lors de l'installation. S'il est ajouté à un groupe de travail, le SAM peut être géré localement.\
Dans un domaine en revanche, c'est le contrôleur de domaine (DC) qui s'occupe de ça.

## Credential Manager
Le Credential Manager est une fonction intégrée à tous les systèmes d'exploitation Windows qui permet aux utilisateurs de **sauvegarder les credentials** qu'ils utilisent pour accéder à diverses ressources réseau et sites Web. Les credentials sauvegardées sont stockées en fonction des profils d'utilisateurs dans le **Credential Locker** de chaque utilisateur.
```console
PS C:\Users\[Username]\AppData\Local\Microsoft\[Vault/Credentials]\
```

## NTDS

Il est très courant de rencontrer des environnements réseau où les systèmes Windows sont joints à un domaine Windows. Cette situation est courante car elle permet aux administrateurs de gérer plus facilement tous les systèmes appartenant à leurs organisations respectives (gestion centralisée).

Dans ce cas, les systèmes Windows envoient toutes les demandes de connexion aux contrôleurs de domaine qui appartiennent à la même forêt Active Directory. Chaque contrôleur de domaine héberge un fichier appelé **NTDS.dit** qui est maintenu synchronisé sur tous les contrôleurs de domaine, à l'exception des contrôleurs de domaine en lecture seule. 

**NTDS.dit** est un fichier de base de données qui stocke les données d'Active Directory, y compris mais sans s'y limiter :

- Les comptes d'utilisateurs (hachage du nom d'utilisateur et du mot de passe)
- Les comptes de groupe
- Les comptes d'ordinateurs
- Les objets de politique de groupe

# John The Ripper (JTR)

John the Ripper est un outil de pentesting  utilisé pour vérifier la force des mots de passe et craquer les mots de passe chiffrés (ou hachés) en utilisant des attaques par force brute ou par dictionnaire.

## Méthodes d'attaques
- Attaque par Dictionnaire : à partir d'une liste de mots et en comparant des hash.
- Attaque BruteForce : tentative de toutes les combinaisons possibles.
- Attaque de la table Arc-en-ciel : utilisation d'une table pré-calculée de hashes associées et leurs mdp en correspondants en clair.

# Modes de craquage
## Single Crack Mode
```console
Misoko@htb[/htb]$ john --format=<hash_type> <hash or hash_file>
```
Nous pouvons voir l'avancement avec **john --show**.
Le résultat sera envoyé dans le fichier **~/.john/john.pot)**.

## Mode "Dictionnaire"
```console
Misoko@htb[/htb]$ john --wordlist=<wordlist_file> --rules <hash_file>
```
Nous pouvons spécifier un ensemble de règles ou appliquer les règles de déformations intégrées aux mots de la liste de mots. Ces règles génèrent des mots de passe candidats en utilisant des transformations telles que l'ajout de chiffres, de lettres en majuscules et de caractères spéciaux.

## Mode Incrémental

Le mode incrémentiel est un mode de John avancé, utilisé pour craquer les mots de passe utilisant un jeu de caractères. Il s'agit d'une attaque hybride, ce qui signifie qu'elle tente de faire correspondre le mot de passe en essayant toutes les combinaisons possibles de caractères du jeu de caractères. Ce mode est le plus efficace mais aussi le plus long de tous les modes John.

Ce mode fonctionne mieux lorsque nous savons ce que le mot de passe pourrait être, car il essaiera toutes les combinaisons possibles dans l'ordre, en commençant par la plus courte.
```console
Misoko@htb[/htb]$ john --incremental <hash_file>
```

## Cracker des fichiers avec John
Il existe de nombreux outils qui permettent à JTR de cracker des fichiers.
Ils contiennent souvent un \*2john\* dans leurs noms et sont à chercher au cas par cas.