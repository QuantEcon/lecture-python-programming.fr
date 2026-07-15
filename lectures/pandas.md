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
  title: Pandas
  headings:
    Overview: Vue d'ensemble
    Series: Series
    DataFrames: DataFrames
    DataFrames::Select Data by Position: Sélectionner les données par position
    DataFrames::Select Data by Conditions: Sélectionner les données par conditions
    DataFrames::Apply Method: Méthode Apply
    DataFrames::Make Changes in DataFrames: Effectuer des modifications dans les DataFrames
    DataFrames::Standardization and Visualization: Standardisation et visualisation
    On-Line Data Sources: Sources de données en ligne
    On-Line Data Sources::Accessing Data with requests: Accéder aux données avec requests
    On-Line Data Sources::Using wbgapi and yfinance to Access Data: Utiliser wbgapi et yfinance pour accéder aux données
    Exercises: Exercices
---

(pd)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# {index}`Pandas <single: Pandas>`

```{index} single: Python; Pandas
```

En plus de ce qui est inclus dans Anaconda, ce cours nécessitera les bibliothèques suivantes :

```{code-cell} ipython3
:tags: [hide-output]

!pip install --upgrade wbgapi
!pip install --upgrade yfinance
```

## Vue d'ensemble

[Pandas](https://pandas.pydata.org/) est un ensemble d'outils d'analyse de données rapides et efficaces pour Python.

Sa popularité a fortement augmenté ces dernières années, en parallèle de l'essor de domaines tels que la science des données et l'apprentissage automatique.

Voici une comparaison de popularité dans le temps face à Matlab et STATA, offerte par Stack Overflow Trends

```{figure} /_static/lecture_specific/pandas/pandas_vs_rest.png
:scale: 100
```

Tout comme [NumPy](https://numpy.org/) fournit le type de données tableau de base ainsi que les opérations fondamentales sur les tableaux, pandas

1. définit des structures fondamentales pour travailler avec les données et
1. les dote de méthodes qui facilitent des opérations telles que
    * la lecture de données
    * l'ajustement des indices
    * le travail avec les dates et les séries temporelles
    * le tri, le regroupement, la réorganisation et le traitement général des données [^mung]
    * la gestion des valeurs manquantes, etc., etc.

Des fonctionnalités statistiques plus sophistiquées sont laissées à d'autres paquets, tels que [statsmodels](https://www.statsmodels.org/) et [scikit-learn](https://scikit-learn.org/), qui sont construits par-dessus pandas.

Ce cours fournira une introduction de base à pandas.

Tout au long du cours, nous supposerons que les importations suivantes ont été effectuées

```{code-cell} ipython3
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import requests
```

Deux types de données importants définis par pandas sont `Series` et `DataFrame`.

Vous pouvez considérer une `Series` comme une « colonne » de données, telle qu'une collection d'observations sur une seule variable.

Un `DataFrame` est un objet à deux dimensions permettant de stocker des colonnes de données liées entre elles.

## Series

```{index} single: Pandas; Series
```

Commençons par les Series.


Nous commençons par créer une série de quatre observations aléatoires

```{code-cell} ipython3
s = pd.Series(np.random.randn(4), name='daily returns')
s
```

Ici, vous pouvez imaginer les indices `0, 1, 2, 3` comme indexant quatre sociétés cotées, et les valeurs étant les rendements quotidiens de leurs actions.

Les `Series` de pandas sont construites par-dessus les tableaux NumPy et prennent en charge de nombreuses opérations similaires

```{code-cell} ipython3
s * 100
```

```{code-cell} ipython3
np.abs(s)
```

Mais les `Series` offrent plus que les tableaux NumPy.

Non seulement elles disposent de méthodes supplémentaires (à orientation statistique)

```{code-cell} ipython3
s.describe()
```

mais leurs indices sont plus flexibles

```{code-cell} ipython3
s.index = ['AMZN', 'AAPL', 'MSFT', 'GOOG']
s
```

Vues de cette manière, les `Series` sont comme des dictionnaires Python rapides et efficaces (avec la restriction que tous les éléments du dictionnaire ont le même type — dans ce cas, des flottants).

En fait, vous pouvez utiliser une grande partie de la même syntaxe que les dictionnaires Python

```{code-cell} ipython3
s['AMZN']
```

```{code-cell} ipython3
s['AMZN'] = 0
s
```

```{code-cell} ipython3
'AAPL' in s
```

## DataFrames

```{index} single: Pandas; DataFrames
```

Alors qu'une `Series` est une seule colonne de données, un `DataFrame` comporte plusieurs colonnes, une pour chaque variable.

En substance, un `DataFrame` dans pandas est analogue à une feuille de calcul Excel (hautement optimisée).

Ainsi, c'est un outil puissant pour représenter et analyser des données naturellement organisées en lignes et en colonnes, souvent avec des indices descriptifs pour les lignes et les colonnes individuelles.

Regardons un exemple qui lit des données à partir du fichier CSV `pandas/data/test_pwt.csv`, tiré des [Penn World Tables](https://www.rug.nl/ggdc/productivity/pwt/pwt-releases/pwt-7.0).

Le jeu de données contient les indicateurs suivants

| Nom de la variable | Description |
| :- : | :- : |
| POP | Population (en milliers) |
| XRAT | Taux de change par rapport au dollar américain |                     
| tcgdp | PIB total converti en PPA (en millions de dollars internationaux) |
| cc | Part de la consommation dans le PIB par habitant converti en PPA (%) |
| cg | Part de la consommation publique dans le PIB par habitant converti en PPA (%) |


Nous allons le lire depuis une URL en utilisant la fonction `read_csv` de `pandas`.

```{code-cell} ipython3
df = pd.read_csv('https://raw.githubusercontent.com/QuantEcon/lecture-python-programming/main/lectures/_static/lecture_specific/pandas/data/test_pwt.csv')
type(df)
```

Voici le contenu de `test_pwt.csv`

```{code-cell} ipython3
df
```

### Sélectionner les données par position

En pratique, une chose que nous faisons tout le temps est de rechercher, sélectionner et travailler avec un sous-ensemble des données qui nous intéressent.

Nous pouvons sélectionner des lignes particulières en utilisant la notation standard de découpage de tableau Python

```{code-cell} ipython3
df[2:5]
```

Pour sélectionner des colonnes, nous pouvons passer une liste contenant les noms des colonnes désirées représentés sous forme de chaînes de caractères

```{code-cell} ipython3
df[['country', 'tcgdp']]
```

Pour sélectionner à la fois des lignes et des colonnes en utilisant des entiers, l'attribut `iloc` doit être utilisé avec le format `.iloc[lignes, colonnes]`.

```{code-cell} ipython3
df.iloc[2:5, 0:4]
```

Pour sélectionner des lignes et des colonnes en utilisant un mélange d'entiers et d'étiquettes, l'attribut `loc` peut être utilisé de manière similaire

```{code-cell} ipython3
df.loc[df.index[2:5], ['country', 'tcgdp']]
```

### Sélectionner les données par conditions

Au lieu d'indexer les lignes et les colonnes en utilisant des entiers et des noms, nous pouvons également obtenir un sous-dataframe qui nous intéresse et qui satisfait certaines conditions (potentiellement complexes).

Cette section illustre différentes manières de procéder.

La manière la plus directe est d'utiliser l'opérateur `[]`.

```{code-cell} ipython3
df[df.POP >= 20000]
```

Pour comprendre ce qui se passe ici, remarquez que `df.POP >= 20000` renvoie une série de valeurs booléennes.

```{code-cell} ipython3
df.POP >= 20000
```

Dans ce cas, `df[___]` prend une série de valeurs booléennes et ne renvoie que les lignes ayant les valeurs `True`.

Prenons un autre exemple,

```{code-cell} ipython3
df[(df.country.isin(['Argentina', 'India', 'South Africa'])) & (df.POP > 40000)]
```

Cependant, il existe une autre manière de faire la même chose, qui peut être légèrement plus rapide pour les grands dataframes, avec une syntaxe plus naturelle.

```{code-cell} ipython3
# ce qui précède est équivalent à
df.query("POP >= 20000")
```

```{code-cell} ipython3
df.query("country in ['Argentina', 'India', 'South Africa'] and POP > 40000")
```

Nous pouvons également autoriser des opérations arithmétiques entre différentes colonnes.

```{code-cell} ipython3
df[(df.cc + df.cg >= 80) & (df.POP <= 20000)]
```

```{code-cell} ipython3
# ce qui précède est équivalent à
df.query("cc + cg >= 80 & POP <= 20000")
```

Par exemple, nous pouvons utiliser le conditionnement pour sélectionner le pays ayant la plus grande part de consommation des ménages dans le PIB `cc`.

```{code-cell} ipython3
df.loc[df.cc == max(df.cc)]
```

Lorsque nous ne voulons regarder que certaines colonnes d'un sous-dataframe sélectionné, nous pouvons utiliser les conditions ci-dessus avec la commande `.loc[__ , __]`.

Le premier argument prend la condition, tandis que le second argument prend une liste de colonnes que nous voulons renvoyer.

```{code-cell} ipython3
df.loc[(df.cc + df.cg >= 80) & (df.POP <= 20000), ['country', 'year', 'POP']]
```

**Application : Sous-ensemble d'un Dataframe**

Les jeux de données du monde réel peuvent être [énormes](https://developers.google.com/machine-learning/crash-course/overfitting).

Il est parfois souhaitable de travailler avec un sous-ensemble de données pour améliorer l'efficacité computationnelle et réduire la redondance.

Imaginons que nous ne nous intéressions qu'à la population (`POP`) et au PIB total (`tcgdp`).

Une manière de réduire le dataframe `df` à ces seules variables est d'écraser le dataframe en utilisant la méthode de sélection décrite ci-dessus

```{code-cell} ipython3
df_subset = df[['country', 'POP', 'tcgdp']]
df_subset
```

Nous pouvons ensuite enregistrer le jeu de données plus petit pour une analyse ultérieure.

```{code-block} python3
:class: no-execute

df_subset.to_csv('pwt_subset.csv', index=False)
```

### Méthode Apply

Une autre méthode Pandas largement utilisée est `df.apply()`.

Elle applique une fonction à chaque ligne/colonne et renvoie une série.

Cette fonction peut être une fonction intégrée comme la fonction `max`, une fonction `lambda`, ou une fonction définie par l'utilisateur.

Voici un exemple utilisant la fonction `max`

```{code-cell} ipython3
df[['year', 'POP', 'XRAT', 'tcgdp', 'cc', 'cg']].apply(max)
```

Cette ligne de code applique la fonction `max` à toutes les colonnes sélectionnées.

La fonction `lambda` est souvent utilisée avec la méthode `df.apply()`

Un exemple trivial consiste à renvoyer elle-même chaque ligne du dataframe

```{code-cell} ipython3
df.apply(lambda row: row, axis=1)
```

```{note}
Pour la méthode `.apply()`
- axis = 0 -- applique la fonction à chaque colonne (variables)
- axis = 1 -- applique la fonction à chaque ligne (observations)
- axis = 0 est le paramètre par défaut
```

Nous pouvons l'utiliser conjointement avec `.loc[]` pour effectuer des sélections plus avancées.

```{code-cell} ipython3
complexCondition = df.apply(
    lambda row: row.POP > 40000 if row.country in ['Argentina', 'India', 'South Africa'] else row.POP < 20000, 
    axis=1), ['country', 'year', 'POP', 'XRAT', 'tcgdp']
```

`df.apply()` renvoie ici une série de valeurs booléennes pour les lignes qui satisfont la condition spécifiée dans l'instruction if-else.

De plus, elle définit également un sous-ensemble de variables d'intérêt.

```{code-cell} ipython3
complexCondition
```

Lorsque nous appliquons cette condition au dataframe, le résultat sera

```{code-cell} ipython3
df.loc[complexCondition]
```

### Effectuer des modifications dans les DataFrames

La capacité d'effectuer des modifications dans les dataframes est importante pour générer un jeu de données propre en vue d'analyses futures.


**1.** Nous pouvons utiliser `df.where()` de manière pratique pour « conserver » les lignes que nous avons sélectionnées et remplacer les autres lignes par n'importe quelles autres valeurs

```{code-cell} ipython3
df.where(df.POP >= 20000, False)
```

**2.** Nous pouvons simplement utiliser `.loc[]` pour spécifier la colonne que nous voulons modifier, et attribuer des valeurs

```{code-cell} ipython3
df.loc[df.cg == max(df.cg), 'cg'] = np.nan
df
```

**3.** Nous pouvons utiliser la méthode `.apply()` pour modifier *des lignes/colonnes dans leur ensemble*

```{code-cell} ipython3
def update_row(row):
    # modifie POP
    row.POP = np.nan if row.POP<= 10000 else row.POP

    # modifie XRAT
    row.XRAT = row.XRAT / 10
    return row

df.apply(update_row, axis=1)
```

**4.** Nous pouvons utiliser la méthode `.map()` pour modifier toutes les *entrées individuelles* du dataframe d'un seul coup.

```{code-cell} ipython3
# Arrondit tous les nombres décimaux à 2 décimales
df.map(lambda x : round(x,2) if type(x)!=str else x)
```

**Application : Imputation des valeurs manquantes**

Le remplacement des valeurs manquantes est une étape importante du traitement des données.

Insérons aléatoirement quelques valeurs NaN

```{code-cell} ipython3
for idx in list(zip([0, 3, 5, 6], [3, 4, 6, 2])):
    df.iloc[idx] = np.nan

df
```

La fonction `zip()` crée ici des paires de valeurs à partir des deux listes (c'est-à-dire [0,3], [3,4] ...)

Nous pouvons utiliser à nouveau la méthode `.map()` pour remplacer toutes les valeurs manquantes par 0

```{code-cell} ipython3
# remplace toutes les valeurs NaN par 0
def replace_nan(x):
    if type(x)!=str:
        return  0 if np.isnan(x) else x
    else:
        return x

df.map(replace_nan)
```

Pandas nous fournit également des méthodes pratiques pour remplacer les valeurs manquantes.

Par exemple, l'imputation simple utilisant les moyennes des variables peut être facilement réalisée dans pandas

```{code-cell} ipython3
df = df.fillna(df.iloc[:,2:8].mean())
df
```

L'imputation des valeurs manquantes est un vaste domaine de la science des données impliquant diverses techniques d'apprentissage automatique.

Il existe également des [outils plus avancés](https://scikit-learn.org/stable/modules/impute.html) en Python pour imputer les valeurs manquantes.

### Standardisation et visualisation

Imaginons que nous ne nous intéressions qu'à la population (`POP`) et au PIB total (`tcgdp`).

Une manière de réduire le dataframe `df` à ces seules variables est d'écraser le dataframe en utilisant la méthode de sélection décrite ci-dessus

```{code-cell} ipython3
df = df[['country', 'POP', 'tcgdp']]
df
```

Ici, l'indice `0, 1,..., 7` est redondant car nous pouvons utiliser les noms des pays comme indice.

Pour ce faire, nous définissons l'indice comme étant la variable `country` du dataframe

```{code-cell} ipython3
df = df.set_index('country')
df
```

Donnons aux colonnes des noms légèrement meilleurs

```{code-cell} ipython3
df.columns = 'population', 'total GDP'
df
```

La variable `population` est en milliers, revenons aux unités simples

```{code-cell} ipython3
df['population'] = df['population'] * 1e3
df
```

Ensuite, nous allons ajouter une colonne montrant le PIB réel par habitant, en multipliant par 1 000 000 au passage car le PIB total est en millions

```{code-cell} ipython3
df['GDP percap'] = df['total GDP'] * 1e6 / df['population']
df
```

L'une des choses agréables avec les objets `DataFrame` et `Series` de pandas est qu'ils disposent de méthodes de tracé et de visualisation qui fonctionnent via Matplotlib.

Par exemple, nous pouvons facilement générer un diagramme en barres du PIB par habitant

```{code-cell} ipython3
ax = df['GDP percap'].plot(kind='bar')
ax.set_xlabel('country', fontsize=12)
ax.set_ylabel('GDP per capita', fontsize=12)
plt.show()
```

Pour le moment, le dataframe est ordonné par ordre alphabétique des pays — modifions-le pour l'ordonner selon le PIB par habitant

```{code-cell} ipython3
df = df.sort_values(by='GDP percap', ascending=False)
df
```

Un tracé comme précédemment donne maintenant

```{code-cell} ipython3
ax = df['GDP percap'].plot(kind='bar')
ax.set_xlabel('country', fontsize=12)
ax.set_ylabel('GDP per capita', fontsize=12)
plt.show()
```

## Sources de données en ligne

```{index} single: Data Sources
```

Python facilite l'interrogation de bases de données en ligne de manière programmatique.

Une base de données importante pour les économistes est [FRED](https://fred.stlouisfed.org/) — une vaste collection de données de séries temporelles maintenue par la Fed de St. Louis.

Par exemple, supposons que nous soyons intéressés par le [taux de chômage](https://fred.stlouisfed.org/series/UNRATE).

(Pour télécharger les données au format csv, cliquez sur `Download` en haut à droite et sélectionnez l'option `CSV (data)`).

Alternativement, nous pouvons accéder au fichier CSV depuis un programme Python.

Cela peut être fait avec diverses méthodes.

Nous commençons par une méthode de relativement bas niveau, puis revenons à pandas.

### Accéder aux données avec {index}`requests <single: requests>`

```{index} single: Python; requests
```

Une option consiste à utiliser [requests](https://requests.readthedocs.io/en/latest/), une bibliothèque Python standard pour demander des données via Internet.

Pour commencer, essayez le code suivant sur votre ordinateur

```{code-cell} ipython3
r = requests.get('https://fred.stlouisfed.org/graph/fredgraph.csv?bgcolor=%23e1e9f0&chart_type=line&drp=0&fo=open%20sans&graph_bgcolor=%23ffffff&height=450&mode=fred&recession_bars=on&txtcolor=%23444444&ts=12&tts=12&width=1318&nt=0&thu=0&trc=0&show_legend=yes&show_axis_titles=yes&show_tooltip=yes&id=UNRATE&scale=left&cosd=1948-01-01&coed=2024-06-01&line_color=%234572a7&link_values=false&line_style=solid&mark_type=none&mw=3&lw=2&ost=-99999&oet=99999&mma=0&fml=a&fq=Monthly&fam=avg&fgst=lin&fgsnd=2020-02-01&line_index=1&transformation=lin&vintage_date=2024-07-29&revision_date=2024-07-29&nd=1948-01-01')
```

S'il n'y a pas de message d'erreur, c'est que l'appel a réussi.

Si vous obtenez une erreur, alors il y a deux causes probables

1. Vous n'êtes pas connecté à Internet — espérons que ce n'est pas le cas.
1. Votre machine accède à Internet via un serveur proxy, et Python n'en est pas conscient.

Dans le second cas, vous pouvez soit

* passer à une autre machine
* résoudre votre problème de proxy en lisant [la documentation](https://requests.readthedocs.io/en/latest/)

En supposant que tout fonctionne, vous pouvez maintenant utiliser l'objet `source` renvoyé par l'appel `requests.get('https://research.stlouisfed.org/fred2/series/UNRATE/downloaddata/UNRATE.csv')`

```{code-cell} ipython3
url = 'https://fred.stlouisfed.org/graph/fredgraph.csv?bgcolor=%23e1e9f0&chart_type=line&drp=0&fo=open%20sans&graph_bgcolor=%23ffffff&height=450&mode=fred&recession_bars=on&txtcolor=%23444444&ts=12&tts=12&width=1318&nt=0&thu=0&trc=0&show_legend=yes&show_axis_titles=yes&show_tooltip=yes&id=UNRATE&scale=left&cosd=1948-01-01&coed=2024-06-01&line_color=%234572a7&link_values=false&line_style=solid&mark_type=none&mw=3&lw=2&ost=-99999&oet=99999&mma=0&fml=a&fq=Monthly&fam=avg&fgst=lin&fgsnd=2020-02-01&line_index=1&transformation=lin&vintage_date=2024-07-29&revision_date=2024-07-29&nd=1948-01-01'
source = requests.get(url).content.decode().split("\n")
source[0]
```

```{code-cell} ipython3
source[1]
```

```{code-cell} ipython3
source[2]
```

Nous pourrions maintenant écrire du code supplémentaire pour analyser ce texte et le stocker sous forme de tableau.

Mais c'est inutile — la fonction `read_csv` de pandas peut gérer cette tâche pour nous.

Nous utilisons `parse_dates=True` afin que pandas reconnaisse notre colonne de dates, permettant un filtrage simple par date

```{code-cell} ipython3
data = pd.read_csv(url, index_col=0, parse_dates=True)
```

Les données ont été lues dans un DataFrame pandas appelé `data` que nous pouvons maintenant manipuler de la manière habituelle

```{code-cell} ipython3
type(data)
```

```{code-cell} ipython3
data.head()  # Une méthode utile pour jeter un coup d'œil rapide à un dataframe
```

```{code-cell} ipython3
pd.set_option('display.precision', 1)
data.describe()  # Votre sortie peut différer légèrement
```

Nous pouvons également tracer le taux de chômage de 2006 à 2012 comme suit

```{code-cell} ipython3
ax = data['2006':'2012'].plot(title='US Unemployment Rate', legend=False)
ax.set_xlabel('year', fontsize=12)
ax.set_ylabel('%', fontsize=12)
plt.show()
```

Notez que pandas offre de nombreuses autres alternatives de types de fichiers.

Pandas dispose d'une [grande variété](https://pandas.pydata.org/pandas-docs/stable/user_guide/io.html) de méthodes de haut niveau que nous pouvons utiliser pour lire des fichiers excel, json, parquet ou nous connecter directement à un serveur de base de données.

### Utiliser {index}`wbgapi <single: wbgapi>` et {index}`yfinance <single: yfinance>` pour accéder aux données

La bibliothèque Python [wbgapi](https://pypi.org/project/wbgapi/) peut être utilisée pour récupérer des données à partir des nombreuses bases de données publiées par la Banque mondiale.

```{note}
Vous pouvez trouver des informations utiles sur le paquet [wbgapi](https://pypi.org/project/wbgapi/) dans cet [article de blog de la Banque mondiale](https://blogs.worldbank.org/en/opendata/introducing-wbgapi-new-python-package-accessing-world-bank-data), en plus de ce [tutoriel](https://github.com/tgherzog/wbgapi/blob/master/examples/wbgapi-quickstart.ipynb)
```

Nous utiliserons également [yfinance](https://pypi.org/project/yfinance/) pour récupérer des données de Yahoo finance dans les exercices.

Pour l'instant, parcourons un exemple de téléchargement et de tracé de données — cette fois-ci de la Banque mondiale.

La Banque mondiale [collecte et organise des données](https://data.worldbank.org/indicator) sur un large éventail d'indicateurs.

Par exemple, [voici](https://data.worldbank.org/indicator/GC.DOD.TOTL.GD.ZS) quelques données sur la dette publique en ratio du PIB.

L'exemple de code suivant récupère les données pour vous et trace les séries temporelles pour les États-Unis et l'Australie

```{code-cell} ipython3
import wbgapi as wb
wb.series.info('GC.DOD.TOTL.GD.ZS')
```

```{code-cell} ipython3
govt_debt = wb.data.DataFrame('GC.DOD.TOTL.GD.ZS', economy=['USA','AUS'], time=range(2005,2016))
govt_debt = govt_debt.T    # déplace les années des colonnes vers les lignes pour le tracé
```

```{code-cell} ipython3
govt_debt.plot(xlabel='year', ylabel='Government debt (% of GDP)');
```

## Exercices

```{exercise-start}
:label: pd_ex1
```

Avec ces importations :

```{code-cell} ipython3
import datetime as dt
import yfinance as yf
```

Écrivez un programme pour calculer la variation en pourcentage du prix au cours de l'année 2021 pour les actions suivantes :

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

Voici la première partie du programme

```{code-cell} ipython3
def read_data(ticker_list,
          start=dt.datetime(2021, 1, 1),
          end=dt.datetime(2021, 12, 31)):
    """
    This function reads in closing price data from Yahoo
    for each tick in the ticker_list.
    """
    ticker = pd.DataFrame()

    for tick in ticker_list:
        stock = yf.Ticker(tick)
        prices = stock.history(start=start, end=end)

        # Change the index to date-only
        prices.index = pd.to_datetime(prices.index.date)
        
        closing_prices = prices['Close']
        ticker[tick] = closing_prices

    return ticker

ticker = read_data(ticker_list)
```

Complétez le programme pour tracer le résultat sous forme de diagramme en barres comme celui-ci :

```{image} /_static/lecture_specific/pandas/pandas_share_prices.png
:scale: 80
:align: center
```

```{exercise-end}
```

```{solution-start} pd_ex1
:class: dropdown
```

Il existe plusieurs façons d'aborder ce problème en utilisant Pandas pour calculer la variation en pourcentage.

Premièrement, vous pouvez extraire les données et effectuer le calcul comme suit :

```{code-cell} ipython3
p1 = ticker.iloc[0]    #Get the first set of prices as a Series
p2 = ticker.iloc[-1]   #Get the last set of prices as a Series
price_change = (p2 - p1) / p1 * 100
price_change
```

Alternativement, vous pouvez utiliser une méthode intégrée `pct_change` et la configurer pour effectuer le calcul correct en utilisant l'argument `periods`.

```{code-cell} ipython3
change = ticker.pct_change(periods=len(ticker)-1, axis='rows')*100
price_change = change.iloc[-1]
price_change
```

Puis pour tracer le graphique

```{code-cell} ipython3
price_change.sort_values(inplace=True)
price_change.rename(index=ticker_list, inplace=True)
```

```{code-cell} ipython3
fig, ax = plt.subplots(figsize=(10,8))
ax.set_xlabel('stock', fontsize=12)
ax.set_ylabel('percentage change in price', fontsize=12)
price_change.plot(kind='bar', ax=ax)
plt.show()
```

```{solution-end}
```


```{exercise-start}
:label: pd_ex2
```

En utilisant la méthode `read_data` introduite dans {ref}`pd_ex1`, écrivez un programme pour obtenir la variation en pourcentage d'une année sur l'autre pour les indices suivants :

```{code-cell} ipython3
indices_list = {'^GSPC': 'S&P 500',
               '^IXIC': 'NASDAQ',
               '^DJI': 'Dow Jones',
               '^N225': 'Nikkei'}
```

Complétez le programme pour afficher des statistiques descriptives et tracer le résultat sous forme de graphique de série temporelle comme celui-ci :

```{image} /_static/lecture_specific/pandas/pandas_indices_pctchange.png
:scale: 80
:align: center
```

```{exercise-end}
```

```{solution-start} pd_ex2
:class: dropdown
```

En suivant le travail que vous avez effectué dans {ref}`pd_ex1`, vous pouvez interroger les données en utilisant `read_data` en mettant à jour les dates de début et de fin en conséquence.

```{code-cell} ipython3
indices_data = read_data(
        indices_list,
        start=dt.datetime(1971, 1, 1),  #Common Start Date
        end=dt.datetime(2021, 12, 31)
)
```

Ensuite, extrayez le premier et le dernier ensemble de prix par année sous forme de DataFrames et calculez les rendements annuels comme suit :

```{code-cell} ipython3
yearly_returns = pd.DataFrame()

for index, name in indices_list.items():
    p1 = indices_data.groupby(indices_data.index.year)[index].first()  # Get the first set of returns as a DataFrame
    p2 = indices_data.groupby(indices_data.index.year)[index].last()   # Get the last set of returns as a DataFrame
    returns = (p2 - p1) / p1
    yearly_returns[name] = returns

yearly_returns
```

Ensuite, vous pouvez obtenir des statistiques descriptives en utilisant la méthode `describe`.

```{code-cell} ipython3
yearly_returns.describe()
```

Puis, pour tracer le graphique

```{code-cell} ipython3
fig, axes = plt.subplots(2, 2, figsize=(10, 8))

for iter_, ax in enumerate(axes.flatten()):            # Flatten 2-D array to 1-D array
    index_name = yearly_returns.columns[iter_]         # Get index name per iteration
    ax.plot(yearly_returns[index_name])                # Plot pct change of yearly returns per index
    ax.set_ylabel("percent change", fontsize = 12)
    ax.set_title(index_name)

plt.tight_layout()
```

```{solution-end}
```

[^mung]: Wikipédia définit le « munging » comme le nettoyage de données d'une forme brute vers une forme structurée et épurée.