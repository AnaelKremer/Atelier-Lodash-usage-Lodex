# Manipuler des objets

Dans Lodex, de nombreuses colonnes ou valeurs peuvent contenir des objets (paires clé-valeur) : noms d'auteurs, adresses, informations bibliographiques… Lodash offre de nombreuses fonctions pour les parcourir, les transformer ou les enrichir.

Certaines fonctions ci-dessous sont exclusives aux objets, tandis que d'autres s'appliquent plus largement aux collections (tableaux ou objets). Cette feuille se concentre sur les fonctions spécifiques aux objets. Les fonctions applicables aux collections seront abordées [ici.]()

> [!NOTE]  
> Vous pouvez tester chaque *exemple* dans [EZS Playground](https://ezs-playground.lodex.inist.fr/). Ecrivez l'input de cette façon : 
> ```json
> [
>   {
>     "value": {
>       "title": "Mouvement historique et histoire suspendue",
>       "year": 2012,
>       "source": "Vingtième Siècle Revue d histoire",
>       "authors": [
>         {
>           "forename": "Hartmut",
>           "surname": "Rosa",
>           "fullname": "Hartmut Rosa"
>         },
>         {
>           "forename": "Johann",
>           "surname": "Chapoutot",
>           "fullname": "Johann Chapoutot"
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
> path = entree
> value = get("value.title")
>
> [dump]
> indent = true
> ```

## get  

Récupère la valeur d’une clé (y compris dans des objets imbriqués).  

```js
value = get("value.title")
// → Sortie : "Mouvement historique et histoire suspendue"
```

## set  

Modifie ou crée une clé avec une nouvelle valeur.  

> [!WARNING]  
> **Fonction mutante** 
> Cette fonction modifie la structure passée en argument. 
> Dans un pipeline, cela peut entraîner des effets de bord si la valeur est réutilisée.

```js
value = get("value.entree").set("DOI", null)
// Entrée : [{"value":{"entree":{"title":"Exemple"}}}]
// Sortie : [{"value":{"entree":{"title":"Exemple","DOI":null}},"sortie":{"title":"Exemple","DOI":null}}]
```

L'objet initial entrée est également modifié.  

✅ Alternative non mutante recommandée : `cloneDeep`

```js
value = get("value.entree").cloneDeep().set("DOI", null)
// Entrée : [{"value":{"entree":{"title":"Exemple"}}}]
// Sortie : [{"value":{"entree":{"title":"Exemple"}},"sortie":{"title":"Exemple","DOI":null}}]
```

## update  

Met à jour la valeur d’une clé à l’aide d’une fonction.  

```js
value = get("value").update("year", y => y + 1)
// → Sortie : "year": 2013
```

## has  

Vérifie si une clé existe ou non.  

```js
value = get("value").has("source")
// → Sortie : true
```

## omit  

Retourne un nouvel objet sans certaines clés.  

```js
value = get("value").omit(["source", "year"])
// → Sortie : { title: "Mouvement historique et histoire suspendue", authors: [...] }
```

## pick  

Retourne un nouvel objet avec uniquement certaines clés.  

```js
value = get("value").pick(["title", "year"])
// → Sortie : { title: "Mouvement historique et histoire suspendue", year: 2012 }
```

## keys  

Retourne les clés d’un objet.  

```js
value = get("value").keys()
// → Sortie : ["title", "year", "source", "authors"]
```

## values  

Retourne les valeurs d’un objet.  

```js
value = get("value").values()
// → Sortie : ["Mouvement historique...", 2012, "Vingtième Siècle Revue d histoire", [ ... ]]
```

## toPairs  

Convertit un objet en un tableau de paires [clé, valeur]. Chaque élément du tableau résultant est un tableau à deux éléments : le premier est la clé (ou le nom de la propriété), et le second est la valeur associée. 

```js
value = get("value").toPairs()
// → Sortie : [["title","Mouvement historique..."], ["year", 2012], ...]
```

## mapValues  

Applique une fonction à toutes les valeurs d’un objet.  

```js
value = get("value").mapValues(value => _.isString(value) ? value.toUpperCase() : value)

// → Sortie : {"title": "MOUVEMENT HISTORIQUE ET HISTOIRE SUSPENDUE","year": 2012,"source": "VINGTIÈME SIÈCLE REVUE D HISTOIRE"...}
// → Si une valeur est une chaîne de caractères, on la passe en majuscules.
```

## mapKeys  

Applique une fonction à toutes les clés d’un objet.  

```js
value = get("value").mapKeys((value, key) => key.toUpperCase())
// → Sortie : {"TITLE": "Mouvement historique et histoire suspendue","YEAR": 2012,"SOURCE": "Vingtième Siècle Revue d histoire"...}
```

## invert  

Inverse les clés et valeurs.  

```js
value = get("value").invert()
// → Sortie : {"2012": "year","Mouvement historique et histoire suspendue": "title"...}
```

## transform  

Transforme un objet en un autre en appliquant une fonction à chaque clé/valeur.  

```js
value = get("value").transform((result, val, key) => {result[key] = _.isNumber(val) ? val + 10 : val}, {})
// → Sortie : "year": 2022
// → Si une clé a comme valeur un nombre, on y ajoute 10.
```

👉 [Chapitre suivant](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/07-collections.md)