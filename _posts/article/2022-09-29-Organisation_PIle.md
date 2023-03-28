---
title:  "Organisation Pile / convention d'appels de fonction"
date:   2022-09-29
category: article
---
# Specifications
Il est important de notifier que les informations suivantes sont basés sur l'ABI (Application Binary interface) System-v x86-64.
L'interface binaire d'application System V est un ensemble de spécifications qui détaillent les conventions d'appel, les formats de fichier objet, les formats de fichier exécutable etc.

C'est aujourd'hui l'ABI standard utilisé par les principaux systèmes d'exploitation Unix tels que Linux, les systèmes BSD et bien d'autres. Le format exécutable et lisible (ELF) fait partie de l'ABI System V.  
## Rappel Pile
Avant toute chose il est important de rappeler la structure (volontairement simplifiée) de la mémoire vive d'un processus.
![memorySchema](/assets/images/article/Organisation_Pile/processRam.png){:height="700" width="400" style="display: block; margin: 0 auto"}
On peut voir divers sections :  
**section .text :** elle contient les opcodes (code machine) du programme à exécuter.  
**section .data :** elle contient toutes les données globales initialisées.  
**section .bss :** elle contient toutes les données globales non-initialisées.  

De plus, chaque processus dispose d’une pile (stack) qui contiendra les variables
locales et d’un tas où seront stockés les variables allouées dynamiquement (avec la fonction
malloc() en C, l’opérateur new en C++).   

Encore une fois, tout ceci est simplifié afin de rappeler seulement le nécessaire pour la suite.

Comment on peut le voir sur le schéma de la **Fig1**, la pile croît **vers les adresses basses**, et l’adresse de
base de celle-ci se situe dans les adresses hautes de l’espace utilisateur (user land), c'est-à-dire
0xbfffffff.
(L’espace user étant situé entre 0x00000000 et 0xbfffffff et l’espace kernel entre
0xc0000000 et 0xffffffff.)

C'est à dire que plus on ajoute de valeurs dans la pile, plus ces valeurs seront situées sur des adresses basses.
La pile linux fonctionnant comme une LIFO (Last in, first out), il est plus commun de voir les valeurs ajoutées
sur le haut, ce qui faut qu'on se retrouve avec les adresses hautes en bas et les adresses basses en haut.  
La pile sera donc représenté comme ci-après :
![pileSchema](/assets/images/article/Organisation_Pile/pile.png){:height="900" width="600" style="display: block; margin: 0 auto"}
## Registre
Dans le cas de l'ABI standard, les registres principaux suivants sont utilisés comme tel:

```
rbp: Base Pointer. Il pointe sur le bas de la frame courante   
de la pile.  
rsp: Stack Pointer. Il pointe sur le haut de la frame courante  
de la pile.
rip: Instruction Pointer. Il pointe sur l'instruction qui va être exécutée.
rax: valeur de retour d'une fonction
rdi: 1er argument de fonction
rsi: 2eme argument de fonction
rdx: 3eme argument de fonction
rcx: 4eme argument de fonction
```  
Les registres vont être utilisés de diverses manières : variables locales, paramètres, etc.
Durant les appels de fonctions, il y a de la concurrence pour l'utilisation des registres (entre fonction appelante
et fonction appelée). Pour gérer cette concurrence, nous allons utiliser la pile afin de sauvegarder les registres dans une fonction
afin de pouvoir s'en servir dans une autre, puis les restaurer lorque nous avons fini de nous en servir.  

Il existe deux façons de sauvegarder et restaurer les registres, à l'intérieur de la fonction appelante ou bien de l'appelée.
Certains registres sont spécifiques à une méthode particulières, ils sont désignés par "callers-saved" ou "callee-saved".

### Les registres "caller-saved"

