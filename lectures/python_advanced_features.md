---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3
  language: python
  name: python3
translation:
  title: Fonctionnalités supplémentaires du langage
  headings:
    Overview: Vue d'ensemble
    Iterables and iterators: Itérables et itérateurs
    Iterables and iterators::Iterators: Itérateurs
    Iterables and iterators::Iterators in for loops: Les itérateurs dans les boucles for
    Iterables and iterators::Iterables: Itérables
    Iterables and iterators::Iterators and built-ins: Itérateurs et fonctions natives
    '`*` and `**` operators': Les opérateurs `*` et `**`
    '`*` and `**` operators::Unpacking arguments': Déballage des arguments
    '`*` and `**` operators::Arbitrary arguments': Arguments arbitraires
    Type hints: Indications de type
    Type hints::Basic syntax: Syntaxe de base
    Type hints::Common types: Types courants
    Type hints::Hints don't enforce types: Les indications n'imposent pas les types
    Type hints::Why use type hints?: "Pourquoi utiliser les indications de type\_?"
    Type hints::Type hints in scientific Python: Les indications de type dans le Python scientifique
    Decorators and descriptors: Décorateurs et descripteurs
    Decorators and descriptors::Decorators: Décorateurs
    Decorators and descriptors::Decorators::An example: Un exemple
    Decorators and descriptors::Decorators::Enter decorators: Entrée des décorateurs
    Decorators and descriptors::Descriptors: Descripteurs
    Decorators and descriptors::Descriptors::A solution: Une solution
    Decorators and descriptors::Descriptors::How it works: Comment cela fonctionne
    Decorators and descriptors::Descriptors::Decorators and properties: Décorateurs et propriétés
    Generators: Générateurs
    Generators::Generator expressions: Expressions génératrices
    Generators::Generator functions: Fonctions génératrices
    Generators::Generator functions::Example 1: Exemple 1
    Generators::Generator functions::Example 2: Exemple 2
    Generators::Advantages of iterators: Avantages des itérateurs
    Exercises: Exercices
---

(python_advanced_features)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Fonctionnalités supplémentaires du langage

## Vue d'ensemble

Pour ce dernier cours, notre conseil est de *le sauter en première lecture*, à moins que vous n'ayez une envie irrépressible de le lire.

Il se trouve ici

1. comme référence, afin que nous puissions y renvoyer lorsque cela est nécessaire, et
1. pour ceux qui ont travaillé sur un certain nombre d'applications et qui souhaitent maintenant en apprendre davantage sur le langage Python

Une variété de sujets sont traités dans ce cours, notamment les itérateurs, les indications de type, les décorateurs et descripteurs, ainsi que les générateurs.

## Itérables et itérateurs

```{index} single: Python; Iteration
```

Nous avons {ref}`déjà dit quelque chose <iterating_version_1>` sur l'itération en Python.

Examinons maintenant de plus près comment tout cela fonctionne, en nous concentrant sur l'implémentation Python de la boucle `for`.

(iterators)=
### Itérateurs

```{index} single: Python; Iterators
```

Les itérateurs constituent une interface uniforme pour parcourir les éléments d'une collection.

Ici, nous parlerons de l'utilisation des itérateurs — plus tard, nous apprendrons à construire les nôtres.

Formellement, un *itérateur* est un objet doté d'une méthode `__next__`.

Par exemple, les objets fichier sont des itérateurs.

Pour le constater, jetons un nouveau coup d'œil aux {ref}`données des villes américaines <us_cities_data>`,
qui sont écrites dans le répertoire de travail courant dans la cellule suivante

```{code-cell} ipython
%%file us_cities.txt
new york: 8244910
los angeles: 3819702
chicago: 2707120
houston: 2145146
philadelphia: 1536471
phoenix: 1469471
san antonio: 1359758
san diego: 1326179
dallas: 1223229
```

```{code-cell} python3
f = open('us_cities.txt')
f.__next__()
```

```{code-cell} python3
f.__next__()
```

Nous voyons que les objets fichier possèdent effectivement une méthode `__next__`, et que l'appel de cette méthode renvoie la ligne suivante du fichier.

La méthode next est également accessible via la fonction native `next()`,
qui appelle directement cette méthode

```{code-cell} python3
next(f)
```

Les objets renvoyés par `enumerate()` sont également des itérateurs

```{code-cell} python3
e = enumerate(['foo', 'bar'])
next(e)
```

```{code-cell} python3
next(e)
```

de même que les objets reader du module `csv`.

Créons un petit fichier csv contenant des données de l'indice NIKKEI

