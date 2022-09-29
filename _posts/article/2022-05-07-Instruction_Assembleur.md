---
title:  "Instruction Assembleur"
date:   2022-05-07
category: article
---
# Assembleur
Dans cet article, nous pouvons voir les bases de l'assembleur, ou du moins ce que je considère comme étant le plus utile à retenir.
## Registre
Un registre de processeur est l'un des plus petits emplacements de stockage de données du processeur.
On l'utilise principalement pour stocker une instruction, une adresse de stockage ou toute autre donnée (une séquence de bits ou des caractères individuels, par exemple).  
``` 
rbp: Base Pointer. Il pointe sur le bas de la frame courante   
de la pile.  
rsp: Stack Pointer. Il pointe sur le haut de la frame courante  
de la pile.  rip : Instruction Pointer. Il point sur l'instruc-  
tion qui va être exécutée.

On retrouve également d'autres registres.
Ces registres peuvent être utilisés pour toutes actions moins  
spécifiques.  
rax:  
rbx:  
rcx:  
rdx:  
rsi:  
rdi:  
r8:  
r9:  
r10:  
r11:  
r12:  
r13:  
r14:  
r15:
```  
Sous linux (x64), les arguments d'une fonction sont passés par  
des registres. Les premiers args sont passés par ces registres :  
```
rdi:    Premier Argument
rsi:    Second Argument
rdx:    Troisième Argument
rcx:    Quatrième Argument
r8:     Cinquième Argument
r9:     Sixième Argument
```  
Les registres ont des dérivés qui ont différentes tailles.
Prenons le cas de eax.  
En x64, il y a les registres rax, eax, ax et al. Le registre rax  
pointe sur le début des 8 premiers bits. Le registre eax pointe sur  
les quatre octets inférieurs du registre rax. Le registre ax correspond aux deux derniers octets du registre rax. Enfin, le registre 
al est le dernier octet du registre rax.

## Words
Un "Word" (mot) représente juste deux octets de données. Un dword correspond à quatre octets de données. Un qword est constitué de huit octets de données.  

## Instructions
Une instruction est une opération qui va être exécuté par le processeur.  
Nous allons voir les principales ici :  
### mov  
L'instruction mov déplace les données d'un registre à un autre :
```nasm
mov rax, rbx
```
Cette instruction va déplacer les données du registre rbx dans le registre rax.
### Déréférencement
Si vous voyez parfois des crochets comme [], ils sont destinés à la déréférence, qui traite des pointeurs.  
Un pointeur est une valeur qui pointe vers une adresse mémoire particulière.
Déréférencer un pointeur signifie traiter un pointeur comme la valeur vers laquelle il pointe. Par exemple :
```nasm
mov rax, [rdx]
```
déplacera la valeur pointée par rdx dans le registre rax. A l'inverse :
```nasm
mov [rax], rdx
```
déplace la valeur du registre rdx dans la mémoire indiquée par le registre rax. La valeur réelle du registre rax ne change pas.
On garde la même adresse pour rax mais sa valeur pointée aura changée.
### lea
L'instruction lea calcule l'adresse du second opérande, et déplace cette adresse dans le premier. Par exemple :
```nasm
lea rdi, [rcx+0x10]
```
Cela déplacera l'adresse rcx+0x10 dans le registre rdi.
### add
L'instruction add calcule simplement la somme de deux valeurs et les stockent. Par exemple :
```nasm
add eax, ebx
```
calculera eax + ebx et stockera le resultat dans eax.
### sub
Cette instruction fonctionne sur le meme principe que add mais avec la soustraction.

### xor 
Cette instruction effectue l'opération binaire xor sur les deux arguments qui lui sont donnés, et stocke le résultat dans la première opération :
```nasm
xor rax, rcx
```
Cela rendra le registre rax égal à rax ^ rcx.  
Les opérations and et or font essentiellement la même chose, mais avec les opérateurs binaires and ou or.

### push
L'instruction push augmente la pile de 8 octets (pour x64, 4 pour x86), puis "pousse" le contenu d'un registre sur le nouvel espace de la pile. Par exemple :
```nasm
push rax
```
Augmentera la pile de 8 octets, et le contenu du registre rax sera au sommet de la pile.
### pop
L'instruction pop fait sortir les 8 premiers octets (pour x64, 4 pour x86) de la pile et les place dans l'argument. Ensuite, elle réduit la pile. Par exemple
```nasm
pop rax
```
Les 8 premiers octets de la pile se retrouveront dans le registre rax.
### jmp
L'instruction jmp permet de sauter à une adresse d'instruction. Elle est utilisée pour rediriger l'exécution du code. Par exemple
```nasm
jmp 0x602010
```
Cette instruction fera sauter l'exécution du code à 0x602010, et exécutera l'instruction qui s'y trouve.
### call & ret
Cette instruction est similaire à l'instruction jmp. La différence est qu'elle va pousser les valeurs de rbp et rip sur la pile, puis sauter à l'adresse qui lui est donnée. Elle est utilisée pour appeler des fonctions. Une fois la fonction terminée, une instruction ret est appelée qui utilise les valeurs poussées de rbp et rip (les pointeurs de base et d'instruction sauvegardés).
### cmp
L'instruction cmp est similaire à celle de l'instruction sub. Sauf qu'elle ne stocke pas le résultat dans le premier argument. Elle vérifie si le résultat est inférieur à zéro, supérieur à zéro ou égal à zéro. En fonction de la valeur, elle active les drapeaux en conséquence.
Elle est souvent suivit d'une instruction jz, jnz, qui en fonction du flag qu'elle a modifié, influera sur l'exécution du code.
### jnz / jz
Ces instructions jump if not zero et jump if zero (jnz/jz) sont assez similaires à l'instruction jump. La différence est qu'elles n'exécutent le saut qu'en fonction de l'état du drapeau zéro. Pour jz, le saut ne sera effectué que si le drapeau zéro est activé. L'inverse est vrai pour jnz.
