# Passwd, Shadow & Opasswd

Les distributions linux utilisent divers mécanismes d'authentification. Un des plus connu est **PAM (Pluggable Authentification Modules)**.
Les modules utilisés pour ce mécanisme sont **pam_unix.so** et **pam_unix2.so**, et sont localisés dans **/usr/lib/x86_x64-linux-gnu/security/** pour les distributions Linux.
Ces modules gèrent les informations utilisateurs, d'authentification, de sessions ainsi que les mots de pases actuels et anciens.
Lorsqu'on appelle **passwd**, **PAM** est appelé et prend les mesures appropriés.

Les fichiers lu, gérés et mis à jours sont **/etc/passwd** ainsi que **/etc:shadow**.

## Fichier Passwd
Le fichier **/etc/passwd** contient des informations sur chaque utilisateurs du système.
Il peut être lu par tous les utilisateurs et services.
Chaque ligne du fichier contient è champs séparés par :
| cry0l1t3   | : | x             | : | 1000 | : | 1000 | : | cry0l1t3,,,        | : | /home/cry0l1t3 | : | /bin/bash |
|------------|---|---------------|---|------|---|------|---|--------------------|---|----------------|---|-----------|
| Login name |   | Password info |   | UID  |   | GUID |   | Full name/comments |   | Home directory |   | Shell     |

Les anciens systèmes stockaient les hashes de mots de passe dans la colonne **Password info**. 
De nos jours, on voit plutôt un x dans la case, ce qui indique que le mot de passe est stocké sous forme chiffré dans le fichier **/etc/shadow**.
Cependant, cette valeur peut aussi indiqué que le fichier est modifiable, ce qui permettrait d'effacer le champ pour l'utilisateur root, rendant le champ **Password Info** vide.

### Changement de /etc/passwd - Avant
```console
root:x:0:0:root:/root:/bin/bash
```
### Changement de /etc/passwd - Après
```console
root::0:0:root:/root:/bin/bash
```
## Shadow File

Le fichier Shadow est uniquement responsable des mots de passe et de leur gestion. Son format ressemble à celui du **/etc/passwd**.
Il contient les informations relatives aux mots de passes des utilisateurs créés.
Ainsi, si un utilisateur du **/etc/passwd** n'a pas d'entrée dans le fichier **/etc/shadow**, il est considéré comme invalide.
Le fichier **/etc/shadow** est également lisible seulement par les utilisateurs 
ayant les droits d'administrateurs.

### Format du Shadow
| cry0l1t3 | : | \$6\$wBRzy\$...SNIP...x9cDWUxW1 | : | 18937          | : | 0           | : | 99999       | : | 7              | :                 | :               | :      |
|----------|---|------------------------------|---|----------------|---|-------------|---|-------------|---|----------------|-------------------|-----------------|--------|
| Username |   | Encrypted password           |   | Last PW change |   | Min. PW age |   | Max. PW age |   | Warning period | Inactivity period | Expiration date | Unused |


### Fichier Shadow

```console
[cry0l1t3@parrot]─[~]$ sudo cat /etc/shadow

root:*:18747:0:99999:7:::
sys:!:18747:0:99999:7:::
...SNIP...
cry0l1t3:$6$wBRzy$...SNIP...x9cDWUxW1:18937:0:99999:7:::
```

Si le mot de passe contient **!** ou **\***, l'utilisateur ne peut pas se connecter avec un mot de passe Unix.
Cependant, d'autres méthodes d'authentification, tel que Kerberos ou basé sur des clés, peut être utilisé.
Il en va de même concernant les **mots de passe chiffrés.**
Également, le mot de passe a un format particulier :
- **\$type\$sel\$hash**

On peut retrouver les types d'algorithmes pour chiffrés :
### Algorithm Types

- \$1\$ – MD5
- \$2a\$ – Blowfish
- \$2y\$ – Eksblowfish
- \$5\$ – SHA-256
- \$6\$ – SHA-512

Par défault, on utilise le SHA-512 sur les dernières distributions de linux.

## Opasswd

Le PAM (**pam_unix.so)**) peut aussi empêcher la réutilisation des anciens mots de passe. Le fichier où sont stockés les anciens mots de passe est **/etc/security/opasswd**. Les droits de root/admin sont requis pour lire le fichier par défault.


## Cracker les Credentials Linux 

### Unshadow
Unshadow permet de combiner passwd et shadow pour s'en servir dans d'autres logiciels après.
```console
Misoko@htb[/htb]$ sudo cp /etc/passwd /tmp/passwd.bak 
Misoko@htb[/htb]$ sudo cp /etc/shadow /tmp/shadow.bak 
Misoko@htb[/htb]$ unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```
### Hashcat - Cracker des hashes Unshadow

```console
Misoko@htb[/htb]$ hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

### Cracker des Hashes MD5
```console
Misoko@htb[/htb]$ hashcat -m 500 -a 0 md5-hashes.list rockyou.txt
```

















