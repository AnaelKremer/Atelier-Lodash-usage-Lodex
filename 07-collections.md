# Manipuler des collections

Dans Lodex on peut utiliser des fonctions dédiées aux **collections** sur :
- Des **tableaux** (`arrays`),
- Des **objets** (`objects`),
- Des **chaînes de caractères** (`strings`).

Les fonctions Lodash de cette catégorie permettent d’itérer, filtrer, transformer ou analyser ces collections, quelle que soit leur forme.


> [!NOTE]
> Vous pouvez tester chaque *exemple* dans [EZS Playground](http://ezs-playground.daf.intra.inist.fr/).  
> Utilisez l'input suivant :  
> ```json
> [
>   {
>     "value": {
>       "title": "Istex: A Database of Twenty Million Scientific Papers with a Mining Tool Which Uses Named Entities",
>       "year": 2019,
>       "source": "Information",
>       "publisher": "MDPI",
>       "authors": [
>         {
>           "fullname": "Denis Maurel",
>           "rnsr": ["201220254T"],
>           "affiliations": [
>             {
>               "address": "Laboratoire d’Informatique Fondamentale et Appliquée de Tours LIFAT, Bases de données et traitement des langues naturelles BDTLN, FR"
>             }
>           ]
>         },
>         {
>           "fullname": "Enza Morale",
>           "rnsr": ["198822446E"],
>           "affiliations": [
>             {
>               "address": "UAR76 / UPS76, Centre National de la Recherche Scientifique CNRS, Institut de l’information scientifique et technique INIST, 2, Allée du Parc de Brabois, Rue Jean Zay, 54500 Vandœuvre-lès-Nancy, FR"
>             }
>           ]
>         },
>         {
>           "fullname": "Nicolas Thouvenin",
>           "rnsr": ["198822446E"],
>           "affiliations": [
>             {
>               "address": "UAR76 / UPS76, Centre National de la Recherche Scientifique CNRS, Institut de l’information scientifique et technique INIST, 2, Allée du Parc de Brabois, Rue Jean Zay, 54500 Vandœuvre-lès-Nancy, FR"
>             }
>           ]
>         },
>         {
>           "fullname": "Patrice Ringot",
>           "rnsr": ["198822446E"],
>           "affiliations": [
>             {
>               "address": "UAR76 / UPS76, Centre National de la Recherche Scientifique CNRS, Institut de l’information scientifique et technique INIST, 2, Allée du Parc de Brabois, Rue Jean Zay, 54500 Vandœuvre-lès-Nancy, FR"
>             }
>           ]
>         },
>         {
>           "fullname": "Angel Turri",
>           "rnsr": ["198822446E"],
>           "affiliations": [
>             {
>               "address": "UAR76 / UPS76, Centre National de la Recherche Scientifique CNRS, Institut de l’information scientifique et technique INIST, 2, Allée du Parc de Brabois, Rue Jean Zay, 54500 Vandœuvre-lès-Nancy, FR"
>             }
>           ]
>         }
>       ]
>     }
>   }
> ]
> ```
>
> puis collez ce script que vous adapterez.
> 
> ```js
> [use]
> plugin = basics
>
> [JSONParse]
> separator = *
>
> [replace]
> path = sortie
> value = get("value")
>
> [dump]
> indent = true
> ```

## map

Renvoie un tableau de valeurs en parcourant chaque élément de la collection.  
Cela permet donc d'extraire des propriétés d'une collection :

```js
value = get("value.authors").map("fullname")
// → Sortie : ["Denis Maurel","Enza Morale","Nicolas Thouvenin","Patrice Ringot","Angel Turri"]
```

ou d'appliquer une transformation à ces éléments :  

```js
value = get("value.authors").map(auth => auth.fullname.toUpperCase())
// → Sortie : ["DENIS MAUREL","ENZA MORALE","NICOLAS THOUVENIN","PATRICE RINGOT","ANGEL TURRI"]
```
## flatMap

Combine les rôles de `map` et `flatten`, très pratique pour récupérer en une seule étape des valeurs situées dans des *tableaux imbriqués*.  
Dans cet exemple, *authors* est un tableau et chaque auteur possède une propriété *affiliations* qui est elle-même un tableau contenant les adresses.  
Si l'on souhaite extraie les adresses, il faut donc d'abord aplanir *affiliations* :

```js
value = get("value.authors").map("affiliations").flatten().map("address")
// → Sortie : ["Laboratoire d’Informatique Fondamentale...", "UAR76 / UPS76, Centre National de la Recherche Scientifique C...",...]
```

`flatMap` permet de faire ceci en une seule fois.
```js
value = get("value.authors").flatMap("affiliations").map("address")
```

