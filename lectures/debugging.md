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
  title: Débogage et gestion des erreurs
  headings:
    Overview: Vue d'ensemble
    Debugging: Débogage
    Debugging::The `debug` Magic: La commande magique `debug`
    Debugging::Setting a Break Point: Définir un point d'arrêt
    Debugging::Other Useful Magics: Autres commandes magiques utiles
    Handling Errors: Gestion des erreurs
    Handling Errors::Errors in Python: Les erreurs en Python
    Handling Errors::Assertions: Assertions
    Handling Errors::Handling Errors During Runtime: Gestion des erreurs pendant l'exécution
    Handling Errors::Handling Errors During Runtime::Catching Exceptions: Capturer les exceptions
    Exercises: Exercices
---

(debugging)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Débogage et gestion des erreurs

```{index} single: Debugging
```

```{epigraph}
« Le débogage est deux fois plus difficile que l'écriture du code initial.
Par conséquent, si vous écrivez le code aussi astucieusement que possible, vous n'êtes, par définition,
pas assez intelligent pour le déboguer. » -- Brian Kernighan
```

## Vue d'ensemble

Êtes-vous l'un de ces programmeurs qui remplissent leur code d'instructions `print` lorsqu'ils essaient de déboguer leurs programmes ?

Bon, nous avons tous fait cela autrefois.

