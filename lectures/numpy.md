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
  title: NumPy
  headings:
    Overview: Vue d'ensemble
    NumPy Arrays: Tableaux NumPy
    NumPy Arrays::Basics: Notions de base
    NumPy Arrays::Shape and Dimension: Forme et dimension
    NumPy Arrays::Creating Arrays: Création de tableaux
    NumPy Arrays::Array Indexing: Indexation de tableaux
    NumPy Arrays::Array Methods: Méthodes des tableaux
    Arithmetic Operations: Opérations arithmétiques
    Matrix Multiplication: Multiplication matricielle
    Broadcasting: Broadcasting
    Mutability and Copying Arrays: Mutabilité et copie de tableaux
    Mutability and Copying Arrays::Mutability: Mutabilité
    Mutability and Copying Arrays::Making Copies: Faire des copies
    Additional Features: Fonctionnalités supplémentaires
    Additional Features::Universal Functions: Fonctions universelles
    Additional Features::Comparisons: Comparaisons
    Additional Features::Sub-packages: Sous-packages
    Additional Features::Implicit Multithreading: Multithreading implicite
    Exercises: Exercices
---

(np)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# {index}`NumPy <single: NumPy>`

```{index} single: Python; NumPy
```

```{epigraph}
« Soyons clairs : le travail de la science n'a rien à voir avec le consensus. Le consensus est l'affaire de la politique. La science, au contraire, ne requiert qu'un seul chercheur qui se trouve avoir raison, ce qui signifie qu'il ou elle dispose de résultats vérifiables par référence au monde réel. En science, le consensus n'a aucune importance. Ce qui importe, ce sont des résultats reproductibles. » -- Michael Crichton
```

En plus de ce qui est inclus dans Anaconda, ce cours nécessitera les bibliothèques suivantes :

```{code-cell} ipython3
:tags: [hide-output]

!pip install quantecon
```

## Vue d'ensemble