```{code-cell} ipython
%%file test_table.csv
Date,Open,High,Low,Close,Volume,Adj Close
2009-05-21,9280.35,9286.35,9189.92,9264.15,133200,9264.15
2009-05-20,9372.72,9399.40,9311.61,9344.64,143200,9344.64
2009-05-19,9172.56,9326.75,9166.97,9290.29,167000,9290.29
2009-05-18,9167.05,9167.82,8997.74,9038.69,147800,9038.69
2009-05-15,9150.21,9272.08,9140.90,9265.02,172000,9265.02
2009-05-14,9212.30,9223.77,9052.41,9093.73,169400,9093.73
2009-05-13,9305.79,9379.47,9278.89,9340.49,176000,9340.49
2009-05-12,9358.25,9389.61,9298.61,9298.61,188400,9298.61
2009-05-11,9460.72,9503.91,9342.75,9451.98,230800,9451.98
2009-05-08,9351.40,9464.43,9349.57,9432.83,220200,9432.83
```

```{code-cell} python3
from csv import reader

f = open('test_table.csv', 'r')
nikkei_data = reader(f)
next(nikkei_data)
```

```{code-cell} python3
next(nikkei_data)
```

### Les itérateurs dans les boucles for

```{index} single: Python; Iterators
```

Tous les itérateurs peuvent être placés à droite du mot-clé `in` dans les instructions de boucle `for`.

En fait, c'est ainsi que fonctionne la boucle `for`. Si nous écrivons

```{code-block} python3
:class: no-execute

for x in iterator:
    <code block>
```

alors l'interpréteur

* appelle `iterator.___next___()` et lie `x` au résultat
* exécute le bloc de code
* répète jusqu'à ce qu'une erreur `StopIteration` se produise

Vous savez donc maintenant comment fonctionne cette syntaxe d'apparence magique

```{code-block} python3
:class: no-execute

f = open('somefile.txt', 'r')
for line in f:
    # do something
```

L'interpréteur ne fait que

1. appeler `f.__next__()` et lier `line` au résultat
1. exécuter le corps de la boucle

Cela continue jusqu'à ce qu'une erreur `StopIteration` se produise.

### Itérables

```{index} single: Python; Iterables
```

Vous savez déjà que nous pouvons placer une liste Python à droite de `in` dans une boucle `for`

```{code-cell} python3
for i in ['spam', 'eggs']:
    print(i)
```

Cela signifie-t-il donc qu'une liste est un itérateur ?

La réponse est non

```{code-cell} python3
x = ['foo', 'bar']
type(x)
```

```{code-cell} python3
---
tags: [raises-exception]
---
next(x)
```

Alors pourquoi pouvons-nous itérer sur une liste dans une boucle `for` ?

La raison est qu'une liste est *itérable* (par opposition à un itérateur).

Formellement, un objet est itérable s'il peut être converti en un itérateur à l'aide de la fonction native `iter()`.

Les listes sont l'un de ces objets

```{code-cell} python3
x = ['foo', 'bar']
type(x)
```

```{code-cell} python3
y = iter(x)
type(y)
```

```{code-cell} python3
next(y)
```

```{code-cell} python3
next(y)
```

```{code-cell} python3
---
tags: [raises-exception]
---
next(y)
```

De nombreux autres objets sont itérables, tels que les dictionnaires et les tuples.

Bien entendu, tous les objets ne sont pas itérables

```{code-cell} python3
---
tags: [raises-exception]
---
iter(42)
```

Pour conclure notre discussion sur les boucles `for`

* Les boucles `for` fonctionnent soit sur des itérateurs, soit sur des itérables.
* Dans le second cas, l'itérable est converti en itérateur avant le début de la boucle.

### Itérateurs et fonctions natives

```{index} single: Python; Iterators
```

Certaines fonctions natives qui agissent sur les séquences fonctionnent également avec les itérables

* `max()`, `min()`, `sum()`, `all()`, `any()`

Par exemple

```{code-cell} python3
x = [10, -10]
max(x)
```

```{code-cell} python3
y = iter(x)
type(y)
```

```{code-cell} python3
max(y)
```

Une chose à retenir à propos des itérateurs est qu'ils s'épuisent à l'usage

```{code-cell} python3
x = [10, -10]
y = iter(x)
max(y)
```

```{code-cell} python3
---
tags: [raises-exception]
---
max(y)
```

## Les opérateurs `*` et `**`

`*` et `**` sont des outils pratiques et largement utilisés pour déballer des listes et des tuples et pour permettre aux utilisateurs de définir des fonctions qui prennent un nombre arbitraire d'arguments en entrée.

Dans cette section, nous explorerons comment les utiliser et distinguerons leurs cas d'usage.


### Déballage des arguments

