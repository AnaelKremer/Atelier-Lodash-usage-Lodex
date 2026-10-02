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
  {"doi": "10.11111"},
  {"doi": "10.22222"}
]
```

```js
[assign]
path = date
value = fix(new Date())
```

```json
[
  {"doi": "10.11111",
   "date": "2025-10-05T16:09:28.554Z"},
  {"doi": "10.22222",
   "date": "2025-10-05T16:09:28.555Z"}
]
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
[
  {"doi": "10.11111",
   "date": "5 octobre 2025"},
  {"doi": "10.22222",
   "date": "5 octobre 2025"}
]
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
[
  {"doi": "10.11111",
   "date": "dimanche 5 octobre 2025 à 18:20:44"},
  {"doi": "10.22222",
   "date": "dimanche 5 octobre 2025 à 18:20:44"}
]
```

---

### Extraire l’année depuis une date de publication hétérogène

Dans certains jeux de données bibliographiques, le champ date de publication n’a malheureusement pas un format unique.
On peut par exemple rencontrer :
- "2023-10-25"
- "09-07-1986"
- "1986"
- "July 1986"  

Dans ce contexte, il n'est pas possible de s’appuyer sur un simple découpage de la chaîne avec `split` suivi d’un `first` ou `last` comme on peut le faire sur des données homogènes.  

Pour ce faire on cherchera directement l'année, soit **quatre chiffres consécutifs**, à l'aide d'une expression régulière et de `match`.  


```js
[assign]
path = value
value = get("value.publicationDate") \
  .thru(date => date.match(/[0-9]{4}/) ?? "aucune année trouvée") \
  .toString()
```

- `.match(/[0-9]{4}/)` cherche la première occurence de quatre chiffres consécutifs
- `?? "aucune année trouvée"` petite sécurité qui permet de retourner cette mention si aucune année n'est trouvée
- `.toString()` garantit une sortie homogène sous forme de chaîne (si certaines valeurs étaient en Number)

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

---

### Remplacer des valeurs en fonction d'un dictionnaire de correspondance

Lorsqu’il n’y a qu’un ou deux cas, on peut tout à fait transformer une valeur à l’aide d’un `thru` et d’une condition ternaire `(===, ? :)`.  

Mais dès que les cas se multiplient, ce type d’écriture devient :

- difficile à lire,

- difficile à maintenir,

- et potentiellement source d’erreurs.

Plutôt que d’écrire de longues fonctions conditionnelles (ou pire, de cumuler des `replace`), il est souvent plus simple et plus lisible d’utiliser un dictionnaire de correspondance.

Dans Lodex, ce rôle est parfaitement rempli par l’instruction `[env]`, qui permet de définir une table de valeurs réutilisable dans tout le pipeline.

👉 On remplace alors une logique conditionnelle complexe par une simple opération de lookup dans un dictionnaire.

On va donc commencer par définir notre table de correspondance à l’aide de `[env]`, sous la forme d’un dictionnaire clé → valeur :

```js
[env]
path = oaStatus
value = fix({ \
  "gold":   "voie dorée", \
  "green":  "voie verte", \
  "bronze": "voie bronze", \
  "hybrid": "voie hybride", \
  "diamond": "voie diamant", \
  "closed": "accès fermé" \
})
```

Dans le bloc `[env]`, le paramètre `path` permet de donner un nom à notre table de correspondance (ou dictionnaire).
Ce nom servira ensuite à y accéder n’importe où dans le pipeline.  

On utilise ensuite `fix` pour définir une valeur fixe, ici un objet *JavaScript*.
Chaque clé de cet objet correspond à une valeur présente dans le dataset, et chaque valeur associée correspond à la valeur de remplacement souhaitée.  

Il ne reste plus qu'à appliquer cette table à nos données :

```js
[
  {
    "value": {
      "entree": "gold"}},
  {
    "value": {
      "entree": "green"}},
  {
    "value": {
      "entree": ""}}
]
```

```js
[assign]
path = sortie
value = get("value.entree").thru(status => env("oaStatus")[status] ?? "statut inconnu")
```

- `get("value.entree")` on récupère la valeur brute du dataset
- `.thru(status => env("oaStatus")[status] || "statut inconnu")`
  - on interroge le dictionnaire défini dans `[env]`
  - si la clé existe, on récupère la valeur correspondante
  - sinon, on renvoie une valeur par défaut ("statut inconnu")


:point_down:

```js
[{
    "value": {
        "entree": "gold"},
    "sortie": "voie dorée"},
{
    "value": {
        "entree": "green"},
    "sortie": "voie verte"},
{
    "value": {
        "entree": ""},
    "sortie": "statut inconnu"
}]
```

💡 Si dans la colonne entrée on a un tableau avec plusieurs valeurs, et non plus une chaîne de caractères, un simple `map` en lieu et place de `thru` remplacera toutes les valeurs du tableau.

---

### Remplacer une valeur en fonction d’un dictionnaire de correspondance via un fichier CSV distant  

Si l'on doit utiliser une table de correspondance volumineuse, il est possible de remplacer la valeur d’un champ à partir d’un dictionnaire de correspondance (ou table d'équivalence) stocké dans un fichier CSV distant. Ce dernier doit être **hébergé en ligne** et **accessible publiquement via une URL** (sans authentification), idéalement en **accès direct** au fichier (lien de téléchargement / lien brut). 

Dans cet exemple, on souhaite remplacer les valeurs contenues dans une colonne *typeDeDocument* grâce au fichier CSV *testTable.csv*.

| typeDeDocument | 
|----------------|
| "Article"      |
| "Art"          |

**testTable.csv**  
documentType;homogenizedType  
Article;Journal Article  
Art ;Journal Article  

On crée un enrichissement dans Lodex et colle ce script :  

```js
[assign]
path = value
value = get("value.typeDeDocument")
; On sélectionne notre colonne Lodex à rechercher dans le dictionnaire

[combine]
path = value
primer = https://mapping-tables.daf.intra.inist.fr/testTable.csv
; URL vers le fichier CSV accessible via internet
default = n/a
; Valeur par défaut si aucune correspondance n’est trouvée dans le dictionnaire

[combine/URLStream]
; Sert à récupérer le fichier distant et à l'injecter dans le sous flux
path = false
; Sans cette ligne, EZS tente une lecture JSON.
; Avec false on lui indique de récupérer le fichier tel quel pour le parser ensuite

[combine/CSVParse]
; C'est ici que l'on parse notre fichier CSV
separator = fix(";")
; On précise le séparateur du fichier CSV pour qu'il soit correctement parsé

[combine/CSVObject]
; Cette instruction permet de transformer chaque ligne en objet
; Les clés de l'objet correspondent à la 1ère ligne du CSV (les titres des colonnes donc)


