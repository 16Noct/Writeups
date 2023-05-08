# Beat me!

Last edited time: May 8, 2023 2:50 PM
Owner: Anonymous
Tags: Client-Side, Javascript, Obfuscation, Web
Verification: Verified

> **Titre du Challenge:** Beat me!
> 
> 
> **Catégorie:** Web
> 
> **Difficulté :** Medium
> 
> **Auteur :** Eteck#3426 
> 

### — Énoncé

```c
A pro player challenge you to a new game. 
He spent a huge amount of time on it, and did an extremely good score.
Your goal is to beat him by any way
```

### — Résolution

1. **Analyse du fonctionnement de l’application**

En arrivant sur le site, on tombe sur l’écran d’accueil du jeu, et nous sommes invités à entrer un pseudonyme, puis à lancer le jeu. Le jeu s’arrête lorsque nos points de vie tombent à 0. Observons ce qu’il se passe lorsque cela arrive : 

Dès lors que nos points de vie tombent à 0, on constate que plusieurs requêtes sont effectuées, on relève une requête `POST` sur `/scores` avec le corps suivant : 

`{"score":"0","pseudo":"test","signature":-640686249}`

![Untitled](./img/Beat%20me!/Untitled.png)

**→ On comprend alors que cette requête a pour but de rentrer notre score dans le tableau des scores.** 

![Untitled](./img/Beat%20me!/Untitled%201.png)

**→ On peut alors essayer de renvoyer cette requête avec en passant un score différent, supérieur à celui du joueur “Eteck”, afin de valider le challenge :**

Testons avec le corps suivant : `{"score":"1337421","pseudo":"test","signature":-640686249}`

Ce à quoi le serveur répond par : 

```c
HTTP/1.1 401 Unauthorized
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 27
ETag: W/"1b-5VqlTBvPmQpTk4eooG8mRLIm3c4"
Date: Mon, 08 May 2023 11:24:08 GMT
Connection: keep-alive
Keep-Alive: timeout=5

{"msg":"Invalid signature"}
```

**→ Le serveur n’a pas mit le tableau des scores à jour puisque la signature ne correspondrait pas au score entré, il faudra maintenant trouver un moyen de générer une signature correcte pour le score que l’on souhaiterait rentrer. Pour ce faire, analysons le code.**

1. **Analyse du code** 

Grâce à l’outil “Réseau” de notre navigateur, on peut retrouver le fichier initiateur de la requête XHR 

![Untitled](./img/Beat%20me!/Untitled%202.png)

**→ Le fichier qui nous intéresse ici est `main[...].js`**

Lorsque l’on clique dessus, cela nous ramène vers la **pile d’éxecution** des fonctions du fichier, qui nous permet de voir à quel endroit du code sont appelés les différentes fonctions, ainsi que l’ordre dans lequel elles sont appellées. En l’analysant, deux choses sautent aux yeux : 

![Untitled](./img/Beat%20me!/Untitled%203.png)

l’appel de la fonction “ajax” suivi de la fonction “send”, qui correspondent à l’initiation de la requête XHR. Regardons maintenant les fonctions appellées juste avant : 

![Untitled](./img/Beat%20me!/Untitled%204.png)

en cliquant à droite, cela nous ramène directement sur l’endroit du code ou les fonctions sont appellées. Prenons la fonction appellée juste avant “ajax” → 

![Untitled](./img/Beat%20me!/Untitled%205.png)

Tout de suite, on constate que le code que nous avons devant les yeux s’apparente au corp ( en rouge ) et à l’entête ( en vert ) de la requête d’insertion des scores vue précédemment.

Posons un **breakpoint** sur la ligne qui nous intéresse :

![Untitled](./img/Beat%20me!/Untitled%206.png)

Relançons le jeu et observons ce qu’il se passe: Le site se pause à notre breakpoint on observe cela :

![Untitled](./img/Beat%20me!/Untitled%207.png)

Le score est passé en paramètre à une fonction **`_0x3f306f`** chargée de générer la signature. 

Maintenant, plusieurs solutions s’offrent à nous, nous pouvons éventuellement comprendre le fonctionnement de la fonction **`_0x3f306f`** en analysant le code afin de recalculer nous même la signature, malheureusement celui-ci est obfusqué et difficle à comprendre. Je choisis alors la facilité : puisque le jeu est client-sided, nous pouvons controler l’intégralité des variables présentes dans le code. **Notre solution consistera donc en la modification de la variable passé en paramètre à  `_0x3f306f` en passant un score plus grand que `1337420`, afin que l’appel à cette fonction nous génère une signature valide pour notre score.**

1. **Résolution** 

Pour modifier le code javascript que nous reçevons, j’utilise burp afin de l’intercepter et de le modifier. Configurons d’abord Burp afin qu’il capture également les fichiers .js → on se rend alors dans `Proxy > Proxy Settings > Request Interception Rules` et éditer la première ligne :

![Untitled](./img/Beat%20me!/Untitled%208.png)

on supprime l’extension js de la liste : 

![Untitled](./img/Beat%20me!/Untitled%209.png)

**→ On se rend maintenant sur le site, et on intercèpte le fichier `main[…].js`**

![Untitled](./img/Beat%20me!/Untitled%2010.png)

**→ On va intercepter la réponse à la requête, afin de pouvoir modifier le code reçu par la suite.** 

![Untitled](./img/Beat%20me!/Untitled%2011.png)

Après avoir Forward la requête, on reçoit la réponse et on cherche dans celle-ci l’appel à la fonction **`_0x3f306f`** dont on souhaite modifier le paramètre

![Untitled](./img/Beat%20me!/Untitled%2012.png)

On modifie alors le paramètre en rentrant directement une valeur supérieure à 1337420 : 

![Untitled](./img/Beat%20me!/Untitled%2013.png)

→ Puis on forward et on lance le jeu. On joue jusqu’à atteindre la requête de mise à jour des scores, on l’intercepte et on remarque que le corps de notre requête, toujours pour un score de 0, a changé : `{score: "0", pseudo: "test", signature: -1691900128}` Vous l’aurez donc compris, nous venons de récupérer la signature pour un score de **`999999999`**

Il ne nous reste plus qu’à renvoyer la requête avec le corps suivant : `{score: "999999999", pseudo: "test", signature: -1691900128}`

```c
POST /scores HTTP/1.1
Host: 13.37.17.31:54145
Content-Length: 53
Accept: */*
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.5615.138 Safari/537.36
Content-Type: application/json
Origin: http://13.37.17.31:54145
Referer: http://13.37.17.31:54145/
Accept-Encoding: gzip, deflate
Accept-Language: fr-FR,fr;q=0.9,en-US;q=0.8,en;q=0.7
Connection: close

{"score":"999999999","pseudo":"test","signature":-1691900128}

```

On récupère le flag dans la réponse : 

```c
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 111
ETag: W/"6f-FwGjRP8p8AUqN90e0TtdvcnzVmg"
Date: Mon, 08 May 2023 12:45:39 GMT
Connection: close

{"msg":"Score added to the leaderboard","pseudo":"PWNME{ChE4t_oN_cLI3N7_G4m3_Is_Not_3aS1}","score":"999999999"}
```

Flag : `PWNME{ChE4t_oN_cLI3N7_G4m3_Is_Not_3aS1}`

@Noct

---

### — Références

[https://developer.mozilla.org/fr/docs/Glossary/Call_stack](https://developer.mozilla.org/fr/docs/Glossary/Call_stack)

[https://portswigger.net/burp/documentation/desktop](https://portswigger.net/burp/documentation/desktop)