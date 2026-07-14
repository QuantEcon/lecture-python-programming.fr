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
  title: SciPy
  headings:
    Overview: Vue d'ensemble
    SciPy versus NumPy: SciPy versus NumPy
    Statistics: Statistiques
    Statistics::Random Variables and Distributions: Variables aléatoires et lois de probabilité
    Statistics::Alternative Syntax: Syntaxe alternative
    Statistics::Other Goodies in scipy.stats: Autres trouvailles dans scipy.stats
    Roots and Fixed Points: Racines et points fixes
    Roots and Fixed Points::Bisection: Bisection
    Roots and Fixed Points::The Newton-Raphson Method: La méthode de Newton-Raphson
    Roots and Fixed Points::Hybrid Methods: Méthodes hybrides
    Roots and Fixed Points::Multivariate Root-Finding: Recherche de racines multivariée
    Roots and Fixed Points::Fixed Points: Points fixes
    Optimization: Optimisation
    Optimization::Multivariate Optimization: Optimisation multivariée
    Integration: Intégration
    Linear Algebra: Algèbre linéaire
    Exercises: Exercices
---

(sp)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# {index}`SciPy <single: SciPy>`

```{index} single: Python; SciPy
```

En plus de ce qui est inclus dans Anaconda, ce cours nécessitera les bibliothèques suivantes :

```{code-cell} ipython3
:tags: [hide-output]

!pip install --upgrade quantecon
```

Nous utilisons les importations suivantes.

```{code-cell} ipython3
import numpy as np
import quantecon as qe
```

## Vue d'ensemble