(D'accord, il nous arrive encore de le faire...)

Mais dès que vous commencerez à écrire des programmes plus volumineux, vous aurez besoin d'un meilleur système.

Vous voudrez peut-être aussi gérer les erreurs potentielles de votre code au moment où elles se produisent.

Dans ce cours, nous verrons comment déboguer nos programmes et améliorer la gestion des erreurs.

## Débogage

```{index} single: Debugging
```

Les outils de débogage pour Python varient selon les plateformes, les IDE et les éditeurs.

Par exemple, un [débogueur visuel](https://jupyterlab.readthedocs.io/en/stable/user/debugger.html) est disponible dans JupyterLab.

Ici, nous nous concentrerons sur Jupyter Notebook et vous laisserons explorer les autres configurations.

Nous aurons besoin des imports suivants

```{code-cell} ipython
import numpy as np
import matplotlib.pyplot as plt
```

(debug_magic)= 
### La commande magique `debug`

Considérons un exemple simple (et plutôt artificiel)

```{code-cell} ipython
---
tags: [raises-exception]
---
def plot_log():
    fig, ax = plt.subplots(2, 1)
    x = np.linspace(1, 2, 10)
    ax.plot(x, np.log(x))
    plt.show()

plot_log()  # Appelle la fonction, génère le graphique
```

Ce code est censé tracer la fonction `log` sur l'intervalle $[1, 2]$.

Mais il y a une erreur ici : `plt.subplots(2, 1)` devrait être simplement `plt.subplots()`.

(L'appel `plt.subplots(2, 1)` renvoie un tableau NumPy contenant deux objets axes, adapté pour avoir deux sous-graphiques sur la même figure)

La trace d'exécution montre que l'erreur se produit lors de l'appel de méthode `ax.plot(x, np.log(x))`.

L'erreur se produit parce que nous avons fait par erreur de `ax` un tableau NumPy, et un tableau NumPy n'a pas de méthode `plot`.

Mais faisons semblant de ne pas comprendre cela pour le moment.

Nous pourrions soupçonner qu'il y a un problème avec `ax`, mais lorsque nous essayons d'examiner cet objet, nous obtenons l'exception suivante :

```{code-cell} python3
---
tags: [raises-exception]
---
ax
```

Le problème est que `ax` a été défini à l'intérieur de `plot_log()`, et le nom est
perdu une fois que cette fonction se termine.

Essayons de procéder autrement.

Nous exécutons à nouveau le premier bloc de cellule, générant la même erreur

```{code-cell} python3
---
tags: [raises-exception]
---
def plot_log():
    fig, ax = plt.subplots(2, 1)
    x = np.linspace(1, 2, 10)
    ax.plot(x, np.log(x))
    plt.show()

plot_log()  # Appelle la fonction, génère le graphique
```

Mais cette fois, nous tapons le bloc de cellule suivant

```{code-block} ipython
:class: no-execute
%debug
```

Vous devriez être basculé dans une nouvelle invite qui ressemble à ceci

```{code-block} ipython
:class: no-execute
ipdb>
```

(Vous verrez peut-être pdb> à la place)

Nous pouvons maintenant examiner la valeur de nos variables à ce point du programme, avancer pas à pas dans le code, etc.

Par exemple, ici nous tapons simplement le nom `ax` pour voir ce qui se passe avec
cet objet :

```{code-block} ipython
:class: no-execute
ipdb> ax
array([<matplotlib.axes.AxesSubplot object at 0x290f5d0>,
       <matplotlib.axes.AxesSubplot object at 0x2930810>], dtype=object)
```

Il est maintenant très clair que `ax` est un tableau, ce qui clarifie la source du
problème.

Pour savoir ce que vous pouvez faire d'autre depuis l'intérieur de `ipdb` (ou `pdb`), utilisez l'aide
en ligne

```{code-block} ipython
:class: no-execute
ipdb> h

Documented commands (type help <topic>):
========================================
EOF    bt         cont      enable  jump  pdef   r        tbreak   w
a      c          continue  exit    l     pdoc   restart  u        whatis
alias  cl         d         h       list  pinfo  return   unalias  where
args   clear      debug     help    n     pp     run      unt
b      commands   disable   ignore  next  q      s        until
break  condition  down      j       p     quit   step     up

Miscellaneous help topics:
==========================
exec  pdb

Undocumented commands:
======================
retval  rv

ipdb> h c
c(ont(inue))
Continue execution, only stop when a breakpoint is encountered.
```

### Définir un point d'arrêt

L'approche précédente est pratique mais parfois insuffisante.

Considérons la version modifiée suivante de notre fonction ci-dessus

```{code-cell} python3
---
tags: [raises-exception]
---
def plot_log():
    fig, ax = plt.subplots()
    x = np.logspace(1, 2, 10)
    ax.plot(x, np.log(x))
    plt.show()

plot_log()
```

Ici, le problème d'origine est corrigé, mais nous avons accidentellement écrit
`np.logspace(1, 2, 10)` au lieu de `np.linspace(1, 2, 10)`.

Maintenant, il n'y aura pas d'exception, mais le graphique ne sera pas correct.

Pour investiguer, il serait utile de pouvoir inspecter des variables comme `x` pendant l'exécution de la fonction.

À cette fin, nous ajoutons un « point d'arrêt » en insérant `breakpoint()` à l'intérieur du bloc de code de la fonction

```{code-block} python3
:class: no-execute
def plot_log():
    breakpoint()
    fig, ax = plt.subplots()
    x = np.logspace(1, 2, 10)
    ax.plot(x, np.log(x))
    plt.show()

plot_log()
```

Maintenant, exécutons le script et investiguons via le débogueur

```{code-block} ipython
:class: no-execute
> <ipython-input-6-a188074383b7>(6)plot_log()
-> fig, ax = plt.subplots()
(Pdb) n
> <ipython-input-6-a188074383b7>(7)plot_log()
-> x = np.logspace(1, 2, 10)
(Pdb) n
> <ipython-input-6-a188074383b7>(8)plot_log()
-> ax.plot(x, np.log(x))
(Pdb) x
array([ 10.        ,  12.91549665,  16.68100537,  21.5443469 ,
        27.82559402,  35.93813664,  46.41588834,  59.94842503,
        77.42636827, 100.        ])
```

Nous avons utilisé `n` deux fois pour avancer pas à pas dans le code (une ligne à la fois).

Puis nous avons affiché la valeur de `x` pour voir ce qui se passait avec cette variable.

Pour quitter le débogueur, utilisez `q`.

### Autres commandes magiques utiles

Dans ce cours, nous avons utilisé la commande magique IPython `%debug`.

Il existe de nombreuses autres commandes magiques utiles :

* `%precision 4` définit la précision d'affichage des flottants à 4 décimales
* `%whos` donne une liste des variables et de leurs valeurs
* `%quickref` donne une liste des commandes magiques

La liste complète des commandes magiques est [ici](https://ipython.readthedocs.io/en/stable/interactive/magics.html).


## Gestion des erreurs

```{index} single: Python; Handling Errors
```

Il est parfois possible d'anticiper les bogues et les erreurs au moment où nous écrivons le code.

Par exemple, la variance empirique non biaisée de l'échantillon $y_1, \ldots, y_n$
est définie comme

$$
s^2 := \frac{1}{n-1} \sum_{i=1}^n (y_i - \bar y)^2
\qquad \bar y = \text{ moyenne empirique}
$$

Cela peut être calculé dans NumPy en utilisant `np.var`.

Mais si vous écriviez une fonction pour gérer un tel calcul, vous pourriez
anticiper une erreur de division par zéro lorsque la taille de l'échantillon est égale à un.

Une action possible est de ne rien faire --- le programme va simplement planter et afficher un message d'erreur.

Mais il vaut parfois la peine d'écrire votre code de manière à anticiper et à gérer les erreurs d'exécution qui pourraient survenir selon vous.

Pourquoi ?

* Parce que les informations de débogage fournies par l'interpréteur sont souvent moins utiles que celles qui peuvent être fournies par un message d'erreur bien rédigé.
* Parce que les erreurs qui provoquent l'arrêt de l'exécution interrompent les flux de travail.
* Parce que cela réduit la confiance de vos utilisateurs dans votre code (si vous écrivez pour d'autres).


Dans cette section, nous discuterons des différents types d'erreurs en Python et des techniques pour gérer les erreurs potentielles dans nos programmes.

### Les erreurs en Python

Nous avons vu `AttributeError` et `NameError` dans {any}`nos exemples précédents <debug_magic>`.

En Python, il existe deux types d'erreurs -- les erreurs de syntaxe et les exceptions.

```{index} single: Python; Exceptions
```

Voici un exemple d'un type d'erreur courant

```{code-cell} python3
---
tags: [raises-exception]
---
def f:
```

Comme une syntaxe illégale ne peut pas être exécutée, une erreur de syntaxe met fin à l'exécution du programme.

Voici un autre type d'erreur, sans rapport avec la syntaxe

```{code-cell} python3
---
tags: [raises-exception]
---
1 / 0
```

En voici une autre

```{code-cell} python3
---
tags: [raises-exception]
---
x1 = y1
```

Et encore une autre

```{code-cell} python3
---
tags: [raises-exception]
---
'foo' + 6
```

Et encore une autre

```{code-cell} python3
---
tags: [raises-exception]
---
X = []
x = X[0]
```

À chaque occasion, l'interpréteur nous informe du type d'erreur

* `NameError`, `TypeError`, `IndexError`, `ZeroDivisionError`, etc.

En Python, ces erreurs sont appelées des *exceptions*.

### Assertions

```{index} single: Python; Assertions
```

Parfois, les erreurs peuvent être évitées en vérifiant si votre programme s'exécute comme prévu.

Un moyen relativement simple de gérer les vérifications est le mot-clé `assert`.

Par exemple, imaginons un instant que la fonction `np.var` n'existe
pas et que nous devons écrire la nôtre

```{code-cell} python3
def var(y):
    n = len(y)
    assert n > 1, 'Sample size must be greater than one.'
    return np.sum((y - y.mean())**2) / float(n-1)
```

Si nous exécutons ceci avec un tableau de longueur un, le programme se terminera et
affichera notre message d'erreur

```{code-cell} python3
---
tags: [raises-exception]
---
var([1])
```

L'avantage est que nous pouvons

* échouer tôt, dès que nous savons qu'il y aura un problème
* fournir des informations spécifiques sur la raison de l'échec d'un programme

### Gestion des erreurs pendant l'exécution

```{index} single: Python; Runtime Errors
```

L'approche utilisée ci-dessus est un peu limitée, car elle conduit toujours à
la terminaison.

Parfois, nous pouvons gérer les erreurs de manière plus élégante, en traitant les cas particuliers.

Voyons comment cela se fait.

#### Capturer les exceptions

Nous pouvons capturer et traiter les exceptions à l'aide de blocs `try` -- `except`.

Voici un exemple simple

```{code-cell} python3
def f(x):
    try:
        return 1.0 / x
    except ZeroDivisionError:
        print('Error: division by zero.  Returned None')
    return None
```

Lorsque nous appelons `f`, nous obtenons la sortie suivante

```{code-cell} python3
f(2)
```

```{code-cell} python3
f(0)
```

```{code-cell} python3
f(0.0)
```

L'erreur est capturée et l'exécution du programme n'est pas interrompue.

Notez que les autres types d'erreurs ne sont pas capturés.

Si nous craignons que l'utilisateur passe une chaîne de caractères, nous pouvons également capturer cette erreur

```{code-cell} python3
def f(x):
    try:
        return 1.0 / x
    except ZeroDivisionError:
        print('Error: Division by zero.  Returned None')
    except TypeError:
        print(f'Error: x cannot be of type {type(x)}.  Returned None')
    return None
```

Voici ce qui se passe

```{code-cell} python3
f(2)
```

```{code-cell} python3
f(0)
```

```{code-cell} python3
f('foo')
```

Si nous nous sentons paresseux, nous pouvons capturer ces erreurs ensemble

```{code-cell} python3
def f(x):
    try:
        return 1.0 / x
    except:
        print(f'Error.  An issue has occurred with x = {x} of type: {type(x)}')
    return None
```

Voici ce qui se passe

```{code-cell} python3
f(2)
```

```{code-cell} python3
f(0)
```

```{code-cell} python3
f('foo')
```

En général, il vaut mieux être précis.


## Exercices

```{exercise-start}
:label: debug_ex1
```

Supposons que nous ayons un fichier texte `numbers.txt` contenant les lignes suivantes

```{code-block} none
:class: no-execute

prices
3
8

7
21
```

En utilisant `try` -- `except`, écrivez un programme qui lit le contenu du fichier et additionne les nombres, en ignorant les lignes sans nombres.

Vous pouvez utiliser la fonction `open()` que nous avons apprise {any}`précédemment<iterators>` pour ouvrir `numbers.txt`.
```{exercise-end}
```


```{solution-start} debug_ex1
:class: dropdown
```

Enregistrons d'abord les données

```{code-cell} python3
%%file numbers.txt
prices
3
8

7
21
```

```{code-cell} python3
f = open('numbers.txt')

total = 0.0
for line in f:
    try:
        total += float(line)
    except ValueError:
        pass

f.close()

print(total)
```

```{solution-end}
```