---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.17.2
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
translation:
  title: Aventures avec la différentiation automatique
  headings:
    Overview: Vue d'ensemble
    What is automatic differentiation?: "Qu'est-ce que la différentiation automatique\_?"
    What is automatic differentiation?::Autodiff is not finite differences: La différentiation automatique n'est pas la différence finie
    What is automatic differentiation?::Autodiff is not symbolic calculus: La différentiation automatique n'est pas le calcul symbolique
    What is automatic differentiation?::Autodiff: La différentiation automatique
    Some experiments: Quelques expériences
    Some experiments::A differentiable function: Une fonction différentiable
    Some experiments::Absolute value function: Fonction valeur absolue
    Some experiments::Differentiating through control flow: Différentier à travers le flux de contrôle
    Some experiments::Differentiating through a linear interpolation: Différentier à travers une interpolation linéaire
    Gradient Descent: Descente de gradient
    Gradient Descent::A function for gradient descent: Une fonction pour la descente de gradient
    Gradient Descent::Simulated data: Données simulées
    Gradient Descent::Minimizing squared loss by gradient descent: Minimiser la perte quadratique par descente de gradient
    Gradient Descent::Adding a squared term: Ajout d'un terme quadratique
    Exercises: Exercices
---

# Aventures avec la différentiation automatique


```{include} _admonition/gpu.md
```

## Vue d'ensemble

Ce cours propose une introduction plus approfondie à la différentiation automatique
à l'aide de Google JAX, en s'appuyant sur {doc}`notre bref aperçu <jax_intro>`.

La différentiation automatique est l'un des éléments clés de l'apprentissage automatique
et de l'intelligence artificielle modernes.

À ce titre, elle a attiré de nombreux investissements et il existe plusieurs
implémentations puissantes.

L'une des meilleures est constituée par les routines de différentiation automatique
contenues dans JAX.

Bien que d'autres bibliothèques logicielles offrent également cette fonctionnalité, la version de JAX est
particulièrement puissante car elle s'intègre très bien avec les autres composants
essentiels de JAX (par exemple, la compilation JIT et la parallélisation).

La différentiation automatique peut être utilisée non seulement
pour l'IA, mais aussi pour de nombreux problèmes rencontrés en modélisation mathématique, tels que
les problèmes d'optimisation non linéaire multidimensionnelle et de recherche de racines.

En plus de ce qui est inclus dans Anaconda, ce cours nécessitera les bibliothèques suivantes :

```{code-cell} ipython3
:tags: [hide-output]

!pip install jax
```

Nous avons besoin des importations suivantes

```{code-cell} ipython3
import jax
import jax.numpy as jnp
import matplotlib.pyplot as plt
import numpy as np
from sympy import symbols
```

## Qu'est-ce que la différentiation automatique ?

La différentiation automatique (autodiff) est une technique de calcul des dérivées sur un ordinateur.

### La différentiation automatique n'est pas la différence finie

La dérivée de $f(x) = \exp(2x)$ est

$$
    f'(x) = 2 \exp(2x)
$$



Un ordinateur qui ne sait pas comment calculer des dérivées pourrait approximer cela par le rapport de différence finie

$$
    (Df)(x) := \frac{f(x+h) - f(x)}{h}
$$

où $h$ est un petit nombre positif.

```{code-cell} ipython3
def f(x):
    "Fonction originale."
    return np.exp(2 * x)

def f_prime(x):
    "Dérivée exacte."
    return 2 * np.exp(2 * x)

def Df(x, h=0.1):
    "Dérivée approchée (différence finie)."
    return (f(x + h) - f(x))/h

x_grid = np.linspace(-2, 1, 200)
fig, ax = plt.subplots()
ax.plot(x_grid, f_prime(x_grid), label="$f'$")
ax.plot(x_grid, Df(x_grid), label="$Df$")
ax.legend()
plt.show()
```

