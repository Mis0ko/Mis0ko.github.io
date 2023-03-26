---
title:  "Scan de Service"
category: "Les bases du test d'intrusion"
tag: "Introduction"
---
### Nmap
Nmap est un outil de scans de ports.
- par défaut, il scan les 1000 ports les plus communs, en mode TCP.
- Si le STATE d'un service est **Filtered**, cela signifique que le service est disponible que pour certaines @IP **(firewall config)**
- nmap nous fournit les noms de services en fonction de leurs ports par défaut (si le port ne correspond pas à son protocole par défaut, on ne le verra pas avant d'intéragir avec le service).
- **-sC** pour + de détails, **-sV** pour un scan de version + OS , **-p-** scan tous les ports.

### Nmap script
On peut améliorer les scans de nmap en utilisant des scripts.
```console
Misoko@home[/home]$ nmap --script <script name> -p<port> <host>
```
### Banner grabbing
Les services vont donner une information pour s'identifier une fois une connexion établie.\
Avec Nmap, on peut faire cette technique.
```console
Misoko@home[/home]$ nmap -sV --script=banner <target>
Misoko@home[/home]$ nc -nv @IP Port
Misoko@home[/home]$ nmap -sV --script=banner -p21 10.10.10.0/24
```

### FTP

```console
Misoko@home[/home]$ ftp -p @IP
```
Commande disponible : ls, cd, get, etc.

### SMB
Protocole permettant le partage de documents sur windows.\
Script sur nmap : **smb-os-discovery.nse**\
RCE possible comme **EthernalBlue**

Un outil qui peut énumérer et intéragir avec les dossiers partagés de SMB : **smbclient**.
- -L pour spécifier qu'on veut récupérer une liste des dossiers partagés disponible de la machine à distance.
- -N supprime l'invité de mot de passe.
```console
Misoko@home[/home]$ smbclient -N -L \\\\@IP
Misoko@home[/home]$ smbclient -N -L \\\\@IP\\users
```

### SNMP
**SNMP Community strings** permet d'avoir accès à des infos et stats d'un router ou appareil(pour avoir son accès). Elles sont souvent laissé par défaut.

Avec SNMP version 1 et 2, l'accès est controlé en utilisant une string claire (de communauté), et si on connait le nom, on y a accès.
Dans SNMP version 3, le chiffrement et l'authentification ont été ajouté.