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
  title: Pandas pour les données de panel
  headings:
    Overview: Vue d'ensemble
    Slicing and Reshaping Data: Découpage et remodelage des données
    Merging Dataframes and Filling NaNs: Fusion de dataframes et remplissage des NaN
    Grouping and Summarizing Data: Groupement et synthèse des données
    Final Remarks: Remarques finales
    Exercises: Exercices
---

```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

(ppd)=
# {index}`Pandas pour les données de panel <single: Pandas pour les données de panel>`

```{index} single: Python; Pandas
```

En plus de ce qui est inclus dans Anaconda, ce cours nécessitera les bibliothèques suivantes :

```{code-cell} ipython3
:tags: [hide-output]

!pip install --upgrade seaborn
```

Nous utilisons les importations suivantes.

```{code-cell} ipython3
import matplotlib.pyplot as plt
import seaborn as sns
sns.set_theme()
```

## Vue d'ensemble

Dans un {doc}`cours précédent sur pandas <pandas>`, nous avons examiné le travail avec des ensembles de données simples.

Les économètres ont souvent besoin de travailler avec des ensembles de données plus complexes, comme les panels.

Les tâches courantes comprennent

* Importer des données, les nettoyer et les remodeler selon plusieurs axes.
* Sélectionner une série temporelle ou une coupe transversale à partir d'un panel.
* Grouper et résumer des données.

`pandas` (dérivé de « panel » et « data ») contient des outils puissants et
faciles à utiliser pour résoudre exactement ce genre de problèmes.

Dans ce qui suit, nous utiliserons un ensemble de données de panel des salaires minimums réels de l'OCDE pour créer :

* des statistiques descriptives sur plusieurs dimensions de nos données
* une série temporelle du salaire minimum moyen des pays de l'ensemble de données
* des estimations par noyau de la densité des salaires par continent

Nous commencerons par lire nos données de panel au format long à partir d'un fichier CSV et
par remodeler le `DataFrame` résultant avec `pivot_table` afin de construire un `MultiIndex`.

Des détails supplémentaires seront ajoutés à notre `DataFrame` en utilisant la fonction
`merge` de pandas, et les données seront résumées avec la fonction `groupby`.

## Découpage et remodelage des données

Nous allons lire un ensemble de données de l'OCDE des salaires minimums réels dans 32
pays et l'assigner à `realwage`.

L'ensemble de données est accessible via le lien suivant :

```{code-cell} ipython3
url1 = 'https://raw.githubusercontent.com/QuantEcon/lecture-python-programming/main/lectures/_static/lecture_specific/pandas_panel/realwage.csv'
```

```{code-cell} ipython3
import pandas as pd

# Afficher 6 colonnes à des fins de visualisation
pd.set_option('display.max_columns', 6)

# Réduire les décimales à 2
pd.options.display.float_format = '{:,.2f}'.format

realwage = pd.read_csv(url1)
```

Regardons ce avec quoi nous devons travailler

```{code-cell} ipython3
realwage.head()  # Afficher les 5 premières lignes
```

Les données sont actuellement au format long, ce qui est difficile à analyser lorsque les données comportent plusieurs dimensions.

Nous utiliserons `pivot_table` pour créer un panel au format large, avec un `MultiIndex` pour gérer les données de dimension supérieure.

Les arguments de `pivot_table` doivent spécifier les données (values), l'index et les colonnes que nous voulons dans notre dataframe résultant.

En passant une liste dans columns, nous pouvons créer un `MultiIndex` dans notre axe des colonnes

```{code-cell} ipython3
realwage = realwage.pivot_table(values='value',
                                index='Time',
                                columns=['Country', 'Series', 'Pay period'])
