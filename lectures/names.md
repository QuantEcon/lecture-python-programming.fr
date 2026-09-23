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
  title: Noms et espaces de nommage
  headings:
    Overview: Vue d'ensemble
    Variable Names in Python: Noms de variables en Python
    Namespaces: Espaces de nommage
    Viewing Namespaces: Consulter les espaces de nommage
    Interactive Sessions: Sessions interactives
    The Global Namespace: L'espace de nommage global
    Local Namespaces: Espaces de nommage locaux
    The `__builtins__` Namespace: L'espace de nommage `__builtins__`
    Name Resolution: Résolution des noms
    Name Resolution::Mutable Versus Immutable Parameters: Paramètres mutables et immuables
---

(oop_names)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Noms et espaces de nommage

## Vue d'ensemble

Ce cours porte entièrement sur les noms de variables, sur la façon dont ils peuvent être utilisés et sur la manière dont ils sont interprétés par l'interpréteur Python.

Cela peut sembler un peu ennuyeux, mais le modèle qu'a adopté Python pour gérer les noms est élégant et intéressant.

De plus, vous vous épargnerez de nombreuses heures de débogage si vous comprenez bien le fonctionnement des noms en Python.

(var_names)=
## Noms de variables en Python

```{index} single: Python; Variable Names
```

Considérons l'instruction Python

```{code-cell} python3
x = 42
```

Nous savons désormais que lorsque cette instruction est exécutée, Python crée un objet de type `int` dans la mémoire de votre ordinateur, contenant

* la valeur `42`
* certains attributs associés

Mais qu'est-ce que `x` lui-même ?

En Python, `x` est appelé un **nom**, et l'instruction `x = 42` **lie** le nom `x` à l'objet entier dont nous venons de parler.

En interne, ce processus de liaison des noms aux objets est implémenté sous la forme d'un dictionnaire — nous y reviendrons dans un instant.

Il n'y a aucun problème à lier deux noms ou plus à un même objet, quel que soit cet objet

```{code-cell} python3
def f(string):      # Crée une fonction appelée f
    print(string)   # qui affiche toute chaîne qui lui est passée

g = f
id(g) == id(f)
```

```{code-cell} python3
g('test')
```

Lors de la première étape, un objet fonction est créé, et le nom `f` lui est lié.

Après avoir lié le nom `g` au même objet, nous pouvons l'utiliser partout où nous utiliserions `f`.

Que se passe-t-il lorsque le nombre de noms liés à un objet tombe à zéro ?

Voici un exemple de cette situation, où le nom `x` est d'abord lié à un objet puis **relié** à un autre

```{code-cell} python3
x = 'foo'
id(x)
x = 'bar'  
id(x)
```

Dans ce cas, après avoir relié `x` à `'bar'`, aucun nom n'est lié au premier objet `'foo'`.

Cela entraîne la récupération de `'foo'` par le ramasse-miettes.

Autrement dit, l'emplacement mémoire qui stocke cet objet est libéré et rendu au système d'exploitation.

Le fonctionnement des ramasse-miettes est encore un domaine de recherche actif en informatique.

