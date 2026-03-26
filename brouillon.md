
doi,title,authors,publication-year
https://doi.org/10.2307/2005041,Theory of Self-Reproducing Automata,Jacob T. Schwartz|John von Neumann|Arthur W. Burks,1967
https://doi.org/10.2307/1232672,The Theory of Games and Economic Behavior,Robert W. Harrison|John von Neumann|Oskar Morgenstern,1945



```ini
[exchange]
value = self().thru(data => \
  _.assign({}, \
    _.mapValues(_.head(data.value), (v, k) => \
      k === "requete" \
        ? _.chain(data.value).map(k).value() \
        : v \
    ) \
  ) \
)
```