realwage.head()
```

Pour filtrer plus facilement nos données de série temporelle par la suite, nous convertirons l'index en `DateTimeIndex`

```{code-cell} ipython3
realwage.index = pd.to_datetime(realwage.index)
type(realwage.index)
```

Les colonnes contiennent plusieurs niveaux d'indexation, appelés
`MultiIndex`, les niveaux étant ordonnés hiérarchiquement (Country >
Series > Pay period).

Un `MultiIndex` est le moyen le plus simple et le plus flexible de gérer les données de panel
dans pandas

```{code-cell} ipython3
type(realwage.columns)
```

```{code-cell} ipython3
realwage.columns.names
```

Comme précédemment, nous pouvons sélectionner le pays (le niveau le plus élevé de notre
`MultiIndex`)

```{code-cell} ipython3
realwage['United States'].head()
```

L'empilement et le désempilement des niveaux du `MultiIndex` seront utilisés
tout au long de ce cours pour remodeler notre dataframe dans le format dont nous avons besoin.

`.stack()` fait pivoter le niveau le plus bas du `MultiIndex` des colonnes vers
l'index des lignes (`.unstack()` fonctionne dans la direction opposée — essayez-le)

```{code-cell} ipython3
realwage.stack(future_stack=True).head()
```

Nous pouvons également passer un argument pour sélectionner le niveau que nous aimerions
empiler

```{code-cell} ipython3
realwage.stack(level='Country', future_stack=True).head()  # future_stack=True est requis jusqu'à pandas>3.0
```

L'utilisation d'un `DatetimeIndex` facilite la sélection d'une période temporelle
particulière.

Sélectionner une année et empiler les deux niveaux inférieurs du
`MultiIndex` crée une coupe transversale de nos données de panel

```{code-cell} ipython3
realwage.loc['2015'].stack(level=(1, 2), future_stack=True).transpose().head() # future_stack=True est requis jusqu'à pandas>3.0
```

Pour le reste du cours, nous travaillerons avec un dataframe des salaires minimums
réels horaires par pays et par période, mesurés en dollars américains de
2015.

Pour créer notre dataframe filtré (`realwage_f`), nous pouvons utiliser la méthode `xs`
pour sélectionner des valeurs aux niveaux inférieurs du multiindex, tout en conservant
les niveaux supérieurs (les pays dans ce cas)

```{code-cell} ipython3
realwage_f = realwage.xs(('Hourly', 'In 2015 constant prices at 2015 USD exchange rates'),
                         level=('Pay period', 'Series'), axis=1)
realwage_f.head()
```

## Fusion de dataframes et remplissage des NaN

Similairement aux bases de données relationnelles comme SQL, pandas dispose de méthodes intégrées pour
fusionner des ensembles de données ensemble.

En utilisant les informations sur les pays de
[WorldData.info](https://www.worlddata.info/downloads/), nous ajouterons
le continent de chaque pays à `realwage_f` avec la fonction `merge`.

L'ensemble de données est accessible via le lien suivant :

```{code-cell} ipython3
url2 = 'https://raw.githubusercontent.com/QuantEcon/lecture-python-programming/main/lectures/_static/lecture_specific/pandas_panel/countries.csv'
```

```{code-cell} ipython3
worlddata = pd.read_csv(url2, sep=';')
worlddata.head()
```

Tout d'abord, nous sélectionnerons uniquement les variables country et continent de
`worlddata` et renommerons la colonne en « Country »

```{code-cell} ipython3
worlddata = worlddata[['Country (en)', 'Continent']]
worlddata = worlddata.rename(columns={'Country (en)': 'Country'})
worlddata.head()
```

Nous voulons fusionner notre nouveau dataframe, `worlddata`, avec `realwage_f`.

La fonction `merge` de pandas permet de joindre des dataframes ensemble par
lignes.

Nos dataframes seront fusionnés en utilisant les noms de pays, ce qui nous oblige à utiliser
la transposée de `realwage_f` afin que les lignes correspondent aux noms de pays
dans les deux dataframes

```{code-cell} ipython3
realwage_f.transpose().head()
```

Nous pouvons utiliser une jointure left, right, inner ou outer pour fusionner nos
ensembles de données :

* la jointure left n'inclut que les pays de l'ensemble de données de gauche
* la jointure right n'inclut que les pays de l'ensemble de données de droite
* la jointure outer inclut les pays qui sont soit dans l'ensemble de gauche, soit dans celui de droite
* la jointure inner n'inclut que les pays communs aux deux ensembles de données de gauche et de droite

Par défaut, `merge` utilisera une jointure inner.

Ici, nous passerons `how='left'` pour conserver tous les pays de
`realwage_f`, mais écarter les pays de `worlddata` qui n'ont pas
d'entrée de données correspondante dans `realwage_f`.

Ceci est illustré par la zone ombrée en rouge dans le diagramme suivant

```{figure} /_static/lecture_specific/pandas_panel/venn_diag.png
```

Nous devrons également spécifier où se trouve le nom du pays dans chaque
dataframe, qui sera la `key` utilisée pour fusionner les
dataframes (« on »).

Notre dataframe « left » (`realwage_f.transpose()`) contient les pays dans
l'index, donc nous définissons `left_index=True`.

Notre dataframe « right » (`worlddata`) contient les pays dans la
colonne « Country », donc nous définissons `right_on='Country'`

```{code-cell} ipython3
merged = pd.merge(realwage_f.transpose(), worlddata,
                  how='left', left_index=True, right_on='Country')