[combine/replace]
; Avec cette instruction on construit le dictionnaire sous forme d'objet clé / valeur (id et value)
path = id
; On définit l'identifiant de la ligne CSV
value = get("documentType")
; Colonne du CSV utilisée pour la correspondance (l'id sera donc par exemple "Article")

path = value
value = get("homogenizedType")
; On définit les valeurs à retourner en cas de correspondance ("Journal Article" dans notre exemple)

; Après combine on obtient donc des objets de type { "id": "Article", "value": "Journal Article" }

[exchange]
value = get("value")
; Ce dernier bloc est facultatif, il permet de ne récupérer que le résultat final à savoir la chaîne "Journal Article"
```

---

### Remplacer des valeurs en fonction d’un dictionnaire de correspondance via un fichier TSV distant  

Cet exemple illustre moins le changement de format (CSV → TSV) que la gestion des tableaux :  
comment appliquer une table de correspondance distante à chaque valeur d’une liste en appliquant un traitement itératif.  

On cherchera ici à verbaliser les données de la colonne *countryCode* à l'aide du fichier TSV *Pays_test.tsv*

| countryCode | 
|-------------|
| ["BE","FR"] |
| ["IT"]      |

**Pays_test.tsv**
From	To  
BE	Belgium  
FR	France  
IT	Italy  

```js
[assign]
path = value
value = get("value.countryCode")

[map]
path = value
; Sert à traiter individuellement chaque valeur de la liste (chaque code ici)

[map/replace]
path = tempkey
value = self()
; Création d'une clé temporaire pour stocker la valeur actuelle

[map/combine]
path = tempkey
primer = https://mapping-tables.daf.intra.inist.fr/Pays_test.tsv
; URL vers un fichier TSV accessible via internet
default = n/a

[map/combine/URLStream]
path = false

[map/combine/CSVParse]
separator = fix("\t")
; On précise ici que le séparateur est une tabulation

[map/combine/CSVObject]

[map/combine/replace]
path = id
value = get('From')
path = value
value = get('To')
; Une fois le TSV parsé en objets JSON, la valeur de "From" est transformé en clé
; et celle de "To" en valeur ({"From": "FR", "To": "France"})

; On récupère donc ceci [{"tempkey":{"id":"BE","value":"Belgium"}},{"tempkey":{"id":"FR","value":"France"}}]

[map/exchange]
value = get('tempkey.value')
; Extraction de la valeur finale
```
---

### Remplacer des valeurs en fonction d'un dictionnaire de correspondance via un fichier JSONL distant

Après avoir vu l’utilisation de fichiers TSV comme tables de correspondance, nous allons maintenant nous intéresser au format JSONL (JSON Lines).  

Le JSONL est particulièrement pratique dans l’écosystème Lodex, car :
- chaque ligne est un objet JSON autonome
- il est facilement lisible et traitable en streaming
- Lodex exporte naturellement ses données au format JSONL, **ce qui permet de réutiliser directement les données d’un autre Lodex comme table de correspondance.**  

**Cas d’usage : enrichir une notice contenant plusieurs RNSR avec les laboratoires associés**

On dispose d’un premier Lodex qui joue le rôle de dictionnaire :
- une colonne rnsr
- une colonne laboratoires

On exporte ces deux colonnes en JSONL (format d’export Lodex).
Chaque ligne du JSONL correspondra donc à une entrée du dictionnaire :  

**tableRnsrLaboratoire.jsonl**

{"rnsr":"200918450V","laboratoire":"Institut des sciences de la Terre Paris"}
{"rnsr":"199812866Y","laboratoire":"Laboratoire de géologie de l'Ecole Normale Supérieure"}

Dans notre jeu de données (un second Lodex), chaque notice peut contenir plusieurs RNSR (un tableau de valeurs).
Pour chaque notice, on voudra donc récupérer l’ensemble des laboratoires correspondant à chacun des RNSR.

| codesRnsr                   | 
|-----------------------------|
| ["200918450V"]              |
| ["199812866Y","200918450V"] |

:point_down:

```js
[assign]
path = value
value = get("value.codesRnsr")

[map]
path = value

[map/replace]
path = tmpkey
value = self()

[map/combine]
path = tmpkey
primer = https://mapping-tables.daf.intra.inist.fr/tableRnsrLaboratoire.jsonl
default = n/a

[map/combine/URLStream]
path = false

[map/combine/unpack]
; On utilise ici l'instruction unpack pour lire et traiter le fichier ligne par ligne

[map/combine/replace]
path = id
value = get('rnsr')
path = value
value = get("laboratoire")

[map/exchange]
value = get("tmpkey.value")
```

---

### Importer des données pré-calculées dans le dataset

Dans Lodex, certains traitements peuvent être exécutés **en dehors du dataset principal**, sous forme de **pré-calculs**.  
Ces pré-calculs sont stockés séparément et ne modifient pas directement les notices du jeu de données.  

Certains pré-calculs produisent des résultats globaux à l'échelle du corpus, d'autres sont **liés aux notices** et renvoient donc explicitement les identifiants (id) des documents concernés.  

Pour ce dernier cas, il est possible de récupérer ces résultats via un enrichissement afin de les injecter dans le dataset.  

Cela permet de les exploiter ensuite comme n'importe quelle autre colonne : 
- champs de ressource
- facettes
- graphiques divers  

**Structure des résultats de pré-calcul**  

Les résultats d'un pré-calcul sont des **objets JSON** ayant la forme suivante :  

```js
{
  "id": "uid:/iSZ2QaTPJ",
  "value": [
    "Security",
    "methodology",
    "mobile devices",
    "wide variety",
    "constraints"
  ],
  "lodexStamp":{
    "importedDate":"Sun Jan 25 2026",
    "precomputedId":"69767dc39bfb77f07861fffd",
    "webServiceURL":"https://data-homogenise.services.istex.fr/v1/homogenise?sid=lodex"}
}
```

Afin de récupérer les résultats stockés dans `value` on écrira ce script dans un enrichissement :  

```js
[assign]
path = value
value = get("id")
; On remplace la valeur courante par l’identifiant unique de chaque notice 
; C’est cette clé qui permet de retrouver les données pré-calculées correspondantes.

[expand]
path = value

[expand/precomputedSelect]
name = homogeniseKeywords
; On indique ici la source à interroger, soit le nom que l'on a attribué au pré-calcul
filter = fix({id: self.value})
; On filtre pour ne récupérer que la partie du JSON associée à la notice

[expand/aggregate]

[exchange]
value = get("value")
; Enfin, on ne conserve que les résultats stockés dans la clé value du pré-calcul
```

> [!NOTE]
> Il est possible de se passer du dernier bloc `[exchange]`.  
> Dans ce cas, l’enrichissement renverra les **objets JSON complets** issus du pré-calcul (tels que présentés plus haut),  
> et non uniquement la valeur extraite.

---

### Tronquer les titres issus d'un pré-calcul topRefExtract

Le pré-calcul topRefExtract produit initialement un graphe au format GEXF, utilisé pour la visualisation, ce graphe est ensuite traduit en JSON comme cet exemple :  

```JSON
    {
        "id": "https://doi.org/10.1098/rspb.2000.1204",
        "value": {
            "label": "Fragile transmission cycles of tick-borne encephalitis virus may be disrupted by predicted climate change",
            "viz$color": {
                "r": "0",
                "g": "0",
                "b": "255",
                "a": "0.8"
            },
            "viz$size": {
                "value": "50.0"
            },
            "targets": [
                {
                    "id": "https://doi.org/10.1126/science.289.5485.1763"
                }
            ]
        },
        "attributes": {
            "count": "0",
            "statut": "citant",
            "title": "Fragile transmission cycles of tick-borne encephalitis virus may be disrupted by predicted climate change"
        },
        "lodexStamp": {
            "importedDate": "Fri Jan 30 2026",
            "precomputedId": "697c63e6e9d4b0e5e69d3a6b",
            "webServiceURL": "https://data-topcitation.services.istex.fr/v1/topcitation?nbCitations=4&sid=lodex"
        }
    }
```

Pour des raisons d’affichage du graphe, les titres des documents contenus dans le champ `label` de `value` peuvent s’avérer trop longs.  
Afin d’améliorer la lisibilité du graphe, il est possible de tronquer ces titres.  

Le graphe est construit à partir des données contenues dans les clés `id` et `value` des objets issus du pré-calcul.  
Il est donc nécessaire de reconstruire un objet `value` similaire à celui produit par le pré-calcul, en ne modifiant que le champ `label` (le titre du document). Les autres propriétés de `value` sont reprises telles quelles afin de préserver la structure et les métriques utilisées par le graphe

Pour ce faire, on crée donc un enrichissement en mode avancé en sélectionnant comme source de données notre pré-calcul, puis on applique ce script :  

```js
[assign]
path = value
value = get("value.value").thru(obj => \
  ({ \
    ...obj, \
    label: ( \
      typeof obj.label === "string" && obj.label.length > 35 \
        ? _.truncate(obj.label, { length: 35 }) \
        : obj.label \
    ) \
  }) \
)
```

- `...obj` garantit que toutes les propriétés de l’objet value sont conservées
- `typeof obj.label === "string" && obj.label.length > 35` **si** la valeur de label est une chaîne de caractèreset **et si** sa longueur est supérieure à 35 caractères
- `_.truncate(obj.label, { length: 35 })` alors on applique une troncature (qui ajoute automatiquement `...` à la fin de la chaîne)

On obtient alors notre nouvel objet :

```JSON
    {
        "id": "https://doi.org/10.1098/rspb.2000.1204",
        "value": {
            "label": "Fragile transmission cycles of tick-borne encephalitis virus may be disrupted by predicted climate change",
            "viz$color": {
                "r": "0",
                "g": "0",
                "b": "255",
                "a": "0.8"
            },
            "viz$size": {
                "value": "50.0"
            },
            "targets": [
                {
                    "id": "https://doi.org/10.1126/science.289.5485.1763"
                }
            ]
        },
        "attributes": {
            "count": "0",
            "statut": "citant",
            "title": "Fragile transmission cycles of tick-borne encephalitis virus may be disrupted by predicted climate change"
        },
        "lodexStamp": {
            "importedDate": "Fri Jan 30 2026",
            "precomputedId": "697c5509e9d4b0e5e69d3a68",
            "webServiceURL": "https://data-topcitation.services.istex.fr/v1/topcitation?nbCitations=4&sid=lodex"
        },
        "enrichedValue": {
            "label": "Fragile transmission cycles of t...",
            "viz$color": {
                "r": "0",
                "g": "0",
                "b": "255",
                "a": "0.8"
            },
            "viz$size": {
                "value": "50.0"
            },
            "targets": [
                {
                    "id": "https://doi.org/10.1126/science.289.5485.1763"
                }
            ]
        }
    }
```

---

### Structurer une chaîne de caractères pour distinguer un libellé et une catégorie

Il peut être utile de restructurer une chaîne de caractères afin de faire apparaître explicitement une **catégorie** associée à un libellé.

Par exemple, dans un corpus, certaines valeurs peuvent se terminer par `Positif` ou `Négatif` :

```json
[
  { "entree": "Financement Négatif" },
  { "entree": "Reconnaissance Positif" },
  { "entree": "Politique RH Négatif" },
  { "entree": "International" }
]
```

On peut souhaiter transformer ces valeurs selon une structure commune de la forme :

```text
libellé : catégorie
```

On obtiendra ainsi :

```json
[
  {
    "entree": "Financement Négatif",
    "sortie": "Financement : négatif"
  },
  {
    "entree": "Reconnaissance Positif",
    "sortie": "Reconnaissance : positif"
  },
  {
    "entree": "Politique RH Négatif",
    "sortie": "Politique RH : négatif"
  },
  {
    "entree": "International",
    "sortie": "International : ni positif ni négatif"
  }
]
```

Cette structure est particulièrement utile lorsqu'un même champ contient à la fois une information descriptive et une catégorie exploitable dans un graphique ou une facette.


```ini
[assign]
path = sortie
value = get("value.entree") \
  .split(" ") \
  .thru(words => \
    _.includes(["Positif", "Négatif"], _.last(words)) \
      ? [ \
          _.initial(words).join(" "), \
          _.last(words) == "Positif" ? "positif" : "négatif" \
        ].join(" : ") \
      : [ \
          words.join(" "), \
          "ni positif ni négatif" \
        ].join(" : ") \
  )
```


Le script repose sur quelques fonctions Lodash très utiles pour manipuler des chaînes de caractères.

* `split(" ")` découpe la chaîne en un tableau de mots
* `thru()` permet d'appliquer un traitement personnalisé sur ce tableau
* `_.last()` récupère le dernier mot
* `_.includes()` vérifie si ce mot correspond à une catégorie connue
* `_.initial()` récupère tous les mots sauf le dernier afin de reconstruire le libellé


La première étape consiste à transformer la chaîne en tableau :

```js
get("value.entree").split(" ")
```

| Chaîne                 | Tableau obtenu                   |
| ---------------------- | -------------------------------- |
| `Financement Négatif`  | `["Financement", "Négatif"]`     |
| `Politique RH Négatif` | `["Politique", "RH", "Négatif"]` |


Le dernier mot est récupéré avec :

```js
_.last(words)
```

| Tableau                         | Dernier élément |
| ------------------------------- | --------------- |
| `["Financement", "Négatif"]`    | `Négatif`       |
| `["Reconnaissance", "Positif"]` | `Positif`       |

On vérifie ensuite si ce mot appartient aux catégories recherchées :

```js
_.includes(["Positif", "Négatif"], _.last(words))
```

Lorsque la catégorie est présente, tous les autres mots sont conservés :

```js
_.initial(words).join(" ")
```

| Tableau                          | Libellé obtenu |
| -------------------------------- | -------------- |
| `["Politique", "RH", "Négatif"]` | `Politique RH` |
| `["Financement", "Négatif"]`     | `Financement`  |

Le libellé et la catégorie sont ensuite réunis :

```js
[
  _.initial(words).join(" "),
  _.last(words) == "Positif" ? "positif" : "négatif"
].join(" : ")
```

Ce qui produit :

| Entrée                   | Sortie                     |
| ------------------------ | -------------------------- |
| `Financement Négatif`    | `Financement : négatif`    |
| `Reconnaissance Positif` | `Reconnaissance : positif` |

Si la chaîne ne se termine ni par `Positif` ni par `Négatif`, le libellé est conservé et une catégorie par défaut est ajoutée :

```js
[
  words.join(" "),
  "ni positif ni négatif"
].join(" : ")
```

| Entrée          | Sortie                                  |
| --------------- | --------------------------------------- |
| `International` | `International : ni positif ni négatif` |

Une fois les valeurs structurées, Vega-Lite peut récupérer indépendamment le libellé et la catégorie :

```js
split(datum._id, " : ")[0]
split(datum._id, " : ")[1]
```

Le premier élément peut être utilisé comme **libellé** du graphique, tandis que le second peut alimenter un **sélecteur**, une **couleur** ou une **facette**.

> **À adapter :** les valeurs recherchées (`Positif` et `Négatif`) sont sensibles à la casse. Si votre corpus contient `positif`, `NEGATIF` ou toute autre variante, adaptez simplement la liste utilisée dans `_.includes()`.

---

### Créer un champ booléen indiquant la présence de certaines valeurs

On peut créer un champ booléen indiquant si une ou plusieurs valeurs recherchées sont présentes dans une donnée.

Dans cet exemple, on cherche si `VAL A` ou `VAL B` est présente :

```ini
[assign]
path = contientAouB
value = get('value').castArray().some(v => ['VAL A', 'VAL B'].includes(v))
```

Le champ `contientAouB` contiendra alors `true` si au moins une des valeurs recherchées est présente, et `false` dans le cas contraire.

L'utilisation de `castArray()` permet d'appliquer le même traitement que la donnée soit une valeur simple ou un tableau.

Par exemple :

```json
["VAL A", "VAL C"]
```

renverra :

```json
true
```

alors que :

```json
["VAL C", "VAL D"]
```

renverra :

```json
false
```

---

#### Variante avec une expression régulière

On peut également vérifier si au moins une des valeurs respecte une expression régulière :

```ini
[assign]
path = contientPresqueMIL
value = get('value').castArray().some(v => String(v).search(/mil[aeiuo]+/i) !== -1)
```

Le champ `contientPresqueMIL` contiendra une valeur booléenne indiquant si au moins une des valeurs respecte l'expression régulière `/mil[aeiuo]+/i`.

Ici :

- `castArray()` permet de travailler indifféremment avec une valeur simple ou un tableau ;
- `some()` renvoie `true` dès qu'au moins un élément satisfait la condition ;
- `String(v)` s'assure que la valeur testée est une chaîne de caractères ;
- `search()` recherche le motif défini par l'expression régulière et renvoie `-1` lorsqu'il n'est pas trouvé.

---

### Regrouper des valeurs numériques par ordre de grandeur

Lorsque les valeurs d'un champ numérique présentent de très grands écarts, il peut être utile de les regrouper automatiquement par **ordre de grandeur**.

Cette transformation permet par exemple d'obtenir les catégories suivantes :

* `0`
* `1 à 9`
* `10 à 99`
* `100 à 999`
* `1 000 à 9 999`
* `10 000 à 99 999`

La borne inférieure de chaque intervalle correspond à une puissance de 10. La borne supérieure correspond à la puissance de 10 suivante, moins 1.

Par exemple, la valeur `357` appartient à l'intervalle `100 à 999`, tandis que la valeur `12 450` appartient à l'intervalle `10 000 à 99 999`.

```ini
[assign]
path = value
value = get("value.dataset") \
  .toNumber() \
  .thru(v => v === 0 \
    ? "0" \
    : Math.pow(10, Math.floor(Math.log10(v))) \
      + " à " \
      + (Math.pow(10, Math.floor(Math.log10(v)) + 1) - 1) \
  )
```

* `get("value.dataset")` récupère la valeur du champ `dataset`
* `.toNumber()` convertit la valeur en nombre
* `.thru()` permet d'appliquer un traitement personnalisé à cette valeur
* Si la valeur est égale à `0`, le script retourne directement `"0"`
* `Math.log10(v)` détermine l'ordre de grandeur de la valeur
* `Math.floor()` retient la puissance de 10 immédiatement inférieure
* `Math.pow()` permet de calculer les bornes inférieure et supérieure de l'intervalle

**Exemples**

| Valeur initiale | Valeur obtenue    |
| --------------: | :---------------- |
|             `0` | `0`               |
|             `4` | `1 à 9`           |
|            `27` | `10 à 99`         |
|           `357` | `100 à 999`       |
|         `2 450` | `1 000 à 9 999`   |
|        `12 450` | `10 000 à 99 999` |

> **À adapter :** remplacez `value.dataset` par le chemin du champ numérique que vous souhaitez traiter.

Ce type de regroupement est particulièrement utile pour créer des facettes ou des graphiques lorsque les valeurs sont très dispersées et que l'on souhaite les comparer sans définir manuellement chaque intervalle.

---

### Sélectionner certaines valeurs avant de lancer un enrichissement

Lorsqu'un enrichissement repose sur une colonne dont certaines valeurs sont absentes ou inutilisables, il est préférable de **ne pas envoyer ces valeurs au web service**.

Prenons l'exemple d'un corpus de 100 000 notices dont seulement 50 000 possèdent un DOI.  
Les autres notices contiennent une chaîne vide `""`.

Si l'on lance directement l'enrichissement sur la colonne `DOI`, les valeurs vides seront également envoyées au web service. Celui-ci effectuera donc inutilement des requêtes qui ne pourront pas produire de résultat.

On peut éviter ces traitements en utilisant l'instruction `[remove]` avant l'appel au web service :

```ini
[remove]
test = get("value.DOI").isEqual("")
```

`[remove]` retire ici du flux d'enrichissement toutes les notices dont le champ `DOI` contient une chaîne vide.

Seules les notices possédant un DOI poursuivent alors le traitement et sont envoyées au web service.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJ0aXRsZVwiOiBcIkFydGljbGUgQVwiLFxuICAgICAgXCJET0lcIjogXCIxMC4xMDAwL2FydGljbGUtYVwiXG4gICAgfVxuICB9LFxuICB7XG4gICAgXCJ2YWx1ZVwiOiB7XG4gICAgICBcInRpdGxlXCI6IFwiQXJ0aWNsZSBCXCIsXG4gICAgICBcIkRPSVwiOiBcIlwiXG4gICAgfVxuICB9LFxuICB7XG4gICAgXCJ2YWx1ZVwiOiB7XG4gICAgICBcInRpdGxlXCI6IFwiQXJ0aWNsZSBDXCIsXG4gICAgICBcIkRPSVwiOiBcIjEwLjEwMDAvYXJ0aWNsZS1jXCJcbiAgICB9XG4gIH0sXG4gIHtcbiAgICBcInZhbHVlXCI6IHtcbiAgICAgIFwidGl0bGVcIjogXCJBcnRpY2xlIERcIixcbiAgICAgIFwiRE9JXCI6IFwiXCJcbiAgICB9XG4gIH1cbl0iLCJzY3JpcHQiOiJbdXNlXVxucGx1Z2luID0gYmFzaWNzXG5cbltKU09OUGFyc2VdXG5zZXBhcmF0b3IgPSAqXG5cbltyZW1vdmVdXG50ZXN0ID0gZ2V0KFwidmFsdWUuRE9JXCIpLmlzRXF1YWwoXCJcIilcblxuW2RlYnVnXVxudGV4dCA9IGJlZm9yZSBnZW5lcmF0aW5nIGFuIGlkZW50aWZpZXIgcGVyIG9iamVjdFxuXG5bZHVtcF1cbmluZGVudCA9IHRydWUifQ==)

> [!NOTE]
> Dans le contexte d'un **enrichissement**, `[remove]` permet d'écarter certaines valeurs du traitement en cours.
>
> Dans le contexte d'un **loader**, la même instruction agit sur le flux de données et peut donc supprimer des lignes entières avant leur import dans Lodex.

Il est également possible d'inverser la condition avec le paramètre `reverse`.

```ini
[remove]
test = get("value.DOI").isEqual("")
reverse = true
```

Dans ce cas, le comportement est inversé : seules les notices répondant à la condition poursuivent le traitement.

Cette technique permet notamment de limiter les appels inutiles à un web service et peut être adaptée à d'autres conditions que la présence d'une chaîne vide.

---

### Effectuer un second enrichissement uniquement sur les données manquantes

Après un premier enrichissement, certaines ressources peuvent avoir obtenu une réponse tandis que d'autres n'ont retourné aucun résultat.

Par exemple, la colonne `PremierEnrich` contient un objet lorsque l'enrichissement a réussi et la valeur `"n/a"` lorsqu'aucune correspondance n'a été trouvée.

Si l'on souhaite effectuer un second enrichissement à partir d'une autre donnée, il est inutile de réinterroger les ressources pour lesquelles le premier enrichissement a déjà réussi.

On peut utiliser `[remove]` pour retirer du flux les ressources dont `PremierEnrich` contient déjà un objet :

```ini
[remove]
test = get("value.PremierEnrich").isObject()

[assign]
path = value
value = get("value.AdressePostale")
```

`isObject()` teste ici si le premier enrichissement a retourné un objet.

Les ressources pour lesquelles c'est le cas sont retirées du flux par `[remove]`. Il ne reste donc que celles pour lesquelles le premier enrichissement a échoué.

`[assign]` remplace ensuite `value` par la valeur de `AdressePostale`, qui pourra être envoyée au second service d'enrichissement.

Par exemple :

```json
[
  {
    "value": {
      "PremierEnrich": {"code": "UMR7503"},
      "AdressePostale": "Nancy"
    }
  },
  {
    "value": {
      "PremierEnrich": "n/a",
      "AdressePostale": "Paris"
    }
  }
]
```

donne :

```json
[
  {
    "value": "Paris"
  }
]
```

Seule la ressource qui n'avait pas obtenu de résultat lors du premier enrichissement poursuit donc le traitement.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJQcmVtaWVyRW5yaWNoXCI6IHtcImNvZGVcIjogXCJVTVI3NTAzXCJ9LFxuICAgICAgXCJBZHJlc3NlUG9zdGFsZVwiOiBcIk5hbmN5XCJcbiAgICB9XG4gIH0sXG4gIHtcbiAgICBcInZhbHVlXCI6IHtcbiAgICAgIFwiUHJlbWllckVucmljaFwiOiBcIm4vYVwiLFxuICAgICAgXCJBZHJlc3NlUG9zdGFsZVwiOiBcIlBhcmlzXCJcbiAgICB9XG4gIH1cbl0iLCJzY3JpcHQiOiJbdXNlXVxucGx1Z2luID0gYmFzaWNzXG5cbltKU09OUGFyc2VdXG5zZXBhcmF0b3IgPSAqXG5cbltyZW1vdmVdXG50ZXN0ID0gZ2V0KFwidmFsdWUuUHJlbWllckVucmljaFwiKS5pc09iamVjdCgpXG5cblthc3NpZ25dXG5wYXRoID0gdmFsdWVcbnZhbHVlID0gZ2V0KFwidmFsdWUuQWRyZXNzZVBvc3RhbGVcIilcblxuW2RlYnVnXVxudGV4dCA9IGJlZm9yZSBnZW5lcmF0aW5nIGFuIGlkZW50aWZpZXIgcGVyIG9iamVjdFxuXG5bZHVtcF1cbmluZGVudCA9IHRydWUifQ==)

> [!TIP]
> Cette méthode peut également être utilisée lorsqu'on ajoute de nouvelles données à un corpus déjà enrichi : elle permet de ne lancer l'enrichissement que sur les ressources qui ne disposent pas encore d'un résultat.

---

### Importer et traiter des annotations dans le dataset

Sur un corpus soumis à des annotations, il est possible d'importer les annotations dans le dataset afin de réaliser différents traitements sur les ressources annotées.

Il faut tout d'abord :

1. exporter les annotations depuis l'onglet **Annotations** ;
2. importer ce même fichier depuis l'onglet **Données** ;
3. sélectionner le loader **JSON - Fichier d'annotations**.

> [!NOTE]
> La colonne `annotations` n'est pas nécessairement visible dans Lodex, car toutes les ressources ne possèdent pas forcément une annotation. Les annotations sont néanmoins bien importées dans MongoDB.

#### Compter le nombre d'annotations d'une ressource

On peut commencer par déterminer quelles ressources ont été annotées et compter le nombre d'annotations associées à chacune d'elles.

Par exemple, pour créer un champ `nombreAnnotations` :

```ini
[assign]
path = value
value = get("value.annotations").size()
```

`size()` retourne ici le nombre d'éléments contenus dans le tableau `annotations`.

Cette information peut ensuite être utilisée pour effectuer différents traitements, notamment pour déterminer si une demande de suppression constitue l'unique annotation d'une ressource.

---

#### Identifier les ressources dont la suppression est demandée

On peut ensuite examiner les valeurs `proposedValue` des annotations afin d'identifier les ressources pour lesquelles un annotateur a demandé la suppression :

```ini
[assign]
path = value
value = get("value.annotations") \
  .flatMap("proposedValue") \
  .filter(val => val != null) \
  .map(val => (val === "OUI, je souhaite que cette ressource soit supprimée de ce jeu de données" && self.value.nombreAnnotations === 1) ? "Oui, la ressource peut être supprimée" : (val === "OUI, je souhaite que cette ressource soit supprimée de ce jeu de données" && self.value.nombreAnnotations > 1) ? "Attention il y a plusieurs annotations" : "") \
  .join("")
```

Le traitement :

- récupère les valeurs `proposedValue` avec `flatMap()` ;
- élimine les valeurs `null` avec `filter()` ;
- recherche la demande de suppression avec `map()` ;
- vérifie le nombre total d'annotations grâce à `self.value.nombreAnnotations` ;
- rassemble enfin le résultat avec `join("")`.

Deux situations sont distinguées :

- si la demande de suppression constitue **l'unique annotation**, la valeur `"Oui, la ressource peut être supprimée"` est produite ;
- si la ressource possède **plusieurs annotations**, la valeur `"Attention il y a plusieurs annotations"` est produite afin d'éviter une suppression automatique sans vérification.

Il est ensuite possible de filtrer les ressources portant la première valeur afin de supprimer uniquement celles pour lesquelles la demande de suppression est sans ambiguïté.

---

### Lancer un web service uniquement sur les lignes respectant certaines conditions

Lors d'un enrichissement, il peut être utile de n'appeler un web service que pour les ressources qui respectent certaines conditions.

Dans cet exemple, le web service **TEEFT** est utilisé pour extraire des termes à partir du champ `abstract`, mais uniquement lorsque :

- la ressource possède un résumé ;
- la langue indiquée dans `language` est `fr` ou `en`.

Les ressources qui ne respectent pas ces conditions reçoivent la valeur `"n/a"`.

On commence par vérifier que la langue est bien `fr` ou `en` :

```ini
[use]
plugin = basics

[swing]
test = get("value.language").thru(lang => _.includes(["fr","en"], lang))
reverse = true
size = 10

[swing/assign]
path = value
value = n/a
```

`thru()` permet ici de tester si la langue appartient à la liste `["fr","en"]`.

Avec `reverse = true`, les ressources dont la langue n'appartient pas à cette liste sont dirigées vers `[swing/assign]` et prennent la valeur `"n/a"`.

On vérifie ensuite que le résumé n'est pas vide :

```ini
[swing]
test = get("value.abstract").isEmpty()
size = 10

[swing/assign]
path = value
value = n/a
```

Les ressources possédant une langue autorisée et un résumé peuvent alors être orientées vers le service correspondant à leur langue.

Pour les résumés en français :

```ini
[swing]
test = get("value.language").isEqual("fr")
size = 10

[swing/assign]
path = value
value = update("value.abstract", (item) => { try { return JSON.parse(item); } catch { return item; } }).get("value.abstract")

[swing/URLConnect]
url = https://terms-extraction.services.istex.fr/v2/teeft/fr
timeout = 3600000
noerror = false
retries = 2
```

Pour les résumés en anglais :

```ini
[swing]
test = get("value.language").isEqual("en")
size = 10

[swing/assign]
path = value
value = update("value.abstract", (item) => { try { return JSON.parse(item); } catch { return item; } }).get("value.abstract")

[swing/URLConnect]
url = https://terms-extraction.services.istex.fr/v2/teeft/en
timeout = 3600000
noerror = false
retries = 2
```

Le script complet permet ainsi de distinguer trois situations :

- langue `fr` et résumé présent → appel du service TEEFT français ;
- langue `en` et résumé présent → appel du service TEEFT anglais ;
- autre langue ou résumé vide → aucun appel à TEEFT et valeur `"n/a"`.

> [!TIP]
> Ce principe n'est pas propre à TEEFT. L'enchaînement `[swing]` / `[swing/assign]` peut être utilisé pour conditionner l'appel à d'autres web services à partir d'une ou plusieurs valeurs du dataset. Il permet notamment d'éviter des requêtes inutiles et d'orienter les données vers des services différents selon leur contenu.

[Tester le filtrage conditionnel dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJsYW5ndWFnZVwiOiBcImZyXCIsXG4gICAgICBcImFic3RyYWN0XCI6IFwiVW4gculzdW3pXCJcbiAgICB9XG4gIH0sXG4gIHtcbiAgICBcInZhbHVlXCI6IHtcbiAgICAgIFwibGFuZ3VhZ2VcIjogXCJkZVwiLFxuICAgICAgXCJhYnN0cmFjdFwiOiBcIkVpbiBBYnN0cmFjdFwiXG4gICAgfVxuICB9LFxuICB7XG4gICAgXCJ2YWx1ZVwiOiB7XG4gICAgICBcImxhbmd1YWdlXCI6IFwiZW5cIixcbiAgICAgIFwiYWJzdHJhY3RcIjogXCJcIlxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6Ilt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW3N3aW5nXVxudGVzdCA9IGdldChcInZhbHVlLmxhbmd1YWdlXCIpLnRocnUobGFuZyA9PiBfLmluY2x1ZGVzKFtcImZyXCIsXCJlblwiXSwgbGFuZykpXG5yZXZlcnNlID0gdHJ1ZVxuc2l6ZSA9IDEwXG5cbltzd2luZy9hc3NpZ25dXG5wYXRoID0gdmFsdWVcbnZhbHVlID0gbi9hXG5cbltzd2luZ11cbnRlc3QgPSBnZXQoXCJ2YWx1ZS5hYnN0cmFjdFwiKS5pc0VtcHR5KClcbnNpemUgPSAxMFxuXG5bc3dpbmcvYXNzaWduXVxucGF0aCA9IHZhbHVlXG52YWx1ZSA9IG4vYVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZVxuICAifQ==)

---

### Créer des URI distinctes pour chaque ressource

L'instruction `[identify]` permet d'ajouter automatiquement un identifiant distinct à chaque ressource.

```ini
[identify]
```

Par exemple :

```json
[
  {
    "value": "Article A"
  },
  {
    "value": "Article B"
  }
]
```

devient :

```json
[
  {
    "value": "Article A",
    "uri": "uid:/..."
  },
  {
    "value": "Article B",
    "uri": "uid:/..."
  }
]
```

Chaque ressource reçoit ainsi une propriété `uri` contenant un identifiant de type `uid:/...`.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjogXCJBcnRpY2xlIEFcIlxuICB9LFxuICB7XG4gICAgXCJ2YWx1ZVwiOiBcIkFydGljbGUgQlwiXG4gIH0sXG4gIHtcbiAgICBcInZhbHVlXCI6IFwiQXJ0aWNsZSBDXCJcbiAgfVxuXSIsInNjcmlwdCI6Ilt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW2lkZW50aWZ5XVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZVxuICAifQ==)

> [!WARNING]
> Dans Lodex, les valeurs de `uri` doivent être distinctes. Elles servent à identifier les ressources ; des URI dupliquées peuvent empêcher certaines opérations.

---

### Ajouter un identifiant pérenne de type ARK

```ini
[assign]
path = value
value = get('value.uri')

[expand]
size = 10
path=value

[expand/URLConnect]
url = https://ark-tools.services.istex.fr/v1/67375/stamp?subpublisher=XXX
```

Ces instructions vont attribuer un identifiant pérenne à chaque ligne.

`XXX` est à remplacer par le code du `subpublisher` approprié et enregistré dans le registre de l'Inist : http://inist-registry.ark.inist.fr/

> [!WARNING]
> Chaque identifiant ARK créé par ce web service est créé uniquement une seule fois. S'il n'est pas utilisé, l'identifiant est perdu.

> [!TIP]
> Pour remplacer l'UID attribué automatiquement par Lodex, il suffit de créer un enrichissement nommé `uri`.

---

### Remplacer des UID par des identifiants ARK

```ini
[assign]
path = value
value = get('value.uri')

[swing]
test = get('value').startsWith('uid:/')

[swing/expand]
size = 10
path = value

[swing/expand/URLConnect]
url = https://ark-tools.services.istex.fr/v1/67375/stamp?subpublisher=XXX
```

Ces instructions vont attribuer un identifiant pérenne à chaque ligne si et seulement si elle possède un UID comme URI.

`XXX` est à remplacer par le code du `subpublisher` approprié et enregistré dans le registre de l'Inist : http://inist-registry.ark.inist.fr/

> [!WARNING]
> Chaque identifiant ARK créé par ce web service est créé uniquement une seule fois. S'il n'est pas utilisé, l'identifiant est perdu.

> [!TIP]
> Pour remplacer l'UID attribué automatiquement par Lodex, il suffit de créer un enrichissement nommé `uri`.

---

### Créer une colonne en s’assurant que toutes les lignes contiennent un tableau (même vide)

```ini
[assign]
path = value
value = get("value.Entités nommées (Unitex).placeName", [])
```

Ces instructions créent une colonne à partir du sous-champ `placeName` de la colonne `Entités nommées (Unitex)`. S'il n’y a pas de valeur, le tableau sera vide.

---

### Créer un objet par défaut pour toutes les lignes qui contiennent la valeur `n/a`

Il peut être utile de remplacer une valeur `n/a` par un objet structuré afin de conserver un format homogène dans une colonne.

```ini
[assign]
path = value
value = get("value.loterre")

[swing]
test = get('value').isEqual('n/a')

[swing/replace]
path = value.information
value = Aucune réponse
```

Pour toutes les lignes contenant `n/a`, on obtient alors :

```json
{
  "information": "Aucune réponse"
}
```

Les lignes contenant déjà une autre valeur sont conservées telles quelles.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJsb3RlcnJlXCI6IFwibi9hXCJcbiAgICB9XG4gIH0sXG4gIHtcbiAgICBcInZhbHVlXCI6IHtcbiAgICAgIFwibG90ZXJyZVwiOiB7XG4gICAgICAgIFwiaWRcIjogXCJodHRwOi8vZGF0YS5sb3RlcnJlLmZyL2FyazovNjczNzUvLi4uXCIsXG4gICAgICAgIFwibGFiZWxcIjogXCJTY2llbmNlIG91dmVydGVcIlxuICAgICAgfVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6Ilt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW2Fzc2lnbl1cbnBhdGggPSB2YWx1ZVxudmFsdWUgPSBnZXQoXCJ2YWx1ZS5sb3RlcnJlXCIpXG5cbltzd2luZ11cbnRlc3QgPSBnZXQoJ3ZhbHVlJykuaXNFcXVhbCgnbi9hJylcblxuW3N3aW5nL3JlcGxhY2VdXG5wYXRoID0gdmFsdWUuaW5mb3JtYXRpb25cbnZhbHVlID0gQXVjdW5lIHLpcG9uc2VcblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlIn0=)

---

### Récupérer dans une liste d’objets deux valeurs simples

Il est possible de simplifier une liste d’objets en ne conservant que certaines propriétés de chaque objet.

```ini
[assign]
path = value
value = get('value.concepts loterre').map(item => _.pick(item, ['prefLabel@en', 'about']))
```

Ces instructions créent une colonne à partir d’une liste d’objets en simplifiant chaque objet pour ne garder que deux champs différents.

Dans cet exemple, seules les propriétés `prefLabel@en` et `about` sont conservées. Les autres champs de chaque objet sont ignorés.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJjb25jZXB0cyBsb3RlcnJlXCI6IFtcbiAgICAgICAge1xuICAgICAgICAgIFwicHJlZkxhYmVsQGVuXCI6IFwiT3BlbiBzY2llbmNlXCIsXG4gICAgICAgICAgXCJwcmVmTGFiZWxAZnJcIjogXCJTY2llbmNlIG91dmVydGVcIixcbiAgICAgICAgICBcImFib3V0XCI6IFwiaHR0cDovL2RhdGEubG90ZXJyZS5mci9hcms6LzY3Mzc1L0FCQ1wiLFxuICAgICAgICAgIFwidHlwZVwiOiBcIkNvbmNlcHRcIlxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJwcmVmTGFiZWxAZW5cIjogXCJSZXNlYXJjaCBkYXRhXCIsXG4gICAgICAgICAgXCJwcmVmTGFiZWxAZnJcIjogXCJEb25u6WVzIGRlIHJlY2hlcmNoZVwiLFxuICAgICAgICAgIFwiYWJvdXRcIjogXCJodHRwOi8vZGF0YS5sb3RlcnJlLmZyL2FyazovNjczNzUvREVGXCIsXG4gICAgICAgICAgXCJ0eXBlXCI6IFwiQ29uY2VwdFwiXG4gICAgICAgIH1cbiAgICAgIF1cbiAgICB9XG4gIH1cbl0iLCJzY3JpcHQiOiJbdXNlXVxucGx1Z2luID0gYmFzaWNzXG5cbltKU09OUGFyc2VdXG5zZXBhcmF0b3IgPSAqXG5cblthc3NpZ25dXG5wYXRoID0gdmFsdWVcbnZhbHVlID0gZ2V0KCd2YWx1ZS5jb25jZXB0cyBsb3RlcnJlJykubWFwKGl0ZW0gPT4gXy5waWNrKGl0ZW0sIFsncHJlZkxhYmVsQGVuJywgJ2Fib3V0J10pKVxuXG5bZHVtcF1cbmluZGVudCA9IHRydWUifQ==)

---

### Récupérer une valeur dans une liste de listes d’objets

Lorsque des données sont imbriquées dans plusieurs listes d’objets, on peut enchaîner `map` et `flatten` pour atteindre les valeurs recherchées.

Par exemple, chaque auteur peut posséder une liste d’affiliations, chaque affiliation contenant elle-même une adresse.

```ini
[replace]
path = lesAdresses
value = get('auteurs').map('affiliations').flatten().map('address')
```

Le premier `map('affiliations')` récupère les listes d’affiliations de chaque auteur.  
`flatten()` les rassemble ensuite dans une seule liste.  
Le dernier `map('address')` récupère enfin l’adresse de chaque affiliation.

On obtient par exemple :

```json
{
  "lesAdresses": [
    "Nancy",
    "Paris",
    "Strasbourg"
  ]
}
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwiYXV0ZXVyc1wiOiBbXG4gICAgICB7XG4gICAgICAgIFwibmFtZVwiOiBcIkFsaWNlIE1hcnRpblwiLFxuICAgICAgICBcImFmZmlsaWF0aW9uc1wiOiBbXG4gICAgICAgICAge1xuICAgICAgICAgICAgXCJuYW1lXCI6IFwiVW5pdmVyc2l06SBkZSBMb3JyYWluZVwiLFxuICAgICAgICAgICAgXCJhZGRyZXNzXCI6IFwiTmFuY3lcIlxuICAgICAgICAgIH0sXG4gICAgICAgICAge1xuICAgICAgICAgICAgXCJuYW1lXCI6IFwiQ05SU1wiLFxuICAgICAgICAgICAgXCJhZGRyZXNzXCI6IFwiUGFyaXNcIlxuICAgICAgICAgIH1cbiAgICAgICAgXVxuICAgICAgfSxcbiAgICAgIHtcbiAgICAgICAgXCJuYW1lXCI6IFwiSmVhbiBEdXBvbnRcIixcbiAgICAgICAgXCJhZmZpbGlhdGlvbnNcIjogW1xuICAgICAgICAgIHtcbiAgICAgICAgICAgIFwibmFtZVwiOiBcIlVuaXZlcnNpdOkgZGUgU3RyYXNib3VyZ1wiLFxuICAgICAgICAgICAgXCJhZGRyZXNzXCI6IFwiU3RyYXNib3VyZ1wiXG4gICAgICAgICAgfVxuICAgICAgICBdXG4gICAgICB9XG4gICAgXVxuICB9XG5dIiwic2NyaXB0IjoiW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBsZXNBZHJlc3Nlc1xudmFsdWUgPSBnZXQoJ2F1dGV1cnMnKS5tYXAoJ2FmZmlsaWF0aW9ucycpLmZsYXR0ZW4oKS5tYXAoJ2FkZHJlc3MnKVxuXG5bZHVtcF1cbmluZGVudCA9IHRydWUifQ==)

