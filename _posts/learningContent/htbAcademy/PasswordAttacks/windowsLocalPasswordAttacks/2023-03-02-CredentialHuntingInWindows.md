# "Cedentials Hunting" sur Windows
Une fois l'accès à une machine à travers une GUI ou CLI, nous pouvons passer à la **"Credentials Hunting"**.
Elle consiste au processus d'effectuer une recherche approfondie des credentials à travers le système de fichiers et divers applications.
On considère ici que l'on a accès à une machine Windows 10.

## Recherche centralisée
Il existe des outils sur windows permettant une recherche centralisée dans la plupart des applications et OS. Un utilisateur peut par exemple avoir documenté les mots de passe quelquepart dans le système ou laisser les credentials par défaut.
On utilisera les outils fournits par Windows pour trouver cela.

Par exemple, il est bon de noter les mots clés suivants et de s'en servir lors de nos recherches.
| Passwords     | Passphrases  | Keys        |
|---------------|--------------|-------------|
| Username      | User account | Creds       |
| Users         | Passkeys     | Passphrases |
| configuration | dbcredential | dbpassword  |
| pwd           | Login        | Credentials |

## Outils de Recherche

### Accès GUI
Avec l'accès à un GUI, on peut essayer de passer par le "**Windows Search**" pour trouver des fichiers avec des mots clés comme ceux cités précédemment.
L'outil permet de chercher dans le système de fichier et les applications contenant le mot clé fournit.

### Lazagne
C'est un outil permettant de récupérer tous les credentials des applications installées sur l'hoste qui auraient stockées les credentials de manière non sécurisé.

On peut récupérer une copie indépendante [ici](https://github.com/AlessandroZ/LaZagne/releases/) ou le [github](https://github.com/AlessandroZ/LaZagne).
On peut utiliser **xfreerdp** pour avoir un client **RDP** et ainsi faire un drag and drop de l'exécutable LaZagne.
Une fois l'exécutable sur place, on ouvre un **cmd ou PowerShell** et on exécute les commandes.
```console
C:\Users\bob\Desktop> start laZagne.exe all -oJ -vv
```
Cette commande exécute **Lazagne** sur tous les modules inclus.

### findstr

On peut utiliser [findstr](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/findstr) pour trouver des motifs (comme ceux du tableau dans le début de l'article) à travers différents types de fichiers.
```console
C:\> findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```

### Considérations supplémentaires
Il existe des centaines d'outils pour trouver les credentials sur des systèmes d'exploitations Windows. L'important est de comprendre quel OS Windows est installé et comment il fonctionne car cela nous aidera dans le choix des outils.

Les approches ne seront pas les mêmes avec un Windows Server qu'avec un Windows Desktop.

D'autres pistes à garder en tête lors de la chasse aux credentials :
- Mots de passe auprès du Group Policy dans le share SYSVOL.
- Mots de passe auprès scripts dans le share SYSVOL.
- Mots de passe auprès des scripts dans les shares IT 
- Mots de passe dans le fichier web.config sur les machines de dev et dans les shares IT.
- unattend.xml
- Mots de passe dans les champs de description de l'utilisateur ou de l'ordinateur AD
- Base de données KeePass --> récupérer les hash, les cracker et obtenir pleins d'accès.
- Trouvaille dans les systèmes des utilisateurs et les shares
- Fichiers comme pass.txt, passwords.docx, passwords.xlsx trouvés sur les systèmes des utilisateurs, shares, Sharepoint

