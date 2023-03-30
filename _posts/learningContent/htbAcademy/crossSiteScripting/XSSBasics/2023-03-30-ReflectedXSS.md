---
title:  "XSS Réfléchie"
category: "Notions de base sur les XSS"
tag: "Cross-Site Scripting (XSS)"
---

Il existe deux types de vulnérabilités de **XSS non persistantes** : **l'XSS réfléchie** qui est traité par le serveur back-end et **l'XSS basé sur le DOM (DOM-based)** qui est entièrement traité côté client et n'atteint jamais le serveur.

Les **XSS non persistantes** sont temporaires et peuvent seulement affectés l'utilisateur ciblé puisqu'elles ne sont pas persistantes après un rafraîchissement de la page.

## Où sont utilisées les XSS réfléchies ?
Les XSS réfléchies apparaîssent lorsqu'une entrée utilisateur atteint le serveur web et qu'il nous la renvoie sans qu'elle soit filtrée.
Il existe de nombreux cas d'utilisations comme :
- les retours de messages d'erreurs.
- les confirmations de messages.
- les zones de recherches.
- ...

Cependant ces messages sont temporaires. Une fois que nous quittons la page, ils disparaîssent.
Puisque les XSS ne sont pas persistantes, on peut se demander comment cibler les victimes.

## Comment sont ciblées les victimes des XSS réfléchies ?

Cela dépend du type de la requête HTTP qui est exécutée. On peut voir avec **l'outil de développement de Firefox** (raccourci **\[CTRL+I]**) un exemple de requête GET.

<img src="/assets/images/htbAcademy/HTTPReq.png" alt="Alt text">

Les requêtes **GET** envoie leurs paramètres et données dans l'url. Pour cibler un utilisateur, il faut donc lui envoyer un url contenant le **payload**.

