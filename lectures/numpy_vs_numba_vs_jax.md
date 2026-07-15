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
  title: NumPy vs Numba vs JAX
  headings:
    Vectorized operations: Opérations vectorisées
    Vectorized operations::Problem Statement: Énoncé du problème
    Vectorized operations::NumPy vectorization: Vectorisation avec NumPy
    Vectorized operations::Memory Issues: Problèmes de mémoire
    Vectorized operations::A Comparison with Numba: Une comparaison avec Numba
    Vectorized operations::Parallelized Numba: Numba parallélisé
    Vectorized operations::Vectorized code with JAX: Code vectorisé avec JAX
    Vectorized operations::JAX plus vmap: JAX plus vmap
    Vectorized operations::Summary: Résumé
    Sequential operations: Opérations séquentielles
    Sequential operations::Numba Version: Version Numba
    Sequential operations::JAX Version: Version JAX
    Sequential operations::JAX Version::First Attempt: Première tentative
    Sequential operations::JAX Version::Second Attempt: Deuxième tentative
    Sequential operations::Summary: Résumé
    Overall recommendations: Recommandations générales
---

(numpy_numba_jax)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# NumPy vs Numba vs JAX

Dans les cours précédents, nous avons présenté trois bibliothèques fondamentales pour le calcul scientifique et numérique :

* [NumPy](numpy)
* [Numba](numba)
* [JAX](jax_intro)

Laquelle devons-nous utiliser dans une situation donnée ?

Ce cours aborde cette question, au moins partiellement, en présentant quelques cas d'usage.

Avant de commencer, notons que les deux premières forment un couple naturel : NumPy et Numba fonctionnent bien ensemble.

JAX, en revanche, se distingue.

Lorsque nous examinerons chaque approche, nous prendrons en compte non seulement l'efficacité et l'empreinte mémoire, mais aussi la clarté et la facilité d'utilisation.

En plus de ce qui est inclus dans Anaconda, ce cours nécessitera les bibliothèques suivantes :

```{code-cell} ipython3
---
tags: [hide-output]
---
!pip install quantecon jax
```

```{include} _admonition/gpu.md
```

Nous utiliserons les importations suivantes.

```{code-cell} ipython3
from functools import partial

import numpy as np
import numba
import quantecon as qe
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d.axes3d import Axes3D
from matplotlib import cm
import jax
import jax.numpy as jnp
from jax import lax
```

## Opérations vectorisées

Certaines opérations peuvent être parfaitement vectorisées --- toutes les boucles sont facilement éliminées et les opérations numériques se réduisent à des calculs sur des tableaux.

Dans ce cas, quelle approche est la meilleure ?

### Énoncé du problème

Considérons le problème de maximisation d'une fonction $f$ de deux variables $(x,y)$ sur le carré $[-a, a] \times [-a, a]$.

Pour $f$ et $a$, choisissons

$$
f(x,y) = \frac{\cos(x^2 + y^2)}{1 + x^2 + y^2}
\quad \text{et} \quad
a = 3
$$

Voici un tracé de $f$

```{code-cell} ipython3

def f(x, y):
    return np.cos(x**2 + y**2) / (1 + x**2 + y**2)

xgrid = np.linspace(-3, 3, 50)
ygrid = xgrid
x, y = np.meshgrid(xgrid, ygrid)

fig = plt.figure(figsize=(10, 8))
ax = fig.add_subplot(111, projection='3d')
ax.plot_surface(x,
                y,
                f(x, y),
                rstride=2, cstride=2,
                cmap=cm.viridis,
                alpha=0.7,
                linewidth=0.25)
ax.set_zlim(-0.5, 1.0)
ax.set_xlabel('$x$', fontsize=14)
ax.set_ylabel('$y$', fontsize=14)
plt.show()
```

Pour les besoins de cet exercice, nous allons utiliser la force brute pour la maximisation.

