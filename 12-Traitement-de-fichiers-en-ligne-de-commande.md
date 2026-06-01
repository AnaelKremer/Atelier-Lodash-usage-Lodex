# Principe

Sous **Linux**, ou sous **Windows** via **WSL**, il est possible de manipuler des fichiers et d'automatiser des traitements directement depuis un terminal.

Cette approche est couramment utilisée avec **EZS** pour exécuter des loaders, enchaîner des transformations, concaténer des fichiers ou générer de nouveaux jeux de données.

Les exemples qui suivent présentent les principales commandes **Bash** utiles pour travailler avec des loaders **EZS** et des fichiers de données.

Certaines opérations de traitement avancées, comme `aggregate`, ne peuvent être réalisées qu'au moment de l'import dans **Lodex**, sur les données en cours de traitement.    
Une fois les données importées, il n'est plus possible d'agréger directement de nouvelles données avec celles déjà présentes dans l'instance. Pour reconstruire une agrégation, il faudrait généralement exporter le jeu de données, y ajouter les nouvelles données, puis réimporter l'ensemble après traitement.  

La maîtrise d'**EZS** en ligne de commande permet d'effectuer ces opérations rapidement, sans avoir à multiplier les exports et réimports dans **Lodex**. Elle facilite également la création de pipelines de traitement, l'agrégation de plusieurs fichiers et l'automatisation de tâches récurrentes.

## Exécuter un loader sur un fichier CSV

L’exemple suivant montre comment appliquer un loader à un fichier en CSV puis à restituer la sortie en JSONl.

Ce loader :

- lit un fichier CSV
- transforme chaque ligne en objet
- normalise les noms de colonnes
- ajoute une URI si nécessaire
- supprime les doublons
- ajoute des métadonnées d’import

### Exemple de commande

```bash
cat donnees.csv | ezs loader-csv.ini > donnees.jsonl
```

Cette commande est composée de trois parties :

```bash
cat donnees.csv
```

Lit le contenu du fichier `donnees.csv` et l'envoie vers la sortie standard du terminal.

```bash
|
```

Le caractère `|`, appelé **pipe**, transmet le résultat de la commande précédente à la commande suivante.

```bash
ezs loader-csv.ini
```

Exécute le loader `loader-csv.ini` sur les données reçues en entrée.

```bash
>
```

Le caractère `>` redirige la sortie de la commande vers un fichier.

```bash
donnees.jsonl
```

Nom du fichier de sortie qui contiendra le résultat du traitement.

### Résultat du traitement 

On part du fichier `donnees.csv` constitué de ceci :  

```csv
Nom & prénom,Établissement d'origine
Jean Dupont,Université de Lorraine
Marie Martin,CNRS
```

On lui applique ce loader :  

```ini
append = pack
label = csv

[use]
plugin = basics
plugin = analytics

[CSVParse]

[CSVObject]

[exchange]
value = self().mapKeys((value, key) => \
    _.camelCase( \
        _.deburr(key) \
          .replace(/[^a-zA-Z0-9_ ]/g, "") \
    ) \
)

[swing]
test = pick(['URI', 'uri']).pickBy(_.identity).isEmpty()
[swing/identify]

[dedupe]
ignore = true

[OBJFlatten]
separator = fix('.')
reverse = true
safe = true

[assign]
path = importDateTime
value = fix(new Date()).thru(d => \
  new Intl.DateTimeFormat("fr-FR", { \
    timeZone: "Europe/Paris", \
    year: "numeric", \
    month: "long", \
    day: "numeric", \
    hour: "2-digit", \
    minute: "2-digit", \
    second: "2-digit" \
  }).format(d) \
)

path = uri
value = get('uri').trim()
```

👉 On obtient le fichier `donnees.jsonl` structuré comme cela : 

