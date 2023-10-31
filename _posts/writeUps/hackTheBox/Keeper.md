---
title:  "HacktheBox WriteUp : Keeper"
category: HackTheBox
tag: writeups
---
### Skills
- Nmap
- gobuster
- Request Tracker (RT)
- CVE-2023-32784
- KeePass
- PuTTY


### Scanner le réseau

On scan le réseau, qui nous permet d'identifier un serveur IIS sur le port 80.
```console
$ nmap -sC -sV 10.10.11.237
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Aero Theme Hub
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

### Identification des endpoints
```console
$ gobuster dir -u http://10.10.11.227:80 --wordlist /usr/share/dirb/wordlists/common.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.11.227:80
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 149]
Progress: 4614 / 4615 (99.98%)
===============================================================
```

On voit qu'il n'y a q'une endpoint (parmi les endpoints "communs").

On ajouter le nom de domaine dans notre /etc/hosts:
```console
$ echo "10.10.11.227 tickets.keeper.htb" | sudo tee -a /etc/hosts
```

### Acces Initial

On tombe sur un portail de connection :
<img src="/assets/images/WriteUps/HackTheBox/Keeper/LoginPage.png" width="600px" height="300px" style="display: block; margin: 0 auto"/>

En faisant une recherche "RT 4.4.4+dfsg-2ubuntu1 default credentials" sur google, on tombe sur [cet article](https://forum.bestpractical.com/t/forgot-admin-password-of-rt/33451/2)
nous indiquant que ls credentials par défault de la technologie en question sont "root:password".

En essayant, on voit qu'on a un accès au service :

<img src="/assets/images/WriteUps/HackTheBox/Keeper/accessRoot.png" width="800px" height="300px" style="display: block; margin: 0 auto"/>

Request Tracker (RT) est un système de ticketing qui
permet à un groupe de personnes de gérer des tâches, problèmes et demandes soumis par une communauté d'utilisateurs.

En explorant le service, on tombe sur une liste d'utilisateur qui nous interpelle dans la section `http://tickets.keeper.htb/rt/Admin/Users/`, notamment l'utilisateur `lnorgaard`.

<img src="/assets/images/WriteUps/HackTheBox/Keeper/usersRT.png" width="600px" height="100px" style="display: block; margin: 0 auto"/>

En cliquant dessus, on tombe sur des informations pertinentes : un mot de passe intial donné (`Welcome2023!`), qui nous permet de nous connecter en ssh et d'obtenir le flag user :

<img src="/assets/images/WriteUps/HackTheBox/Keeper/passwordLnorgaard.png" width="900px" height="600px" style="display: block; margin: 0 auto"/>

## Elevation de privilège

Une fois connecté en ssh avec l'utilisateur lnorgaar, on voit qu'il y a un fichier `RT30000.zip` que l'on dezip.
Ce dernier contient 2 fichiers :
```console
lnorgaard@keeper:~$ ls
KeePassDumpFull.dmp  passcodes.kdbx  RT30000.zip  user.txt
```

KeePass est un gestionnaire de mots de passe open source gratuit. Les mots de passe peuvent être stockés dans une base de données chiffrée, qui peut être déverrouillée avec une clé principale.

En faisant des recherches, on tombe sur une vulnérabilité de `KeePass` avec un article d'explication sur la [CVE-2023-32784](https://sysdig.com/blog/keepass-cve-2023-32784-detection/) avec un [POC](https://github.com/CMEPW/keepass-dump-masterkey).

L'exécution du POC nous donne un mot de passe avec des caractères manquants (lié au fonctionnement de la vulnérabilité) :
<img src="/assets/images/WriteUps/HackTheBox/Keeper/pocExec.png" width="600px" height="150px" style="display: block; margin: 0 auto"/>

En faisant des recherches sur google, on tombe sur le nom d'un fleuve Danois : `Rødgrød med fløde`.

Ceci est surement le mot de passe du gestionnaire de la BDD des mdp.
Pour cela on va utiliser Keepass. 
`Rødgrød med fløde` n'a pas fonctionné comme master key, en essayant `rødgrød med fløde` l'accès nous est donné.
<img src="/assets/images/WriteUps/HackTheBox/Keeper/keepassMasterKey.png" width="600px" height="400px" style="display: block; margin: 0 auto"/>

On voit une note qui inclut un fichier de clé d'utilisateur PuTTY.
<img src="/assets/images/WriteUps/HackTheBox/Keeper/rootSSHKey.png" width="600px" height="400px" style="display: block; margin: 0 auto"/>

Comme le format par défaut de PuTTY est .ppk, copions le contenu de la note dans un nouveau fichier avec l'extension de fichier .ppk.
Utilisons ensuite `puttygen` pour créer un fichier de clé `.pem` à l'aide du fichier `.ppk`.

Ensuite il suffit de se connecter par ssh avec la clé, et d'obtenir le flag root.

```console
$ puttygen key.ppk -O private-openssh -o key.pem
$ ssh -i key.pem root@10.10.11.227
root@keeper:~# id
uid=0(root) gid=0(root) groups=0(root)
```

## Ref

- [Credentials par défault](https://forum.bestpractical.com/t/forgot-admin-password-of-rt/33451/2)
- [CVE-2023-32784](https://sysdig.com/blog/keepass-cve-2023-32784-detection/)
- [POC CVE-2023-32784](https://github.com/CMEPW/keepass-dump-masterkey)