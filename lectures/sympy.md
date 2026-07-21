---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.14.5
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
translation:
  title: SymPy
  headings:
    Overview: Vue d'ensemble
    Getting Started: Pour commencer
    Symbolic algebra: Algèbre symbolique
    Symbolic algebra::Symbols: Symboles
    Symbolic algebra::Expressions: Expressions
    Symbolic algebra::Equations: Équations
    'Symbolic algebra::Equations::Example: fixed point computation': "Exemple\_: calcul de point fixe"
    Symbolic algebra::Inequalities and logic: Inégalités et logique
    Symbolic algebra::Series: Séries
    'Symbolic algebra::Series::Example: bank deposits': "Exemple\_: dépôts bancaires"
    'Symbolic algebra::Series::Example: discrete random variable': "Exemple\_: variable aléatoire discrète"
    Symbolic Calculus: Calcul symbolique
    Symbolic Calculus::Limits: Limites
    Symbolic Calculus::Derivatives: Dérivées
    Symbolic Calculus::Integrals: Intégrales
    Plotting: Représentation graphique
    'Application: Two-person Exchange Economy': "Application\_: économie d'échange à deux personnes"
    Exercises: Exercices
---

(sympy=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

(sympy)=
# {index}`SymPy <single: SymPy>`

```{index} single: Python; SymPy
```

## Vue d'ensemble

Contrairement aux bibliothèques numériques qui traitent des valeurs, [SymPy](https://www.sympy.org/en/index.html) se concentre sur la manipulation directe de symboles et d'expressions mathématiques.

SymPy fournit [un large éventail de fonctionnalités](https://www.sympy.org/en/features.html) comprenant

- l'expression symbolique
- la résolution d'équations
- la simplification
- le calcul infinitésimal
- les matrices
- les mathématiques discrètes, etc.

Ces fonctions font de SymPy une alternative open-source populaire à d'autres logiciels de calcul symbolique propriétaires tels que Mathematica.

Dans ce cours, nous explorerons certaines des fonctionnalités de SymPy et démontrerons comment utiliser les fonctions de base de SymPy pour résoudre des modèles économiques.

## Pour commencer

Importons d'abord la bibliothèque et initialisons l'imprimante pour la sortie symbolique

```{code-cell} ipython3
from sympy import *
from sympy.plotting import plot, plot3d_parametric_line, plot3d
from sympy.solvers.inequalities import reduce_rational_inequalities
from sympy.stats import Poisson, Exponential, Binomial, density, moment, E, cdf

import numpy as np
import matplotlib.pyplot as plt

# Activer l'imprimante mathjax
init_printing(use_latex='mathjax')
```

## Algèbre symbolique

### Symboles

Nous initialisons d'abord quelques symboles avec lesquels travailler

```{code-cell} ipython3
x, y, z = symbols('x y z')
```

Les symboles sont les unités de base du calcul symbolique dans SymPy.

### Expressions

Nous pouvons maintenant utiliser les symboles `x`, `y` et `z` pour construire des expressions et des équations.

Ici, nous construisons d'abord une expression simple

```{code-cell} ipython3
expr = (x+y) ** 2
expr
```

Nous pouvons développer cette expression avec la fonction `expand`

```{code-cell} ipython3
expand_expr = expand(expr)
expand_expr
```

et la factoriser de nouveau sous forme factorisée avec la fonction `factor`

```{code-cell} ipython3
factor(expand_expr)
```

Nous pouvons résoudre cette expression

```{code-cell} ipython3
solve(expr)
```

Notez que cela équivaut à résoudre l'équation suivante pour `x`

$$
(x + y)^2 = 0 
$$

```{note}
[Solvers](https://docs.sympy.org/latest/modules/solvers/index.html) est un module important doté d'outils pour résoudre différents types d'équations.

Une variété de solveurs sont disponibles dans SymPy selon la nature du problème.
```

### Équations

SymPy fournit plusieurs fonctions pour manipuler les équations.

Développons une équation avec l'expression que nous avons définie précédemment

```{code-cell} ipython3
eq = Eq(expr, 0)
eq
```

La résolution de cette équation par rapport à $x$ donne le même résultat que la résolution directe de l'expression

```{code-cell} ipython3
solve(eq, x)
```

SymPy peut traiter des équations à solutions multiples

```{code-cell} ipython3
eq = Eq(expr, 1)
solve(eq, x)
```

La fonction `solve` peut également combiner plusieurs équations ensemble et résoudre un système d'équations

```{code-cell} ipython3
eq2 = Eq(x, y)
eq2
```

```{code-cell} ipython3
solve([eq, eq2], [x, y])
```

Nous pouvons aussi résoudre pour la valeur de $y$ en substituant simplement $x$ par $y$

```{code-cell} ipython3
expr_sub = expr.subs(x, y)
expr_sub
```

```{code-cell} ipython3
solve(Eq(expr_sub, 1))
```

Voici un autre exemple d'équation avec le symbole `x` et les fonctions `sin`, `cos` et `tan` utilisant la fonction `Eq`

```{code-cell} ipython3
# Créer une équation
eq = Eq(cos(x) / (tan(x)/sin(x)), 0)
eq
```

Maintenant nous simplifions cette équation en utilisant la fonction `simplify`

```{code-cell} ipython3
# Simplifier une expression
simplified_expr = simplify(eq)
simplified_expr
```

De nouveau, nous utilisons la fonction `solve` pour résoudre cette équation

```{code-cell} ipython3
# Résoudre l'équation
sol = solve(eq, x)
sol
```

SymPy peut aussi traiter des équations plus complexes impliquant la trigonométrie et les nombres complexes.

Nous le démontrons en utilisant la [formule d'Euler](https://en.wikipedia.org/wiki/Euler%27s_formula)

```{code-cell} ipython3
# 'I' représente le nombre imaginaire i 
euler = cos(x) + I*sin(x)
euler
```

```{code-cell} ipython3
simplify(euler)
```

Si cela vous intéresse, nous vous encourageons à lire le cours sur la [trigonométrie et les nombres complexes](https://python.quantecon.org/complex_and_trig.html).

#### Exemple : calcul de point fixe

Le calcul de point fixe est fréquemment utilisé en économie et en finance.

Ici, nous résolvons le point fixe de la dynamique de croissance de Solow-Swan :

$$
k_{t+1}=s f\left(k_t\right)+(1-\delta) k_t, \quad t=0,1, \ldots
$$

où $k_t$ est le stock de capital, $f$ est une fonction de production, $\delta$ est un taux de dépréciation.

Nous souhaitons calculer le point fixe de cette dynamique, c'est-à-dire la valeur de $k$ telle que $k_{t+1} = k_t$.

Avec $f(k) = Ak^\alpha$, nous pouvons montrer le point fixe unique de la dynamique $k^*$ avec papier et crayon :

$$
k^*:=\left(\frac{s A}{\delta}\right)^{1 /(1-\alpha)}
$$ 

Cela peut être facilement calculé dans SymPy

```{code-cell} ipython3
A, s, k, α, δ = symbols('A s k^* α δ')
```

Maintenant nous résolvons pour le point fixe $k^*$

$$
k^* = sA(k^*)^{\alpha}+(1-\delta) k^*
$$

```{code-cell} ipython3
# Définir la dynamique de croissance de Solow-Swan
solow = Eq(s*A*k**α + (1-δ)*k, k)
solow
```

```{code-cell} ipython3
solve(solow, k)
```

### Inégalités et logique

SymPy permet aussi aux utilisateurs de définir des inégalités et des opérateurs d'ensembles et fournit un large éventail d'[opérations](https://docs.sympy.org/latest/modules/solvers/inequalities.html).

```{code-cell} ipython3
reduce_inequalities([2*x + 5*y <= 30, 4*x + 2*y <= 20], [x])
```

```{code-cell} ipython3
And(2*x + 5*y <= 30, x > 0)
```

### Séries

Les séries sont largement utilisées en économie et en statistique, de l'évaluation des actifs à l'espérance de variables aléatoires discrètes.

Nous pouvons construire une simple série de sommations en utilisant la fonction `Sum` et les symboles `Indexed`

```{code-cell} ipython3
x, y, i, j = symbols("x y i j")
sum_xy = Sum(Indexed('x', i)*Indexed('y', j), 
            (i, 0, 3),
            (j, 0, 3))
sum_xy
```

Pour évaluer la somme, nous pouvons [`lambdify`](https://docs.sympy.org/latest/modules/utilities/lambdify.html#sympy.utilities.lambdify.lambdify) la formule.

L'expression lambdifiée peut prendre des valeurs numériques en entrée pour $x$ et $y$ et calculer le résultat

```{code-cell} ipython3
sum_xy = lambdify([x, y], sum_xy)
grid = np.arange(0, 4, 1)
sum_xy(grid, grid)
```

#### Exemple : dépôts bancaires

Imaginez une banque avec $D_0$ comme dépôt au temps $t$.

Elle prête $(1-r)$ de ses dépôts et conserve une fraction $r$ en réserves de trésorerie.

Ses dépôts sur un horizon temporel infini peuvent s'écrire

$$
\sum_{i=0}^\infty (1-r)^i D_0
$$

Calculons les dépôts au temps $t$

```{code-cell} ipython3
D = symbols('D_0')
r = Symbol('r', positive=True)
Dt = Sum('(1 - r)^i * D_0', (i, 0, oo))
Dt
```

Nous pouvons appeler la méthode `doit` pour évaluer la série

```{code-cell} ipython3
Dt.doit()
```

Simplifier l'expression ci-dessus donne

```{code-cell} ipython3
simplify(Dt.doit())
```

Ceci est cohérent avec la solution dans le cours sur les [séries géométriques](https://intro.quantecon.org/geom_series.html#example-the-money-multiplier-in-fractional-reserve-banking).


#### Exemple : variable aléatoire discrète

Dans l'exemple suivant, nous calculons l'espérance d'une variable aléatoire discrète.

Définissons une variable aléatoire discrète $X$ suivant une [loi de Poisson](https://en.wikipedia.org/wiki/Poisson_distribution) :

$$
f(x) = \frac{\lambda^x e^{-\lambda}}{x!}, \quad x = 0, 1, 2, \ldots
$$

```{code-cell} ipython3
λ = symbols('lambda')

# Nous restreignons le symbole x aux entiers positifs
x = Symbol('x', integer=True, positive=True)
pmf = λ**x * exp(-λ) / factorial(x)
pmf
```

Nous pouvons vérifier si la somme des probabilités pour toutes les valeurs possibles égale $1$ :

$$
\sum_{x=0}^{\infty} f(x) = 1
$$

```{code-cell} ipython3
sum_pmf = Sum(pmf, (x, 0, oo))
sum_pmf.doit()
```

L'espérance de la distribution est :

$$
E(X) = \sum_{x=0}^{\infty} x f(x) 
$$

```{code-cell} ipython3
fx = Sum(x*pmf, (x, 0, oo))
fx.doit()
```

SymPy inclut un sous-module de statistique appelé [`Stats`](https://docs.sympy.org/latest/modules/stats.html).

`Stats` offre des distributions intégrées et des fonctions sur les distributions de probabilité.

Le calcul ci-dessus peut aussi être condensé en une seule ligne en utilisant la fonction d'espérance `E` dans le module `Stats`

```{code-cell} ipython3
λ = Symbol("λ", positive = True)

# Utilisation de la méthode sympy.stats.Poisson()
X = Poisson("x", λ)
E(X)
```

## Calcul symbolique

SymPy nous permet d'effectuer diverses opérations de calcul infinitésimal, telles que les limites, la différentiation et l'intégration.


### Limites

Nous pouvons calculer les limites d'une expression donnée en utilisant la fonction `limit`

```{code-cell} ipython3
# Définir une expression
f = x**2 / (x-1)

# Calculer la limite
lim = limit(f, x, 0)
lim
```

### Dérivées

Nous pouvons différencier toute expression SymPy en utilisant la fonction `diff`

```{code-cell} ipython3
# Différencier une fonction par rapport à x
df = diff(f, x)
df
```

### Intégrales

Nous pouvons calculer des intégrales définies et indéfinies en utilisant la fonction `integrate`

```{code-cell} ipython3
# Calculer l'intégrale indéfinie
indef_int = integrate(df, x)
indef_int
```

Utilisons cette fonction pour calculer la fonction génératrice des moments de la [loi exponentielle](https://en.wikipedia.org/wiki/Exponential_distribution) avec la fonction de densité de probabilité :

$$
f(x) = \lambda e^{-\lambda x}, \quad x \ge 0
$$

```{code-cell} ipython3
λ = Symbol('lambda', positive=True)
x = Symbol('x', positive=True)
pdf = λ * exp(-λ*x)
pdf
```

```{code-cell} ipython3
t = Symbol('t', positive=True)
moment_t = integrate(exp(t*x) * pdf, (x, 0, oo))
simplify(moment_t)
```

Notez que nous pouvons aussi utiliser le module `Stats` pour calculer le moment

```{code-cell} ipython3
X = Exponential(x, λ)
```

```{code-cell} ipython3
moment(X, 1)
```

```{code-cell} ipython3
E(X**t)
```

En utilisant la fonction `integrate`, nous pouvons dériver la fonction de répartition de la loi exponentielle avec $\lambda = 0.5$

```{code-cell} ipython3
λ_pdf = pdf.subs(λ, 1/2)
λ_pdf
```

```{code-cell} ipython3
integrate(λ_pdf, (x, 0, 4))
```

Utiliser `cdf` dans le module `Stats` donne la même solution

```{code-cell} ipython3
cdf(X, 1/2)
```

```{code-cell} ipython3
# Insérer une valeur pour z 
λ_cdf = cdf(X, 1/2)(4)
λ_cdf
```

```{code-cell} ipython3
# Substituer λ
λ_cdf.subs({λ: 1/2})
```

## Représentation graphique

SymPy fournit une puissante fonctionnalité de représentation graphique.

D'abord, nous traçons une fonction simple en utilisant la fonction `plot`

```{code-cell} ipython3
f = sin(2 * sin(2 * sin(2 * sin(x))))
p = plot(f, (x, -10, 10), show=False)
p.title = 'Un graphique simple'
p.show()
```

De manière similaire à Matplotlib, SymPy fournit une interface pour personnaliser le graphique

```{code-cell} ipython3
plot_f = plot(f, (x, -10, 10), 
              xlabel='', ylabel='', 
              legend = True, show = False)
plot_f[0].label = 'f(x)'
df = diff(f)
plot_df = plot(df, (x, -10, 10), 
            legend = True, show = False)
plot_df[0].label = 'f\'(x)'
plot_f.append(plot_df[0])
plot_f.show()
```

Il prend aussi en charge le tracé de fonctions implicites et la visualisation d'inégalités

```{code-cell} ipython3
p = plot_implicit(Eq((1/x + 1/y)**2, 1))
```

```{code-cell} ipython3
p = plot_implicit(And(2*x + 5*y <= 30, 4*x + 2*y >= 20),
                     (x, -1, 10), (y, -10, 10))
```

et des visualisations dans un espace tridimensionnel

```{code-cell} ipython3
p = plot3d(cos(2*x + y), zlabel='')
```

## Application : économie d'échange à deux personnes

Imaginez une économie d'échange pur avec deux personnes ($a$ et $b$) et deux biens enregistrés en proportions ($x$ et $y$).

Elles peuvent échanger des biens entre elles selon leurs préférences.

Supposons que les fonctions d'utilité des consommateurs sont données par

$$
u_a(x, y) = x^{\alpha} y^{1-\alpha}
$$

$$
u_b(x, y) = (1 - x)^{\beta} (1 - y)^{1-\beta}
$$

où $\alpha, \beta \in (0, 1)$.

D'abord nous définissons les symboles et les fonctions d'utilité

```{code-cell} ipython3
# Définir les symboles et les fonctions d'utilité
x, y, α, β = symbols('x, y, α, β')
u_a = x**α * y**(1-α)
u_b = (1 - x)**β * (1 - y)**(1 - β)
```

```{code-cell} ipython3
u_a
```

```{code-cell} ipython3
u_b
```

Nous nous intéressons à l'allocation Pareto-optimale des biens $x$ et $y$.

Notez qu'un point est Pareto-efficace lorsque l'allocation est optimale pour une personne étant donné l'allocation pour l'autre personne.

En termes d'utilité marginale :

$$
\frac{\frac{\partial u_a}{\partial x}}{\frac{\partial u_a}{\partial y}} = \frac{\frac{\partial u_b}{\partial x}}{\frac{\partial u_b}{\partial y}}
$$

```{code-cell} ipython3
# Un point est Pareto-efficace lorsque l'allocation est optimale 
# pour une personne étant donné l'allocation pour l'autre personne

pareto = Eq(diff(u_a, x)/diff(u_a, y), 
            diff(u_b, x)/diff(u_b, y))
pareto
```

```{code-cell} ipython3
# Résoudre l'équation
sol = solve(pareto, y)[0]
sol
```

Calculons les allocations Pareto-optimales de l'économie (courbes de contrat) avec $\alpha = \beta = 0.5$ en utilisant SymPy

```{code-cell} ipython3
# Substituer α = 0.5 et β = 0.5
sol.subs({α: 0.5, β: 0.5})
```

Nous pouvons utiliser ce résultat pour visualiser davantage de courbes de contrat sous différents paramètres

```{code-cell} ipython3
# Tracer une plage de α et de β
params = [{α: 0.5, β: 0.5}, 
          {α: 0.1, β: 0.9},
          {α: 0.1, β: 0.8},
          {α: 0.8, β: 0.9},
          {α: 0.4, β: 0.8}, 
          {α: 0.8, β: 0.1},
          {α: 0.9, β: 0.8},
          {α: 0.8, β: 0.4},
          {α: 0.9, β: 0.1}]

p = plot(xlabel='x', ylabel='y', show=False)

for param in params:
    p_add = plot(sol.subs(param), (x, 0, 1), 
                 show=False)
    p.append(p_add[0])
p.show()
```

Nous vous invitons à jouer avec les paramètres et à voir comment les courbes de contrat changent et à réfléchir aux deux questions suivantes :

- Pouvez-vous imaginer une manière de dessiner le même graphique en utilisant `numpy` ?
- Quelle serait la difficulté d'écrire une implémentation `numpy` ?

## Exercices

```{exercise}
:label: sympy_ex1

La règle de L'Hôpital énonce que pour deux fonctions $f(x)$ et $g(x)$, si $\lim_{x \to a} f(x) = \lim_{x \to a} g(x) = 0$ ou $\pm \infty$, alors

$$
\lim_{x \to a} \frac{f(x)}{g(x)} = \lim_{x \to a} \frac{f'(x)}{g'(x)}
$$

Utilisez SymPy pour vérifier la règle de L'Hôpital pour les fonctions suivantes

$$
f(x) = \frac{y^x - 1}{x}
$$

lorsque $x$ tend vers $0$
```

```{solution-start} sympy_ex1
:class: dropdown
```

Définissons d'abord la fonction

```{code-cell} ipython3
f_upper = y**x - 1
f_lower = x
f = f_upper/f_lower
f
```

Sympy est suffisamment intelligent pour résoudre cette limite

```{code-cell} ipython3
lim = limit(f, x, 0)
lim
```

Nous comparons le résultat suggéré par la règle de L'Hôpital

```{code-cell} ipython3
lim = limit(diff(f_upper, x)/
            diff(f_lower, x), x, 0)
lim
```

```{solution-end}
```

```{exercise}
:label: sympy_ex2

L'[estimation par maximum de vraisemblance (MLE)](https://python.quantecon.org/mle.html) est une méthode pour estimer les paramètres d'un modèle statistique.

Elle implique généralement de maximiser une fonction de log-vraisemblance et de résoudre la dérivée du premier ordre.

La loi binomiale est donnée par

$$
f(x; n, θ) = \frac{n!}{x!(n-x)!}θ^x(1-θ)^{n-x}
$$

où $n$ est le nombre d'essais et $x$ est le nombre de succès.

Supposons que nous avons observé une série de résultats binaires avec $x$ succès sur $n$ essais.

Calculez le MLE de $θ$ en utilisant SymPy
```

```{solution-start} sympy_ex2
:class: dropdown
```

D'abord, nous définissons la loi binomiale

```{code-cell} ipython3
n, x, θ = symbols('n x θ')

binomial_factor = (factorial(n)) / (factorial(x)*factorial(n-r))
binomial_factor
```

```{code-cell} ipython3
bino_dist = binomial_factor * ((θ**x)*(1-θ)**(n-x))
bino_dist
```

Maintenant nous calculons la fonction de log-vraisemblance et résolvons pour le résultat

```{code-cell} ipython3
log_bino_dist = log(bino_dist)
```

```{code-cell} ipython3
log_bino_diff = simplify(diff(log_bino_dist, θ))
log_bino_diff
```

```{code-cell} ipython3
solve(Eq(log_bino_diff, 0), θ)[0]
```

```{solution-end}
```