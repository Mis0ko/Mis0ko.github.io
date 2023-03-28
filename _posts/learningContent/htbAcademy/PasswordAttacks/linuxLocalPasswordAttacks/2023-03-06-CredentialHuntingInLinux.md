---
title:  "\"Credentials Hunting\" sur Linux"
category: "Attaque de mots de passe local Linux"
tag: "Attaques de mots de passe"
---

Une manière d'élever ses privilèges une fois que nous avons accès au système et la recherche de credentials au niveau local. C'est ce que nous allons voir ici dans le cas d'une machine linux.

Il existe plusieurs sources pour cela que l'on peut classer dans 4 catégories.

| Fichiers        | Historique              | Mémoire               | Key-Rings                  |
|--------------|----------------------|----------------------|----------------------------|
| Configs      | Logs                 | Cache                | Browser stored credentials |
| Databases    | Command-line History | In-memory Processing |                            |
| Notes        |                      |                      |                            |
| Scripts      |                      |                      |                            |
| Source codes |                      |                      |                            |
| Cronjobs     |                      |                      |                            |
| SSH Keys     |                      |                      |                            |

# Les Fichiers
Sur linux, tout est fichier. Il est donc important de d'inspecter chaque catégorie de fichiers qui pourraient contenir des mots de passes ou credentials à savoir :

| Fichiers de configuration  | Bases de données | Notes    |
|----------------------------|------------------|----------|
| Scripts                    | Cronjobs         | SSH keys |

Les fichiers de configurations sont d'autant plus important sur Linux car ils peuvent expliquer comment les services fonctionnent en plus de donner accès à des credentials.
Leurs extensions sont souvent **.config, .conf, .cnf**

## Fichier de configuration
On peut lancer la commande suivante pour récupérer tous les fichiers de configuration et l'enregistrer dans un fichier texte afin de pouvoir les traiter individuellement après.
```console
cry0l1t3@unixclient:~$ for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```
## Credentials dans des fichiers de configuration
Une autre façon de faire est de scanner directement chaque fichier avec une extension particulière et d'afficher les lignes contenant certains mots clés.\
Dans l'exemple suivant, on cherche les mots clés **user, password et pass** dans les fichiers d'extensions **.cnf**
```console
cry0l1t3@unixclient:~$ for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```

## Bases de données
Même chose pour les bases de données.
```console
cry0l1t3@unixclient:~$ for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```
## Notes
Pour les notes c'est plus compliqué sachant que sur linux, l'extension n'est pas forcément obligatoire. De ce fait, on peut chercher les fichiers .txt et autre extension dans le Desktop, home ou autre.

Dans l'exemple suivant on recherche dans le /home les fichiers ayant l'extension .txt et juste une extension.
```console
cry0l1t3@unixclient:~$ find /home/* -type f -name "*.txt" -o ! -name "*.*"
```
## Les Scripts 
Les Scripts sont souvent utilisés pour lancer des tâches répétitives par les admin systèmes ainsi que les devs. De ce fait, ils contiennent souvent des credentials.
```console
cry0l1t3@unixclient:~$ for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```

## Cronjobs
Les cronjobs sont des exécutions de commandes, programmes et scripts.
Ils sont divisés en 2 parties, la zone système (**/etc/crontab**) et les exécutions dépendantes de l'utilisateur. 