---

### Transformer une liste d’objets en liste de valeurs simples

Il est possible de transformer chaque objet d'une liste en une valeur simple construite à partir de plusieurs de ses propriétés.

Cette méthode peut notamment être utile pour **préparer des données hiérarchiques**, en associant dans une même valeur un rang, un niveau ou un code à une autre information.

```ini
[assign]
path = value
value = get("value.Catégories").map((categorie) => `${categorie.rang}-${categorie.code.value}`)
```

Par exemple :

```json
[
  {
    "rang": 1,
    "code": {
      "value": "SHS"
    }
  },
  {
    "rang": 2,
    "code": {
      "value": "INFO"
    }
  }
]
```

devient :

```json
[
  "1-SHS",
  "2-INFO"
]
```

Chaque objet est ainsi transformé en une chaîne composée ici des propriétés `rang` et `code.value`. Les autres propriétés éventuelles de l'objet sont ignorées.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJDYXTpZ29yaWVzXCI6IFtcbiAgICAgICAge1xuICAgICAgICAgIFwicmFuZ1wiOiAxLFxuICAgICAgICAgIFwiY29kZVwiOiB7XG4gICAgICAgICAgICBcInZhbHVlXCI6IFwiU0hTXCJcbiAgICAgICAgICB9XG4gICAgICAgIH0sXG4gICAgICAgIHtcbiAgICAgICAgICBcInJhbmdcIjogMixcbiAgICAgICAgICBcImNvZGVcIjoge1xuICAgICAgICAgICAgXCJ2YWx1ZVwiOiBcIklORk9cIlxuICAgICAgICAgIH1cbiAgICAgICAgfVxuICAgICAgXVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6Ilt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW3JlcGxhY2VdXG5wYXRoID0gdmFsdWVcbnZhbHVlID0gZ2V0KFwidmFsdWUuQ2F06Wdvcmllc1wiKS5tYXAoKGNhdGVnb3JpZSkgPT4gYCR7Y2F0ZWdvcmllLnJhbmd9LSR7Y2F0ZWdvcmllLmNvZGUudmFsdWV9YClcblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlIn0=)

