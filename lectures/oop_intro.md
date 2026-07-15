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
  title: "POO I\_: Objets et méthodes"
  headings:
    Overview: Vue d'ensemble
    Objects: Objets
    Objects::Type: Type
    Objects::Identity: Identité
    'Objects::Object Content: Data and Attributes': "Contenu de l'objet\_: données et attributs"
    Objects::Methods: Méthodes
    Inspection Using Rich: Inspection avec Rich
    A Little Mystery: Un petit mystère
    Summary: Résumé
    Exercises: Exercices
---

(oop_intro)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# POO I : Objets et méthodes

## Vue d'ensemble

Le paradigme de programmation traditionnel (pensez à Fortran, C, MATLAB, etc.) est appelé [procédural](https://en.wikipedia.org/wiki/Procedural_programming).

Il fonctionne de la manière suivante

* Le programme possède un état qui correspond aux valeurs de ses variables.
* Des fonctions sont appelées pour agir sur cet état et le transformer.
* Les sorties finales sont produites par une séquence d'appels de fonctions.

Deux autres paradigmes importants sont la [programmation orientée objet](https://en.wikipedia.org/wiki/Object-oriented_programming) (POO) et la [programmation fonctionnelle](https://en.wikipedia.org/wiki/Functional_programming).


Dans le paradigme POO, les données et les fonctions sont regroupées ensemble dans des « objets » --- et les fonctions, dans ce contexte, sont appelées **méthodes**.

Les méthodes sont appelées pour transformer les données contenues dans l'objet.

* Pensez à une liste Python qui contient des données et possède des méthodes telles que `append()` et `pop()` qui transforment ces données.

Les langages de programmation fonctionnelle sont construits sur l'idée de composer des fonctions.

* Des exemples influents comprennent [Lisp](https://en.wikipedia.org/wiki/Common_Lisp), [Haskell](https://en.wikipedia.org/wiki/Haskell) et [Elixir](https://en.wikipedia.org/wiki/Elixir_(programming_language)).

Alors, dans laquelle de ces catégories Python s'inscrit-il ?

En réalité, Python est un langage pragmatique qui mêle les styles orienté objet, fonctionnel et procédural, plutôt que d'adopter une approche puriste.

D'une part, cela permet à Python et à ses utilisateurs de sélectionner les bons aspects de différents paradigmes.

D'autre part, ce manque de pureté peut parfois prêter à confusion.

Heureusement, cette confusion est réduite au minimum si vous comprenez qu'à un niveau fondamental, Python *est* orienté objet.

Par cela, nous voulons dire qu'en Python, *tout est objet*.

Dans ce cours, nous expliquons ce que signifie cette affirmation et pourquoi elle est importante.

Nous utiliserons la bibliothèque tierce suivante


```{code-cell} python3
:tags: [hide-output]
!pip install rich
```


## Objets

```{index} single: Python; Objects
```

En Python, un *objet* est une collection de données et d'instructions conservées dans la mémoire de l'ordinateur, qui comprend

1. un type
1. une identité unique
1. des données (c'est-à-dire du contenu)
1. des méthodes

Ces concepts sont définis et discutés successivement ci-dessous.

(type)=
### Type

```{index} single: Python; Type
```

Python propose différents types d'objets, afin d'accommoder différentes catégories de données.

Par exemple

```{code-cell} python3
s = 'This is a string'
type(s)
```

```{code-cell} python3
x = 42   # Créons maintenant un entier
type(x)
```

Le type d'un objet est important pour de nombreuses expressions.

Par exemple, l'opérateur d'addition entre deux chaînes de caractères signifie la concaténation

```{code-cell} python3
'300' + 'cc'
```

En revanche, entre deux nombres, il signifie l'addition ordinaire

```{code-cell} python3
300 + 400
```

Considérez l'expression suivante

```{code-cell} python3
---
tags: [raises-exception]
---
'300' + 400
```

Ici, nous mélangeons les types, et il n'est pas clair pour Python si l'utilisateur souhaite

* convertir `'300'` en entier puis l'ajouter à `400`, ou
* convertir `400` en chaîne puis la concaténer avec `'300'`

Certains langages pourraient tenter de deviner, mais Python est *fortement typé*

* Le type est important, et la conversion de type implicite est rare.
* Python répondra plutôt en levant une `TypeError`.

Pour éviter l'erreur, vous devez clarifier en modifiant le type concerné.

Par exemple,

```{code-cell} python3
int('300') + 400   # Pour additionner en tant que nombres, convertir la chaîne en entier
```

(identity)=
### Identité

```{index} single: Python; Identity
```

En Python, chaque objet possède un identifiant unique, qui aide Python (et nous-mêmes) à suivre l'objet.

L'identité d'un objet peut être obtenue via la fonction `id()`

```{code-cell} python3
y = 2.5
z = 2.5
id(y)
```

```{code-cell} python3
id(z)
```

Dans cet exemple, `y` et `z` ont par hasard la même valeur (c'est-à-dire `2.5`), mais ce ne sont pas le même objet.

L'identité d'un objet est en fait simplement l'adresse de l'objet en mémoire.

### Contenu de l'objet : données et attributs

```{index} single: Python; Content
```

Si nous posons `x = 42`, alors nous créons un objet de type `int` qui contient
la donnée `42`.

En fait, il contient davantage, comme le montre l'exemple suivant

```{code-cell} python3
x = 42
x
```

```{code-cell} python3
x.imag
```

```{code-cell} python3
x.__class__
```

Lorsque Python crée cet objet entier, il stocke avec lui diverses informations auxiliaires, telles que la partie imaginaire et le type.

Tout nom suivant un point est appelé un *attribut* de l'objet situé à gauche du point.

* par exemple, ``imag`` et `__class__` sont des attributs de `x`.

Nous voyons dans cet exemple que les objets possèdent des attributs contenant des informations auxiliaires.

Ils possèdent également des attributs qui agissent comme des fonctions, appelés *méthodes*.

Ces attributs sont importants, alors discutons-en en profondeur.

(methods)=
### Méthodes

```{index} single: Python; Methods
```

Les méthodes sont des *fonctions regroupées avec les objets*.

Formellement, les méthodes sont des attributs d'objets qui sont **appelables** -- c'est-à-dire des attributs qui peuvent être appelés comme des fonctions

```{code-cell} python3
x = ['foo', 'bar']
callable(x.append)
```

```{code-cell} python3
callable(x.__doc__)
```

Les méthodes agissent généralement sur les données contenues dans l'objet auquel elles appartiennent, ou combinent ces données avec d'autres données

```{code-cell} python3
x = ['a', 'b']
x.append('c')
s = 'This is a string'
s.upper()
```

```{code-cell} python3
s.lower()
```

```{code-cell} python3
s.replace('This', 'That')
```

Une grande partie des fonctionnalités de Python s'organise autour d'appels de méthodes.

Par exemple, considérez le morceau de code suivant

```{code-cell} python3
x = ['a', 'b']
x[0] = 'aa'  # Affectation d'élément via la notation entre crochets
x
```

Il ne semble pas qu'aucune méthode ne soit utilisée ici, mais en fait la notation d'affectation entre crochets n'est qu'une interface pratique vers un appel de méthode.

Ce qui se passe réellement, c'est que Python appelle la méthode `__setitem__`, comme suit

```{code-cell} python3
x = ['a', 'b']
x.__setitem__(0, 'aa')  # Équivalent à x[0] = 'aa'
x
```

(Si vous le souhaitiez, vous pourriez modifier la méthode `__setitem__`, de sorte que l'affectation entre crochets fasse quelque chose de totalement différent)

## Inspection avec Rich

Il existe un joli paquet appelé [rich](https://github.com/Textualize/rich) qui
nous aide à visualiser le contenu d'un objet.

Par exemple,

```{code-cell} python3
from rich import inspect
x = 10
inspect(10)
```
Si nous voulons voir également les méthodes, nous pouvons utiliser

```{code-cell} python3
inspect(10, methods=True)
```

En fait, il existe encore plus de méthodes, comme vous pouvez le constater en exécutant `inspect(10, all=True)`.



## Un petit mystère

Dans ce cours, nous avons affirmé que Python est, au fond, un langage orienté objet.

Mais voici un exemple qui semble plus procédural.

```{code-cell} python3
x = ['a', 'b']
m = len(x)
m
```

Si Python est orienté objet, pourquoi n'utilisons-nous pas `x.len()` ?

La réponse est liée au fait que Python vise la lisibilité et un style cohérent.

En Python, il est courant que les utilisateurs construisent des objets personnalisés --- nous verrons comment
faire cela {doc}`plus tard <python_oop>`.

Il est assez courant que les utilisateurs ajoutent à ceux-ci des méthodes qui mesurent la longueur de
l'objet, définie de manière appropriée.

Lorsqu'il s'agit de nommer une telle méthode, les choix naturels sont `len()` et `length()`.

Si certains utilisateurs choisissent `len()` et d'autres `length()`, alors le style sera
incohérent et plus difficile à retenir.

Pour éviter cela, le créateur de Python a choisi d'ajouter
`len()` comme fonction intégrée, afin de souligner que `len()` est la convention.

Ceci dit, Python *reste* néanmoins orienté objet en coulisses.

En effet, la liste `x` évoquée ci-dessus possède une méthode appelée `__len__()`.

Tout ce que fait la fonction `len()`, c'est appeler cette méthode.

Autrement dit, le code suivant est équivalent :

```{code-cell} python3
x = ['a', 'b']
len(x)
```
et

```{code-cell} python3
x = ['a', 'b']
x.__len__()
```


## Résumé

Le message de ce cours est clair :

* En Python, *tout ce qui est en mémoire est traité comme un objet*.

Cela inclut non seulement les listes, les chaînes de caractères, etc., mais aussi des choses moins évidentes, telles que

* les fonctions (une fois qu'elles ont été chargées en mémoire)
* les modules (idem)
* les fichiers ouverts en lecture ou en écriture
* les entiers, etc.

Se rappeler que tout est un objet vous aidera à interagir avec vos programmes
et à écrire un code clair et pythonique.

## Exercices

```{exercise-start}
:label: oop_intro_ex1
```

Nous avons déjà rencontré le {any}`type de données booléen <boolean>` précédemment.

En utilisant ce que nous avons appris dans ce cours, affichez une liste des méthodes de
l'objet booléen `True`.

```{hint}
:class: dropdown

Vous pouvez utiliser `callable()` pour vérifier si un attribut d'un objet peut être appelé comme une fonction
```

```{exercise-end}
```

```{solution-start} oop_intro_ex1
:class: dropdown
```

Tout d'abord, nous devons trouver tous les attributs de `True`, ce qui peut être fait via

```{code-cell} python3
print(sorted(True.__dir__()))
```

ou

```{code-cell} python3
print(sorted(dir(True)))
```

Puisque le type de données booléen est un type primitif, vous pouvez aussi le trouver dans l'espace de noms intégré

```{code-cell} python3
print(dir(__builtins__.bool))
```

Ici, nous utilisons une boucle `for` pour filtrer les attributs qui sont appelables

```{code-cell} python3
attributes = dir(__builtins__.bool)
callablels = []

for attribute in attributes:
  # Utilise eval() pour évaluer une chaîne comme une expression
  if callable(eval(f'True.{attribute}')):
    callablels.append(attribute)
print(callablels)
```


```{solution-end}
```