Vous pouvez [en lire davantage sur le fonctionnement des ramasse-miettes](https://rushter.com/blog/python-garbage-collector/) si cela vous intéresse.

## Espaces de nommage

```{index} single: Python; Namespaces
```

Rappelons que l'instruction

```{code-cell} python3
x = 42
```

lie le nom `x` à l'objet entier situé à droite.

Nous avons également mentionné que ce processus de liaison de `x` au bon objet est implémenté sous la forme d'un dictionnaire.

Ce dictionnaire est appelé un espace de nommage.

```{admonition} Définition
Un **espace de nommage** est une table de correspondance entre des noms et des objets en mémoire.
```


Python utilise plusieurs espaces de nommage, qu'il crée à la volée selon les besoins.

Par exemple, chaque fois que nous importons un module, Python crée un espace de nommage pour ce module.

Pour voir cela en action, supposons que nous écrivions un script `mathfoo.py` composé d'une seule ligne

```{code-cell} python3
%%file mathfoo.py
pi = 'foobar'
```

Maintenant, nous démarrons l'interpréteur Python et l'importons

```{code-cell} python3
import mathfoo
```

Ensuite, importons le module `math` de la bibliothèque standard

```{code-cell} python3
import math
```

Ces deux modules possèdent un attribut appelé `pi`

```{code-cell} python3
math.pi
```

```{code-cell} python3
mathfoo.pi
```

Ces deux liaisons différentes de `pi` existent dans des espaces de nommage différents, chacun implémenté sous la forme d'un dictionnaire.

Si vous le souhaitez, vous pouvez consulter directement le dictionnaire, en utilisant `module_name.__dict__`.

```{code-cell} python3
import math

math.__dict__.items()
```

```{code-cell} python3
import mathfoo

mathfoo.__dict__
```

Comme vous le savez, nous accédons aux éléments de l'espace de nommage en utilisant la notation d'attribut par point

```{code-cell} python3
math.pi
```

Cela est entièrement équivalent à `math.__dict__['pi']`

```{code-cell} python3
math.__dict__['pi'] 
```

## Consulter les espaces de nommage

Comme nous l'avons vu ci-dessus, l'espace de nommage `math` peut être affiché en tapant `math.__dict__`.

Une autre façon d'en voir le contenu est de taper `vars(math)`

```{code-cell} python3
vars(math).items()
```

Si vous souhaitez seulement voir les noms, vous pouvez taper

```{code-cell} python3
# Affiche les 10 premiers noms
dir(math)[0:10]
```

Remarquez les noms spéciaux `__doc__` et `__name__`.

Ceux-ci sont initialisés dans l'espace de nommage chaque fois qu'un module est importé

* `__doc__` est la chaîne de documentation du module
* `__name__` est le nom du module

```{code-cell} python3
print(math.__doc__)
```

```{code-cell} python3
math.__name__
```

## Sessions interactives

```{index} single: Python; Interpreter
```

En Python, **tout** code exécuté par l'interpréteur s'exécute dans un module.

Qu'en est-il des commandes tapées à l'invite ?

Elles sont également considérées comme étant exécutées au sein d'un module — dans ce cas, un module appelé `__main__`.

Pour vérifier cela, nous pouvons examiner le nom du module actuel via la valeur de `__name__` donnée à l'invite

```{code-cell} python3
print(__name__)
```

Lorsque nous exécutons un script à l'aide de la commande `%run` d'IPython, le contenu du fichier est également exécuté comme faisant partie de `__main__`.

Pour le voir, créons un fichier `mod.py` qui affiche son propre attribut `__name__`

```{code-cell} ipython
%%file mod.py
print(__name__)
```

Maintenant, examinons deux façons différentes de l'exécuter dans IPython

```{code-cell} python3
import mod  # Importation standard
```

```{code-cell} ipython
%run mod.py  # Exécution interactive
```

Dans le second cas, le code est exécuté comme faisant partie de `__main__`, donc `__name__` est égal à `__main__`.

Pour voir le contenu de l'espace de nommage de `__main__`, nous utilisons `vars()` plutôt que `vars(__main__)`.

Si vous faites cela dans IPython, vous verrez tout un tas de variables dont IPython a besoin et qu'il a initialisées lorsque vous avez démarré votre session.

Si vous préférez ne voir que les variables que vous avez initialisées, utilisez `%whos`

```{code-cell} ipython
x = 2
y = 3

import numpy as np

%whos
```

## L'espace de nommage global

```{index} single: Python; Namespace (Global)
```

La documentation Python fait souvent référence à « l'espace de nommage global ».

L'espace de nommage global est *l'espace de nommage du module en cours d'exécution*.

Par exemple, supposons que nous démarrions l'interpréteur et commencions à faire des affectations.

Nous travaillons maintenant dans le module `__main__`, et donc l'espace de nommage de `__main__` est l'espace de nommage global.

Ensuite, nous importons un module appelé `amodule`

```{code-block} python3
:class: no-execute

import amodule
```

À ce stade, l'interpréteur crée un espace de nommage pour le module `amodule` et commence à exécuter les commandes du module.

Pendant ce temps, l'espace de nommage `amodule.__dict__` est l'espace de nommage global.

Une fois l'exécution du module terminée, l'interpréteur revient au module depuis lequel l'instruction d'importation a été faite.

Dans ce cas, il s'agit de `__main__`, donc l'espace de nommage de `__main__` redevient l'espace de nommage global.

## Espaces de nommage locaux

```{index} single: Python; Namespace (Local)
```

Fait important : lorsque nous appelons une fonction, l'interpréteur crée un *espace de nommage local* pour cette fonction, et y enregistre les variables.

La raison de ceci sera expliquée dans un instant.

Les variables de l'espace de nommage local sont appelées *variables locales*.

Lorsque l'exécution de la fonction se termine, son espace de nommage local est oublié.

Pendant que la fonction s'exécute, nous pouvons voir le contenu de l'espace de nommage local avec `locals()`.

Par exemple, considérons

```{code-cell} python3
def f(x):
    a = 2
    print(locals())
    return a * x
```

Maintenant, appelons la fonction

```{code-cell} python3
f(1)
```

Vous pouvez voir l'espace de nommage local de `f` avant qu'il ne soit détruit.

## L'espace de nommage `__builtins__`

```{index} single: Python; Namespace (__builtins__)
```

Nous avons utilisé diverses fonctions natives, telles que `max(), dir(), str(), list(), len(), range(), type()`, etc.

Comment fonctionne l'accès à ces noms ?

* Ces définitions sont fournies par le module `builtins`.
* Elles disposent de leur propre espace de nommage appelé `__builtins__`.

```{code-cell} python3
# Affiche les 10 premiers noms dans `__main__`
dir()[0:10]
```

```{code-cell} python3
# Affiche les 10 premiers noms dans `__builtins__`
dir(__builtins__)[0:10]
```

Nous pouvons accéder aux éléments de l'espace de nommage comme suit

```{code-cell} python3
__builtins__.max
```

Mais `__builtins__` est spécial, car nous pouvons toujours y accéder directement aussi

```{code-cell} python3
max
```

```{code-cell} python3
__builtins__.max == max
```

La section suivante explique comment cela fonctionne...

## Résolution des noms

```{index} single: Python; Namespace (Resolution)
```

Les espaces de nommage sont formidables car ils nous aident à organiser les noms de variables.

(Tapez `import this` à l'invite et regardez le dernier élément affiché)

Cependant, nous devons comprendre comment l'interpréteur Python travaille avec plusieurs espaces de nommage.

Comprendre le flux d'exécution nous aidera à déterminer quelles variables sont accessibles dans une portée donnée et comment les utiliser lors de l'écriture et du débogage des programmes.


À tout moment de l'exécution, il existe en fait au moins deux espaces de nommage accessibles directement.

(« Accessible directement » signifie sans utiliser de point, comme dans `pi` plutôt que `math.pi`)

Ces espaces de nommage sont

* L'espace de nommage global (du module en cours d'exécution)
* L'espace de nommage des primitives

Si l'interpréteur exécute une fonction, alors les espaces de nommage directement accessibles sont

* L'espace de nommage local de la fonction
* L'espace de nommage global (du module en cours d'exécution)
* L'espace de nommage des primitives

