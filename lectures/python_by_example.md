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
  title: Un exemple introductif
  headings:
    Overview: Vue d'ensemble
    'The Task: Plotting a White Noise Process': "La tâche\_: tracer un processus de bruit blanc"
    Version 1: Version 1
    Version 1::Imports: Importations
    Version 1::Imports::Why So Many Imports?: "Pourquoi autant d'importations\_?"
    Version 1::Imports::Packages: Packages
    Version 1::Imports::Subpackages: Sous-packages
    Version 1::Importing Names Directly: Importer directement des noms
    Version 1::Random Draws: Tirages aléatoires
    Alternative Implementations: Implémentations alternatives
    Alternative Implementations::A Version with a For Loop: Une version avec une boucle for
    Alternative Implementations::Lists: Listes
    Alternative Implementations::The For Loop: La boucle for
    Alternative Implementations::A Comment on Indentation: Une remarque sur l'indentation
    Alternative Implementations::While Loops: Boucles while
    Another Application: Une autre application
    Exercises: Exercices
---

(python_by_example)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Un exemple introductif

```{index} single: Python; Exemple introductif
```

## Vue d'ensemble

Nous sommes maintenant prêts à commencer à apprendre le langage Python lui-même.

Dans ce cours, nous allons écrire puis décortiquer de petits programmes Python.

L'objectif est de vous présenter la syntaxe de base de Python et ses structures de données.

Des concepts plus approfondis seront abordés dans les cours ultérieurs.

Vous devriez avoir lu le {doc}`cours <getting_started>` sur la prise en main de Python avant de commencer celui-ci.


## La tâche : tracer un processus de bruit blanc

Supposons que nous voulions simuler et tracer le processus de bruit blanc
$\epsilon_0, \epsilon_1, \ldots, \epsilon_T$, où chaque tirage $\epsilon_t$ est un tirage indépendant d'une loi normale centrée réduite.

En d'autres termes, nous voulons générer des figures qui ressemblent à ceci :

```{figure} /_static/lecture_specific/python_by_example/test_program_1_updated.png
:scale: 120
```

(Ici $t$ est sur l'axe horizontal et $\epsilon_t$ sur l'axe
vertical.)

Nous ferons cela de plusieurs façons différentes, en apprenant à chaque fois quelque chose de plus
sur Python.

## Version 1

(ourfirstprog)=
Voici quelques lignes de code qui accomplissent la tâche que nous nous sommes fixée

```{code-cell} ipython
import numpy as np
import matplotlib.pyplot as plt

rng = np.random.default_rng()
ϵ_values = rng.standard_normal(100)
plt.plot(ϵ_values)
plt.show()
```

Décomposons ce programme et voyons comment il fonctionne.

(import)=
### Importations

Les deux premières lignes du programme importent des fonctionnalités provenant de bibliothèques
de code externes.

La première ligne importe {doc}`NumPy <numpy>`, un package Python privilégié pour des tâches telles que

* le travail avec des tableaux (vecteurs et matrices)
* les fonctions mathématiques courantes comme `cos` et `sqrt`
* la génération de nombres aléatoires
* l'algèbre linéaire, etc.

Après `import numpy as np`, nous avons accès à ces attributs via la syntaxe `np.attribute`.

Voici deux autres exemples

```{code-cell} python3
np.sqrt(4)
```

```{code-cell} python3
np.log(4)
```


#### Pourquoi autant d'importations ?

Les programmes Python nécessitent généralement plusieurs instructions d'importation.

La raison en est que le cœur du langage est délibérément maintenu petit, afin qu'il soit facile à apprendre, à maintenir et à améliorer.

Lorsque vous voulez faire quelque chose d'intéressant avec Python, vous avez presque toujours besoin
d'importer des fonctionnalités supplémentaires.


#### Packages

```{index} single: Python; Packages
```

Comme indiqué ci-dessus, NumPy est un package Python.

Les packages sont utilisés par les développeurs pour organiser le code qu'ils souhaitent partager.

En fait, un **package** est simplement un répertoire contenant

1. des fichiers avec du code Python --- appelés **modules** dans le jargon Python
1. éventuellement du code compilé accessible par Python (par exemple, des fonctions compilées à partir de code C ou FORTRAN)
1. un fichier appelé `__init__.py` qui spécifie ce qui sera exécuté lorsque nous tapons `import package_name`

Vous pouvez vérifier l'emplacement de votre `__init__.py` pour NumPy en Python en exécutant le code :

```{code-block} ipython
:class: no-execute

import numpy as np

print(np.__file__)
```

