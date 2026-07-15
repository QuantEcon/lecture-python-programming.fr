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
  title: Fondamentaux de Python
  headings:
    Overview: Vue d'ensemble
    Data Types: Types de données
    Data Types::Primitive Data Types: Types de données primitifs
    Data Types::Primitive Data Types::Boolean Values: Valeurs booléennes
    Data Types::Primitive Data Types::Numeric Types: Types numériques
    Data Types::Containers: Conteneurs
    Data Types::Containers::Slice Notation: Notation de tranche
    Data Types::Containers::Sets and Dictionaries: Ensembles et dictionnaires
    Input and Output: Entrée et sortie
    Input and Output::Paths: Chemins
    Iterating: Itération
    Iterating::Looping over Different Objects: Boucler sur différents objets
    Iterating::Looping without Indices: Boucler sans indices
    Iterating::List Comprehensions: Compréhensions de liste
    Comparisons and Logical Operators: Comparaisons et opérateurs logiques
    Comparisons and Logical Operators::Comparisons: Comparaisons
    Comparisons and Logical Operators::Combining Expressions: Combiner des expressions
    Coding Style and Documentation: Style de codage et documentation
    'Coding Style and Documentation::Python Style Guidelines: PEP8': "Directives de style Python\_: PEP8"
    Coding Style and Documentation::Docstrings: Docstrings
    Exercises: Exercices
---

(python_done_right)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Fondamentaux de Python

## Vue d'ensemble

Nous avons couvert beaucoup de matière assez rapidement, en nous concentrant sur des exemples.

Voyons maintenant certaines fonctionnalités fondamentales de Python de manière plus systématique.

Cette approche est moins passionnante mais aide à clarifier certains détails.

## Types de données

```{index} single: Python; Data Types
```

Les programmes informatiques suivent généralement toute une gamme de types de données.

Par exemple, `1.5` est un nombre à virgule flottante, tandis que `1` est un entier.

Les programmes doivent distinguer ces deux types pour diverses raisons.

L'une est qu'ils sont stockés différemment en mémoire.

Une autre est que les opérations arithmétiques sont différentes

* Par exemple, l'arithmétique en virgule flottante est mise en œuvre sur la plupart des machines par une
  unité de calcul en virgule flottante (FPU) spécialisée.

En général, les flottants sont plus informatifs mais les opérations arithmétiques sur les entiers
sont plus rapides et plus précises.

Python fournit de nombreux autres types de données Python intégrés, dont certains que nous avons déjà rencontrés

* chaînes de caractères, listes, etc.

Apprenons-en un peu plus à leur sujet.

### Types de données primitifs

(boolean)=
#### Valeurs booléennes

Un type de données simple est celui des **valeurs booléennes**, qui peuvent être soit `True`, soit `False`

```{code-cell} python3
x = True
x
```

Nous pouvons vérifier le type de n'importe quel objet en mémoire à l'aide de la fonction `type()`.

```{code-cell} python3
type(x)
```

Dans la ligne de code suivante, l'interpréteur évalue l'expression à droite de = et lie y à cette valeur

```{code-cell} python3
y = 100 < 10
y
```

```{code-cell} python3
type(y)
```

Dans les expressions arithmétiques, `True` est converti en `1` et `False` est converti en `0`.

Cela s'appelle l'**arithmétique booléenne** et est souvent utile en programmation.

Voici quelques exemples

```{code-cell} python3
x + y
```

```{code-cell} python3
x * y
```

```{code-cell} python3
True + True
```

```{code-cell} python3
bools = [True, True, False, True]  # Liste de valeurs booléennes

sum(bools)
```

#### Types numériques

Les types numériques sont également des types de données primitifs importants.

Nous avons déjà vu les types `integer` et `float`.

Les **nombres complexes** sont un autre type de données primitif en Python

```{code-cell} python3
x = complex(1, 2)
y = complex(2, 1)
print(x * y)

type(x)
```

### Conteneurs

Python possède plusieurs types de base pour stocker des collections de données (éventuellement hétérogènes).

Nous avons {ref}`déjà abordé les listes <lists_ref>`.

```{index} single: Python; Tuples
```

