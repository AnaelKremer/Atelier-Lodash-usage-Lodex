# Autres fonctions utiles

Certaines fonctions Lodash ne sont pas spécifiques aux **chaînes**, **tableaux**, **objets** ou **collections**, mais sont utiles pour tester, convertir ou manipuler des données de manière générale.  

## thru  

**`thru` est le couteau suisse de Lodash ! ** 

Si la fonction qu’il vous faut pour transformer vos données n’existe pas en **Lodash**, vous pouvez presque toujours la construire grâce à `thru`.  

La force de thru est qu’elle sert de "pont" entre **Lodash** et **JavaScript** : à l’intérieur de la fonction fléchée (=>), elle permet d'écrire n’importe quelle logique en JavaScript pur.  

Cela permet d’appliquer des conditions, des calculs ou des traitements complexes et propres à nos métiers qui ne sont pas couverts par les fonctions **Lodash** existantes.  

Contrairement aux fonctions spécialisées (pensées pour un type de données précis : tableau, objet, chaîne…), `thru` n’impose aucune contrainte et peut s'appliquer sur tout.  

Par exemple, face à un jeu de données très hétérogène, on peut écrire un script qui va vérifier si la valeur est un tableau, puis lui appliquer certaines opérations, ou si c’est une chaîne de caractères, lui appliquer d’autres transformations.

Quelques exemples : 

Ici on évite d’écrire plusieurs enrichissements successifs : tout est géré en une seule transformation conditionnelle.

```js
value = get("value.entree").thru(y => y <= 2000 ? "ancien" : "récent")
// Entrée : 2019 → Sortie : "récent"
// Entrée : 1986 → Sortie : "ancien"
```

Cet exemple n’a qu’une **vocation illustrative** : 

1. On teste d'abord si la valeur est *vide*, si c'est le cas on y met la chaîne *"non renseigné"*
2. On teste ensuite si la valeur est un tableau.
    - Si le tableau ne possède qu'un seul élément, on le transforme simplement chaaîne de caractères. 
    - Si le tableau possède plusieurs éléments, on garde uniquement le 1er élément, le transforme en chaîne et finit la chaîne par *" et autres"*. 
3. Enfin si aucune condition n'est remplie on renvoi simplement la valeur (ce pourrait être un nombre ici)

```js
value = get("value.entree").thru(v => \
  _.isEmpty(v) ? "non renseigné" : \
  _.isArray(v) ? \
    (v.length === 1 ? v[0] : v[0] + " et autres") : \
  v \
)
// Entrée : "banane" → Sortie : "banane"
// Entrée : ["banane", "pomme", "poire"] → Sortie : "banane et autres"
// Entrée : ["pomme"] → Sortie : "pomme"
// Entrée : "" → Sortie : "non renseigné"
```

## uniqueId  

Génère un identifiant unique. Peut être utile pour numéroter les lignes d'un corpus.

```js
value = fix("notice_").uniqueId()
// → Sortie : "notice_1"
```

## now  

Renvoie l’horodatage courant en millisecondes.  

```js
value = fix("exemple").now()
// → Sortie : 1737475843287
```

## max  

Renvoie la plus grande valeur d’un tableau.  

```js
value = get("value.entree").max()
// Entrée : [1, 2, 3] → Sortie : 3
```

## maxBy

Parcourt un tableau d’objets et renvoie celui dont la propriété spécifiée a la plus grande valeur.  

```js
value = get("value.entree").maxBy("age")
// Entrée : [{"name":"Alice","age":42},{"name":"Bob","age":36},{"name":"Charlie","age":50}]
// → Sortie : {"name":"Charlie", "age":50}
```

## min  

Renvoie la plus petite valeur d’un tableau.  

```js
value = get("value.entree").min()
// Entrée : [1, 2, 3] → Sortie : 1
```

## minBy

Parcourt un tableau d’objets et renvoie celui dont la propriété spécifiée a la plus petite valeur.  

```js
value = get("value.entree").minBy("age")
// Entrée : [{"name":"Alice","age":42},{"name":"Bob","age":36},{"name":"Charlie","age":50}]
// → Sortie : {"name":"Bob","age":36}
```

## sum  

Additionne tous les éléments d’un tableau.  

```js
value = get("value.entree").sum()
// Entrée : [1, 2, 3] → Sortie : 6
```

## mean 

Calcule la moyenne arithmétique des valeurs numériques d’un tableau.  

```js
value = get("value.entree").mean()
// Entrée : [1, 2, 3] → Sortie : 2
```

## add

Additionne deux nombres.

```js
value = get("value.entree").add(self.value.entree2)
// Entrée 1: 8 
// Entrée 2: 2
// → Sortie : 10
```
## subtract

Soustrait le second nombre du premier.

```js
value = get("value.entree").subtract(self.value.entree2)
// Entrée 1: 8 
// Entrée 2: 2
// → Sortie : 6
```
## multiply

Multiplie deux nombres.

```js
value = get("value.entree").multiply(self.value.entree2)
// Entrée 1: 8 
// Entrée 2: 2
// → Sortie : 16
```
## divide

Divise le premier nombre par le second.

```js
value = get("value.entree").divide(self.value.entree2)
// Entrée 1: 8 
// Entrée 2: 2
// → Sortie : 4
```

## round

Arrondit un nombre au plus proche entier par défaut :

```js
value = get("value.entree").round()
// Entrée : 3.14159265359 → Sortie : 3
```

ou à une précision donnée : 

```js
value = get("value.entree").round(2)
// Entrée : 3.14159265359 → Sortie : 3.14
```