Lorsque nous opérons sur une liste de paramètres, nous avons souvent besoin d'extraire le contenu de la liste sous forme d'arguments individuels plutôt que sous forme de collection lors de leur passage aux fonctions.

Heureusement, l'opérateur `*` peut nous aider à déballer les listes et les tuples en [*arguments positionnels*](pos_args) dans les appels de fonction.

Pour rendre les choses concrètes, considérons les exemples suivants :

Sans `*`, la fonction `print` affiche une liste

```{code-cell} python3
l1 = ['a', 'b', 'c']

print(l1)
```

Tandis que la fonction `print` affiche les éléments individuels puisque `*` déballe la liste en arguments individuels

```{code-cell} python3
print(*l1)
```

Déballer la liste à l'aide de `*` en arguments positionnels équivaut à les définir individuellement lors de l'appel de la fonction

```{code-cell} python3
print('a', 'b', 'c')
```

Cependant, l'opérateur `*` est plus pratique si nous voulons les réutiliser à nouveau

```{code-cell} python3
l1.append('d')

print(*l1)
```

De même, `**` est utilisé pour déballer des arguments.

La différence est que `**` déballe les *dictionnaires* en *arguments nommés*.

`**` est souvent utilisé lorsqu'il y a de nombreux arguments nommés que nous voulons réutiliser.

Par exemple, supposons que nous voulions tracer plusieurs graphiques en utilisant les mêmes réglages graphiques,
cela peut impliquer de définir de manière répétitive de nombreux paramètres graphiques, généralement définis à l'aide d'arguments nommés.

Dans ce cas, nous pouvons utiliser un dictionnaire pour stocker ces paramètres et utiliser `**` pour déballer les dictionnaires en arguments nommés lorsqu'ils sont nécessaires.

Parcourons ensemble un exemple simple et distinguons l'utilisation de `*` et `**`

```{code-cell} python3
import numpy as np
import matplotlib.pyplot as plt

# Mise en place du cadre et des sous-graphiques
fig, ax = plt.subplots(2, 1)
plt.subplots_adjust(hspace=0.7)

# Créer une fonction qui génère des données synthétiques
def generate_data(β_0, β_1, σ=30, n=100):
    x_values = np.arange(0, n, 1)
    y_values = β_0 + β_1 * x_values + np.random.normal(size=n, scale=σ)
    return x_values, y_values

# Stocker les arguments nommés pour les lignes et les légendes dans un dictionnaire
line_kargs = {'lw': 1.5, 'alpha': 0.7}
legend_kargs = {'bbox_to_anchor': (0., 1.02, 1., .102), 
                'loc': 3, 
                'ncol': 4,
                'mode': 'expand', 
                'prop': {'size': 7}}

β_0s = [10, 20, 30]
β_1s = [1, 2, 3]

# Utiliser une boucle for pour tracer les lignes
def generate_plots(β_0s, β_1s, idx, line_kargs, legend_kargs):
    label_list = []
    for βs in zip(β_0s, β_1s):
    
        # Utiliser * pour déballer le tuple βs et le tuple de sortie de la fonction generate_data
        # Utiliser ** pour déballer le dictionnaire d'arguments nommés pour les lignes
        ax[idx].plot(*generate_data(*βs), **line_kargs)

        label_list.append(f'$β_0 = {βs[0]}$ | $β_1 = {βs[1]}$')

    # Utiliser ** pour déballer le dictionnaire d'arguments nommés pour les légendes
    ax[idx].legend(label_list, **legend_kargs)

generate_plots(β_0s, β_1s, 0, line_kargs, legend_kargs)

# Nous pouvons facilement réutiliser et mettre à jour nos paramètres
β_1s.append(-2)
β_0s.append(40)
line_kargs['lw'] = 2
line_kargs['alpha'] = 0.4

generate_plots(β_0s, β_1s, 1, line_kargs, legend_kargs)
plt.show()
```

Dans cet exemple, `*` a déballé les paramètres regroupés `βs` et la sortie de la fonction `generate_data` stockée dans des tuples,
tandis que `**` a déballé les paramètres graphiques stockés dans `legend_kargs` et `line_kargs`.

Pour résumer, lorsque `*list`/`*tuple` et `**dictionary` sont passés dans des *appels de fonction*, ils sont déballés en arguments individuels plutôt qu'en collection.

La différence est que `*` déballera les listes et les tuples en *arguments positionnels*, tandis que `**` déballera les dictionnaires en *arguments nommés*.

### Arguments arbitraires

Lorsque nous *définissons* des fonctions, il est parfois souhaitable de permettre aux utilisateurs de placer autant d'arguments qu'ils le souhaitent dans une fonction.

Vous avez peut-être remarqué que la fonction `ax.plot()` pouvait gérer un nombre arbitraire d'arguments.

