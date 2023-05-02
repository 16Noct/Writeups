# WriteUp - Deviens Champion, sers toi de tout ce que tu as appris.

---

> **Titre du Challenge:** Deviens Champion, sers toi de tout ce que tu as appris.
> 
> 
> **Catégorie:** Stéganographie
> 
> **Difficulté :** Moyenne
> 
> **Points:** 50
> 
> **Auteur:** Lmeaou
> 

Le challenge repose essentiellement sur le bloc de texte suivant : 

```
hey, vous connaissez chat GPT, le site ou on peut discuter avec une IA ?
je lui ai fait écrire ce petit dialogue tout à l'heure, c'est sympa non ? 
Salut, j'ai vu un granivol tout à l'heure dans le jardin. Moi, j'ai vu un abra près
du parc. Ah oui ? J'ai aussi vu un empiflor dans les bois. Cool !
Et moi, j'ai rencontré une Poissirène au bord de l'étang. Waouh, c'est génial ! 
Moi, j'ai croisé un Snubull sur le chemin du retour.
Impressionnant ! J'ai aussi vu un Héliatronc voler au-dessus des arbres. 
C'est vraiment fascinant ! Et toi, tu as vu quelque chose d'intéressant ? 
Oui, j'ai vu une Sabelette courir dans l'herbe. Et moi, j'ai vu deux Tadmorvs 
se pourchasser dans les hautes herbes. C'est rigolo ! J'ai aussi vu un Insolourdo
près de la rivière. Moi, j'ai vu une Nidoran♀ dans les buissons. Ah oui ? 
J'ai vu un Nidorino à l'orée de la forêt. Oui, c'est vraiment cool !
Et toi, tu as vu quoi d'autre ? J'ai vu un autre Insolourdo et un Tadmorv se battre. 
Intéressant ! Moi, j'ai vu un Nidorino qui chassait une Noeunoeuf, il était aidé par une Nidoran♀. Oh oui, c'était fascinant ! Et toi, tu as vu quelque chose d'intéressant ? Oui, j'ai vu un Arbok se faufiler dans les buissons. Ah oui ? Moi, j'ai vu un Aéromite 
voler au-dessus des arbres. C'est génial ! Et toi, tu as vu quelque chose d'intéressant ? Oui, 
j'ai vu une Nidoran♀ se cacher dans les fougères. Ah oui, c'est intéressant ! Et moi, 
j'ai vu un Insolourdo se reposer sous un arbre. Cool ! Et moi, j'ai vu deux Mimitoss sauter dans les hautes herbes. 
Oui, c'était vraiment sympa ! Et toi, tu as vu quoi d'autre ? J'ai vu un Goupix se 
faufiler dans les broussailles. Ah oui, c'est super !
On a vu plein d'espèces intéressantes aujourd'hui.
```

En lisant le texte, on s’aperçoit assez rapidement d’une chose : plusieurs noms de pokemons sont cités. On comprend donc que l’information est dissimulée grâce à ces noms. On extrait donc ces noms, un a un dans un fichier, en conservant l’ordre du texte.

Ce qui donne : 

```c
granivol
abra
empiflor
Poissirène
Snubbull
Héliatronc
Sabelette
Tadmorv
Tadmorv
Insolourdo
Nidoran(f)
Nidorino
Insolourdo
Tadmorv
Nidorino
Noeunoeuf
Nidoran
Arbok
Aéromite
Nidoran
Insolourdo
Mimitoss
Mimitoss
Goupix
```

( Certains sont présents deux fois de suite car ils sont précédé par un déterminant qui indique le nombre. )

Ensuite, on sait que le début du flag commence par “CYBN{”, il faut donc trouver un lien entre le premier pokemon “gravinol” et la lettre C. 

Pour se renseigner sur un pokemon en particulier on peut par exemple utiliser le site  [https://www.pokepedia.fr/](https://www.pokepedia.fr/granivol) qui fonctionne comme une encyclopédie pour les pokemons.

En se rendant sur la page du pokemon concerné, on remarque plusieurs informations qui permettent d’identifier celui-ci

![Untitled](./images/Untitled.png)

On remarque que le code identifiant le pokemon dans la région de Johto ( OAC ) correspond au code ASCII (067) de la lettre C. On peut donc émettre l’hypothèse que l’information est bien cachée selon ce procédé.

Pour confirmer notre hypothèse, on réitère le procédé avec le second pokemon “abra”

![Untitled](./images/Untitled2.png)

Le code 089 correspond à la lettre “Y” dans la table ASCII. L’hypothèse se confirme.

En répétant ce cheminement, on retrouve le flag :

```
granivol C
abra Y
empiflor B
Poissirène N
Snubbull {
Héliatronc g
Sabelette 0
Tadmorv t
Tadmorv t
Insolourdo 4
Nidoran(f) _
Nidorino c
Insolourdo 4
Tadmorv t
Nidorino c
Noeunoeuf h
Nidoran _
Arbok 3
Aéromite m
Nidoran _
Insolourdo 4
Mimitoss l
Mimitoss l
Goupix }

CYBN{g0tt4_c4tch_3m_4ll}
```