Un type de données apparenté est celui des **tuples**, qui sont des listes « immuables »

```{code-cell} python3
x = ('a', 'b')  # Parenthèses au lieu des crochets
x = 'a', 'b'    # Ou sans crochets --- la signification est identique
x
```

```{code-cell} python3
type(x)
```

En Python, un objet est appelé **immuable** si, une fois créé, l'objet ne peut plus être modifié.

Inversement, un objet est **mutable** s'il peut encore être modifié après sa création.

Les listes Python sont mutables

```{code-cell} python3
x = [1, 2]
x[0] = 10
x
```

Mais les tuples ne le sont pas

```{code-cell} python3
---
tags: [raises-exception]
---
x = (1, 2)
x[0] = 10
```

Nous en dirons davantage sur le rôle des données mutables et immuables un peu plus tard.

Les tuples (et les listes) peuvent être « déballés » comme suit

```{code-cell} python3
integers = (10, 20, 30)
x, y, z = integers
x
```

```{code-cell} python3
y
```

Vous avez en fait {ref}`déjà vu un exemple de cela <tuple_unpacking_example>`.

Le déballage de tuple est pratique et nous l'utiliserons souvent.

#### Notation de tranche

```{index} single: Python; Slicing
```

Pour accéder à plusieurs éléments d'une séquence (une liste, un tuple ou une chaîne), vous pouvez utiliser la notation de tranche
de Python.

Par exemple,

```{code-cell} python3
a = ["a", "b", "c", "d", "e"]
a[1:]
```

```{code-cell} python3
a[1:3]
```

La règle générale est que `a[m:n]` renvoie `n - m` éléments, en commençant par `a[m]`.

Les nombres négatifs sont également autorisés

```{code-cell} python3
a[-2:]  # Les deux derniers éléments de la liste
```

Vous pouvez également utiliser le format `[start:end:step]` pour spécifier le pas

```{code-cell} python3
a[::2]
```

En utilisant un pas négatif, vous pouvez renvoyer la séquence dans l'ordre inverse

```{code-cell} python3
a[-2::-1] # Parcourir à rebours de l'avant-dernier élément jusqu'au premier élément
```

La même notation de tranche fonctionne sur les tuples et les chaînes

```{code-cell} python3
s = 'foobar'
s[-3:]  # Sélectionner les trois derniers éléments
```

#### Ensembles et dictionnaires

```{index} single: Python; Sets
```

```{index} single: Python; Dictionaries
```

