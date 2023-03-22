---
title:  "utilisation_de_nmap"
category: "Blabla"
tag: "learning_content"
---
# Découverte des hôtes.
Pour les pentest interne ==> Découvrir une vue des systèmes.
Une technique très efficace pour la découverte d'hôte est d'utiliser les requêtes echo d'ICMP.

### Scan d'un sous réseau
```console
Misoko@home$ sudo nmap @IP/MASK -sn -oA tnet | grep for | cut -d" " -f5
```
| Options     | Description |
| ----------- | ----------- |
| -sn         | désactive le scan de port       |
| -oA tnet    | enregistre les resultats dans tous les formats de nmap (fichiers commencent par tnet)      |

Fonctionne seulement si le firewall l'autorise.

### Scan d'une liste d'IP

Avec une liste d'IP stocké dans un fichier de la forme :
```
10.129.2.4
10.129.2.10
10.129.2.11
10.129.2.18
10.129.2.19
10.129.2.20
10.129.2.28
```

on peut performer le même type de scan:
```console
Misoko@home$ sudo nmap -sn -oA tnet -iL IPList.txt | grep for | cut -d" " -f5
```
Le résultat représente les IP actives parmi celles de la liste,
(actives = elles répondent aux requêtes ICMP).

### Scan d'une plusieurs IP
```console
Misoko@home$ sudo nmap -sn -oA tnet @IP1 @IP2 | grep for | cut -d" " -f5
```

### Stratégies de découvertes d'hôtes
On peut utiliser cette [documentation](https://nmap.org/book/host-discovery-strategies.html)
Mais des options à retenir :
| Options     | Description |
| ----------- | ----------- |
| -sn         | désactive le scan de port, et permet de ping par des Echo d'ICMP automatiquement, sinon par défaut c'est des messages ARP     |
| -PE   | Requête d'Echo ICMP |
| --packet-trace | permet de voir le cheminement des packets |
| --reason | affiche la raison spécifique d'un résultat |

Sans **ICMP reply**, on considère que l'hôte n'est pas alive.

# Scan des hôtes et ports 

Durant un scan de ports, le port peut être ouvert, fermé, filtré ou une combinaison des états précédents.
Un port fermé est souvent détecté par un retour RST lors d'une connection TCP.
Le port filtré lui est repéré grâce à un timeout, souvent les firewalls vont jeter les packets
qu'ils considèrent comme étranger.
## Découvertes des ports ouverts TCP
Pour optimiser le temps de scan en plus de se focaliser sur des ports en particulier,
on peut également réduire les types de scans avec des options comme ci-dessous.
On aura une meilleure vue du scan TCP.

| Options     | Description |
| ----------- | ----------- |
| -sS         |  Scan les 1000 premiers ports TCP par défaut (pas besoin de spécifier l'option |
| --top-ports=10)        |  choisis un nombre de port à scanner |
| -F         |  Scan les 100 premiers ports |
| -Pn | désactives les requêtes ICMP |
| -n | désactive la résolution DNS |
| --disable-arp-ping | désactive le scan de ping arp |

## Ports filtrés
Un port filtré peut avoir plusieurs raisons.
Les paquets peuvent être **jetés**(dropped) ou **rejeté**(rejected).
Quand un paquet est rejeté, si nmap ne reçoit pas de réponse de la cible, il va
renvoyer une requête (peut être spécifiés le nombre de relances). 
| Options     | Description |
| ----------- | ----------- |
| --packet-trace | Montre tous les packets envoyés et reçus |

## Découvertes des ports ouverts UDP
Les scans sont beaucoup plus long en raison du fait que UDP est un protocole "stateless" (sans état), et qui ne nécessite pas le handshake en 3 temps.
| Options     | Description |
| ----------- | ----------- |
| -sU | Performe un scan UDP |
| -sV | Performe un scan de services |

# Sauvegarde des résultats
Nmap permet la sauvegarde dans 3 fichiers de format, nmap, gnmap et xml
| Options     | Description |
| ----------- | ----------- |
| -oA cible | sauvegarde les résultat dans tous les formats avec comme base de nom de fichier "cible" |
| -oN file | création du fichier .nmap |
| -oG file | création du fichier .gnmap |
| -oX file | création du fichier .xml |

## Rapport Nmap
Il est possible de faire un rapport visuel avec le fichier xml grâce à l'outil **xsltproc**.

```console
xsltproc target.xml -o target.html
```
## Performance
Quand on doit scanner de grands réseaux, mieux vaut bien choisir la performance des scans.
Par contre cela a aussi des effets sur les scans (on peut louper des ports, etc).
Documentation : https://nmap.org/book/man-performance.html


| Options     | Description |
| ----------- | ----------- |
| --max-rtt-timeout \<time>| timeout |
| ----initial-rtt-timeout \<time>| timeout initial |
| --min-parallelism  \<number>| fréquence |
| --max-retries \<number>| nombre de tentative |
| -T <0-5> | vitesse ( 0-paranoiaque, 5-insane) |

# Bypass des mesures de sécurité

## Evasion de Pare-feu/IDS/IPS 
IDS : intrusion detection system
IPS : intrusion prevention system
L'IDS scanne le réseau pour des potentielles attaques et en fait un rapport. L'IPS complète l'IDS en prenant des mesures défensives spécifiques aux attaques éventuellement détectées.

On peut scanner des réseaux à l'aide d'autres adresses  (-S) avec nmap afin de tester le comportement défensif installé dans un réseau.