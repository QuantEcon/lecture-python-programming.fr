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
  title: Python pour le calcul scientifique
  headings:
    Overview: Vue d'ensemble
    Major Scientific Libraries: Principales bibliothèques scientifiques
    Major Scientific Libraries::Why do we need them?: "Pourquoi en avons-nous besoin\_?"
    Major Scientific Libraries::Python's Scientific Ecosystem: L'écosystème scientifique de Python
    Why is Pure Python Slow?: "Pourquoi le Python pur est-il lent\_?"
    Why is Pure Python Slow?::Type Checking: Vérification de type
    Why is Pure Python Slow?::Type Checking::Dynamic typing: Typage dynamique
    Why is Pure Python Slow?::Type Checking::Static types: Types statiques
    Why is Pure Python Slow?::Data Access: Accès aux données
    Why is Pure Python Slow?::Data Access::Summing with Compiled Code: Sommation avec du code compilé
    Why is Pure Python Slow?::Data Access::Summing in Pure Python: Sommation en Python pur
    Why is Pure Python Slow?::Summary: Résumé
    Accelerating Python: Accélérer Python
    Accelerating Python::Vectorization: Vectorisation
    Accelerating Python::Vectorization vs pure Python loops: Vectorisation vs boucles Python pures
    Accelerating Python::JIT compilers: Compilateurs JIT
    Parallelization: Parallélisation
    Parallelization::Parallelization on CPUs: Parallélisation sur les CPU
    Parallelization::Parallelization on CPUs::Multithreading: Multithreading
    Parallelization::Parallelization on CPUs::Multiprocessing: Multiprocessing
    Parallelization::Parallelization on CPUs::Which Should We Use?: "Que devrions-nous utiliser\_?"
    Parallelization::Hardware Accelerators: Accélérateurs matériels
    Parallelization::Accessing GPU Resources: Accéder aux ressources GPU
---

(speed)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Python pour le calcul scientifique

```{epigraph}
« Nous devrions oublier les petites optimisations, disons environ 97 % du temps :
l'optimisation prématurée est la source de tous les maux. » -- Donald Knuth
```

## Vue d'ensemble

Python est le langage le plus populaire pour de nombreux aspects du calcul scientifique.

Cela s'explique par

* la nature accessible et expressive du langage lui-même,
* l'immense gamme de bibliothèques scientifiques de haute qualité,
* le fait que le langage et les bibliothèques soient open source,
* le rôle central que joue Python dans la science des données, l'apprentissage automatique et l'IA.

Dans les cours précédents, nous avons utilisé quelques bibliothèques scientifiques Python, notamment NumPy et Matplotlib.

Cependant, notre attention principale portait sur le langage Python de base, plutôt que sur les bibliothèques.

Nous nous tournons maintenant vers les bibliothèques scientifiques et leur accordons toute notre attention.

Dans ce cours introductif, nous aborderons les sujets suivants :

1. Quels sont les principaux éléments de l'écosystème scientifique Python ?
1. Comment s'articulent-ils entre eux ?
1. Comment la situation évolue-t-elle au fil du temps ?

En plus de ce qui est inclus dans Anaconda, ce cours nécessitera

```{code-cell} ipython
---
tags: [hide-output]
---
!pip install quantecon
```

Commençons par quelques importations :

```{code-cell} ipython
import numpy as np
import quantecon as qe
import matplotlib.pyplot as plt
import random
```


## Principales bibliothèques scientifiques

Passons brièvement en revue les bibliothèques scientifiques de Python.


### Pourquoi en avons-nous besoin ?

Nous avons besoin des bibliothèques scientifiques de Python pour deux raisons :

1. Python est petit
2. Python est lent

**Python est petit**

Le cœur de Python est petit par conception -- cela facilite l'optimisation, l'accessibilité et la maintenance

Les bibliothèques scientifiques fournissent les routines que nous ne voulons pas -- et ne devrions probablement pas -- écrire nous-mêmes

* intégration numérique, interpolation, algèbre linéaire, recherche de racines, etc.

**Python est lent**

Une autre raison pour laquelle nous avons besoin des bibliothèques scientifiques est que le Python pur est relativement lent.

Les bibliothèques scientifiques accélèrent l'exécution en utilisant trois stratégies principales :

1. Vectorisation : fourniture de code machine compilé et d'interfaces qui rendent ce code accessible
1. Compilation JIT : compilateurs qui convertissent des instructions de type Python en code machine rapide au moment de l'exécution
2. Parallélisation : répartition des tâches sur plusieurs threads / CPU / GPU / TPU

Nous discuterons de ces idées en profondeur ci-dessous.


### L'écosystème scientifique de Python

À QuantEcon, les bibliothèques scientifiques que nous utilisons le plus souvent sont