Ce type de dérivée numérique est souvent imprécis et instable.

L'une des raisons est que

$$
    \frac{f(x+h) - f(x)}{h} \approx \frac{0}{0}
$$

Les petits nombres au numérateur et au dénominateur provoquent des erreurs d'arrondi.

La situation est exponentiellement pire en grande dimension / avec des dérivées d'ordre supérieur.

+++

### La différentiation automatique n'est pas le calcul symbolique

+++

Le calcul symbolique tente d'utiliser les règles de différentiation pour produire une unique
expression sous forme close représentant une dérivée.

```{code-cell} ipython3
m, a, b, x = symbols('m a b x')
f_x = (a*x + b)**m
f_x.diff((x, 6))  # dérivée d'ordre 6
```

Le calcul symbolique n'est pas bien adapté au calcul
haute performance.

L'un de ses inconvénients est que le calcul symbolique ne peut pas différentier à travers le flux de contrôle.

De plus, l'utilisation du calcul symbolique peut impliquer des calculs redondants.

Par exemple, considérons

$$
    (f g h)'
    = (f' g + g' f) h + (f g) h'
$$

Si nous évaluons en $x$, alors nous évaluons $f(x)$ et $g(x)$ deux fois chacun.

De plus, le calcul de $f'(x)$ et de $f(x)$ peut impliquer des termes similaires (par exemple, $f(x) = \exp(2x) \implies f'(x) = 2f(x)$) mais cela n'est pas exploité en algèbre symbolique.

+++

### La différentiation automatique

La différentiation automatique produit des fonctions qui évaluent les dérivées à des valeurs numériques
transmises par le code appelant, plutôt que de produire une unique expression symbolique
représentant l'ensemble de la dérivée.

Les dérivées sont construites en décomposant les calculs en éléments constitutifs via la règle de dérivation en chaîne.

La règle de dérivation en chaîne est appliquée jusqu'au point où les termes se réduisent à des fonctions primitives que le programme sait différentier exactement (addition, soustraction, exponentiation, sinus et cosinus, etc.)

+++

## Quelques expériences

+++

Commençons par quelques fonctions à valeurs réelles sur $\mathbb R$.

+++

### Une fonction différentiable

+++

Testons la différentiation automatique de JAX avec une fonction relativement simple.

```{code-cell} ipython3
def f(x):
    return jnp.sin(x) - 2 * jnp.cos(3 * x) * jnp.exp(- x**2)
```

Nous utilisons `grad` pour calculer le gradient d'une fonction à valeurs réelles :

```{code-cell} ipython3
f_prime = jax.grad(f)
```

Traçons le résultat :

```{code-cell} ipython3
x_grid = jnp.linspace(-5, 5, 100)
```

```{code-cell} ipython3
fig, ax = plt.subplots()
ax.plot(x_grid, [f(x) for x in x_grid], label="$f$")
ax.plot(x_grid, [f_prime(x) for x in x_grid], label="$f'$")
ax.legend()
plt.show()
```

### Fonction valeur absolue

+++

Que se passe-t-il si la fonction n'est pas différentiable ?

```{code-cell} ipython3
def f(x):
    return jnp.abs(x)
```

```{code-cell} ipython3
f_prime = jax.grad(f)
```

```{code-cell} ipython3
fig, ax = plt.subplots()
ax.plot(x_grid, [f(x) for x in x_grid], label="$f$")
ax.plot(x_grid, [f_prime(x) for x in x_grid], label="$f'$")
ax.legend()
plt.show()
```

Au point non différentiable $0$, `jax.grad` renvoie la dérivée à droite :

```{code-cell} ipython3
f_prime(0.0)
```

### Différentier à travers le flux de contrôle

+++

Essayons de différentier à travers quelques boucles et conditions.