#### Variante : générer automatiquement le rang

Si le rang n'est pas présent dans les données, il peut être calculé directement à partir de la position de chaque objet dans le tableau :

```ini
[assign]
path = value
value = get("value.Domaines").map((domaine, i) => `${i+1} - ${domaine.code.value}`)
```

Ici, `i` correspond à l'index de l'élément dans le tableau. Comme les index commencent à `0`, on utilise `i + 1` pour obtenir une numérotation commençant à `1`.

On peut ainsi obtenir :

```json
[
  "1 - SHS",
  "2 - INFO"
]
```

Cette variante est particulièrement pratique lorsque **l'ordre des éléments du tableau porte lui-même l'information hiérarchique**.

---

### Exclure d’un tableau les valeurs commençant par une chaîne donnée

Il est possible de filtrer un tableau de chaînes de caractères en fonction du début de chaque valeur.

Par exemple, un champ contenant des affiliations peut également contenir par erreur des adresses e-mail. On peut supprimer toutes les valeurs commençant par `E-mail` :

```ini
[assign]
path = value
value = get("value.affiliations").filter(value => !value.startsWith("E-mail"))
```

Par exemple :

```json
[
  "Université de Lorraine",
  "E-mail : jean.dupont@example.fr",
  "CNRS",
  "E-mail : alice.martin@example.fr",
  "Université de Strasbourg"
]
```