Si nous regardons la [documentation](https://github.com/matplotlib/matplotlib/blob/v3.6.2/lib/matplotlib/axes/_axes.py#L1417-L1669) de la fonction, nous pouvons voir que la fonction est définie comme

```
Axes.plot(*args, scalex=True, scaley=True, data=None, **kwargs)
```

Nous retrouvons les opérateurs `*` et `**` dans le contexte de la *définition de fonction*.

En fait, `*args` et `**kargs` sont omniprésents dans les bibliothèques scientifiques de Python pour réduire la redondance et permettre des entrées flexibles.

`*args` permet à la fonction de gérer des *arguments positionnels* de taille variable

```{code-cell} python3
l1 = ['a', 'b', 'c']
l2 = ['b', 'c', 'd']

def arb(*ls):
    print(ls)

arb(l1, l2)
```

Les entrées sont passées à la fonction et stockées dans un tuple.

Essayons davantage d'entrées

```{code-cell} python3
l3 = ['z', 'x', 'b']
arb(l1, l2, l3)
```

De même, Python nous permet d'utiliser `**kargs` pour passer un nombre arbitraire d'*arguments nommés* dans les fonctions

```{code-cell} python3
def arb(**ls):
    print(ls)

# Notez qu'il s'agit d'arguments nommés
arb(l1=l1, l2=l2)
```

Nous pouvons voir que Python utilise un dictionnaire pour stocker ces arguments nommés.

Essayons davantage d'entrées

```{code-cell} python3
arb(l1=l1, l2=l2, l3=l3)
```

Dans l'ensemble, `*args` et `**kargs` sont utilisés lors de la *définition d'une fonction* ; ils permettent à la fonction de prendre une entrée de taille arbitraire.

La différence est que les fonctions avec `*args` pourront prendre des *arguments positionnels* de taille arbitraire, tandis que `**kargs` permettra aux fonctions de prendre un nombre arbitraire d'*arguments nommés*.

## Indications de type

```{index} single: Python; Type Hints
```

Python est un langage à *typage dynamique*, ce qui signifie que vous n'avez pas besoin de déclarer les types des variables.

(Voir notre {doc}`discussion précédente <need_for_speed>` sur les types dynamiques par rapport aux types statiques.)

Cependant, Python prend en charge les **indications de type** optionnelles (également appelées annotations de type) qui vous permettent d'indiquer les types attendus des variables, des paramètres de fonction et des valeurs de retour.

Les indications de type ont été introduites à partir de Python 3.5 et ont évolué dans les versions suivantes.
Toute la syntaxe présentée ici fonctionne dans Python 3.9 et versions ultérieures.

```{note}
Les indications de type sont *ignorées par l'interpréteur Python à l'exécution* — elles n'affectent pas la manière dont votre code s'exécute. Elles sont purement informatives et servent de documentation pour les humains et les outils.
```

### Syntaxe de base

Les indications de type utilisent les deux-points `:` pour annoter les variables et les paramètres, et la flèche `->` pour annoter les types de retour.

Voici un exemple simple :

```{code-cell} python3
def greet(name: str, times: int) -> str:
    return (name + '! ') * times

greet('hello', 3)
```

Dans cette définition de fonction :

- `name: str` indique que `name` est censé être une chaîne de caractères
- `times: int` indique que `times` est censé être un entier
- `-> str` indique que la fonction renvoie une chaîne de caractères

Vous pouvez également annoter directement les variables :

```{code-cell} python3
x: int = 10
y: float = 3.14
name: str = 'Python'
```

### Types courants

Les indications de type les plus fréquemment utilisées sont les types natifs :

| Type      | Exemple                          |
|-----------|----------------------------------|
| `int`     | `x: int = 5`                    |
| `float`   | `x: float = 3.14`              |
| `str`     | `x: str = 'hello'`             |
| `bool`    | `x: bool = True`               |
| `list`    | `x: list = [1, 2, 3]`          |
| `dict`    | `x: dict = {'a': 1}`           |

Pour les conteneurs, vous pouvez spécifier les types de leurs éléments :

```{code-cell} python3
prices: list[float] = [9.99, 4.50, 2.89]
counts: dict[str, int] = {'apples': 3, 'oranges': 5}
```

### Les indications n'imposent pas les types

Un point important pour les nouveaux programmeurs Python : les indications de type ne sont *pas imposées* à l'exécution.

Python ne lèvera pas d'erreur si vous passez le « mauvais » type :

```{code-cell} python3
def add(x: int, y: int) -> int:
    return x + y

# Passe des flottants — Python ne se plaint pas
add(1.5, 2.7)
```

Les indications disent `int`, mais Python accepte volontiers les arguments `float` et renvoie `4.2` — qui n'est pas non plus un `int`.

C'est une différence essentielle par rapport aux langages à typage statique comme C ou Java, où les types incompatibles provoquent des erreurs de compilation.

### Pourquoi utiliser les indications de type ?

Si Python les ignore, pourquoi s'en soucier ?

1. **Lisibilité** : Les indications de type rendent les signatures de fonction auto-documentées. Un lecteur sait immédiatement quels types une fonction attend et renvoie.
2. **Prise en charge par l'éditeur** : Les IDE comme VS Code utilisent les indications de type pour fournir une meilleure autocomplétion, une détection d'erreurs et une documentation en ligne.
3. **Vérification des erreurs** : Des outils comme [mypy](https://mypy.readthedocs.io/en/stable/) et [pyrefly](https://pyrefly.org/) analysent les indications de type pour détecter les bogues *avant* que vous n'exécutiez votre code.
4. **Code généré par LLM** : Les grands modèles de langage produisent fréquemment du code avec des indications de type, donc comprendre la syntaxe vous aide à lire et à utiliser leur sortie.

### Les indications de type dans le Python scientifique

Les indications de type se rattachent à la discussion sur le {doc}`besoin de vitesse <need_for_speed>` :

* Les bibliothèques haute performance comme [JAX](https://docs.jax.dev/en/latest/) et [Numba](https://numba.pydata.org/) s'appuient sur la connaissance des types de variables pour compiler du code machine rapide.
* Bien que ces bibliothèques infèrent les types à l'exécution plutôt que de lire directement les indications de type Python, le *concept* est le même — une information de type explicite permet l'optimisation.
* À mesure que l'écosystème Python évolue, on s'attend à ce que le lien entre les indications de type et les outils de performance se renforce.

Pour l'instant, le principal avantage des indications de type dans le Python quotidien est la *clarté et la prise en charge par les outils*, ce qui devient de plus en plus précieux à mesure que les programmes grandissent en taille.

## Décorateurs et descripteurs

```{index} single: Python; Decorators
```

```{index} single: Python; Descriptors
```

Examinons quelques éléments de syntaxe spéciaux qui sont couramment utilisés par les développeurs Python.

Vous n'aurez peut-être pas besoin des concepts suivants immédiatement, mais vous les verrez
dans le code d'autres personnes.

Il vous faut donc les comprendre à un moment donné de votre apprentissage de Python.

### Décorateurs

```{index} single: Python; Decorators
```

Les décorateurs constituent un peu de sucre syntaxique qui, bien que facilement évitable, s'est révélé populaire.

Il est très facile de dire ce que font les décorateurs.

En revanche, il faut un peu d'effort pour expliquer *pourquoi* vous pourriez les utiliser.

#### Un exemple

Supposons que nous travaillons sur un programme qui ressemble à peu près à ceci

```{code-cell} python3
import numpy as np

def f(x):
    return np.log(np.log(x))

def g(x):
    return np.sqrt(42 * x)

# Le programme se poursuit avec divers calculs utilisant f et g
```

Supposons maintenant qu'il y ait un problème : occasionnellement, des nombres négatifs sont fournis à `f` et `g` dans les calculs qui suivent.

Si vous l'essayez, vous verrez que lorsque ces fonctions sont appelées avec des nombres négatifs, elles renvoient un objet NumPy appelé `nan`.

Cela signifie « not a number » (et indique que vous essayez d'évaluer
une fonction mathématique en un point où elle n'est pas définie).

Ce n'est peut-être pas ce que nous voulons, car cela provoque d'autres problèmes difficiles à détecter plus tard.

Supposons qu'au lieu de cela, nous voulions que le programme se termine chaque fois que cela se produit, avec un message d'erreur sensé.

Ce changement est assez facile à mettre en œuvre

```{code-cell} python3
import numpy as np

def f(x):
    assert x >= 0, "Argument must be nonnegative"
    return np.log(np.log(x))

def g(x):
    assert x >= 0, "Argument must be nonnegative"
    return np.sqrt(42 * x)

# Le programme se poursuit avec divers calculs utilisant f et g
```

Remarquez cependant qu'il y a ici une certaine répétition, sous la forme de deux lignes de code identiques.

La répétition rend notre code plus long et plus difficile à maintenir, et c'est donc
quelque chose que nous essayons vraiment d'éviter.

Ici, ce n'est pas grand-chose, mais imaginez maintenant qu'au lieu de simplement `f` et `g`, nous ayons 20 fonctions de ce type que nous devons modifier exactement de la même manière.

Cela signifie que nous devons répéter la logique de test (c'est-à-dire la ligne `assert` testant la non-négativité) 20 fois.

La situation est encore pire si la logique de test est plus longue et plus complexe.

Dans ce genre de scénario, l'approche suivante serait plus soignée

```{code-cell} python3
import numpy as np

def check_nonneg(func):
    def safe_function(x):
        assert x >= 0, "Argument must be nonnegative"
        return func(x)
    return safe_function

def f(x):
    return np.log(np.log(x))

def g(x):
    return np.sqrt(42 * x)

f = check_nonneg(f)
g = check_nonneg(g)
# Le programme se poursuit avec divers calculs utilisant f et g
```

Cela semble compliqué, alors examinons-le lentement.

Pour démêler la logique, considérons ce qui se passe lorsque nous écrivons `f = check_nonneg(f)`.

Cela appelle la fonction `check_nonneg` avec le paramètre `func` fixé à `f`.

Maintenant, `check_nonneg` crée une nouvelle fonction appelée `safe_function` qui
vérifie que `x` est non négatif puis appelle `func` sur celui-ci (ce qui revient au même que `f`).

Enfin, le nom global `f` est alors fixé à `safe_function`.

Maintenant, le comportement de `f` est tel que nous le souhaitons, et il en va de même pour `g`.

En même temps, la logique de test n'est écrite qu'une seule fois.

#### Entrée des décorateurs

```{index} single: Python; Decorators
```

La dernière version de notre code n'est toujours pas idéale.

Par exemple, si quelqu'un lit notre code et veut savoir comment
`f` fonctionne, il cherchera la définition de la fonction, qui est

```{code-cell} python3
def f(x):
    return np.log(np.log(x))
```

Il pourrait bien manquer la ligne `f = check_nonneg(f)`.

Pour cette raison et pour d'autres, les décorateurs ont été introduits en Python.

Avec les décorateurs, nous pouvons remplacer les lignes

```{code-cell} python3
def f(x):
    return np.log(np.log(x))

def g(x):
    return np.sqrt(42 * x)

f = check_nonneg(f)
g = check_nonneg(g)
```

par

```{code-cell} python3
@check_nonneg
def f(x):
    return np.log(np.log(x))

@check_nonneg
def g(x):
    return np.sqrt(42 * x)
```

Ces deux morceaux de code font exactement la même chose.

S'ils font la même chose, avons-nous vraiment besoin de la syntaxe des décorateurs ?

Eh bien, remarquez que les décorateurs se placent juste au-dessus des définitions de fonction.

Ainsi, quiconque regarde la définition de la fonction les verra et sera
conscient que la fonction est modifiée.

De l'avis de beaucoup de gens, cela fait de la syntaxe des décorateurs une amélioration significative du langage.

(descriptors)=
### Descripteurs

```{index} single: Python; Descriptors
```

Les descripteurs résolvent un problème courant concernant la gestion des variables.

Pour comprendre le problème, considérons une classe `Car`, qui simule une voiture.

Supposons que cette classe définisse les variables `miles` et `kms`, qui donnent la distance parcourue en miles
et en kilomètres respectivement.

Une version très simplifiée de la classe pourrait ressembler à ceci

```{code-cell} python3
class Car:

    def __init__(self, miles=1000):
        self.miles = miles
        self.kms = miles * 1.61

    # Certaines autres fonctionnalités, détails omis
```

Un problème potentiel que nous pourrions avoir ici est qu'un utilisateur modifie l'une de ces
variables mais pas l'autre

```{code-cell} python3
car = Car()
car.miles
```

```{code-cell} python3
car.kms
```

```{code-cell} python3
car.miles = 6000
car.kms
```

Dans les deux dernières lignes, nous voyons que `miles` et `kms` ne sont pas synchronisés.

Ce que nous voulons vraiment, c'est un mécanisme par lequel chaque fois qu'un utilisateur définit l'une de ces variables, *l'autre est automatiquement mise à jour*.

#### Une solution

En Python, ce problème est résolu à l'aide des *descripteurs*.

Un descripteur n'est qu'un objet Python qui implémente certaines méthodes.

Ces méthodes sont déclenchées lorsque l'objet est accédé via la notation d'attribut par point.

La meilleure façon de comprendre cela est de le voir en action.

Considérons cette version alternative de la classe `Car`

```{code-cell} python3
class Car:

    def __init__(self, miles=1000):
        self._miles = miles
        self._kms = miles * 1.61

    def set_miles(self, value):
        self._miles = value
        self._kms = value * 1.61

    def set_kms(self, value):
        self._kms = value
        self._miles = value / 1.61

    def get_miles(self):
        return self._miles

    def get_kms(self):
        return self._kms

    miles = property(get_miles, set_miles)
    kms = property(get_kms, set_kms)
```

Vérifions d'abord que nous obtenons le comportement souhaité

```{code-cell} python3
car = Car()
car.miles
```

```{code-cell} python3
car.miles = 6000
car.kms
```

Oui, c'est ce que nous voulons — `car.kms` est automatiquement mis à jour.

#### Comment cela fonctionne

Les noms `_miles` et `_kms` sont des noms arbitraires que nous utilisons pour stocker les valeurs des variables.

Les objets `miles` et `kms` sont des *propriétés*, un type courant de descripteur.

Les méthodes `get_miles`, `set_miles`, `get_kms` et `set_kms` définissent
ce qui se passe lorsque vous obtenez (c'est-à-dire accédez) ou définissez (liez) ces variables

* Les méthodes dites « getter » et « setter ».

La fonction native Python `property` prend les méthodes getter et setter et crée une propriété.

Par exemple, après que `car` a été créé en tant qu'instance de `Car`, l'objet `car.miles` est une propriété.

Étant une propriété, lorsque nous définissons sa valeur via `car.miles = 6000`, sa méthode setter
est déclenchée — dans ce cas `set_miles`.

#### Décorateurs et propriétés

```{index} single: Python; Decorators
```

```{index} single: Python; Properties
```

De nos jours, il est très courant de voir la fonction `property` utilisée via un décorateur.

Voici une autre version de notre classe `Car` qui fonctionne comme avant mais utilise maintenant
des décorateurs pour mettre en place les propriétés

```{code-cell} python3
class Car:

    def __init__(self, miles=1000):
        self._miles = miles
        self._kms = miles * 1.61

    @property
    def miles(self):
        return self._miles

    @property
    def kms(self):
        return self._kms

    @miles.setter
    def miles(self, value):
        self._miles = value
        self._kms = value * 1.61

    @kms.setter
    def kms(self, value):
        self._kms = value
        self._miles = value / 1.61
```

Nous n'entrerons pas dans tous les détails ici.

Pour plus d'informations, vous pouvez vous référer à la [documentation des descripteurs](https://docs.python.org/3/howto/descriptor.html).

(paf_generators)=
## Générateurs

```{index} single: Python; Generators
```

Un générateur est une sorte d'itérateur (c'est-à-dire qu'il fonctionne avec une fonction `next`).

Nous étudierons deux façons de construire des générateurs : les expressions génératrices et les fonctions génératrices.

### Expressions génératrices

La façon la plus simple de construire des générateurs consiste à utiliser des *expressions génératrices*.

Tout comme une liste en compréhension, mais avec des parenthèses rondes.

Voici la liste en compréhension :

```{code-cell} python3
singular = ('dog', 'cat', 'bird')
type(singular)
```

```{code-cell} python3
plural = [string + 's' for string in singular]
plural
```

```{code-cell} python3
type(plural)
```

Et voici l'expression génératrice

```{code-cell} python3
singular = ('dog', 'cat', 'bird')
plural = (string + 's' for string in singular)
type(plural)
```

```{code-cell} python3
next(plural)
```

```{code-cell} python3
next(plural)
```

```{code-cell} python3
next(plural)
```

Puisque `sum()` peut être appelée sur des itérateurs, nous pouvons faire ceci

```{code-cell} python3
sum((x * x for x in range(10)))
```

La fonction `sum()` appelle `next()` pour obtenir les éléments, additionne les termes successifs.

En fait, nous pouvons omettre les parenthèses externes dans ce cas

```{code-cell} python3
sum(x * x for x in range(10))
```

### Fonctions génératrices

```{index} single: Python; Generator Functions
```

La façon la plus flexible de créer des objets générateurs est d'utiliser des fonctions génératrices.

Examinons quelques exemples.

#### Exemple 1

Voici un exemple très simple de fonction génératrice

```{code-cell} python3
def f():
    yield 'start'
    yield 'middle'
    yield 'end'
```

Cela ressemble à une fonction, mais utilise un mot-clé `yield` que nous n'avons pas rencontré auparavant.

Voyons comment cela fonctionne après avoir exécuté ce code

```{code-cell} python3
type(f)
```

```{code-cell} python3
gen = f()
gen
```

```{code-cell} python3
next(gen)
```

```{code-cell} python3
next(gen)
```

```{code-cell} python3
next(gen)
```

```{code-cell} python3
---
tags: [raises-exception]
---
next(gen)
```

La fonction génératrice `f()` est utilisée pour créer des objets générateurs (dans ce cas `gen`).

Les générateurs sont des itérateurs, car ils prennent en charge une méthode `next`.

Le premier appel à `next(gen)`

* Exécute le code dans le corps de `f()` jusqu'à ce qu'il rencontre une instruction `yield`.
* Renvoie cette valeur à l'appelant de `next(gen)`.

Le second appel à `next(gen)` commence à exécuter *à partir de la ligne suivante*

```{code-cell} python3
def f():
    yield 'start'
    yield 'middle'  # Cette ligne !
    yield 'end'
```

et continue jusqu'à l'instruction `yield` suivante.

À ce moment-là, il renvoie la valeur qui suit `yield` à l'appelant de `next(gen)`, et ainsi de suite.

Lorsque le bloc de code se termine, le générateur lève une erreur `StopIteration`.

#### Exemple 2

Notre prochain exemple reçoit un argument `x` de l'appelant

```{code-cell} python3
def g(x):
    while x < 100:
        yield x
        x = x * x
```

Voyons comment cela fonctionne

```{code-cell} python3
g
```

```{code-cell} python3
gen = g(2)
type(gen)
```

```{code-cell} python3
next(gen)
```

```{code-cell} python3
next(gen)
```

```{code-cell} python3
next(gen)
```

```{code-cell} python3
---
tags: [raises-exception]
---
next(gen)
```

L'appel `gen = g(2)` lie `gen` à un générateur.

À l'intérieur du générateur, le nom `x` est lié à `2`.

Lorsque nous appelons `next(gen)`

* Le corps de `g()` s'exécute jusqu'à la ligne `yield x`, et la valeur de `x` est renvoyée.

Notez que la valeur de `x` est conservée à l'intérieur du générateur.

Lorsque nous appelons `next(gen)` à nouveau, l'exécution continue *là où elle s'était arrêtée*

```{code-cell} python3
def g(x):
    while x < 100:
        yield x
        x = x * x  # l'exécution continue à partir d'ici
```

Lorsque `x < 100` échoue, le générateur lève une erreur `StopIteration`.

Soit dit en passant, la boucle à l'intérieur du générateur peut être infinie

```{code-cell} python3
def g(x):
    while 1:
        yield x
        x = x * x
```

### Avantages des itérateurs

Quel est l'avantage d'utiliser un itérateur ici ?

Supposons que nous voulions échantillonner une loi binomiale(n, 0.5).

Une façon de le faire est la suivante

```{code-cell} python3
import random
n = 10000000
draws = [random.uniform(0, 1) < 0.5 for i in range(n)]
sum(draws)
```

Mais nous créons ici deux énormes listes, `range(n)` et `draws`.

Cela utilise beaucoup de mémoire et est très lent.

Si nous rendons `n` encore plus grand, alors ceci se produit

```{code-cell} python3
---
tags: [raises-exception]
---
n = 100000000
draws = [random.uniform(0, 1) < 0.5 for i in range(n)]
```

Nous pouvons éviter ces problèmes en utilisant des itérateurs.

Voici la fonction génératrice

```{code-cell} python3
def f(n):
    i = 1
    while i <= n:
        yield random.uniform(0, 1) < 0.5
        i += 1
```

Maintenant, faisons la somme

```{code-cell} python3
n = 10000000
draws = f(n)
draws
```

```{code-cell} python3
sum(draws)
```

En résumé, les itérables

* évitent le besoin de créer de grandes listes/tuples, et
* fournissent une interface uniforme pour l'itération qui peut être utilisée de manière transparente dans les boucles `for`


## Exercices


```{exercise-start}
:label: paf_ex1
```

Complétez le code suivant, et testez-le en utilisant [ce fichier csv](https://raw.githubusercontent.com/QuantEcon/lecture-python-programming/main/lectures/_static/lecture_specific/python_advanced_features/test_table.csv), que nous supposons que vous avez placé dans votre répertoire de travail courant

```{code-block} python3
:class: no-execute

def column_iterator(target_file, column_number):
    """A generator function for CSV files.
    When called with a file name target_file (string) and column number
    column_number (integer), the generator function returns a generator
    that steps through the elements of column column_number in file
    target_file.
    """
    # put your code here

dates = column_iterator('test_table.csv', 1)

for date in dates:
    print(date)
```

```{exercise-end}
```

```{solution-start} paf_ex1
:class: dropdown
```

Une solution est la suivante

```{code-cell} python3
def column_iterator(target_file, column_number):
    """A generator function for CSV files.
    When called with a file name target_file (string) and column number
    column_number (integer), the generator function returns a generator
    which steps through the elements of column column_number in file
    target_file.
    """
    f = open(target_file, 'r')
    for line in f:
        yield line.split(',')[column_number - 1]
    f.close()

dates = column_iterator('test_table.csv', 1)

i = 1
for date in dates:
    print(date)
    if i == 10:
        break
    i += 1
```

```{solution-end}
```