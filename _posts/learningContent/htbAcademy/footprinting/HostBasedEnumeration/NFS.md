# NFS (Network File System) : port 111 et 2049

Système de fichiers de réseaux ayant le même but que SMB (version UNIX) : accéder au système de fichiers à travers un réseau comme s'ils étaient locaux. La version 3 (NFSc3) authentifiait l'ordinateur client, avec la version 4, elle authentifie l'utilisateur.

La version 4 est seulement sur le port 2049. NFS est basé sur le protocol **[ONC-RPC/SUN-RPC](https://en.wikipedia.org/wiki/Sun_RPC)** exposé sur le port 111.
**NFS** n'a pas de système d'**authentification** et d'**authorisation**.
**NFS** doit être utiliser seulement sur des réseaux secure pour des problèmes d'authentification via UNIX avec le **UID/GID** (voir doc).

## Configuration 
Le fichier de configuration est **/etc/exports**.
Ce fichier contient une table des systèmes dee fichiers physiques du serveur NFS accessible aux clients. La **NFS Exports Table** montre les options de configurations.

Exemple de partage de la ressource /mnt/nfs dans le fichier /etc/exports, accessible au réseau IP/MASK avec les options sync/no_subtree_check.
`/mnt/nfs IP/mask(sync, no_subtree_check)`

Une fois le service NFS découvert à l'aide de nmap par exemplen, nous pouvons "monter" (dupliquer) dans un nouveau dossier sur notre machine et y naviguer comme si cela était un dossier local?

## Voir les Ressources NFS disponibles

```Misoko@home[/home]$ showmount -e IP ```

## Monter une ressource NFS

`Misoko@home[/home]$ mkdir dossierLocal`
`Misoko@home[/home]$ mount -t nfs IP:/ ./dossierLocal/ -o nolock`
`Misoko@home[/home]$ cd dossierLocal`
`Misoko@home[/home]$ tree .`

## Lister du contenu avec les UIDs et GUIDs
```Misoko@home[/home]$ ls -n mnt/nfs/```

## Démontage d'une ressource
```Misoko@home[/home]$ umount ./dossierLocal```