devient :

```json
[
  "Université de Lorraine",
  "CNRS",
  "Université de Strasbourg"
]
```

`filter()` parcourt les valeurs du tableau et `startsWith("E-mail")` vérifie si chaque chaîne commence par `E-mail`.

L'opérateur `!` inverse le résultat du test : seules les valeurs **ne commençant pas** par `E-mail` sont conservées.

> `startsWith()` teste uniquement le début de la chaîne. Une valeur contenant `E-mail` à un autre emplacement ne sera donc pas exclue.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJhZmZpbGlhdGlvbnNcIjogW1xuICAgICAgICBcIlVuaXZlcnNpdOkgZGUgTG9ycmFpbmVcIixcbiAgICAgICAgXCJFLW1haWwgOiBqZWFuLmR1cG9udEBleGFtcGxlLmZyXCIsXG4gICAgICAgIFwiQ05SU1wiLFxuICAgICAgICBcIkUtbWFpbCA6IGFsaWNlLm1hcnRpbkBleGFtcGxlLmZyXCIsXG4gICAgICAgIFwiVW5pdmVyc2l06SBkZSBTdHJhc2JvdXJnXCJcbiAgICAgIF1cbiAgICB9XG4gIH1cbl0iLCJzY3JpcHQiOiJbdXNlXVxucGx1Z2luID0gYmFzaWNzXG5cbltKU09OUGFyc2VdXG5zZXBhcmF0b3IgPSAqXG5cbltyZXBsYWNlXVxucGF0aCA9IHZhbHVlXG52YWx1ZSA9IGdldChcInZhbHVlLmFmZmlsaWF0aW9uc1wiKS5maWx0ZXIodmFsdWUgPT4gIXZhbHVlLnN0YXJ0c1dpdGgoXCJFLW1haWxcIikpXG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZSJ9)

