# Comprendre les types de données et les opérateurs

## Les différents types de données

> “Bad programmers worry about the code. Good programmers worry about data structures and their relationships.”  
> — *Linus Torvalds*, créateur de Linux et Git

Comme nous l’avons vu, les fonctions Lodash s’appliquent à différents **types de données**, que l’on peut regrouper en deux grandes familles :  
→ **les types primitifs** et **les types objets**.


### Types de données primitifs

- **`Number`**  
  Représente des nombres, entiers (integer) ou à virgule flottante (floating).  
  *Exemple :* `age = 42`

- **`String`**  
  Chaîne de caractères, représente une séquence de caractères délimitée par des guillemets (`" "`).  
  *Exemples :* `"John"` ou `"42"` (⚠ ici, `"42"` est une chaîne, pas un nombre)

- **`Boolean`**  
  Représente une valeur booléenne, vraie ou fausse (`true` ou `false`).  
  *Exemple :* `is_OA = false`

- **`Null`**  
  Représente une valeur nulle ou l'absence de valeur. En général null est utilisé pour indiquer de façon intentionnelle et explicite que la variable n'a pas de valeur.  
  *Exemple :* `abstract = null` (la notice n’a pas de résumé)

- **`Undefined`**  
  Indique une valeur **non définie**.  
  En pratique, il est déconseillé d’assigner soi-même `undefined` en raison de sa proximité conceptuelle avec `null`.

JavaScript renvoi aussi "undefined" si l'on essaye d’accéder a une propriété qui n'existe pas. Si par exemple on veut extraire les valeurs associées à une clé, mais que cette dernière n'existe pas dans toutes les notices alors le résultat sera "undefined" pour les objets n'ayant pas cette clé.

---

### Types de données objets

Les **types objets** sont des structures de données plus complexes.  
Ils permettent de **regrouper des valeurs** et **des fonctionnalités liées** dans une seule entité.

- **`Object`**  
  En JavaScript, un objet est une **collection non ordonnée de paires clé-valeur**.  
  Chaque paire est appelée une **propriété** de l’objet.

  - Les **clés** (ou *noms de propriétés*) sont des chaînes de caractères (ou des symboles).
  - Les **valeurs** peuvent être de **n’importe quel type de données** : primitives ou objets.

##### Exemple : 
L'objet ci-dessous a pour valeurs des chaînes de caractères, des nombres, un booléen,un objet (address) ainsi qu'un tableau (children).

```json
{
  "name": "John",
  "age": 42,
  "address": {
    "number": 1,
    "street": "Main Street",
    "city": "Nowhere"
  },
  "married": true,
  "children": ["Bob", "Alice"]
}
```

- **`Array`**  
  Un tableau représente une **collection ordonnée d’éléments** indexés numériquement.  
  Chaque élément est accessible via son **indice** (ex: `array[0]`, `array[1]`, etc.).

  Dans l’exemple ci-dessus :
  - `"Bob"` est à l’index `[0]`
  - `"Alice"` est à l’index `[1]`

🔀 Ces types peuvent **s’imbriquer** :
- Des **tableaux dans des objets**
- Des **objets dans des tableaux**
- Des **objets dans des objets dans des objets**…
- Ou encore des **tableaux de tableaux** (matrices)

---

## Les opérateurs JavaScript

Il existe une multitude d'opérateurs en Javascript très bien détaillés [ici](https://developer.mozilla.org/fr/docs/Web/JavaScript/Guide/Expressions_and_operators).
Ceux-ci permettent d'exprimer des conditions, de comparer ou évaluer des données, nous ne verrons ici que les opérateurs indispensables dans Lodex.

| Symbole     | Signification           | Exemple                                                 |
|-------------|-------------------------|----------------------------------------------------------|
| `=`         | Affectation              | `value = 3`                                                  |
| `==`        | Égalité                  | `3 == "3"` // `true`                                     |
| `===`       | Égalité stricte          | `3 === "3"` // `false`                                   |
| `!`         | NON logique (NOT)        | `!true` // `false`                                       |
| `!=`        | Inégalité                | `3 != "3"` // `false`                                    |
| `!==`       | Inégalité stricte        | `3 !== "3"` // `true`                                    |
| `&&`        | ET logique (AND)         | `a=true` et `b=false` → `a && b` // `false`              |
| `\|\|`       | OU logique (OR)          | `a \|\| b`                                              |
| `? :`       | Conditionnel (if-else)   | `age = 18` → `age >= 18 ? "majeur" : "mineur"`           |
| `>`         | Supérieur à              | `a = 2`, `b = 5` → `a > b` // `false`                    |
| `>=`        | Supérieur ou égal à      | `a >= b` // `false`                                      |
| `<`         | Inférieur à              | `a < b` // `true`                                        |
| `<=`        | Inférieur ou égal à      | `a <= b` // `true`                                       |


