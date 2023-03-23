# IMAP  / POP3 : port 110, 143, 993, 995

**IMAP (Internet Message Access Protocol)** est un protocole permettant l'accès aux mails depuis un serveur mail.
Il permet la gestion de mails directement **depuis le serveur** et supporte les structures de dossier.
C'est un protocole réseaux basé sur le modèle client-serveur et il permet la synchronisation d'un client mail local avec une boîte mail d'un serveur.

**POP3 (Post Office Protocol)** lui permet seulement le listage, récupération et la suppression de mails en tant que fonctions du serveur mail.
Il est donc nécessaire d'utiliser un protocole comme **IMAP** en parallèle de **POP3** pour des fonctionnalités supplémentaires comme l'accès à plusieurs boîtes aux lettre pendant une session, préselection des mails, etc.


Les clients ont une copie de cette structure en local, mais la BDD de tous les mails des clients est centralisée sur un serveur, et les mails persiste jusqu'à leur supprésion déclenché par un client.

L'accès a de nombreuses fonctionnaités nécessitent une session actives mais il est possible d'exécuter des commandes en asynchrone en local et elles seront envoyer une fois la connection réétabli (et authentification réussie).

En principe, SMTP est utilisé pour l'envoie d'emails.
Les emails envoyés finissent dans un dossier IMAP, permettant à tous les clients d'avoir accès à ces emails envoyés. La création de ces dossiers fait partie d'un des avantages de IMAP.

Par défaut, IMAP fonctionne sans chiffrement et transmet les commandes, mails, noms d'utilisateurs, mot de passe, etc en clair. En principe, une couche SSL/TLS est appliqué par dessus pour remédier à cela.

### énumeration nmap
```console
Misoko@home[/home]$ sudo nmap IP -sV -p110,143,993,995 -sC
```
### CURL avec authentification
Si on possède des credentials, on peut lire ou envoyer des emails.
```console
Misoko@home[/home]$ curl -k 'imaps://IP' --user user:p4ssw0rd -v
```
Ensuite il faut utiliser des commandes propres à IMAP.

On peut utiliser openssl ainsi que ncat afin de communiquer avec le serveur IMAP-POP3.
Une fois connecter, on peut utiliser les commandes de IMAP/POP3 pour faire les actions souhaitées.
### Intéraction avec avec le serveur POP3 à travers SSL
```console
Misoko@home[/home]$ openssl s_client -connect IP:pop3s
```

### Intéraction avec avec le serveur POP3 à travers SSL
```console
Misoko@home[/home]$ openssl s_client -connect IP:imaps
```

## POP3 Commandes courantes
| Commandes     | Description                                                      |
|---------------|------------------------------------------------------------------|
| USER username | Identifie l’utilisateur                                          |
| PASS password | authentifie l’utilisateur avec son mot de passe                  |
| STAT          | Demande le nombre d'emails sauvegardés sur le serveur.           |
| LIST          | Demande au serveur le nombre et la taille de tous les e-mails.   |
| RETR id       | Demande au serveur de livrer l'email demandé par ID.             |
| DELE id       | Demande au serveur de supprimer l'email demandé par ID.          |
| CAPA          | Demande au serveur d'afficher les capacités du serveur.          |
| RSET          | Demande au serveur de réinitialiser les informations transmises. |
| QUIT          | Ferme la connexion avec le serveur POP3.                         |
A1 FETCH 1 body[text]
## IMAP Commandes courantes
| Commandes                     | Description                                                                                                               |
|-------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| 1 LOGIN username password     | Le login de l'utilisateur.                                                                                                |
| 1 LIST " " *                   | Liste tous les répertoires.                                                                                               |
| 1 CREATE "INBOX"              | Crée une boîte aux lettres avec un nom spécifié.                                                                          |
| 1 DELETE "INBOX"              | Supprime une boîte aux lettres.                                                                                           |
| 1 RENAME "ToRead" "Important" | Renomme une boîte aux lettres.                                                                                            |
| 1 LSUB " " *                   | Renvoie un sous-ensemble de noms à partir de l'ensemble des noms que l'utilisateur a déclaré comme étant actif ou abonné. |
| 1 SELECT INBOX                | Sélectionne une boîte aux lettres afin de pouvoir accéder aux messages qu'elle contient.                                  |
| 1 UNSELECT INBOX              | Quitter la boîte aux lettres sélectionnée.                                                                                |
| 1 FETCH \<ID> all              | Récupère les données associées à un message dans la boîte aux lettres.                                                    |
| 1 CLOSE                       | Supprime tous les messages dont l'indicateur Supprimé est activé.                                                         |
| 1 LOGOUT                      | Ferme la connexion avec le serveur IMAP.                                                                                  |

Autre commandes IMAP :
https://donsutherland.org/crib/imap