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

à l'aide de deobfucer [de4js](https://lelinhtinh.github.io/de4js/).
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
In order to generate the invite code, make a POST request to \/api\/v1\/invite\/generate
```

```HTTP
HTTP/1.1 200 OK
Date: Mon, 23 Oct 2023 20:24:42 GMT
Content-Type: application/json
Content-Length: 249

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


curl -v 2million.htb/api --cookie "PHPSESSID=spe0t6mqn3o6m9nftlvak53rdk" | jq