Parfois, des fonctions sont définies à l'intérieur d'autres fonctions, comme ceci

```{code-cell} python3
def f():
    a = 2
    def g():
        b = 4
        print(a * b)
    g()
```

Ici, `f` est la *fonction englobante* de `g`, et chaque fonction dispose de ses propres espaces de nommage.

Maintenant, nous pouvons énoncer la règle qui régit la résolution des espaces de nommage :

L'ordre dans lequel l'interpréteur recherche les noms est

1. l'espace de nommage local (s'il existe)
1. la hiérarchie des espaces de nommage englobants (s'ils existent)
1. l'espace de nommage global
1. l'espace de nommage des primitives

Si le nom ne se trouve dans aucun de ces espaces de nommage, l'interpréteur lève une erreur `NameError`.

C'est ce qu'on appelle la **règle LEGB** (Local, Enclosing, Global, Built-in, c'est-à-dire local, englobant, global et natif).

Voici un exemple qui aide à illustrer cela.

Les visualisations ici sont créées par [nbtutor](https://github.com/lgpage/nbtutor) dans un notebook Jupyter.

Elles peuvent vous aider à mieux comprendre votre programme lorsque vous apprenez un nouveau langage.

Considérons un script `test.py` qui ressemble à ce qui suit

```{code-cell} python3
%%file test.py
def g(x):
    a = 1
    x = x + a
    return x

a = 0
y = g(10)
print("a = ", a, "y = ", y)
```

Que se passe-t-il lorsque nous exécutons ce script ?

```{code-cell} ipython
%run test.py
```

Tout d'abord,

* L'espace de nommage global `{}` est créé.

```{figure} /_static/lecture_specific/oop_intro/global.png
```

* L'objet fonction est créé, et `g` lui est lié au sein de l'espace de nommage global.
* Le nom `a` est lié à `0`, encore une fois dans l'espace de nommage global.

```{figure} /_static/lecture_specific/oop_intro/global2.png
```

Ensuite, `g` est appelé via `y = g(10)`, ce qui conduit à la séquence d'actions suivante

* L'espace de nommage local de la fonction est créé.
* Les noms locaux `x` et `a` sont liés, de sorte que l'espace de nommage local devient `{'x': 10, 'a': 1}`.

Notez que le `a` global n'a pas été affecté par le `a` local.

```{figure} /_static/lecture_specific/oop_intro/local1.png
```


* L'instruction `x = x + a` utilise le `a` local et le `x` local pour calculer `x + a`, et lie le nom local `x` au résultat.
* Cette valeur est renvoyée, et `y` lui est lié dans l'espace de nommage global.
* Les `x` et `a` locaux sont supprimés (et l'espace de nommage local est libéré).

```{figure} /_static/lecture_specific/oop_intro/local_return.png
```


(mutable_vs_immutable)=
### Paramètres {index}`mutables <single: Mutable>` et {index}`immuables <single: Immutable>`

C'est le bon moment pour en dire un peu plus sur les objets mutables et immuables.

Considérons le segment de code

```{code-cell} python3
def f(x):
    x = x + 1
    return x

x = 1
print(f(x), x)
```

Nous comprenons maintenant ce qui va se passer ici : le code affiche `2` comme valeur de `f(x)` et `1` comme valeur de `x`.

Tout d'abord, `f` et `x` sont enregistrés dans l'espace de nommage global.

L'appel `f(x)` crée un espace de nommage local et y ajoute `x`, lié à `1`.

Ensuite, ce `x` local est relié au nouvel objet entier `2`, et cette valeur est renvoyée.

Rien de tout cela n'affecte le `x` global.

Cependant, c'est une autre histoire lorsque nous utilisons un type de données **mutable** tel qu'une liste

```{code-cell} python3
def f(x):
    x[0] = x[0] + 1
    return x

x = [1]
print(f(x), x)
```

Cela affiche `[2]` comme valeur de `f(x)` et également `[2]` comme valeur de `x`.

Voici ce qui se passe

* `f` est enregistré comme fonction dans l'espace de nommage global

```{figure} /_static/lecture_specific/oop_intro/mutable1.png
```

* `x` est lié à `[1]` dans l'espace de nommage global

```{figure} /_static/lecture_specific/oop_intro/mutable2.png
```

* L'appel `f(x)`
    * Crée un espace de nommage local
    * Ajoute `x` à l'espace de nommage local, lié à `[1]`

```{figure} /_static/lecture_specific/oop_intro/mutable3.png
```

```{note}
Le `x` global et le `x` local se réfèrent au même `[1]`
```

Nous pouvons voir que l'identité du `x` local et l'identité du `x` global sont les mêmes

```{code-cell} python3
def f(x):
    x[0] = x[0] + 1
    print(f"l'identité de x local est {id(x)}")
    return x

x = [1]
print(f"l'identité de x global est {id(x)}")
print(f(x), x)
```

* Au sein de `f(x)`
    * La liste `[1]` est modifiée en `[2]`
    * Renvoie la liste `[2]`

```{figure} /_static/lecture_specific/oop_intro/mutable4.png
```
* L'espace de nommage local est libéré, et le `x` local est perdu

```{figure} /_static/lecture_specific/oop_intro/mutable5.png
```

Si vous souhaitez modifier le `x` local et le `x` global séparément, vous pouvez créer une [*copie*](https://docs.python.org/3/library/copy.html) de la liste et affecter la copie au `x` local.

Nous vous laissons explorer cela.