---

### Inverser des éléments dans des chaînes de caractères

Il est possible de combiner plusieurs fonctions pour modifier des chaînes de caractères contenues dans un tableau.

Par exemple, pour transformer des noms d'auteurs écrits sous la forme `Nom, Prénom` en `Prénom, Nom` :

```ini
[assign]
path = value
value = get("value.auteurs").map(x => x.split(", ").reverse().join(", "))
```

Par exemple :

```json
[
  "Durand, Jacques",
  "Dubois, Daniel"
]
```

devient :

```json
[
  "Jacques, Durand",
  "Daniel, Dubois"
]
```

Le traitement est appliqué à chaque valeur du tableau avec `map()` :

- `split(", ")` transforme chaque chaîne en tableau ;
- `reverse()` inverse l'ordre des éléments ;
- `join(", ")` reconstruit la chaîne de caractères.

Cette combinaison peut notamment être utile pour **normaliser la forme des noms de personnes** dans un jeu de données.

### Convertir une chaîne JSON en objet et inversement

Il arrive qu'une structure JSON soit stockée dans Lodex sous la forme d'une simple chaîne de caractères. À l'inverse, il peut être nécessaire de transformer un objet ou un tableau en chaîne JSON.

JavaScript fournit deux fonctions complémentaires pour effectuer ces conversions :

- `JSON.parse` : chaîne JSON → objet ou tableau
- `JSON.stringify` : objet ou tableau → chaîne JSON

#### Transformer une chaîne JSON en objet avec `JSON.parse`

Par exemple :

```json
{
  "jsonValue": "{\"title\":\"Mon document\",\"year\":2026,\"openAccess\":true}"
}
```

La valeur de `jsonValue` ressemble à un objet JSON, mais il s'agit en réalité d'une chaîne de caractères.

```ini
[assign]
path = value
value = get('value.jsonValue').thru(JSON.parse)
```

On obtient :

```json
{
  "title": "Mon document",
  "year": 2026,
  "openAccess": true
}
```

`get('value.jsonValue')` récupère la chaîne, puis `thru(JSON.parse)` la désérialise pour obtenir une véritable structure exploitable.

> La chaîne doit contenir du JSON valide. Dans le cas contraire, `JSON.parse` provoquera une erreur.

[Tester JSON.parse dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJqc29uVmFsdWVcIjogXCJ7XFxcInRpdGxlXFxcIjpcXFwiTW9uIGRvY3VtZW50XFxcIixcXFwieWVhclxcXCI6MjAyNixcXFwib3BlbkFjY2Vzc1xcXCI6dHJ1ZX1cIlxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6Ilt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW3JlcGxhY2VdXG5wYXRoID0gdmFsdWVcbnZhbHVlID0gZ2V0KCd2YWx1ZS5qc29uVmFsdWUnKS50aHJ1KEpTT04ucGFyc2UpXG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZSJ9)

#### Transformer un objet en chaîne JSON avec `JSON.stringify`

À l'inverse, si `jsonValue` contient un véritable objet :

```json
{
  "jsonValue": {
    "title": "Mon document",
    "year": 2026,
    "openAccess": true,
    "authors": [
      "Alice Martin",
      "Jean Dupont"
    ]
  }
}
```

on peut le sérialiser :

```ini
[assign]
path = value
value = get('value.jsonValue').thru(JSON.stringify)
```

La structure devient alors une chaîne JSON :

```text
{"title":"Mon document","year":2026,"openAccess":true,"authors":["Alice Martin","Jean Dupont"]}
```

Cette fois, `thru(JSON.stringify)` effectue donc l'opération inverse de `JSON.parse`.

En résumé :

```text
objet / tableau
      ↓
 JSON.stringify
      ↓
  chaîne JSON
      ↓
   JSON.parse
      ↓
objet / tableau
```

[Tester JSON.stringify dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJqc29uVmFsdWVcIjoge1xuICAgICAgICBcInRpdGxlXCI6IFwiTW9uIGRvY3VtZW50XCIsXG4gICAgICAgIFwieWVhclwiOiAyMDI2LFxuICAgICAgICBcIm9wZW5BY2Nlc3NcIjogdHJ1ZSxcbiAgICAgICAgXCJhdXRob3JzXCI6IFtcbiAgICAgICAgICBcIkFsaWNlIE1hcnRpblwiLFxuICAgICAgICAgIFwiSmVhbiBEdXBvbnRcIlxuICAgICAgICBdXG4gICAgICB9XG4gICAgfVxuICB9XG5dIiwic2NyaXB0IjoiW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSB2YWx1ZVxudmFsdWUgPSBnZXQoJ3ZhbHVlLmpzb25WYWx1ZScpLnRocnUoSlNPTi5zdHJpbmdpZnkpXG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZSJ9)

---

### Créer un objet avec des propriétés issues de différentes colonnes

Il est possible de construire un nouvel objet en regroupant des valeurs provenant de plusieurs colonnes.

Par exemple, on dispose de deux champs contenant chacun des informations différentes :

```json
{
  "istex": {
    "ark": "ark:/67375/WNG-NVGLRQV3-C"
  },
  "conditor": {
    "sourceUidChain": "!hal$hal-03566649!"
  }
}
```

On peut créer un nouvel objet avec `fix({})` :

```ini
[assign]
path = value
value = fix({ \
  ark: self.value.istex.ark, \
  sourceUidChain: self.value.conditor.sourceUidChain \
})
```

On obtient :

```json
{
  "ark": "ark:/67375/WNG-NVGLRQV3-C",
  "sourceUidChain": "!hal$hal-03566649!"
}
```

`fix({})` permet ici de définir la structure du nouvel objet.

Les propriétés `ark` et `sourceUidChain` correspondent aux noms que l'on souhaite donner aux propriétés du nouvel objet, tandis que `self.value.istex.ark` et `self.value.conditor.sourceUidChain` permettent de récupérer dynamiquement les valeurs dans les colonnes d'origine.

Cette méthode est notamment utile pour **regrouper dans une même structure des informations provenant de plusieurs champs ou de plusieurs enrichissements**.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJpc3RleFwiOiB7XG4gICAgICAgIFwiYXJrXCI6IFwiYXJrOi82NzM3NS9XTkctTlZHTFJRVjMtQ1wiXG4gICAgICB9LFxuICAgICAgXCJjb25kaXRvclwiOiB7XG4gICAgICAgIFwic291cmNlVWlkQ2hhaW5cIjogXCIhaGFsJGhhbC0wMzU2NjY0OSFcIlxuICAgICAgfVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6Ilt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW3JlcGxhY2VdXG5wYXRoID0gdmFsdWVcbnZhbHVlID0gZml4KHsgXFxcbiAgYXJrOiBzZWxmLnZhbHVlLmlzdGV4LmFyaywgXFxcbiAgc291cmNlVWlkQ2hhaW46IHNlbGYudmFsdWUuY29uZGl0b3Iuc291cmNlVWlkQ2hhaW4gXFxcbn0pXG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZSJ9)

