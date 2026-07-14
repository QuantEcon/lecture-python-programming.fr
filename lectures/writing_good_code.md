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
  title: Écrire du bon code
  headings:
    Overview: Vue d'ensemble
    An Example of Poor Code: Un exemple de code médiocre
    Good Coding Practice: Bonnes pratiques de codage
    Good Coding Practice::Don't Use Magic Numbers: N'utilisez pas de nombres magiques
    Good Coding Practice::Don't Repeat Yourself: Ne vous répétez pas
    Good Coding Practice::Minimize Global Variables: Minimisez les variables globales
    Good Coding Practice::Minimize Global Variables::JIT Compilation: Compilation JIT
    Good Coding Practice::Use Functions or Classes: Utilisez des fonctions ou des classes
    Good Coding Practice::Use Functions or Classes::Which One, Functions or Classes?: "Lesquelles, des fonctions ou des classes\_?"
    Revisiting the Example: Retour sur l'exemple
    Exercises: Exercices
---

(writing_good_code)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Écrire du bon code

```{index} single: Models; Code style
```

```{epigraph}
« N'importe quel idiot peut écrire du code qu'un ordinateur peut comprendre. Les bons programmeurs écrivent du code que les humains peuvent comprendre. » -- Martin Fowler
```


## Vue d'ensemble

Lorsque les programmes informatiques sont petits, un code mal écrit n'est pas trop coûteux.

Mais davantage de données, des modèles plus sophistiqués et une plus grande puissance de calcul nous permettent de nous attaquer à des problèmes plus difficiles impliquant l'écriture de programmes plus longs.

Pour de tels programmes, un investissement dans de bonnes pratiques de codage rapportera de forts dividendes.

Les principaux avantages sont une productivité accrue et un code plus rapide.

Dans ce cours, nous passons en revue quelques éléments des bonnes pratiques de codage.

Nous abordons également des développements modernes en calcul scientifique --- comme la compilation à la volée (just in time) --- et la manière dont ils affectent la conception d'un bon programme.

## Un exemple de code médiocre

Examinons du code mal écrit.

Le rôle de ce code est de générer et de tracer des séries temporelles du modèle de Solow simplifié

```{math}
:label: gc_solmod

k_{t+1} = s k_t^{\alpha} + (1 - \delta) k_t,
\quad t = 0, 1, 2, \ldots
```

Ici

* $k_t$ est le capital à l'instant $t$ et
* $s, \alpha, \delta$ sont des paramètres (épargne, un paramètre de productivité et dépréciation)

Pour chaque paramétrage, le code

1. fixe $k_0 = 1$
1. itère en utilisant {eq}`gc_solmod` pour produire une séquence $k_0, k_1, k_2 \ldots , k_T$
1. trace la séquence

Les tracés seront regroupés en trois sous-figures.

Dans chaque sous-figure, deux paramètres sont maintenus fixes tandis qu'un autre varie

```{code-cell} ipython
import numpy as np
import matplotlib.pyplot as plt

# Allouer la mémoire pour les séries temporelles
k = np.empty(50)

fig, axes = plt.subplots(3, 1, figsize=(8, 16))

# Trajectoires avec différents α
δ = 0.1
s = 0.4
α = (0.25, 0.33, 0.45)

for j in range(3):
    k[0] = 1
    for t in range(49):
        k[t+1] = s * k[t]**α[j] + (1 - δ) * k[t]
    axes[0].plot(k, 'o-', label=rf"$\alpha = {α[j]},\; s = {s},\; \delta={δ}$")

axes[0].grid(lw=0.2)
axes[0].set_ylim(0, 18)
axes[0].set_xlabel('temps')
axes[0].set_ylabel('capital')
axes[0].legend(loc='upper left', frameon=True)

# Trajectoires avec différents s
δ = 0.1
α = 0.33
s = (0.3, 0.4, 0.5)

for j in range(3):
    k[0] = 1
    for t in range(49):
        k[t+1] = s[j] * k[t]**α + (1 - δ) * k[t]
    axes[1].plot(k, 'o-', label=rf"$\alpha = {α},\; s = {s[j]},\; \delta={δ}$")

axes[1].grid(lw=0.2)
axes[1].set_xlabel('temps')
axes[1].set_ylabel('capital')
axes[1].set_ylim(0, 18)
axes[1].legend(loc='upper left', frameon=True)

# Trajectoires avec différents δ
δ = (0.05, 0.1, 0.15)
α = 0.33
s = 0.4

for j in range(3):
    k[0] = 1
    for t in range(49):
        k[t+1] = s * k[t]**α + (1 - δ[j]) * k[t]
    axes[2].plot(k, 'o-', label=rf"$\alpha = {α},\; s = {s},\; \delta={δ[j]}$")

axes[2].set_ylim(0, 18)
axes[2].set_xlabel('temps')
axes[2].set_ylabel('capital')
axes[2].grid(lw=0.2)
axes[2].legend(loc='upper left', frameon=True)

plt.show()
```

