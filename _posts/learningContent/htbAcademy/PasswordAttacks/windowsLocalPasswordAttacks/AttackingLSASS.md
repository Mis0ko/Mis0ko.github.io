# Attaque de LSASS
LSASS est un service qui joue un rôle central dans la gestion des credentials et les processus d'authentification au sein de Windows.

Lors de la première connexion, LSASS va :

- Mettre en cache les credentials localement en mémoire
- Créer des [jetons d'accès](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-tokens)
- Appliquer les politiques de sécurité
- écrire dans le [journal de sécurité de Windows](https://learn.microsoft.com/en-us/windows/win32/eventlog/event-logging-security).

Nous allons voir des outils et techniques afin de dump la mémoire de LSASS et extraire les informations d'identification d'une cible fonctionnant sous Windows.

## Méthode du gestionnaire des tâches
En ayant accès à une session graphique interactive avec la cible, nous pouvons utiliser le gestionnaire de tâches pour créer un "memory dump".

```console
Ouvrez le Gestionnaire des tâches > Sélectionnez l'onglet Processus > 
Recherchez et faites un click droit sur le processus de l'Autorité 
de sécurité locale (LSA) > Sélectionnez Créer un fichier de vidage (dump file).
```

Un fichier **lsass.DMP** est créé et sauvegardé dans :

```console
C:\Users\loggedonusersdirectory\AppData\Local\Temp
```

C'est le fichier à transférer à notre hôte d'attaque (que l'on peut faire de la même manière que pour l'attaque de SAM).

## Méthode Rundll32.exe & Comsvcs.dll 

Avec la méthode précédente, nous sommes limité à l'utilisation d'une GUI.
Avec cette méthode, nous utilisons la commande [rundll32.exe](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/rundll32) avec une ligne de commande.
Il faut noter que cette méthode est détecté des antivirus modernes.

Avant de laner la commande permettant de créé le fichier dump de LSASS, nous devons connaître le **PID** de **lsass.exe**.

### Trouver le PID de LSASS à partir du cmd
```console
C:\Windows\system32> tasklist /svc
```
### Trouver le PID de LSASS à partir du powershell
```console
PS C:\Windows\system32> Get-Process lsass
```

### Création de lsass.dmp en utilisant powershell
Une fois le PID récupéré, on peut créer le dump.
```console
PS C:\Windows\system32> rundll32 C:\windows\system32\comsvcs.dll, MiniDump PID C:\lsass.dmp full
```
Cette commande exécute **rundll32.exe** qui appelle ***comscs.dll** qui appelle la fonction **MiniDumpWriteDump(MiniDump)** pour dump la mémoire du processus LSASS dans la localisation **C:\lsass.dmp**.

La plupart des AV récent bloque la commande et il faudra trouver un moyen de les bypass.


## Utilisation de Pypykatz pour extraire les Crédentials
[Pypykatz](https://github.com/skelsec/pypykatz) est une version python de Mimikatz, permettant d'obtenir les crédentials sur un système Windows (soit un host d'attaque Windows, soit la machine cible, ce qui n'est pas idéale). On peut donc l'utiliser sous linux.

LSASS stocke les credentials qui ont eu une session active sur les OS Windows. Dumper la mémoire du processus LSASS dans un fichier est équivalant à prendre un "snapchot" de la mémoire à l'instant t.
Si des sessions étaient actives à ce moment là, les credentials seront présent dans le fichier.

### Utilisation de Pypykatz
L'outil permet de parser les credentials dans le dump de la mémoire de LSASS. 
```console
Misoko@htb[/htb]$ pypykatz lsa minidump /home/peter/Documents/lsass.dmp 
```
On retrouve plusieurs sections dans le output :
- **[MSV](https://learn.microsoft.com/en-us/windows/win32/secauthn/msv1-0-authentication-package)**, qui est un paquet d'authentification de Windows auquel LSA fait appel pour valider les tentatives de connexion par rapport à la base de données SAM.
- **WDIGEST**, qui est un ancien protocole d'authentification activé par défaut dans Windows XP - Windows 8 et Windows Server 2003 - Windows Server 2012. LSASS met en cache les informations d'identification utilisées par WDIGEST en texte clair.
- **Kerberos** qui est un protocole d'authentification réseau utilisé par Active Directory dans les environnements de domaine Windows. Les comptes d'utilisateurs du domaine reçoivent des tickets lors de l'authentification avec Active Directory. Ce ticket est utilisé pour permettre à l'utilisateur d'accéder aux ressources partagées du réseau auxquelles il a été autorisé à accéder sans avoir à saisir ses informations d'identification à chaque fois. LSASS met en cache les mots de passe, ekeys, tickets et pins associés à Kerberos. Il est possible de les extraire de la mémoire du processus LSASS et de les utiliser pour accéder à d'autres systèmes joints au même domaine.
- **DPAPI** (Data Protection Application Programming Interface) qui est un ensemble d'API des systèmes d'exploitation Windows utilisées pour chiffrer et déchiffrer les blobs de données DPAPI sur une base par utilisateur pour les fonctionnalités du système d'exploitation Windows et diverses applications tierces. On retrouve parmi les applications qui utilisent **DPAPI** : **IE, Google chrome, outlook, DRC, etc**.

### Cracker un hash NT avec Hashcat
```console
Misoko@htb[/htb]$ sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```
