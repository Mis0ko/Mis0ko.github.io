---
title:  "Introduction de nmap"
category: "Introduction Nmap"
tag: "Énumeration de réseaux avec Nmap"
---
# Utilisation de NMAP
### Architecture de NMAP
Nmap utilises les techniques de scan suivantes :
- Découverte des hôtes
- Analyse des ports
- Énumération et détection des services
- Détection du système d'exploitation
- Interaction scriptable avec le service cible (Nmap Scripting Engine)

### Syntaxe
```console
Misoko@home$ nmap <scan types> <options> <target>
```

Pour les scans TCP, nmap envoie une requête SYN et en fonction des resultats :
- réception d'un SYN/ACK, le port est ouvert.
- réception d'un RST(reset), le port est fermé.
- timeout : le port est filtered, probablement dû à un firewall.