```{code-cell} ipython3
def f(x):
    def f1(x):
        for i in range(2):
            x *= 0.2 * x
        return x
    def f2(x):
        x = sum((x**i + i) for i in range(3))
        return x
    y = f1(x) if x < 0 else f2(x)
    return y
```

```{code-cell} ipython3
f_prime = jax.grad(f)
```

```{code-cell} ipython3
x_grid = jnp.linspace(-5, 5, 100)
```

```{code-cell} ipython3
fig, ax = plt.subplots()
ax.plot(x_grid, [f(x) for x in x_grid], label="$f$")
ax.plot(x_grid, [f_prime(x) for x in x_grid], label="$f'$")
ax.legend()
plt.show()
```

### Différentier à travers une interpolation linéaire

+++

Nous pouvons différentier à travers une interpolation linéaire, même si la fonction n'est pas lisse :

```{code-cell} ipython3
n = 20
xp = jnp.linspace(-5, 5, n)
yp = jnp.cos(2 * xp)

fig, ax = plt.subplots()
ax.plot(x_grid, jnp.interp(x_grid, xp, yp))
plt.show()
```

```{code-cell} ipython3
f_prime = jax.grad(jnp.interp)
```

```{code-cell} ipython3
f_prime_vec = jax.vmap(f_prime, in_axes=(0, None, None))
```

```{code-cell} ipython3
fig, ax = plt.subplots()
ax.plot(x_grid, f_prime_vec(x_grid, xp, yp))
plt.show()
```

## Descente de gradient

+++

Essayons d'implémenter la descente de gradient.

Comme application simple, nous utiliserons la descente de gradient pour résoudre les estimations des paramètres par MCO dans une régression linéaire simple.

+++

### Une fonction pour la descente de gradient

+++

Voici une implémentation de la descente de gradient.

```{code-cell} ipython3
def grad_descent(f,       # Fonction à minimiser
                 args,    # Arguments supplémentaires de la fonction
                 x0,      # Condition initiale
                 λ=0.1,   # Taux d'apprentissage initial
                 tol=1e-5, 
                 max_iter=1_000):
    """
    Minimise la fonction f par descente de gradient, en partant de l'estimation x0.

    Le taux d'apprentissage est calculé selon la méthode de Barzilai-Borwein.
    
    """
    
    f_grad = jax.grad(f)
    x = jnp.array(x0)
    df = f_grad(x, args)
    ϵ = tol + 1
    i = 0
    while ϵ > tol and i < max_iter:
        new_x = x - λ * df
        new_df = f_grad(new_x, args)
        Δx = new_x - x
        Δdf = new_df - df
        λ = jnp.abs(Δx @ Δdf) / (Δdf @ Δdf)
        ϵ = jnp.max(jnp.abs(Δx))
        x, df = new_x, new_df
        i += 1
        
    return x
    
```

### Données simulées

Nous allons tester notre fonction de descente de gradient en minimisant une somme des moindres carrés dans un problème de régression.

Générons quelques données simulées :

```{code-cell} ipython3
n = 100
key = jax.random.key(1234)
x = jax.random.uniform(key, (n,))

α, β, σ = 0.5, 1.0, 0.1  # Fixe la vraie ordonnée à l'origine et la vraie pente.
key, subkey = jax.random.split(key)
ϵ = jax.random.normal(subkey, (n,))

y = α * x + β + σ * ϵ
```

```{code-cell} ipython3
fig, ax = plt.subplots()
ax.scatter(x, y)
plt.show()
```

Commençons par calculer la pente et l'ordonnée à l'origine estimées à l'aide de solutions sous forme close.

```{code-cell} ipython3
mx = x.mean()
my = y.mean()
α_hat = jnp.sum((x - mx) * (y - my)) / jnp.sum((x - mx)**2)
β_hat = my - α_hat * mx
```

```{code-cell} ipython3
α_hat, β_hat
```

