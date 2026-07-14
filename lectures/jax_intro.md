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
  title: JAX
  headings:
    JAX as a NumPy Replacement: JAX comme remplacement de NumPy
    JAX as a NumPy Replacement::Similarities: Similitudes
    JAX as a NumPy Replacement::Differences: Différences
    JAX as a NumPy Replacement::Differences::Speed!: "La vitesse\_!"
    JAX as a NumPy Replacement::Differences::Speed!::With NumPy: Avec NumPy
    JAX as a NumPy Replacement::Differences::Speed!::With JAX: Avec JAX
    JAX as a NumPy Replacement::Differences::Size Experiment: Expérience sur la taille
    JAX as a NumPy Replacement::Differences::Precision: Précision
    JAX as a NumPy Replacement::Differences::Immutability: Immuabilité
    JAX as a NumPy Replacement::Differences::A Workaround: Une solution de contournement
    Functional Programming: Programmation fonctionnelle
    Functional Programming::Pure functions: Fonctions pures
    Functional Programming::Examples -- Pure and Impure: Exemples -- pures et impures
    Functional Programming::Why Functional Programming?: "Pourquoi la programmation fonctionnelle\_?"
    Random numbers: Nombres aléatoires
    Random numbers::NumPy / MATLAB Approach: Approche NumPy / MATLAB
    Random numbers::JAX: JAX
    Random numbers::Benefits: Avantages
    JIT Compilation: Compilation JIT
    JIT Compilation::With NumPy: Avec NumPy
    JIT Compilation::With JAX: Avec JAX
    JIT Compilation::Compiling the Whole Function: Compiler la fonction entière
    JIT Compilation::How JIT compilation works: Comment fonctionne la compilation JIT
    JIT Compilation::Compiling non-pure functions: Compilation de fonctions non pures
    Vectorization with `vmap`: Vectorisation avec `vmap`
    Vectorization with `vmap`::A simple example: Un exemple simple
    Vectorization with `vmap`::Combining transformations: Combinaison des transformations
    'Automatic differentiation: a preview': "Différenciation automatique\_: un aperçu"
    Exercises: Exercices
---

# JAX

