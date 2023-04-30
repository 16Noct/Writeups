# ENISA Flag Store 1/2

Last edited time: April 27, 2023 12:17 AM
Owner: Anonymous
Tags: Go, Injection SQL, SQLi

> **Titre du Challenge:** ENISA Flag Store 1/2
> 
> 
> **Catégorie:** Web
> 
> **Difficulté : ⭐**
> 

**Énoncé** —

```python
L'ENISA a décidé de mettre en place un nouveau service en ligne disponible à
l'année pour les équipes qui participent à l'ECSC. Ce service permet aux joueurs
des différentes équipes nationales de se créer des comptes individuels en 
utilisant des tokens secrets par pays. Une fois le compte créé, les joueurs
peuvent voir les flags capturés dans différents CTF.

Les données personnelles des utilisateurs (mot de passe et pays) sont protégées 
dans la base de données, seuls les noms d'utilisateurs sont stockés en clair.

Le token pour la Team France est ohnah7bairahPh5oon7naqu1caib8euh.

Pour cette première épreuve, on vous met au défi d'aller voler un flag FCSC{...} 
à l'équipe suisse :-)

https://enisa-flag-store.france-cybersecurity-challenge.fr/
```

**Résolution —**

Tout d’abord, on constate que le site cible se compose de trois pages : 

```python
/ -> Page de login
/signup -> Page de création de code
/flags -> Page d'affichage des flags
```

D’après la consigne, dans le cas ou l’on réussirait à se log comme joueur suisse, c’est sur la page `/flags` qu’on retrouverait le flag. 

**Étape 1 : Analyse du code source**

→ Penchons nous maintenant sur le code source mis à notre disposition

Au cours de notre lecture du code, on relève deux fonctions particulièrement intéressantes pour ce que l’on cherche à faire : 

- **checkToken(country string, token string)** → La fonction appelée pour la vérification du token sur /signup
- **getData(user User) →** La fonction appelée pour la récupération des flags de l’utilisateur sur /flags

Ces deux fonctions comprennent une requête SQL, en l’analysant, on constate deux vulnérabilités dans celles-ci

1. **checkToken**

```
SELECT id FROM country_tokens
WHERE country = SUBSTR($1, 1, 2)
AND token = encode(digest($2, 'sha1'), 'hex')
```

> Ici, pour vérifier que le pays entré par l’utilisateur corresponde bien à celui du token, on vérifie uniquement si les deux premières lettres du pays entré correspondent à un pays dont le token est celui entré.
 
**Problème :** 
            le pays “Fr + n’importe quoi” sera considéré comme correct pour le token 
            `ohnah7bairahPh5oon7naqu1caib8euh`
> 

1. **getData**

```jsx
SELECT ctf, challenge, flag, points
FROM flags WHERE country = '%s';
```

> Ici, pour récupérer les flags d’un pays, on utilise la valeur de la colonne “country” pour l’utilisateur actuellement connecté.
 
**Problème :** 
            une donnée modifiable par l’utilisateur est directement présent dans la requête, sans    aucune vérification, cela présente donc un risque d’Injection SQL.
> 

 

**Étape 2 : Injection SQL**

→ En faisant le lien entre les deux vulnérabilités précédemment trouvées : Notre but va donc maintenant être d’injecter du code SQL dans le champs “country” d’un nouvel utilisateur dans la base de donnée, afin que, lorsque l’on se rende sur `/flags`, on reçoive également les flags suisses

On veut que la requête suivante soit effectuée : ( celle-ci aura pour effet de récupérer tous les flags, quelque soit le pays. )

```jsx
SELECT ctf, challenge, flag, points
FROM flags WHERE country = '' OR 1='1';
```

On en déduit donc notre payload : `' OR 1='1`

**→ On revient sur signup pour se créer un compte, on intercepte la requête avec burp et on injecte notre payload dans le paramètre country**

![Untitled](./img/ENISA%20Flag%20Store%201%202/Untitled.png)

La requête s’est bien exécutée et on est redirigé sur `/`, ce qui signifie que notre compte a bien été crée : 

![Untitled](./img/ENISA%20Flag%20Store%201%202/Untitled%201.png)

→ On se connecte, on se rend sur `/flags` et on récupère le flag du challenge.

![Untitled](./img/ENISA%20Flag%20Store%201%202/Untitled%202.png)

**Flag :** `FCSC{fad3a47d8ded28565aa2f68f6e2dbc37343881ba67fe39c5999a0102c387c34b}`

@Noct

---

**Références —**

[https://portswigger.net/burp/documentation/desktop](https://portswigger.net/burp/documentation/desktop)