* [NumPy](https://numpy.org/)
* [SciPy](https://scipy.org/)
* [Matplotlib](https://matplotlib.org/)
* [JAX](https://github.com/jax-ml/jax)
* [Pandas](https://pandas.pydata.org/)
* [Numba](https://numba.pydata.org/)

Voici comment elles s'articulent :

* NumPy pose les fondations en fournissant un type de données tableau de base (pensez aux
  vecteurs et aux matrices) et des fonctions pour agir sur ces tableaux (par exemple, la multiplication
  matricielle).
* SciPy s'appuie sur NumPy en ajoutant des méthodes numériques couramment utilisées en science (interpolation, optimisation, recherche de racines, etc.).
* Matplotlib est utilisé pour générer des figures, en mettant l'accent sur le tracé de données stockées dans des tableaux NumPy.
* JAX inclut des opérations de traitement de tableaux similaires à NumPy, la différentiation
  automatique, un compilateur juste-à-temps centré sur la parallélisation, et une intégration automatisée avec des accélérateurs matériels tels que les
  GPU.
* Pandas fournit des types et des fonctions pour manipuler les données.
* Numba fournit un compilateur juste-à-temps qui fonctionne bien avec NumPy et aide à accélérer le code Python.

Nous discuterons de toutes ces bibliothèques en détail dans cette série de cours.


## Pourquoi le Python pur est-il lent ?

Comme mentionné ci-dessus, le code numérique écrit en Python pur est relativement lent.

Essayons de comprendre ce qui explique les vitesses d'exécution lentes.

### Vérification de type

Une source de surcoût dans les opérations en Python pur est la vérification de type.

Essayons de comprendre les enjeux.

#### Typage dynamique

```{index} single: Dynamic Typing
```

Considérons cette opération Python

```{code-cell} python3
a, b = 10, 10
a + b
```

Même pour cette opération simple, l'interpréteur Python a pas mal de travail à faire.

Par exemple, dans l'instruction `a + b`, l'interpréteur doit savoir quelle
opération invoquer.

Si `a` et `b` sont des chaînes de caractères, alors `a + b` nécessite une concaténation de chaînes

```{code-cell} python3
a, b = 'foo', 'bar'
a + b
```

Si `a` et `b` sont des listes, alors `a + b` nécessite une concaténation de listes

```{code-cell} python3
a, b = ['foo'], ['bar']
a + b
```


En conséquence, lors de l'exécution de `a + b`, Python doit d'abord vérifier le type des
objets, puis appeler l'opération correcte.

Cela implique un surcoût.

Si nous exécutons cette expression de manière répétée dans une boucle serrée, le surcoût devient important.


#### Types statiques

```{index} single: Static Types
```

Les langages compilés évitent ces surcoûts avec des types explicites et statiques.

Par exemple, considérons le code C suivant, qui additionne les entiers de 1 à 10

```{code-block} c
:class: no-execute

#include <stdio.h>

int main(void) {
    int i;
    int sum = 0;
    for (i = 1; i <= 10; i++) {
        sum = sum + i;
    }
    printf("sum = %d\n", sum);
    return 0;
}
```

Les variables `i` et `sum` sont explicitement déclarées comme des entiers.

De plus, lorsque nous faisons une instruction telle que `int i`, nous faisons une promesse au compilateur
que `i` sera *toujours* un entier, tout au long de l'exécution du programme.

De ce fait, la signification de l'addition dans l'expression `sum + i` est totalement sans ambiguïté.

Il n'y a pas besoin de vérification de type et donc pas de surcoût.


### Accès aux données

Un autre frein à la vitesse pour les langages de haut niveau est l'accès aux données.

Pour illustrer, considérons le problème de la sommation de données --- disons, une collection d'entiers.

#### Sommation avec du code compilé

En C ou en Fortran, un tableau d'entiers est stocké dans un seul bloc contigu de mémoire

* Par exemple, un entier de 64 bits est stocké dans 8 octets de mémoire.
* Un tableau de $n$ tels entiers occupe $8n$ octets consécutifs.

De plus, le type de données est connu au moment de la compilation.

Ainsi, chaque point de données successif peut être accédé en avançant dans l'espace mémoire
d'une quantité connue et fixe.


#### Sommation en Python pur

Python essaie de reproduire ces idées dans une certaine mesure.

Par exemple, dans l'implémentation standard de Python (CPython), les éléments d'une liste sont
placés dans des emplacements mémoire qui sont dans un certain sens contigus.

Cependant, ces éléments de liste ressemblent davantage à des pointeurs vers des données qu'à des données réelles.

Ainsi, il y a toujours un surcoût impliqué dans l'accès aux valeurs de données elles-mêmes.

Un tel surcoût est un coupable majeur en ce qui concerne l'exécution lente.


### Résumé

La discussion ci-dessus signifie-t-elle que nous devrions simplement passer à C ou Fortran pour tout ?

La réponse est : Certainement pas !

Pour tout programme donné, relativement peu de lignes seront jamais critiques en termes de temps.

Il est donc bien plus efficace d'écrire la majeure partie de notre code dans un langage à haute productivité comme Python.

De plus, même pour les lignes de code qui *sont* critiques en termes de temps, nous pouvons désormais
égaler ou surpasser les binaires compilés à partir de C ou Fortran en utilisant les bibliothèques scientifiques de Python.

À ce propos, nous soulignons que, ces dernières années, accélérer le code est devenu essentiellement
synonyme de parallélisation.

Cette tâche est mieux laissée aux compilateurs spécialisés !



## Accélérer Python

Dans cette section, nous examinons trois techniques connexes pour accélérer le code Python.

Ici, nous nous concentrerons sur les idées fondamentales.

Plus tard, nous examinerons des bibliothèques spécifiques et comment elles implémentent ces idées.



### {index}`Vectorisation <single: Vectorization>`

```{index} single: Python; Vectorization
```

Une méthode pour éviter le trafic mémoire et la vérification de type est la [programmation
par tableaux](https://en.wikipedia.org/wiki/Array_programming).

De nombreux économistes désignent généralement la programmation par tableaux par le terme « vectorisation ».

```{note}
En informatique, ce terme a [une signification légèrement différente](https://en.wikipedia.org/wiki/Automatic_vectorization).
```

L'idée clé est d'envoyer les opérations de traitement de tableaux par lots à du code machine natif précompilé et
efficace.

Le code machine lui-même est généralement compilé à partir de C ou de
Fortran soigneusement optimisé.

Par exemple, lorsqu'on travaille dans un langage de haut niveau, l'opération d'inversion d'une
grande matrice peut être sous-traitée à du code machine efficace qui est précompilé
à cette fin et fourni aux utilisateurs dans le cadre d'un package.

Les principaux avantages sont

1. la vérification de type est payée *par tableau*, plutôt que par élément, et
1. les tableaux contenant des éléments du même type de données sont efficaces en termes
   d'accès mémoire.

L'idée de la vectorisation remonte à MATLAB, qui utilise la vectorisation de manière intensive.


```{figure} /_static/lecture_specific/need_for_speed/matlab.png
```

NumPy utilise un modèle similaire, inspiré de MATLAB


### Vectorisation vs boucles Python pures

Essayons une comparaison rapide de vitesse pour illustrer comment la vectorisation peut
accélérer le code.

Voici du code non vectorisé, qui utilise une boucle Python native pour générer,
mettre au carré, puis additionner un grand nombre de variables aléatoires :

```{code-cell} python3
n = 1_000_000
```

```{code-cell} python3
with qe.Timer():
    y = 0      # Accumulera et stockera la somme
    for i in range(n):
        x = random.uniform(0, 1)
        y += x**2
```

Le code vectorisé suivant utilise NumPy, que nous étudierons bientôt en détail,
pour parvenir au même résultat.

```{code-cell} ipython
rng = np.random.default_rng()
with qe.Timer():
    x = rng.uniform(0, 1, n)
    y = np.sum(x**2)
```

Comme vous pouvez le voir, le deuxième bloc de code s'exécute beaucoup plus rapidement.

Il décompose la boucle en trois opérations de base

1. tirer `n` uniformes
1. les mettre au carré
1. les additionner

Ces opérations sont envoyées comme opérateurs par lots à du code machine optimisé.




(numba-p_c_vectorization)=
### Compilateurs JIT

Au mieux, la vectorisation produit du code rapide et simple.

Cependant, elle n'est pas sans inconvénients.

Un problème est qu'elle peut être très gourmande en mémoire.

Cela est dû au fait que la vectorisation tend à créer de nombreux tableaux intermédiaires avant
de produire le calcul final.

Un autre problème est que tous les algorithmes ne peuvent pas être vectorisés.

En raison de ces problèmes, la plupart du calcul haute performance s'éloigne de
la vectorisation traditionnelle et se tourne vers l'utilisation de [compilateurs
juste-à-temps](https://en.wikipedia.org/wiki/Just-in-time_compilation).

Dans les cours ultérieurs de cette série, nous apprendrons comment les bibliothèques
Python modernes exploitent les compilateurs juste-à-temps pour générer du code machine rapide, efficace et
parallélisé.




## Parallélisation

La croissance de la fréquence d'horloge des CPU (c'est-à-dire la vitesse à laquelle une seule chaîne logique
peut être exécutée) a considérablement ralenti ces dernières années.

Les concepteurs de puces et les programmeurs ont répondu à ce ralentissement en
cherchant une voie différente vers une exécution rapide : la parallélisation.

Cela implique

1. d'augmenter le nombre de CPU intégrés dans chaque machine
1. de connecter des accélérateurs matériels tels que les GPU et les TPU

Pour les programmeurs, le défi a été d'exploiter ce matériel
en exécutant de nombreux processus en parallèle.

Ci-dessous, nous discutons de la parallélisation pour le calcul scientifique, en mettant l'accent sur

1. les outils de parallélisation en Python et
1. comment ces outils peuvent être appliqués à des problèmes économiques quantitatifs.


### Parallélisation sur les CPU

Passons en revue les deux principaux types de parallélisation basée sur les CPU couramment utilisés dans
le calcul scientifique et discutons de leurs avantages et inconvénients.


#### Multithreading

Le multithreading signifie exécuter plusieurs threads d'exécution au sein d'un seul processus.

Tous les threads partagent le même espace mémoire, ils peuvent donc lire et écrire dans les mêmes tableaux sans copier de données.

Par exemple, lorsqu'une opération numérique sur un grand tableau s'exécute sur un ordinateur portable moderne, la charge de travail peut être répartie sur les multiples cœurs de CPU de la machine, chaque cœur traitant une portion du tableau.

```{note}
Le Python natif peine à implémenter le multithreading en raison de certaines [caractéristiques de conception
héritées](https://wiki.python.org/moin/GlobalInterpreterLock).
Mais ce n'est pas une restriction pour les bibliothèques scientifiques comme NumPy et Numba.
Les fonctions importées de ces bibliothèques et le code compilé en JIT s'exécutent dans des environnements
d'exécution de bas niveau où les restrictions héritées de Python ne s'appliquent pas.
```


#### Multiprocessing

Le multiprocessing signifie exécuter plusieurs processus indépendants, chacun avec son propre espace mémoire séparé.

Comme la mémoire n'est pas partagée, les processus communiquent en s'échangeant des données.

Le multiprocessing peut s'exécuter sur une seule machine ou être distribué sur un cluster de machines connectées par un réseau.


#### Que devrions-nous utiliser ?

Pour le travail numérique sur une seule machine, le multithreading est généralement préféré --- il est léger et le modèle de mémoire partagée est très pratique.

Le multiprocessing devient important lors du passage à l'échelle au-delà d'une seule machine.

Pour la grande majorité de ce que nous faisons dans ces cours, le multithreading suffira.


### Accélérateurs matériels

Une source de parallélisme plus spectaculaire provient d'accélérateurs matériels
spécialisés, en particulier les **GPU** (Graphics Processing Units, unités de traitement graphique).

Les GPU ont été conçus à l'origine pour le rendu graphique, qui nécessite d'effectuer
la même opération sur de nombreux pixels simultanément.

```{figure} /_static/lecture_specific/need_for_speed/geforce.png
:scale: 40
```

Cette architecture --- des milliers de cœurs simples exécutant la même instruction
sur différents points de données --- s'avère idéale pour le calcul scientifique.

```{note}
Un **cœur** est une unité de traitement indépendante au sein d'une puce --- un circuit qui
peut exécuter des instructions par lui-même. Un CPU possède généralement un petit nombre de
cœurs puissants, chacun capable de gérer des séquences complexes d'opérations. Un GPU
regroupe plutôt des milliers de cœurs plus petits et plus simples, chacun conçu pour effectuer des
opérations arithmétiques de base. La puissance du GPU vient du fait que tous ces
cœurs travaillent simultanément sur différentes parties du même problème.
```

Lorsqu'un calcul peut être exprimé comme des opérations indépendantes sur de grands tableaux de
données, les GPU peuvent être plusieurs ordres de grandeur plus rapides que les CPU.

Les **TPU** (Tensor Processing Units, unités de traitement tensoriel), conçus par Google pour l'apprentissage automatique,
suivent une philosophie similaire, en optimisant pour des opérations matricielles parallèles massives.


### Accéder aux ressources GPU

De nombreuses stations de travail et ordinateurs portables sont désormais équipés de GPU performants, et un seul
GPU moderne est souvent suffisant pour des projets de recherche individuels.

Les bibliothèques Python modernes comme JAX, largement abordée dans cette série de cours,
détectent et utilisent automatiquement les GPU disponibles avec un minimum de modifications de code.

Pour les problèmes à plus grande échelle, les serveurs multi-GPU (souvent 4 à 8 GPU par machine) sont
de plus en plus courants.

```{figure} /_static/lecture_specific/need_for_speed/dgx.png
:scale: 40
```

Avec un logiciel approprié, les calculs peuvent être distribués sur plusieurs GPU,
que ce soit au sein d'un seul serveur ou sur un cluster.

Nous explorerons le calcul GPU plus en détail dans les cours ultérieurs, en l'appliquant à une
gamme d'applications économiques.