# Scripts avancés et cas d'usage

## Nettoyage des chaînes de caractères

### Supprimer les espaces superflus

Lorsque l'on réalise des opérations de nettoyage - comme par exemple enelever la ponctuation, les caractères spéciaux etc - il peut arriver que l'on ai plusieurs espaces consécutifs entre des mots. On utilisera `replace` avec une *regex* afin de remplacer ces multiples espaces par un seul.

```js
value = get("value.entree").replace(/\s+/g, " ").trim()
// Entrée : "Ceci      est      un       test    " → Sortie : "Ceci est un test"
```

---

### Supprimer les caractères spéciaux et les accents

```js
value = get("value.entree").deburr().replace(/[^a-zA-Z0-9 ]/g, "").trim()
// Entrée : "Eurêka !" → Sortie : "Eureka "
```

---

### Supprimer les caractères spéciaux tout en conservant les accents

```js
value = get("value.entree").replace(/[^a-zA-ZÀ-ÖØ-öø-ÿ0-9 ]/g, "").trim()
// Entrée : "Eurêka !" → Sortie : "Eurêka"
```

---

### Nettoyer les balises et entités HTML

Afin de nettoyer un texte stylisé en HTML il convient d'abord d'utiliser `unescape` pour décoder les entités HTML. Par exemple `&amp;` deviendra `&`.  
Puis à l'aide d'une *regex* on supprime les balises HTML.

```js
value = get("value.entree").thru(str => _.unescape(str)) \
    .replace(/<[^>]+>/g, '')
// Entrée : "First lifetime investigations of <i>N</i> &gt; 82 iodine isotopes: The quest for collectivity"
// → Sortie : "First lifetime investigations of N > 82 iodine isotopes: The quest for collectivity"
```

---

### Convertir les symboles grecs en alphabet latin

Dans de nombreux titres et résumés apparaissent des symboles grecs, si pour des besoins de normalisation par exemple vous souhaitez les convertir, appliquer ce script.

```js
[env]
path = replaceGreeks
value = fix((title)=> \
  Object \
    .entries({ \
      'α': 'alpha', \
      'β': 'beta', \
      'γ': 'gamma', \
      'δ': 'delta', \
      'ε': 'epsilon', \
      'θ': 'theta', \
      'λ': 'lambda', \
      'μ': 'mu', \
      'π': 'pi', \
      'σ': 'sigma', \
      'φ': 'phi', \
      'ω': 'omega' \
    }) \
    .reduce( \
      (acc, [symbol, name]) => \
        acc \
          .split(symbol) \
          .join(name), \
      title \
    ) \
)
// Le script qui verbalise le symboles est stocké dans une variale d'environnement [ENV].
// Ce qui permet de pouvoir l'utiliser plusieurs fois sur les champs title et abstract avec simplement
// .thru(env("replaceGreeks"))

[replace]
path = sortie
value = get("value.title").thru(env("replaceGreeks"))

// Entrée : "From α to ω."
// → Sortie : "From alpha to omega."

```

## Exemples de transformations diverses

Ces scripts ne sont pas forcément tous des cas d'usages existants, mais peuvent servir à montrer les bonnes syntaxes à adopter pour combiner vos fonctions.

### Construire un tableau à partir d’un champ et d'un autre champ transformé pour l'occasion

Cas simple, on souhaite obtenir dans un tableau l'année de publication, la revue et l'éditeur d'une publication.

```json
{
  "year": 2019,
  "source": "Information",
  "publisher": "Mdpi"
}
```

Mais on souhaite également convertir l'année en chaîne (c'était un nombre), mettre en minuscules la revue et en majuscules l'éditeur. Mais **sans modifier les champs du dataset**, juste dans le cas de ce tableau.  

Plutôt que d'écrire 3 enrichissements, puis de les concaténer, on peut réaliser toutes les opérations en un seul script.

```js
value=get("value.year").toString() \
// On récupère l'année puis la transforme en chaîne
    .concat(_.toLower(self.value.source)) \
// On concatène ensuite la revue que l'on met en minuscules. 
// Pour cela on place la fonction de transformation avant le nom du champ, que l'on met entre parenthèses pour l'occasion. 
// Et on n'oublie pas de préfixer la fonction par le _ de Lodash
    .concat(_.toUpper(self.value.publisher))
// Enfin on concatène le dernier champ en le transformant de la même façon
```