Deux autres types de conteneurs que nous devrions mentionner avant de poursuivre sont les [ensembles](https://docs.python.org/3/tutorial/datastructures.html#sets) et les [dictionnaires](https://docs.python.org/3/tutorial/datastructures.html#dictionaries).

Les dictionnaires ressemblent beaucoup aux listes, sauf que les éléments sont nommés au lieu d'être
numérotés

```{code-cell} python3
d = {'name': 'Frodo', 'age': 33}
type(d)
```

```{code-cell} python3
d['age']
```

Les noms `'name'` et `'age'` sont appelés les *clés*.

Les objets vers lesquels les clés sont mappées (`'Frodo'` et `33`) sont appelés les `values` (valeurs).

Les ensembles sont des collections non ordonnées sans doublons, et les méthodes d'ensemble fournissent les
opérations ensemblistes habituelles

```{code-cell} python3
s1 = {'a', 'b'}
type(s1)
```

```{code-cell} python3
s2 = {'b', 'c'}
s1.issubset(s2)
```

```{code-cell} python3
s1.intersection(s2)
```

La fonction `set()` crée des ensembles à partir de séquences

```{code-cell} python3
s3 = set(('foo', 'bar', 'foo'))
s3
```

## Entrée et sortie

```{index} single: Python; IO
```

Passons brièvement en revue la lecture et l'écriture dans des fichiers texte, en commençant par l'écriture

```{code-cell} python3
f = open('newfile.txt', 'w')   # Ouvrir 'newfile.txt' en écriture
f.write('Testing\n')           # Ici '\n' signifie nouvelle ligne
f.write('Testing again')
f.close()
```

Ici

* La fonction intégrée `open()` crée un objet fichier pour écrire dedans.
* `write()` et `close()` sont tous deux des méthodes des objets fichier.

Où se trouve ce fichier que nous avons créé ?

Rappelez-vous que Python maintient une notion de répertoire de travail courant (pwd) qui peut être localisé depuis Jupyter ou IPython via

```{code-cell} ipython
%pwd
```

Si aucun chemin n'est spécifié, c'est là que Python écrit.

Nous pouvons également utiliser Python pour lire le contenu de `newline.txt` comme suit

```{code-cell} python3
f = open('newfile.txt', 'r')
out = f.read()
out
```

```{code-cell} python3
print(out)
```

En fait, l'approche recommandée dans le Python moderne est d'utiliser une instruction `with` pour garantir que les fichiers sont correctement acquis et libérés.

Contenir les opérations dans le même bloc améliore aussi la clarté de votre code.

```{note}
Ce type de bloc est formellement appelé un [*contexte*](https://realpython.com/python-with-statement/#the-with-statement-approach).
```

Essayons de convertir les deux exemples ci-dessus en une instruction `with`.

Nous modifions d'abord l'exemple d'écriture
```{code-cell} python3

with open('newfile.txt', 'w') as f:  
    f.write('Testing\n')         
    f.write('Testing again')
```

Notez que nous n'avons pas besoin d'appeler la méthode `close()` puisque le bloc `with`
garantira que le flux est fermé à la fin du bloc.

Avec de légères modifications, nous pouvons également lire des fichiers en utilisant `with`

```{code-cell} python3
with open('newfile.txt', 'r') as fo:
    out = fo.read()
    print(out)
```
Supposons maintenant que nous voulions lire une entrée depuis un fichier et écrire une sortie dans un autre.
Voici comment nous pourrions accomplir cette tâche tout en acquérant et en restituant correctement
les ressources au système d'exploitation à l'aide d'instructions `with` :

```{code-cell} python3
with open("newfile.txt", "r") as f:
    file = f.readlines()
    with open("output.txt", "w") as fo:
        for i, line in enumerate(file):
            fo.write(f'Line {i}: {line} \n')
```

Le fichier de sortie sera

```{code-cell} python3
with open('output.txt', 'r') as fo:
    print(fo.read())
```

Nous pouvons simplifier l'exemple ci-dessus en regroupant les deux instructions `with` sur une seule ligne

```{code-cell} python3
with open("newfile.txt", "r") as f, open("output2.txt", "w") as fo:
        for i, line in enumerate(f):
            fo.write(f'Line {i}: {line} \n')
```

Le fichier de sortie sera identique

```{code-cell} python3
with open('output2.txt', 'r') as fo:
    print(fo.read())
```

Supposons que nous voulions continuer à écrire dans le fichier existant
au lieu de l'écraser.

nous pouvons passer au mode `a` qui signifie mode d'ajout (append)

```{code-cell} python3
with open('output2.txt', 'a') as fo:
    fo.write('\nThis is the end of the file')
```

```{code-cell} python3
with open('output2.txt', 'r') as fo:
    print(fo.read())
```

```{note}
Notez que nous n'avons couvert ici que les modes `r`, `w` et `a`, qui sont les modes les plus couramment utilisés.
Python fournit [une variété de modes](https://www.geeksforgeeks.org/python/reading-writing-text-files-python/)
que vous pourriez expérimenter.
```

### Chemins

```{index} single: Python; Paths
```

Notez que si `newfile.txt` ne se trouve pas dans le répertoire de travail courant, alors cet appel à `open()` échoue.

Dans ce cas, vous pouvez déplacer le fichier vers le pwd ou spécifier le [chemin complet](https://en.wikipedia.org/wiki/Path_%28computing%29) vers le fichier

```{code-block} python3
:class: no-execute

f = open('insert_full_path_to_file/newfile.txt', 'r')
```

(iterating_version_1)=
## Itération

```{index} single: Python; Iteration
```

L'une des tâches les plus importantes en informatique est de parcourir une
séquence de données et d'effectuer une action donnée.

L'une des forces de Python est son interface simple et flexible pour ce type d'itération via
la boucle `for`.

### Boucler sur différents objets

De nombreux objets Python sont « itérables », dans le sens où l'on peut boucler dessus.

Pour donner un exemple, écrivons le fichier us_cities.txt, qui liste des villes américaines et leur population, dans le répertoire de travail courant.

(us_cities_data)=
```{code-cell} ipython
%%writefile us_cities.txt
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

Ici `%%writefile` est une [commande magique de cellule IPython](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cell-magics).

Supposons que nous voulions rendre l'information plus lisible, en mettant les noms en majuscules et en ajoutant des virgules pour marquer les milliers.

Le programme ci-dessous lit les données et effectue la conversion :

```{code-cell} python3
data_file = open('us_cities.txt', 'r')
for line in data_file:
    city, population = line.split(':')         # Déballage de tuple
    city = city.title()                        # Mettre en majuscule les noms de ville
    population = f'{int(population):,}'        # Ajouter des virgules aux nombres
    print(city.ljust(15) + population)
data_file.close()
```

Ici `f'` est une f-string [utilisée pour insérer des variables dans des chaînes](https://docs.python.org/3/library/string.html#formatspec).

Le reformatage de chaque ligne est le résultat de trois méthodes de chaîne différentes,
dont les détails peuvent être laissés pour plus tard.

La partie intéressante de ce programme pour nous est la ligne 2, qui montre que

1. L'objet fichier `data_file` est itérable, dans le sens où il peut être placé à droite de `in` dans une boucle `for`.
1. L'itération parcourt chaque ligne du fichier.

Cela conduit à la syntaxe propre et pratique présentée dans notre programme.

De nombreux autres types d'objets sont itérables, et nous en aborderons certains plus tard.

### Boucler sans indices

Une chose que vous avez peut-être remarquée est que Python tend à privilégier les boucles sans indexation explicite.

Par exemple,

```{code-cell} python3
x_values = [1, 2, 3]  # Un x itérable
for x in x_values:
    print(x * x)
```

est préféré à

```{code-cell} python3
for i in range(len(x_values)):
    print(x_values[i] * x_values[i])
```

Lorsque vous comparez ces deux alternatives, vous pouvez voir pourquoi la première est préférée.

Python fournit certaines facilités pour simplifier les boucles sans indices.

L'une est `zip()`, qui est utilisée pour parcourir des paires provenant de deux séquences.

Par exemple, essayez d'exécuter le code suivant

```{code-cell} python3
countries = ('Japan', 'Korea', 'China')
cities = ('Tokyo', 'Seoul', 'Beijing')
for country, city in zip(countries, cities):
    print(f'The capital of {country} is {city}')
```

La fonction `zip()` est également utile pour créer des dictionnaires --- par
exemple

```{code-cell} python3
names = ['Tom', 'John']
marks = ['E', 'F']
dict(zip(names, marks))
```

Si nous avons réellement besoin de l'indice d'une liste, une option est d'utiliser `enumerate()`.

Pour comprendre ce que fait `enumerate()`, considérez l'exemple suivant

```{code-cell} python3
letter_list = ['a', 'b', 'c']
for index, letter in enumerate(letter_list):
    print(f"letter_list[{index}] = '{letter}'")
```
(list_comprehensions)=
### Compréhensions de liste

```{index} single: Python; List comprehension
```

Nous pouvons également simplifier considérablement le code pour générer la liste de tirages aléatoires en utilisant quelque chose appelé une *compréhension de liste*.

Les [compréhensions de liste](https://en.wikipedia.org/wiki/List_comprehension) sont un outil Python élégant pour créer des listes.

Considérez l'exemple suivant, où la compréhension de liste se trouve du
côté droit de la deuxième ligne

```{code-cell} python3
animals = ['dog', 'cat', 'bird']
plurals = [animal + 's' for animal in animals]
plurals
```

Voici un autre exemple

```{code-cell} python3
range(8)
```

```{code-cell} python3
doubles = [2 * x for x in range(8)]
doubles
```

## Comparaisons et opérateurs logiques

### Comparaisons

```{index} single: Python; Comparison
```

De nombreux types d'expressions différents s'évaluent à l'une des valeurs booléennes (c'est-à-dire `True` ou `False`).

Un type courant est celui des comparaisons, telles que

```{code-cell} python3
x, y = 1, 2
x < y
```

```{code-cell} python3
x > y
```

L'une des belles fonctionnalités de Python est que nous pouvons *chaîner* les inégalités

```{code-cell} python3
1 < 2 < 3
```

```{code-cell} python3
1 <= 2 <= 3
```

Comme nous l'avons vu précédemment, pour tester l'égalité nous utilisons `==`

```{code-cell} python3
x = 1    # Affectation
x == 2   # Comparaison
```

Pour « différent de » utilisez `!=`

```{code-cell} python3
1 != 2
```

Notez que lors du test de conditions, nous pouvons utiliser **n'importe quelle** expression Python valide

```{code-cell} python3
x = 'yes' if 42 else 'no'
x
```

```{code-cell} python3
x = 'yes' if [] else 'no'
x
```

Que se passe-t-il ici ?

La règle est :

* Les expressions qui s'évaluent à zéro, à des séquences ou conteneurs vides (chaînes, listes, etc.) et `None` sont toutes équivalentes à `False`.
    * par exemple, `[]` et `()` sont équivalents à `False` dans une clause `if`
* Toutes les autres valeurs sont équivalentes à `True`.
    * par exemple, `42` est équivalent à `True` dans une clause `if`

### Combiner des expressions

```{index} single: Python; Logical Expressions
```

Nous pouvons combiner des expressions à l'aide de `and`, `or` et `not`.

Ce sont les connecteurs logiques standard (conjonction, disjonction et négation)

```{code-cell} python3
1 < 2 and 'f' in 'foo'
```

```{code-cell} python3
1 < 2 and 'g' in 'foo'
```

```{code-cell} python3
1 < 2 or 'g' in 'foo'
```

```{code-cell} python3
not True
```

```{code-cell} python3
not not True
```

Rappelez-vous

* `P and Q` est `True` si les deux sont `True`, sinon `False`
* `P or Q` est `False` si les deux sont `False`, sinon `True`

Nous pouvons également utiliser `all()` et `any()` pour tester une séquence d'expressions

```{code-cell} python3
all([1 <= 2 <= 3, 5 <= 6 <= 7])
```
```{code-cell} python3
all([1 <= 2 <= 3, "a" in "letter"])
```
```{code-cell} python3
any([1 <= 2 <= 3, "a" in "letter"])
```

```{note}
* `all()` renvoie `True` lorsque *toutes* les valeurs/expressions booléennes de la séquence sont `True`
* `any()` renvoie `True` lorsque *l'une quelconque* des valeurs/expressions booléennes de la séquence est `True`
```

## Style de codage et documentation

Un style de codage cohérent et l'utilisation de
la documentation peuvent rendre le code plus facile à comprendre et à maintenir.

### Directives de style Python : PEP8

```{index} single: Python; PEP8
```

Vous pouvez trouver la philosophie de programmation de Python en tapant `import this` à l'invite de commande.

Entre autres choses, Python favorise fortement la cohérence dans le style de programmation.

Nous avons tous entendu le dicton sur la cohérence et les petits esprits.

En programmation, comme en mathématiques, le contraire est vrai

* Un article mathématique où les symboles $\cup$ et $\cap$ seraient
  inversés serait très difficile à lire, même si l'auteur vous le disait à la
  première page.

En Python, le style standard est exposé dans [PEP8](https://peps.python.org/pep-0008/).

(Occasionnellement, nous nous écarterons de PEP8 dans ces cours pour mieux correspondre à la notation mathématique)

(Docstrings)=
### Docstrings

```{index} single: Python; Docstrings
```

Python dispose d'un système pour ajouter des commentaires aux modules, classes, fonctions, etc. appelé *docstrings*.

Ce qui est agréable avec les docstrings, c'est qu'elles sont disponibles au moment de l'exécution.

Essayez d'exécuter ceci

```{code-cell} python3
def f(x):
    """
    This function squares its argument
    """
    return x**2
```

Après avoir exécuté ce code, la docstring est disponible

```{code-cell} ipython
f?
```

```{code-block} ipython
:class: no-execute

Type:       function
String Form:<function f at 0x2223320>
File:       /home/john/temp/temp.py
Definition: f(x)
Docstring:  This function squares its argument
```

```{code-cell} ipython
f??
```

```{code-block} ipython
:class: no-execute

Type:       function
String Form:<function f at 0x2223320>
File:       /home/john/temp/temp.py
Definition: f(x)
Source:
def f(x):
    """
    This function squares its argument
    """
    return x**2
```

Avec un point d'interrogation, nous affichons la docstring, et avec deux, nous obtenons aussi le code source.

Vous pouvez trouver les conventions pour les docstrings dans [PEP257](https://peps.python.org/pep-0257/).

## Exercices

Résolvez les exercices suivants.

(Pour certains, la fonction intégrée `sum()` s'avère pratique).

```{exercise-start}
:label: pyess_ex1
```
Partie 1 : Étant donné deux listes ou tuples numériques `x_vals` et `y_vals` de longueur égale, calculez
leur produit scalaire en utilisant `zip()`.

Partie 2 : En une seule ligne, comptez le nombre de nombres pairs dans 0,...,99.

Partie 3 : Étant donné `pairs = ((2, 5), (4, 2), (9, 8), (12, 10))`, comptez le nombre de paires `(a, b)`
telles que `a` et `b` soient tous deux pairs.

```{hint}
:class: dropdown

`x % 2` renvoie 0 si `x` est pair, 1 sinon.

```

```{exercise-end}
```


```{solution-start} pyess_ex1
:class: dropdown
```

**Solution de la partie 1 :**

Voici une solution possible

```{code-cell} python3
x_vals = [1, 2, 3]
y_vals = [1, 1, 1]
sum([x * y for x, y in zip(x_vals, y_vals)])
```

Cela fonctionne aussi

```{code-cell} python3
sum(x * y for x, y in zip(x_vals, y_vals))
```

**Solution de la partie 2 :**

Une solution est

```{code-cell} python3
sum([x % 2 == 0 for x in range(100)])
```

Cela fonctionne aussi :

```{code-cell} python3
sum(x % 2 == 0 for x in range(100))
```

Quelques alternatives moins naturelles qui aident néanmoins à illustrer la
flexibilité des compréhensions de liste sont

```{code-cell} python3
len([x for x in range(100) if x % 2 == 0])
```

et

```{code-cell} python3
sum([1 for x in range(100) if x % 2 == 0])
```

**Solution de la partie 3 :**

Voici une possibilité

```{code-cell} python3
pairs = ((2, 5), (4, 2), (9, 8), (12, 10))
sum([x % 2 == 0 and y % 2 == 0 for x, y in pairs])
```

```{solution-end}
```

```{exercise-start}
:label: pyess_ex2
```

Considérez le polynôme

```{math}
:label: polynom0

p(x)
= a_0 + a_1 x + a_2 x^2 + \cdots a_n x^n
= \sum_{i=0}^n a_i x^i
```

Écrivez une fonction `p` telle que `p(x, coeff)` calcule la valeur dans {eq}`polynom0` étant donné un point `x` et une liste de coefficients `coeff` ($a_1, a_2, \cdots a_n$).

Essayez d'utiliser `enumerate()` dans votre boucle.

```{exercise-end}
```

```{solution-start} pyess_ex2
:class: dropdown
```
Voici une solution :

```{code-cell} python3
def p(x, coeff):
    return sum(a * x**i for i, a in enumerate(coeff))
```

```{code-cell} python3
p(1, (2, 4))
```

```{solution-end}
```


```{exercise-start}
:label: pyess_ex3
```

Écrivez une fonction qui prend une chaîne comme argument et renvoie le nombre de lettres majuscules dans la chaîne.

```{hint}
:class: dropdown

`'foo'.upper()` renvoie `'FOO'`.

```

```{exercise-end}
```

```{solution-start} pyess_ex3
:class: dropdown
```

Voici une solution :

```{code-cell} python3
def f(string):
    count = 0
    for letter in string:
        if letter == letter.upper() and letter.isalpha():
            count += 1
    return count

f('The Rain in Spain')
```

Une alternative, solution plus pythonique :

```{code-cell} python3
def count_uppercase_chars(s):
    return sum([c.isupper() for c in s])

count_uppercase_chars('The Rain in Spain')
```

```{solution-end}
```



```{exercise}
:label: pyess_ex4

Écrivez une fonction qui prend deux séquences `seq_a` et `seq_b` comme arguments et
renvoie `True` si chaque élément de `seq_a` est également un élément de `seq_b`, sinon
`False`.

* Par « séquence », nous entendons une liste, un tuple ou une chaîne.
* Faites l'exercice sans utiliser les [ensembles](https://docs.python.org/3/tutorial/datastructures.html#sets) ni les méthodes d'ensemble.
```

```{solution-start} pyess_ex4
:class: dropdown
```

Voici une solution :

```{code-cell} python3
def f(seq_a, seq_b):
    for a in seq_a:
        if a not in seq_b:
            return False
    return True

# == test == #
print(f("ab", "cadb"))
print(f("ab", "cjdb"))
print(f([1, 2], [1, 2, 3]))
print(f([1, 2, 3], [1, 2]))
```

Une alternative, solution plus pythonique utilisant `all()` :

```{code-cell} python3
def f(seq_a, seq_b):
  return all([i in seq_b for i in seq_a])

# == test == #
print(f("ab", "cadb"))
print(f("ab", "cjdb"))
print(f([1, 2], [1, 2, 3]))
print(f([1, 2, 3], [1, 2]))
```

Bien sûr, si nous utilisons le type de données `sets`, la solution est plus facile

```{code-cell} python3
def f(seq_a, seq_b):
    return set(seq_a).issubset(set(seq_b))
```

```{solution-end}
```


```{exercise}
:label: pyess_ex5

Lorsque nous couvrirons les bibliothèques numériques, nous verrons qu'elles incluent de nombreuses
alternatives pour l'interpolation et l'approximation de fonctions.

Néanmoins, écrivons notre propre routine d'approximation de fonction comme exercice.

En particulier, sans utiliser aucun import, écrivez une fonction `linapprox` qui prend comme arguments

* Une fonction `f` mappant un certain intervalle $[a, b]$ dans $\mathbb R$.
* Deux scalaires `a` et `b` fournissant les limites de cet intervalle.
* Un entier `n` déterminant le nombre de points de grille.
* Un nombre `x` satisfaisant `a <= x <= b`.

et renvoie l'[interpolation linéaire par morceaux](https://en.wikipedia.org/wiki/Linear_interpolation) de `f` en `x`, basée sur `n` points de grille régulièrement espacés `a = point[0] < point[1] < ... < point[n-1] = b`.

Visez la clarté, pas l'efficacité.
```

```{solution-start} pyess_ex5
:class: dropdown
```
Voici une solution :

```{code-cell} python3
def linapprox(f, a, b, n, x):
    """
    Evaluates the piecewise linear interpolant of f at x on the interval
    [a, b], with n evenly spaced grid points.

    Parameters
    ==========
        f : function
            The function to approximate

        x, a, b : scalars (floats or integers)
            Evaluation point and endpoints, with a <= x <= b

        n : integer
            Number of grid points

    Returns
    =======
        A float. The interpolant evaluated at x

    """
    length_of_interval = b - a
    num_subintervals = n - 1
    step = length_of_interval / num_subintervals

    # === trouver le premier point de grille plus grand que x === #
    point = a
    while point <= x:
        point += step

    # === x doit se situer entre les points de grille (point - step) et point === #
    u, v = point - step, point

    return f(u) + (x - u) * (f(v) - f(u)) / (v - u)
```

```{solution-end}
```


```{exercise-start}
:label: pyess_ex6
```

En utilisant la syntaxe de compréhension de liste, nous pouvons simplifier la boucle dans le
code suivant.

```{code-cell} python3
import numpy as np

rng = np.random.default_rng()
n = 100
ϵ_values = []
for i in range(n):
    e = rng.standard_normal()
    ϵ_values.append(e)
```

```{exercise-end}
```

```{solution-start} pyess_ex6
:class: dropdown
```

Voici une solution.

```{code-cell} python3
rng = np.random.default_rng()
n = 100
ϵ_values = [rng.standard_normal() for i in range(n)]
```

```{solution-end}
```