merged.head()
```

Les pays qui apparaissaient dans `realwage_f` mais pas dans `worlddata` auront
`NaN` dans la colonne Continent.

Pour vérifier si cela s'est produit, nous pouvons utiliser `.isnull()` sur la
colonne continent et filtrer le dataframe fusionné

```{code-cell} ipython3
merged[merged['Continent'].isnull()]
```

Nous avons trois valeurs manquantes !

Une option pour traiter les valeurs NaN consiste à créer un dictionnaire contenant
ces pays et leurs continents respectifs.

`.map()` fera correspondre les pays de `merged['Country']` avec leur
continent à partir du dictionnaire.

Remarquez comment les pays qui ne figurent pas dans notre dictionnaire sont mappés avec `NaN`

```{code-cell} ipython3
missing_continents = {'Korea': 'Asia',
                      'Russian Federation': 'Europe',
                      'Slovak Republic': 'Europe'}

merged['Country'].map(missing_continents)
```

Nous ne voulons pas écraser toute la série avec ce mappage.

`.fillna()` ne remplit que les valeurs `NaN` dans `merged['Continent']`
avec le mappage, tout en laissant les autres valeurs de la colonne inchangées

```{code-cell} ipython3
merged['Continent'] = merged['Continent'].fillna(merged['Country'].map(missing_continents))

# Vérifier si les continents ont été correctement mappés

merged[merged['Country'] == 'Korea']
```

Nous combinerons également les Amériques en un seul continent — cela rendra notre visualisation plus agréable par la suite.

Pour ce faire, nous utiliserons `.replace()` et boucler sur une liste des valeurs de continent que nous voulons remplacer

```{code-cell} ipython3
replace = ['Central America', 'North America', 'South America']
merged['Continent'] = merged['Continent'].replace(to_replace=replace, value='America')
```

Maintenant que nous avons toutes les données que nous voulons dans un seul `DataFrame`, nous allons
le remodeler à nouveau sous forme de panel avec un `MultiIndex`.

Nous devrions également veiller à trier l'index en utilisant `.sort_index()` afin de
pouvoir filtrer efficacement notre dataframe par la suite.

Par défaut, les niveaux seront triés de haut en bas

```{code-cell} ipython3
merged = merged.set_index(['Continent', 'Country']).sort_index()
merged.head()
```

Lors de la fusion, nous avons perdu notre `DatetimeIndex`, car nous avons fusionné des colonnes qui
n'étaient pas au format datetime

```{code-cell} ipython3
merged.columns
```

Maintenant que nous avons défini les colonnes fusionnées comme index, nous pouvons recréer un
`DatetimeIndex` en utilisant `.to_datetime()`

```{code-cell} ipython3
merged.columns = pd.to_datetime(merged.columns)
merged.columns = merged.columns.rename('Time')
merged.columns
```

Le `DatetimeIndex` a tendance à fonctionner plus harmonieusement dans l'axe des lignes, donc nous
allons transposer `merged`

```{code-cell} ipython3
merged = merged.transpose()
merged.head()
```

## Groupement et synthèse des données

Le groupement et la synthèse des données peuvent être particulièrement utiles pour
comprendre les grands ensembles de données de panel.

Un moyen simple de résumer les données consiste à appeler une [méthode
d'agrégation](https://pandas.pydata.org/pandas-docs/stable/getting_started/intro_tutorials/06_calculate_statistics.html)
sur le dataframe, comme `.mean()` ou `.max()`.

Par exemple, nous pouvons calculer le salaire minimum réel moyen pour chaque
pays sur la période de 2006 à 2016 (par défaut, l'agrégation se fait sur les
lignes)

```{code-cell} ipython3
merged.mean().head(10)
```

En utilisant cette série, nous pouvons tracer le salaire minimum réel moyen au cours de la
dernière décennie pour chaque pays de notre ensemble de données

```{code-cell} ipython3
merged.mean().sort_values(ascending=False).plot(kind='bar',
                                                title="Salaire minimum réel moyen 2006 - 2016")

