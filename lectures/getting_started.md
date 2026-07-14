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
  title: Premiers pas
  headings:
    Overview: Vue d'ensemble
    Python in the Cloud: Python dans le cloud
    Local Install: Installation locale
    Local Install::The Anaconda Distribution: La distribution Anaconda
    Local Install::Installing Anaconda: Installer Anaconda
    Local Install::Updating `conda`: Mettre à jour `conda`
    Jupyter Notebooks: Notebooks Jupyter
    Jupyter Notebooks::Starting the Jupyter Notebook: Démarrer le notebook Jupyter
    Jupyter Notebooks::Notebook Basics: Bases du notebook
    Jupyter Notebooks::Notebook Basics::Running Cells: Exécuter des cellules
    Jupyter Notebooks::Notebook Basics::Modal Editing: Édition modale
    Jupyter Notebooks::Notebook Basics::Inserting Unicode (e.g., Greek Letters): Insérer de l'Unicode (par exemple, des lettres grecques)
    Jupyter Notebooks::Notebook Basics::A Test Program: Un programme de test
    Jupyter Notebooks::Working with the Notebook: Travailler avec le notebook
    Jupyter Notebooks::Working with the Notebook::Tab Completion: Complétion par tabulation
    Jupyter Notebooks::Working with the Notebook::On-Line Help: Aide en ligne
    Jupyter Notebooks::Working with the Notebook::Other Content: Autre contenu
    Jupyter Notebooks::Debugging Code: Déboguer du code
    Jupyter Notebooks::Sharing Notebooks: Partager des notebooks
    Jupyter Notebooks::QuantEcon Notes: QuantEcon Notes
    Installing Libraries: Installer des bibliothèques
    Working with Python Files: Travailler avec des fichiers Python
    Working with Python Files::Editing and Execution: Édition et exécution
    'Working with Python Files::Editing and Execution::Option 1: JupyterLab': "Option 1\_: JupyterLab"
    'Working with Python Files::Editing and Execution::Option 2: Using a Text Editor': "Option 2\_: Utiliser un éditeur de texte"
    Exercises: Exercices
---

(getting_started)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

<!-- TODO: Review this styling -->

<style>
  .auto {
    width : 70% ;
    height : auto ;
    } 
  .terminal{
    width : 80% ;
    height : auto ;
  }  
</style>


# Premiers pas

```{index} single: Python
```

## Vue d'ensemble

Dans ce cours, vous apprendrez comment

1. utiliser Python dans le cloud
1. mettre en place un environnement Python local et le faire fonctionner
1. exécuter des commandes Python simples
1. exécuter un programme d'exemple
1. installer les bibliothèques de code qui sous-tendent ces cours

## Python dans le cloud

La façon la plus simple de commencer à coder en Python est de l'exécuter dans le cloud.