Certes, le code suit plus ou moins la norme [PEP8](https://peps.python.org/pep-0008/).

En même temps, il est très mal structuré.

Voyons pourquoi c'est le cas, et ce que nous pouvons y faire.

## Bonnes pratiques de codage

Il existe généralement de nombreuses façons différentes d'écrire un programme qui accomplit une tâche donnée.

Pour les petits programmes, comme celui ci-dessus, la façon dont vous écrivez le code n'a pas trop d'importance.

Mais si vous êtes ambitieux et voulez produire des choses utiles, vous écrirez aussi des programmes de taille moyenne à grande.

Dans ces contextes, le style de codage compte **énormément**.

Heureusement, beaucoup de gens intelligents ont réfléchi à la meilleure façon d'écrire du code.

Voici quelques préceptes de base.

### N'utilisez pas de nombres magiques

Si vous regardez le code ci-dessus, vous verrez des nombres comme `50`, `49` et `3` dispersés à travers le code.

Ces types de littéraux numériques dans le corps de votre code sont parfois appelés « nombres magiques ».

Ce n'est pas un compliment.

Bien que les littéraux numériques ne soient pas tous à proscrire, les nombres montrés dans le programme ci-dessus devraient assurément être remplacés par des constantes nommées.

Par exemple, le code ci-dessus pourrait déclarer la variable `time_series_length = 50`.

Ensuite, dans les boucles, `49` devrait être remplacé par `time_series_length - 1`.

Les avantages sont :

* la signification est beaucoup plus claire partout
* pour modifier la longueur des séries temporelles, il ne faut changer qu'une seule valeur

### Ne vous répétez pas

L'autre péché mortel dans l'extrait de code ci-dessus est la répétition.

Des blocs de logique (comme la boucle pour générer les séries temporelles) sont répétés avec seulement de légères modifications.

Cela viole un principe fondamental de la programmation : Ne vous répétez pas (DRY, pour « Don't repeat yourself »).

* Aussi appelé DIE (la duplication est le mal, « duplication is evil »).

Oui, nous savons que vous pouvez simplement copier-coller et changer quelques symboles.

Mais en tant que programmeur, votre objectif devrait être d'**automatiser** la répétition, **et non** de la faire vous-même.

Plus important encore, répéter la même logique à différents endroits signifie qu'à terme, l'un d'eux sera probablement erroné.

Si vous voulez en savoir plus, lisez l'excellent résumé qui se trouve sur [cette page](https://code.tutsplus.com/3-key-software-principles-you-must-understand--net-25161t).

Nous verrons ci-dessous comment éviter la répétition.

### Minimisez les variables globales

Certes, les variables globales (c'est-à-dire les noms attribués à des valeurs en dehors de toute fonction ou classe) sont pratiques.

Les programmeurs débutants utilisent généralement les variables globales sans retenue --- comme nous l'avons nous-mêmes fait autrefois.

Mais les variables globales sont dangereuses, en particulier dans les programmes de taille moyenne à grande, car

* elles peuvent affecter ce qui se passe dans n'importe quelle partie de votre programme
* elles peuvent être modifiées par n'importe quelle fonction

Cela rend beaucoup plus difficile d'être certain de ce qu'une petite partie d'un morceau de code donné commande réellement.

Voici une [discussion utile sur le sujet](https://wiki.c2.com/?GlobalVariablesAreBad).

Bien qu'une variable globale occasionnelle dans de petits scripts ne soit pas un gros problème, nous vous recommandons de vous entraîner à les éviter.

(Nous verrons comment juste en dessous).

#### Compilation JIT

Pour le calcul scientifique, il y a une autre bonne raison d'éviter les variables globales.

Comme {doc}`nous l'avons vu dans des cours précédents <numba>`, la compilation JIT peut générer d'excellentes performances pour les langages de script comme Python.

Mais la tâche du compilateur utilisé pour la compilation JIT devient plus difficile lorsque des variables globales sont présentes.

Autrement dit, l'inférence de type requise pour la compilation JIT est plus sûre et plus efficace lorsque les variables sont isolées à l'intérieur d'une fonction.

### Utilisez des fonctions ou des classes

Heureusement, nous pouvons facilement éviter les maux des variables globales et du code WET.

* WET signifie « we enjoy typing » (nous aimons taper) et est l'opposé de DRY.

Nous pouvons le faire en utilisant fréquemment des fonctions ou des classes.

En fait, les fonctions et les classes sont conçues spécifiquement pour nous aider à éviter de nous humilier en répétant du code ou en utilisant excessivement des variables globales.

#### Lesquelles, des fonctions ou des classes ?

Les deux peuvent être utiles, et en fait elles fonctionnent bien ensemble.

Nous en apprendrons davantage sur ces sujets au fil du temps.

(La préférence personnelle fait aussi partie de l'histoire)

Ce qui est vraiment important, c'est que vous utilisiez l'une ou l'autre, ou les deux.

## Retour sur l'exemple

Voici du code qui reproduit le tracé ci-dessus avec un meilleur style de codage.

```{code-cell} python3
from itertools import product

def plot_path(ax, αs, s_vals, δs, time_series_length=50):
    """
    Add a time series plot to the axes ax for all given parameters.
    """
    k = np.empty(time_series_length)

    for (α, s, δ) in product(αs, s_vals, δs):
        k[0] = 1
        for t in range(time_series_length-1):
            k[t+1] = s * k[t]**α + (1 - δ) * k[t]
        ax.plot(k, 'o-', label=rf"$\alpha = {α},\; s = {s},\; \delta = {δ}$")

    ax.set_xlabel('temps')
    ax.set_ylabel('capital')
    ax.set_ylim(0, 18)
    ax.legend(loc='upper left', frameon=True)

fig, axes = plt.subplots(3, 1, figsize=(8, 16))

# Paramètres (αs, s_vals, δs)
set_one = ([0.25, 0.33, 0.45], [0.4], [0.1])
set_two = ([0.33], [0.3, 0.4, 0.5], [0.1])
set_three = ([0.33], [0.4], [0.05, 0.1, 0.15])

for (ax, params) in zip(axes, (set_one, set_two, set_three)):
    αs, s_vals, δs = params
    plot_path(ax, αs, s_vals, δs)

plt.show()
```

Si vous examinez ce code, vous verrez que

* il utilise une fonction pour éviter la répétition.
* Les variables globales sont mises en quarantaine en les regroupant à la fin, et non au début du programme.
* Les nombres magiques sont évités.
* La boucle à la fin, où le travail réel est effectué, est courte et relativement simple.

## Exercices

```{exercise-start}
:label: wgc-exercise-1
```

Voici du code qui a besoin d'être amélioré.

Il concerne un problème simple d'offre et de demande.

L'offre est donnée par

$$
q_s(p) = \exp(\alpha p) - \beta.
$$

La courbe de demande est

$$
q_d(p) = \gamma p^{-\delta}.
$$

Les valeurs $\alpha$, $\beta$, $\gamma$ et
$\delta$ sont des **paramètres**

L'équilibre $p^*$ est le prix tel que
$q_d(p) = q_s(p)$.

Nous pouvons résoudre cet équilibre à l'aide d'un algorithme de recherche de racine.
Plus précisément, nous trouverons le $p$ tel que $h(p) = 0$,
où

$$
h(p) := q_d(p) - q_s(p)
$$

Cela donne le prix d'équilibre $p^*$. À partir de là, nous obtenons la
quantité d'équilibre par $q^* = q_s(p^*)$

Les valeurs des paramètres seront

- $\alpha = 0.1$
- $\beta = 1$
- $\gamma = 1$
- $\delta = 1$

```{code-cell} ipython3
from scipy.optimize import brentq

# Calculer l'équilibre
def h(p):
    return p**(-1) - (np.exp(0.1 * p) - 1)  # demande - offre

p_star = brentq(h, 2, 4)
q_star = np.exp(0.1 * p_star) - 1

print(f'Le prix d\'équilibre est {p_star: .2f}')
print(f'La quantité d\'équilibre est {q_star: .2f}')
```

Traçons également nos résultats.

```{code-cell} ipython3
# Maintenant, tracer
grid = np.linspace(2, 4, 100)
fig, ax = plt.subplots()

qs = np.exp(0.1 * grid) - 1
qd = grid**(-1)


ax.plot(grid, qd, 'b-', lw=2, label='demande')
ax.plot(grid, qs, 'g-', lw=2, label='offre')

ax.set_xlabel('prix')
ax.set_ylabel('quantité')
ax.legend(loc='upper center')

plt.show()
```

Nous voulons également considérer les déplacements de l'offre et de la demande.

Par exemple, voyons ce qui se passe lorsque la demande se déplace vers le haut, avec $\gamma$ augmentant à $1.25$ :

```{code-cell} ipython3
# Calculer l'équilibre
def h(p):
    return 1.25 * p**(-1) - (np.exp(0.1 * p) - 1)

p_star = brentq(h, 2, 4)
q_star = np.exp(0.1 * p_star) - 1

print(f'Le prix d\'équilibre est {p_star: .2f}')
print(f'La quantité d\'équilibre est {q_star: .2f}')
```

```{code-cell} ipython3
# Maintenant, tracer
p_grid = np.linspace(2, 4, 100)
fig, ax = plt.subplots()

qs = np.exp(0.1 * p_grid) - 1
qd = 1.25 * p_grid**(-1)


ax.plot(grid, qd, 'b-', lw=2, label='demande')
ax.plot(grid, qs, 'g-', lw=2, label='offre')

ax.set_xlabel('prix')
ax.set_ylabel('quantité')
ax.legend(loc='upper center')

plt.show()
```

Maintenant, nous pourrions considérer les déplacements de l'offre, mais vous saisissez déjà l'idée qu'il y a
beaucoup de code répété ici.

Réorganisez et améliorez la clarté du code ci-dessus en utilisant les principes discutés
dans ce cours.

```{exercise-end}
```

```{solution-start} wgc-exercise-1
:class: dropdown
```

Voici une solution, qui utilise une classe :

```{code-cell} ipython3
class Equilibrium:

    def __init__(self, α=0.1, β=1, γ=1, δ=1):
        self.α, self.β, self.γ, self.δ = α, β, γ, δ

    def qs(self, p):
        return np.exp(self.α * p) - self.β

    def qd(self, p):
        return self.γ * p**(-self.δ)

    def compute_equilibrium(self):
        def h(p):
            return self.qd(p) - self.qs(p)
        p_star = brentq(h, 2, 4)
        q_star = np.exp(self.α * p_star) - self.β

        print(f'Le prix d\'équilibre est {p_star: .2f}')
        print(f'La quantité d\'équilibre est {q_star: .2f}')

    def plot_equilibrium(self):
        # Maintenant, tracer
        grid = np.linspace(2, 4, 100)
        fig, ax = plt.subplots()

        ax.plot(grid, self.qd(grid), 'b-', lw=2, label='demande')
        ax.plot(grid, self.qs(grid), 'g-', lw=2, label='offre')

        ax.set_xlabel('prix')
        ax.set_ylabel('quantité')
        ax.legend(loc='upper center')

        plt.show()
```

Créons une instance avec les valeurs par défaut des paramètres.

```{code-cell} ipython3
eq = Equilibrium()
```

Maintenant, nous allons calculer l'équilibre et le tracer.

```{code-cell} ipython3
eq.compute_equilibrium()
```

```{code-cell} ipython3
eq.plot_equilibrium()
```

L'une des belles choses à propos de notre code réorganisé est que, lorsque nous changeons
les paramètres, nous n'avons pas besoin de nous répéter :

```{code-cell} ipython3
eq.γ = 1.25
```

```{code-cell} ipython3
eq.compute_equilibrium()
```

```{code-cell} ipython3
eq.plot_equilibrium()
```

```{solution-end}
```