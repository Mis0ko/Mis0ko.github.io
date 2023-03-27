---
title:  "Services de réseaux : port 5985, 5986, 3389"
category: "Attaques par mot de passe à distance"
tag: "Attaques de mots de passe"
---
De nombreux serveurs sont gérés à distance à travers le réseau.\
Les services les plus utilisés sont **RDP, WinRM et SSH**.

## WinRM (Windows Remote Management) : port 5985, 5986
C'est l'implémentation de Microsoft du protocole **WS-Management.**
Il permet de gérer des systèmes windows en utilisant le protocole **SOAP**.\
Il est désactivé par défaut sur Windows 10.

## CrackMapExec

```console
Misoko@htb[/htb]$ crackmapexec <proto> <target-IP> -u <user or userlist> -p <password or passwordlist>
```
La présence de **(Pwn3d!)** signifie qu'on a accès à un compte utilisateur.

## Evil-WinRM
Un autre outil pour communiquer avec WinRM.
```console
Misoko@htb[/htb]$ evil-winrm -i <target-IP> -u <username> -p <password>
```

## SSH
### Hydra - SSH
```console
Misoko@htb[/htb]$ hydra -L user.list -P password.list ssh://10.129.42.197
```

## RDP (Remote Desktop Protocol) : 3389
C'est un protocole permettant la connexion à distance aux systèmes Windows.

### Hydra - RDP

```console
Misoko@htb[/htb]$ hydra -L user.list -P password.list rdp://10.129.42.197
```

### xFreeRDP
Il existe de nombreux clients pour communiquer par RDP. \
Par exemple xFreeRDP avec la syntaxe suivante :
```console
Misoko@htb[/htb]$ xfreerdp /v:<target-IP> /u:<username> /p:<password>
```

## SMB
Protocole responsable pour le transfert de données entre un client et serveur au sein d'un réseau local.
On peut utiliser hydra pour SMB également.

### Hydra - SMB
```console
Misoko@htb[/htb]$ hydra -L user.list -P password.list smb://10.129.42.197
```

### Metasploit Framework


```console
Misoko@htb[/htb]$ msfconsole -q
msf6 > use auxiliary/scanner/smb/smb_login
msf6 auxiliary(scanner/smb/smb_login) > options 
msf6 auxiliary(scanner/smb/smb_login) > set user_file user.list
msf6 auxiliary(scanner/smb/smb_login) > set pass_file password.list
msf6 auxiliary(scanner/smb/smb_login) > set rhosts 10.129.42.197
msf6 auxiliary(scanner/smb/smb_login) > run
```
### CrackMapExec
Nous pouvons voir les ressources partagées ainsi que leurs privilèges avec CrackMapExec
```console
Misoko@htb[/htb]$ crackmapexec smb 10.129.42.197 -u "user" -p "password" --shares
```

### smbClient

```console
Misoko@htb[/htb]$ smbclient -U user \\\\10.129.42.197\\SHARENAME
```
