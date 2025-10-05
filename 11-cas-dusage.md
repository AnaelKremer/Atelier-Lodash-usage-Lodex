# Scripts avancés et cas d'usage

## Nettoyage des chaînes de caractères

### Mettre une chaîne de caractères sous une forme canonique normalisée

Si vous souhaitez réaliser un important travail de normalisation des chaînes de caractères, voici la 1ère fonction à appliquer afin d'éviter toute surprise.  

Il peut arriver de rencontrer, dans d'importants jeux de données hétérogènes, des **variantes Unicode** comme par exemple `e\u0301cole`.  

Pour traiter ces cas on utilisera donc une méthode **JavaScript native**, `normalize` (que l'on appelle donc via une fonction fléchée) avec la forme de normalisation `NFKC` (*Normalization Form Compatibility Composition*).

```js
value = get("value.entree") \
    .thru(str => str.normalize("NFKC")) \
    .replace(/\s+/g, " ") \
    .deburr()
    .toLower() \
    .trim()
// Entrée : "\u00E9cole" → Sortie : "ecole"
```

---

### Supprimer les espaces superflus

Lorsque l'on réalise des opérations de nettoyage - comme par exemple enlever la ponctuation, les caractères spéciaux etc - il peut arriver que l'on ai plusieurs espaces consécutifs entre des mots. On utilisera `replace` avec une *regex* afin de remplacer ces multiples espaces par un seul.

```js
value = get("value.entree").replace(/\s+/g, " ").trim()
// Entrée : "Ceci      est      un       test    " → Sortie : "Ceci est un test"
```

---

### Supprimer les caractères spéciaux et les accents

```js
value = get("value.entree").deburr().replace(/[^a-zA-Z0-9 ]/g, "").trim()
// Entrée : "Eurêka !" → Sortie : "Eureka"
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

### Générer une date selon les conventions françaises

La fonction **JavaScript** `new Date` permet de retourner la date et l'heure ainsi que la valeur UTC (universelle).  

```json
[
  {
    "doi": "10.11111"
  },
  {
    "doi": "10.22222"
  }
]
```

```js
[assign]
path = date
value = fix(new Date())
```

```json
[{
    "doi": "10.11111",
    "date": "2025-10-05T16:09:28.554Z"
},
{
    "doi": "10.22222",
    "date": "2025-10-05T16:09:28.555Z"
}]
```

Si l'on souhaite obtenir la date selon les conventions françaises (pour dater le corpus, ou une transformation) on peut ajouter une autre fonction **JavaScript** :

```js
[assign]
path = date
value = fix(new Date()) \
  .thru(d => new Intl.DateTimeFormat('fr-FR', { \
    year: 'numeric', \
    month: 'long', \
    day: 'numeric', \
  }).format(d))
```

```json
[{
    "doi": "10.11111",
    "date": "5 octobre 2025"
},
{
    "doi": "10.22222",
    "date": "5 octobre 2025"
}]
```

Et si l'on souhaite davantage de précisions :  

```js
[assign]
path = date
value = fix(new Date()) \
  .thru(d => new Intl.DateTimeFormat('fr-FR', { \
    timeZone: 'Europe/Paris', \
    year: 'numeric', \
    month: 'long', \
    weekday: 'long', \
    day: 'numeric', \
    hour: '2-digit', \
    minute: '2-digit', \
    second: '2-digit' \
  }).format(d))
```

:point_down:

```json
[{
    "doi": "10.11111",
    "date": "dimanche 5 octobre 2025 à 18:20:44"
},
{
    "doi": "10.22222",
    "date": "dimanche 5 octobre 2025 à 18:20:44"
}]
```

---

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

## Transformations globales (dans le cadre d'un loader)

### Numéroter les lignes de son dataset
  
Il peut être utile, pour retrouver certaines données, d'avoir un corpus numéroté. Pour cela on peut utiliser la fonction `uniqueId` et customiser notre identifiant pour mieux s'adapter à nos besoin.  

Ici par exemple, pour un corpus de donnée de plusieurs centaines de milliers de documents, on créé un champ *line* à 6 chiffres :

```json
[
  {
    "doi": "10.11111"
  },
  {
    "doi": "10.22222"
  },
 {
    "doi": "10.33333"
  },
 {
    "doi": "10.44444"
  },
 {
    "doi": "10.99999"
  }
]
```

```js
[assign]
path = line
value = fix("").uniqueId().padStart(6, "0")
```

:point_down:

```json
[{
    "doi": "10.11111",
    "line": "000001"
},
{
    "doi": "10.22222",
    "line": "000002"
},
{
    "doi": "10.33333",
    "line": "000003"
},
{
    "doi": "10.44444",
    "line": "000004"
},
{
    "doi": "10.99999",
    "line": "000005"
}]
```

💡 On peut également mettre cet identifiant unique en guise d'uri :

```js
[assign]
path = uri
value = fix("").uniqueId().padStart(6, "0")
```

---

### Dédoublonner des lignes parfaitement identiques

Dans certains jeux de données, on ne dispose pas de champ discriminant clair (comme un DOI ou autre identifiant unique) qui permet de détecter et supprimer les doublons.  

Dans ces situations, la seule solution consiste à comparer l’objet entier. Si deux lignes sont strictement identiques, l’une d’elles doit être supprimée.  

Or, deux objets JavaScript identiques peuvent être difficiles à comparer directement, surtout dans un flux de données.  
La seule solution consiste à générer une **empreinte unique** de chaque objet, puis à dédoublonner sur cette empreinte.  

Pour ce faire, il faut utiliser un **SHA** *(Secure Hash Algorithm)*.  
C'est une fonction de hachage cryptographique qui transforme une donnée (texte, nombre, objet JSON…) en une empreinte numérique unique (une longue chaîne de caractères).  
*(Si vous souhaitez en connaitre davantage sur ce point, je vous conseille cet excellent [livre](https://www.deboecksuperieur.com/livre/9782807345331-les-algorithmes-c-est-plus-simple-avec-un-dessin)).*

Il faut donc appliquer une fonction SHA à chaque ligne de son dataset pour pouvoir dédoublonner.

Démonstration :

```json
[
  { "a": 1, "b": 2 },
  { "a": 2, "b": 3 },
  { "a": 1, "b": 2 }
]
```

```js
[assign]
; On met l'objet complet dans un champ hash
path = hash
value = self().cloneDeep().thru(obj => _(obj).toPairs().sortBy(0).fromPairs().value())
; Par sécurité on copie un objet propre pour éviter les références circulaires et on trie ses clés par ordre alphabétique

[map]
; On "prépare" le champ hash pour lui appliquer plusieurs transformations spécifiques
path = hash

[map/identify]
; On génère un identifiant SHA
scheme = sha
path = hash

[map/exchange]
value = get('hash')
```

A cette étape du script on a bien généré des empreintes numériques dans le champ *hash* :

```json
[{
    "a": 1,
    "b": 2,
    "hash": [
        "sha:/43258cff783fe7036d8a43033f830adfc60ec037382473548ac742b888292777z"
    ]
},
{
    "a": 2,
    "b": 3,
    "hash": [
        "sha:/206f7b5543e6f2ef39bf334988fd7097b725caeed16588cd9d785480f2f0f8f6S"
    ]
},
{
    "a": 1,
    "b": 2,
    "hash": [
        "sha:/43258cff783fe7036d8a43033f830adfc60ec037382473548ac742b888292777z"
    ]
}]
```

Ne reste plus qu'à dédoublonner et éventuellement enlever le champ *hash* qui a joué son rôle.

```js
[dedupe]
path = hash
ignore = true

[exchange]
value = omit("hash")
```

:point_down:

```json
[
  {"a":1,"b":2},
  {"a":2,"b":3}
]
```

---

### Trier les colonnes de son dataset par ordre alphabétique

```json
[
  {
    "title": "Bibliométrie prête à l'emploi avec OpenAlex : retour d'expérience",
    "authors": [
      "Carine Bach",
      "Lucile Bourguignon",
      "Christa Guélé",
      "Philippe Houdry",
      "Anaël Kremer"
    ],
    "document_type": "Working Paper",
    "year": 2025,
    "abstract": "En décembre 2023, le MESR a mis en place un partenariat pluriannuel avec OpenAlex. En janvier 2024, le CNRS annonce se désabonner de Scopus...",
    "keywords": [
      "OpenAlex",
      "bibliométrie"
    ]
  }
]
```

```js
[exchange]
value = self() \
    .toPairs() \
    .sortBy(0) \
    .fromPairs() 
```

On utilise évidemment `[exchange]` pour remplacer le dataset d'origine par notre dataset transformé.  

- `self` pour travailler sur tout l'objet courant,  
- `toPairs` transforme l'objet en un tableau de paires clé/valeur => `["document_type", "Working Paper"],["year", 2025]...`,
- `sortBy(0)` trie le tableau par le premier élément de chaque paire (donc la clé),
- `fromPairs` reconstruit l'objet.

```json
[{
    "abstract": "En décembre 2023, le MESR a mis en place un partenariat pluriannuel avec OpenAlex. En janvier 2024, le CNRS annonce se désabonner de Scopus...",
    "authors": [
        "Carine Bach",
        "Lucile Bourguignon",
        "Christa Guélé",
        "Philippe Houdry",
        "Anaël Kremer"
    ],
    "document_type": "Working Paper",
    "keywords": [
        "OpenAlex",
        "bibliométrie"
    ],
    "title": "Bibliométrie prête à l'emploi avec OpenAlex : retour d'expérience",
    "year": 2025
}]
```

---

### Supprimer des préfixes ou suffixes dans des noms de colonnes.  

Il arrive fréquemment que les colonnes d’un dataset contiennent des préfixes ou suffixes parasites. Par exemple : 

```json
[{
  "metaTitle": "Bibliométrie prête à l'emploi avec OpenAlex : retour d'expérience",
  "metaDocument_type": "Working Paper",
  "metaYear": 2025
}]
```

```js
[exchange]
value = self().mapKeys((value, key) => \
    _.startsWith(key, "meta") \
        ? key.slice(4) \
        : key \
)
```

- `mapKeys` permet d'appliquer une fonction à toutes les clés d'un objet,
- `.startsWith(key, "meta")` permet de ne capturer que les clés commençant par "meta",
- `key.slice(4)` supprime les 4 premiers caractères de ces clés, donc meta,
- `: key` renvoie toutes les autres clés inchangées.

```json
[{
    "Title": "Bibliométrie prête à l'emploi avec OpenAlex : retour d'expérience",
    "Document_type": "Working Paper",
    "Year": 2025
}]
```

Si l'on souhaite supprimer des suffixes, on utilisera `endsWith` en lieu et place de `startsWith` et modifiera `slice` pour supprimer les derniers caractères cette fois.

```js
[exchange]
value = self().mapKeys((value, key) => \
    _.endsWith(key, "_old") \
        ? key.slice(0,-4) \
        : key \
)
```

---

### Standardiser des noms de colonnes.

Si vous vous souvenez des "bonnes pratiques" [énnoncées](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/03-syntaxe-et-bonnes-pratiques.md#le-nommage-des-colonnesenrichissements) au début de ce parcours, vous savez qu'il est important d'avoir des colonnes nommées correctement.  

Mais souvent on doit faire avec ce que l'on a... 

On a déjà vu comment nettoyer des chaînes de caractères, en combinant `mapKeys` à `[exchange]` on peut appliquer ces transformations à toutes les colonnes du datatset !  

```json
[
  {
    "_is_oa_?": true,
    "_publication_year": 2025,
    "_author_affiliations": "CNRS"
  }
]

```

```js
[exchange]
value = self().mapKeys((value, key) => \
    _.camelCase( \
        _.deburr(key) \
          .replace(/[^a-zA-Z0-9_ ]/g, "") \
    ) \
)
```

- `mapKeys` pour itére sur toutes les clés d'un objet,
- `camelCase` qui applique la convention de nommage du même nom,
- `deburr` pour enlever les accents,
- `replace` avec une *regex* pour retirer la ponctuation ou caractères spéciaux.

:point_down:

```json
[{
    "isOa": true,
    "publicationYear": 2025,
    "authorAffiliations": "CNRS"
}]
```

---

### Transformer les valeurs de type chaînes de caractères de plusieurs colonnes

On peut transformer les valeurs de plusieurs colonnes à l'aide `mapValues` et `[exchange]` en sélectionnant les colonnes à transformer :

```json
[
  {
    "title": "Bibliométrie prête à l'emploi avec OpenAlex : retour d'expérience",
    "authors": [
"Carine Bach", "Lucile Bourguignon", "Christa Guélé", "Philippe Houdry", "Anaël Kremer"],
    "document_type": "Working Paper",
    "year": 2025,
    "abstract": "En décembre 2023, le MESR a mis en place un partenariat pluriannuel avec OpenAlex. En janvier 2024, le CNRS annonce se désabonner de Scopus...",
    "keywords": ["OpenAlex", "bibliométrie"]
  }
]
```

```js
[exchange]
value = self().mapValues( \
    (value, key) => \
        ['title', 'abstract','document_type'].includes(key) \
            ? _.toLower(_.deburr(_.trim(value))) \
            : value \
)
```

- `mapValues` applique une transformation à toutes les valeurs de l’objet, clé par clé,
- `includes(key)` ne transforme que si la clé fait partie de la liste ['title', 'abstract','document_type'],
- `deburr` supprime les accents et diacritiques (é → e),
- `trim` supprime les espaces superflus au début et à la fin de la chaîne,
- `toLower` convertit tout le texte en minuscules.

:point_down:

```json
[{
    "title": "bibliometrie prete a l'emploi avec openalex : retour d'experience",
    "authors": [
        "Carine Bach",
        "Lucile Bourguignon",
        "Christa Guélé",
        "Philippe Houdry",
        "Anaël Kremer"
    ],
    "document_type": "working paper",
    "year": 2025,
    "abstract": "en decembre 2023, le mesr a mis en place un partenariat pluriannuel avec openalex. en janvier 2024, le cnrs annonce se desabonner de scopus...",
    "keywords": [
        "OpenAlex",
        "bibliométrie"
    ]
}]
```

💡 On peut également étendre les transformations à **tous les champs de type string** !

```js
[exchange]
value = self().mapValues( \
    (value) => \
        _.isString(value) \
            ? _.toLower(_.deburr(_.trim(value))) \
            : value \
)
```

L'utilisation de `isString` ici permet d'appliquer les fonction à tous les champs ayant des données de type string sans devoir sélectionner tous les champs manuellement.

---

### Transformer des valeurs dans des tableaux pour plusieurs colonnes

Ici on va appliquer nos transformations uniquement à des tableaux, et seulement si les valeurs de ces tableaux sont des strings : 

```json
[
  {
    "title": "Bibliométrie prête à l'emploi avec OpenAlex : retour d'expérience",
    "authors": [
      "Carine Bach",
      "Lucile Bourguignon",
      "Christa Guélé",
      "Philippe Houdry",
      "Anaël Kremer"
    ],
    "des booléens": [
      true,
      false,
      true
    ],
    "year": 2025,
    "keywords": [
      "OpenAlex",
      "bibliométrie"
    ]
  }
]
```

```js
[exchange]
value = self().mapValues( \
    (value, key) => \
        _.isArray(value) \
            ? value.map(v => _.isString(v) ? _.capitalize(_.deburr(_.trim(v))) : v) \
            : value \
)
```

Cela s'écrit de la même façon que dans l'exemple précédent, on ajoute seulement une nouvelle condition avec `isArray` et un `map` pour itérer sur chaque élément des tableaux.

```json
[{
    "title": "Bibliométrie prête à l'emploi avec OpenAlex : retour d'expérience",
    "authors": [
        "Carine bach",
        "Lucile bourguignon",
        "Christa guele",
        "Philippe houdry",
        "Anael kremer"
    ],
    "un tableau de booléens": [
        true,
        false,
        true
    ],
    "year": 2025,
    "keywords": [
        "Openalex",
        "Bibliometrie"
    ]
}]
```

On constate que tous les champs n'étant pas des tableaux n'ont pas été impactés, et que le champ *des booléens* qui ne contenait pas de strings n'a pas été transformé non plus.  
Seuls les tableaux *authors* et *keywords* ont été modifiés.

---

### Transformer des champs en tableaux à partir de chaînes délimitées

On peut rencontrer dans certains jeux de données des champs contenant plusieurs valeurs sous forme de chaînes avec un séparateur.  
Cela peut facilmeent être transformé en tableaux :

```json
[
  {
    "authors": "Carine Bach;Lucile Bourguignon;Christa Guélé;Philippe Houdry;Anaël Kremer",
    "year": 2025,
    "keywords": "OpenAlex;bibliométrie"
  }
]
```

Toujours les célèbres `mapValues` et `includes`, il suffit esnuite de splitter les champs avec le bon séparateur.

```js
[exchange]
value = self().mapValues((value, key) => \
  ['authors', 'keywords'].includes(key) \
    ? _.split(value, ';') \
    : value \
)
```

:point_down:

```json
[{
    "authors": [
        "Carine Bach",
        "Lucile Bourguignon",
        "Christa Guélé",
        "Philippe Houdry",
        "Anaël Kremer"
    ],
    "year": 2025,
    "keywords": [
        "OpenAlex",
        "bibliométrie"
    ]
}]
```

---

### Trier les lignes du dataset en fonction d'un champ

On souhaite, par exemple, réorganiser son dataset selon le `nom` et par ordre alphabétique.

```json
[
  { "nom": "Dupont", "prenom": "Bob" },
  { "nom": "Martin", "prenom": "Zoe" },
  { "nom": "Dupont", "prenom": "Alice" },
  { "nom": "Martin", "prenom": "Alex" }
]
```

```js
[sort]
path = nom
```

:point_down:

```json
[{
    "nom": "Dupont",
    "prenom": "Bob"
},
{
    "nom": "Dupont",
    "prenom": "Alice"
},
{
    "nom": "Martin",
    "prenom": "Zoe"
},
{
    "nom": "Martin",
    "prenom": "Alex"
}]
```

> [!TIP]
> On peut inverser le tri en ajoutant simplement 
> `reverse = true`

---

### Trier les lignes du dataset en fonction de plusieurs champs

Pour le même exemple, on souhaiterait affiner le tri. Si le même nom est présent plusieurs fois, on classe par prénom et par ordre alphabétique.

```json
[
  { "nom": "Dupont", "prenom": "Bob" },
  { "nom": "Martin", "prenom": "Zoe" },
  { "nom": "Dupont", "prenom": "Alice" },
  { "nom": "Martin", "prenom": "Alex" }
]
```

Il est nécessaire de mettre 2 fois l'instruction, et on fait bien attention à l'ordre des instructions. Du plus fin au plus général. 

```js
[sort]
path = prenom

[sort]
path = nom
```

:point_down:

```json
[{
    "nom": "Dupont",
    "prenom": "Alice"
},
{
    "nom": "Dupont",
    "prenom": "Bob"
},
{
    "nom": "Martin",
    "prenom": "Alex"
},
{
    "nom": "Martin",
    "prenom": "Zoe"
}]
```

---

### Eclater puis fusionner des données

Ce type de manipulation est utile lorsqu’on veut **changer la logique d’organisation** d’un dataset :  
passer d’un format « une ligne par publication » à un format « une ligne par affiliation », par exemple.  

Prenons un jeu de données où figurent des publications et les affiliations associées :   

| doi | affiliations   |
|------------|-----------------------------------|
| 10.11111   | laboratoire A ; laboratoire B     |
| 10.22222   | laboratoire B ; laboratoire C     |

Soit l'objet : 

```json
[
  {
    "doi": "10.11111",
    "affiliations": ["laboratoire A", "laboratoire B"]
  },
  {
    "doi": "10.22222",
    "affiliations": ["laboratoire B", "laboratoire C"]
  }
]
```

On souhaiterais réorganiser le jeu de données de sorte à obtenir une ligne pour chaque affiliation avec les publications associées, soit :  

| affiliations | doi       |
|---------------------------------|--------------------|
| laboratoire A                   | 10.11111           |
| laboratoire B                   | 10.11111 ; 10.22222|
| laboratoire C                   | 10.22222           |

Il va donc falloir dans un premier temps multiplier les lignes à partir des affiliations. Pour cela, 3 petites lignes suffisent.  

```js
[exploding]
id = doi
value = affiliations
```

On obtient désormais ceci :

```json
[{
    "id": "10.11111",
    "value": "laboratoire A"
},
{
    "id": "10.11111",
    "value": "laboratoire B"
},
{
    "id": "10.22222",
    "value": "laboratoire B"
},
{
    "id": "10.22222",
    "value": "laboratoire C"
}]
```

On notera qu’à cette étape, nous avons perdu les noms de nos champs d’origine :
- les doi sont désormais contenus dans la clé `id`,
- les affiliations se trouvent dans la clé `value`.  

Avant d’utiliser `[aggregate]`, il faut donc inverser cette logique :
- placer les affiliations (actuellement dans `value`) dans `id`,
- mettre les DOIs (actuellement dans `id`) dans `value`.

Cela se fait en quelques lignes :

```js
[replace]
path = id
value = get("value")

path = value
value = self().omit("value")

[aggregate]
path = id
```

Les données sont maintenant agrégées mais il faut encore reconstruire l'objet :  

```json
[{
    "id": "laboratoire A",
    "value": [
        {
            "id": "10.11111"
        }
    ]
},
{
    "id": "laboratoire B",
    "value": [
        {
            "id": "10.11111"
        },
        {
            "id": "10.22222"
        }
    ]
},
{
    "id": "laboratoire C",
    "value": [
        {
            "id": "10.22222"
        }
    ]
}]
```

On renomme donc les clés pour récupérer les noms explicites.  

- On change donc le nom de la clé `id` en `affiliation`
- On créé une clé `doi` où l'on récupère toutes les valeurs des champs `id`, eux mêmes contenus dans `value`.

```js
[exchange]
value = fix({ \
  affiliation: self.id, \
  doi: _.map(self.value, "id") \
})
```

:point_down: Résultat final :

```json
[{
    "affiliation": "laboratoire A",
    "doi": [
        "10.11111"
    ]
},
{
    "affiliation": "laboratoire B",
    "doi": [
        "10.11111",
        "10.22222"
    ]
},
{
    "affiliation": "laboratoire C",
    "doi": [
        "10.22222"
    ]
}]
```