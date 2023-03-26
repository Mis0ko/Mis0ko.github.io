---
title:  "Outils de bases"
category: "Les bases du test d'intrusion"
tag: "Introduction"
---
# Shell

| Reverse Shell | initie la connection à un programme en attente (listener") sur la box attaquante |
| Bind Shell | "lie" à un port spécifique sur la machine cible et attend une connection provenant de l'attaquant. |
| Web Shell | execute des commandes de système d'exploitation via des navigateur web (pas interactif ou semi intéractif), Peut être utiliser pour lancer des script php par exemple  |

# Protocoles et ports associés

| Port | Protocol |
| ------ | ----------- |
| 20/21   | FTP |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 80 | HTTP |
| 88 | kerberos |
| 161 | SNMP |
| 389 | LDAP |
| 443 | SSL/TLS HTTPS |
| 445 | SMB |
| 3389 | RDP |

# Netcat

Netcat permet de :
- se connecter à des shells.
- se connecter à des ports en écoute et intéragir avec des services s'exécutant sur ces ports.
- transférer des fichiers entre machines.

Par exemple sur SSH :
```console
misoko@home$ nc 10.10.10.10 22

SSH-2.0-OpenSSH_8.4p1 Debian-3
```
Cette technique (**"Banner Grabbing"**) permet d'identifier des services.

Autre outils : **Socat** (+ d'outils). 

# Tmux (terminal multiplexers)

Commande prefixe (avant chaque combo): CTRL + B

| post prefixe | action |
| ------ | ----------- |
| C  | ouvre un nouveau terminal |
|  n | switch au panneaux n |
| SHIFT + % | split (vertical) le terminal en 2 panneaux |
| " | split (horizontal) le terminal en 2 panneaux  |
| deplacement flèche | change de terminal "splittés" |

[cheatsheet](https://tmuxcheatsheet.com/)

### vim

Une fois un fichier ouvert, on est en mode read-only (lecture):

| Commande | Description |
| ------ | ----------- |
| i | entre en mode édition |
| x | coupe le caractère |
| dw | coupe le mot |
| dd | coupe la ligne |
| yw | copie le mot |
| yy | copie la ligne |
| p | colle |
| : | passe en mode commande |
| :1 | va à la ligne 1 |
| :w | sauvegarde le fichier |
| :q | quitte |
| :q! | quitte sans sauvegarder |
| :wq | sauvegarde et quitte |

On peut multiplier toute commande en ajoutant un chiffre devant.\
Par exemple '4yw' va copier 4 mots au lieu d'un.

[cheatsheet](https://vimsheet.com/)