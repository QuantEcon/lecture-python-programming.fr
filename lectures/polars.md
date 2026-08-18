---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.7
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
translation:
  title: Polars
  headings:
    Overview: Aperçu général
    Series: Series
    DataFrames: DataFrames
    DataFrames::Selecting data: Sélection des données
    DataFrames::Filtering by conditions: Filtrage par conditions
    DataFrames::Column expressions: Expressions de colonnes
    DataFrames::Missing values: Valeurs manquantes
    DataFrames::Visualization: Visualisation
    Lazy evaluation: Évaluation paresseuse
    Lazy evaluation::Eager vs lazy: Mode immédiat vs mode paresseux
    Lazy evaluation::Query optimization: Optimisation des requêtes
    Lazy evaluation::Performance comparison: Comparaison des performances
    On-line data sources: Sources de données en ligne
    Exercises: Exercices
---

(pl)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Polars

```{index} single: Python; Polars
```

En plus de ce qui est fourni dans Anaconda, ce cours nécessitera les bibliothèques suivantes :

```{code-cell} ipython3
:tags: [hide-output]

!pip install --upgrade polars yfinance
```

## Aperçu général

[Polars](https://pola.rs/) est une bibliothèque de manipulation de données rapide pour Python, écrite en Rust.

Elle a gagné une popularité considérable en tant qu'alternative moderne à {doc}`pandas <pandas>` en raison de ses avantages en matière de performance.

Polars est conçue en tenant compte de la performance et de l'efficacité mémoire, en s'appuyant sur :

* Le [format colonnaire Apache Arrow](https://arrow.apache.org/docs/format/Columnar.html) pour un accès rapide aux données
* L'[évaluation paresseuse](https://en.wikipedia.org/wiki/Lazy_evaluation) pour optimiser l'exécution des requêtes
* Le traitement parallèle pour utiliser tous les cœurs de processeur disponibles
* Une API expressive construite autour d'expressions de colonnes

```{tip}
*Pourquoi envisager Polars plutôt que pandas ?*

* **Mémoire** : pandas nécessite généralement 5 à 10 fois la taille de votre jeu de données en RAM ; Polars n'en nécessite que 2 à 4 fois
* **Vitesse** : Polars est 10 à 100 fois plus rapide pour de nombreuses opérations courantes
* **Voir** : [Les benchmarks TPC-H de Polars](https://pola.rs/benchmarks/) pour des comparaisons de performance à jour
```

Tout au long du cours, nous supposerons que les importations suivantes ont été effectuées

```{code-cell} ipython3
import polars as pl
import numpy as np
import matplotlib.pyplot as plt
```

Comme {doc}`pandas`, Polars définit deux types de données importants : `Series` et `DataFrame`.

Vous pouvez considérer une `Series` comme une colonne de données, telle qu'une collection d'observations sur une seule variable.

Un `DataFrame` est un objet à deux dimensions permettant de stocker des colonnes de données liées entre elles.

## Series

```{index} single: Polars; Series
```

Commençons par les Series.

Nous commençons par créer une série de quatre observations aléatoires

```{code-cell} ipython3
rng = np.random.default_rng()
s = pl.Series(name='daily returns', values=rng.standard_normal(4))
s
```

```{note}
Contrairement aux Series {doc}`pandas <pandas>`, les Series Polars n'ont pas d'indice de ligne.
Polars est centré sur les colonnes --- l'accès aux données est géré par des expressions
de colonnes et des masques booléens plutôt que par des étiquettes de ligne.
Consultez le [guide de migration Polars pour les utilisateurs de pandas](https://docs.pola.rs/user-guide/migration/pandas/) pour plus de détails.
```

Les `Series` Polars sont construites sur les tableaux [Apache Arrow](https://arrow.apache.org/) et prennent en charge de nombreuses opérations familières

```{code-cell} ipython3
s * 100
```

Les valeurs absolues sont disponibles sous forme de méthode

```{code-cell} ipython3
s.abs()
```

Nous pouvons également obtenir rapidement des statistiques récapitulatives

```{code-cell} ipython3
s.describe()
```

Étant donné que Polars n'a pas d'indice de ligne, les données étiquetées nécessitent un `DataFrame`.

Par exemple, pour associer des symboles boursiers à des rendements :

```{code-cell} ipython3
df = pl.DataFrame({
    'company': ['AMZN', 'AAPL', 'MSFT', 'GOOG'],
    'daily returns': rng.standard_normal(4)
})
df
```

Nous accédons à une valeur en filtrant sur une expression de colonne

```{code-cell} ipython3
df.filter(
    pl.col('company') == 'AMZN'
).select('daily returns').item()
```

Les mises à jour utilisent également des expressions plutôt qu'une affectation par indice

```{code-cell} ipython3
df = df.with_columns(
    pl.when(pl.col('company') == 'AMZN')
    .then(0)
    .otherwise(pl.col('daily returns'))
    .alias('daily returns')
)
df
```

Nous pouvons également vérifier l'appartenance

```{code-cell} ipython3
'AAPL' in df['company']
```

## DataFrames

```{index} single: Polars; DataFrames
```

Alors qu'une `Series` est une seule colonne de données, un `DataFrame` comporte plusieurs colonnes, une pour chaque variable.

Comme dans {doc}`pandas`, travaillons avec les données des [Penn World Tables](https://www.rug.nl/ggdc/productivity/pwt/pwt-releases/pwt-7.0).

Nous les lisons à l'aide de `pl.read_csv`

```{code-cell} ipython3
url = ('https://raw.githubusercontent.com/QuantEcon/'
       'lecture-python-programming/main/lectures/_static/'
       'lecture_specific/pandas/data/test_pwt.csv')
df = pl.read_csv(url)
df
```

### Sélection des données

Nous pouvons sélectionner des lignes par découpage (slicing) et des colonnes par nom

```{code-cell} ipython3
df[2:5]
```

Pour sélectionner des colonnes spécifiques, passez une liste de noms à `select`

```{code-cell} ipython3
df.select(['country', 'tcgdp'])
```

Ces opérations peuvent être combinées

```{code-cell} ipython3
df[2:5].select(['country', 'tcgdp'])
```

### Filtrage par conditions

La méthode `filter` accepte des expressions booléennes construites à partir de `pl.col`

```{code-cell} ipython3
df.filter(pl.col('POP') >= 20000)
```

Plusieurs conditions peuvent être combinées avec `&` (et) et `|` (ou)

```{code-cell} ipython3
df.filter(
    (pl.col('country').is_in(['Argentina', 'India', 'South Africa'])) &
    (pl.col('POP') > 40000)
)
```

Les expressions peuvent impliquer des opérations arithmétiques entre colonnes

```{code-cell} ipython3
df.filter(
    (pl.col('cc') + pl.col('cg') >= 80) & (pl.col('POP') <= 20000)
)
```

Sélectionnons le pays ayant la part de consommation des ménages la plus élevée

```{code-cell} ipython3
df.filter(pl.col('cc') == pl.col('cc').max())
```

### Expressions de colonnes

Une différence essentielle avec pandas est que Polars utilise des **expressions de colonnes** pour les transformations plutôt que des appels `apply` élément par élément.

Voici un exemple calculant le maximum de chaque colonne numérique

```{code-cell} ipython3
df.select(
    pl.col(['year', 'POP', 'XRAT', 'tcgdp', 'cc', 'cg'])
    .max()
    .name.suffix('_max')
)
```

Les expressions peuvent être utilisées dans `with_columns` pour ajouter ou modifier des colonnes

```{code-cell} ipython3
df.with_columns(
    (pl.col('XRAT') / 10).alias('XRAT_scaled'),
    pl.col(pl.Float64).round(2)
)
```

La logique conditionnelle utilise `pl.when(...).then(...).otherwise(...)`

```{code-cell} ipython3
df.with_columns(
    pl.when(pl.col('POP') >= 20000)
    .then(pl.col('POP'))
    .otherwise(None)
    .alias('POP_filtered')
).select(['country', 'POP', 'POP_filtered'])
```

```{note}
Polars fournit `map_elements` comme solution de secours pour appliquer des fonctions
Python arbitraires ligne par ligne, mais cela contourne le moteur d'expression
optimisé et devrait être évité lorsqu'une expression native existe.
```

### Valeurs manquantes

Insérons quelques valeurs nulles pour démontrer les techniques d'imputation

```{code-cell} ipython3
df_nulls = df.with_row_index().with_columns(
    pl.when(pl.col('index') == 0)
    .then(None).otherwise(pl.col('XRAT')).alias('XRAT'),
    pl.when(pl.col('index') == 3)
    .then(None).otherwise(pl.col('cc')).alias('cc'),
    pl.when(pl.col('index') == 5)
    .then(None).otherwise(pl.col('tcgdp')).alias('tcgdp'),
    pl.when(pl.col('index') == 6)
    .then(None).otherwise(pl.col('POP')).alias('POP'),
).drop('index')
df_nulls
```

Remplissons toutes les valeurs nulles par zéro

```{code-cell} ipython3
df_nulls.fill_null(0)
```

Ou remplissons avec les moyennes des colonnes

```{code-cell} ipython3
cols = ['cc', 'tcgdp', 'POP', 'XRAT']
df_nulls.with_columns(
    pl.col(cols).fill_null(pl.col(cols).mean())
)
```

Polars prend également en charge le remplissage en avant (`fill_null(strategy='forward')`) et l'interpolation.

Il existe des [outils d'imputation plus avancés](https://scikit-learn.org/stable/modules/impute.html) disponibles dans scikit-learn.

### Visualisation

Construisons une colonne de PIB par habitant et traçons-la

```{code-cell} ipython3
df = (df
    .select(['country', 'POP', 'tcgdp'])
    .rename({'POP': 'population', 'tcgdp': 'total GDP'})
    .with_columns(
        (pl.col('population') * 1e3).alias('population')
    )
    .with_columns(
        (pl.col('total GDP') * 1e6 / pl.col('population'))
        .alias('GDP percap')
    )
    .sort('GDP percap', descending=True)
)
df
```

Nous pouvons extraire directement les colonnes pour matplotlib

```{note}
Polars fournit également une [API de tracé intégrée](https://docs.pola.rs/user-guide/misc/visualization/)
basée sur Altair (par exemple, `df.plot.bar(x=..., y=...)`).
Nous utilisons matplotlib ici pour rester cohérents avec le reste de la série de cours.
```

```{code-cell} ipython3
fig, ax = plt.subplots()
ax.bar(df['country'].to_list(), df['GDP percap'].to_list())
ax.set_xlabel('country', fontsize=12)
ax.set_ylabel('GDP per capita', fontsize=12)
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()
```

## Évaluation paresseuse

```{index} single: Polars; Lazy Evaluation
```

L'une des fonctionnalités les plus puissantes de Polars est l'**évaluation paresseuse** (lazy evaluation).

Au lieu d'exécuter chaque opération immédiatement, le mode paresseux collecte le plan de requête complet et l'optimise avant de l'exécuter.

### Mode immédiat vs mode paresseux

```{code-cell} ipython3
# Rechargement du jeu de données
url = ('https://raw.githubusercontent.com/QuantEcon/'
       'lecture-python-programming/main/lectures/_static/'
       'lecture_specific/pandas/data/test_pwt.csv')
df_full = pl.read_csv(url)
```

L'API **immédiate** (eager) s'exécute directement (comme pandas)

```{code-cell} ipython3
result_eager = (df_full
    .filter(pl.col('tcgdp') > 1000)
    .select(['country', 'year', 'tcgdp'])
    .sort('tcgdp', descending=True)
)
result_eager.head()
```

L'API **paresseuse** (lazy) construit plutôt un plan de requête

```{code-cell} ipython3
lazy_query = (df_full.lazy()
    .filter(pl.col('tcgdp') > 1000)
    .select(['country', 'year', 'tcgdp'])
    .sort('tcgdp', descending=True)
)
print(lazy_query.explain())
```

Appelez `collect` pour exécuter le plan

```{code-cell} ipython3
result_lazy = lazy_query.collect()
result_lazy.head()
```

### Optimisation des requêtes

Le moteur paresseux applique automatiquement plusieurs optimisations :

* **Descente des prédicats** (predicate pushdown) --- les filtres sont appliqués le plus tôt possible
* **Descente des projections** (projection pushdown) --- seules les colonnes nécessaires sont lues depuis la source
* **Élimination des sous-expressions communes** --- les calculs en double sont fusionnés

Voyons comment Polars réécrit une requête à plusieurs étapes

```{code-cell} ipython3
optimized = (df_full.lazy()
    .select(['country', 'year', 'tcgdp', 'POP'])
    .filter(pl.col('tcgdp') > 500)
    .with_columns(
        (pl.col('tcgdp') / pl.col('POP')).alias('gdp_per_capita')
    )
    .filter(pl.col('gdp_per_capita') > 10)
    .select(['country', 'year', 'gdp_per_capita'])
)

print("Optimized plan:")
print(optimized.explain())
```

L'exécution du plan nous donne le résultat final

```{code-cell} ipython3
optimized.collect()
```

### Comparaison des performances

Comparons pandas, Polars en mode immédiat, et Polars en mode paresseux sur la même tâche.

Nous commençons avec un petit jeu de données (les Penn World Tables utilisées ci-dessus) pour montrer
que pour de petites données, les différences sont négligeables

```{code-cell} ipython3
import pandas as pd
import time

# Petit jeu de données -- Penn World Tables (~8 lignes)
url = ('https://raw.githubusercontent.com/QuantEcon/'
       'lecture-python-programming/main/lectures/_static/'
       'lecture_specific/pandas/data/test_pwt.csv')
small_pd = pd.read_csv(url)
small_pl = pl.read_csv(url)
```

Maintenant, chronométrons la même opération de filtrage-sélection-tri dans chaque bibliothèque

```{code-cell} ipython3
# pandas
start = time.perf_counter()
_ = (small_pd
     .query('tcgdp > 500')
     [['country', 'year', 'tcgdp', 'POP']]
     .assign(gdp_pc=lambda d: d['tcgdp'] / d['POP'])
     .sort_values('gdp_pc', ascending=False))
pd_small = time.perf_counter() - start

# Polars immédiat
start = time.perf_counter()
_ = (small_pl
     .filter(pl.col('tcgdp') > 500)
     .select(['country', 'year', 'tcgdp', 'POP'])
     .with_columns((pl.col('tcgdp') / pl.col('POP')).alias('gdp_pc'))
     .sort('gdp_pc', descending=True))
pl_small = time.perf_counter() - start

print(f"Small data  --  pandas: {pd_small:.4f}s | Polars eager: {pl_small:.4f}s")
```

Sur quelques lignes, la différence de vitesse est négligeable --- utilisez celle
des API que vous trouvez la plus pratique.

Passons maintenant à 5 millions de lignes, où la différence devient évidente.

La tâche consiste à : filtrer les lignes où `value > 0`, calculer un produit
pondéré `value * weight`, puis calculer la moyenne de ce produit au sein de chaque groupe ---
une moyenne pondérée groupée.

```{code-cell} ipython3
n = 5_000_000
rng = np.random.default_rng(42)

groups = rng.choice(['A', 'B', 'C', 'D'], n)
values = rng.standard_normal(n)
weights = rng.random(n)
extra1 = rng.standard_normal(n)
extra2 = rng.standard_normal(n)

big_pd = pd.DataFrame({
    'group': groups, 'value': values,
    'weight': weights, 'extra1': extra1, 'extra2': extra2
})
big_pl = pl.DataFrame({
    'group': groups, 'value': values,
    'weight': weights, 'extra1': extra1, 'extra2': extra2
})
```

D'abord, la référence pandas

```{code-cell} ipython3
start = time.perf_counter()
tmp = big_pd[big_pd['value'] > 0][['group', 'value', 'weight']].copy()
tmp['weighted'] = tmp['value'] * tmp['weight']
_ = tmp.groupby('group')['weighted'].mean()
pd_time = time.perf_counter() - start
print(f"pandas:       {pd_time:.4f}s")
```

Ensuite, Polars en mode immédiat

```{code-cell} ipython3
start = time.perf_counter()
_ = (big_pl
    .filter(pl.col('value') > 0)
    .select(['group', 'value', 'weight'])
    .with_columns(
        (pl.col('value') * pl.col('weight')).alias('weighted'))
    .group_by('group')
    .agg(pl.col('weighted').mean()))
eager_time = time.perf_counter() - start
print(f"Polars eager: {eager_time:.4f}s")
```

Et enfin, Polars en mode paresseux

```{code-cell} ipython3
start = time.perf_counter()
_ = (big_pl.lazy()
    .filter(pl.col('value') > 0)
    .select(['group', 'value', 'weight'])
    .with_columns(
        (pl.col('value') * pl.col('weight')).alias('weighted'))
    .group_by('group')
    .agg(pl.col('weighted').mean())
    .collect())
lazy_time = time.perf_counter() - start
print(f"Polars lazy:  {lazy_time:.4f}s")
```

Ce qu'il faut retenir :

* Pour de **petits volumes de données** (milliers de lignes), pandas et Polars
  se comportent de manière similaire --- choisissez en fonction de vos préférences d'API et de l'écosystème.
* Pour des **volumes de données moyens à grands** (centaines de milliers de lignes et plus),
  Polars peut être significativement plus rapide grâce à son moteur Rust, à l'exécution
  parallèle et (en mode paresseux) à l'optimisation des requêtes.

L'API paresseuse est particulièrement puissante lors de la lecture depuis le disque --- `scan_csv` retourne directement un `LazyFrame`, de sorte que les filtres et projections sont poussés jusqu'au lecteur de fichier.

```{tip}
Utilisez `pl.scan_csv(path)` plutôt que `pl.read_csv(path)` lorsque vous travaillez avec
de gros fichiers CSV.
Seules les colonnes et lignes réellement nécessaires seront lues depuis le disque.
Voir [la documentation E/S de Polars](https://docs.pola.rs/user-guide/io/csv/).
```

## Sources de données en ligne

```{index} single: Data Sources
```

Comme dans {doc}`pandas`, Python permet d'interroger facilement des bases de données en ligne.

Une base de données importante pour les économistes est [FRED](https://fred.stlouisfed.org/) --- une vaste collection de séries temporelles maintenue par la Fed de St. Louis.

La méthode `read_csv` de Polars peut récupérer des données directement depuis une URL.

Nous utilisons `try_parse_dates=True` pour analyser automatiquement la colonne de date

```{code-cell} ipython3
fred_url = ('https://fred.stlouisfed.org/graph/fredgraph.csv?'
            'bgcolor=%23e1e9f0&chart_type=line&drp=0&'
            'fo=open%20sans&graph_bgcolor=%23ffffff&'
            'height=450&mode=fred&recession_bars=on&'
            'txtcolor=%23444444&ts=12&tts=12&width=1318&'
            'nt=0&thu=0&trc=0&show_legend=yes&'
            'show_axis_titles=yes&show_tooltip=yes&'
            'id=UNRATE&scale=left&cosd=1948-01-01&'
            'coed=2024-06-01&line_color=%234572a7&'
            'link_values=false&line_style=solid&'
            'mark_type=none&mw=3&lw=2&ost=-99999&'
            'oet=99999&mma=0&fml=a&fq=Monthly&fam=avg&'
            'fgst=lin&fgsnd=2020-02-01&line_index=1&'
            'transformation=lin&vintage_date=2024-07-29&'
            'revision_date=2024-07-29&nd=1948-01-01')
data = pl.read_csv(fred_url, try_parse_dates=True)
```

Examinons les premières lignes

```{code-cell} ipython3
data.head()
```

Et obtenons des statistiques récapitulatives

```{code-cell} ipython3
data.describe()
```

Traçons le taux de chômage de 2006 à 2012

```{code-cell} ipython3
filtered = data.filter(
    (pl.col('observation_date') >= pl.date(2006, 1, 1)) &
    (pl.col('observation_date') <= pl.date(2012, 12, 31))
)

fig, ax = plt.subplots()
ax.plot(filtered['observation_date'].to_list(),
        filtered['UNRATE'].to_list())
ax.set_title('US Unemployment Rate')
ax.set_xlabel('year', fontsize=12)
ax.set_ylabel('%', fontsize=12)
plt.show()
```

Polars prend en charge [de nombreux formats de fichiers](https://docs.pola.rs/user-guide/io/) tels que Excel, JSON, Parquet, ainsi que des connexions directes à des bases de données.

## Exercices

```{exercise-start}
:label: pl_ex1
```

Avec ces importations :

```{code-cell} ipython3
import datetime as dt
import yfinance as yf
```

Écrivez un programme pour calculer la variation en pourcentage du prix sur l'année 2021 pour les actions suivantes :

```{code-cell} ipython3
ticker_list = {'INTC': 'Intel',
               'MSFT': 'Microsoft',
               'IBM': 'IBM',
               'BHP': 'BHP',
               'TM': 'Toyota',
               'AAPL': 'Apple',
               'AMZN': 'Amazon',
               'C': 'Citigroup',
               'QCOM': 'Qualcomm',
               'KO': 'Coca-Cola',
               'GOOG': 'Google'}
```

Voici une fonction qui lit les cours de clôture dans un DataFrame Polars :

```{code-cell} ipython3
def read_data_polars(ticker_list,
                     start=dt.datetime(2021, 1, 1),
                     end=dt.datetime(2021, 12, 31)):
    """
    Read closing price data from Yahoo Finance
    and return a Polars DataFrame.
    """
    dataframes = []

    for tick in ticker_list:
        stock = yf.Ticker(tick)
        prices = stock.history(start=start, end=end)
        df = pl.DataFrame({
            'Date': list(prices.index.date),
            tick: prices['Close'].values
        }).with_columns(pl.col('Date').cast(pl.Date))
        dataframes.append(df)

    result = dataframes[0]
    for df in dataframes[1:]:
        result = result.join(
            df, on='Date', how='full', coalesce=True
        )
    return result.sort('Date')

ticker = read_data_polars(ticker_list)
```

```{note}
Les jointures Polars ne garantissent pas l'ordre des lignes en sortie --- les clés
présentes uniquement d'un côté sont ajoutées plutôt qu'insérées à leur place.
Il s'agit du même thème « pas d'indice, pas d'alignement automatique » évoqué plus haut : en l'absence
d'étiquettes de ligne sur lesquelles s'aligner, l'ordre est quelque chose que l'on demande explicitement.
D'où le `sort('Date')` avant le retour, dont dépend tout calcul ultérieur
de type `first()`/`last()`.
```

Complétez le programme pour tracer le résultat sous forme de graphique à barres.

```{exercise-end}
```

```{solution-start} pl_ex1
:class: dropdown
```

Calculons les variations en pourcentage à l'aide d'expressions Polars :

```{code-cell} ipython3
price_change = ticker.select([
    ((pl.col(tick).drop_nulls().last() / pl.col(tick).drop_nulls().first() - 1) * 100)
    .alias(tick)
    for tick in ticker_list.keys()
]).transpose(
    include_header=True,
    header_name='ticker',
    column_names=['pct_change']
).with_columns(
    pl.col('ticker')
    .replace_strict(ticker_list, default=pl.col('ticker'))
    .alias('company')
).sort('pct_change')

print(price_change)
```

Traçons les résultats directement avec matplotlib :

```{code-cell} ipython3
companies = price_change['company'].to_list()
changes = price_change['pct_change'].to_list()
colors = ['red' if x < 0 else 'blue' for x in changes]

fig, ax = plt.subplots(figsize=(10, 8))
ax.bar(companies, changes, color=colors)
ax.set_xlabel('stock', fontsize=12)
ax.set_ylabel('percentage change in price', fontsize=12)
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()
```

```{solution-end}
```


```{exercise-start}
:label: pl_ex2
```

En utilisant `read_data_polars` de {ref}`pl_ex1`, obtenez la variation en pourcentage d'une année sur l'autre pour les indices suivants :

```{code-cell} ipython3
indices_list = {'^GSPC': 'S&P 500',
               '^IXIC': 'NASDAQ',
               '^DJI': 'Dow Jones',
               '^N225': 'Nikkei'}
```

Tracez le résultat sous forme de graphique en série temporelle.

```{exercise-end}
```

```{solution-start} pl_ex2
:class: dropdown
```

```{code-cell} ipython3
indices_data = read_data_polars(
    indices_list,
    start=dt.datetime(1971, 1, 1),
    end=dt.datetime(2021, 12, 31)
)

indices_data = indices_data.with_columns(
    pl.col('Date').dt.year().alias('year')
)
```

Calculons les rendements annuels à l'aide d'opérations de regroupement :

```{code-cell} ipython3
yearly_returns = indices_data.group_by('year').agg([
    *[pl.col(idx).drop_nulls().first().alias(f'{idx}_first')
      for idx in indices_list],
    *[pl.col(idx).drop_nulls().last().alias(f'{idx}_last')
      for idx in indices_list]
])

for idx, name in indices_list.items():
    yearly_returns = yearly_returns.with_columns(
        ((pl.col(f'{idx}_last') - pl.col(f'{idx}_first'))
         / pl.col(f'{idx}_first') * 100).alias(name)
    )

yearly_returns = (yearly_returns
    .select(['year', *indices_list.values()])
    .sort('year')
)
print(yearly_returns)
```

Statistiques récapitulatives :

```{code-cell} ipython3
yearly_returns.select(list(indices_list.values())).describe()
```

Traçons chaque indice dans un sous-graphique :

```{code-cell} ipython3
fig, axes = plt.subplots(2, 2, figsize=(12, 10))
years = yearly_returns['year'].to_list()

for iter_, ax in enumerate(axes.flatten()):
    name = list(indices_list.values())[iter_]
    values = yearly_returns[name].to_list()
    ax.plot(years, values, 'o-', linewidth=2, markersize=4)
    ax.axhline(y=0, color='k', linestyle='--', alpha=0.3)
    ax.set_ylabel('yearly return (%)', fontsize=12)
    ax.set_xlabel('year', fontsize=12)
    ax.set_title(name, fontsize=12)

plt.tight_layout()
plt.show()
```

```{solution-end}
```