[SciPy](https://scipy.org/) s'appuie sur NumPy pour fournir des outils courants de programmation scientifique tels que

* [algèbre linéaire](https://docs.scipy.org/doc/scipy/reference/linalg.html)
* [intégration numérique](https://docs.scipy.org/doc/scipy/reference/integrate.html)
* [interpolation](https://docs.scipy.org/doc/scipy/reference/interpolate.html)
* [optimisation](https://docs.scipy.org/doc/scipy/reference/optimize.html)
* [lois de probabilité et génération de nombres aléatoires](https://docs.scipy.org/doc/scipy/reference/stats.html)
* [traitement du signal](https://docs.scipy.org/doc/scipy/reference/signal.html)
* etc., etc.

Comme NumPy, SciPy est stable, mature et largement utilisé.

De nombreuses routines de SciPy sont de fines surcouches autour de bibliothèques Fortran standard de l'industrie telles que [LAPACK](https://en.wikipedia.org/wiki/LAPACK), [BLAS](https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms), etc.

Il n'est pas vraiment nécessaire d'« apprendre » SciPy dans son ensemble.

Une approche plus courante consiste à avoir une idée de ce que contient la bibliothèque, puis à consulter la [documentation](https://docs.scipy.org/doc/scipy/reference/index.html) au besoin.

Dans ce cours, nous cherchons uniquement à mettre en évidence quelques parties utiles du package.

## {index}`SciPy <single: SciPy>` versus {index}`NumPy <single: NumPy>`

SciPy est un package qui contient divers outils construits sur NumPy, en utilisant son type de données tableau et les fonctionnalités connexes.

````{note} 
Dans les anciennes versions de SciPy (`scipy < 0.15.1`), importer le package importait aussi les symboles NumPy dans l'espace de noms global, comme on peut le voir dans cet extrait du fichier d'initialisation de SciPy :

```python
from numpy import *
from numpy.random import rand, randn
from numpy.fft import fft, ifft
from numpy.lib.scimath import *
```

Cependant, il est préférable d'utiliser explicitement les fonctionnalités de NumPy.

```python
import numpy as np

a = np.identity(3)
```

Les versions plus récentes de SciPy (1.15+) n'importent plus automatiquement les symboles NumPy.
````

Ce qui est utile dans SciPy, c'est la fonctionnalité de ses sous-packages

* `scipy.optimize`, `scipy.integrate`, `scipy.stats`, etc.

Explorons quelques-uns des principaux sous-packages.

## Statistiques

```{index} single: SciPy; Statistics
```

Le sous-package `scipy.stats` fournit

* de nombreux objets de variables aléatoires (densités, fonctions de répartition, échantillonnage aléatoire, etc.)
* certaines procédures d'estimation
* certains tests statistiques

### Variables aléatoires et lois de probabilité

Rappelons que `numpy.random` fournit des outils pour générer des variables aléatoires

```{code-cell} python3
rng = np.random.default_rng()
rng.beta(5, 5, size=3)
```

Ceci génère un tirage de la loi ayant la fonction de densité ci-dessous lorsque `a, b = 5, 5`

```{math}
:label: betadist2

f(x; a, b) = \frac{x^{(a - 1)} (1 - x)^{(b - 1)}}
    {\int_0^1 u^{(a - 1)} (1 - u)^{(b - 1)} du}
    \qquad (0 \leq x \leq 1)
```

Parfois, nous avons besoin d'accéder à la densité elle-même, ou à la fonction de répartition, aux quantiles, etc.

Pour cela, nous pouvons utiliser `scipy.stats`, qui fournit toutes ces fonctionnalités ainsi que la génération de nombres aléatoires dans une seule interface cohérente.

Voici un exemple d'utilisation

```{code-cell} ipython
from scipy.stats import beta
import matplotlib.pyplot as plt

q = beta(5, 5)      # Beta(a, b), avec a = b = 5
obs = q.rvs(2000)   # 2000 observations
grid = np.linspace(0.01, 0.99, 100)

fig, ax = plt.subplots()
ax.hist(obs, bins=40, density=True)
ax.plot(grid, q.pdf(grid), 'k-', linewidth=2)
plt.show()
```

L'objet `q` qui représente la loi possède des méthodes utiles supplémentaires, notamment

```{code-cell} python3
q.cdf(0.4)      # Fonction de répartition
```

```{code-cell} python3
q.ppf(0.8)      # Fonction quantile (fonction de répartition inverse)
```

```{code-cell} python3
q.mean()
```

La syntaxe générale pour créer ces objets qui représentent des lois de probabilité (de type `rv_frozen`) est

> `name = scipy.stats.distribution_name(shape_parameters, loc=c, scale=d)`

Ici, `distribution_name` est l'un des noms de lois de probabilité de [scipy.stats](https://docs.scipy.org/doc/scipy/reference/stats.html).

Les paramètres `loc` et `scale` transforment la variable aléatoire originale
$X$ en $Y = c + d X$.

### Syntaxe alternative

Il existe une manière alternative d'appeler les méthodes décrites ci-dessus.

Par exemple, le code qui génère la figure ci-dessus peut être remplacé par

```{code-cell} python3
obs = beta.rvs(5, 5, size=2000)
grid = np.linspace(0.01, 0.99, 100)

fig, ax = plt.subplots()
ax.hist(obs, bins=40, density=True)
ax.plot(grid, beta.pdf(grid, 5, 5), 'k-', linewidth=2)
plt.show()
```

### Autres trouvailles dans scipy.stats

Il existe une variété de fonctions statistiques dans `scipy.stats`.

Par exemple, `scipy.stats.linregress` implémente la régression linéaire simple

```{code-cell} python3
from scipy.stats import linregress

x = rng.standard_normal(200)
y = 2 * x + 0.1 * rng.standard_normal(200)
gradient, intercept, r_value, p_value, std_err = linregress(x, y)
gradient, intercept
```

Pour voir la liste complète, consultez la [documentation](https://docs.scipy.org/doc/scipy/reference/stats.html#statistical-functions-scipy-stats).

## Racines et points fixes

Une **racine** ou un **zéro** d'une fonction réelle $f$ sur $[a,b]$ est un $x \in [a, b]$ tel que $f(x)=0$.

Par exemple, si nous traçons la fonction

```{math}
:label: root_f

f(x) = \sin(4 (x - 1/4)) + x + x^{20} - 1
```

avec $x \in [0,1]$, nous obtenons

```{code-cell} python3
f = lambda x: np.sin(4 * (x - 1/4)) + x + x**20 - 1
x = np.linspace(0, 1, 100)

fig, ax = plt.subplots()
ax.plot(x, f(x), label='$f(x)$')
ax.axhline(ls='--', c='k')
ax.set_xlabel('$x$', fontsize=12)
ax.set_ylabel('$f(x)$', fontsize=12)
ax.legend(fontsize=12)
plt.show()
```

L'unique racine est d'environ 0,408.

Considérons quelques techniques numériques pour trouver les racines.

### {index}`Bisection <single: Bisection>`

```{index} single: SciPy; Bisection
```

L'un des algorithmes les plus courants pour la recherche numérique de racines est la *bissection*.

Pour comprendre l'idée, rappelez-vous le jeu bien connu où

* Le joueur A pense à un nombre secret entre 1 et 100
* Le joueur B demande s'il est inférieur à 50
    * Si oui, B demande s'il est inférieur à 25
    * Si non, B demande s'il est inférieur à 75

Et ainsi de suite.

C'est la bissection.

Voici une implémentation simpliste de l'algorithme en Python.

Elle fonctionne pour toutes les fonctions continues croissantes suffisamment bien comportées avec $f(a) < 0 < f(b)$

(bisect_func)=
```{code-cell} python3
def bisect(f, a, b, tol=10e-5):
    """
    Implements the bisection root finding algorithm, assuming that f is a
    real-valued function on [a, b] satisfying f(a) < 0 < f(b).
    """
    lower, upper = a, b

    while upper - lower > tol:
        middle = 0.5 * (upper + lower)
        if f(middle) > 0:   # la racine est entre lower et middle
            lower, upper = lower, middle
        else:               # la racine est entre middle et upper
            lower, upper = middle, upper

    return 0.5 * (upper + lower)
```

Testons-la en utilisant la fonction $f$ définie dans {eq}`root_f`

```{code-cell} python3
bisect(f, 0, 1)
```

Sans surprise, SciPy fournit sa propre fonction de bissection.

Testons-la en utilisant la même fonction $f$ définie dans {eq}`root_f`

```{code-cell} python3
from scipy.optimize import bisect

bisect(f, 0, 1)
```

### La {index}`méthode de Newton-Raphson <single: Newton-Raphson Method>`

```{index} single: SciPy; Newton-Raphson Method
```

Un autre algorithme de recherche de racines très courant est la [méthode de Newton-Raphson](https://en.wikipedia.org/wiki/Newton%27s_method).

Dans SciPy, cet algorithme est implémenté par `scipy.optimize.newton`.

Contrairement à la bissection, la méthode de Newton-Raphson utilise l'information sur la pente locale afin d'accélérer la convergence.

Étudions cela en utilisant la même fonction $f$ définie ci-dessus.

Avec une condition initiale appropriée pour la recherche, nous obtenons la convergence :

```{code-cell} python3
from scipy.optimize import newton

newton(f, 0.2)   # Démarre la recherche à la condition initiale x = 0.2
```

Mais d'autres conditions initiales conduisent à un échec de la convergence :

```{code-cell} python3
newton(f, 0.7)   # Démarre la recherche à x = 0.7 à la place
```

### Méthodes hybrides

Un principe général des méthodes numériques est le suivant :

* Si vous avez une connaissance spécifique d'un problème donné, vous pourriez être en mesure de l'exploiter pour gagner en efficacité.
* Sinon, le choix de l'algorithme implique un compromis entre vitesse et robustesse.

En pratique, la plupart des algorithmes par défaut pour la recherche de racines, l'optimisation et les points fixes utilisent des méthodes *hybrides*.

Ces méthodes combinent généralement une méthode rapide avec une méthode robuste de la manière suivante :

1. Tenter d'utiliser une méthode rapide
1. Vérifier les diagnostics
1. Si les diagnostics sont mauvais, passer à un algorithme plus robuste

Dans `scipy.optimize`, la fonction `brentq` est une telle méthode hybride et constitue un bon choix par défaut

```{code-cell} python3
from scipy.optimize import brentq

brentq(f, 0, 1)
```

Ici, la solution correcte est trouvée et la vitesse est meilleure que la bissection :

```{code-cell} ipython
with qe.Timer(unit="milliseconds"):
    brentq(f, 0, 1)
```

```{code-cell} ipython
with qe.Timer(unit="milliseconds"):
    bisect(f, 0, 1)
```

### Recherche de racines multivariée

```{index} single: SciPy; Multivariate Root-Finding
```

Utilisez `scipy.optimize.fsolve`, une surcouche pour une méthode hybride dans MINPACK.

Voir la [documentation](https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.fsolve.html) pour plus de détails.

### Points fixes

Un **point fixe** d'une fonction réelle $f$ sur $[a,b]$ est un $x \in [a, b]$ tel que $f(x)=x$.

```{index} single: SciPy; Fixed Points
```

SciPy dispose également d'une fonction pour trouver des points fixes (scalaires)

```{code-cell} python3
from scipy.optimize import fixed_point

fixed_point(lambda x: x**2, 10.0)  # 10.0 est une supposition initiale
```

Si vous n'obtenez pas de bons résultats, vous pouvez toujours revenir au chercheur de racines `brentq`, car
le point fixe d'une fonction $f$ est la racine de $g(x) := x - f(x)$.

## {index}`Optimisation <single: Optimization>`

```{index} single: SciPy; Optimization
```

La plupart des packages numériques ne fournissent que des fonctions de *minimisation*.

La maximisation peut être réalisée en rappelant que le maximiseur d'une fonction $f$ sur le domaine $D$ est
le minimiseur de $-f$ sur $D$.

La minimisation est étroitement liée à la recherche de racines : pour les fonctions lisses, les optima intérieurs correspondent aux racines de la dérivée première.

Le compromis vitesse/robustesse décrit ci-dessus est également présent dans l'optimisation numérique.

À moins que vous n'ayez des informations préalables que vous pouvez exploiter, il est généralement préférable d'utiliser des méthodes hybrides.

Pour la minimisation univariée (c'est-à-dire scalaire) sous contrainte, une bonne option hybride est `fminbound`

```{code-cell} python3
from scipy.optimize import fminbound

fminbound(lambda x: x**2, -1, 2)  # Recherche dans [-1, 2]
```

### Optimisation multivariée

```{index} single: Optimization; Multivariate
```

Les optimiseurs locaux multivariés incluent `minimize`, `fmin`, `fmin_powell`, `fmin_cg`, `fmin_bfgs` et `fmin_ncg`.

Les optimiseurs locaux multivariés sous contrainte incluent `fmin_l_bfgs_b`, `fmin_tnc`, `fmin_cobyla`.

Voir la [documentation](https://docs.scipy.org/doc/scipy/reference/optimize.html) pour plus de détails.

## {index}`Intégration <single: Integration>`

```{index} single: SciPy; Integration
```

La plupart des méthodes d'intégration numérique fonctionnent en calculant l'intégrale d'un polynôme approximant.

L'erreur résultante dépend de la qualité de l'ajustement du polynôme à l'intégrande, qui dépend à son tour du degré de « régularité » de l'intégrande.

Dans SciPy, le module pertinent pour l'intégration numérique est `scipy.integrate`.

Un bon choix par défaut pour l'intégration univariée est `quad`

```{code-cell} python3
from scipy.integrate import quad

integral, error = quad(lambda x: x**2, 0, 1)
integral
```

En fait, `quad` est une interface vers une routine d'intégration numérique très standard de la bibliothèque Fortran QUADPACK.

Elle utilise la [quadrature de Clenshaw-Curtis](https://en.wikipedia.org/wiki/Clenshaw-Curtis_quadrature), basée sur un développement en termes de polynômes de Tchebychev.

Il existe d'autres options pour l'intégration univariée — une option utile est `fixed_quad`, qui est rapide et fonctionne donc bien à l'intérieur des boucles `for`.

Il existe également des fonctions pour l'intégration multivariée.

Voir la [documentation](https://docs.scipy.org/doc/scipy/reference/integrate.html) pour plus de détails.

## {index}`Algèbre linéaire <single: Linear Algebra>`

```{index} single: SciPy; Linear Algebra
```

Nous avons vu que NumPy fournit un module d'algèbre linéaire appelé `linalg`.

SciPy fournit également un module d'algèbre linéaire du même nom.

Ce dernier n'est pas un sur-ensemble exact du premier, mais dans l'ensemble il possède davantage de fonctionnalités.

Nous vous laissons explorer l'[ensemble des routines disponibles](https://docs.scipy.org/doc/scipy/reference/linalg.html).

## Exercices

Les premiers exercices concernent l'évaluation d'une option d'achat européenne sous
l'hypothèse de neutralité au risque. Le prix satisfait

$$
P = \beta^n \mathbb E \max\{ S_n - K, 0 \}
$$

où

1. $\beta$ est un facteur d'actualisation,
2. $n$ est la date d'échéance,
2. $K$ est le prix d'exercice et
3. $\{S_t\}$ est le prix de l'actif sous-jacent à chaque instant $t$.

Par exemple, si l'option d'achat consiste à acheter des actions Amazon au prix d'exercice $K$, le propriétaire a le droit (mais non l'obligation) d'acheter 1 action Amazon au prix $K$ après $n$ jours.

Le gain est donc $\max\{S_n - K, 0\}$

Le prix est l'espérance du gain, actualisée à la valeur actuelle.


```{exercise-start}
:label: sp_ex01
```

Supposons que $S_n$ suit la loi [log-normale](https://en.wikipedia.org/wiki/Log-normal_distribution) de paramètres $\mu$ et $\sigma$. Soit $f$ la densité de cette loi. Alors

$$
P = \beta^n \int_0^\infty \max\{x - K, 0\} f(x) dx
$$

Tracez la fonction 

$$
g(x) = \beta^n  \max\{x - K, 0\} f(x)
$$ 

sur l'intervalle $[0, 400]$ lorsque `μ, σ, β, n, K = 4, 0.25, 0.99, 10, 40`.

```{hint}
:class: dropdown

Depuis `scipy.stats`, vous pouvez importer `lognorm` puis utiliser `lognorm.pdf(x, σ, scale=np.exp(μ))` pour obtenir la densité $f$.
```

```{exercise-end}
```

```{solution-start} sp_ex01
:class: dropdown
```

Voici une solution possible

```{code-cell} ipython3
from scipy.integrate import quad
from scipy.stats import lognorm

μ, σ, β, n, K = 4, 0.25, 0.99, 10, 40

def g(x):
    return β**n * np.maximum(x - K, 0) * lognorm.pdf(x, σ, scale=np.exp(μ))

x_grid = np.linspace(0, 400, 1000)
y_grid = g(x_grid) 

fig, ax = plt.subplots()
ax.plot(x_grid, y_grid, label="$g$")
ax.legend()
plt.show()
```

```{solution-end}
```

```{exercise}
:label: sp_ex02

Afin d'obtenir le prix de l'option, calculez l'intégrale de cette fonction numériquement en utilisant `quad` de `scipy.integrate`.

```

```{solution-start} sp_ex02
:class: dropdown
```

```{code-cell} ipython3
P, error = quad(g, 0, 1_000)
print(f"The numerical integration based option price is {P:.3f}")
```

```{solution-end}
```

```{exercise}
:label: sp_ex03

Essayez d'obtenir un résultat similaire en utilisant Monte-Carlo pour calculer le terme d'espérance dans le prix de l'option, plutôt que `quad`.

En particulier, utilisez le fait que si $S_n^1, \ldots, S_n^M$ sont des tirages indépendants
de la loi log-normale spécifiée ci-dessus, alors, par la loi des
grands nombres,

$$ \mathbb E \max\{ S_n - K, 0 \} 
    \approx
    \frac{1}{M} \sum_{m=1}^M \max \{S_n^m - K, 0 \}
    $$
    
Posez `M = 10_000_000`

```

```{solution-start} sp_ex03
:class: dropdown
```

Voici une solution :

```{code-cell} ipython3
rng = np.random.default_rng()
M = 10_000_000
S = np.exp(μ + σ * rng.standard_normal(M))
return_draws = np.maximum(S - K, 0)
P = β**n * np.mean(return_draws) 
print(f"The Monte Carlo option price is {P:3f}")
```


```{solution-end}
```



```{exercise}
:label: sp_ex1

Dans {ref}`ce cours <functions>`, nous avons abordé le concept d'{ref}`appels récursifs de fonctions <recursive_functions>`.

Essayez d'écrire une implémentation récursive de la fonction de bissection maison {ref}`décrite ci-dessus <bisect_func>`.

Testez-la sur la fonction {eq}`root_f`.
```

```{solution-start} sp_ex1
:class: dropdown
```

Voici une solution raisonnable :


```{code-cell} python3
def bisect(f, a, b, tol=10e-5):
    """
    Implements the bisection root-finding algorithm, assuming that f is a
    real-valued function on [a, b] satisfying f(a) < 0 < f(b).
    """
    lower, upper = a, b
    if upper - lower < tol:
        return 0.5 * (upper + lower)
    else:
        middle = 0.5 * (upper + lower)
        print(f'Current mid point = {middle}')
        if f(middle) > 0:   # Implique que la racine est entre lower et middle
            return bisect(f, lower, middle)
        else:               # Implique que la racine est entre middle et upper
            return bisect(f, middle, upper)
```

Nous pouvons la tester comme suit

```{code-cell} python3
f = lambda x: np.sin(4 * (x - 0.25)) + x + x**20 - 1
bisect(f, 0, 1)
```

```{solution-end}
```