1. Évaluer $f$ pour tous les $(x,y)$ d'une grille sur le carré.
1. Renvoyer le maximum des valeurs observées.

Juste pour illustrer l'idée, voici une version non vectorisée qui utilise des boucles Python.

```{code-cell} ipython3
grid = np.linspace(-3, 3, 50)
m = -np.inf
for x in grid:
    for y in grid:
        z = f(x, y)
        m = max(m, z)
```


### Vectorisation avec NumPy

Passons à NumPy et utilisons une grille plus grande

```{code-cell} ipython3
grid = np.linspace(-3, 3, 3_000)  # Grande grille
```

Comme première tentative de vectorisation, nous pourrions essayer quelque chose comme ceci

```{code-cell} ipython3
# Grande grille
z = np.max(f(grid, grid))    # C'est faux !
```

Le problème ici est que `f(grid, grid)` ne respecte pas la boucle imbriquée.

Par rapport à la figure ci-dessus, cela ne calcule que les valeurs de `f` le long de la diagonale.

Pour amener NumPy à calculer `f(x,y)` sur chaque paire `x,y`, nous devons utiliser `np.meshgrid`.

Ici, nous utilisons `np.meshgrid` pour créer des grilles d'entrée bidimensionnelles `x` et `y` telles que `f(x, y)` génère toutes les évaluations sur la grille produit.


```{code-cell} ipython3
# Grande grille
grid = np.linspace(-3, 3, 3_000)

x_mesh, y_mesh = np.meshgrid(grid, grid)      # meshgrid de style MATLAB

with qe.Timer():
    z_max_numpy = np.max(f(x_mesh, y_mesh))   # Cela fonctionne
```

Dans la version vectorisée, toutes les boucles s'exécutent dans du code compilé.

L'utilisation de `meshgrid` nous permet de reproduire la boucle for imbriquée.

Le résultat devrait être proche de un :

```{code-cell} ipython3
print(f"NumPy result: {z_max_numpy:.6f}")
```

### Problèmes de mémoire

Nous avons donc la bonne solution en un temps raisonnable --- mais l'utilisation de la mémoire est énorme.

Alors que les tableaux plats consomment peu de mémoire

```{code-cell} ipython3
grid.nbytes 
```

les grilles de maillage sont bidimensionnelles et donc très gourmandes en mémoire

```{code-cell} ipython3
x_mesh.nbytes + y_mesh.nbytes
```

De plus, l'exécution eager de NumPy crée de nombreux tableaux intermédiaires de la même taille !

Ce type d'utilisation de la mémoire peut poser un gros problème dans les calculs de recherche réels.


### Une comparaison avec Numba

Voyons si nous pouvons obtenir de meilleures performances en utilisant Numba avec une simple boucle.

```{code-cell} ipython3
@numba.jit
def compute_max_numba(grid):
    m = -np.inf
    for x in grid:
        for y in grid:
            z = np.cos(x**2 + y**2) / (1 + x**2 + y**2)
            m = max(m, z)
    return m
```

Testons-le :

```{code-cell} ipython3
grid = np.linspace(-3, 3, 3_000)

with qe.Timer():
    # Première exécution
    z_max_numba = compute_max_numba(grid)
```

Exécutons-le à nouveau pour éliminer le temps de compilation.

```{code-cell} ipython3
with qe.Timer():
    # Deuxième exécution
    compute_max_numba(grid)
```

Remarquez comme nous n'utilisons presque pas de mémoire --- nous n'avons besoin que de la `grid` unidimensionnelle

De plus, la vitesse d'exécution est bonne.

Sur la plupart des machines, la version Numba sera un peu plus rapide que NumPy.

La raison en est un code machine efficace ainsi que moins d'opérations de lecture-écriture en mémoire.


### Numba parallélisé

Essayons maintenant la parallélisation avec Numba en utilisant `prange` :