---

### Affectation (`=`)
Permet d'assigner une valeur à une variable.  
Ex. : `value = 3` affecte la valeur `3` à la variable `value`.  
💡 Dans un loader par exemple, on **affecte** un nom à une nouvelle colonne par `path = maNouvelleColonne`


### Égalité (`==`) vs Égalité stricte (`===`)
- `==` compare **la valeur**, en faisant des conversions automatiques si nécessaire (`3 == "3"` renvoie `true`).
- `===` compare **la valeur et le type** : (`3 === "3"` renvoie `false` car l’un est un nombre, l’autre une chaîne).

> [!WARNING] 
> Evitez d'utiliser `==` qui compare une égalité abstraite. L'opérateur effectue implicitement une coercition de type et peut retourner des bizarreries comme false == '0' // true, où sont considérés comme égaux un booléen  > et une châine de caractères...


### Négation (`!`) et inégalités
- `!` inverse une valeur booléenne : `!true` → `false`
- `!=` vérifie que deux valeurs sont différentes (tolère la conversion)
- `!==` vérifie qu’elles sont différentes **et de types différents**

> [!WARNING] 
> Même remarque que pour `==`, il y a une coercition implicite en utilsant `!=`.

> [!TIP]
> On peut également utiliser `!` pour inverser des fonctions.
> Si l'on veut savoir si un champ **n'est pas vide**, il n'existe pas de fonction dédiée. Il faudrait utiliser la fonction `_.isEmpty`, puis inverser le résultat. Ce qui revient à dire "si c'est faux alors c'est vrai" et 
> inversement... Manipulation qui est tout sauf logique et surtout incompréhensible pour quiconque lirait ce code. Heureusement Lodash permet **d'inverser des fonctions**  de façon très simple, en ajoutant `!` devant la fonction. 
> Ainsi il est facilement possible d'inverser tout type de test retournant un booléen avec `!_.isEmpty`, `!_.isEqual`, `!_.includes`, `!_.startsWith` etc.

### Logique booléenne : `&&` et `||`
- `&&` (ET logique) retourne `true` **si les deux conditions sont vraies**
- `||` (OU logique) retourne `true` **si au moins une condition est vraie**

> [!WARNING] 
> Attention de ne pas écrire `&` et `|` qui ne sont pas des opérateurs logiques mais "bit à bit" et qui renvoient donc les nombres 0 ou 1.

### Ternaire (`condition ? valeurSiVrai : valeurSiFaux`)
Une façon compacte d’écrire un `if/else` :    
```js
thru(age => age >= 18 ? "majeur" : "mineur");
```

Au lieu de :
```js
thru(function(age) {
  if (age >= 18) {
    return "majeur";
  } else {
    return "mineur";
  }
});
```

#### Comparateurs numériques
- `>` : supérieur à  
- `>=` : supérieur ou égal à  
- `<` : inférieur à  
- `<=` : inférieur ou égal à

---

## Les fonctions fléchées `=>`ou anonymes

Une expression de fonction fléchée est une syntaxe plus compacte pour écrire des fonctions classiques, peut être plus claires pour des débutants, mais beaucoup plus verbeuses.
Cette syntaxe est omniprésente dans Lodex quand on utilise des fonctions comme map, thru, filter etc.

```js
.thru(function(number) {
  return number + 1;
})
```
Cette fonction qui permet d'ajouter 1 au nombre reçu s'écrit de façon plus courte par :

```js
.thru(number => number + 1)
```

> [!IMPORTANT]
> Dès lors que l'on utilise une fonction fléchée via =>, on **n’écrit plus du Lodash pur, mais du JavaScript**.
> Cela implique donc que certaines fonctions doivent être appelées **différemment**, en fonction de leur nature (native JS ou Lodash).

*Par exemple :*

✅ Il est tout à fait possible d’écrire : `.map(item => item.trim())`

Ici, `.trim()` est une méthode native de JavaScript sur les chaînes de caractères, on peut l’appeler de la façon habituelle dans Lodex.
La fonction applique `.trim()` à chaque élément du tableau (via `.map()`) pour retirer les espaces et caractères invisibles en début et fin de chaîne.

❌ En revanche, si on tente : `.map(item => item.startCase())`

Cela échoue, car `startCase()` n’existe pas en JavaScript *(fonction qui met en majuscule la première lettre de chaque mot d’une chaîne)*. Étant une fonction propre à Lodash, elle doit être appelée avec le préfixe `_` pour que Lodex sache qu’il faut utiliser Lodash. 

✅ Il faut écrire : `.map(item => _.startCase(item))`

> [!NOTE]
> Outre l'ajout de `_` avant la fonction, on remarque également que la valeur à traiter est passée en argument entre parenthèses. *item* dan notre exemple.

👉 [Chapitre suivant](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/03-syntaxe-et-bonnes-pratiques.md)