Les registres "callers-saved" sont sauvegardés sur la pile dans la fonction appelante une seconde fonction (à "l'extérieur" de la fonction appelée),
cela se traduit par une instruction push.  
La restauration se fait après le call de la fonction appelée qui se traduit par une instruction pull.
Voici un example du comportement de sauvegarde de registres "callers-saved" :
```nasm
push r10  ; Push des registres caller-saved en utilisation
call func
pop r10 ; restoration après return
```
Avantage : Si les fonctions appelées (func dans l'exemple) est autorisée à modifier les registres en question (r10),
il est plus facile d'écrire les fonctions sans avoir à s'occuper de la sauvegarde des dits registres.

### Les registres "callee-saved"

Les registres "callee-saved" sont sauvegardés sur la pile à l'intérieur de la fonction appelée.
La restauration se fait à la fin de la fonction avant le ret.
Voici un example du comportement de sauvegarde de registres "callee-saved" :
```nasm
func:
    push ebp  ; Push des registres callee-saved en utilisation  
    ...  
    pop ebp ; restoration avant return
    ret
```
Avantage : Si les fonctions appelées (func dans l'exemple) doit préserver les registres en question (ebp),
il est plus facile d'appeler les fonctions sans avoir à s'occuper de la sauvegarde des dits registres dans la fonction appelante.

## Push et Pop
Il est important de rappeler l'impact de push et pop sur la pile.
### Push
L'instruction push équivaut à mettre une valeur sur la pile et d'ajuster la valeur de rsp pour que celui-ci pointe
sur le haut de la pile (en décrémentant la valeur de rsp).
![pushSchema](/assets/images/article/Organisation_Pile/push.png){:height="700" width="1100" style="display: block; margin: 0 auto"}
### Pop
L'instruction pop équivaut à retirer une valeur de la pile en la stockant dans un registre, et d'ajuster la valeur de rsp pour que celui-ci pointe
sur le haut de la pile (en incrémentant la valeur de rsp).
![popSchema](/assets/images/article/Organisation_Pile/pop.png){:height="700" width="1100" style="display: block; margin: 0 auto"}
### Remarques
1. La pile grandit à l'envers donc nous commençons "à la fin" de la mémoire et on empile des "stack frames" (voir plus bas).
Si nous voulons regarder à x octets en bas de la pile, il faut regarder à rsp + x.
2. Si nous voulons modifier la pile et ne pas s'occuper des valeurs à enlever par exemple (avec pop), nous pouvons simplement
faire affecter à rsp la valeur rsp + x octets pour libérer les x premiers octets.
3. **!!! Déjà évoqué juste avant : il faut se rappeler que push et pop édite la rsp à chaque opérations de la taille des opérandes (opérande de taille minimum 1 word) !!!**


# Fonction
Ici nous allons voir le fonctionnement des fonctions en lien avec la pile. 
Pour commencer, il faut rappeler deux instructions particulières. 
## call & ret
# call 
Basiquement faire un **call func** consiste faire à pousser sur la pile le rip de l'instruction suivante (**push rip**) suivi d'un **jump func**.  
Cependant, la pile réagit à ce procédé, et donc le pointeur de pile (rsp) aussi.  
rsp passe de 0xbffff7e4 à 0xbffff7dc car on a décrémenté 8 octets de rsp en raison du **push rip**.

&rarr; **Le but de ce push rip est que la pile permet de garder une trace où retourner après la fin de l'execution de la fonction func.**  
On peut voir dans le schéma suivant le changement des registres exposés durant le procédé de **call func** 
![callSchema](/assets/images/article/Organisation_Pile/call.png){:height="700" width="1100" style="display: block; margin: 0 auto"}
## ret 
Faire un **ret** consiste faire à faire un **pop rip** suivi d'un **jump rip**.   
Cependant, la pile réagit à ce procédé, et donc le pointeur de pile (rsp) aussi.  
rsp passe de 0xbffff7dc à 0xbffff7e4 car on a incrémenté 8 octets de rsp en raison du **pop rip**.

&rarr; **Puisque call a stocké préalablement l'instruction de suite du "call func", il suffit de pop cette valeur pour reprendre la suite d'execution du programme.**
On peut voir dans le schéma suivant le changement des registres exposés durant le procédé de **ret**.
![retSchema](/assets/images/article/Organisation_Pile/ret.png){:height="700" width="1100" style="display: block; margin: 0 auto"}



## Passage des paramètres
Traditionnellement, le passage des paramètres se faisait avec des **push** sur la pile dans l'ordre inverse auxquelles ils sont passés dans la fonction (voir exemple). Cependant, une convention a vu le jour dans laquelle nous utilisons des registres à usage specifique pour les paramètres des fonctions : 
- Les 6 premiers paramètres non flottants sont stockés dans les registres **rdi, rsi, rdx, rcx, r8 et r9**.
- Les 8 premiers paramètres flottants sont stockés dans **xmm 0&rarr; xmm7**.
- Le reste : dans la pile dans l'ordre inverse.
- Pour les paramètres > 64bits : on les passe sur la pile ou en mémoire en tant qu'@.

### exemple
`int f(long x, float y, char* z)`

Les registres pour stocker les paramètres seront les suivants :
```nasm
rdi  : x (signed qword)
xmm0 : y
rsi  : z (@, qword)
eax  : return value (dword)
```  
### Remarque
Les paramètres mis sur la pile sont en dessous de **rip**. Si on push des paramètres, il faut ajouter
la taille de ces paramètres à la pile pour maintenir l'alignement de 16x+8 (avant un call).
&rarr; Sinon segfault &rarr; penser à décrémenter.

## Retour des fonctions avec paramètres sur la pile
Rappel : **ret = pop rip + jump rip**
Si nous avons utilisé la pile pour stocker les paramètres de la fonction, il faut s'assurer que le haut de la pile 
soit égal à rip au moment de l'exécution du ret.
Sinon on doit ajuster la valeur de la pile. Si nous avions décrémenté la pile pour le passage des paramètres,
on peut alors la réincrémenté si rip est pas au dessus de la pile.

&rarr; Quand **ret** est exécuté, les 8 premiers octets vont être traité comme une @ de 64bits et **jump** dessus. Si ce n'est pas rip, il en résultera surement un segfault.

