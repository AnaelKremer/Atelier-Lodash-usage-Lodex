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

Cette approche est particulièrement utile pour agréger des snapshots, fusionner des exports successifs ou reconstruire un jeu de données consolidé à partir de plusieurs fichiers JSONL.