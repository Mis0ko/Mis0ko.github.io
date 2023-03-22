# FTP (File Transfer Protocol) : port 20 / 21
Protocole de Transfert de fichier sur le port 21 (connection) et 20 (transfert de données).
Il existe un mode actif et passif. Le mode passif consiste au serveur d'annoncer son port pour que le client initie la connection, et évite donc les restrictions du firewall côté client.

**Anonymous FTP** est un mode offert par certains serverus qui permet l'upload et le téléchargement sans connection au serveur.

## vsFTPd
Serveur FTP fonctionnant en UDP sur Linux. 
Le fichier de configuration se troue sur **/etc/vsftpd.conf.**

### connection
```Misoko@home[/home]$ ftp @IP```
### Vue d'ensemble du serveur
```ftp> status```
### Listing récursif
```ftp> ls -R```
### Télécharger un fichier
```ftp> get Directory\file.txt```
### Télécharger tous les fichiers 
```wget -m --no-passive ftp://anonymous:anonymous@IP```
### Upload un fichier
```ftp> put FichierAUpload```

## Interaction de service 
```Misoko@home[/home]$ nc -nv @IP 21```
```Misoko@home[/home]$ telnet @IP 21```
Si le serveur FTP s'exécute avec un chiffrage TLS/SSL, on pourra utiliser le client **openssl**.
```Misoko@home[/home]$ openssl s_client -connect @IP:21 -starttls ftp```

