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
  title: Fonctions
  headings:
    Overview: Vue d'ensemble
    Function Basics: Les bases des fonctions
    Function Basics::Built-In Functions: Fonctions intégrées
    Function Basics::Third Party Functions: Fonctions tierces
    Defining Functions: Définir des fonctions
    Defining Functions::Basic Syntax: Syntaxe de base
    Defining Functions::Keyword Arguments: Arguments par mot-clé
    Defining Functions::The Flexibility of Python Functions: La flexibilité des fonctions Python
    'Defining Functions::One-Line Functions: `lambda`': "Fonctions en une ligne\_: `lambda`"
    Defining Functions::Why Write Functions?: "Pourquoi écrire des fonctions\_?"
    Applications: Applications
    Applications::Random Draws: Tirages aléatoires
    Applications::Adding Conditions: Ajouter des conditions
    Recursive Function Calls (Advanced): Appels de fonction récursifs (Avancé)
    Exercises: Exercices
    Advanced Exercises: Exercices avancés
---

(functions)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Fonctions

```{index} single: Python; Fonctions définies par l'utilisateur
```

## Vue d'ensemble

Les fonctions sont une construction extrêmement utile fournie par presque tous les langages de programmation.

Nous avons déjà rencontré plusieurs fonctions, telles que

* la fonction `sqrt()` de NumPy et
* la fonction intégrée `print()`

Dans ce cours, nous allons

1. traiter les fonctions de manière systématique et couvrir la syntaxe et les cas d'usage, et
2. apprendre à construire nos propres fonctions définies par l'utilisateur.

Nous utiliserons les importations suivantes.

```{code-cell} ipython
import numpy as np
import matplotlib.pyplot as plt
```

## Les bases des fonctions

Une fonction est une section nommée d'un programme qui implémente une tâche spécifique.

De nombreuses fonctions existent déjà et nous pouvons les utiliser telles quelles.

Nous passerons d'abord en revue ces fonctions, puis nous discuterons de la façon dont nous pouvons construire les nôtres.

### Fonctions intégrées

Python possède un certain nombre de fonctions **intégrées** qui sont disponibles sans `import`.

Nous en avons déjà rencontré quelques-unes

```{code-cell} python3
max(19, 20)
```

```{code-cell} python3
print('foobar')
```

```{code-cell} python3
str(22)
```

```{code-cell} python3
type(22)
```

