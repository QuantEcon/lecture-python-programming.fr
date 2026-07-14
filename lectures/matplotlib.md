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
  title: Matplotlib
  headings:
    Overview: Vue d'ensemble
    Overview::Matplotlib's Split Personality: La double personnalité de Matplotlib
    The APIs: Les API
    The APIs::The MATLAB-style API: L'API de style MATLAB
    The APIs::The Object-Oriented API: L'API orientée objet
    The APIs::Tweaks: Ajustements
    More Features: Fonctionnalités supplémentaires
    More Features::Multiple Plots on One Axis: Plusieurs tracés sur un même axe
    More Features::Multiple Subplots: Plusieurs sous-graphiques
    More Features::3D Plots: Tracés 3D
    More Features::A Customizing Function: Une fonction de personnalisation
    More Features::Style Sheets: Feuilles de style
    Further Reading: Pour aller plus loin
    Exercises: Exercices
---

(matplotlib)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# {index}`Matplotlib <single: Matplotlib>`

```{index} single: Python; Matplotlib
```

## Vue d'ensemble

Nous avons déjà produit un certain nombre de figures dans ces cours en utilisant [Matplotlib](https://matplotlib.org/).

Matplotlib est une bibliothèque graphique remarquable, conçue pour le calcul scientifique, avec

* des graphiques 2D et 3D de haute qualité
* une sortie dans tous les formats habituels (PDF, PNG, etc.)
* une intégration LaTeX
* un contrôle fin de tous les aspects de la présentation
* de l'animation, etc.

### La double personnalité de Matplotlib

Matplotlib est inhabituelle dans le sens où elle offre deux interfaces différentes pour tracer des graphiques.

L'une est une simple API (Application Programming Interface) de style MATLAB, qui a été écrite pour aider les réfugiés de MATLAB à trouver un environnement familier.

L'autre est une API orientée objet plus « pythonique ».

Pour les raisons décrites ci-dessous, nous vous recommandons d'utiliser la seconde API.

Mais commençons par discuter de la différence.

## Les API

```{index} single: Matplotlib; Simple API
```

### L'API de style MATLAB

Voici le genre d'exemple simple que vous pourriez trouver dans les traitements introductifs

```{code-cell} ipython
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 200)
y = np.sin(x)

plt.plot(x, y, 'b-', linewidth=2)
plt.show()
```

C'est simple et pratique, mais aussi quelque peu limité et peu pythonique.

Par exemple, dans les appels de fonction, beaucoup d'objets sont créés et transmis sans se faire connaître du programmeur.

Les programmeurs Python tendent à préférer un style de programmation plus explicite (exécutez `import this` dans un bloc de code et regardez la deuxième ligne).

Cela nous conduit à l'API alternative, orientée objet, de Matplotlib.

### L'API orientée objet

Voici le code correspondant à la figure précédente en utilisant l'API orientée objet

```{code-cell} python3
fig, ax = plt.subplots()
ax.plot(x, y, 'b-', linewidth=2)
plt.show()
```

Ici, l'appel `fig, ax = plt.subplots()` renvoie une paire, où

* `fig` est une instance de `Figure` — comme une toile vierge.
* `ax` est une instance d'`AxesSubplot` — pensez à un cadre dans lequel tracer.

La fonction `plot()` est en réalité une méthode de `ax`.

Bien qu'il y ait un peu plus de saisie, l'utilisation plus explicite des objets nous donne un meilleur contrôle.

Cela deviendra plus clair au fur et à mesure.

### Ajustements

Ici, nous avons changé la ligne en rouge et ajouté une légende

```{code-cell} python3
fig, ax = plt.subplots()
ax.plot(x, y, 'r-', linewidth=2, label='fonction sinus', alpha=0.6)
ax.legend()
plt.show()
```

Nous avons également utilisé `alpha` pour rendre la ligne légèrement transparente — ce qui la fait paraître plus lisse.

L'emplacement de la légende peut être modifié en remplaçant `ax.legend()` par `ax.legend(loc='upper center')`.

```{code-cell} python3
fig, ax = plt.subplots()
ax.plot(x, y, 'r-', linewidth=2, label='fonction sinus', alpha=0.6)
ax.legend(loc='upper center')
plt.show()
```

Si tout est correctement configuré, alors ajouter du LaTeX est trivial

```{code-cell} python3
fig, ax = plt.subplots()
ax.plot(x, y, 'r-', linewidth=2, label=r'$y=\sin(x)$', alpha=0.6)
ax.legend(loc='upper center')
plt.show()
```

Contrôler les graduations, ajouter des titres, etc. est également simple

```{code-cell} python3
fig, ax = plt.subplots()
ax.plot(x, y, 'r-', linewidth=2, label=r'$y=\sin(x)$', alpha=0.6)
ax.legend(loc='upper center')
ax.set_yticks([-1, 0, 1])
ax.set_title('Graphique de test')
plt.show()
```

## Fonctionnalités supplémentaires

Matplotlib dispose d'un vaste ensemble de fonctions et de fonctionnalités, que vous pourrez découvrir au fil du temps selon vos besoins.

Nous n'en mentionnons que quelques-unes.

### Plusieurs tracés sur un même axe

```{index} single: Matplotlib; Multiple Plots on One Axis
```

Il est facile de générer plusieurs tracés sur les mêmes axes.

Voici un exemple qui génère aléatoirement trois densités normales et ajoute une étiquette avec leur moyenne

```{code-cell} python3
from scipy.stats import norm
from random import uniform

fig, ax = plt.subplots()
x = np.linspace(-4, 4, 150)
for i in range(3):
    m, s = uniform(-1, 1), uniform(1, 2)
    y = norm.pdf(x, loc=m, scale=s)
    current_label = rf'$\mu = {m:.2}$'
    ax.plot(x, y, linewidth=2, alpha=0.6, label=current_label)
ax.legend()
plt.show()
```

### Plusieurs sous-graphiques

```{index} single: Matplotlib; Subplots
```

Parfois, nous voulons plusieurs sous-graphiques dans une seule figure.

Voici un exemple qui génère 6 histogrammes

```{code-cell} python3
num_rows, num_cols = 3, 2
fig, axes = plt.subplots(num_rows, num_cols, figsize=(10, 12))
for i in range(num_rows):
    for j in range(num_cols):
        m, s = uniform(-1, 1), uniform(1, 2)
        x = norm.rvs(loc=m, scale=s, size=100)
        axes[i, j].hist(x, alpha=0.6, bins=20)
        t = rf'$\mu = {m:.2}, \quad \sigma = {s:.2}$'
        axes[i, j].set(title=t, xticks=[-4, 0, 4], yticks=[])
plt.show()
```

### Tracés 3D

```{index} single: Matplotlib; 3D Plots
```

Matplotlib fait un bon travail pour les tracés 3D — en voici un exemple

```{code-cell} python3
from mpl_toolkits.mplot3d.axes3d import Axes3D
from matplotlib import cm


def f(x, y):
    return np.cos(x**2 + y**2) / (1 + x**2 + y**2)

xgrid = np.linspace(-3, 3, 50)
ygrid = xgrid
x, y = np.meshgrid(xgrid, ygrid)

fig = plt.figure(figsize=(10, 6))
ax = fig.add_subplot(111, projection='3d')
ax.plot_surface(x,
                y,
                f(x, y),
                rstride=2, cstride=2,
                cmap=cm.jet,
                alpha=0.7,
                linewidth=0.25)
ax.set_zlim(-0.5, 1.0)
plt.show()
```

### Une fonction de personnalisation

Peut-être trouverez-vous un ensemble de personnalisations que vous utilisez régulièrement.

Supposons que nous préférions généralement que nos axes passent par l'origine et qu'ils comportent une grille.

Voici un bel exemple de [Matthew Doty](https://github.com/xcthulhu) montrant comment l'API orientée objet peut être utilisée pour construire une fonction `subplots` personnalisée qui implémente ces changements.

Lisez attentivement le code et voyez si vous pouvez suivre ce qui se passe

```{code-cell} python3
def subplots():
    "Sous-graphiques personnalisés avec des axes passant par l'origine"
    fig, ax = plt.subplots()

    # Placer les axes de façon à ce qu'ils passent par l'origine
    for spine in ['left', 'bottom']:
        ax.spines[spine].set_position('zero')
    for spine in ['right', 'top']:
        ax.spines[spine].set_color('none')

    ax.grid()
    return fig, ax


fig, ax = subplots()  # Appeler la version locale, et non plt.subplots()
x = np.linspace(-2, 10, 200)
y = np.sin(x)
ax.plot(x, y, 'r-', linewidth=2, label='fonction sinus', alpha=0.6)
ax.legend(loc='lower right')
plt.show()
```

La fonction `subplots` personnalisée

1. appelle en interne la fonction standard `plt.subplots` pour générer la paire `fig, ax`,
1. effectue les personnalisations souhaitées sur `ax`, et
1. renvoie la paire `fig, ax` au code appelant.

### Feuilles de style

Une autre fonctionnalité utile de Matplotlib est celle des [feuilles de style](https://matplotlib.org/stable/gallery/style_sheets/style_sheets_reference.html).

Nous pouvons utiliser des feuilles de style pour créer des graphiques avec des styles uniformes.

Nous pouvons obtenir une liste des styles disponibles en affichant l'attribut `plt.style.available`


```{code-cell} python3
print(plt.style.available)
```

Nous pouvons maintenant utiliser la méthode `plt.style.use()` pour définir la feuille de style.

Écrivons une fonction qui prend le nom d'une feuille de style et trace différents graphiques avec ce style

```{code-cell} python3

def draw_graphs(style='default'):

    # Définir une feuille de style
    plt.style.use(style)

    fig, axes = plt.subplots(nrows=1, ncols=4, figsize=(10, 3))
    x = np.linspace(-13, 13, 150)

    # Fixer les valeurs de graine pour reproduire les résultats des tirages aléatoires
    np.random.seed(9)

    for i in range(3):

        # Tirer la moyenne et l'écart-type à partir de lois uniformes
        m, s = np.random.uniform(-8, 8), np.random.uniform(2, 2.5)

        # Générer un tracé de densité normale
        y = norm.pdf(x, loc=m, scale=s)
        axes[0].plot(x, y, linewidth=3, alpha=0.7)

        # Créer un nuage de points avec des valeurs X et Y aléatoires 
        # tirées de lois normales
        rnormX = norm.rvs(loc=m, scale=s, size=150)
        rnormY = norm.rvs(loc=m, scale=s, size=150)
        axes[1].plot(rnormX, rnormY, ls='none', marker='o', alpha=0.7)

        # Créer un histogramme avec des valeurs X aléatoires
        axes[2].hist(rnormX, alpha=0.7)

        # et un graphique en ligne avec des valeurs Y aléatoires
        axes[3].plot(x, rnormY, linewidth=2, alpha=0.7)

    style_name = style.split('-')[0]
    plt.suptitle(f'Style : {style_name}', fontsize=13)
    plt.show()

```

Voyons à quoi ressemblent quelques-uns de ces styles.

D'abord, nous traçons des graphiques avec la feuille de style `seaborn`

```{code-cell} python3
draw_graphs(style='seaborn-v0_8')
```

Nous pouvons utiliser `grayscale` pour supprimer les couleurs des graphiques

```{code-cell} python3
draw_graphs(style='grayscale')
```

Voici à quoi ressemble `ggplot`

```{code-cell} python3
draw_graphs(style='ggplot')
```

Nous pouvons aussi utiliser le style `dark_background`

```{code-cell} python3
draw_graphs(style='dark_background')
```

Vous pouvez utiliser la fonction pour expérimenter avec d'autres styles de la liste.

Si cela vous intéresse, vous pouvez même créer vos propres feuilles de style.

Les paramètres de vos feuilles de style sont stockés dans une variable de type dictionnaire `plt.rcParams`

```{code-cell} python3
---
tags: [hide-output]
---
 
print(plt.rcParams.keys())

```

Il existe de nombreux paramètres que vous pourriez définir pour vos feuilles de style.

Définissez les paramètres de votre feuille de style en : 

1. créant votre propre [fichier `matplotlibrc`](https://matplotlib.org/stable/users/explain/customizing.html), ou
2. mettant à jour les valeurs stockées dans la variable de type dictionnaire `plt.rcParams`

Changeons le style de nos lignes de densité superposées en utilisant la seconde méthode

```{code-cell} python3
from cycler import cycler

# revenir à la feuille de style par défaut
plt.style.use('default')

# Vous pouvez mettre à jour des valeurs individuelles à l'aide des clés :

# Définir le style de police en italique
plt.rcParams['font.style'] = 'italic'

# Mettre à jour la largeur de ligne
plt.rcParams['lines.linewidth'] = 2


# Vous pouvez également mettre à jour plusieurs valeurs à la fois avec la méthode update() :

parameters = {

    # Modifier la taille de figure par défaut
    'figure.figsize': (5, 4),

    # Ajouter des lignes de grille horizontales
    'axes.grid': True,
    'axes.grid.axis': 'y',

    # Mettre à jour les couleurs des lignes de densité
    'axes.prop_cycle': cycler('color', 
                            ['dimgray', 'slategrey', 'darkgray'])
}

plt.rcParams.update(parameters)


```

```{note} 

Ces réglages sont `globaux`. 

Tout graphique généré après avoir modifié les paramètres dans `.rcParams` sera affecté par ce réglage.

```

```{code-cell} python3
fig, ax = plt.subplots()
x = np.linspace(-4, 4, 150)
for i in range(3):
    m, s = uniform(-1, 1), uniform(1, 2)
    y = norm.pdf(x, loc=m, scale=s)
    current_label = rf'$\mu = {m:.2}$'
    ax.plot(x, y, linewidth=2, alpha=0.6, label=current_label)
ax.legend()
plt.show()
```

Appliquez à nouveau la feuille de style `default` pour revenir au style par défaut

```{code-cell} python3

plt.style.use('default')

# Réinitialiser la taille de figure par défaut
plt.rcParams['figure.figsize'] = (10, 6)

```

## Pour aller plus loin

* La [galerie Matplotlib](https://matplotlib.org/stable/gallery/index.html) propose de nombreux exemples.
* Un excellent [tutoriel Matplotlib](https://scipy-lectures.org/intro/matplotlib/index.html) de Nicolas Rougier, Mike Muller et Gael Varoquaux.
* [mpltools](https://tonysyu.github.io/mpltools/index.html) permet de basculer facilement entre les styles de tracé.
* [Seaborn](https://github.com/mwaskom/seaborn) facilite les graphiques statistiques courants dans Matplotlib.

## Exercices

```{exercise-start}
:label: mpl_ex1
```

Tracez la fonction

$$
f(x) = \cos(\pi \theta x) \exp(-x)
$$

sur l'intervalle $[0, 5]$ pour chaque $\theta$ dans `np.linspace(0, 2, 10)`.

Placez toutes les courbes dans la même figure.

Le résultat devrait ressembler à ceci

```{image} /_static/lecture_specific/matplotlib/matplotlib_ex1.png
:scale: 130
:align: center
```

```{exercise-end}
```

```{solution-start} mpl_ex1
:class: dropdown
```

Voici une solution

```{code-cell} ipython3
def f(x, θ):
    return np.cos(np.pi * θ * x ) * np.exp(- x)

θ_vals = np.linspace(0, 2, 10)
x = np.linspace(0, 5, 200)
fig, ax = plt.subplots()

for θ in θ_vals:
    ax.plot(x, f(x, θ))

plt.show()
```

```{solution-end}
```