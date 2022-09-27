---
title:  "[Cheat sheet] GDB-GEF"
date:   2022-05-07
category: article
---
Cette partie a pour but de décrire le deboggueur gdb et de rassembler les bases pour s'en servir.
Gdb est un débogueur (spécifiquement le débogueur gnu). Gef est un wrapper de gdb, conçu pour nous donner quelques fonctionnalités étendues.
Un débogueur est un logiciel qui nous permet d'effectuer différents types d'analyse d'un processus en cours d'exécution, et de le modifier de différentes manières.
# Installation
Pour l'installer, vous pouvez trouver les instructions sur la page github.
(https://github.com/hugsy/gef).
Si vous êtes sur linux, vous pouvez utiliser la commande suivante :
```
bash -c "$(curl -fsSL https://gef.blah.cat/sh)"
```
# Execution
Pour lancer gdb-gef sur un binaire vous avez plusieurs choix, comme par exemple le lancer directement avec le fichier en argument :
```
gdb ./myfile
```
ou bien lancer gdb et appeler le fichier depuis l'interface cli :
```
gdb
gef➤  file monfichier
Reading symbols from monfichier...
(No debugging symbols found in monfichier)
gef➤  break main
Breakpoint 1 at 0x8048409
gef➤  r
Starting program: /home/misoko/Téléchargements/test 
```
  
Afin d'entrer en mode débogueur, nous pouvons définir des points d'arrêt (breakpoints). Les points d'arrêt sont des endroits du programme où GDB saura arrêter l'exécution pour nous permettre d'examiner le contenu de la pile. Le point d'arrêt le plus courant est celui de la partie "main", que nous pouvons définir avec "break main" ou "b main".
```
gef➤  break main
Breakpoint 1 at 0x8048409
```

# Navigation
Il existe plusieurs façons de naviguer à travers un exécutable, voici les principales proposées par gdb-gef :
```
next : permet de faire parcourir une ligne de code, mais vous fera passer par des appels de fonction tels que puts.
nexti : permet de faire parcourir le programme instruction par instruction, mais ne permet pas d'entrer dans les appels de fonction.
step : permet de faire parcourir une ligne de code, mais vous fera passer par des appels de fonction.
stepi : permet de faire parcourir une instruction à la fois, en passant par les appels de fonction.
```

# Breakpoints

Il existe plusieurs façons de définir des breakpoins.
Lorsque nous connaissons le nom de la fonction, par exemple main, nous pouvons simplement faire :
```
gef➤  break main
```
puis faire un run ou "r" pour lancer l'execution.  
D'autres façons peuvent être de la forme :
```
gef➤  b *main+22
Breakpoint 1 at 0x8044212

gef➤  break * 0x8044213
Breakpoint 2 at 0x8044213
```
La console du débogueur est l'endroit où nous pouvons utiliser le débogueur pour fournir différents types d'analyse et modifier des éléments du binaire. Pour l'instant, continuons à regarder les points d'arrêt. Pour afficher tous les points d'arrêt :
```
gef➤  info breakpoints
Num     Type           Disp Enb Address    What
1       breakpoint     keep y   0x8044212 <main+22>
    breakpoint already hit 1 time
2       breakpoint     keep y   0x8044213 <main+22>
```
Nous pouvons enlever des breakpoints :
```
gef➤  delete 2
```

# Analyse des registres

Nous pouvons voir les valeurs des registres à l'air de commandes dans gdb.
Il existe énormément de façons de regarder les registres, nous completerons cette page en fonction de ce 