#### Sous-packages

```{index} single: Python; Sous-packages
```

Considérons la ligne `rng = np.random.default_rng()`.

Ici `np` fait référence au package NumPy, tandis que `random` est un **sous-package** de NumPy.

Les sous-packages sont simplement des packages qui sont des sous-répertoires d'un autre package.

Par exemple, vous pouvez trouver le dossier `random` sous le répertoire de NumPy.

### Importer directement des noms

Rappelons le code que nous avons vu ci-dessus

```{code-cell} python3
import numpy as np

np.sqrt(4)
```

Voici une autre façon d'accéder à la fonction racine carrée de NumPy

```{code-cell} python3
from numpy import sqrt

sqrt(4)
```

C'est également correct.

L'avantage est qu'il y a moins à taper si nous utilisons `sqrt` souvent dans notre code.

L'inconvénient est que, dans un long programme, ces deux lignes pourraient être
séparées par de nombreuses autres lignes.

Il est alors plus difficile pour les lecteurs de savoir d'où vient `sqrt`, s'ils le souhaitent.

### Tirages aléatoires

Pour revenir à notre programme qui trace le bruit blanc, les trois lignes restantes
après les instructions d'importation sont

```{code-cell} ipython
ϵ_values = rng.standard_normal(100)
plt.plot(ϵ_values)
plt.show()
```

La première ligne génère 100 tirages (quasi) indépendants d'une loi normale centrée réduite et les stocke
dans `ϵ_values`.

Les deux lignes suivantes génèrent le graphique.

Nous pouvons et allons examiner ci-dessous diverses façons de configurer et d'améliorer ce graphique.

## Implémentations alternatives

Essayons d'écrire quelques versions alternatives de {ref}`notre premier programme <ourfirstprog>`, qui traçait des tirages IID de la loi normale centrée réduite.

Les programmes ci-dessous sont moins efficaces que l'original, et donc
quelque peu artificiels.

Mais ils nous aident à illustrer une syntaxe et une sémantique Python importantes dans un cadre familier.

### Une version avec une boucle for

Voici une version qui illustre les boucles `for` et les listes Python.

(firstloopprog)=
```{code-cell} python3
ts_length = 100
ϵ_values = []   # liste vide

for i in range(ts_length):
    e = rng.standard_normal()
    ϵ_values.append(e)

plt.plot(ϵ_values)
plt.show()
```

En bref,

* La première ligne définit la longueur souhaitée de la série temporelle.
* La ligne suivante crée une *liste* vide appelée `ϵ_values` qui stockera les valeurs $\epsilon_t$ au fur et à mesure que nous les générons.
* L'instruction `# liste vide` est un *commentaire*, et est ignorée par l'interpréteur de Python.
* Les trois lignes suivantes constituent la boucle `for`, qui tire de manière répétée un nouveau nombre aléatoire $\epsilon_t$ et l'ajoute à la fin de la liste `ϵ_values`.
* Les deux dernières lignes génèrent le graphique et l'affichent à l'utilisateur.

Étudions certaines parties de ce programme plus en détail.

(lists_ref)=
### Listes

```{index} single: Python; Listes
```

Considérons l'instruction `ϵ_values = []`, qui crée une liste vide.

Les listes sont une structure de données native de Python utilisée pour regrouper une collection d'objets.

Les éléments des listes sont ordonnés, et les doublons sont autorisés dans les listes.

Par exemple, essayez

```{code-cell} python3
x = [10, 'foo', False]
type(x)
```