Ce cours propose une brève introduction à [Google JAX](https://github.com/jax-ml/jax).

```{include} _admonition/gpu.md
```

JAX est une bibliothèque de calcul scientifique haute performance qui fournit

* une interface de type [NumPy](https://en.wikipedia.org/wiki/NumPy) capable de paralléliser automatiquement sur les CPU et les GPU,
* un compilateur just-in-time pour accélérer une large gamme d'opérations
  numériques, et
* la [différenciation automatique](https://en.wikipedia.org/wiki/Automatic_differentiation).

De plus en plus, JAX maintient et fournit également des [routines de calcul
scientifique plus spécialisées](https://docs.jax.dev/en/latest/jax.scipy.html), telles que celles que l'on trouve à l'origine dans [SciPy](https://en.wikipedia.org/wiki/SciPy).

En plus de ce qui est inclus dans Anaconda, ce cours nécessitera les bibliothèques suivantes :

```{code-cell} ipython3
:tags: [hide-output]

!pip install jax quantecon
```

Nous utiliserons les importations suivantes

```{code-cell} ipython3
import jax
import jax.numpy as jnp
import matplotlib.pyplot as plt
import numpy as np
import quantecon as qe
```



## JAX comme remplacement de NumPy

Examinons les similitudes et les différences entre JAX et NumPy.

### Similitudes

Ci-dessus, nous importons `jax.numpy as jnp`, qui fournit une interface de type NumPy
pour les opérations sur les tableaux.

L'une des caractéristiques attrayantes de JAX est que, dans la mesure du possible, cette interface
se conforme à l'API de NumPy.

En conséquence, nous pouvons souvent utiliser JAX comme un remplacement direct de NumPy.

Voici quelques opérations standard sur les tableaux utilisant `jnp` :

```{code-cell} ipython3
a = jnp.asarray((1.0, 3.2, -1.5))
```

```{code-cell} ipython3
print(a)
```

```{code-cell} ipython3
print(jnp.sum(a))
```

```{code-cell} ipython3
print(jnp.dot(a, a))
```

Il faut cependant se rappeler que l'objet tableau `a` n'est pas un tableau NumPy :

```{code-cell} ipython3
a
```

```{code-cell} ipython3
type(a)
```

Même les applications à valeurs scalaires sur les tableaux renvoient des tableaux JAX plutôt que des scalaires !

```{code-cell} ipython3
jnp.sum(a)
```



### Différences

Examinons maintenant quelques différences entre les opérations sur les tableaux de JAX et de NumPy.

(jax_speed)=
#### La vitesse !

Une différence majeure est que JAX est plus rapide --- et parfois beaucoup plus rapide.

Pour illustrer cela, supposons que nous voulions évaluer la fonction cosinus en de nombreux points.

```{code-cell}
n = 50_000_000
x = np.linspace(0, 10, n)   # Tableau NumPy
```

##### Avec NumPy

Essayons avec NumPy

```{code-cell}
with qe.Timer():
    # Premier chronométrage NumPy
    y = np.cos(x)
```

Et encore une fois.

```{code-cell}
with qe.Timer():
    # Deuxième chronométrage NumPy
    y = np.cos(x)
```

Ici 

* NumPy utilise un binaire pré-compilé pour appliquer le cosinus à un tableau de flottants
* Le binaire s'exécute sur le CPU de la machine locale


##### Avec JAX

Essayons maintenant avec JAX.

```{code-cell}
x = jnp.linspace(0, 10, n)
```

Chronométrons la même procédure.

```{code-cell}
with qe.Timer():
    # Première exécution
    y = jnp.cos(x)
    # Maintient l'interpréteur jusqu'à ce que l'opération sur le tableau soit terminée
    y.block_until_ready()
```

```{note}
Ci-dessus, la méthode `block_until_ready`
maintient l'interpréteur jusqu'à ce que les résultats du calcul soient renvoyés.
Ceci est nécessaire pour chronométrer l'exécution, car JAX utilise une répartition asynchrone, qui
permet à l'interpréteur Python de prendre de l'avance sur les calculs numériques.
```

Chronométrons-le de nouveau.

```{code-cell}
with qe.Timer():
    # Deuxième exécution
    y = jnp.cos(x)
    # Maintient l'interpréteur
    y.block_until_ready()
```

Sur un GPU, ce code s'exécute beaucoup plus rapidement que son équivalent NumPy.

De plus, en général, la deuxième exécution est plus rapide que la première grâce à la compilation JIT.

Cela est dû au fait que même les fonctions intégrées comme `jnp.cos` sont compilées en JIT --- et la
première exécution inclut le temps de compilation.

Pourquoi JAX voudrait-il compiler en JIT des fonctions intégrées comme `jnp.cos` au lieu de
simplement fournir des versions pré-compilées, comme NumPy ?

La raison est que le compilateur JIT veut se spécialiser sur la *taille* du tableau
utilisé (ainsi que sur le type de données).

La taille importe pour générer du code optimisé, car une parallélisation efficace
nécessite d'adapter la taille de la tâche au matériel disponible.


#### Expérience sur la taille

Nous pouvons vérifier l'affirmation selon laquelle JAX se spécialise sur la taille du tableau en changeant la taille
de l'entrée et en observant les temps d'exécution. 

```{code-cell}
x = jnp.linspace(0, 10, n + 1)
```

```{code-cell}
with qe.Timer():
    # Première exécution
    y = jnp.cos(x)
    # Maintient l'interpréteur
    y.block_until_ready()
```


```{code-cell}
with qe.Timer():
    # Deuxième exécution
    y = jnp.cos(x)
    # Maintient l'interpréteur
    y.block_until_ready()
```

Le temps d'exécution augmente puis diminue à nouveau (ceci sera plus évident sur le GPU).

Ceci est conforme à la discussion ci-dessus -- la première exécution après le changement de taille
du tableau montre la surcharge de compilation.

Une discussion plus approfondie de la compilation JIT est fournie ci-dessous.



#### Précision

Une autre différence entre NumPy et JAX est que JAX utilise par défaut des flottants sur 32 bits.

Cela est dû au fait que JAX est souvent utilisé pour le calcul sur GPU, et la plupart des calculs GPU utilisent des flottants sur 32 bits.

L'utilisation de flottants sur 32 bits peut entraîner des gains de vitesse significatifs avec une faible perte de précision.

Cependant, pour certains calculs, la précision importe.

Dans ces cas, on peut imposer les flottants sur 64 bits via la commande 

```{code-cell} ipython3
jax.config.update("jax_enable_x64", True)
```

Vérifions que cela fonctionne :

```{code-cell} ipython3
jnp.ones(3)
```


#### Immuabilité

En tant que remplacement de NumPy, une différence plus significative est que les tableaux sont traités comme **immuables**.

Par exemple, avec NumPy nous pouvons écrire

```{code-cell} ipython3
a = np.linspace(0, 1, 3)
a
```

puis muter les données en mémoire :

```{code-cell} ipython3
a[0] = 1
a
```

Dans JAX, cela échoue 😱.


```{code-cell} ipython3
a = jnp.linspace(0, 1, 3)
a
```

```{code-cell} ipython3
try:
    a[0] = 1
except Exception as e:
    print(e)

```

Les concepteurs de JAX ont choisi de rendre les tableaux immuables parce que 

1. JAX utilise un *style de programmation fonctionnelle* et
2. la programmation fonctionnelle évite généralement les données mutables

Nous discutons de ces idées {ref}`ci-dessous <jax_func>`.


(jax_at_workaround)=
#### Une solution de contournement

JAX fournit bien une alternative directe à la modification de tableau sur place
via la [méthode `at`](https://docs.jax.dev/en/latest/_autosummary/jax.numpy.ndarray.at.html).

```{code-cell} ipython3
a = jnp.linspace(0, 1, 3)
```

Appliquer `at[0].set(1)` renvoie une nouvelle copie de `a` dont le premier élément est fixé à 1

```{code-cell} ipython3
a = a.at[0].set(1)
a
```

Évidemment, l'utilisation de `at` présente des inconvénients :

* La syntaxe est lourde et 
* nous voulons éviter de créer de nouveaux tableaux en mémoire à chaque fois que nous changeons une seule valeur !

Par conséquent, la plupart du temps, nous essayons d'éviter cette syntaxe.

(Bien qu'elle puisse en fait être efficace à l'intérieur des fonctions compilées en JIT -- mais laissons cela de côté pour l'instant.)


(jax_func)=
## Programmation fonctionnelle

D'après la documentation de JAX :

*Lorsque vous vous promenez dans la campagne italienne, les gens n'hésiteront pas à vous dire que JAX a « una anima di pura programmazione funzionale ».*

En d'autres termes, JAX suppose un
style de [programmation fonctionnelle](https://en.wikipedia.org/wiki/Functional_programming).

### Fonctions pures

L'implication majeure est que les fonctions JAX doivent être pures.

Les [fonctions pures](https://en.wikipedia.org/wiki/Pure_function) ont les caractéristiques suivantes :

1. *Déterministes*
2. *Sans effets secondaires*

[Déterministe](https://en.wikipedia.org/wiki/Deterministic_algorithm) signifie

*  Même entrée $\implies$ même sortie
*  Les sorties ne dépendent pas de l'état global

En particulier, les fonctions pures renverront toujours le même résultat si elles sont invoquées avec les mêmes entrées.

[Sans effets secondaires](https://en.wikipedia.org/wiki/Side_effect_(computer_science)) signifie que la fonction

* Ne modifiera pas l'état global
* Ne modifiera pas les données passées à la fonction (données immuables)



### Exemples -- pures et impures

Voici un exemple d'une fonction *impure*

```{code-cell} ipython3
tax_rate = 0.1

def add_tax(prices):
    for i, price in enumerate(prices):
        prices[i] = price * (1 + tax_rate)

prices = [10.0, 20.0]
add_tax(prices)
prices
```

Cette fonction n'est pas pure parce que

* effets secondaires --- elle modifie la variable globale `prices`
* non déterministe --- un changement de la variable globale `tax_rate` modifiera
  les sorties de la fonction, même avec le même tableau d'entrée `prices`.

Voici une version *pure*

```{code-cell} ipython3

def add_tax_pure(prices, tax_rate):
    new_prices = [price * (1 + tax_rate) for price in prices]
    return new_prices

tax_rate = 0.1
prices = (10.0, 20.0)
after_tax_prices = add_tax_pure(prices, tax_rate)
after_tax_prices
```

Elle est pure parce que 

* toutes les dépendances sont explicites via les arguments de la fonction
* et elle ne modifie aucun état externe 


### Pourquoi la programmation fonctionnelle ?

Chez QuantEcon, nous adorons les fonctions pures parce qu'elles

* Facilitent les tests : chaque fonction peut fonctionner de manière isolée
* Favorisent un comportement déterministe et donc la reproductibilité
* Empêchent les bugs qui découlent de la mutation d'un état partagé

Le compilateur JAX adore les fonctions pures et la programmation fonctionnelle parce que

* Les dépendances de données sont explicites, ce qui aide à optimiser les calculs complexes
* Les fonctions pures sont plus faciles à différencier (autodiff)
* Les fonctions pures sont plus faciles à paralléliser et à optimiser (elles ne dépendent pas d'un état mutable partagé)

Une autre façon de voir cela est la suivante :

JAX représente les fonctions sous forme de graphes de calcul, qui sont ensuite compilés ou
transformés (par ex., différenciés)

Ces graphes de calcul décrivent comment un ensemble donné d'entrées est transformé en une sortie.

Les graphes de calcul de JAX sont purs par construction.

JAX utilise un style de programmation fonctionnelle afin que les fonctions construites par l'utilisateur se traduisent
directement dans les représentations théoriques des graphes prises en charge par JAX.


## Nombres aléatoires

La génération de nombres aléatoires dans JAX diffère significativement des schémas que l'on trouve dans NumPy ou MATLAB.



### Approche NumPy / MATLAB

Dans NumPy / MATLAB, la génération fonctionne en maintenant un état global caché.

```{code-cell} ipython3
np.random.seed(42)
print(np.random.randn(2))   
```

Chaque fois que nous appelons une fonction aléatoire, l'état caché est mis à jour :

```{code-cell} ipython3
print(np.random.randn(2)) 
```

Cette fonction n'est *pas pure* parce que :

* Elle est non déterministe : mêmes entrées, sorties différentes
* Elle a des effets secondaires : elle modifie l'état du générateur de nombres aléatoires global

Cela est dangereux sous parallélisation --- il faut contrôler soigneusement ce qui se passe dans chaque
thread.


### JAX


Dans JAX, l'état du générateur de nombres aléatoires est contrôlé explicitement.

Nous produisons d'abord une clé, qui initialise le générateur de nombres aléatoires.

```{code-cell} ipython3
seed = 1234
key = jax.random.key(seed)
```

Nous pouvons maintenant utiliser la clé pour générer des nombres aléatoires :

```{code-cell} ipython3
x = jax.random.normal(key, (3, 3))
x
```

Si nous utilisons à nouveau la même clé, nous initialisons à la même graine, donc les nombres aléatoires sont les mêmes :

```{code-cell} ipython3
jax.random.normal(key, (3, 3))
```

Pour produire un tirage (quasi-) indépendant, une option est de « diviser » (split) la clé existante :

```{code-cell} ipython3
key, subkey = jax.random.split(key)
```

```{code-cell} ipython3
jax.random.normal(key, (3, 3))
```

```{code-cell} ipython3
jax.random.normal(subkey, (3, 3))
```

Le diagramme suivant illustre comment `split` produit un arbre de clés à partir d'une
racine unique, chaque clé générant des tirages aléatoires indépendants.

```{code-cell} ipython3
:tags: [hide-input]

fig, ax = plt.subplots(figsize=(8, 4))
ax.set_xlim(-0.5, 6.5)
ax.set_ylim(-0.5, 3.5)
ax.set_aspect('equal')
ax.axis('off')

box_style = dict(boxstyle="round,pad=0.3", facecolor="white",
                 edgecolor="black", linewidth=1.5)
box_used = dict(boxstyle="round,pad=0.3", facecolor="#d4edda",
                edgecolor="black", linewidth=1.5)

# Clé racine
ax.text(3, 3, "key₀", ha='center', va='center', fontsize=11,
        bbox=box_style)

# Niveau 1
ax.annotate("", xy=(1.5, 2), xytext=(3, 2.7),
            arrowprops=dict(arrowstyle="->", lw=1.5))
ax.annotate("", xy=(4.5, 2), xytext=(3, 2.7),
            arrowprops=dict(arrowstyle="->", lw=1.5))
ax.text(1.5, 2, "key₁", ha='center', va='center', fontsize=11,
        bbox=box_style)
ax.text(4.5, 2, "subkey₁", ha='center', va='center', fontsize=11,
        bbox=box_used)
ax.text(5.7, 2, "→ tirage", ha='left', va='center', fontsize=10,
        color='green')

# Étiqueter le split
ax.text(2, 2.65, "split", ha='center', va='center', fontsize=9,
        fontstyle='italic', color='gray')

# Niveau 2
ax.annotate("", xy=(0.5, 1), xytext=(1.5, 1.7),
            arrowprops=dict(arrowstyle="->", lw=1.5))
ax.annotate("", xy=(2.5, 1), xytext=(1.5, 1.7),
            arrowprops=dict(arrowstyle="->", lw=1.5))
ax.text(0.5, 1, "key₂", ha='center', va='center', fontsize=11,
        bbox=box_style)
ax.text(2.5, 1, "subkey₂", ha='center', va='center', fontsize=11,
        bbox=box_used)
ax.text(3.7, 1, "→ tirage", ha='left', va='center', fontsize=10,
        color='green')

ax.text(0.7, 1.65, "split", ha='center', va='center', fontsize=9,
        fontstyle='italic', color='gray')

# Niveau 3
ax.annotate("", xy=(0, 0), xytext=(0.5, 0.7),
            arrowprops=dict(arrowstyle="->", lw=1.5))
ax.annotate("", xy=(1.5, 0), xytext=(0.5, 0.7),
            arrowprops=dict(arrowstyle="->", lw=1.5))
ax.text(0, 0, "key₃", ha='center', va='center', fontsize=11,
        bbox=box_style)
ax.text(1.5, 0, "subkey₃", ha='center', va='center', fontsize=11,
        bbox=box_used)
ax.text(2.7, 0, "→ tirage", ha='left', va='center', fontsize=10,
        color='green')
ax.text(0, 0.65, "split", ha='center', va='center', fontsize=9,
        fontstyle='italic', color='gray')

ax.text(3, -0.5, "⋮", ha='center', va='center', fontsize=14)

ax.set_title("Arbre de division des clés PRNG", fontsize=13, pad=10)
plt.tight_layout()
plt.show()
```

Cette syntaxe semblera inhabituelle pour un utilisateur de NumPy ou Matlab --- mais elle prendra plus de
sens lorsque nous aborderons la programmation parallèle.

La fonction ci-dessous produit `k` matrices `n x n` aléatoires (quasi-) indépendantes en utilisant `split`.

```{code-cell} ipython3
def gen_random_matrices(
        key,   # Clé JAX pour les nombres aléatoires
        n=2,   # Les matrices seront n x n
        k=3    # Nombre de matrices à générer
    ):
    matrices = []
    for _ in range(k):
        key, subkey = jax.random.split(key)
        A = jax.random.uniform(subkey, (n, n))
        matrices.append(A)
    return matrices
```

```{code-cell} ipython3
seed = 42
key = jax.random.key(seed)
gen_random_matrices(key)
```

Cette fonction est *pure*

* Déterministe : mêmes entrées, même sortie
* Sans effets secondaires : aucun état caché n'est modifié


### Avantages

Comme mentionné ci-dessus, cette explicitude est précieuse :

* Reproductibilité : facile de reproduire les résultats en réutilisant les clés
* Parallélisation : contrôle de ce qui se passe sur des threads séparés 
* Débogage : l'absence d'état caché rend le code plus facile à tester
* Compatibilité JIT : le compilateur peut optimiser les fonctions pures de manière plus agressive


## Compilation JIT

Le compilateur just-in-time (JIT) de JAX accélère l'exécution en générant
un code machine efficace qui varie selon la taille de la tâche et le matériel.

Nous avons vu la puissance du compilateur JIT de JAX combiné à du matériel parallèle
{ref}`ci-dessus <jax_speed>`, lorsque nous avons appliqué `cos` à un grand tableau.

Ici, nous étudions la compilation JIT pour des fonctions plus complexes


### Avec NumPy

Nous essaierons d'abord avec NumPy, en utilisant

```{code-cell}
def f(x):
    y = np.cos(2 * x**2) + np.sqrt(np.abs(x)) + 2 * np.sin(x**4) - x**2
    return y
```

Exécutons avec un grand `x`

```{code-cell}
n = 50_000_000
x = np.linspace(0, 10, n)
```

```{code-cell}
with qe.Timer():
    # Chronométrer le code NumPy
    y = f(x)
```

Modèle d'exécution **impatient** (eager)

* Chaque opération est exécutée immédiatement dès qu'elle est rencontrée, matérialisant son
  résultat avant que l'opération suivante ne commence.

Inconvénients

* Parallélisation minimale 
* Empreinte mémoire lourde --- produit de nombreux tableaux intermédiaires
* Beaucoup de lectures/écritures en mémoire



### Avec JAX

En première approche, nous remplaçons `np` par `jnp` partout :

```{code-cell}
def f(x):
    y = jnp.cos(2 * x**2) + jnp.sqrt(jnp.abs(x)) + 2 * jnp.sin(x**4) - x**2
    return y


x = jnp.linspace(0, 10, n)
```

Chronométrons-le maintenant.

```{code-cell}
with qe.Timer():
    # Premier appel
    y = f(x)
    # Maintient l'interpréteur
    jax.block_until_ready(y);
```

```{code-cell}
with qe.Timer():
    # Deuxième appel
    y = f(x)
    # Maintient l'interpréteur
    jax.block_until_ready(y);
```

Le résultat est similaire à l'exemple `cos` --- JAX est plus rapide, en particulier à la
deuxième exécution après la compilation JIT.

Cela est dû au fait que les opérations individuelles sur les tableaux sont parallélisées sur le GPU

Mais nous utilisons toujours l'exécution impatiente 

* beaucoup de mémoire à cause des tableaux intermédiaires
* beaucoup de lectures/écritures en mémoire

De plus, de nombreux noyaux séparés sont lancés sur le GPU

### Compiler la fonction entière

Heureusement, avec JAX, nous avons un autre tour dans notre sac --- nous pouvons compiler en JIT
la fonction entière, et pas seulement les opérations individuelles.

Le compilateur fusionne toutes les opérations sur les tableaux en un seul noyau optimisé

Essayons ceci avec la fonction `f` :

```{code-cell}
f_jax = jax.jit(f)
```

```{code-cell}
with qe.Timer():
    # Première exécution
    y = f_jax(x)
    # Maintient l'interpréteur
    jax.block_until_ready(y);
```

```{code-cell}
with qe.Timer():
    # Deuxième exécution
    y = f_jax(x)
    # Maintient l'interpréteur
    jax.block_until_ready(y);
```

Le temps d'exécution s'est encore amélioré --- maintenant parce que nous avons fusionné toutes les opérations

* Optimisation agressive basée sur la séquence de calcul entière
* Élimination des multiples appels à l'accélérateur matériel 

L'empreinte mémoire est également bien plus faible --- aucune création de tableaux intermédiaires

Au passage, une syntaxe plus courante lorsqu'on cible une fonction pour le compilateur JIT
est

```{code-cell} ipython3
@jax.jit
def f(x):
    pass # placer le corps de la fonction ici
```

### Comment fonctionne la compilation JIT

Lorsque nous appliquons `jax.jit` à une fonction, JAX la *trace* : au lieu d'exécuter
les opérations immédiatement, il enregistre la séquence d'opérations sous forme de
graphe de calcul et transmet ce graphe au compilateur
[XLA](https://openxla.org/xla).

XLA fusionne et optimise ensuite les opérations en un seul noyau compilé
adapté au matériel disponible (CPU, GPU ou TPU).

Le premier appel à une fonction compilée en JIT entraîne une surcharge de compilation, mais
les appels ultérieurs avec les mêmes formes et types d'entrée réutilisent le code compilé
mis en cache et s'exécutent à pleine vitesse.


### Compilation de fonctions non pures

Bien que JAX ne génère généralement pas d'erreurs lors de la compilation de fonctions impures,
l'exécution devient imprévisible !

Voici une illustration de ce fait :

```{code-cell} ipython3
a = 1  # global

@jax.jit
def f(x):
    return a + x
```

```{code-cell} ipython3
x = jnp.ones(2)
```

```{code-cell} ipython3
f(x)
```

Dans le code ci-dessus, la valeur globale `a=1` est fusionnée dans la fonction jittée.

Même si nous changeons `a`, la sortie de `f` ne sera pas affectée --- tant que la même version compilée est appelée.

```{code-cell} ipython3
a = 42
```

```{code-cell} ipython3
f(x)
```

Changer la dimension de l'entrée déclenche une nouvelle compilation de la fonction, moment auquel le changement de la valeur de `a` prend effet :

```{code-cell} ipython3
x = jnp.ones(3)
```

```{code-cell} ipython3
f(x)
```

Morale de l'histoire : écrivez des fonctions pures lorsque vous utilisez JAX !




## Vectorisation avec `vmap`

Une autre transformation puissante de JAX est `jax.vmap`, qui vectorise automatiquement
une fonction écrite pour une entrée unique afin qu'elle opère sur des
lots.

Cela évite d'avoir à écrire manuellement du code vectorisé ou d'utiliser des boucles explicites.

### Un exemple simple

Supposons que nous ayons une fonction qui calcule la différence entre la moyenne et la médiane pour un tableau de nombres.

```{code-cell} ipython3
def mm_diff(x):
    return jnp.mean(x) - jnp.median(x)
```

Nous pouvons l'appliquer à un seul vecteur :

```{code-cell} ipython3
x = jnp.array([1.0, 2.0, 5.0])
mm_diff(x)
```

Supposons maintenant que nous ayons une matrice et que nous voulions calculer ces statistiques pour chaque ligne.

Sans `vmap`, nous aurions besoin d'une boucle explicite :

```{code-cell} ipython3
X = jnp.array([[1.0, 2.0, 5.0],
               [4.0, 5.0, 6.0],
               [1.0, 8.0, 9.0]])

for row in X:
    print(mm_diff(row))
```

Cependant, les boucles Python sont lentes et ne peuvent pas être compilées ou
parallélisées efficacement par JAX.

Avec `vmap`, nous pouvons éviter les boucles et garder le calcul sur l'accélérateur :

```{code-cell} ipython3
batch_mm_diff = jax.vmap(mm_diff)    # Crée une nouvelle version « vectorisée »
batch_mm_diff(X)                     # Applique à chaque ligne de X
```


### Combinaison des transformations

L'une des forces de JAX est que les transformations se composent naturellement.

Par exemple, nous pouvons compiler en JIT une fonction vectorisée :

```{code-cell} ipython3
fast_batch_mm_diff = jax.jit(jax.vmap(mm_diff))
fast_batch_mm_diff(X)
```

Cette composition de `jit`, `vmap`, et (comme nous le verrons ensuite) `grad` est centrale dans
la conception de JAX et le rend particulièrement puissant pour le calcul scientifique et
l'apprentissage automatique.


## Différenciation automatique : un aperçu

JAX peut utiliser la différenciation automatique pour calculer des gradients.

Cela peut être extrêmement utile pour l'optimisation et la résolution de systèmes non linéaires.

Voici une illustration simple faisant intervenir la fonction $f(x) = x^2 / 2$ :

```{code-cell} ipython3
def f(x):
    return (x**2) / 2

f_prime = jax.grad(f)
```

```{code-cell} ipython3
f_prime(10.0)
```

Traçons la fonction et sa dérivée, en notant que $f'(x) = x$.

```{code-cell} ipython3
fig, ax = plt.subplots()
x_grid = jnp.linspace(-4, 4, 200)
ax.plot(x_grid, f(x_grid), label="$f$")
ax.plot(x_grid, [f_prime(x) for x in x_grid], label="$f'$")
ax.legend(loc='upper center')
plt.show()
```

La différenciation automatique est un sujet vaste avec de nombreuses applications en économie
et en finance.  Nous en fournissons un traitement plus approfondi dans {doc}`notre cours sur
l'autodiff <autodiff>`.


## Exercices


```{exercise-start}
:label: jax_intro_ex2
```

Dans la section Exercices de {doc}`notre cours sur Numba <numba>`, nous avons {ref}`utilisé Monte-Carlo
pour évaluer une option d'achat européenne <numba_ex4>`.

Le code a été accéléré par le multithreading basé sur Numba.

Essayez d'écrire une version de cette opération pour JAX, en utilisant tous les mêmes
paramètres.



```{exercise-end}
```


```{solution-start} jax_intro_ex2
:class: dropdown
```
Voici une solution :

```{code-cell} ipython3
M = 10_000_000

n, β, K = 20, 0.99, 100
μ, ρ, ν, S0, h0 = 0.0001, 0.1, 0.001, 10, 0

@jax.jit
def compute_call_price_jax(β=β,
                           μ=μ,
                           S0=S0,
                           h0=h0,
                           K=K,
                           n=n,
                           ρ=ρ,
                           ν=ν,
                           M=M,
                           key=jax.random.key(1)):

    s = jnp.full(M, np.log(S0))
    h = jnp.full(M, h0)

    def update(i, loop_state):
        s, h, key = loop_state
        key, subkey = jax.random.split(key)
        Z = jax.random.normal(subkey, (2, M))
        s = s + μ + jnp.exp(h) * Z[0, :]
        h = ρ * h + ν * Z[1, :]
        new_loop_state = s, h, key
        return new_loop_state

    initial_loop_state = s, h, key
    final_loop_state = jax.lax.fori_loop(0, n, update, initial_loop_state)
    s, h, key = final_loop_state

    expectation = jnp.mean(jnp.maximum(jnp.exp(s) - K, 0))

    return β**n * expectation
```

```{note}
Nous utilisons `jax.lax.fori_loop` au lieu d'une boucle `for` Python.
Cela permet à JAX de compiler la boucle efficacement sans la dérouler,
ce qui réduit considérablement le temps de compilation pour les grands tableaux.
```

Exécutons-le une fois pour le compiler :

```{code-cell} ipython3
with qe.Timer():
    compute_call_price_jax().block_until_ready()
```

Et maintenant chronométrons-le :

```{code-cell} ipython3
with qe.Timer():
    compute_call_price_jax().block_until_ready()
```

```{solution-end}
```