```{code-cell} ipython3
fig, ax = plt.subplots()
ax.scatter(x, y)
ax.plot(x, α_hat * x + β_hat, 'k-')
ax.text(0.1, 1.55, rf'$\hat \alpha = {α_hat:.3}$')
ax.text(0.1, 1.50, rf'$\hat \beta = {β_hat:.3}$')
plt.show()
```

### Minimiser la perte quadratique par descente de gradient

+++

Voyons si nous pouvons obtenir les mêmes valeurs avec notre fonction de descente de gradient.

Commençons par définir la fonction de perte des moindres carrés.

```{code-cell} ipython3
@jax.jit
def loss(params, data):
    a, b = params
    x, y = data
    return jnp.sum((y - a * x - b)**2)
```

Maintenant, nous la minimisons :

```{code-cell} ipython3
p0 = jnp.zeros(2)  # Estimation initiale pour α, β
data = x, y
α_hat, β_hat = grad_descent(loss, data, p0)
```

Traçons les résultats.

```{code-cell} ipython3
fig, ax = plt.subplots()
x_grid = jnp.linspace(0, 1, 100)
ax.scatter(x, y)
ax.plot(x_grid, α_hat * x_grid + β_hat, 'k-', alpha=0.6)
ax.text(0.1, 1.55, rf'$\hat \alpha = {α_hat:.3}$')
ax.text(0.1, 1.50, rf'$\hat \beta = {β_hat:.3}$')
plt.show()
```

Remarquez que nous obtenons les mêmes estimations que celles issues des solutions sous forme close.

+++

### Ajout d'un terme quadratique

Essayons maintenant d'ajuster un polynôme du second degré.

Voici notre nouvelle fonction de perte.

```{code-cell} ipython3
@jax.jit
def loss(params, data):
    a, b, c = params
    x, y = data
    return jnp.sum((y - a * x**2 - b * x - c)**2)
```

Nous minimisons maintenant en trois dimensions.

Essayons.

```{code-cell} ipython3
p0 = jnp.zeros(3)
α_hat, β_hat, γ_hat = grad_descent(loss, data, p0)

fig, ax = plt.subplots()
ax.scatter(x, y)
ax.plot(x_grid, α_hat * x_grid**2 + β_hat * x_grid + γ_hat, 'k-', alpha=0.6)
ax.text(0.1, 1.55, rf'$\hat \alpha = {α_hat:.3}$')
ax.text(0.1, 1.50, rf'$\hat \beta = {β_hat:.3}$')
plt.show()
```

## Exercices

```{exercise-start}
:label: auto_ex1
```

La fonction `jnp.polyval` évalue des polynômes.

Par exemple, si `len(p)` vaut 3, alors `jnp.polyval(p, x)` renvoie

$$
    f(p, x) := p_0 x^2 + p_1 x + p_2
$$

Utilisez cette fonction pour la régression polynomiale.

La perte (empirique) devient

$$
    \ell(p, x, y) 
    = \sum_{i=1}^n (y_i - f(p, x_i))^2
$$

Fixez $k=4$ et fixez l'estimation initiale de `params` à `jnp.zeros(k)`.

Utilisez la descente de gradient pour trouver le tableau `params` qui minimise la fonction
de perte et tracez le résultat (en suivant les exemples ci-dessus).


```{exercise-end}
```

```{solution-start} auto_ex1
:class: dropdown
```

Voici une solution.

```{code-cell} ipython3
def loss(params, data):
    x, y = data
    return jnp.sum((y - jnp.polyval(params, x))**2)
```

```{code-cell} ipython3
k = 4
p0 = jnp.zeros(k)
p_hat = grad_descent(loss, data, p0)
print('Vecteur de paramètres estimé :')
print(p_hat)
print('\n\n')

fig, ax = plt.subplots()
ax.scatter(x, y)
ax.plot(x_grid, jnp.polyval(p_hat, x_grid), 'k-', alpha=0.6)
plt.show()
```


```{solution-end}
```