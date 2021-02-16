+++
page_template = "article.html"
title = "bypass_branches_conditionnelles"
date = 2021-02-15

draft = false
[taxonomies]
tags = ["binary", "patching", "toolbox"]
+++
En attendant que j'ai plus de temps pour écrire des articles plus copieux, nous allons voir rapidement **la technique de patching d'un binaire**.

Nous allons plus précisement étudier le cas d'une branche conditionnelle et __comment contourner une vérification__.

# La situation initiale

Nous allons étudier un code C très simple, qui permet à un utilisateur autorisé d'executer un binaire root.
```c
#include <stdio.h>
#include <stdlib.h>
void main(void){
    int isAdmin = 0;
    
    if(isAdmin) {
        printf("accessing very secret part\n");
        system("/bin/super_root_bin");
    } else {
        printf("you're a simple human, sorry...\n");
    }
}
``` 

Notons qu'ici, le code est très simpliste, dans un but de démonstration, il est très rare de trouver des structures aussi simples en réalité.

On compile avec gcc notre code et on l'execute, on obtient sans surprise `you're a simple human, sorry...`.

Mais nous allons maintenant tenter de rentrer dans le `if`.

# Exploitation
On allons utiliser deux techniques pour rentrer dans la boucle conditionnelle: on verra comment patcher le binaire avec `Cutter` puis avec une méthode plus brute et radicale avec `gdb`
## Premiere méthode: Patcher le binaire avec Cutter
### Présentation de Cutter
[Cutter](https://cutter.re/) est une interface graphique très sympa pour r2 ([radare2](https://rada.re/n/radare2.html)), un outil très pratique d'exploitation binaire. Dans ce billet, je n'entrerai pas en details dans le fonctionnement et l'utilisation de r2 qui est un outil extremement puissant et polyvalent et qui pourrait faire l'objet d'un article entier tant il est complet.


{{ resize_image(path="articles/toolbox_patching/images/interface_cutter.png", width=600, height=600, op="fit") }}

On ouvre Cutter et on choisit bien l'option `-w` qui permet d'ouvrir le binaire en lecture et en écriture. Ensuite, l'interface principale de Cutter apparaît. Présentons succintement cette interface:
{{ resize_image(path="articles/toolbox_patching/images/main_window_cutter.png", width=800, height=800, op="fit") }}

(1) correspond à l'interface principale de Cutter, par défaut, on atterrit sur l'onglet `dissassembly` qui permet d'avoir un code desassemblé du binaire ouvert, et en bas, plusieurs onglets permettent de naviguer dans les différentes vues.

(2) correspond à l'explorateur de symboles du binaire: il liste tout les symboles et les différentes fonctions, et permet de faire des recherches dessus. On note d'ailleurs la présence de la fonction *main* qui va nous concerner très bientôt.

(3) enfin correspond à la partie navigation de Cutter, et on voit une bar qui montre la répartition des adresses mémoire utilisées par le programme analysé.

### Entrons dans le vif du sujet
Maintenant, il est temps de mettre les main dans le cambouis. Comme je l'ai introduit precedemment, on peut naviguer directement à la fonction `main` en utilisant l'explorateur de symboles.

{{ resize_image(path="articles/toolbox_patching/images/main_function.png", width=600, height=600, op="fit") }}

De l'assembleur... Bon gardons notre calme. Même sans grande connaissance de l'assembleur, il est plutôt aisé de comprendre ces lignes. Les instructions de `0x1149` à `0x114d` constituent le *prologue* de la fonction, et permettent de pousser sur la pile (la stack) les arguments de la fonction main.

Ensuite, on assigne à `var_4h 0`: on reconnaît notre ligne `int isAdmin = 0`. Puis vient la partie la plus intéressante du code: 
```asm
cmp     dword [var_4h], 0 ; if(var_4h == 0)
je      0x1178 ; jump if equal to 0x1178
```
C'est ici qu'on retrouve la gestion de la condition:

On compare d'abord la variable, précedemment allouée, à 0.
Puis l'instruction **je** ou **j**ump **e**qual permet de sauter à l'adresse **0x1178** si la condition est vérifiée. En réalité, sous le capot, `cmp` définit le **Zero flag** en fonction du résultat de la comparaison, mais nous y reviendront dans la deuxième technique.

Regardons ce qui se passe à l'addresse **0x1178**, on peut d'ailleurs voir que Cutter nous indique d'une petite flèche où aller, c'est fort aimable.
```asm
0x00001178      lea     rdi, str.you_re_a_simple_human__sorry... ; 0x2038 ; const char *s
```
Grâce à radare2, on comprend très facilement ce que cette ligne fait: en effet, r2 nous donne `str.you_re_a_simple_human__sorry...`, ce qui veut dire qu'on arrive bien dans notre branchement `else` de notre code.

c'est bizarre non? La comparaison est vrai donc on rentre dans le branchement conditionnel `if` qui correspond au else de notre code: cette différence est probablement liée à une optimisation de *gcc*.

Maintenant qu'on a localisé la portion de code assembleur qui nous interesse, réflechissons à ce qu'on veut modifier pour rentrer dans la condition.  Pour cela, on peut regarder du côté des instructions et des opcodes disponibles en x86 et plus précisement les instructions de sauts conditionnels:

{{ resize_image(path="articles/toolbox_patching/images/extract_spec_x86.png", width=600, height=600, op="fit") }}

On voit dans cette documentation que `JE`, d'opcode `0x74` effectue le saut si `ZF` est à 1. Mais on apprend également qu'il existe l'instruction inverse `JNE`, d'opcode `0x75` qui effectue le saut si `ZF` est à 0.

Intéressant... Si on remplace `JE` par `JNE`, on ne sautera plus (puisque `ZF` sera toujours à 1) et on continuera le flux d'execution. Etudions ce qui suit notre jump:

```asm
0x0000115e      lea     rdi, str.accessing_very_secret_part ; 0x2008 ; const char *s
0x00001165      call    puts       ; sym.imp.puts ; int puts(const char *s)
0x0000116a      lea     rdi, str.bin_super_root_bin ; 0x2023 ; const char *string
0x00001171      call    system     ; sym.imp.system ; int system(const char *string)
```

Plusieurs points très interessants dans ce code: déjà, on retrouve la chaine `accessing_very_secret_part` qui permet de dire qu'on est dans la partie "protégée" du programme. De plus, on remarque un `call system` qui correspond à l'appel à notre fonction privilégiée. Bingo, c'est bien là où on veut aller. Il faut donc patcher (modifier) `JE` et le remplacer par `JNE`. 

Mettons nous au travail. Depuis l'interface de Cutter, on peut editer l'instruction `JE` en `JNE`. Notez, qu'il est également possible de remplacer le `JE` par un `NOP` (NO oPeration) qui permet également de bypass la condition.

{{ resize_image(path="articles/toolbox_patching/images/patch_instr.png", width=600, height=600, op="fit") }}

Ici, on remplace `je 0x1178` par `jne 0x1178` et radare2 va changer tout seul l'opcode (on passe bien de `0x74` à `0x75`). Voilà, le binaire est patché :) . On peut quitter Cutter et vérfier cela avec `objdump`, un autre outil bien pratique à tout reverse engineer qui se respecte. On lance `objdump --disassemble=main ./vuln` qui permet desassembler la fonction main et on observe bien le changement d'opcode:

{{ resize_image(path="articles/toolbox_patching/images/success_patching.png", width=600, height=600, op="fit") }}

Hourra, et si on lance maintenant le binaire, on obtient un magnifique `accessing very secret part` suivi de l'execution de notre programme sensible. Nous avons bien réussi à bypass la vérification \o/.

## Deuxième méthode: utiliser GDB
Nous avons vu une méthode très confortable pour éditer le binaire, mais maintenant, nous allons adopter une méthode plus brute et utile dans un contexte différent.
### Petite partie théorique
Avant de rentrer dans le vif du sujet, nous allons rapidement revenir sur le concept d'**EFLAGS**. J'avais évoqué la notion de `ZF` dans la partie précédente, et je vous avez promis d'y revenir, et bien, c'est le moment! En effet, `ZF` ou Zero Flag fait parti des flags qui consistuent le registre de status dans un processeur x86. Ce registre est un peu particulier et contient des informations qui décrivent l'état du processeur à chaque instant pour chaque programme. Ce registre porte le nom de **FLAGS** et fait 16 bits. Il a été etendue à 32 et 64 bits pour les architectures modernes, avec respectivement les registres **EFLAGS** et **RFLAGS**.  Il y a de nombreux flags dans ce registre, je ne compte pas tous les présenter dans ce post mais on peut par exemple citer `PF` ou Parity Flag qui vaut 1 si le résultat de la derniere opération réalisée par le CPU est paire et 0 sinon ou encore `SF` pour Sign Flag, qui stock l'information du signe du résultat de la dernière operation. Mais `ZF` dans tout ça? Le Zero Flag stock le résultat de la derniere comparaison réalisée avec le CPU, notamment avec l'operation `cmp`. Plus exactement l'instruction `cmp a b` réalise une soustraction entre a et b et met `ZF` à 1 si la différence est nulle (i.e a et b sont identiques), 0 sinon. 
Et les instructions de saut de la famille des `J` (comme JE, JNE, JZ, JNZ) se basent sur ce flag pour décider si un jump doit être fait ou pas. Voyez vous où je veux en venir? En effet, si on peut manipuler la valeur de `ZF`, on peut manipuler le flux d'execution conditionnel d'un programme!

### À l'attaque!
Avec toutes ces bonnes idées en tête, nous allons nous attaquer au binaire déjà étudié en partie 1.

Pour cette partie, nous utiliserons GDB. GDB ou GNU DeBugger est le debugger le plus populaire sous linux, il permet de poser des `breakpoints` dans un programme, d'executer instruction par instruction et de récuperer à chaque étape l'état du CPU. on le lance en muet avec l'option `-q`

{{ resize_image(path="articles/toolbox_patching/images/gdb_1.png", width=600, height=600, op="fit") }}

r est un raccourci de la commande run, qui permet de lancer le binaire, et on obtient notre habituel message qui nous signale que nous ne sommes que de simples humains. ensuite GDB quitte le programme et nous affiche le pid (process ID) du programme ainsi que son code d'arrêt. Plutôt décu? Nous allons maintenant commencer à vraiment utiliser la puissance de gdb: la première étape est de desassembler la fonction `main` afin de savoir où poser un point d'arrêt intéressant.
On va utiliser la commande `set disassembly-flavor intel` pour demander à gdb d'afficher le code assembleur avec la synthaxe intel, qui est plus simple à lire


{{ resize_image(path="articles/toolbox_patching/images/disass_gdb.png", width=600, height=600, op="fit") }}

Vous commencez à être habitués, voici le code assembleur de notre fonction main, qui ressemble beaucoup à celui qu'on avait trouvé avec Cutter. On reconnait l'instruction de comparaison puis le saut conditionnel, et on sait maintenant qu'il va falloir manipuler le fameux registre `FLAGS` à ce moment là. Mais pour pouvoir étudier en détail ce qu'il se passe, on va poser notre breakpoint au début de la fonction main, en utilisant la commande `b *main` pour *b*reak au symbole main (d'où la présence de l'astérisque). On relance avec `r`. Cette fois ci, gdb nous indique un breakpoint et son adresse dans la mémoire: le programme est en pause. Maintenant, on peut inspecter l'état des registres du processeur au moment de la pause, en utilisant la commande `info register` ou `i r`.
{{ resize_image(path="articles/toolbox_patching/images/info_regs.png", width=600, height=600, op="fit") }}

On peut voir la valeur de tout les registres classiques d'un processeur x86-64, comme le pointeur d'instruction rip (le r signifique qu'il pointe des instructions de 64 bits). Mais on voit également `eflags`, le fameux registre d'états! GDB nous indique directement les flags qui sont actifs: `PF` le flag de parité, `ZF` notre flag cible, ainsi que le flag de signe `SF`.
### Un peu de binaire

{{ resize_image(path="articles/toolbox_patching/images/eflags.png", width=600, height=600, op="fit") }}

Il peut être interessant de s'arrêter quelques instant ici, pour mieux comprendre comment marche ce registre si particulier. nous avons vu que `EFLAGS` est un registre sur 16 bits (j'ai dit tout à l'heure que `EFLAGS` était étendue à 32 bits, mais pour des questions de simplicité, on allons ignorer les bits supplementaires). Mais concrètement, à quoi cela ressemble-t-il? Et bien, GDB peut nous aider à le comprendre: en effet, on peut utiliser la commande `print/x $eflags` pour afficher la valeur en he*x*adécimal du registre: 0x246. Pourquoi cette valeur? Pour mieux comprendre, il est préférable de regarder la valeur en binaire de 0x246, ce qui peut être fait avec en remplaçant x par t dans la commande precedente, et on obtient cette fois-ci: Ob1001000110 et tout devient plus clair. Les bits 2, 3, 7, 10 sont mis à 1, en comptant de droite à gauche. et on obtient bien les flags `PF`, `ZF` et `SF` comme prevu! Maintenant que nous savons lire le registre `EFLAGS`, il peut être intéressant de se demander comment manipuler ce registre. Comme beaucoup d'autres opérations bas niveau, il faut se pencher sur opérations bit à bit (ou bitwise operation).

> En logique, une opération bit à bit est un calcul manipulant les données directement au niveau des bits, selon une arithmétique booléenne *- Wikipedia*

l'[article](https://fr.wikipedia.org/wiki/Op%C3%A9ration_bit_%C3%A0_bit) wikipédia donne tout les éléments qui vont nous être utiles ici: décalage à gauche (ou plus couramment left bit shift), ou exclusif et ou (xor et or) vont être nos principaux outils.

### à l'attaque (le retour)
Maintenant que l'on sait où on va, on peut revenir à GDB, et **s**tep **i**instruction (si) pour executer la prochaine instruction puis mettre en pause le programme.On peut regarder ou nous sommes dans le binaire à l'aide de `disass` qui indique par une flèche la ligne où nous sommes arrêtés. nous allons jusqu'à l'instruction de jump.
 
{{ resize_image(path="articles/toolbox_patching/images/stepi_gdb.png", width=600, height=600, op="fit") }}

La petite flèche nous indique que nous sommes arrivés. Si vous avez bien compris jusqu'ici, on doit trouver dans le registre EFLAGS le flag `ZF` à 1, car cmp a bien renvoyé 0. Si on vérifie avec `i r`, tout est bon. Et maintenant, nous allons utiliser des opérations bit à bit ainsi que la commande `set` de gdb pour passer le flag `ZF` à 0. 

{{ resize_image(path="articles/toolbox_patching/images/changed_eflags.png", width=600, height=600, op="fit") }}

Qu'est ce que fait la ligne `set $eflags ^= (1 << 6)`? Et bien, elle applique l'operation xor sur le registre eflags avec la valeur binaire `1 << 6` ou 100000 (`<< n ` signifie un décalage vers la gauche, le sens des chevrons, de `n` bits).
on obtient donc 1 xor 1 = 0, puisque le ZF est déjà à 1 et se trouve en 7 ème position du registre(on commence bien à compter à 0). les autres registres restent inchangés.

Finalement, on obtient bien la partie protégée et on a bien contournée la vérification, victoire!

{{ resize_image(path="articles/toolbox_patching/images/end_gdb.png", width=600, height=600, op="fit") }}

# Conclusion
Dans ce court article, je vous ai presenté quelques techniques de bases pour contourner la vérification d'une condition, mais également quelques outils très utiles lorsqu'on veut manipuler et étudier le comportement de binaire. Je ne suis pas allé très loin dans les détails d'implementation ni dans le fonctionnement exacte du registre EFLAGS, car cela sortait du cadre de cet article. De plus, ce sont des techniques appliquées dans des cas très simples, mais la logique restera similaire si l'on veut contourner une protection logicielle mal implementée, l'objectif restera de contourner les mecanismes de verification. Je pense consacrer un prochain article à l'utilisation plus poussée de Cutter et radare2, et à des techniques de reverse plus avancées.

Stay tuned, to be continued ~