---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.7
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
translation:
  title: Numba
  headings:
    Overview: Vue d'ensemble
    Compiling Functions: Compiler des fonctions
    Compiling Functions::An Example: Un exemple
    Compiling Functions::An Example::Base Version: Version de base
    Compiling Functions::An Example::Acceleration via Numba: Accélération via Numba
    Compiling Functions::How and When it Works: Comment et quand cela fonctionne
    Sharp Bits: Points délicats
    Sharp Bits::Typing: Typage
    Sharp Bits::Global Variables: Variables globales
    Multithreaded Loops in Numba: Boucles multithreadées dans Numba
    Exercises: Exercices
---

(numba_lecture)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Numba

En plus de ce qui est inclus dans Anaconda, ce cours nécessitera les bibliothèques suivantes :

```{code-cell} ipython3
:tags: [hide-output]

!pip install quantecon
```

Veuillez également vous assurer que vous disposez de la dernière version d'Anaconda, car les anciennes versions sont une {doc}`source fréquente d'erreurs <troubleshooting>`.

Commençons par quelques importations :

```{code-cell} ipython3
import numpy as np
import quantecon as qe
import matplotlib.pyplot as plt
```


## Vue d'ensemble

Dans un {doc}`cours précédent <need_for_speed>`, nous avons abordé la vectorisation, 
qui peut améliorer la vitesse d'exécution en envoyant les opérations de traitement de tableaux par lots vers du code de bas niveau efficace.

Cependant, comme {ref}`discuté dans ce cours <numba-p_c_vectorization>`,
les schémas de vectorisation traditionnels présentent des faiblesses :

* Très gourmands en mémoire pour les opérations composées sur les tableaux
* Inefficaces ou impossibles pour certains algorithmes

Une façon de contourner ces problèmes consiste à utiliser [Numba](https://numba.pydata.org/), un
**compilateur à la volée (JIT)** pour Python.

Numba compile les fonctions en instructions machine natives lors de l'exécution.

Lorsqu'il y parvient, le résultat est une performance comparable à celle du code C ou Fortran compilé.

De plus, Numba peut effectuer des astuces utiles telles que le {ref}`multithreading <multithreading>`.

Ce cours présente les idées essentielles.


```{note}
Certains lecteurs pourraient être curieux de connaître la relation entre Numba et [Julia](https://julialang.org/),
qui contient son propre compilateur JIT. Bien que les deux compilateurs soient similaires à
de nombreux égards, Numba est moins ambitieux, ne cherchant à compiler qu'un petit sous-ensemble
du langage Python. Bien que cela puisse ressembler à un défaut, c'est aussi une
force : la nature plus restrictive de Numba le rend facile à bien utiliser et
performant dans ce qu'il fait.
```



(numba_link)=
## {index}`Compiler des fonctions <single: Compiler des fonctions>`

```{index} single: Python; Numba
```


(quad_map_eg)=
### Un exemple

Considérons un problème difficile à vectoriser (c'est-à-dire à confier à des opérations de traitement de tableaux). 

Le problème consiste à générer la trajectoire via l'application quadratique

$$
    x_{t+1} = \alpha x_t (1 - x_t)
$$

Dans ce qui suit, nous posons $\alpha = 4$.

#### Version de base

Voici le tracé d'une trajectoire typique, à partir de $x_0 = 0.1$, avec $t$ sur l'axe des abscisses

```{code-cell} ipython3
def qm(x0, n, α=4.0):
    x = np.empty(n+1)
    x[0] = x0
    for t in range(n):
      x[t+1] = α * x[t] * (1 - x[t])
    return x

x = qm(0.1, 250)
fig, ax = plt.subplots()
ax.plot(x, 'b-', lw=2, alpha=0.8)
ax.set_xlabel('$t$', fontsize=12)
ax.set_ylabel('$x_{t}$', fontsize = 12)
plt.show()
```

Voyons combien de temps cela prend pour un grand $n$

```{code-cell} ipython3
n = 10_000_000

with qe.Timer() as timer1:
    # Chronométrage de la version Python de base
    x = qm(0.1, n)

```


#### Accélération via Numba

Pour accélérer la fonction `qm` à l'aide de Numba, nous importons d'abord la fonction `jit`


```{code-cell} ipython3
from numba import jit
```

Nous l'appliquons maintenant à `qm`, produisant une nouvelle fonction :

```{code-cell} ipython3
qm_numba = jit(qm)
```

La fonction `qm_numba` est une version de `qm` qui est « ciblée » pour
la compilation JIT.

Nous expliquerons ce que cela signifie dans un instant.

Chronométrons cette nouvelle version :

```{code-cell} ipython3
with qe.Timer() as timer2:
    # Chronométrage de la version jittée
    x = qm_numba(0.1, n)
```

C'est un gain de vitesse important.

En fait, la fois suivante et toutes les fois suivantes, elle s'exécute encore plus vite car la
fonction a été compilée et se trouve en mémoire :

(qm_numba_result)=

```{code-cell} ipython3
with qe.Timer() as timer3:
    # Deuxième exécution
    x = qm_numba(0.1, n)
```

Voici le gain de vitesse

```{code-cell} ipython3
timer1.elapsed /  timer3.elapsed
```

C'est un grand gain pour une petite modification de notre code d'origine.

Voyons comment cela fonctionne.

### Comment et quand cela fonctionne

Numba tente de générer du code machine rapide en utilisant l'infrastructure fournie
par le [projet LLVM](https://llvm.org/).

Il le fait en inférant les informations de type à la volée.

(Voir notre {doc}`cours précédent <need_for_speed>` sur le calcul scientifique pour une discussion sur les types.)

L'idée de base est la suivante :

* Python est très flexible et nous pourrions donc appeler la fonction qm avec de nombreux types.
    * par exemple, `x0` pourrait être un tableau NumPy ou une liste, `n` pourrait être un entier ou un flottant, etc.
* Cela rend très difficile la génération de code machine efficace *à l'avance* (c'est-à-dire avant l'exécution).
* Cependant, lorsque nous *appelons* effectivement la fonction, par exemple en exécutant `qm(0.5, 10)`,
      les types de `x0`, `α` et `n` sont déterminés.
* De plus, les types *des autres variables* dans `qm` *peuvent être inférés une fois les types d'entrée connus*.
* La stratégie de Numba et des autres compilateurs JIT consiste donc à *attendre que la fonction soit appelée*, puis à compiler.

C'est ce qu'on appelle la compilation « à la volée » (« just-in-time »).

Notez que, si vous effectuez l'appel `qm_numba(0.5, 10)` puis le faites suivre de `qm_numba(0.9, 20)`, la compilation n'a lieu qu'au premier appel.

C'est parce que le code compilé est mis en cache et réutilisé au besoin.

C'est pourquoi, dans le code ci-dessus, la deuxième exécution de `qm_numba` est plus rapide.

```{admonition} Remarque
En pratique, plutôt que d'écrire `qm_numba = jit(qm)`, nous utilisons généralement
la syntaxe de *décorateur* et plaçons `@jit` avant la définition de la fonction. Cela équivaut
à ajouter `qm = jit(qm)` après la définition. 
```


## Points délicats

Numba est relativement facile à utiliser mais pas toujours sans accroc.

Passons en revue certains des problèmes que les utilisateurs rencontrent.

### Typage

Une inférence de type réussie est la clé de la compilation JIT.

Dans un contexte idéal, Numba peut inférer toutes les informations de type nécessaires.

Lorsque Numba *ne peut pas* inférer toutes les informations de type, il lève une erreur.

Par exemple, dans le contexte ci-dessous, Numba est incapable de déterminer le type de la
fonction `g` lors de la compilation de `iterate`

```{code-cell} ipython3
@jit
def iterate(f, x0, n):
    x = x0
    for t in range(n):
        x = f(x)
    return x

# Non jitté
def g(x):
    return np.cos(x) - 2 * np.sin(x)

# Ce code génère une erreur
try:
    iterate(g, 0.5, 100)
except Exception as e:
    print(e)
```

Dans le cas présent, nous pouvons facilement corriger cela en compilant `g`.

```{code-cell} ipython3
@jit
def g(x):
    return np.cos(x) - 2 * np.sin(x)

iterate(g, 0.5, 100)
```

Dans d'autres cas, comme lorsque nous voulons utiliser des fonctions de bibliothèques externes
telles que `SciPy`, il pourrait ne pas y avoir de solution de contournement simple.


### Variables globales

Une autre chose à laquelle il faut faire attention lors de l'utilisation de Numba est la gestion des
variables globales.

Par exemple, considérez le code suivant

```{code-cell} ipython3
a = 1

@jit
def add_a(x):
    return a + x

print(add_a(10))
```

```{code-cell} ipython3
a = 2

print(add_a(10))
```

Remarquez que changer la variable globale n'a eu aucun effet sur la valeur renvoyée par la
fonction 😱.

Lorsque Numba compile du code machine pour les fonctions, il traite les variables globales comme
des constantes afin de garantir la stabilité des types.

Pour éviter cela, passez les valeurs comme arguments de fonction plutôt que de dépendre des variables globales.


(multithreading)=
## Boucles multithreadées dans Numba

En plus de la compilation JIT, Numba offre un support pour le calcul parallèle sur CPU et GPU.

L'outil clé pour la parallélisation sur CPU dans Numba est la fonction `prange`, qui indique à
Numba d'exécuter les itérations de boucle en parallèle sur les cœurs disponibles.

Pour illustrer, examinons d'abord un morceau de code simple, à un seul thread (c'est-à-dire non parallélisé).

Le code simule la mise à jour de la richesse $w_t$ d'un ménage via la règle

$$
w_{t+1} = R_{t+1} s w_t + y_{t+1}
$$

Ici

* $R$ est le taux de rendement brut des actifs
* $s$ est le taux d'épargne du ménage et
* $y$ est le revenu du travail.

Nous modélisons à la fois $R$ et $y$ comme des tirages indépendants d'une loi
log-normale.

Voici le code :

```{code-cell} ipython3
@jit
def update(w, r=0.1, s=0.3, v1=0.1, v2=1.0):
    " Met à jour la richesse du ménage. "
    # Tirage des chocs
    R = np.exp(v1 * np.random.randn()) * (1 + r)
    y = np.exp(v2 * np.random.randn())
    # Mise à jour de la richesse
    w = R * s * w + y
    return w
```

Voyons comment la richesse évolue sous cette règle.

```{code-cell} ipython3
fig, ax = plt.subplots()

T = 100
w = np.empty(T)
w[0] = 5
for t in range(T-1):
    w[t+1] = update(w[t])

ax.plot(w)
ax.set_xlabel('$t$', fontsize=12)
ax.set_ylabel('$w_{t}$', fontsize=12)
plt.show()
```

Supposons maintenant que nous ayons une grande population de ménages et que nous voulions
savoir quelle sera la richesse médiane.

Ce n'est pas facile à résoudre avec un crayon et du papier, nous utiliserons donc la simulation
à la place :

1. Simuler un grand nombre de ménages dans le temps
2. Calculer la richesse médiane 

Voici le code :

```{code-cell} ipython3
@jit
def compute_long_run_median(w0=1, T=1000, num_reps=50_000):
    obs = np.empty(num_reps)
    # Pour chaque ménage
    for i in range(num_reps):
        # Fixer la condition initiale et avancer dans le temps
        w = w0
        for t in range(T):
            w = update(w)
        # Enregistrer la valeur finale
        obs[i] = w
    # Prendre la médiane de toutes les valeurs finales
    return np.median(obs)
```

Voyons à quelle vitesse cela s'exécute :

```{code-cell} ipython3
with qe.Timer():
    # Préchauffage
    compute_long_run_median()
```

```{code-cell} ipython3
with qe.Timer():
    # Deuxième exécution
    compute_long_run_median()
```

Pour accélérer cela, nous allons le paralléliser via le multithreading.

Pour ce faire, nous ajoutons l'option `parallel=True` et remplaçons `range` par `prange` :

```{code-cell} ipython3
from numba import prange

@jit(parallel=True)
def compute_long_run_median_parallel(
        w0=1, T=1000, num_reps=50_000
    ):
    obs = np.empty(num_reps)
    for i in prange(num_reps):  # Parallélisation sur les ménages
        w = w0
        for t in range(T):
            w = update(w)
        obs[i] = w
    return np.median(obs)
```

Regardons le chronométrage :

```{code-cell} ipython3
with qe.Timer():
    # Préchauffage
    compute_long_run_median_parallel()
```

```{code-cell} ipython3
with qe.Timer():
    # Deuxième exécution
    compute_long_run_median_parallel()
```

L'accélération est significative.

Remarquez que nous parallélisons entre les ménages plutôt que sur le temps -- les mises à jour d'un
ménage individuel à travers les périodes de temps sont intrinsèquement séquentielles.

Pour la parallélisation basée sur GPU, voir nos {doc}`cours sur JAX <jax_intro>`.

## Exercices

{ref}`speed_ex1` et {ref}`numba_ex3` estiment tous deux $\pi$ par Monte-Carlo à partir d'échantillons aléatoires dans le carré unité.

Nous les générons ici et les stockons dans `u_draws` et `v_draws` afin de pouvoir les utiliser dans les deux exercices et comparer les résultats

```{code-cell} ipython3
n = 1_000_000
rng = np.random.default_rng()
u_draws = rng.uniform(size=n)
v_draws = rng.uniform(size=n)
```

```{exercise}
:label: speed_ex1

{ref}`Précédemment <pbe_ex5>`, nous avons examiné comment approcher $\pi$ par
Monte-Carlo.

Utilisez la même idée ici, mais rendez le code efficace en utilisant Numba.

Comparez la vitesse avec et sans Numba lorsque la taille de l'échantillon est grande.
```

```{solution-start} speed_ex1
:class: dropdown
```

Voici une solution :

```{code-cell} ipython3
@jit
def calculate_pi(u_draws, v_draws):
    n = len(u_draws)
    count = 0
    for i in range(n):
        u, v = u_draws[i], v_draws[i]
        d = np.sqrt((u - 0.5)**2 + (v - 0.5)**2)
        if d < 0.5:
            count += 1

    area_estimate = count / n
    return area_estimate * 4  # division par le rayon**2
```

Voyons maintenant à quelle vitesse cela s'exécute :

```{code-cell} ipython3
with qe.Timer():
    calculate_pi(u_draws, v_draws)
```

```{code-cell} ipython3
with qe.Timer():
    calculate_pi(u_draws, v_draws)
```

Si nous désactivons la compilation JIT en supprimant `@jit`, le code prend environ
150 fois plus de temps sur notre machine.

Nous obtenons donc un gain de vitesse de 2 ordres de grandeur en ajoutant quatre caractères.

La solution ci-dessus adopte l'une des deux approches naturelles : elle *tire tous les
points aléatoires d'abord*, les stocke dans `u_draws` et `v_draws`, puis laisse la
fonction jittée les parcourir en boucle.

L'autre approche consiste à *tirer chaque point à l'intérieur de la boucle*.

Pour ce faire avec un `Generator` NumPy, nous passons `rng` en argument et appelons `rng.uniform()` à l'intérieur du corps de la boucle

```{code-cell} ipython3
@jit
def calculate_pi_in_loop(rng, n):
    count = 0
    for i in range(n):
        u, v = rng.uniform(), rng.uniform()
        d = np.sqrt((u - 0.5)**2 + (v - 0.5)**2)
        if d < 0.5:
            count += 1
    return (count / n) * 4
```

```{code-cell} ipython3
with qe.Timer():
    calculate_pi_in_loop(rng, n)
```

Dans ce contexte séquentiel, les deux approches donnent des estimations tout aussi bonnes et s'exécutent à une
vitesse similaire, mais elles ne sont pas équivalentes en termes d'*utilisation de la mémoire*. 

La première approche
doit conserver l'ensemble des $2n$ tirages en mémoire à la fois --- deux tableaux de `n` nombres à virgule
flottante, soit environ `16n` octets (environ $1.6$ Go lorsque `n = 100_000_000`). 

La
seconde tire chaque point à la demande et le rejette, de sorte que son empreinte mémoire
n'augmente pas avec `n`.

Cela pourrait suggérer que tirer à l'intérieur de la boucle est le meilleur choix par défaut.

 Mais comme nous
le verrons dans {ref}`numba_ex_race`, tirer à l'intérieur de la boucle interagit
mal avec la parallélisation.

```{solution-end}
```

```{exercise-start}
:label: speed_ex2
```

Dans la série de cours [Introduction to Quantitative Economics with
Python](https://intro.quantecon.org/intro.html), vous pouvez tout apprendre
sur les chaînes de Markov à états finis.

Pour l'instant, concentrons-nous simplement sur la simulation d'un exemple très simple d'une telle chaîne.

Supposons que la volatilité des rendements d'un actif puisse se trouver dans l'un des deux régimes --- élevé ou faible.

Les probabilités de transition entre les états sont les suivantes

```{image} /_static/lecture_specific/sci_libs/nfs_ex1.png
:align: center
```

Par exemple, prenons une durée de période d'un jour, et supposons que l'état actuel est élevé.

Nous voyons sur le graphique que l'état de demain sera

* élevé avec une probabilité de 0,8
* faible avec une probabilité de 0,2

Votre tâche est de simuler une séquence d'états quotidiens de volatilité selon cette règle.

Fixez la longueur de la séquence à `n = 1_000_000` et commencez dans l'état élevé.

Implémentez une version en Python pur et une version Numba, et comparez les vitesses.

Pour tester votre code, évaluez la fraction du temps que la chaîne passe dans l'état faible.

Si votre code est correct, elle devrait être d'environ 2/3.


```{hint}
:class: dropdown

* Représentez l'état faible par 0 et l'état élevé par 1.
* Si vous voulez stocker des entiers dans un tableau NumPy puis appliquer la compilation JIT, utilisez `x = np.empty(n, dtype=np.int64)`.

```

```{exercise-end}
```

```{solution-start} speed_ex2
:class: dropdown
```

Nous posons

- 0 représente « faible »
- 1 représente « élevé »

```{code-cell} ipython3
p, q = 0.1, 0.2  # Prob. de quitter respectivement l'état faible et l'état élevé
```

Voici une version en Python pur de la fonction

```{code-cell} ipython3
n = 1_000_000
rng = np.random.default_rng()
U = rng.uniform(0, 1, size=n)

def compute_series(n, U):
    x = np.empty(n, dtype=np.int64)
    x[0] = 1  # Commencer dans l'état 1
    for t in range(1, n):
        current_x = x[t-1]
        if current_x == 0:
            x[t] = U[t] < p
        else:
            x[t] = U[t] > q
    return x
```

Exécutons ce code et vérifions que la fraction du temps passé dans l'état
faible est d'environ 0,666

```{code-cell} ipython3
x = compute_series(n, U)
print(np.mean(x == 0))  # Fraction du temps où x est dans l'état 0
```

C'est (approximativement) la bonne sortie.

Chronométrons-le maintenant :

```{code-cell} ipython3
with qe.Timer():
    compute_series(n, U)
```

Ensuite, implémentons une version Numba, ce qui est facile

```{code-cell} ipython3
compute_series_numba = jit(compute_series)
```

Vérifions que nous obtenons toujours les bons nombres

```{code-cell} ipython3
x = compute_series_numba(n, U)
print(np.mean(x == 0))
```

Voyons le temps

```{code-cell} ipython3
with qe.Timer():
    compute_series_numba(n, U)
```

C'est une belle amélioration de vitesse pour une ligne de code !

```{solution-end}
```

```{exercise}
:label: numba_ex3

Dans {ref}`un exercice précédent <speed_ex1>`, nous avons utilisé Numba pour accélérer un
effort de calcul de la constante $\pi$ par Monte-Carlo.

Essayez maintenant d'ajouter la parallélisation et voyez si vous obtenez des gains de vitesse supplémentaires.

Vous ne devez pas vous attendre à d'énormes gains ici car, bien qu'il y ait de nombreuses
tâches indépendantes (tirer un point et tester s'il est dans le cercle), chacune a un
temps d'exécution faible.

De manière générale, la parallélisation est moins efficace lorsque les
tâches individuelles à paralléliser sont très petites par rapport au temps d'exécution total.

Cela est dû aux surcoûts associés à la répartition de toutes ces petites tâches sur plusieurs CPU.

Néanmoins, avec un matériel adapté, il est possible d'obtenir des gains de vitesse non triviaux dans cet exercice.

Pour la taille de la simulation Monte-Carlo, utilisez quelque chose de substantiel, comme
`n = 100_000_000`.
```

```{solution-start} numba_ex3
:class: dropdown
```

Voici une solution :

```{code-cell} ipython3
@jit(parallel=True)
def calculate_pi(u_draws, v_draws):
    n = len(u_draws)
    count = 0
    for i in prange(n):
        u, v = u_draws[i], v_draws[i]
        d = np.sqrt((u - 0.5)**2 + (v - 0.5)**2)
        if d < 0.5:
            count += 1

    area_estimate = count / n
    return area_estimate * 4  # division par le rayon**2
```

Voyons maintenant à quelle vitesse cela s'exécute :

```{code-cell} ipython3
with qe.Timer():
    calculate_pi(u_draws, v_draws)
```

```{code-cell} ipython3
with qe.Timer():
    calculate_pi(u_draws, v_draws)
```

En activant et désactivant la parallélisation (en choisissant `True` ou
`False` dans l'annotation `@jit`), nous pouvons tester le gain de vitesse que
le multithreading apporte en plus de la compilation JIT.

Sur notre station de travail, nous constatons que la parallélisation augmente la vitesse d'exécution d'un
facteur de 2 ou 3.

(Si vous exécutez localement, vous obtiendrez des nombres différents, dépendant principalement
du nombre de CPU sur votre machine.)

Remarquez que nous avons tiré tous les points aléatoires *avant* la boucle et les avons passés
comme tableaux, de sorte que la boucle parallèle ne fait que *lire* depuis la mémoire.

Tirer les points *à l'intérieur* de la boucle parallèle est étonnamment délicat.


Nous étudions pourquoi, et comment le faire de manière sûre, dans
{ref}`numba_ex_race`.

```{solution-end}
```


```{exercise}
:label: numba_ex_race

Dans {ref}`numba_ex3`, nous avons tiré tous les points aléatoires *avant* la boucle parallèle.

Il est tentant de plutôt tirer chaque point *à l'intérieur* de la boucle `prange`, en passant un générateur `rng` en argument et en appelant `rng.uniform()` dans le corps de la boucle.

Essayez-le : le code devrait s'exécuter et renvoyer un nombre proche de $\pi$, pourtant il y a un bug subtil dans cette approche.

Enquêtez comme suit :

1. Appelez votre fonction quelques fois avec la *même* graine et vérifiez si le résultat est reproductible.
2. Répétez l'estimation de nombreuses fois sur une gamme de tailles d'échantillon et comparez sa dispersion à celle d'une version parallèle correcte.

Expliquez ensuite ce qui ne va pas et donnez une manière correcte de tirer à l'intérieur d'une boucle parallèle.

Astuce : essayez d'utiliser une fonction aléatoire ancienne telle que `np.random.uniform()` au lieu d'un `Generator` et voyez ce qui se passe.
```

```{solution-start} numba_ex_race
:class: dropdown
```

Voici la version tentante.

Nous passons `rng` en argument et l'appelons à l'intérieur de la boucle `prange`.

```{code-cell} ipython3
n = 1_000_000
rng = np.random.default_rng()

@jit(parallel=True)
def calculate_pi_in_loop(rng, n):
    count = 0
    for i in prange(n):
        u, v = rng.uniform(), rng.uniform()
        d = np.sqrt((u - 0.5)**2 + (v - 0.5)**2)
        if d < 0.5:
            count += 1
    return (count / n) * 4

calculate_pi_in_loop(rng, n)
```

Le code s'exécute sans erreur et renvoie quelque chose de proche de $\pi$.

Mais quelque chose ne va pas silencieusement avec les résultats.

Ici, chaque thread tire du *même* générateur `rng`.

Un générateur produit chaque nombre en mettant à jour un état interne.

Sous `prange`, de nombreux threads lisent et mettent à jour ce seul état à la fois, sans coordination entre eux.

C'est une [**course aux données** (data race)](https://docs.oracle.com/cd/E19205-01/820-0619/geojs/index.html).

Elle crée des corrélations entre les tirages et peut même faire que certains tirages soient dupliqués de manière
imprévisible.

Deux symptômes révèlent le problème.

*Symptôme 1 : le résultat n'est plus reproductible.*

Un générateur correct renvoie la même réponse chaque fois qu'on lui donne la même graine.

À cause de la course aux données, l'ordre dans lequel les threads touchent l'état partagé affecte le flux de tirages, de sorte que la réponse n'est pas reproductible même lorsque la graine est fixée.

```{code-cell} ipython3
for seed in (1, 1, 1):
    print(calculate_pi_in_loop(np.random.default_rng(seed), n))
```

Chaque appel utilise la même graine, pourtant les réponses diffèrent.

*Symptôme 2 : l'estimateur est bien plus bruité qu'il ne devrait l'être.*

Les tirages dupliqués et corrélés portent moins d'information que $n$ tirages indépendants, de sorte que la taille d'échantillon *effective* est bien plus petite que $n$.

La solution consiste à donner à chaque thread son propre état aléatoire, ce que les anciennes fonctions de NumPy telles que `np.random.uniform()` font automatiquement sous Numba.

```{code-cell} ipython3
@jit(parallel=True)
def calculate_pi_legacy(n):
    count = 0
    for i in prange(n):
        u, v = np.random.uniform(0, 1), np.random.uniform(0, 1)
        d = np.sqrt((u - 0.5)**2 + (v - 0.5)**2)
        if d < 0.5:
            count += 1
    return (count / n) * 4
```

Pour voir le coût de la course, nous répétons chaque estimation de nombreuses fois et traçons sa dispersion par rapport à la version correcte à mesure que la taille de l'échantillon augmente.

```{code-cell} ipython3
sample_sizes = np.logspace(3, 6, 10).astype(int)
num_reps = 20

methods = [("état par thread (correct)",
            lambda n: calculate_pi_legacy(n), 'C0'),
           ("générateur partagé dans prange (course aux données)",
            lambda n: calculate_pi_in_loop(np.random.default_rng(), n), 'C1')]

fig, ax = plt.subplots()
for label, estimate, color in methods:
    draws = np.array([[estimate(int(m)) for _ in range(num_reps)]
                      for m in sample_sizes])
    means, stds = draws.mean(axis=1), draws.std(axis=1)
    ax.plot(sample_sizes, means, color=color, marker='o', ms=3, label=label)
    ax.fill_between(sample_sizes, means - 2 * stds, means + 2 * stds,
                    color=color, alpha=0.2)
ax.axhline(np.pi, color='k', lw=0.8, ls='--', label=r'$\pi$')
ax.set_xscale('log')
ax.set_xlabel('nombre d\'échantillons')
ax.set_ylabel(r'estimation de $\pi$')
ax.legend()
plt.show()
```

Les deux bandes sont centrées sur $\pi$, mais la bande associée à la course aux données est bien plus large que l'autre et se rétrécit très lentement à mesure que la taille de l'échantillon augmente.

L'autre option sûre est celle de {ref}`numba_ex3` : tirer les points avant la boucle afin que la boucle parallèle ne fasse que lire depuis la mémoire.

```{solution-end}
```


```{exercise}
:label: numba_ex_draw_speed

Nous avons maintenant deux façons correctes d'estimer $\pi$ en parallèle.

L'une tire tous les points *avant* la boucle, comme dans {ref}`numba_ex3`.

L'autre les tire *à l'intérieur* de la boucle avec des fonctions anciennes, comme dans {ref}`numba_ex_race`.

Comparez leur vitesse à `n = 100_000_000`, en incluant le temps passé à générer les points aléatoires.
```

```{solution-start} numba_ex_draw_speed
:class: dropdown
```

Nous chronométrons chaque approche du début à la fin, de sorte que la version qui pré-tire paie pour la construction de ses tableaux.

```{code-cell} ipython3
n = 100_000_000
rng = np.random.default_rng()

with qe.Timer():
    u_draws = rng.uniform(size=n)
    v_draws = rng.uniform(size=n)
    calculate_pi(u_draws, v_draws)
```

```{code-cell} ipython3
with qe.Timer():
    calculate_pi_legacy(n)
```

Tirer à l'intérieur de la boucle est bien plus rapide.

La version qui pré-tire génère ses deux tableaux sur un seul thread avant le début de la boucle.

La version en boucle répartit plutôt la génération de nombres aléatoires sur tous les threads.

Elle évite également d'allouer deux tableaux de `n` nombres, ce qui économise à la fois du temps et de la mémoire.

```{solution-end}
```


```{exercise}
:label: numba_ex4

Dans {doc}`notre cours sur SciPy<scipy>`, nous avons abordé l'évaluation d'une option d'achat dans un
contexte où le prix de l'action sous-jacente suivait une distribution simple et bien
connue.

Ici, nous abordons un contexte plus réaliste.

Rappelons que le prix de l'option obéit à

$$
P = \beta^n \mathbb E \max\{ S_n - K, 0 \}
$$

où

1. $\beta$ est un facteur d'actualisation,
2. $n$ est la date d'échéance,
3. $K$ est le prix d'exercice et
4. $\{S_t\}$ est le prix de l'actif sous-jacent à chaque instant $t$.

Supposons que `n, β, K = 20, 0.99, 100`.

Supposons que le prix de l'action obéit à

$$
\ln \frac{S_{t+1}}{S_t} = \mu + \sigma_t \xi_{t+1}
$$

où

$$
    \sigma_t = \exp(h_t),
    \quad
        h_{t+1} = \rho h_t + \nu \eta_{t+1}
$$

Ici $\{\xi_t\}$ et $\{\eta_t\}$ sont IID et normales centrées réduites.

(Il s'agit d'un modèle de **volatilité stochastique**, où la volatilité $\sigma_t$
varie dans le temps.)

Utilisez les valeurs par défaut `μ, ρ, ν, S0, h0 = 0.0001, 0.1, 0.001, 10, 0`.

(Ici `S0` est $S_0$ et `h0` est $h_0$.)

En générant $M$ trajectoires $s_0, \ldots, s_n$, calculez l'estimation Monte-Carlo

$$
    \hat P_M
    := \beta^n \mathbb E \max\{ S_n - K, 0 \}
    \approx
    \frac{1}{M} \sum_{m=1}^M \max \{S_n^m - K, 0 \}
$$


du prix, en appliquant Numba et la parallélisation.

```


```{solution-start} numba_ex4
:class: dropdown
```


Avec $s_t := \ln S_t$, la dynamique du prix devient

$$
s_{t+1} = s_t + \mu + \exp(h_t) \xi_{t+1}
$$

En utilisant ce fait, la solution peut s'écrire comme suit.

Notez que les tirages aléatoires sont conservés à l'intérieur de la boucle interne plutôt que pré-alloués,
afin d'éviter de créer de grands tableaux de chocs de taille `M * n`.


```{code-cell} ipython3
M = 10_000_000

n, β, K = 20, 0.99, 100
μ, ρ, ν, S0, h0 = 0.0001, 0.1, 0.001, 10, 0

@jit(parallel=True)
def compute_call_price_parallel(β=β,
                                μ=μ,
                                S0=S0,
                                h0=h0,
                                K=K,
                                n=n,
                                ρ=ρ,
                                ν=ν,
                                M=M):
    current_sum = 0.0
    # Pour chaque trajectoire d'échantillon
    for m in prange(M):
        s = np.log(S0)
        h = h0
        # Simuler en avançant dans le temps
        for t in range(n):
            s = s + μ + np.exp(h) * np.random.randn()
            h = ρ * h + ν * np.random.randn()
        # Et ajouter la valeur max{S_n - K, 0} à current_sum
        current_sum += max(np.exp(s) - K, 0)

    return β**n * current_sum / M
```

Essayez de basculer entre `parallel=True` et `parallel=False` et notez le temps d'exécution.

Si vous êtes sur une machine avec de nombreux CPU, la différence devrait être significative.

```{solution-end}
```