Le premier élément de `x` est un [entier](https://en.wikipedia.org/wiki/Integer_(computer_science)), le suivant est une [chaîne de caractères](https://en.wikipedia.org/wiki/String_(computer_science)), et le troisième est une [valeur booléenne](https://en.wikipedia.org/wiki/Boolean_data_type).

Lors de l'ajout d'une valeur à une liste, nous pouvons utiliser la syntaxe `list_name.append(some_value)`

```{code-cell} python3
x
```

```{code-cell} python3
x.append(2.5)
x
```

Ici `append()` est ce qu'on appelle une **méthode**, c'est-à-dire une fonction « attachée à » un objet --- dans ce cas, la liste `x`.

Nous apprendrons tout sur les méthodes {doc}`plus tard <oop_intro>`, mais juste pour vous donner une idée,

* Les objets Python tels que les listes, les chaînes de caractères, etc. ont tous des méthodes utilisées pour manipuler les données contenues dans l'objet.
* Les objets chaînes de caractères ont des [méthodes de chaîne](https://docs.python.org/3/library/stdtypes.html#string-methods), les objets listes ont des [méthodes de liste](https://docs.python.org/3/tutorial/datastructures.html#more-on-lists), etc.

Une autre méthode de liste utile est `pop()`

```{code-cell} python3
x
```

```{code-cell} python3
x.pop()
```

```{code-cell} python3
x
```

Les listes en Python sont indexées à partir de zéro (comme en C, Java ou Go), donc le premier élément est référencé par `x[0]`

```{code-cell} python3
x[0]   # premier élément de x
```

```{code-cell} python3
x[1]   # deuxième élément de x
```

### La boucle for

```{index} single: Python; Boucle for
```

Considérons maintenant la boucle `for` du {ref}`programme ci-dessus <firstloopprog>`, qui était

```{code-cell} python3
for i in range(ts_length):
    e = rng.standard_normal()
    ϵ_values.append(e)
```

Python exécute les deux lignes indentées `ts_length` fois avant de passer à la suite.

Ces deux lignes sont appelées un **bloc de code**, car elles constituent le « bloc » de code sur lequel nous bouclons.

Contrairement à la plupart des autres langages, Python connaît l'étendue du bloc de code *uniquement grâce à l'indentation*.

Dans notre programme, l'indentation diminue après la ligne `ϵ_values.append(e)`, indiquant à Python que cette ligne marque la limite inférieure du bloc de code.

Plus de détails sur l'indentation ci-dessous --- pour l'instant, regardons un autre exemple de boucle `for`

```{code-cell} python3
animals = ['dog', 'cat', 'bird']
for animal in animals:
    print("The plural of " + animal + " is " + animal + "s")
```

Cet exemple aide à clarifier comment fonctionne la boucle `for` : lorsque nous exécutons une
boucle de la forme

```{code-block} python3
:class: no-execute

for variable_name in sequence:
    <code block>
```

L'interpréteur Python effectue ce qui suit :

* Pour chaque élément de la `sequence`, il « lie » le nom `variable_name` à cet élément puis exécute le bloc de code.


### Une remarque sur l'indentation

```{index} single: Python; Indentation
```

En discutant de la boucle `for`, nous avons expliqué que les blocs de code sur lesquels on boucle sont délimités par l'indentation.

En fait, en Python, *tous* les blocs de code (c'est-à-dire ceux qui se trouvent dans les boucles, les clauses if, les définitions de fonctions, etc.) sont délimités par l'indentation.

Ainsi, contrairement à la plupart des autres langages, les espaces blancs dans le code Python affectent la sortie du programme.

Une fois que vous vous y êtes habitué, c'est une bonne chose : cela

* force une indentation propre et cohérente, améliorant la lisibilité
* supprime l'encombrement, comme les accolades ou les instructions de fin utilisées dans d'autres langages

D'un autre côté, cela demande un peu de soin pour être fait correctement, alors veuillez retenir :

* La ligne précédant le début d'un bloc de code se termine toujours par deux points
    * `for i in range(10):`
    * `if x > y:`
    * `while x < 100:`
    * etc.
* Toutes les lignes d'un bloc de code doivent avoir la même quantité d'indentation.
* La norme Python est de 4 espaces, et c'est ce que vous devriez utiliser.

### Boucles while

```{index} single: Python; Boucle while
```

La boucle `for` est la technique la plus courante d'itération en Python.

Mais, à des fins d'illustration, modifions {ref}`le programme ci-dessus <firstloopprog>` pour utiliser une boucle `while` à la place.

(whileloopprog)=
```{code-cell} python3
ts_length = 100
ϵ_values = []
i = 0
while i < ts_length:
    e = rng.standard_normal()
    ϵ_values.append(e)
    i = i + 1
plt.plot(ϵ_values)
plt.show()
```

Une boucle while continuera d'exécuter le bloc de code délimité par l'indentation jusqu'à ce que la condition (```i < ts_length```) soit satisfaite.

Dans ce cas, le programme continuera d'ajouter des valeurs à la liste ```ϵ_values``` jusqu'à ce que ```i``` soit égal à ```ts_length``` :

```{code-cell} python3
i == ts_length #la condition de fin de la boucle while
```

Notez que

* le bloc de code de la boucle `while` est à nouveau délimité uniquement par l'indentation.
* l'instruction `i = i + 1` peut être remplacée par `i += 1`.

## Une autre application

Faisons encore une application avant de passer aux exercices.

Dans cette application, nous traçons le solde d'un compte bancaire au fil du temps.

Il n'y a pas de retraits au cours de la période, dont la dernière date est notée
par $T$.

Le solde initial est $b_0$ et le taux d'intérêt est $r$.

Le solde se met à jour de la période $t$ à $t+1$ selon $b_{t+1} = (1 + r) b_t$.

Dans le code ci-dessous, nous générons et traçons la séquence $b_0, b_1, \ldots, b_T$.

Au lieu d'utiliser une liste Python pour stocker cette séquence, nous utiliserons un tableau
NumPy.

```{code-cell} python3
r = 0.025         # taux d'intérêt
T = 50            # date de fin
b = np.empty(T+1) # un tableau NumPy vide, pour stocker tous les b_t
b[0] = 10         # solde initial

for t in range(T):
    b[t+1] = (1 + r) * b[t]

plt.plot(b, label='solde bancaire')
plt.legend()
plt.show()
```

L'instruction `b = np.empty(T+1)` alloue de l'espace mémoire pour `T+1`
nombres (à virgule flottante).

Ces nombres sont remplis par la boucle `for`.

Allouer la mémoire au départ est plus efficace que d'utiliser une liste Python et
`append`, car cette dernière doit demander de manière répétée de l'espace de stockage au
système d'exploitation.

Remarquez que nous avons ajouté une légende au graphique --- une fonctionnalité que l'on vous demandera
d'utiliser dans les exercices.

## Exercices

Passons maintenant aux exercices. Il est important que vous les complétiez avant
de continuer, car ils présentent de nouveaux concepts dont nous aurons besoin.

```{exercise-start}
:label: pbe_ex1
```

Votre première tâche est de simuler et de tracer la série temporelle corrélée

$$
x_{t+1} = \alpha \, x_t + \epsilon_{t+1}
\quad \text{où} \quad
x_0 = 0
\quad \text{et} \quad t = 0,\ldots,T
$$

La séquence de chocs $\{\epsilon_t\}$ est supposée être IID et suivre une loi normale centrée réduite.

Dans votre solution, limitez vos instructions d'importation à

```{code-cell} python3
import numpy as np
import matplotlib.pyplot as plt
```

Posez $T=200$ et $\alpha = 0.9$.

```{exercise-end}
```

```{solution-start} pbe_ex1
:class: dropdown
```

Voici une solution.

```{code-cell} python3
α = 0.9
T = 200
x = np.empty(T+1)
x[0] = 0
rng = np.random.default_rng()

for t in range(T):
    x[t+1] = α * x[t] + rng.standard_normal()

plt.plot(x)
plt.show()
```

```{solution-end}
```


```{exercise-start}
:label: pbe_ex2

En partant de votre solution à l'exercice 1, tracez trois séries temporelles simulées,
une pour chacun des cas $\alpha=0$, $\alpha=0.8$ et $\alpha=0.98$.

Utilisez une boucle `for` pour parcourir les valeurs de $\alpha$.

Si vous le pouvez, ajoutez une légende, pour aider à distinguer les trois séries temporelles.

```{hint}
:class: dropdown

* Si vous appelez la fonction `plot()` plusieurs fois avant d'appeler `show()`, toutes les lignes que vous produisez se retrouveront sur la même figure.
* Pour la légende, notez que si `var = 42`, l'expression `f'foo{var}'` s'évalue en `'foo42'`.
```

```{exercise-end}
```


```{solution-start} pbe_ex2
:class: dropdown
```

```{code-cell} python3
α_values = [0.0, 0.8, 0.98]
T = 200
x = np.empty(T+1)
rng = np.random.default_rng()

for α in α_values:
    x[0] = 0
    for t in range(T):
        x[t+1] = α * x[t] + rng.standard_normal()
    plt.plot(x, label=f'$\\alpha = {α}$')

plt.legend()
plt.show()
```

```{note}
`f'$\\alpha = {α}$'` dans la solution est une application de la [f-string](https://docs.python.org/3/tutorial/inputoutput.html#tut-f-strings), qui vous permet d'utiliser `{}` pour contenir une expression.

L'expression contenue sera évaluée, et le résultat sera placé dans la chaîne de caractères.
```

```{solution-end}
```

```{exercise-start}
:label: pbe_ex3

De manière similaire aux exercices précédents, tracez la série temporelle

$$
x_{t+1} = \alpha \, |x_t| + \epsilon_{t+1}
\quad \text{où} \quad
x_0 = 0
\quad \text{et} \quad t = 0,\ldots,T
$$

Utilisez $T=200$, $\alpha = 0.9$ et $\{\epsilon_t\}$ comme précédemment.

Recherchez en ligne une fonction qui peut être utilisée pour calculer la valeur absolue $|x_t|$.
```

```{exercise-end}
```


```{solution-start} pbe_ex3
:class: dropdown
```

Voici une solution :

```{code-cell} python3
α = 0.9
T = 200
x = np.empty(T+1)
x[0] = 0
rng = np.random.default_rng()

for t in range(T):
    x[t+1] = α * np.abs(x[t]) + rng.standard_normal()

plt.plot(x)
plt.show()
```

```{solution-end}
```


```{exercise-start}
:label: pbe_ex4
```

Un aspect important de pratiquement tous les langages de programmation est le branchement et
les conditions.

En Python, les conditions sont généralement implémentées avec la syntaxe if--else.

Voici un exemple, qui affiche -1 pour chaque nombre négatif d'un tableau et 1
pour chaque nombre non négatif

```{code-cell} python3
numbers = [-9, 2.3, -11, 0]
```

```{code-cell} python3
for x in numbers:
    if x < 0:
        print(-1)
    else:
        print(1)
```

Maintenant, écrivez une nouvelle solution à l'exercice 3 qui n'utilise pas de fonction existante
pour calculer la valeur absolue.

Remplacez cette fonction existante par une condition if--else.

```{exercise-end}
```

```{solution-start} pbe_ex4
:class: dropdown
```

Voici une façon de faire :

```{code-cell} python3
α = 0.9
T = 200
x = np.empty(T+1)
x[0] = 0
rng = np.random.default_rng()

for t in range(T):
    if x[t] < 0:
        abs_x = - x[t]
    else:
        abs_x = x[t]
    x[t+1] = α * abs_x + rng.standard_normal()

plt.plot(x)
plt.show()
```

Voici une façon plus courte d'écrire la même chose :

```{code-cell} python3
α = 0.9
T = 200
x = np.empty(T+1)
x[0] = 0
rng = np.random.default_rng()

for t in range(T):
    abs_x = - x[t] if x[t] < 0 else x[t]
    x[t+1] = α * abs_x + rng.standard_normal()

plt.plot(x)
plt.show()
```

```{solution-end}
```



```{exercise-start}
:label: pbe_ex5
```

Voici un exercice plus difficile, qui demande de la réflexion et de la planification.

La tâche consiste à calculer une approximation de $\pi$ en utilisant [Monte-Carlo](https://en.wikipedia.org/wiki/Monte_Carlo_method).

N'utilisez aucune importation en dehors de

```{code-cell} python3
import numpy as np
```

```{hint}
:class: dropdown

Vos indices sont les suivants :

* Si $U$ est une variable aléatoire uniforme bivariée sur le carré unité $(0, 1)^2$, alors la probabilité que $U$ se trouve dans un sous-ensemble $B$ de $(0,1)^2$ est égale à l'aire de $B$.
* Si $U_1,\ldots,U_n$ sont des copies IID de $U$, alors, à mesure que $n$ devient grand, la fraction qui tombe dans $B$ converge vers la probabilité d'atterrir dans $B$.
* Pour un cercle, $area = \pi * radius^2$.
```

```{exercise-end}
```


```{solution-start} pbe_ex5
:class: dropdown
```

Considérons le cercle de diamètre 1 inscrit dans le carré unité.

Soit $A$ son aire et soit $r=1/2$ son rayon.

Si nous connaissons $\pi$ alors nous pouvons calculer $A$ via
$A = \pi r^2$.

Mais ici le but est de calculer $\pi$, ce que nous pouvons faire par
$\pi = A / r^2$.

Résumé : si nous pouvons estimer l'aire d'un cercle de diamètre 1, alors diviser
par $r^2 = (1/2)^2 = 1/4$ donne une estimation de $\pi$.

Nous estimons l'aire en échantillonnant des uniformes bivariées et en regardant la
fraction qui tombe dans le cercle.

```{code-cell} python3
n = 1000000 # taille d'échantillon pour la simulation de Monte-Carlo
rng = np.random.default_rng()

count = 0
for i in range(n):

    # tirage de positions aléatoires sur le carré
    u, v = rng.uniform(), rng.uniform()

    # vérifier si le point tombe à l'intérieur de la limite
    # du cercle unité centré en (0.5,0.5)
    d = np.sqrt((u - 0.5)**2 + (v - 0.5)**2)

    # s'il tombe à l'intérieur du cercle inscrit,
    # l'ajouter au compteur
    if d < 0.5:
        count += 1

area_estimate = count / n

print(area_estimate * 4)  # division par radius**2
```

```{solution-end}
```