---

### Réduire plusieurs informations à une seule valeur

Dans certains cas, plusieurs informations permettent de déterminer une seule valeur synthétique.

On souhaite ici déterminer si un document est disponible dans **Istex**, dans **Conditor**, dans les deux bases ou dans aucune.

```ini
[assign]
path = value
value = fix({ \
  ark: self.value.EnrichIstex.ark, \
  sourceUidChain: self.value.EnrichConditor['business/sourceUidChain'] \
}).reduce((result, value, index, collection) => { \
  if(collection.ark && !collection.sourceUidChain){return "Istex"} \
  if(collection.ark && collection.sourceUidChain){return "Istex & Conditor"} \
  if(!collection.ark && collection.sourceUidChain){return "Conditor"} \
  return result \
}, "Aucune Base")
```

`reduce` permet ici de ramener les informations contenues dans l'objet à une seule valeur :

- si `ark` existe mais pas `sourceUidChain` → `"Istex"`
- si `ark` et `sourceUidChain` existent → `"Istex & Conditor"`
- si `sourceUidChain` existe mais pas `ark` → `"Conditor"`
- si aucune des deux valeurs n'existe → `"Aucune Base"`

`"Aucune Base"` est la valeur initiale de l'accumulateur `result`. Elle est conservée lorsqu'aucune des conditions précédentes n'est satisfaite.

Cette approche permet ainsi de **produire une information synthétique à partir de plusieurs propriétés d'un objet**.

---

### Réduire une collection à un tableau de valeurs spécifiques

Dans cet exemple, on dispose d'une matrice dans laquelle chaque tableau contient un **RNSR** en première position, suivi d'un ou plusieurs instituts.

Par exemple :

```json
[
  ["200918450V", "INSU", "INSHS"],
  ["199812866Y", "INSB"],
  ["201220345A", "INSHS"],
  ["200512345B", "INS2I", "INSIS"],
  ["201998765C", "INEE", "INSHS"]
]
```

On souhaite récupérer uniquement les RNSR associés à l'institut `INSHS`.

```ini
[assign]
path = value
value = get("value.MatriceRnsrInstituts").reduce((result, item) => { \
  if (item.slice(1).some(element => element === "INSHS")) { \
    result.push(item[0]) \
  } \
  return result \
}, [])
```

On obtient :

```json
[
  "200918450V",
  "201220345A",
  "201998765C"
]
```

`reduce` parcourt ici chacun des tableaux de la matrice et construit progressivement un nouveau tableau.

`[]` correspond à l'accumulateur de départ : le tableau dans lequel seront ajoutés les RNSR correspondant à notre condition.

```js
item.slice(1)
```

permet d'ignorer le premier élément du tableau, qui contient le RNSR, afin de ne parcourir que les instituts.

```js
.some(element => element === "INSHS")
```

vérifie si au moins l'un de ces instituts correspond à `INSHS`.

Lorsque la condition est satisfaite :

```js
result.push(item[0])
```

ajoute à l'accumulateur le premier élément du tableau, c'est-à-dire le RNSR correspondant.

Ce cas montre ainsi comment utiliser `reduce` pour **parcourir une structure complexe, sélectionner certains éléments et construire progressivement un nouveau tableau de résultats**.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJNYXRyaWNlUm5zckluc3RpdHV0c1wiOiBbXG4gICAgICAgIFtcIjIwMDkxODQ1MFZcIiwgXCJJTlNVXCIsIFwiSU5TSFNcIl0sXG4gICAgICAgIFtcIjE5OTgxMjg2NllcIiwgXCJJTlNCXCJdLFxuICAgICAgICBbXCIyMDEyMjAzNDVBXCIsIFwiSU5TSFNcIl0sXG4gICAgICAgIFtcIjIwMDUxMjM0NUJcIiwgXCJJTlMySVwiLCBcIklOU0lTXCJdLFxuICAgICAgICBbXCIyMDE5OTg3NjVDXCIsIFwiSU5FRVwiLCBcIklOU0hTXCJdXG4gICAgICBdXG4gICAgfVxuICB9XG5dIiwic2NyaXB0IjoiW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSB2YWx1ZVxudmFsdWUgPSBnZXQoXCJ2YWx1ZS5NYXRyaWNlUm5zckluc3RpdHV0c1wiKS5yZWR1Y2UoKHJlc3VsdCwgaXRlbSkgPT4geyBcXFxuICBpZiAoaXRlbS5zbGljZSgxKS5zb21lKGVsZW1lbnQgPT4gZWxlbWVudCA9PT0gXCJJTlNIU1wiKSkgeyBcXFxuICAgIHJlc3VsdC5wdXNoKGl0ZW1bMF0pIFxcXG4gIH0gXFxcbiAgcmV0dXJuIHJlc3VsdCBcXFxufSwgW10pXG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZSJ9)


## Transformations globales (dans le cadre d'un loader)

### Numéroter les lignes de son dataset
  
Il peut être utile, pour retrouver certaines données, d'avoir un corpus numéroté. Pour cela on peut utiliser la fonction `uniqueId` et customiser notre identifiant pour mieux s'adapter à nos besoin.  

Ici par exemple, pour un corpus de donnée de plusieurs centaines de milliers de documents, on crée un champ *line* à 6 chiffres :

```json
[ 
  {"doi": "10.11111"},
  {"doi": "10.22222"},
  {"doi": "10.33333"},
  {"doi": "10.44444"},
  {"doi": "10.99999"}
]
```

```js
[assign]
path = line
value = fix("").uniqueId().padStart(6, "0")
```

:point_down:

