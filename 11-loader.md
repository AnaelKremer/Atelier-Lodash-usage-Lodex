# Traitements avancés dans un loader

## Fonctionnement d'un loader

```mermaid
graph TD;
  A[Fichier brut (CSV, JSON, XML...)] --> B[Loader (script EZS)];
  B --> C[Objets JavaScript];
  C --> D[Base MongoDB (Lodex)];
```

## Les instructions

## ...

### Définir des fonctions réutilisables

On a déjà vu qu'avec l'instruction `[ENV]` il était possible de stocker un dictionnaire que l'on pouvait réutiliser.  

De la même manière, `[ENV]` peut contenir des **fonctions utilitaires**, que l'on définit une fois et que l'on réutilise ensuite à plusieurs endroits du pipeline.  

Par exemple, on a un jeu de données qui va contenir beaucoup de *booléens* (is openacces, has repository, is retracted etc). Dans Lodex on souhaiterait afficher des "Oui" ou "Non" plutôt que les *booléens*.  

On va d'abord écire notre fonction dans `[ENV]` et la nommer correctement (c'est souvent la partie la plus compliquée).

```js
[env]
path = labelizeBoolean
value = fix((bool)=> bool === true ? "Oui" : bool === false ? "Non" : "Inconnu")
```

On utilise `fix` pour déclarer notre fonction, puis on écrit une simple *ternaire*.  

Il suffit ensuite dans un `[assign]` de déclarer le champ que l'on veut transformer, puis d'appeler la *variable d'environnement* dans un `thru`.

```js
[assign]
path = value
value = get("isRetracted").thru(env("labelizeBoolean"))
// Entrée : true → Sortie : "Oui"
```

Et si notre champ ne contient pas un seul, mais plusieurs *booléens* dans un tableau, il suffit de remplacer `thru` par `map` !

```js
[assign]
path = value
value = get("isCnrs").map(env("labelizeBoolean"))
// Entrée :  [false, true, true, false] → Sortie : ["Non","Oui","Oui","Non"]
```

Une fois cette logique acquise, on peut aller encore plus loin et se lancer dans **la composition de fonctions**.  

Plutôt que d’écrire de longs traitements redondants, on crée de petites briques spécialisées (nettoyer une chaîne, nettoyer un tableau), puis on les enchaîne et les combine pour construire des transformations plus complexes.  

---

A titre d'exemple, on souhaite "normaliser" les champs contenant des chaînes de caractères (titre, résumé...). On souhaite également réaliser ces opérations sur les éléments de certains tableaux comme les mots clés. Et enfin on souhaite que données sous formes de tableaux soient dédoublonnées, triées et vidées de leurs valeurs falsy (null, ""...).  

On construit notre première brique : 

```js
[env]
path = normalizeString
value = fix((str)=> \
  _.toLower( \
    _.deburr( \
      _.trim(str) \
    ) \
  ) \
)
```

> [!NOTE]
> On le sait, utiliser `=>` nous fait passer dans du **JavaScript pur**.  
> Mais pour réaliser un chaînage **Lodash** dans une fonction anonyme il convient de mettre le préfixe `_` avant chaque fonction comme ici.  
> Mais lorsque l'on combine des fonctions, la lecture peut devenir difficile. Dans cet exemple, `trim` est d'abord utilisée, puis `deburr` et enfin `toLower`.
>
> Il existe une autre façon de déclarer un chaînage, qui est sans doute plus lisible :
>
> Il faut placer la valeur à transformer dans un **wrapper Lodash**. Littéralement cela signifie que l'on *emballe* la valeur dans un objet spécial afin de pouvoir la passer dans un pipeline de transformations.  
> Et comme vu [ici](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/01-introduction.md#un-encha%C3%AEnement-de-fonctions-lodash) un pipeline **Lodash** doit commencer par `_.chain` et se conclure par `.value()`.

On peut donc écrire notre première brique comme cela :

```js
[env]
path = normalizeString
value = fix((str)=> \
  _.chain(str) \
    .trim() \
    .deburr() \
    .toLower() \
    .value() \
)
```

Puis la tester sur une chaîne de caractères :

```js
[assign]
path = normalizedTitle
value = get("value.title").thru(env("normalizeString"))
// Entrée :  "Bibliométrie prête à l'emploi avec OpenAlex : retour d'expérience" 
// → Sortie : "bibliometrie prete a l'emploi avec openalex : retour d'experience"
```

On peut ensuite créer notre deuxième brique destinée à traiter des tableaux (que l'on ajoute dans [ENV] en dessous de la première) :

```js
path = cleanArray
value = fix((arr) => \
  _.chain(arr) \
    .compact() \
    .uniq() \
    .sort() \
    .value() \
)
```

On peut enfin utiliser nos deux briques sur un même champ :

```js
path = normalizedAllKeywords
value = get("value.authorsKeywords") \
    .concat(self.value.keywords) \
    .map(env("normalizeString")) \
    .thru(env("cleanArray"))
```

:point_down:

```json
{
  "authorsKeywords": [
    "bibliométrie",
    "openalex",
    "cnrs-inist",
    "retour d’expérience"
  ],
  "keywords": [
    "bibliometrie",
    "OpenAlex",
    "CNRS-INIST",
    "expérience utilisateur",
    "Lodex"
  ],
  "normalizedAllKeywords": [
    "bibliometrie",
    "cnrs-inist",
    "experience utilisateur",
    "lodex",
    "openalex",
    "retour d’experience"
  ]
}

```