```{code-cell} ipython3
@numba.jit(parallel=True)
def compute_max_numba_parallel(grid):
    n = len(grid)
    m = -np.inf
    for i in numba.prange(n):
        for j in range(n):
            x = grid[i]
            y = grid[j]
            z = np.cos(x**2 + y**2) / (1 + x**2 + y**2)
            m = max(m, z)
    return m
```

Voici une exécution de préchauffage et un test.

```{code-cell} ipython3
with qe.Timer():
    # Première exécution
    z_max_parallel = compute_max_numba_parallel(grid)
```

Voici le temps pour la version précompilée.

```{code-cell} ipython3
with qe.Timer():
    # Deuxième exécution
    compute_max_numba_parallel(grid)
```

Si vous disposez de plusieurs cœurs, vous devriez constater ici les avantages de la parallélisation.

Assurons-nous que nous obtenons toujours le bon résultat (proche de un) :

```{code-cell} ipython3
print(f"Numba result: {z_max_parallel:.6f}")
```


Pour les machines puissantes et les grilles de plus grande taille, la parallélisation peut générer des gains de vitesse utiles, même sur le CPU.


### Code vectorisé avec JAX

Essayons de reproduire l'approche vectorisée de NumPy avec JAX.

Commençons par la fonction, qui remplace `np` par `jnp` et ajoute `jax.jit`

```{code-cell} ipython3
@jax.jit
def f(x, y):
    return jnp.cos(x**2 + y**2) / (1 + x**2 + y**2)

```

Nous utilisons l'approche meshgrid de style NumPy :

```{code-cell} ipython3
grid = jnp.linspace(-3, 3, 3_000)
x_mesh, y_mesh = jnp.meshgrid(grid, grid)
```

Maintenant, exécutons et mesurons le temps

```{code-cell} ipython3
with qe.Timer():
    # Première exécution
    z_max = jnp.max(f(x_mesh, y_mesh))
    # Maintenir l'interpréteur
    z_max.block_until_ready()

print(f"Plain vanilla JAX result: {z_max:.6f}")
```

Exécutons-le à nouveau pour éliminer le temps de compilation.

```{code-cell} ipython3
with qe.Timer():
    # Deuxième exécution
    z_max = jnp.max(f(x_mesh, y_mesh))
    # Maintenir l'interpréteur
    z_max.block_until_ready()
```

Une fois compilé, JAX est nettement plus rapide que NumPy, en particulier sur un GPU.

Le surcoût de compilation est un coût ponctuel qui est rentabilisé lorsque la fonction est appelée à plusieurs reprises.


### JAX plus vmap

Comme nous avons utilisé `jax.jit` ci-dessus, nous avons évité de créer de nombreux tableaux intermédiaires.

Mais nous créons toujours les grands tableaux `z_max`, `x_mesh` et `y_mesh`.