```json
[
  {"doi": "10.11111",
   "line": "000001"},
  {"doi": "10.22222",
   "line": "000002"},
  {"doi": "10.33333",
   "line": "000003"},
  {"doi": "10.44444",
   "line": "000004"},
  {"doi": "10.99999",
   "line": "000005"}
]
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

- `mapKeys` pour itérer sur toutes les clés d'un objet,
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

### Regrouper des colonnes

Dans certains jeux de données, des informations peuvent être réparties en plusieurs colonnes.  
Par exemple lorsqu'un auteur a plusieurs affiliation, on va retrouver chaque affiliation dans une colonne dédiée (`"Affiliation", "Affiliation (2)"... `).  
Ce type de structure apparaît souvent dans les exports où le nombre d’occurrences n’est pas connu à l’avance. 

Si l'on peut concaténer ces colonnes, il faudrait alors écrire une opération concat pour chaque champ supplémentaire. Cette approche devient rapidement lourde et suppose surtout de connaître à l’avance le nombre maximal de colonnes présentes dans le jeu de données.  

Or, dans la pratique, ce nombre peut varier selon les notices et les sources de données. Il est donc préférable d’utiliser une méthode plus générique permettant de regrouper automatiquement toutes les colonnes correspondant à un même motif (par exemple Affiliation, Affiliation(2), Affiliation(3), etc.), puis de regrouper leurs valeurs dans un seul tableau.  

Le script suivant illustre cette approche en regroupant automatiquement les colonnes Affiliation et Ind Structure, quel que soit le nombre de variantes présentes dans le dataset.  

```JSON
[{
  "Title": "Article",
  "Affiliation": "CNRS",
  "Affiliation(2)": "Université de Bordeaux",
  "Affiliation(78)": "CEA",  
  "Ind Structure": 123,
  "Ind Structure(2)": 456,
  "Ind Structure(78)": 789
}]
```

```js
[exchange]
value = self().thru(data => \
  _.assign({}, \
    _.omitBy(data, (v, k) => /^(Affiliation|Ind Structure)\(\d+\)$/.test(k)), \
    { \
      Affiliation: _.chain(data) \
        .pickBy((v, k) => /^Affiliation(\(\d+\))?$/.test(k)) \
        .values() \
        .without(null, undefined) \
        .value(), \
      "Ind Structure": _.chain(data) \
        .pickBy((v, k) => /^Ind Structure(\(\d+\))?$/.test(k)) \
        .values() \
        .without(null, undefined) \
        .value() \
    } \
  ) \
)
```

On obtient désormais :

```JSON
[{
    "Title": "Article",
    "Affiliation": [
        "CNRS",
        "Université de Bordeaux",
        "CEA"
    ],
    "Ind Structure": [
        123,
        456,
        789
    ]
}]
```

`value = self().thru(data => ...)`  
On récupère l’objet courant (`self()`) et on le passe à une fonction de transformation.  

`_.assign({}, ...)`  
Reconstruit l’objet final après les transformations suivantes : 

`_.omitBy(data, (v, k) => /^(Affiliation|Ind Structure)\(\d+\)$/.test(k))`  
Va supprimer, après les instructions suivantes, les colonnes intermédiaires comme Affiliation(2), Affiliation(3) à partir d'une regex :  

`_.pickBy((v, k) => /^Affiliation(\(\d+\))?$/.test(k))`  
Cette instruction sélectionne toutes les clés correspondant au motif :

- Affiliation
- Affiliation(2)
- Affiliation(3)
- ...

`.values()`  
On récupère ici les valeurs de toutes ces colonnes.  

`.without(null, undefined)`  
Supprime les valeurs vides.  

`.value()`  
Clotûre la chaîne de traitements débutée par `.chain()`

---

### Éclater une colonne multivaluée en plusieurs colonnes

Dans certains jeux de données, une information peut être regroupée dans une seule colonne sous forme de liste de valeurs.

C’est par exemple le cas lorsqu’un champ `Affiliation` contient plusieurs affiliations dans un tableau :

```JSON
[{"Affiliation": ["CNRS","Université de Bordeaux","INRAE"]}]
```

Dans certains cas, on peut souhaiter faire l’opération inverse du regroupement de colonnes : créer automatiquement autant de colonnes qu’il y a de valeurs dans le champ multivalué.

On veut donc avoir :

```JSON
[{
  "Affiliation": "CNRS",
  "Affiliation(2)": "Université de Bordeaux",
  "Affiliation(3)": "INRAE"
}]
```

Prenons cet exemple avec plus de cas de figure :

```JSON
[{"Affiliation":["CNRS","Université de Bordeaux","INRAE"]},
{"Affiliation":["CNRS"]},
{"Affiliation":["Université de Lorraine","CNRS","Université de Lorraine"]},
{"Affiliation":[]},
{"Affiliation":["CNRS",null,"","INRIA"]}]
```

Le script suivant permet de créer automatiquement les colonnes `Affiliation`, `Affiliation(2)`, `Affiliation(3)`, etc., sans connaître à l’avance le nombre de valeurs présentes dans le champ.

```js
[exchange]
value = self().thru(data => \
  _.assign({}, \
    _.omit(data, "Affiliation"), \
    _.chain(data.Affiliation || []) \
      .castArray() \
      .without(null, undefined, "") \
      .map((value, index) => [ \
        index === 0 ? "Affiliation" : "Affiliation(" + (index + 1) + ")", \
        value \
      ]) \
      .fromPairs() \
      .value() \
  ) \
)
```

`value = self().thru(data => ...)`
On récupère l’objet courant (`self()`) et on le passe à une fonction de transformation.

`_.assign({}, ...)`
Reconstruit l’objet final après les transformations suivantes.

`_.omit(data, "Affiliation")`
Conserve toutes les colonnes de la notice, à l’exception de `Affiliation`. Cette colonne est supprimée temporairement afin d’être reconstruite sous la forme de plusieurs colonnes (`Affiliation`, `Affiliation(2)`, `Affiliation(3)`, etc.).

`_.chain(data.Affiliation || [])`
Démarre une chaîne de traitements Lodash sur le contenu de la colonne `Affiliation`. Le `|| []` permet d’utiliser un tableau vide si le champ `Affiliation` est absent.

`.castArray()`
S’assure que la valeur traitée est bien un tableau. Si la valeur n’est pas un tableau, elle est transformée en tableau contenant une seule valeur.

`.without(null, undefined, "")`
Supprime les valeurs vides.

`.map((value, index) => [...])`
Parcourt chaque valeur du tableau et crée une paire composée :

* du nom de la colonne à créer
* de la valeur à placer dans cette colonne

Par exemple :

```js
[
  "CNRS",
  "Université de Bordeaux",
  "INRAE"
]
```

devient :

```js
[
  ["Affiliation", "CNRS"],
  ["Affiliation(2)", "Université de Bordeaux"],
  ["Affiliation(3)", "INRAE"]
]
```

`index === 0 ? "Affiliation" : "Affiliation(" + (index + 1) + ")"`
Permet de nommer la première colonne `Affiliation`, puis les suivantes `Affiliation(2)`, `Affiliation(3)`, etc.

`.fromPairs()`
Transforme la liste de paires en objet :


```JSON
{
  "Affiliation": "CNRS",
  "Affiliation(2)": "Université de Bordeaux",
  "Affiliation(3)": "INRAE"
}
```

`.value()`
Clôture la chaîne de traitements débutée par `.chain()`.

On obtient donc :

```JSON
[{
    "Affiliation": "CNRS",
    "Affiliation(2)": "Université de Bordeaux",
    "Affiliation(3)": "INRAE"
},
{
    "Affiliation": "CNRS"
},
{
    "Affiliation": "Université de Lorraine",
    "Affiliation(2)": "CNRS"
},
{},
{
    "Affiliation": "CNRS",
    "Affiliation(2)": "INRIA"
}]
```

---

#### Variante avec une chaîne de caractères séparée par `;`

Dans d’autres jeux de données, les valeurs ne sont pas stockées dans un tableau, mais dans une chaîne de caractères contenant un séparateur.

Par exemple :

```JSON
[{
  "Affiliation": "CNRS;Université de Bordeaux;INRAE"
}]
```

Dans ce cas, il suffit de remplacer `.castArray()` par `.split(";")`.

```js
[exchange]
value = self().thru(data => \
  _.assign({}, \
    _.omit(data, "Affiliation"), \
    _.chain(data.Affiliation || "") \
      .split(";") \
      .map(value => _.trim(value)) \
      .without(null, undefined, "") \
      .map((value, index) => [ \
        index === 0 ? "Affiliation" : "Affiliation(" + (index + 1) + ")", \
        value \
      ]) \
      .fromPairs() \
      .value() \
  ) \
)
```

On peut ajouter :
`.map(value => _.trim(value))`
Supprime les espaces inutiles avant et après chaque valeur.

Le reste du traitement est identique : une fois la chaîne transformée en liste de valeurs, le script crée automatiquement les colonnes `Affiliation`, `Affiliation(2)`, `Affiliation(3)`, etc.

---


### Normaliser un dataset issu d'un parsing XML

Il peut arriver que certaines sources de données produisent des champs **structurés sous forme d’objets**, plutôt que des valeurs simples.  
C’est notamment le cas lorsque des données issues de formats riches (comme le XML) sont converties en JSON : le contenu textuel et les métadonnées associées (langue, type, etc.) sont alors séparés.  

On peut par exemple rencontrer des champs de la forme :  

```JSON
{
  "source": "HAL",
  "title": {
    "type": "string",
    "xmlLang": "fr",
    "value": "bla bla bla"
  },
  "year": {
    "type": "number",
    "value": 2023
  }
}
```

Dans Lodex, ce niveau de structuration est souvent inutile, il ajoute de la complexité et augmente le volume de données à stocker.  
On peut facilement restructurer l'intégralité du jeu de données via ce script :

```js
[exchange]
value = self().mapValues(v => _.get(v, 'value', v))
```

- On utilise `[exchange]` pour remplacer tout l'objet d'origine
- On pointe sur `self` pour manipuler l'objet complet
- `mapValues` va appliquer une transformation à **chaque valeur**, clé par clé
- `_.get(v, 'value', v)` 
  - Si la valeur du champ est un objet et qu’il possède une propriété *value*, on retourne la valeur associée
  - Si la valeur du champ est déjà une valeur simple (string, nombre) on la retourne telle quelle

On obtient alors :

```JSON
{
  "source": "HAL",
  "title": "bla bla bla",
  "year": 2023
}
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

### Générer automatiquement un lot de colonnes à partir d’une même règle de transformation

Dans les exemples précédents, nous avons vu qu’il était possible de modifier plusieurs colonnes en une seule opération, en appliquant une transformation uniquement à certains champs.  

Ici, le besoin est légèrement différent : on ne souhaite pas modifier les colonnes existantes, mais créer de nouvelles colonnes à partir de plusieurs champs d’origine.    
Les colonnes initiales sont conservées, et une nouvelle version transformée est ajoutée pour chacune d’elles.  

Cette méthode est utile lorsqu’on veut garder la donnée source intacte tout en produisant des champs nettoyés ou adaptés à un traitement ultérieur.  

Prenons un exemple inspiré d’un cas réel dans lequel une même opération de nettoyage devait être appliquée à plus d’une dizaine de colonnes.
Afin de simplifier la démonstration, nous limiterons ici l’exemple à seulement trois champs, mais le principe reste exactement le même quel que soit le nombre de colonnes concernées:  

```json
[
  {
    "fr_domainAllCodeLabel": "Mathématiques\\,Informatique",
    "authIdHasPrimaryStructure": "CNRS\\,Université de Lorraine",
    "title_s": "Titre\\, de l'article"
  }
]
```

On souhaite créer pour chacun de ces champs une nouvelle colonne préfixée par `Nettoyage`, dans laquelle les séquences `\,` sont remplacées par le caractère `#`.  

Le script suivant permet de générer automatiquement ces nouvelles colonnes :  

```js
[exchange]
value = self().thru(data => \
  _.assign({}, \
    data, \
    _.chain([ \
      "fr_domainAllCodeLabel", \
      "authIdHasPrimaryStructure", \
      "title_s" \
    ]) \
      .map(field => [ \
        "Nettoyage" + field, \
        (_.get(data, field, "") || "").replace(/\\,/g, "#") \
      ]) \
      .fromPairs() \
      .value() \
  ) \
)
```

Résultat :

```json
[
  {
    "fr_domainAllCodeLabel": "Mathématiques\\,Informatique",
    "authIdHasPrimaryStructure": "CNRS\\,Université de Lorraine",
    "title_s": "Titre\\, de l'article",
    "Nettoyagefr_domainAllCodeLabel": "Mathématiques#Informatique",
    "NettoyageauthIdHasPrimaryStructure": "CNRS#Université de Lorraine",
    "Nettoyagetitle_s": "Titre# de l'article"
  }
]
```

`_.chain([...])`

Démarre une chaîne de traitements sur la liste des champs à transformer :

```js
[
  "fr_domainAllCodeLabel",
  "authIdHasPrimaryStructure",
  "title_s"
]
```

`_.map(field => [...])`

Parcourt chaque nom de champ et crée une paire contenant :

* le nom de la nouvelle colonne
* la valeur transformée.

Par exemple :

```js
[
  [
    "Nettoyagefr_domainAllCodeLabel",
    "Mathématiques#Informatique"
  ],
  [
    "NettoyageauthIdHasPrimaryStructure",
    "CNRS#Université de Lorraine"
  ]
]
```

`"Nettoyage" + field`

Construit dynamiquement le nom du nouveau champ en ajoutant le préfixe `Nettoyage` devant le nom du champ d'origine.

```txt
fr_domainAllCodeLabel     → Nettoyagefr_domainAllCodeLabel
authIdHasPrimaryStructure → NettoyageauthIdHasPrimaryStructure
title_s                   → Nettoyagetitle_s
```

`_.get(data, field, "")`

Récupère la valeur du champ courant.

Si le champ n'existe pas dans la notice, une chaîne vide est utilisée à la place afin d'éviter les erreurs.

`replace(/\\,/g, "#")`

Applique la transformation souhaitée à la valeur.

Dans cet exemple, toutes les occurrences de `\,` sont remplacées par le caractère `#`.

`.fromPairs()`

Transforme la liste de paires produite par `map()` en objet.

Par exemple :

```js
[
  ["Nettoyagefr_domainAllCodeLabel", "Mathématiques#Informatique"],
  ["NettoyageauthIdHasPrimaryStructure", "CNRS#Université de Lorraine"]
]
```

devient :

```json
{
  "Nettoyagefr_domainAllCodeLabel": "Mathématiques#Informatique",
  "NettoyageauthIdHasPrimaryStructure": "CNRS#Université de Lorraine"
}
```

Cette approche est particulièrement utile lorsqu'il faut créer un grand nombre de colonnes calculées à partir d'une même règle de transformation. Il suffit alors d'ajouter ou de retirer des noms de champs dans la liste passée à `_.chain()`.

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
- On crée une clé `doi` où l'on récupère toutes les valeurs des champs `id`, eux mêmes contenus dans `value`.

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