[NumPy](https://en.wikipedia.org/wiki/NumPy) est une bibliothèque de premier ordre pour la programmation numérique

* Largement utilisée dans le monde universitaire, la finance et l'industrie.
* Mature, rapide, stable et en développement continu.

Nous avons déjà vu du code impliquant NumPy dans les cours précédents.

Dans ce cours, nous allons entamer une discussion plus systématique de 

1. les tableaux NumPy et
1. les opérations fondamentales de traitement de tableaux fournies par NumPy.


(Pour une référence alternative, consultez [la documentation officielle de NumPy](https://numpy.org/doc/stable/reference/).)

Nous utiliserons les importations suivantes.

```{code-cell} python3
import numpy as np
import random
import quantecon as qe
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d.axes3d import Axes3D
from matplotlib import cm
```



(numpy_array)=
## Tableaux NumPy

```{index} single: NumPy; Arrays
```

Le problème essentiel que NumPy résout est le traitement rapide de tableaux.

La structure la plus importante que NumPy définit est un type de données tableau, formellement
appelé [numpy.ndarray](https://numpy.org/doc/stable/reference/arrays.ndarray.html).

Les tableaux NumPy alimentent une très grande proportion de l'écosystème scientifique de Python.

### Notions de base

Pour créer un tableau NumPy ne contenant que des zéros, nous utilisons [np.zeros](https://numpy.org/doc/stable/reference/generated/numpy.zeros.html#numpy.zeros)

```{code-cell} python3
a = np.zeros(3)
a
```

```{code-cell} python3
type(a)
```

Les tableaux NumPy ressemblent quelque peu aux listes natives de Python, sauf que

* Les données *doivent être homogènes* (tous les éléments du même type).
* Ces types doivent être l'un des [types de données](https://numpy.org/doc/stable/reference/arrays.dtypes.html) (`dtypes`) fournis par NumPy.

Les plus importants de ces dtypes sont :

* float64 : nombre à virgule flottante sur 64 bits
* int64 : entier sur 64 bits
* bool : True ou False sur 8 bits

Il existe également des dtypes pour représenter les nombres complexes, les entiers non signés, etc.

Sur les machines modernes, le dtype par défaut pour les tableaux est `float64`

```{code-cell} python3
a = np.zeros(3)
type(a[0])
```

Si nous voulons utiliser des entiers, nous pouvons le spécifier comme suit :

```{code-cell} python3
a = np.zeros(3, dtype=int)
type(a[0])
```

(numpy_shape_dim)=
### Forme et dimension

```{index} single: NumPy; Arrays (Shape and Dimension)
```

Considérons l'affectation suivante

```{code-cell} python3
z = np.zeros(10)
```

Ici `z` est un tableau **plat** --- ni vecteur ligne ni vecteur colonne.

```{code-cell} python3
z.shape
```

Ici, le tuple de forme n'a qu'un seul élément, qui est la longueur du tableau
(les tuples à un seul élément se terminent par une virgule).

Pour lui donner une dimension supplémentaire, nous pouvons modifier l'attribut `shape`

```{code-cell} python3
z.shape = (10, 1)   # Convertit le tableau plat en vecteur colonne (bidimensionnel)
z
```

```{code-cell} python3
z = np.zeros(4)     # Tableau plat
z.shape = (2, 2)    # Tableau bidimensionnel
z
```

Dans le dernier cas, pour créer le tableau 2x2, nous pourrions aussi passer un tuple à la fonction `zeros()`, comme
dans `z = np.zeros((2, 2))`.



(creating_arrays)=
### Création de tableaux

```{index} single: NumPy; Arrays (Creating)
```

Comme nous l'avons vu, la fonction `np.zeros` crée un tableau de zéros.

Vous pouvez probablement deviner ce que crée `np.ones`.

Une fonction apparentée est `np.empty`, qui crée des tableaux en mémoire qui peuvent ensuite être remplis de données

```{code-cell} python3
z = np.empty(3)
z
```

Les nombres que vous voyez ici sont des valeurs parasites.

(Python alloue 3 morceaux contigus de mémoire de 64 bits, et le contenu existant de ces emplacements mémoire est interprété comme des valeurs `float64`)

Pour établir une grille de nombres régulièrement espacés, utilisez `np.linspace`

```{code-cell} python3
z = np.linspace(2, 4, 5)  # De 2 à 4, avec 5 éléments
```

Pour créer une matrice identité, utilisez soit `np.identity`, soit `np.eye`

```{code-cell} python3
z = np.identity(2)
z
```

De plus, les tableaux NumPy peuvent être créés à partir de listes Python, de tuples, etc. en utilisant `np.array`

```{code-cell} python3
z = np.array([10, 20])                 # ndarray à partir d'une liste Python
z
```

```{code-cell} python3
type(z)
```

```{code-cell} python3
z = np.array((10, 20), dtype=float)    # Ici 'float' est équivalent à 'np.float64'
z
```

```{code-cell} python3
z = np.array([[1, 2], [3, 4]])         # Tableau 2D à partir d'une liste de listes
z
```

Voir aussi `np.asarray`, qui remplit une fonction similaire, mais ne fait pas
de copie distincte des données déjà présentes dans un tableau NumPy.

Pour lire les données d'un tableau à partir d'un fichier texte contenant des données numériques, utilisez `np.loadtxt` --- voir [la documentation](https://numpy.org/doc/stable/reference/routines.io.html) pour plus de détails.



### Indexation de tableaux

```{index} single: NumPy; Arrays (Indexing)
```

Pour un tableau plat, l'indexation est la même que pour les séquences Python :

```{code-cell} python3
z = np.linspace(1, 2, 5)
z
```

```{code-cell} python3
z[0]
```

```{code-cell} python3
z[0:2]  # Deux éléments, en commençant par l'élément 0
```

```{code-cell} python3
z[-1]
```

Pour les tableaux 2D, la syntaxe d'indexation est la suivante :

```{code-cell} python3
z = np.array([[1, 2], [3, 4]])
z
```

```{code-cell} python3
z[0, 0]
```

```{code-cell} python3
z[0, 1]
```

Et ainsi de suite.

Les colonnes et les lignes peuvent être extraites comme suit

```{code-cell} python3
z[0, :]
```

```{code-cell} python3
z[:, 1]
```

Les tableaux NumPy d'entiers peuvent aussi être utilisés pour extraire des éléments

```{code-cell} python3
z = np.linspace(2, 4, 5)
z
```

```{code-cell} python3
indices = np.array((0, 2, 3))
z[indices]
```

Enfin, un tableau de `dtype bool` peut être utilisé pour extraire des éléments

```{code-cell} python3
z
```

```{code-cell} python3
d = np.array([0, 1, 1, 0, 0], dtype=bool)
d
```

```{code-cell} python3
z[d]
```

Nous verrons ci-dessous pourquoi cela est utile.

Une remarque en passant : tous les éléments d'un tableau peuvent être fixés à un même nombre en utilisant la notation de tranche

```{code-cell} python3
z = np.empty(3)
z
```

```{code-cell} python3
z[:] = 42
z
```

### Méthodes des tableaux

```{index} single: NumPy; Arrays (Methods)
```

Les tableaux disposent de méthodes utiles, toutes soigneusement optimisées

```{code-cell} python3
a = np.array((4, 3, 2, 1))
a
```

```{code-cell} python3
a.sort()              # Trie a sur place
a
```

```{code-cell} python3
a.sum()               # Somme
```

```{code-cell} python3
a.mean()              # Moyenne
```

```{code-cell} python3
a.max()               # Max
```

```{code-cell} python3
a.argmax()            # Renvoie l'indice de l'élément maximal
```

```{code-cell} python3
a.cumsum()            # Somme cumulée des éléments de a
```

```{code-cell} python3
a.cumprod()           # Produit cumulé des éléments de a
```

```{code-cell} python3
a.var()               # Variance
```

```{code-cell} python3
a.std()               # Écart-type
```

```{code-cell} python3
a.shape = (2, 2)
a.T                   # Équivalent à a.transpose()
```

Une autre méthode qui vaut la peine d'être connue est `searchsorted()`.

Si `z` est un tableau non décroissant, alors `z.searchsorted(a)` renvoie l'indice du
premier élément de `z` qui est `>= a`

```{code-cell} python3
z = np.linspace(2, 4, 5)
z
```

```{code-cell} python3
z.searchsorted(2.2)
```


## Opérations arithmétiques

```{index} single: NumPy; Arithmetic Operations
```

Les opérateurs `+`, `-`, `*`, `/` et `**` agissent tous *élément par élément* sur les tableaux

```{code-cell} python3
a = np.array([1, 2, 3, 4])
b = np.array([5, 6, 7, 8])
a + b
```

```{code-cell} python3
a * b
```

Nous pouvons ajouter un scalaire à chaque élément comme suit

```{code-cell} python3
a + 10
```

La multiplication scalaire est similaire

```{code-cell} python3
a * 10
```

Les tableaux bidimensionnels suivent les mêmes règles générales

```{code-cell} python3
A = np.ones((2, 2))
B = np.ones((2, 2))
A + B
```

```{code-cell} python3
A + 10
```

```{code-cell} python3
A * B
```

(numpy_matrix_multiplication)=
En particulier, `A * B` n'est *pas* le produit matriciel, c'est un produit élément par élément.


## Multiplication matricielle

```{index} single: NumPy; Matrix Multiplication
```

```{index} single: NumPy; Matrix Multiplication
```

Nous utilisons le symbole `@` pour la multiplication matricielle, comme suit :

```{code-cell} python3
A = np.ones((2, 2))
B = np.ones((2, 2))
A @ B
```

La syntaxe fonctionne avec des tableaux plats --- NumPy devine intelligemment ce que vous
voulez :

```{code-cell} python3
A @ (0, 1)
```

Comme nous effectuons une post-multiplication, le tuple est traité comme un vecteur colonne.



(broadcasting)=
## Broadcasting

```{index} single: NumPy; Broadcasting
```

(Cette section prolonge une excellente discussion du broadcasting fournie par [Jake VanderPlas](https://jakevdp.github.io/PythonDataScienceHandbook/02.05-computation-on-arrays-broadcasting.html).)

```{note}
Le broadcasting est un aspect très important de NumPy. En même temps, le broadcasting avancé est relativement complexe et certains des détails ci-dessous peuvent être survolés en première lecture.
```

Dans les opérations élément par élément, les tableaux peuvent ne pas avoir la même forme.
 
Lorsque cela se produit, NumPy étendra automatiquement les tableaux à la même forme chaque fois que possible.

Cette fonctionnalité utile (mais parfois déroutante) de NumPy s'appelle **broadcasting**.

L'intérêt du broadcasting est que

* les boucles `for` peuvent être évitées, ce qui aide le code numérique à s'exécuter rapidement et
* le broadcasting peut nous permettre de mettre en œuvre des opérations sur des tableaux sans réellement créer certaines dimensions de ces tableaux en mémoire, ce qui peut être important lorsque les tableaux sont grands.

Par exemple, supposons que `a` soit un tableau $3 \times 3$ (`a -> (3, 3)`), tandis que `b` est un tableau plat de trois éléments (`b -> (3,)`).

Lors de leur addition, NumPy étendra automatiquement `b -> (3,)` en `b -> (3, 3)`.

L'addition élément par élément produira un tableau $3 \times 3$

```{code-cell} python3

a = np.array(
        [[1, 2, 3], 
         [4, 5, 6], 
         [7, 8, 9]])
b = np.array([3, 6, 9])

a + b
```

Voici une représentation visuelle de cette opération de broadcasting :

```{code-cell} python3
---
tags: [hide-input]
---
# Adapté et modifié à partir du code du livre écrit par Jake VanderPlas (voir https://jakevdp.github.io/PythonDataScienceHandbook/06.00-figure-code.html#Broadcasting)
# Provient à l'origine d'astroML : voir https://www.astroml.org/book_figures/appendix/fig_broadcast_visual.html


def draw_cube(ax, xy, size, depth=0.4,
              edges=None, label=None, label_kwargs=None, **kwargs):
    """draw and label a cube.  edges is a list of numbers between
    1 and 12, specifying which of the 12 cube edges to draw"""
    if edges is None:
        edges = range(1, 13)

    x, y = xy

    if 1 in edges:
        ax.plot([x, x + size],
                [y + size, y + size], **kwargs)
    if 2 in edges:
        ax.plot([x + size, x + size],
                [y, y + size], **kwargs)
    if 3 in edges:
        ax.plot([x, x + size],
                [y, y], **kwargs)
    if 4 in edges:
        ax.plot([x, x],
                [y, y + size], **kwargs)

    if 5 in edges:
        ax.plot([x, x + depth],
                [y + size, y + depth + size], **kwargs)
    if 6 in edges:
        ax.plot([x + size, x + size + depth],
                [y + size, y + depth + size], **kwargs)
    if 7 in edges:
        ax.plot([x + size, x + size + depth],
                [y, y + depth], **kwargs)
    if 8 in edges:
        ax.plot([x, x + depth],
                [y, y + depth], **kwargs)

    if 9 in edges:
        ax.plot([x + depth, x + depth + size],
                [y + depth + size, y + depth + size], **kwargs)
    if 10 in edges:
        ax.plot([x + depth + size, x + depth + size],
                [y + depth, y + depth + size], **kwargs)
    if 11 in edges:
        ax.plot([x + depth, x + depth + size],
                [y + depth, y + depth], **kwargs)
    if 12 in edges:
        ax.plot([x + depth, x + depth],
                [y + depth, y + depth + size], **kwargs)

    if label:
        if label_kwargs is None:
            label_kwargs = {}
        ax.text(x + 0.5 * size, y + 0.5 * size, label,
                ha='center', va='center', **label_kwargs)

solid = dict(c='black', ls='-', lw=1,
             label_kwargs=dict(color='k'))
dotted = dict(c='black', ls='-', lw=0.5, alpha=0.5,
              label_kwargs=dict(color='gray'))
depth = 0.3

# Dessine une figure et un axe sans bordure
fig = plt.figure(figsize=(5, 1), facecolor='w')
ax = plt.axes([0, 0, 1, 1], xticks=[], yticks=[], frameon=False)

# premier bloc
draw_cube(ax, (1, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '1', **solid)
draw_cube(ax, (2, 7.5), 1, depth, [1, 2, 3, 6, 9], '2', **solid)
draw_cube(ax, (3, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '3', **solid)

draw_cube(ax, (1, 6.5), 1, depth, [2, 3, 4], '4', **solid)
draw_cube(ax, (2, 6.5), 1, depth, [2, 3], '5', **solid)
draw_cube(ax, (3, 6.5), 1, depth, [2, 3, 7, 10], '6', **solid)

draw_cube(ax, (1, 5.5), 1, depth, [2, 3, 4], '7', **solid)
draw_cube(ax, (2, 5.5), 1, depth, [2, 3], '8', **solid)
draw_cube(ax, (3, 5.5), 1, depth, [2, 3, 7, 10], '9', **solid)

# deuxième bloc
draw_cube(ax, (6, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '3', **solid)
draw_cube(ax, (7, 7.5), 1, depth, [1, 2, 3, 6, 9], '6', **solid)
draw_cube(ax, (8, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '9', **solid)

draw_cube(ax, (6, 6.5), 1, depth, range(2, 13), '3', **dotted)
draw_cube(ax, (7, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '6', **dotted)
draw_cube(ax, (8, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '9', **dotted)

draw_cube(ax, (6, 5.5), 1, depth, [2, 3, 4, 7, 8, 10, 11, 12], '3', **dotted)
draw_cube(ax, (7, 5.5), 1, depth, [2, 3, 7, 10, 11], '6', **dotted)
draw_cube(ax, (8, 5.5), 1, depth, [2, 3, 7, 10, 11], '9', **dotted)

# troisième bloc
draw_cube(ax, (12, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '4', **solid)
draw_cube(ax, (13, 7.5), 1, depth, [1, 2, 3, 6, 9], '8', **solid)
draw_cube(ax, (14, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '12', **solid)

draw_cube(ax, (12, 6.5), 1, depth, [2, 3, 4], '7', **solid)
draw_cube(ax, (13, 6.5), 1, depth, [2, 3], '11', **solid)
draw_cube(ax, (14, 6.5), 1, depth, [2, 3, 7, 10], '15', **solid)

draw_cube(ax, (12, 5.5), 1, depth, [2, 3, 4], '10', **solid)
draw_cube(ax, (13, 5.5), 1, depth, [2, 3], '14', **solid)
draw_cube(ax, (14, 5.5), 1, depth, [2, 3, 7, 10], '18', **solid)

ax.text(5, 7.0, '+', size=12, ha='center', va='center')
ax.text(10.5, 7.0, '=', size=12, ha='center', va='center');
```

Qu'en est-il de `b -> (3, 1)` ?

Dans ce cas, NumPy étendra automatiquement `b -> (3, 1)` en `b -> (3, 3)`.

L'addition élément par élément produira alors une matrice $3 \times 3$

```{code-cell} python3
b.shape = (3, 1)

a + b
```

Voici une représentation visuelle de cette opération de broadcasting :

```{code-cell} python3
---
tags: [hide-input]
---

fig = plt.figure(figsize=(5, 1), facecolor='w')
ax = plt.axes([0, 0, 1, 1], xticks=[], yticks=[], frameon=False)

# premier bloc
draw_cube(ax, (1, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '1', **solid)
draw_cube(ax, (2, 7.5), 1, depth, [1, 2, 3, 6, 9], '2', **solid)
draw_cube(ax, (3, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '3', **solid)

draw_cube(ax, (1, 6.5), 1, depth, [2, 3, 4], '4', **solid)
draw_cube(ax, (2, 6.5), 1, depth, [2, 3], '5', **solid)
draw_cube(ax, (3, 6.5), 1, depth, [2, 3, 7, 10], '6', **solid)

draw_cube(ax, (1, 5.5), 1, depth, [2, 3, 4], '7', **solid)
draw_cube(ax, (2, 5.5), 1, depth, [2, 3], '8', **solid)
draw_cube(ax, (3, 5.5), 1, depth, [2, 3, 7, 10], '9', **solid)

# deuxième bloc
draw_cube(ax, (6, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 7, 9, 10], '3', **solid)
draw_cube(ax, (7, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '3', **dotted)
draw_cube(ax, (8, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '3', **dotted)

draw_cube(ax, (6, 6.5), 1, depth, [2, 3, 4, 7, 10], '6', **solid)
draw_cube(ax, (7, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '6', **dotted)
draw_cube(ax, (8, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '6', **dotted)

draw_cube(ax, (6, 5.5), 1, depth, [2, 3, 4, 7, 10], '9', **solid)
draw_cube(ax, (7, 5.5), 1, depth, [2, 3, 7, 10, 11], '9', **dotted)
draw_cube(ax, (8, 5.5), 1, depth, [2, 3, 7, 10, 11], '9', **dotted)

# troisième bloc
draw_cube(ax, (12, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '4', **solid)
draw_cube(ax, (13, 7.5), 1, depth, [1, 2, 3, 6, 9], '5', **solid)
draw_cube(ax, (14, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '6', **solid)

draw_cube(ax, (12, 6.5), 1, depth, [2, 3, 4], '10', **solid)
draw_cube(ax, (13, 6.5), 1, depth, [2, 3], '11', **solid)
draw_cube(ax, (14, 6.5), 1, depth, [2, 3, 7, 10], '12', **solid)

draw_cube(ax, (12, 5.5), 1, depth, [2, 3, 4], '16', **solid)
draw_cube(ax, (13, 5.5), 1, depth, [2, 3], '17', **solid)
draw_cube(ax, (14, 5.5), 1, depth, [2, 3, 7, 10], '18', **solid)

ax.text(5, 7.0, '+', size=12, ha='center', va='center')
ax.text(10.5, 7.0, '=', size=12, ha='center', va='center');


```

Dans certains cas, les deux opérandes seront étendus.

Lorsque nous avons `a -> (3,)` et `b -> (3, 1)`, `a` sera étendu en `a -> (3, 3)`, et `b` sera étendu en `b -> (3, 3)`.

Dans ce cas, l'addition élément par élément produira une matrice $3 \times 3$

```{code-cell} python3
a = np.array([3, 6, 9])
b = np.array([2, 3, 4])
b.shape = (3, 1)

a + b
```

Voici une représentation visuelle de cette opération de broadcasting :

```{code-cell} python3
---
tags: [hide-input]
---

# Dessine une figure et un axe sans bordure
fig = plt.figure(figsize=(5, 1), facecolor='w')
ax = plt.axes([0, 0, 1, 1], xticks=[], yticks=[], frameon=False)

# premier bloc
draw_cube(ax, (1, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '3', **solid)
draw_cube(ax, (2, 7.5), 1, depth, [1, 2, 3, 6, 9], '6', **solid)
draw_cube(ax, (3, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '9', **solid)

draw_cube(ax, (1, 6.5), 1, depth, range(2, 13), '3', **dotted)
draw_cube(ax, (2, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '6', **dotted)
draw_cube(ax, (3, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '9', **dotted)

draw_cube(ax, (1, 5.5), 1, depth, [2, 3, 4, 7, 8, 10, 11, 12], '3', **dotted)
draw_cube(ax, (2, 5.5), 1, depth, [2, 3, 7, 10, 11], '6', **dotted)
draw_cube(ax, (3, 5.5), 1, depth, [2, 3, 7, 10, 11], '9', **dotted)

# deuxième bloc
draw_cube(ax, (6, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 7, 9, 10], '2', **solid)
draw_cube(ax, (7, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '2', **dotted)
draw_cube(ax, (8, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '2', **dotted)

draw_cube(ax, (6, 6.5), 1, depth, [2, 3, 4, 7, 10], '3', **solid)
draw_cube(ax, (7, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '3', **dotted)
draw_cube(ax, (8, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '3', **dotted)

draw_cube(ax, (6, 5.5), 1, depth, [2, 3, 4, 7, 10], '4', **solid)
draw_cube(ax, (7, 5.5), 1, depth, [2, 3, 7, 10, 11], '4', **dotted)
draw_cube(ax, (8, 5.5), 1, depth, [2, 3, 7, 10, 11], '4', **dotted)

# troisième bloc
draw_cube(ax, (12, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '5', **solid)
draw_cube(ax, (13, 7.5), 1, depth, [1, 2, 3, 6, 9], '8', **solid)
draw_cube(ax, (14, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '11', **solid)

draw_cube(ax, (12, 6.5), 1, depth, [2, 3, 4], '6', **solid)
draw_cube(ax, (13, 6.5), 1, depth, [2, 3], '9', **solid)
draw_cube(ax, (14, 6.5), 1, depth, [2, 3, 7, 10], '12', **solid)

draw_cube(ax, (12, 5.5), 1, depth, [2, 3, 4], '7', **solid)
draw_cube(ax, (13, 5.5), 1, depth, [2, 3], '10', **solid)
draw_cube(ax, (14, 5.5), 1, depth, [2, 3, 7, 10], '13', **solid)

ax.text(5, 7.0, '+', size=12, ha='center', va='center')
ax.text(10.5, 7.0, '=', size=12, ha='center', va='center');
```

Bien que le broadcasting soit très utile, il peut parfois sembler déroutant.

Par exemple, essayons d'additionner `a -> (3, 2)` et `b -> (3,)`.

```{code-cell} python3
---
tags: [raises-exception]
---
a = np.array(
      [[1, 2],
       [4, 5],
       [7, 8]])
b = np.array([3, 6, 9])

a + b
```

La `ValueError` nous indique que les opérandes n'ont pas pu être broadcastées ensemble.


Voici une représentation visuelle pour montrer pourquoi ce broadcasting ne peut pas être exécuté :

```{code-cell} python3
---
tags: [hide-input]
---
# Dessine une figure et un axe sans bordure
fig = plt.figure(figsize=(3, 1.3), facecolor='w')
ax = plt.axes([0, 0, 1, 1], xticks=[], yticks=[], frameon=False)

# premier bloc
draw_cube(ax, (1, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '1', **solid)
draw_cube(ax, (2, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '2', **solid)

draw_cube(ax, (1, 6.5), 1, depth, [2, 3, 4], '4', **solid)
draw_cube(ax, (2, 6.5), 1, depth, [2, 3, 7, 10], '5', **solid)

draw_cube(ax, (1, 5.5), 1, depth, [2, 3, 4], '7', **solid)
draw_cube(ax, (2, 5.5), 1, depth, [2, 3, 7, 10], '8', **solid)

# deuxième bloc
draw_cube(ax, (6, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '3', **solid)
draw_cube(ax, (7, 7.5), 1, depth, [1, 2, 3, 6, 9], '6', **solid)
draw_cube(ax, (8, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '9', **solid)

draw_cube(ax, (6, 6.5), 1, depth, range(2, 13), '3', **dotted)
draw_cube(ax, (7, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '6', **dotted)
draw_cube(ax, (8, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '9', **dotted)

draw_cube(ax, (6, 5.5), 1, depth, [2, 3, 4, 7, 8, 10, 11, 12], '3', **dotted)
draw_cube(ax, (7, 5.5), 1, depth, [2, 3, 7, 10, 11], '6', **dotted)
draw_cube(ax, (8, 5.5), 1, depth, [2, 3, 7, 10, 11], '9', **dotted)


ax.text(4.5, 7.0, '+', size=12, ha='center', va='center')
ax.text(10, 7.0, '=', size=12, ha='center', va='center')
ax.text(11, 7.0, '?', size=16, ha='center', va='center');
```

Nous pouvons voir que NumPy ne peut pas étendre les tableaux à la même taille.

C'est parce que, lorsque `b` est étendu de `b -> (3,)` à `b -> (3, 3)`, NumPy ne peut pas faire correspondre `b` avec `a -> (3, 2)`.

Les choses deviennent encore plus délicates lorsque nous passons à des dimensions supérieures.

Pour nous aider, nous pouvons utiliser la liste de règles suivante :

* *Étape 1 :* Lorsque les dimensions de deux tableaux ne correspondent pas, NumPy étend celui qui a le moins de dimensions en ajoutant une ou plusieurs dimensions à gauche des dimensions existantes.
    - Par exemple, si `a -> (3, 3)` et `b -> (3,)`, alors le broadcasting ajoutera une dimension à gauche de sorte que `b -> (1, 3)` ;
    - Si `a -> (2, 2, 2)` et `b -> (2, 2)`, alors le broadcasting ajoutera une dimension à gauche de sorte que `b -> (1, 2, 2)` ;
    - Si `a -> (3, 2, 2)` et `b -> (2,)`, alors le broadcasting ajoutera deux dimensions à gauche de sorte que `b -> (1, 1, 2)` (on peut aussi voir ce processus comme un passage par l'*Étape 1* deux fois).


* *Étape 2 :* Lorsque les deux tableaux ont la même dimension mais des formes différentes, NumPy tentera d'étendre les dimensions où l'indice de forme est 1.
    - Par exemple, si `a -> (1, 3)` et `b -> (3, 1)`, alors le broadcasting étendra les dimensions de forme 1 dans `a` et `b` de sorte que `a -> (3, 3)` et `b -> (3, 3)` ;
    - Si `a -> (2, 2, 2)` et `b -> (1, 2, 2)`, alors le broadcasting étendra la première dimension de `b` de sorte que `b -> (2, 2, 2)` ;
    - Si `a -> (3, 2, 2)` et `b -> (1, 1, 2)`, alors le broadcasting étendra `b` sur toutes les dimensions de forme 1 de sorte que `b -> (3, 2, 2)`.

* *Étape 3 :* Après les étapes 1 et 2, si les deux tableaux ne correspondent toujours pas, une `ValueError` sera levée. Par exemple, supposons `a -> (2, 2, 3)` et `b -> (2, 2)`
    - Par l'*Étape 1*, `b` sera étendu en `b -> (1, 2, 2)` ;
    - Par l'*Étape 2*, `b` sera étendu en `b -> (2, 2, 2)` ;
    - Nous pouvons voir qu'ils ne correspondent pas l'un à l'autre après les deux premières étapes. Ainsi, une `ValueError` sera levée



## Mutabilité et copie de tableaux

Les tableaux NumPy sont des types de données mutables, comme les listes Python.

En d'autres termes, leur contenu peut être modifié (muté) en mémoire après l'initialisation.

C'est pratique mais, combiné au modèle de nommage et de référence de Python,
cela peut conduire à des erreurs chez les débutants en NumPy.

Dans cette section, nous passons en revue quelques points clés.


### Mutabilité

Nous avons déjà vu des exemples de mutabilité ci-dessus.

Voici un autre exemple de mutation d'un tableau NumPy

```{code-cell} python3
a = np.array([42, 44])
a
```

```{code-cell} python3
a[-1] = 0  # Change le dernier élément en 0
a
```

La mutabilité conduit au comportement suivant (qui peut être choquant pour les programmeurs MATLAB...)

```{code-cell} python3
rng = np.random.default_rng()
a = rng.standard_normal(3)
a
```

```{code-cell} python3
b = a
b[0] = 0.0
a
```

Ce qui s'est passé, c'est que nous avons modifié `a` en modifiant `b`.

Le nom `b` est lié à `a` et devient simplement une autre référence au
tableau (le modèle d'affectation de Python est décrit plus en détail {doc}`plus loin dans le cours <python_advanced_features>`).

Il a donc des droits égaux pour apporter des modifications à ce tableau.

C'est en fait le comportement par défaut le plus sensé !

Cela signifie que nous ne faisons circuler que des pointeurs vers les données, plutôt que de faire des copies.

Faire des copies est coûteux à la fois en termes de vitesse et de mémoire.

### Faire des copies

Il est bien sûr possible de faire de `b` une copie indépendante de `a` lorsque cela est nécessaire.

Cela peut être fait en utilisant `np.copy`

```{code-cell} python3
a = rng.standard_normal(3)
a
```

```{code-cell} python3
b = np.copy(a)
b
```

Maintenant `b` est une copie indépendante (appelée *copie profonde*)

```{code-cell} python3
b[:] = 1
b
```

```{code-cell} python3
a
```

Notez que la modification de `b` n'a pas affecté `a`.




## Fonctionnalités supplémentaires

Examinons quelques autres fonctionnalités utiles de NumPy.


### Fonctions universelles

```{index} single: NumPy; Vectorized Functions
```

NumPy fournit des versions des fonctions standard `log`, `exp`, `sin`, etc. qui agissent *élément par élément* sur les tableaux

```{code-cell} python3
z = np.array([1, 2, 3])
np.sin(z)
```

Cela élimine le besoin de boucles explicites élément par élément telles que

```{code-cell} python3
n = len(z)
y = np.empty(n)
for i in range(n):
    y[i] = np.sin(z[i])
```

Parce qu'elles agissent élément par élément sur les tableaux, ces fonctions sont parfois appelées **fonctions vectorisées**.

Dans le jargon de NumPy, elles sont aussi appelées **ufuncs**, ou **fonctions universelles**.

Comme nous l'avons vu ci-dessus, les opérations arithmétiques usuelles (`+`, `*`, etc.) fonctionnent
également élément par élément, et combiner celles-ci avec les ufuncs donne un très large ensemble de fonctions rapides opérant élément par élément.

```{code-cell} python3
z
```

```{code-cell} python3
(1 / np.sqrt(2 * np.pi)) * np.exp(- 0.5 * z**2)
```

Toutes les fonctions définies par l'utilisateur n'agissent pas élément par élément.

Par exemple, passer un tableau NumPy à la fonction `f` définie ci-dessous provoque une `ValueError`

```{code-cell} python3
def f(x):
    return 1 if x > 0 else 0
```

La fonction NumPy `np.where` fournit une alternative vectorisée :

```{code-cell} python3
x = rng.standard_normal(4)
x
```

```{code-cell} python3
np.where(x > 0, 1, 0)  # Insère 1 si x > 0 est vrai, sinon 0
```

Vous pouvez aussi utiliser `np.vectorize` pour vectoriser une fonction donnée

```{code-cell} python3
f = np.vectorize(f)
f(x)                # Passe le même vecteur x que dans l'exemple précédent
```

Cependant, cette approche n'atteint pas toujours la même vitesse qu'une fonction vectorisée plus soigneusement conçue.

(Plus tard, nous verrons que JAX dispose d'une version puissante de `np.vectorize` qui peut générer, et génère généralement, du code hautement efficace.)


### Comparaisons

```{index} single: NumPy; Comparisons
```

En règle générale, les comparaisons sur les tableaux sont effectuées élément par élément

```{code-cell} python3
z = np.array([2, 3])
y = np.array([2, 3])
z == y
```

```{code-cell} python3
y[0] = 5
z == y
```

```{code-cell} python3
z != y
```

La situation est similaire pour `>`, `<`, `>=` et `<=`.

Nous pouvons aussi effectuer des comparaisons avec des scalaires

```{code-cell} python3
z = np.linspace(0, 10, 5)
z
```

```{code-cell} python3
z > 3
```

Ceci est particulièrement utile pour l'*extraction conditionnelle*

```{code-cell} python3
b = z > 3
b
```

```{code-cell} python3
z[b]
```

Bien sûr, nous pouvons --- et faisons souvent --- effectuer ceci en une seule étape

```{code-cell} python3
z[z > 3]
```

### Sous-packages

NumPy fournit des fonctionnalités supplémentaires liées à la programmation scientifique
via ses sous-packages.

Nous avons déjà vu comment nous pouvons générer des variables aléatoires en utilisant le
[`Generator` aléatoire de NumPy](https://numpy.org/doc/stable/reference/random/generator.html#random-generator).

```{code-cell} python3
z = rng.standard_normal(10000)  # Génère des variables normales centrées réduites
y = rng.binomial(10, 0.5, size=1000)    # 1 000 tirages de Bin(10, 0.5)
y.mean()
```

Un autre sous-package couramment utilisé est np.linalg

```{code-cell} python3
A = np.array([[1, 2], [3, 4]])

np.linalg.det(A)           # Calcule le déterminant
```

```{code-cell} python3
np.linalg.inv(A)           # Calcule l'inverse
```

```{index} single: SciPy
```

```{index} single: Python; SciPy
```

Une grande partie de ces fonctionnalités est aussi disponible dans [SciPy](https://scipy.org/), une collection de modules qui sont construits par-dessus NumPy.

Nous couvrirons les versions SciPy plus en détail {doc}`bientôt <scipy>`.

Pour une liste complète de ce qui est disponible dans NumPy, consultez [cette documentation](https://numpy.org/doc/stable/reference/routines.html).


### Multithreading implicite 

[Précédemment](need_for_speed), nous avons abordé le concept de parallélisation via le multithreading.

NumPy tente d'implémenter le multithreading dans une grande partie de son code compilé.

Examinons un exemple pour voir cela en action.

Le morceau de code suivant calcule les valeurs propres d'un grand nombre de matrices
générées aléatoirement.

Son exécution prend quelques secondes.

```{code-cell} python3
n = 20
m = 1000
for i in range(n):
    X = rng.standard_normal((m, m))
    λ = np.linalg.eigvals(X)
```

Maintenant, examinons la sortie du moniteur système htop sur notre machine pendant que
ce code s'exécute :

```{figure} /_static/lecture_specific/parallelization/htop_parallel_npmat.png
:scale: 80
```

Nous pouvons voir que 4 des 8 processeurs fonctionnent à pleine vitesse.

Cela est dû au fait que la routine `eigvals` de NumPy divise proprement les tâches et
les distribue à différents threads.





## Exercices


```{exercise-start}
:label: np_ex1
```

Considérons l'expression polynomiale

```{math}
:label: np_polynom

p(x) = a_0 + a_1 x + a_2 x^2 + \cdots a_N x^N = \sum_{n=0}^N a_n x^n
```

{ref}`Plus tôt <pyess_ex2>`, vous avez écrit une fonction simple `p(x, coeff)` pour évaluer {eq}`np_polynom` sans tenir compte de l'efficacité.

Écrivez maintenant une nouvelle fonction qui effectue le même travail, mais qui utilise des tableaux NumPy et des opérations sur tableaux pour ses calculs, plutôt qu'une quelconque forme de boucle Python.

(Une telle fonctionnalité est déjà implémentée sous la forme de `np.poly1d`, mais pour les besoins de l'exercice, n'utilisez pas cette classe)

```{hint}
:class: dropdown
Utilisez `np.cumprod()`
```
```{exercise-end}
```

```{solution-start} np_ex1
:class: dropdown
```

Ce code fait le travail

```{code-cell} python3
def p(x, coef):
    X = np.ones_like(coef)
    X[1:] = x
    y = np.cumprod(X)   # y = [1, x, x**2,...]
    return coef @ y
```

Testons-le

```{code-cell} python3
x = 2
coef = np.linspace(2, 4, 3)
print(coef)
print(p(x, coef))
# À des fins de comparaison
q = np.poly1d(np.flip(coef))
print(q(x))
```

```{solution-end}
```


```{exercise-start}
:label: np_ex2
```

Soit `q` un tableau NumPy de longueur `n` avec `q.sum() == 1`.

Supposons que `q` représente une [fonction de masse de probabilité](https://en.wikipedia.org/wiki/Probability_mass_function).

Nous souhaitons générer une variable aléatoire discrète $x$ telle que $\mathbb P\{x = i\} = q_i$.

En d'autres termes, `x` prend des valeurs dans `range(len(q))` et `x = i` avec probabilité `q[i]`.

L'algorithme standard (de transformation inverse) est le suivant :

* Divisez l'intervalle unité $[0, 1]$ en $n$ sous-intervalles $I_0, I_1, \ldots, I_{n-1}$ tels que la longueur de $I_i$ soit $q_i$.
* Tirez une variable aléatoire uniforme $U$ sur $[0, 1]$ et renvoyez le $i$ tel que $U \in I_i$.

La probabilité de tirer $i$ est la longueur de $I_i$, qui est égale à $q_i$.

Nous pouvons implémenter l'algorithme comme suit

```{code-cell} python3
from random import uniform

def sample(q):
    a = 0.0
    U = uniform(0, 1)
    for i in range(len(q)):
        if a < U <= a + q[i]:
            return i
        a = a + q[i]
```

Si vous ne voyez pas comment cela fonctionne, essayez de suivre le flux pour un exemple simple, tel que `q = [0.25, 0.75]`
Il est utile de dessiner les intervalles sur papier.

Votre exercice consiste à l'accélérer en utilisant NumPy, en évitant les boucles explicites

```{hint}
:class: dropdown

Utilisez `np.searchsorted` et `np.cumsum`

```

Si vous le pouvez, implémentez la fonctionnalité sous la forme d'une classe appelée `DiscreteRV`, où

* les données pour une instance de la classe sont le vecteur de probabilités `q`
* la classe dispose d'une méthode `draw()`, qui renvoie un tirage selon l'algorithme décrit ci-dessus

Si vous le pouvez, écrivez la méthode de sorte que `draw(k)` renvoie `k` tirages de `q`.

```{exercise-end}
```

```{solution-start} np_ex2
:class: dropdown
```

Voici notre première tentative de solution :

```{code-cell} python3
from numpy import cumsum

class DiscreteRV:
    """
    Generates an array of draws from a discrete random variable with vector of
    probabilities given by q.
    """

    def __init__(self, q, seed=None):
        """
        The argument q is a NumPy array, or array like, nonnegative and sums
        to 1
        """
        self.q = q
        self.Q = cumsum(q)
        self.rng = np.random.default_rng(seed)

    def draw(self, k=1):
        """
        Returns k draws from q. For each such draw, the value i is returned
        with probability q[i].
        """
        return self.Q.searchsorted(self.rng.uniform(0, 1, size=k))
```

La logique n'est pas évidente, mais si vous prenez votre temps et la lisez lentement,
vous comprendrez.

Il y a cependant un problème ici.

Supposons que `q` soit modifié après la création d'une instance de `discreteRV`, par exemple par

```{code-cell} python3
q = (0.1, 0.9)
d = DiscreteRV(q)
d.q = (0.5, 0.5)
```

Le problème est que `Q` ne change pas en conséquence, et `Q` correspond aux
données utilisées dans la méthode `draw`.

Pour gérer cela, une option consiste à calculer `Q` chaque fois que la méthode draw
est appelée.

Mais c'est inefficace par rapport à un calcul unique de `Q`.

Une meilleure option consiste à utiliser des descripteurs.

Une solution de la [bibliothèque quantecon](https://github.com/QuantEcon/QuantEcon.py/tree/main/quantecon)
utilisant des descripteurs et se comportant comme nous le souhaitons peut être trouvée
[ici](https://github.com/QuantEcon/QuantEcon.py/blob/main/quantecon/discrete_rv.py).

```{solution-end}
```


```{exercise}
:label: np_ex3

Rappelez-vous notre {ref}`discussion précédente <oop_ex1>` sur la fonction de répartition empirique.

Votre tâche consiste à

1. Rendre la méthode `__call__` plus efficace en utilisant NumPy.
1. Ajouter une méthode qui trace la FDE sur $[a, b]$, où $a$ et $b$ sont des paramètres de la méthode.
```

```{solution-start} np_ex3
:class: dropdown
```

Un exemple de solution est donné ci-dessous.

En substance, nous avons simplement repris [ce code](https://github.com/QuantEcon/QuantEcon.py/blob/main/quantecon/ecdf.py)
de QuantEcon et ajouté une méthode de tracé

```{code-cell} python3
"""
Modifies ecdf.py from QuantEcon to add in a plot method

"""

class ECDF:
    """
    One-dimensional empirical distribution function given a vector of
    observations.

    Parameters
    ----------
    observations : array_like
        An array of observations

    Attributes
    ----------
    observations : array_like
        An array of observations

    """

    def __init__(self, observations):
        self.observations = np.asarray(observations)

    def __call__(self, x):
        """
        Evaluates the ecdf at x

        Parameters
        ----------
        x : scalar(float)
            The x at which the ecdf is evaluated

        Returns
        -------
        scalar(float)
            Fraction of the sample less than x

        """
        return np.mean(self.observations <= x)

    def plot(self, ax, a=None, b=None):
        """
        Plot the ecdf on the interval [a, b].

        Parameters
        ----------
        a : scalar(float), optional(default=None)
            Lower endpoint of the plot interval
        b : scalar(float), optional(default=None)
            Upper endpoint of the plot interval

        """

        # === choisit un intervalle raisonnable si [a, b] n'est pas spécifié === #
        if a is None:
            a = self.observations.min() - self.observations.std()
        if b is None:
            b = self.observations.max() + self.observations.std()

        # === génère le tracé === #
        x_vals = np.linspace(a, b, num=100)
        f = np.vectorize(self.__call__)
        ax.plot(x_vals, f(x_vals))
        plt.show()
```

Voici un exemple d'utilisation

```{code-cell} python3
fig, ax = plt.subplots()
rng = np.random.default_rng()
X = rng.standard_normal(1000)
F = ECDF(X)
F.plot(ax)
```

```{solution-end}
```


```{exercise-start}
:label: np_ex4
```

Rappelez-vous que le [broadcasting](broadcasting) dans NumPy peut nous aider à effectuer des opérations élément par élément sur des tableaux ayant un nombre de dimensions différent sans utiliser de boucles `for`.

Dans cet exercice, essayez d'utiliser des boucles `for` pour reproduire le résultat des opérations de broadcasting suivantes.

**Partie 1** : Essayez de reproduire cet exemple simple en utilisant des boucles `for` et comparez vos résultats avec l'opération de broadcasting ci-dessous.

```{code-cell} python3

rng = np.random.default_rng(123)
x = rng.standard_normal((4, 4))
y = rng.standard_normal(4)
A = x / y
```

Voici la sortie

```{code-cell} python3
---
tags: [hide-output]
---
print(A)
```

**Partie 2** : Passez à la reproduction du résultat de l'opération de broadcasting suivante. Parallèlement, comparez les vitesses du broadcasting et de la boucle `for` que vous implémentez.

Pour cette partie de l'exercice, vous pouvez utiliser les fonctions `tic`/`toc` de la bibliothèque `quantecon` pour chronométrer l'exécution. 

Assurons-nous que cette bibliothèque est installée.

```{code-cell} python3
:tags: [hide-output]
!pip install quantecon
```

Nous pouvons maintenant importer le package quantecon.

```{code-cell} python3

rng = np.random.default_rng(123)
x = rng.standard_normal((1000, 100, 100))
y = rng.standard_normal(100)

with qe.Timer("Broadcasting operation"):
    B = x / y
```

Voici la sortie

```{code-cell} python3
---
tags: [hide-output]
---
print(B)
```

```{exercise-end}
```


```{solution-start} np_ex4
:class: dropdown
```

**Solution de la Partie 1**

```{code-cell} python3
rng = np.random.default_rng(123)
x = rng.standard_normal((4, 4))
y = rng.standard_normal(4)

C = np.empty_like(x)
n = len(x)
for i in range(n):
    for j in range(n):
        C[i, j] = x[i, j] / y[j]
```

Comparez les résultats pour vérifier votre réponse

```{code-cell} python3
---
tags: [hide-output]
---
print(C)
```

Vous pouvez aussi utiliser `array_equal()` pour vérifier votre réponse

```{code-cell} python3
print(np.array_equal(A, C))
```


**Solution de la Partie 2**

```{code-cell} python3

rng = np.random.default_rng(123)
x = rng.standard_normal((1000, 100, 100))
y = rng.standard_normal(100)

with qe.Timer("For loop operation"):
    D = np.empty_like(x)
    d1, d2, d3 = x.shape
    for i in range(d1):
        for j in range(d2):
            for k in range(d3):
                D[i, j, k] = x[i, j, k] / y[k]
```

Notez que la boucle `for` prend beaucoup plus de temps que l'opération de broadcasting.

Comparez les résultats pour vérifier votre réponse

```{code-cell} python3
---
tags: [hide-output]
---
print(D)
```

```{code-cell} python3
print(np.array_equal(B, D))
```

```{solution-end}
```