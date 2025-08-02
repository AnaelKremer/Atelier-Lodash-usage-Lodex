# Manipuler des tableaux  

Dans Lodex, beaucoup de colonnes contiennent des tableaux : listes d’auteurs, mots-clés, affiliations… Lodash propose une riche panoplie de fonctions pour filtrer, ordonner, fusionner, ou transformer ces données.

Certaines fonctions sont dédiées exclusivement aux tableaux, tandis que d'autres s'appliquent plus largement aux collections (tableaux ou objets). Cette feuille se concentre sur les fonctions spécifiques aux tableaux. Les fonctions applicables aux collections seront abordées [ici.]()

> [!NOTE]
> Vous pouvez tester chaque *exemple* dans [EZS Playground](http://ezs-playground.daf.intra.inist.fr/). Ecrivez l'input de cette façon :  
> ```json
> [
>   {
>     "value": {
>       "entree": ["A", "B"],
>       "entree2": ["A", "C"]
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
> value = get("value.entree")
>
> path = entree2
> value = get("value.entree2")
>
> path = sortie
> value = get("value.entree").intersection(self.value.entree2)
>
> [dump]
> indent = true
> ```

## concat

Concatène des tableaux ou des valeurs au tableau existant.

```js
value = get("value.entree").concat(self.value.entree2)
// Entree : ["A", "B"] Entree2 : ["A", "C"] → Sortie : ["A", "B", "A", "C"]
```
## compact

Supprime toutes les valeurs falsy (`false`, `null`, `0`, `""`, `undefined`, `NaN`) d’un tableau.

```js
value = get("value.entree").compact()
// Entree : ["A", null, "", "B", false] → Sortie : ["A", "B"]
```

## uniq

Supprime les doublons dans un tableau.

```js
value = get("value.entree").uniq()
// Entree : ["A", "B", "A", "C"] → Sortie : ["A", "B", "C"]
```

## first / head

Renvoie le premier élément d’un tableau.

```js
value = get("value.entree").head()
// Entree : ["A", "B", "C"] → Sortie : ["A"]
```

## last

Renvoie le dernier élément d’un tableau.

```js
value = get("value.entree").last()
// Entree : ["A", "B", "C"] → Sortie : ["C"]
```

## flatten

Aplati un tableau **d’un seul** niveau. Utile lorsqu'on a un tableau de tableaux ou matrice.

```js
value = get("value.entree").flatten()
// Entree : [[1, 2], [3]] → Sortie : [1, 2, 3]
```

## flattenDeep

Aplati récursivement un tableau **à tous** les niveaux.

```js
value = get("value.entree").flattenDeep()
// Entree : [1, [2, [3, [4]]]] → Sortie : [1, 2, 3, 4]
```

## nth

Renvoie l’élément à la position `n`.

```js
value = get("value.entree").nth(1)
// Entree : ["A", "B", "C"] → Sortie : ["B"]
```

## pull

Supprime certaines valeurs du tableau.

```js
value = get("value.entree").pull("A")
// Entree : ["A", "B", "A", "C"] → Sortie : ["B", "C"]
```

## pullAll

Supprime plusieurs valeurs du tableau.

```js
value = get("value.entree").pullAll(["B","C"])
// Entree : ["A", "B", "C", "D"] → Sortie : ["A", "D"]
```

## remove

> [!WARNING] 
> Cette fonction est assez contre-intuitive, elle supprime tous les éléments qui satisfont une condition mais renvoie le tableau des éléments supprimés.

```js
value = get("value.entree").remove(item=>item.startsWith("B"))
// Entree : ["ABCD", "BCDE", "CDEF", "DEFG"] → Sortie : ["BCDE"]
```

## sort

Trie un tableau (attention : tri alphabétique ou par code Unicode). Pour trier des nombres voir les fonctions avancées.

```js
value = get("value.entree").sort()
// Entree : ["C", "Q", "F", "D"] → Sortie : ["C", "D", "F", "Q"]
```

## reverse

Inverse l'ordre des éléments.

```js
value = get("value.entree").reverse()
// Entrée : ["A", "B", "C"] → Sortie : ["C", "B", "A"]
```

## zip / unzip

Associe les éléments de plusieurs tableaux entre eux (zip), ou inverse l’opération (unzip).

```js
value = zip(self.value.entree,self.value.entree2)
// Peut aussi s'écrire value = get("value.entree").zip(self.value.entree2)
// Entree : ["Niels Bohr", "Albert Einstein"] Entree2 : ["Danemark", "Allemagne"] → Sortie : [["Niels Bohr","Danemark"],["Albert Einstein","Allemagne"]]

value = get("value.entree").unzip()
// Entree : [["Niels Bohr","Danemark"],["Albert Einstein","Allemagne"]] → Sortie : [["Niels Bohr","Albert Einstein"],["Danemark","Allemagne"]]
```

## fromPairs

Transforme un tableau de paires en un objet clé-valeur. ! Attention à toujours bien avoir des tableaux des deux éléments.

```js
value = get("value.entree").unzip()
// Entree : [["Niels Bohr","Danemark"],["Albert Einstein","Allemagne"]] → Sortie : {"Niels Bohr":"Danemark","Albert Einstein":"Allemagne"}
```


