# Service Réseau
De nombreux serveurs sont gérés à distance à travers le réseau.
Les services les plus utilisés sont **RDP, WinRM et SSH**.

## WinRM (Windows Remote Management) : port 5985, 5986
C'est l'implémentation de Microsoft du protocol WS-Management.
Il permet de gérer des systèmes windows en utilisant le protocole **SOAP**.
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
C'est un protocol permettant la connection à distance aux systèmes Windows.

### Hydra - RDP

```console
Misoko@htb[/htb]$ hydra -L user.list -P password.list rdp://10.129.42.197
```

### xFreeRDP
Il existe de nombreux clients pour communiquer par RDP. 
On a pas exemple xFreeRDP avec la syntaxe suivante :
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
Nous pouvons voir les ressources partagées ainsi que leur privilèges avec CrackMapExec
```console
Misoko@htb[/htb]$ crackmapexec smb 10.129.42.197 -u "user" -p "password" --shares
```

### smbClient

```console
Misoko@htb[/htb]$ smbclient -U user \\\\10.129.42.197\\SHARENAME
```


# Mutation de mot de passe

hashcat permet de créer des mutations de mots de passe basée sur une liste donnée et des règles implémentées dans un fichier qui ont la forme :
| Fonction | Description                                                        |
|----------|--------------------------------------------------------------------|
| :        | Ne faites rien.                                                    |
| l        | Toutes les lettres minuscules.                                     |
| u        | Mettre toutes les lettres en majuscules.                           |
| c        | Mettre en majuscule la première lettre et en minuscule les autres. |
| sXY      | Remplacer toutes les occurrences de X par Y.                       |
| $!       | Ajouter le caractère d'exclamation à la fin.                       |

Chaque règle est écrite sur une nouvelle ligne et décrit comment les mots doivent être mutés.
### Génération de la liste de mots à partir d'une règle
```console
Misoko@htb[/htb]$ hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```
**Hashcat** et **John** possèdent des listes de règles prédéfinies dont **best64.rule** qui donne de bons résultats.

### Règles Hashcat existantes
```console
Misoko@htb[/htb]$ ls /usr/share/hashcat/rules/
```


# Réutilisation de MDP / MDP par défaut.

Des mots de passes par défaut sont souvent utilisés.
Voir [DefaultCreds-Cheat-Sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet) pour avoir une liste.

Le fait d'attaquer ces services avec les credentials par défaut s'appelle le **Credential Stuffing**.

## Credential Stuffing - Syntaxe de Hydra
```console
Misoko@htb[/htb]$ hydra -C <user_pass.list> <protocol>://<IP>
```
Il existe également des listes pour les routeurs comme [celle-ci](https://www.softwaretestinghelp.com/default-router-username-and-password-list/).



