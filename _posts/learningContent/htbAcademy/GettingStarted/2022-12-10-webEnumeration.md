---
title:  "Énumeration Web"
category: "Les bases du test d'intrusion"
tag: "Introduction"
---
### [Goburster](https://github.com/OJ/gobuster)
Outil permettant de bruteforce les DNS, serveurs virtuels (vhost) et répertoires.
```console
Misoko@home[/home]$ gobuster dir -u @IP -w /usr/share/dirb/wordlists/common.txt
```

| Status Code HTTP | Description |
| ------ | ----------- |
| 200 | ressources accédées (succès) |
| 403 | accès interdit à la ressource |
| 301 | On est redirigé |

Liste des codes status en détail [ici](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

### Énumération des sous-domaines DNS

On peut chercher les sous domaines avec une énumération gobuster avec le flag dns :

```console
Misoko@home[/home]$ gobuster dns -d mainDomain -w /usr/share/SecLists/Discovery/DNS/namelist.txt
```

## Astuces d'énumération Web

### Banners grabbing / headers de serveurs web
Les headers peuvent indiquer des informations sur le framework utilisé, options d'authentification, misconfiguration, etc.\
On peut utiliser curl (-I pour header et -L pour redirection)
```console
Misoko@home[/home]$ curl -IL https://www.mywebsite.com
```

### Whatweb
Un autre outil pour de l'énumération web
```console
Misoko@home[/home]$ whatweb --no-errors @IP/MASK
```

### Certificats
On peut trouver des informations intéréssantes dans les certificats SSL/TLS.\
Par exemple si on a accès aux emails de l'entreprise pour faire une campagne de phishing.

### robots.txt
Un fichier pour donner des indications aux robots d'indexation des moteurs de recherches (comme Googlebot) quelle ressource peut ou ne peut pas être indexée.