```json
{"uri":"uid:/bQ2AbWDkp","nomPrenom":"Jean Dupont","etablissementDorigine":"Université de Lorraine","importDateTime":"1 juin 2026 à 07:46:26"}
{"uri":"uid:/3CgXYRG6P","nomPrenom":"Marie Martin","etablissementDorigine":"CNRS","importDateTime":"1 juin 2026 à 07:46:26"}
```

---

## Concaténer deux fichiers JSONL

Le format JSONL étant constitué d'une ressource par ligne, il est possible de fusionner plusieurs fichiers simplement avec la commande `cat`.

### Exemple

`janvier.jsonl`

```json
{"nom":"Jean Dupont"}
{"nom":"Marie Martin"}
```

`fevrier.jsonl`

```json
{"nom":"Paul Durand"}
{"nom":"Julie Bernard"}
```

### Commande

```bash
cat janvier.jsonl fevrier.jsonl > donnees.jsonl
```

### Résultat


```json
{"nom":"Jean Dupont"}
{"nom":"Marie Martin"}
{"nom":"Paul Durand"}
{"nom":"Julie Bernard"}
```

La commande :

```bash
cat janvier.jsonl fevrier.jsonl
```

lit successivement les deux fichiers et affiche leur contenu à la suite.

Le caractère `>` permet de rediriger le résultat vers un nouveau fichier.

---

## Concaténer plusieurs fichiers et appliquer un loader

Lorsque plusieurs fichiers suivent une convention de nommage commune, il est possible d'utiliser le caractère `*` pour les sélectionner automatiquement.

### Exemple

```text
fin2026-03.jsonl
fin2026-04.jsonl
fin2026-05.jsonl
fin2026-06.jsonl
```

La commande suivante concatène tous les fichiers commençant par `fin` puis applique un loader :

```bash
cat fin*.jsonl | ezs loaderAggregate.ini > agregation.jsonl
```

Cette commande :

1. sélectionne tous les fichiers correspondant à `fin*.jsonl`
2. concatène leur contenu
3. transmet le résultat au loader `loaderAggregate.ini`
4. enregistre le résultat dans `agregation.jsonl`

```text
fin*.jsonl
     │
     ▼
    cat
     │
     ▼
ezs loaderAggregate.ini
     │
     ▼
agregation.jsonl
```

## Cas avancé : agréger les résultats de plusieurs requêtes OpenAlex

Dans cet exemple, on souhaite lancer deux requêtes OpenAlex proches, puis fusionner leurs résultats.

L'objectif est de :

- récupérer les résultats de chaque requête séparément
- concaténer les fichiers obtenus
- dédoublonner les publications à partir de leur `uri`
- conserver une seule ligne par publication
- **garder dans le champ `query` la liste des requêtes ayant retourné cette publication**

### 1. Préparer les fichiers de requêtes

Chaque requête OpenAlex est enregistrée dans un fichier texte séparé.

Par exemple :

#### `oa1.txt`

```text
?filter=publication_year:2026&search.title_and_abstract=open+science
```

#### `oa2.txt`

```text
?filter=publication_year:2026&type=book-chapter&search.title_and_abstract=open+science
```

### 2. Exécuter le loader OpenAlex sur chaque requête

Chaque fichier de requête est ensuite envoyé au loader OpenAlex.

```bash
cat oa1.txt | ezs loaderOpenAlexV2.ini > oa1.jsonl
cat oa2.txt | ezs loaderOpenAlexV2.ini > oa2.jsonl
```

On obtient alors deux fichiers JSONL :

```text
oa1.jsonl
oa2.jsonl
```

Chaque fichier contient les publications retournées par la requête correspondante.

Le loader OpenAlex ajoute notamment :

- une `uri`, construite à partir de l'identifiant OpenAlex
- un champ `query`, qui conserve la requête utilisée

### 3. Pourquoi ne pas utiliser simplement `dedupe` ?

Le loader OpenAlex construit une URI à partir de l'identifiant OpenAlex.

Par exemple :

```json
{
  "id": "https://openalex.org/W7132051972",
  "uri": "uid:/W7132051972"
}
```