# Définir les étiquettes de pays
country_labels = merged.mean().sort_values(ascending=False).index.get_level_values('Country').tolist()
plt.xticks(range(0, len(country_labels)), country_labels)
plt.xlabel('Pays')

plt.show()
```

Passer `axis=1` à `.mean()` agrégera sur les colonnes (donnant
le salaire minimum moyen pour tous les pays au fil du temps)

```{code-cell} ipython3
merged.mean(axis=1).head()
```

Nous pouvons tracer cette série temporelle sous forme de graphique linéaire

```{code-cell} ipython3
merged.mean(axis=1).plot()
plt.title('Salaire minimum réel moyen 2006 - 2016')
plt.ylabel('USD 2015')
plt.xlabel('Année')
plt.show()
```

Nous pouvons également spécifier un niveau du `MultiIndex` (dans l'axe des colonnes)
sur lequel agréger.

Dans le cas de `groupby`, nous devons utiliser `.T` pour transposer les colonnes en lignes car `pandas` a déprécié l'utilisation de `axis=1` dans la méthode `groupby`.

```{code-cell} ipython3
merged.T.groupby(level='Continent').mean().head()
```

Nous pouvons tracer les salaires minimums moyens de chaque continent sous forme de série temporelle

```{code-cell} ipython3
merged.T.groupby(level='Continent').mean().T.plot()
plt.title('Salaire minimum réel moyen')
plt.ylabel('USD 2015')
plt.xlabel('Année')
plt.show()
```

Nous exclurons l'Australie en tant que continent à des fins de traçage

```{code-cell} ipython3
merged = merged.drop('Australia', level='Continent', axis=1)
merged.T.groupby(level='Continent').mean().T.plot()
plt.title('Salaire minimum réel moyen')
plt.ylabel('USD 2015')
plt.xlabel('Année')
plt.show()
```

`.describe()` est utile pour récupérer rapidement un certain nombre de
statistiques descriptives courantes

```{code-cell} ipython3
merged.stack(future_stack=True).describe()
```

Il s'agit d'une manière simplifiée d'utiliser `groupby`.

L'utilisation de `groupby` suit généralement un processus « diviser-appliquer-combiner » :

* diviser : les données sont groupées en fonction d'une ou plusieurs clés
* appliquer : une fonction est appelée sur chaque groupe indépendamment
* combiner : les résultats des appels de fonction sont combinés dans une nouvelle structure de données

La méthode `groupby` réalise la première étape de ce processus, créant
un nouvel objet `DataFrameGroupBy` avec les données divisées en groupes.

Divisons à nouveau `merged` par continent, cette fois en utilisant la
fonction `groupby`, et nommons l'objet résultant `grouped`

```{code-cell} ipython3
grouped = merged.T.groupby(level='Continent')
grouped
```

L'appel d'une méthode d'agrégation sur l'objet applique la fonction à chaque
groupe, dont les résultats sont combinés dans une nouvelle structure de données.

Par exemple, nous pouvons retourner le nombre de pays de notre ensemble de données pour
chaque continent en utilisant `.size()`.

Dans ce cas, notre nouvelle structure de données est une `Series`

```{code-cell} ipython3
grouped.size()
```

En appelant `.get_group()` pour retourner uniquement les pays d'un seul groupe,
nous pouvons créer une estimation par noyau de la densité de la distribution des salaires
minimums réels en 2016 pour chaque continent.

`grouped.groups.keys()` retournera les clés de l'objet `groupby`

```{code-cell} ipython3
continents = grouped.groups.keys()

for continent in continents:
    sns.kdeplot(grouped.get_group(continent).T.loc['2015'].unstack(), label=continent, fill=True)