Cependant on peut encore simplifier l'écriture et le nombre de fonctions utilisées.

En utilisant `fix` on peut directement créer un tableau en séparant les valeurs par des virgules → `fix(selfvalueA, selfvalueB)`, on n'a donc plus besoin d'ajouter autant de `concat` que de champs.  

Ce qui donne  :

```js
value=fix(_.toString(self.value.year), _.toLower(self.value.source), _.toUpper(self.value.publisher))
// → Sortie : ["2019","information","MDPI"]
```

---

### Construire un tableau à partir d’un champ et d’une liste extraite d’un objet dans un autre champ

Clarifions la problématique, on souhaite par exemple réunir en un seul tableau un *doi* et les *auteurs* associés.  

Le soucis étant que le *doi* se trouve dans un champ (ou colonne), mais que les *auteurs* se trouvent dans un tableau d'objets d'un autre champ.

Pour visualiser, voici le dataset réduit à ces deux seuls champs :

```json
[
  {
    "value": {
      "doi": "10.3390/info10050178",
      "authors": [
        {
          "fullname": "Denis Maurel",
          "affiliations": [
            "Laboratoire d’Informatique Fondamentale et Appliquée de Tours LIFAT, Bases de données et traitement des langues naturelles BDTLN, FR"
          ]
        },
        {
          "fullname": "Enza Morale",
          "affiliations": [
            "UAR76 / UPS76, Centre National de la Recherche Scientifique CNRS, Institut de l’information scientifique et technique INIST, 2, Allée du Parc de Brabois, Rue Jean Zay, 54500 Vandœuvre-lès-Nancy, FR"
          ]
        },
        {
          "fullname": "Nicolas Thouvenin",
          "affiliations": [
            "UAR76 / UPS76, Centre National de la Recherche Scientifique CNRS, Institut de l’information scientifique et technique INIST, 2, Allée du Parc de Brabois, Rue Jean Zay, 54500 Vandœuvre-lès-Nancy, FR"
          ]
        },
        {
          "fullname": "Patrice Ringot",
          "affiliations": [
            "UAR76 / UPS76, Centre National de la Recherche Scientifique CNRS, Institut de l’information scientifique et technique INIST, 2, Allée du Parc de Brabois, Rue Jean Zay, 54500 Vandœuvre-lès-Nancy, FR"
          ]
        },
        {
          "fullname": "Angel Turri",
          "affiliations": [
            "UAR76 / UPS76, Centre National de la Recherche Scientifique CNRS, Institut de l’information scientifique et technique INIST, 2, Allée du Parc de Brabois, Rue Jean Zay, 54500 Vandœuvre-lès-Nancy, FR"
          ]
        }
      ]
    }
  }
]

```

En pratique, on va créer un enrichissement pour extraire les noms des *auteurs* car l'opération nécessite un `map`, puis on fera ensuite un second enrichissement pour concaténer cette liste au *doi*.  

Pour faire cela en une seule étape, il faudrait pouvoir mapper les *auteurs* à l'intérieur de `concat`. ce qui s'écrit comme cela : 

```js
value = get("value.doi").concat(_(self.value.authors).map('fullname'))
```

Seulement, on concatène ici une chaîne (le *doi*) à un tableau (les *auteurs*). On obtient donc comme résultat un tableau avec un tableau imbriqué :  
```["10.3390/info10050178",["Denis Maurel","Enza Morale","Nicolas Thouvenin","Patrice Ringot","Angel Turri"]]```

Pour obtenir un tableau à plat, il faudrait insérer des `join` et `split` (les fonctions de type `flatten` ne marcheront pas ici) mais la lecture du script sera compliquée...

Au lieu de cela nous utiliserons d'abord `fix` au lieu de `get`, ceci afin de pouvoir insérer un **spread operator** `...` dans la fonction.  

Nous ne développerons pas ici ce qu'est un **spread operator** ou [syntaxe de décomposition](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Operators/Spread_syntax), disons simplement qu'il permet d’éclater un tableau en ses éléments individuels afin d’obtenir un tableau plat.

Ainsi : 

```js
value = fix(self.value.doi, ...(_(self.value.authors).map('fullname')))
```

renverra : 

```["10.3390/info10050178","Denis Maurel","Enza Morale","Nicolas Thouvenin","Patrice Ringot","Angel Turri"]```