Heureusement, nous pouvons éviter cela en utilisant [jax.vmap](https://docs.jax.dev/en/latest/_autosummary/jax.vmap.html).

Voici comment nous pouvons l'appliquer à notre problème.


```{code-cell} ipython3
@jax.jit
def compute_max_vmap(grid):
    # Construire une fonction qui prend le max sur tous les x pour un y donné
    compute_column_max = lambda y: jnp.max(f(grid, y))
    # Vectoriser la fonction pour pouvoir l'appeler sur tous les y simultanément
    vectorized_compute_column_max = jax.vmap(compute_column_max)
    # Calculer le max de colonne à chaque ligne
    column_maxes = vectorized_compute_column_max(grid)
    # Calculer le max des max de colonnes et le renvoyer
    return jnp.max(column_maxes)
```

Notez que nous ne créons jamais

* la grille bidimensionnelle `x_mesh`
* la grille bidimensionnelle `y_mesh` ou
* le tableau bidimensionnel `f(x,y)`

Comme avec Numba, nous utilisons simplement le tableau plat `grid`.

Et comme tout est sous un unique `@jax.jit`, le compilateur peut fusionner toutes les opérations en un seul kernel optimisé.

Essayons.

```{code-cell} ipython3
with qe.Timer():
    # Première exécution
    z_max = compute_max_vmap(grid)
    # Maintenir l'interpréteur
    z_max.block_until_ready()

print(f"JAX vmap result: {z_max:.6f}")
```

Exécutons-le à nouveau pour éliminer le temps de compilation :

```{code-cell} ipython3
with qe.Timer():
    # Deuxième exécution
    z_max = compute_max_vmap(grid)
    # Maintenir l'interpréteur
    z_max.block_until_ready()
```


### Résumé

À notre avis, JAX est le gagnant pour les opérations vectorisées.

Il domine NumPy à la fois en termes de vitesse (grâce à la compilation JIT et à la parallélisation) et d'efficacité mémoire (grâce à vmap).

Il domine également Numba lorsqu'il est exécuté sur le GPU.

```{note}
Numba peut prendre en charge la programmation GPU via `numba.cuda`, mais nous devons alors paralléliser à la main. Pour la plupart des cas rencontrés en économie, en économétrie et en finance, il est bien préférable de laisser le compilateur JAX gérer une parallélisation efficace plutôt que d'essayer de coder ces routines nous-mêmes.
```


## Opérations séquentielles

Certaines opérations sont intrinsèquement séquentielles -- et donc difficiles voire impossibles à vectoriser.

Dans ce cas, NumPy est une mauvaise option et il ne nous reste que le choix entre Numba et JAX.

Pour comparer ces choix, nous reviendrons sur le problème de l'itération sur l'application quadratique que nous avons vu dans notre {doc}`cours sur Numba <numba>`.


### Version Numba

Voici la version Numba.

```{code-cell} ipython3
@numba.jit
def qm(x0, n, α=4.0):
    x = np.empty(n+1)
    x[0] = x0
    for t in range(n):
      x[t+1] = α * x[t] * (1 - x[t])
    return x
```

Générons une série temporelle de longueur 10 000 000 et mesurons le temps d'exécution :

```{code-cell} ipython3
n = 10_000_000

with qe.Timer():
    # Première exécution
    x = qm(0.1, n)
```

Exécutons-le à nouveau pour éliminer le temps de compilation :

```{code-cell} ipython3
with qe.Timer():
    # Deuxième exécution
    x = qm(0.1, n)
```

Numba gère cette opération séquentielle de manière très efficace.


### Version JAX

Nous ne pouvons pas remplacer directement `numba.jit` par `jax.jit` car les tableaux JAX sont immuables.

Mais nous pouvons quand même implémenter cette opération

#### Première tentative

Voici une solution de contournement utilisant la syntaxe `at[t].set` dont nous avons {ref}`parlé dans le cours JAX <jax_at_workaround>`.

Nous appliquerons une `lax.fori_loop`, qui est une version d'une boucle for pouvant être compilée par XLA.

```{code-cell} ipython3
cpu = jax.devices("cpu")[0]

@partial(jax.jit, static_argnames=("n",), device=cpu)
def qm_jax_fori(x0, n, α=4.0):

    x = jnp.empty(n + 1).at[0].set(x0)

    def update(t, x):
        return x.at[t + 1].set(α * x[t] * (1 - x[t]))

    x = lax.fori_loop(0, n, update, x)
    return x

```

* Nous maintenons `n` statique car il affecte la taille du tableau et donc JAX veut se spécialiser sur sa valeur dans le code compilé.
* Nous épinglons au CPU via `device=cpu` car cette charge de travail séquentielle se compose de nombreuses petites opérations, laissant peu d'opportunités pour le parallélisme GPU.

Important : bien que `at[t].set` semble créer un nouveau tableau à chaque étape, à l'intérieur d'une fonction compilée en JIT, le compilateur détecte que l'ancien tableau n'est plus nécessaire et effectue la mise à jour sur place !

Mesurons le temps avec les mêmes paramètres :

```{code-cell} ipython3
with qe.Timer():
    # Première exécution
    x_jax = qm_jax_fori(0.1, n)
    # Maintenir l'interpréteur
    x_jax.block_until_ready()
```

Exécutons-le à nouveau pour éliminer le surcoût de compilation :

```{code-cell} ipython3
with qe.Timer():
    # Deuxième exécution
    x_jax = qm_jax_fori(0.1, n)
    # Maintenir l'interpréteur
    x_jax.block_until_ready()
```

JAX est également assez efficace pour cette opération séquentielle !


#### Deuxième tentative

Il existe une autre manière d'implémenter la boucle qui utilise `lax.scan`.

Cette alternative est sans doute plus conforme à l'approche fonctionnelle de JAX --- bien que la syntaxe soit difficile à mémoriser.


```{code-cell} ipython3
@partial(jax.jit, static_argnames=("n",), device=cpu)
def qm_jax_scan(x0, n, α=4.0):
    def update(x, t):
        x_new = α * x * (1 - x)
        return x_new, x_new

    _, x = lax.scan(update, x0, jnp.arange(n))
    return jnp.concatenate([jnp.array([x0]), x])
```

Ce code n'est pas facile à lire mais, en substance, `lax.scan` appelle `update` de manière répétée et accumule les retours `x_new` dans un tableau.

Mesurons le temps avec les mêmes paramètres :

```{code-cell} ipython3
with qe.Timer():
    # Première exécution
    x_jax = qm_jax_scan(0.1, n)
    # Maintenir l'interpréteur
    x_jax.block_until_ready()
```

Exécutons-le à nouveau pour éliminer le surcoût de compilation :

```{code-cell} ipython3
with qe.Timer():
    # Deuxième exécution
    x_jax = qm_jax_scan(0.1, n)
    # Maintenir l'interpréteur
    x_jax.block_until_ready()
```

Étonnamment, JAX offre également de solides performances après compilation.


### Résumé

Bien que Numba et JAX offrent tous deux de solides performances pour les opérations séquentielles, il existe des différences en termes de lisibilité du code et de facilité d'utilisation.

La version Numba est directe et naturelle à lire : nous allouons simplement un tableau et le remplissons élément par élément à l'aide d'une boucle Python standard.

C'est exactement ainsi que la plupart des programmeurs conçoivent l'algorithme.

Les versions JAX, en revanche, nécessitent soit `lax.fori_loop`, soit `lax.scan`, qui sont toutes deux moins intuitives qu'une boucle Python standard.

Bien que la syntaxe `at[t].set` de JAX permette des mises à jour élément par élément, le code global reste plus difficile à lire que l'équivalent Numba.



## Recommandations générales

Prenons maintenant du recul et résumons les compromis.

Pour les **opérations vectorisées**, JAX est le choix le plus solide.

Il égale ou dépasse NumPy en vitesse, grâce à la compilation JIT et à une parallélisation efficace sur les CPU et les GPU.

La transformation `vmap` réduit l'utilisation de la mémoire et conduit souvent à un code plus clair que la vectorisation traditionnelle basée sur meshgrid.

De plus, les fonctions JAX sont automatiquement différentiables, comme nous l'explorons dans {doc}`autodiff`.

Pour les **opérations séquentielles**, Numba possède une syntaxe plus agréable.

Le code est naturel et lisible --- juste une boucle Python avec un décorateur --- et les performances sont excellentes.

JAX peut gérer les problèmes séquentiels via `lax.fori_loop` ou `lax.scan`, mais la syntaxe est moins intuitive.

D'un autre côté, les versions JAX prennent en charge la différenciation automatique.

Cela pourrait présenter un intérêt si, par exemple, nous souhaitons calculer les sensibilités d'une trajectoire aux paramètres du modèle