plt.title('Salaires minimums réels en 2015')
plt.xlabel('Dollars américains')
plt.legend()
plt.show()
```

## Remarques finales

Ce cours a fourni une introduction à certaines des fonctionnalités plus
avancées de pandas, notamment les multiindices, la fusion, le groupement et
le traçage.

D'autres outils qui peuvent être utiles dans l'analyse des données de panel comprennent [xarray](https://docs.xarray.dev/en/stable/), un package python qui
étend pandas aux structures de données à N dimensions.

## Exercices

```{exercise-start}
:label: pp_ex1
```

Dans ces exercices, vous travaillerez avec un ensemble de données de taux
d'emploi en Europe par âge et par sexe provenant d'[Eurostat](https://ec.europa.eu/eurostat/data/database).

L'ensemble de données est accessible via le lien suivant :

```{code-cell} ipython3
url3 = 'https://raw.githubusercontent.com/QuantEcon/lecture-python-programming/main/lectures/_static/lecture_specific/pandas_panel/employ.csv'
```

La lecture du fichier CSV retourne un ensemble de données de panel au format long. Utilisez `.pivot_table()` pour construire
un dataframe au format large avec un `MultiIndex` dans les colonnes.

Commencez par explorer le dataframe et les variables disponibles dans les
niveaux du `MultiIndex`.

Écrivez un programme qui retourne rapidement toutes les valeurs du `MultiIndex`.

```{exercise-end}
```

```{solution-start} pp_ex1
:class: dropdown
```

```{code-cell} ipython3
employ = pd.read_csv(url3)
employ = employ.pivot_table(values='Value',
                            index=['DATE'],
                            columns=['UNIT','AGE', 'SEX', 'INDIC_EM', 'GEO'])
employ.index = pd.to_datetime(employ.index) # s'assurer que les dates sont au format datetime
employ.head()
```

Il s'agit d'un grand ensemble de données, il est donc utile d'explorer les niveaux et
les variables disponibles

```{code-cell} ipython3
employ.columns.names
```

Les variables au sein des niveaux peuvent être rapidement récupérées avec une boucle

```{code-cell} ipython3
for name in employ.columns.names:
    print(name, employ.columns.get_level_values(name).unique())
```

```{solution-end}
```

```{exercise-start}
:label: pp_ex2
```

Filtrez le dataframe ci-dessus pour n'inclure que l'emploi en pourcentage de
la « population active ».

Créez un diagramme en boîte groupé en utilisant `seaborn` des taux d'emploi en 2015
par groupe d'âge et par sexe.

```{hint}
:class: dropdown

`GEO` inclut à la fois des zones et des pays.
```

```{exercise-end}
```

```{solution-start} pp_ex2
:class: dropdown
```

Pour filtrer facilement par pays, déplacez `GEO` au niveau supérieur et triez le
`MultiIndex`

```{code-cell} ipython3
employ.columns = employ.columns.swaplevel(0,-1)
employ = employ.sort_index(axis=1)
```

Nous devons nous débarrasser de quelques éléments dans `GEO` qui ne sont pas des pays.

Un moyen rapide de se débarrasser des zones de l'UE consiste à utiliser une compréhension de liste pour
trouver les valeurs de niveau dans `GEO` qui commencent par « Euro »

```{code-cell} ipython3
geo_list = employ.columns.get_level_values('GEO').unique().tolist()
countries = [x for x in geo_list if not x.startswith('Euro')]
employ = employ[countries]
employ.columns.get_level_values('GEO').unique()
```

Sélectionnez uniquement le pourcentage employé dans la population active à partir du
dataframe

```{code-cell} ipython3
employ_f = employ.xs(('Percentage of total population', 'Active population'),
                     level=('UNIT', 'INDIC_EM'),
                     axis=1)
employ_f.head()
```

Supprimez la valeur « Total » avant de créer le diagramme en boîte groupé

```{code-cell} ipython3
employ_f = employ_f.drop('Total', level='SEX', axis=1)
```

```{code-cell} ipython3
box = employ_f.loc['2015'].unstack().reset_index()
sns.boxplot(x="AGE", y=0, hue="SEX", data=box, palette=("husl"), showfliers=False)
plt.xlabel('')
plt.xticks(rotation=35)
plt.ylabel('Pourcentage de la population (%)')
plt.title('Emploi en Europe (2015)')
plt.legend(bbox_to_anchor=(1,0.5))
plt.show()
```

```{solution-end}
```