Ainsi, si une même publication est retournée par les deux requêtes, elle aura la même `uri` dans `oa1.jsonl` et dans `oa2.jsonl`.

On pourrait donc concaténer les fichiers puis supprimer les doublons avec :

```ini
[dedupe]
ignore = true
```

Cependant, cette méthode conserverait uniquement la dernière notice rencontrée, celle-ci écrasant la précédente.

On ne saurait donc plus si une publication provient uniquement de la première requête, uniquement de la seconde, ou des deux.

L'objectif du loader d'agrégation est donc de dédoublonner les notices tout en conservant cette information de provenance.

### 4. Exemple de notices avant agrégation

#### Dans `oa1.jsonl`

```json
{
  "id":"https://openalex.org/W7132051972",
  "title":"Open science for better research",
  "publication_year":"2026",
  "uri":"uid:/W7132051972",
  "query":"?filter=publication_year:2026&search.title_and_abstract=open+science"
}
```

#### Dans `oa2.jsonl`

```json
{
  "id":"https://openalex.org/W7132051972",
  "title":"Open science for better research",
  "publication_year":"2026",
  "uri":"uid:/W7132051972",
  "query":"?filter=publication_year:2026&type=book-chapter&search.title_and_abstract=open+science"
}
```

### 5. Concaténer les résultats et appliquer le loader d'agrégation

On concatène ensuite les deux fichiers JSONL et on applique le loader d'agrégation.

```bash
cat oa1.jsonl oa2.jsonl | ezs loaderAgregationOpenAlex.ini > oaAgrege.jsonl
```

Le fichier `oaAgrege.jsonl` contiendra une seule ligne par publication.

### 6. Loader `loaderAgregationOpenAlex.ini`

```ini
append = pack
label = json-lines

[use]
plugin = basics
plugin = analytics

[unpack]

[replace]
path = id
value = get("uri")

path = value
value = self()

[aggregate]

[exchange]
value = self().thru(data => \
  _.assign({}, \
    _.mapValues(_.head(data.value), (v, k) => \
      k === "query" \
        ? _.chain(data.value).map(k).uniq().value() \
        : v \
    ) \
  ) \
)
```

### 7. Explication du loader

La première étape prépare l'agrégation :

```ini
[replace]
path = id
value = get("uri")

path = value
value = self()
```

Ici :

- `id` devient la clé de regroupement
- on utilise `uri` car elle identifie de manière unique une publication OpenAlex
- `value` contient la notice complète

L'instruction suivante :

```ini
[aggregate]
```

regroupe toutes les notices possédant la même URI.

Pour notre exemple, les deux notices précédentes sont alors réunies dans un même groupe.

La dernière étape reconstruit une seule notice :

```ini
[exchange]
value = self().thru(data => \
  _.assign({}, \
    _.mapValues(_.head(data.value), (v, k) => \
      k === "query" \
        ? _.chain(data.value).map(k).value() \
        : v \
    ) \
  ) \
)
```

Le traitement fonctionne de la façon suivante :

1. `_.head(data.value)` récupère la première notice du groupe.
2. `_.mapValues(...)` parcourt tous les champs de cette notice.
3. Pour tous les champs sauf `query`, la première valeur rencontrée est conservée.
4. Pour le champ `query`, toutes les valeurs du groupe sont récupérées.
5. `_.assign({}, ...)` reconstruit une nouvelle notice.

### 8. Résultat obtenu

Après agrégation, la publication n'apparaît plus qu'une seule fois :

```json
{
  "id":"https://openalex.org/W7132051972",
  "title":"Open science for better research",
  "publication_year":"2026",
  "uri":"uid:/W7132051972",
  "query":[
    "?filter=publication_year:2026&search.title_and_abstract=open+science",
    "?filter=publication_year:2026&type=book-chapter&search.title_and_abstract=open+science"
  ]
}
```

Le champ `query` permet maintenant de savoir que cette publication a été retrouvée par les deux requêtes.