De plus, les crontabs sont divisées en différentes plages horaires (**:etc/cron.daily**,**:etc/cron.monthly**,**:etc/cron.houly**,**:etc/cron.weekly**).
Les scripts et fichiers utilisés par **cron** peuvent être trouvé dans **/etc/cron.d/**

## Clés SSH
On rappelle que SSH fonctionne avec une paire de clés asymétrique.
Le serveur possède une **clé public** permettant de vérifier la signature généré par la **clé privée** du client. Si l'on possède une clé ssh privée, on n'a donc pas besoin de mots de passe pour se connecter par SSH.

Les fichiers des clés privées et publics n'ont pas de nommage particuliers. Cependant, ils ont souvent le même format ce qui nous permet de les retrouver.

## Clé SSH Privée
```console
cry0l1t3@unixclient:~$ grep -rnw "PRIVATE KEY" /home/* 2>/dev/null | grep ":1"
```
## Clé SSH Public
```console
cry0l1t3@unixclient:~$ grep -rnw "ssh-rsa" /home/* 2>/dev/null | grep ":1"
```

# Historique
Tous les fichiers historiques fournissent des informations cruciales. On va s'intéresser ici aux fichiers qui stokent l'historique des commandes utilisateurs ainsi que les logs qui stockent des informations concernant les processus systèmes.

Globalement, cela englobe **.bash_history** ainsi que **.bashrc** et **.bash_profile**.
## Bash History
```console
cry0l1t3@unixclient:~$ tail -n5 /home/*/.bash*
```
# Logs (journaux d'activité)
Une particularité de Linux consiste en son système de log stockées dans des fichiers texte.\
On peut retrouver les erreurs systèmes, détecter les problèmes concernant les services où découvrir ce que fait le système en tâche d'arrière fond.

On peut diviser les logs en 4 catégories :
- Logs d'applications
- Logs d'évènements
- Logs de Services
- Logs système

De nombreux fichiers logs existent et varient en fonction des applications mais on peut en noter une partie ici :

| Fichiers Logs       | Description                                                      |
|---------------------|------------------------------------------------------------------|
| /var/log/messages   | Journaux d'activité du système générique.                        |
| /var/log/syslog     | Journaux génériques d'activité du système.                       |
| /var/log/auth.log   | (Debian) Tous les journaux relatifs à l'authentification.        |
| /var/log/secure     | (RedHat/CentOS) Tous les journaux relatifs à l'authentification. |
| /var/log/boot.log   | Informations sur l'amorçage.                                     |
| /var/log/dmesg      | Informations et journaux relatifs au matériel et aux pilotes.    |
| /var/log/kern.log   | Avertissements, erreurs et journaux relatifs au noyau.           |
| /var/log/faillog    | Tentatives de connexion échouées.                                |
| /var/log/cron       | Informations relatives aux tâches cron.                          |
| /var/log/mail.log   | Tous les journaux relatifs au serveur de messagerie.             |
| /var/log/httpd      | Tous les journaux relatifs à Apache.                             |
| /var/log/mysqld.log | Tous les journaux relatifs au serveur MySQL.                     |

L'analyse des logs les uns après les autres seraient chronophages et non pertinent.
```console
cry0l1t3@unixclient:~$ tail -n5 /home/*/.bash*
```

# Mémoire et Cache
Beaucoup d'applications et de processus fonctionnent avec des credentials nécessaires pour de l'authentification et ils les stockent dans la mémoire où des fichiers afin qu'ils puissent être utilisés.

Cela peut être des credentials requis par le système pour les utilisateurs connectés. Un autre exemple est celui des navigateurs qui lisent les credentials dans des fichiers.

## Memoire - Mimipenguin

[Mimipenguin](https://github.com/huntergregal/mimipenguin) est un outil qui permet la récupération de credentials sous linux.
Il nécessite néanmoins d'avoir les privilèges root.

```console
cry0l1t3@unixclient:~$ sudo python3 mimipenguin.py
```
Un autre outil similaire et même plus puissant est **LaZagne**.

## Mémoire LaZagne

```console
cry0l1t3@unixclient:~$ sudo python3 laZagne.py all
```

## Navigateurs
Les navigateurs stockent les mots de passes enregistrés localement sous forme chiffrée.\
Par exemple, **Mozilla Firefox** stocke les mots de passe dans un dossier caché du point de vue de l'utilisateur et on retrouve également le nom des sites auxquelles ils correspondent, leurs noms, etc.
Concernant **Firefox**, on retrouve les mots de passes dans le fichier **logins.json**, et ils sont facilement déchiffrables.

## Retrouver les credentials Firefox

```console
Misoko@htb[/htb]$ ls -l ~/.mozilla/firefox/ | grep default
```
on récupère 1bplpd86.default-release comme dossier de stockage qui varie en fonction de l'utilisateur.
```console
Misoko@htb[/htb]$ cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .
```
## Déchiffrement des credentials de Firefox
```console
Misoko@htb[/htb]$ python3.9 firefox_decrypt.py
```