La liste complète des fonctions intégrées de Python se trouve [ici](https://docs.python.org/3/library/functions.html).


### Fonctions tierces

Si les fonctions intégrées ne couvrent pas nos besoins, nous devons soit importer
des fonctions, soit créer les nôtres.

Des exemples d'importation et d'utilisation de fonctions ont été donnés dans le {doc}`cours précédent <python_by_example>`

En voici un autre, qui teste si une année donnée est bissextile :

```{code-cell} python3
import calendar
calendar.isleap(2024)
```

## Définir des fonctions

Dans de nombreux cas, il est utile de pouvoir définir nos propres fonctions.

Commençons par discuter de la façon de procéder.

### Syntaxe de base

Voici une fonction Python très simple, qui implémente la fonction mathématique $f(x) = 2 x + 1$

```{code-cell} python3
def f(x):
    return 2 * x + 1
```

Maintenant que nous avons défini cette fonction, *appelons*-la et vérifions si elle fait ce que nous attendons :

```{code-cell} python3
f(1)   
```

```{code-cell} python3
f(10)
```

Voici une fonction plus longue, qui calcule la valeur absolue d'un nombre donné.

(Une telle fonction existe déjà en tant que fonction intégrée, mais écrivons la nôtre pour
l'exercice.)

```{code-cell} python3
def new_abs_function(x):
    if x < 0:
        abs_value = -x
    else:
        abs_value = x
    return abs_value
```

Passons en revue la syntaxe ici.

* `def` est un mot-clé Python utilisé pour commencer les définitions de fonctions.
* `def new_abs_function(x):` indique que la fonction s'appelle `new_abs_function` et qu'elle possède un seul argument `x`.
* Le code indenté est un bloc de code appelé le *corps de la fonction*.
* Le mot-clé `return` indique que `abs_value` est l'objet qui doit être renvoyé au code appelant.

Toute cette définition de fonction est lue par l'interpréteur Python et stockée en mémoire.

Appelons-la pour vérifier qu'elle fonctionne :

```{code-cell} python3
print(new_abs_function(3))
print(new_abs_function(-3))
```


Notez qu'une fonction peut avoir un nombre arbitraire d'instructions `return` (y compris zéro).

L'exécution de la fonction se termine dès que le premier `return` est atteint, ce qui permet
un code comme dans l'exemple suivant

```{code-cell} python3
def f(x):
    if x < 0:
        return 'negative'
    return 'nonnegative'
```

(Écrire des fonctions avec plusieurs instructions `return` est généralement déconseillé, car
cela peut rendre la logique difficile à suivre.)

Les fonctions sans instruction `return` renvoient automatiquement l'objet Python spécial `None`.

(pos_args)=
### Arguments par mot-clé

```{index} single: Python; arguments par mot-clé
```

Dans un {ref}`cours précédent <python_by_example>`, vous avez rencontré l'instruction

```{code-block} python3
:class: no-execute

plt.plot(x, 'b-', label="white noise")
```

Dans cet appel à la fonction `plot` de Matplotlib, remarquez que le dernier argument est passé avec la syntaxe `name=argument`.

C'est ce qu'on appelle un *argument par mot-clé*, avec `label` comme mot-clé.

Les arguments non-mot-clé sont appelés *arguments positionnels*, car leur signification
est déterminée par l'ordre

* `plot(x, 'b-')` diffère de `plot('b-', x)`

Les arguments par mot-clé sont particulièrement utiles lorsqu'une fonction possède beaucoup d'arguments, auquel cas il est difficile de se souvenir du bon ordre.

Vous pouvez adopter les arguments par mot-clé dans les fonctions définies par l'utilisateur sans aucune difficulté.

L'exemple suivant illustre la syntaxe

```{code-cell} python3
def f(x, a=1, b=1):
    return a + b * x
```

Les valeurs des arguments par mot-clé que nous avons fournies dans la définition de `f` deviennent les valeurs par défaut

```{code-cell} python3
f(2)
```

Elles peuvent être modifiées comme suit

```{code-cell} python3
f(2, a=4, b=5)
```

### La flexibilité des fonctions Python

Comme nous en avons discuté dans le {ref}`cours précédent <python_by_example>`, les fonctions Python sont très flexibles.

En particulier

* N'importe quel nombre de fonctions peut être défini dans un fichier donné.
* Les fonctions peuvent être (et sont souvent) définies à l'intérieur d'autres fonctions.
* N'importe quel objet peut être passé à une fonction comme argument, y compris d'autres fonctions.
* Une fonction peut renvoyer n'importe quel type d'objet, y compris des fonctions.

Nous donnerons des exemples montrant à quel point il est simple de passer une fonction à
une fonction dans les sections suivantes.

### Fonctions en une ligne : `lambda`

```{index} single: Python; fonctions lambda
```

Le mot-clé `lambda` est utilisé pour créer des fonctions simples sur une seule ligne.

Par exemple, les définitions

```{code-cell} python3
def f(x):
    return x**3
```

et

```{code-cell} python3
f = lambda x: x**3
```

sont entièrement équivalentes.

Pour comprendre pourquoi `lambda` est utile, supposons que nous voulions calculer $\int_0^2 x^3 dx$ (et que nous ayons oublié notre calcul infinitésimal du lycée).

La bibliothèque SciPy possède une fonction appelée `quad` qui effectuera ce calcul pour nous.

La syntaxe de la fonction `quad` est `quad(f, a, b)` où `f` est une fonction et `a` et `b` sont des nombres.

Pour créer la fonction $f(x) = x^3$ nous pouvons utiliser `lambda` comme suit

```{code-cell} python3
from scipy.integrate import quad

quad(lambda x: x**3, 0, 2)
```

Ici, la fonction créée par `lambda` est dite *anonyme* car elle n'a jamais reçu de nom.


### Pourquoi écrire des fonctions ?

Les fonctions définies par l'utilisateur sont importantes pour améliorer la clarté de votre code en

* séparant les différents fils de la logique
* facilitant la réutilisation du code

(Écrire deux fois la même chose est [presque toujours une mauvaise idée](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself))

Nous en dirons plus à ce sujet {doc}`plus tard <writing_good_code>`.

## Applications

### Tirages aléatoires

Considérons à nouveau ce code du {doc}`cours précédent <python_by_example>`

```{code-cell} python3
rng = np.random.default_rng()

ts_length = 100
ϵ_values = []   # liste vide

for i in range(ts_length):
    e = rng.standard_normal()
    ϵ_values.append(e)

plt.plot(ϵ_values)
plt.show()
```

Nous allons décomposer ce programme en deux parties :

1. Une fonction définie par l'utilisateur qui génère une liste de variables aléatoires.
1. La partie principale du programme qui
    1. appelle cette fonction pour obtenir des données
    1. trace les données

Ceci est réalisé dans le programme suivant

(funcloopprog)=
```{code-cell} python3
def generate_data(n):
    ϵ_values = []
    for i in range(n):
        e = rng.standard_normal()
        ϵ_values.append(e)
    return ϵ_values

data = generate_data(100)
plt.plot(data)
plt.show()
```

Lorsque l'interpréteur atteint l'expression `generate_data(100)`, il exécute le corps de la fonction avec `n` fixé à 100.

Le résultat net est que le nom `data` est *lié* à la liste `ϵ_values` renvoyée par la fonction.

### Ajouter des conditions

```{index} single: Python; Conditions
```

Notre fonction `generate_data()` est plutôt limitée.

Rendons-la légèrement plus utile en lui donnant la capacité de renvoyer soit des lois normales centrées réduites, soit des variables aléatoires uniformes sur $(0, 1)$ selon les besoins.

Ceci est réalisé dans le morceau de code suivant.

(funcloopprog2)=
```{code-cell} python3
def generate_data(n, generator_type):
    ϵ_values = []
    for i in range(n):
        if generator_type == 'U':
            e = rng.uniform(0, 1)
        else:
            e = rng.standard_normal()
        ϵ_values.append(e)
    return ϵ_values

data = generate_data(100, 'U')
plt.plot(data)
plt.show()
```

Espérons que la syntaxe de la clause if/else soit explicite, l'indentation délimitant à nouveau l'étendue des blocs de code.

Notes

* Nous passons l'argument `U` sous forme de chaîne de caractères, c'est pourquoi nous l'écrivons `'U'`.
* Remarquez que l'égalité est testée avec la syntaxe `==`, et non `=`.
    * Par exemple, l'instruction `a = 10` assigne le nom `a` à la valeur `10`.
    * L'expression `a == 10` s'évalue à `True` ou `False`, selon la valeur de `a`.

Maintenant, il existe plusieurs façons de simplifier le code ci-dessus.

Par exemple, nous pouvons nous débarrasser complètement des conditions en passant simplement le type de générateur souhaité sous forme de fonction, de méthode, ou d'un autre objet [appelable](https://typing.python.org/en/latest/spec/callables.html).

Pour comprendre cela, considérons la version suivante.

(test_program_6)=
```{code-cell} python3
def generate_data(n, generator_type):
    ϵ_values = []
    for i in range(n):
        e = generator_type()
        ϵ_values.append(e)
    return ϵ_values

data = generate_data(100, rng.uniform)
plt.plot(data)
plt.show()
```

Maintenant, lorsque nous appelons la fonction `generate_data()`, nous passons `rng.uniform`
comme deuxième argument.

Cet objet est un *appelable* — c'est-à-dire un objet qui peut être appelé en utilisant des parenthèses.

Lorsque l'appel de fonction `generate_data(100, rng.uniform)` est exécuté, Python exécute le bloc de code de la fonction avec `n` égal à 100 et le nom `generator_type` « lié » à l'appelable `rng.uniform`.

* Pendant l'exécution de ces lignes, les noms `generator_type` et `rng.uniform` sont des « synonymes », et peuvent être utilisés de manière identique.

Ce principe fonctionne de manière plus générale — par exemple, considérons le morceau de code suivant

```{code-cell} python3
max(7, 2, 4)   # max() est une fonction Python intégrée
```

```{code-cell} python3
m = max
m(7, 2, 4)
```

Ici, nous avons créé un autre nom pour la fonction intégrée `max()`, qui pouvait
ensuite être utilisé de manière identique.

Dans le contexte de notre programme, la capacité de lier des noms à des fonctions, ou plus généralement à des objets appelables, signifie qu'il n'y a aucun problème à passer un objet appelable comme argument à un autre appelable — comme nous l'avons fait avec `rng.uniform` ci-dessus.


(recursive_functions)=
## Appels de fonction récursifs (Avancé)

```{index} single: Python; Récursion
```

Ceci est un sujet avancé que vous pouvez ignorer sans problème.

En même temps, c'est une idée élégante que vous devriez apprendre à un moment ou à un autre de
votre carrière de programmeur.

Fondamentalement, une fonction récursive est une fonction qui s'appelle elle-même.

Par exemple, considérons le problème du calcul de $x_t$ pour un certain t lorsque

```{math}
:label: xseqdoub

x_{t+1} = 2 x_t, \quad x_0 = 1
```

De toute évidence, la réponse est $2^t$.

Nous pouvons calculer cela assez facilement avec une boucle

```{code-cell} python3
def x_loop(t):
    x = 1
    for i in range(t):
        x = 2 * x
    return x
```

Nous pouvons aussi utiliser une solution récursive, comme suit

```{code-cell} python3
def x(t):
    if t == 0:
        return 1
    else:
        return 2 * x(t-1)
```

Ce qui se passe ici, c'est que chaque appel successif utilise son propre *cadre* (*frame*) dans la *pile* (*stack*)

* un cadre est l'endroit où sont conservées les variables locales d'un appel de fonction donné
* la pile est la mémoire utilisée pour traiter les appels de fonction
  * une file d'attente First In Last Out (FILO)

Cet exemple est quelque peu artificiel, car la première solution (itérative) serait généralement préférée à la solution récursive.

Nous rencontrerons plus tard des applications de la récursion moins artificielles.


(factorial_exercise)=
## Exercices

```{exercise-start}
:label: func_ex1
```

Rappelez-vous que $n!$ se lit « $n$ factorielle » et est défini comme
$n! = n \times (n - 1) \times \cdots \times 2 \times 1$.

Nous ne considérerons ici $n$ que comme un entier positif.

Il existe des fonctions pour calculer cela dans divers modules, mais
écrivons notre propre version en guise d'exercice.

En particulier, écrivez une fonction `factorial` telle que `factorial(n)` renvoie $n!$
pour tout entier positif $n$.

```{exercise-end}
```


```{solution-start} func_ex1
:class: dropdown
```

Voici une solution :

```{code-cell} python3
def factorial(n):
    k = 1
    for i in range(n):
        k = k * (i + 1)
    return k

factorial(4)
```


```{solution-end}
```


```{exercise-start}
:label: func_ex2
```

La [variable aléatoire binomiale](https://en.wikipedia.org/wiki/Binomial_distribution) $Y \sim Bin(n, p)$ représente le nombre de succès dans $n$ essais binaires, où chaque essai réussit avec probabilité $p$.

En utilisant `rng = np.random.default_rng()`, écrivez une fonction
`binomial_rv` telle que `binomial_rv(n, p)` génère un tirage de $Y$.

```{hint}
:class: dropdown

Si $U$ est uniforme sur $(0, 1)$ et $p \in (0,1)$, alors l'expression `U < p` s'évalue à `True` avec probabilité $p$.
```

```{exercise-end}
```


```{solution-start} func_ex2
:class: dropdown
```

Voici une solution :

```{code-cell} python3
rng = np.random.default_rng()

def binomial_rv(n, p):
    count = 0
    for i in range(n):
        U = rng.uniform()
        if U < p:
            count = count + 1    # Ou count += 1
    return count

binomial_rv(10, 0.5)
```

```{solution-end}
```


```{exercise-start}
:label: func_ex3
```

D'abord, écrivez une fonction qui renvoie une réalisation du dispositif aléatoire suivant

1. Lancez une pièce non biaisée 10 fois.
1. Si une face (pile) apparaît `k` fois ou plus consécutivement dans cette séquence au moins une fois, payez un dollar.
1. Sinon, ne payez rien.

Ensuite, écrivez une autre fonction qui effectue la même tâche sauf que la deuxième règle du dispositif aléatoire ci-dessus devient

- Si une face (pile) apparaît `k` fois ou plus dans cette séquence, payez un dollar.

Utilisez `rng = np.random.default_rng()` pour générer des nombres aléatoires.

```{exercise-end}
```

```{solution-start} func_ex3
:class: dropdown
```

Voici une fonction pour le premier dispositif aléatoire.




```{code-cell} python3
rng = np.random.default_rng()

def draw(k):  # paie si k succès consécutifs dans une séquence

    payoff = 0
    count = 0

    for i in range(10):
        U = rng.uniform()
        count = count + 1 if U < 0.5 else 0
        print(count)    # affiche les comptes pour plus de clarté
        if count == k:
            payoff = 1

    return payoff

draw(3)
```

Voici une autre fonction pour le deuxième dispositif aléatoire.

```{code-cell} python3
def draw_new(k):  # paie si k succès dans une séquence

    payoff = 0
    count = 0

    for i in range(10):
        U = rng.uniform()
        count = count + ( 1 if U < 0.5 else 0 )
        print(count)
        if count == k:
            payoff = 1

    return payoff

draw_new(3)
```

```{solution-end}
```


## Exercices avancés

Dans les exercices suivants, nous écrirons ensemble des fonctions récursives.


```{exercise-start}
:label: func_ex4
```

Les nombres de Fibonacci sont définis par

```{math}
:label: fib

x_{t+1} = x_t + x_{t-1}, \quad x_0 = 0, \; x_1 = 1
```

Les premiers nombres de la suite sont $0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55$.

Écrivez une fonction pour calculer récursivement le $t$-ième nombre de Fibonacci pour tout $t$.

```{exercise-end}
```

```{solution-start} func_ex4
:class: dropdown
```

Voici la solution standard

```{code-cell} python3
def x(t):
    if t == 0:
        return 0
    if t == 1:
        return 1
    else:
        return x(t-1) + x(t-2)
```

Testons-la

```{code-cell} python3
print([x(i) for i in range(10)])
```

```{solution-end}
```

```{exercise-start}
:label: func_ex5
```

Réécrivez la fonction `factorial()` de l'[Exercice 1](factorial_exercise) en utilisant la récursion.

```{exercise-end}
```

```{solution-start} func_ex5
:class: dropdown
```

Voici la solution standard

```{code-cell} python3
def recursion_factorial(n):
   if n == 1:
       return n
   else:
       return n * recursion_factorial(n-1)
```

Testons-la

```{code-cell} python3
print([recursion_factorial(i) for i in range(1, 10)])
```

```{solution-end}
```