(C'est-à-dire en utilisant un serveur distant sur lequel Python est déjà installé.)

Une option à la fois gratuite et fiable est [Google Colab](https://colab.research.google.com/).

Colab a également l'avantage de fournir des GPU, que nous utiliserons dans des cours plus avancés.

Des tutoriels sur la façon de débuter avec Google Colab peuvent être trouvés grâce à des recherches sur le web et par vidéo.

La plupart de nos cours incluent un bouton « Launch notebook » (avec une icône de lecture) en haut à droite qui vous connecte à une version exécutable sur Colab.


## Installation locale

Les installations locales sont préférables si vous avez accès à une machine adaptée et prévoyez de faire une quantité substantielle de programmation Python.

En même temps, les installations locales demandent plus de travail qu'une option cloud comme Colab.

Le reste de ce cours vous guide à travers certains détails associés aux installations locales.


### La distribution Anaconda

Le [paquet Python de base](https://www.python.org/downloads/) est facile à installer mais *n'est pas* ce que vous devriez choisir pour ces cours.

Ces cours nécessitent l'ensemble de l'écosystème de programmation scientifique, qui

* n'est pas fourni par l'installation de base
* est pénible à installer pièce par pièce.

Ainsi, la meilleure approche pour nos besoins est d'installer une distribution Python qui contient

1. le langage Python de base **et**
1. des versions compatibles des bibliothèques scientifiques les plus populaires.

La meilleure distribution de ce type est [Anaconda Python](https://www.anaconda.com/).

Anaconda est

* très populaire
* multiplateforme
* complète
* totalement sans rapport avec la [chanson de Nicki Minaj du même nom](https://www.youtube.com/watch?v=LDZX4ooRsWs)

Anaconda est également livré avec un système de gestion de paquets pour organiser vos bibliothèques de code.

**Tout ce qui suit suppose que vous adoptez cette recommandation !**

(install_anaconda)=
### Installer Anaconda

```{index} single: Python; Anaconda
```

Pour installer Anaconda, [téléchargez](https://www.anaconda.com/download) le binaire et suivez les instructions.

Points importants :

* Assurez-vous d'installer la version correcte pour votre système d'exploitation.
* Si on vous demande pendant le processus d'installation si vous souhaitez faire d'Anaconda votre installation Python par défaut, répondez oui.

### Mettre à jour `conda`

Anaconda fournit un outil appelé `conda` pour gérer et mettre à niveau vos paquets Anaconda.

Une commande `conda` que vous devriez exécuter régulièrement est celle qui met à jour l'ensemble de la distribution Anaconda.

À titre d'exercice pratique, veuillez exécuter ce qui suit

1. Ouvrez un terminal
1. Tapez `conda update conda`

Pour plus d'informations sur conda, tapez conda help dans un terminal.

(ipython_notebook)=
## {index}`Notebooks Jupyter <single: Notebooks Jupyter>`

```{index} single: Python; IPython
```

```{index} single: IPython
```

```{index} single: Jupyter
```

Les notebooks [Jupyter](https://jupyter.org/) sont l'une des nombreuses façons possibles d'interagir avec Python et les bibliothèques scientifiques.

Ils utilisent une interface *basée sur le navigateur* pour Python avec

* La capacité d'écrire et d'exécuter des commandes Python.
* Une sortie formatée dans le navigateur, incluant tableaux, figures, animations, etc.
* La possibilité d'y mêler du texte formaté et des expressions mathématiques.

Grâce à ces fonctionnalités, Jupyter est aujourd'hui un acteur majeur de l'écosystème du calcul scientifique.

Voici une image montrant l'exécution de code (emprunté [ici](https://matplotlib.org/stable/gallery/statistics/hexbin_demo.html)) dans un notebook Jupyter

```{figure} /_static/lecture_specific/getting_started/jp_demo.png
:figclass: auto
```

Bien que Jupyter ne soit pas la seule façon de coder en Python, il est idéal lorsque vous souhaitez

* commencer à coder en Python
* tester de nouvelles idées ou interagir avec de petits morceaux de code
* utiliser de puissants environnements interactifs en ligne tels que [Google Colab](https://research.google.com/colaboratory/)
* partager ou collaborer sur des idées scientifiques avec des étudiants ou des collègues

Ces cours sont conçus pour être exécutés dans des notebooks Jupyter.

### Démarrer le notebook Jupyter

```{index} single: Jupyter Notebook; Setup
```

Une fois Anaconda installé, vous pouvez démarrer le notebook Jupyter.

Soit

* recherchez Jupyter dans votre menu d'applications, soit
* ouvrez un terminal et tapez `jupyter notebook`
    * Les utilisateurs de Windows devraient remplacer « terminal » par « invite de commande Anaconda » dans la ligne précédente.

Si vous utilisez la seconde option, vous verrez quelque chose comme ceci

```{figure} /_static/lecture_specific/getting_started/starting_nb.png
:figclass: terminal
```

La sortie nous indique que le notebook s'exécute à `http://localhost:8888/`

* `localhost` est le nom de la machine locale
* `8888` fait référence au [numéro de port](https://en.wikipedia.org/wiki/Port_%28computer_networking%29) 8888 de votre ordinateur

Ainsi, le kernel Jupyter écoute les commandes Python sur le port 8888 de notre machine locale.

Avec un peu de chance, votre navigateur par défaut s'est également ouvert avec une page web qui ressemble à ceci

```{figure} /_static/lecture_specific/getting_started/nb.png
:figclass: auto
```

Ce que vous voyez ici s'appelle le *tableau de bord* Jupyter.

Si vous regardez l'URL en haut, elle devrait être `localhost:8888` ou similaire, correspondant au message ci-dessus.

En supposant que tout cela a bien fonctionné, vous pouvez maintenant cliquer sur `New` en haut à droite et sélectionner `Python 3` ou similaire.

Voici ce qui apparaît sur notre machine :

```{figure} /_static/lecture_specific/getting_started/nb2.png
:figclass: auto
```

Le notebook affiche une *cellule active*, dans laquelle vous pouvez taper des commandes Python.

### Bases du notebook

```{index} single: Jupyter Notebook; Basics
```

Commençons par la manière d'éditer du code et d'exécuter des programmes simples.

#### Exécuter des cellules

Remarquez que, dans la figure précédente, la cellule est entourée d'une bordure verte.

Cela signifie que la cellule est en *mode édition*.

Dans ce mode, tout ce que vous tapez apparaîtra dans la cellule avec le curseur clignotant.

Lorsque vous êtes prêt à exécuter le code d'une cellule, appuyez sur `Shift-Enter` au lieu du `Enter` habituel.

```{figure} /_static/lecture_specific/getting_started/nb3.png
:figclass: auto
```

```{note}
Il existe également des options de menu et de bouton pour exécuter le code d'une cellule que vous pouvez trouver en explorant.
```

#### Édition modale

La prochaine chose à comprendre à propos du notebook Jupyter est qu'il utilise un système d'édition *modal*.

Cela signifie que l'effet de la frappe au clavier **dépend du mode dans lequel vous vous trouvez**.

Les deux modes sont

1. Le mode édition
    * Indiqué par une bordure verte autour d'une cellule, plus un curseur clignotant
    * Tout ce que vous tapez apparaît tel quel dans cette cellule

1. Le mode commande
    * La bordure verte est remplacée par une bordure bleue
    * Les frappes sont interprétées comme des commandes --- par exemple, taper `b` ajoute une nouvelle cellule sous celle en cours

Pour passer

* au mode commande depuis le mode édition, appuyez sur la touche `Esc` ou `Ctrl-M`
* au mode édition depuis le mode commande, appuyez sur `Enter` ou cliquez dans une cellule

Le comportement modal du notebook Jupyter est très efficace une fois que vous vous y habituez.

#### Insérer de l'Unicode (par exemple, des lettres grecques)

Python prend en charge l'[unicode](https://docs.python.org/3/howto/unicode.html), permettant l'utilisation de caractères tels que $\alpha$ et $\beta$ comme noms dans votre code.

Dans une cellule de code, essayez de taper `\alpha` puis d'appuyer sur la touche tab de votre clavier.

(a_test_program)=
#### Un programme de test

Exécutons un programme de test.

Voici un programme arbitraire que nous pouvons utiliser : [https://matplotlib.org/stable/gallery/pie_and_polar_charts/polar_bar.html](https://matplotlib.org/stable/gallery/pie_and_polar_charts/polar_bar.html).

Sur cette page, vous verrez le code suivant

```{code-cell} ipython
import numpy as np
import matplotlib.pyplot as plt

# Fixation de l'état aléatoire pour la reproductibilité
np.random.seed(19680801)

# Calcul des tranches du camembert
N = 20
θ = np.linspace(0.0, 2 * np.pi, N, endpoint=False)
radii = 10 * np.random.rand(N)
width = np.pi / 4 * np.random.rand(N)
colors = plt.cm.viridis(radii / 10.)

ax = plt.subplot(111, projection='polar')
ax.bar(θ, radii, width=width, bottom=0.0, color=colors, alpha=0.5)

plt.show()
```

Ne vous souciez pas des détails pour l'instant --- exécutons-le simplement et voyons ce qui se passe.

La façon la plus simple d'exécuter ce code est de le copier-coller dans une cellule du notebook.

Avec un peu de chance, vous obtiendrez un graphique similaire.

### Travailler avec le notebook

Voici quelques conseils supplémentaires pour travailler avec les notebooks Jupyter.

#### Complétion par tabulation

Dans le programme précédent, nous avons exécuté la ligne `import numpy as np`

* NumPy est une bibliothèque numérique avec laquelle nous travaillerons en profondeur.

Après cette commande d'importation, les fonctions de NumPy peuvent être accessibles avec une syntaxe du type `np.function_name`.

* Par exemple, essayez `np.random.randn(3)`.

Nous pouvons explorer ces attributs de `np` en utilisant la touche `Tab`.

Par exemple, ici nous tapons `np.random.r` et appuyons sur Tab

```{figure} /_static/lecture_specific/getting_started/nb6.png
:figclass: auto
```

Jupyter propose plusieurs complétions possibles parmi lesquelles choisir.

De cette manière, la touche Tab vous aide à vous rappeler ce qui est disponible et vous fait aussi gagner de la frappe.

(gs_help)=
#### Aide en ligne

```{index} single: Jupyter Notebook; Help
```

Pour obtenir de l'aide sur `np.random.randn`, nous pouvons exécuter `np.random.randn?`.

La documentation apparaît dans une fenêtre divisée du navigateur, comme ceci

```{figure} /_static/lecture_specific/getting_started/nb6a.png
:figclass: auto
```

Cliquer en haut à droite de la partie inférieure divisée ferme l'aide en ligne.

Nous en apprendrons davantage sur la façon de créer de la documentation comme celle-ci {ref}`plus tard <Docstrings>` !

#### Autre contenu

En plus d'exécuter du code, le notebook Jupyter vous permet d'intégrer du texte, des équations, des figures et même des vidéos dans la page.

Par exemple, nous pouvons entrer un mélange de texte brut et de LaTeX au lieu de code.

Ensuite, nous appuyons sur `Esc` pour entrer en mode commande, puis tapons `m` pour indiquer que nous écrivons du [Markdown](https://daringfireball.net/projects/markdown/), un langage de balisage similaire (mais plus simple) au LaTeX.

(Vous pouvez également utiliser votre souris pour sélectionner `Markdown` dans la liste déroulante `Code` juste en dessous de la liste des éléments de menu)

```{figure} /_static/lecture_specific/getting_started/nb7.png
:figclass: auto
```

Maintenant, nous faisons `Shift+Enter` pour produire ceci

```{figure} /_static/lecture_specific/getting_started/nb8.png
:figclass: auto
```

### Déboguer du code

```{index} single: Jupyter Notebook; Debugging
```

Le débogage est le processus d'identification et de suppression des erreurs d'un programme.

Vous passerez beaucoup de temps à déboguer du code, il est donc important d'[apprendre à le faire efficacement](https://www.freecodecamp.org/news/what-is-debugging-how-to-debug-code/).

Si vous utilisez une version plus récente de Jupyter, vous devriez voir une icône de bug à l'extrémité droite de la barre d'outils.

```{figure} /_static/lecture_specific/getting_started/debug.png
:scale: 50%
:figclass: auto
```

Cliquer sur cette icône activera le débogueur Jupyter.

<!-- IDEA: This could be turned into a margin note once supported by quantecon-book-theme -->
```{note}
Vous devrez peut-être aussi ouvrir le panneau du débogueur (View -> Debugger Panel).
```

Vous pouvez définir des points d'arrêt en cliquant sur le numéro de ligne de la cellule que vous souhaitez déboguer.

Lorsque vous exécutez la cellule, le débogueur s'arrêtera au point d'arrêt.

Vous pouvez ensuite parcourir le code ligne par ligne en utilisant les boutons du bouton « Next » de la barre d'outils CALLSTACK (située dans la fenêtre de droite).

<!-- IDEA: add a red square around the area of interest in the image -->
```{figure} /_static/lecture_specific/getting_started/debugger_breakpoint.png
:figclass: auto
```

Vous pouvez explorer davantage de fonctionnalités du débogueur dans la [documentation Jupyter](https://jupyterlab.readthedocs.io/en/latest/user/debugger.html).

### Partager des notebooks

```{index} single: Jupyter Notebook; Sharing
```

```{index} single: Jupyter Notebook; nbviewer
```

Les fichiers de notebook sont simplement des fichiers texte structurés en [JSON](https://en.wikipedia.org/wiki/JSON) et se terminant généralement par `.ipynb`.

Vous pouvez les partager de la manière habituelle dont vous partagez des fichiers --- ou en utilisant des services web tels que [nbviewer](https://nbviewer.org/).

Les notebooks que vous voyez sur ce site sont des représentations html **statiques**.

Pour en exécuter un, téléchargez-le sous forme de fichier `ipynb` en cliquant sur l'icône de téléchargement en haut à droite.

Enregistrez-le quelque part, accédez-y depuis le tableau de bord Jupyter puis exécutez-le comme décrit ci-dessus.

```{note}
Si vous souhaitez partager des notebooks contenant du contenu interactif, vous pourriez vouloir consulter [Binder](https://mybinder.org/).

Pour collaborer avec d'autres personnes sur des notebooks, vous pourriez jeter un œil à

- [Google Colab](https://colab.research.google.com/)
- [Kaggle](https://www.kaggle.com/code)

Pour garder le code privé et utiliser l'interface familière de JupyterLab et Notebook, renseignez-vous sur l'[extension JupyterLab Real-Time Collaboration](https://jupyterlab-realtime-collaboration.readthedocs.io/en/latest/).
```

### QuantEcon Notes

QuantEcon possède son propre site pour partager des notebooks Jupyter liés à l'économie -- [QuantEcon Notes](http://notes.quantecon.org/).

Les notebooks soumis à QuantEcon Notes peuvent être partagés via un lien, et sont ouverts aux commentaires et aux votes de la communauté.

## Installer des bibliothèques

(gs_qe)=
```{index} single: QuantEcon
```

La plupart des bibliothèques dont nous avons besoin sont incluses dans Anaconda.

D'autres bibliothèques peuvent être installées avec `pip` ou `conda`.

Une bibliothèque que nous utiliserons est [QuantEcon.py](https://quantecon.org/quantecon-py/).

(gs_install_qe)=
Vous pouvez installer [QuantEcon.py](https://quantecon.org/quantecon-py/) en démarrant Jupyter et en tapant

```{code-block} ipython3
:class: no-execute

!conda install quantecon
```

dans une cellule.

Alternativement, vous pouvez taper ce qui suit dans un terminal

```{code-block} bash
:class: no-execute

conda install quantecon
```

Plus d'instructions peuvent être trouvées sur la [page de la bibliothèque](https://quantecon.org/quantecon-py/).

Pour passer à la dernière version, ce que vous devriez faire régulièrement, utilisez

```{code-block} bash
:class: no-execute

conda upgrade quantecon
```

Une autre bibliothèque que nous utiliserons est [interpolation.py](https://github.com/EconForge/interpolation.py).

Elle peut être installée en tapant dans Jupyter

```{code-block} ipython3
:class: no-execute

!conda install -c conda-forge interpolation
```

## Travailler avec des fichiers Python

Jusqu'à présent, nous nous sommes concentrés sur l'exécution de code Python saisi dans une cellule de notebook Jupyter.

Traditionnellement, la plupart du code Python a été exécuté d'une manière différente.

Le code est d'abord enregistré dans un fichier texte sur une machine locale

Par convention, ces fichiers texte ont une extension `.py`.

Nous pouvons créer un exemple d'un tel fichier comme suit :

```{code-cell} ipython
%%writefile foo.py

print("foobar")
```

Ceci écrit la ligne `print("foobar")` dans un fichier appelé `foo.py` dans le répertoire local.

Ici, `%%writefile` est un exemple de [cell magic](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cell-magics).

### Édition et exécution

Si vous tombez sur du code enregistré dans un fichier `*.py`, vous devrez considérer les questions suivantes :

1. comment devriez-vous l'exécuter ?
1. Comment devriez-vous le modifier ou l'éditer ?

#### Option 1 : {index}`JupyterLab <single: JupyterLab>`

```{index} single: JupyterLab
```

[JupyterLab](https://github.com/jupyterlab/jupyterlab) est un environnement de développement intégré construit au-dessus des notebooks Jupyter.

Avec JupyterLab, vous pouvez éditer et exécuter des fichiers `*.py` ainsi que des notebooks Jupyter.

Pour démarrer JupyterLab, recherchez-le dans le menu des applications ou tapez `jupyter-lab` dans un terminal.

Vous devriez maintenant pouvoir ouvrir, éditer et exécuter le fichier `foo.py` créé ci-dessus en l'ouvrant dans JupyterLab.

Lisez la documentation ou recherchez une vidéo YouTube récente pour trouver plus d'informations.

#### Option 2 : Utiliser un éditeur de texte

On peut également éditer des fichiers en utilisant un éditeur de texte puis les exécuter depuis des notebooks Jupyter.

Un éditeur de texte est une application spécifiquement conçue pour travailler avec des fichiers texte --- tels que des programmes Python.

Rien ne surpasse la puissance et l'efficacité d'un bon éditeur de texte pour travailler avec du texte de programme.

Un bon éditeur de texte fournira

* des commandes d'édition de texte efficaces (par exemple, copier, coller, rechercher et remplacer)
* la coloration syntaxique, etc.

À l'heure actuelle, un éditeur de texte extrêmement populaire pour coder est [VS Code](https://code.visualstudio.com/).

VS Code est facile à utiliser dès l'installation et possède de nombreuses extensions de haute qualité.

Alternativement, si vous voulez un éditeur de texte gratuit exceptionnel et que cela ne vous dérange pas une courbe d'apprentissage apparemment verticale ainsi que de longues journées de peine et de souffrance pendant que toutes vos voies neuronales sont recâblées, essayez [Vim](https://www.vim.org/).

## Exercices

```{exercise-start}
:label: gs_ex1
```

Si Jupyter est toujours en cours d'exécution, quittez en utilisant `Ctrl-C` dans le terminal où vous l'avez démarré.

Maintenant, relancez, mais cette fois en utilisant `jupyter notebook --no-browser`.

Ceci devrait démarrer le kernel sans lancer le navigateur.

Notez également le message de démarrage : il devrait vous donner une URL telle que `http://localhost:8888` où le notebook s'exécute.

Maintenant

1. Démarrez votre navigateur --- ou ouvrez un nouvel onglet s'il est déjà en cours d'exécution.
1. Entrez l'URL d'en haut (par exemple `http://localhost:8888`) dans la barre d'adresse en haut.

Vous devriez maintenant pouvoir exécuter une session standard de notebook Jupyter.

C'est une manière alternative de démarrer le notebook qui peut aussi être pratique.

Cela peut également fonctionner lorsque vous fermez accidentellement la page web tant que le kernel est toujours en cours d'exécution.

```{exercise-end}
```