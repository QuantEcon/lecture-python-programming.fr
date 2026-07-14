---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.17.2
kernelspec:
  name: python3
  display_name: Python 3 (ipykernel)
  language: python
translation:
  title: Écrire des programmes plus longs
  headings:
    Overview: Vue d'ensemble
    Working with Python files: Travailler avec des fichiers Python
    Development environments: Environnements de développement
    'A step forward from Jupyter Notebooks: JupyterLab': "Un pas en avant par rapport aux notebooks Jupyter\_: JupyterLab"
    'A step forward from Jupyter Notebooks: JupyterLab::Using magic commands': En utilisant les commandes magiques
    'A step forward from Jupyter Notebooks: JupyterLab::Using the terminal': En utilisant le terminal
    A walk through Visual Studio Code: Une visite guidée de Visual Studio Code
    A walk through Visual Studio Code::Using the run button: En utilisant le bouton d'exécution
    A walk through Visual Studio Code::Using the terminal: En utilisant le terminal
    Git your hands dirty: Mettez la main à la pâte avec Git
---

(workspace)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Écrire des programmes plus longs

## Vue d'ensemble

Jusqu'à présent, nous avons exploré l'utilisation des notebooks Jupyter pour écrire et exécuter du code Python.

Bien qu'ils soient efficaces et adaptables lorsqu'on travaille avec de courts fragments de code, les notebooks ne constituent pas le meilleur choix pour les programmes et scripts plus longs.

Les notebooks Jupyter sont bien adaptés au calcul interactif (c'est-à-dire aux flux de travail en science des données) et peuvent aider à exécuter des blocs de code un à la fois.

Les fichiers texte et les scripts permettent d'écrire et d'exécuter de longs fragments de code en une seule fois.

Nous allons explorer l'utilisation des scripts Python comme alternative.

Les environnements de développement Jupyter Lab et Visual Studio Code (VS Code) sont ensuite présentés, accompagnés d'une introduction au contrôle de version (Git).

Dans ce cours, vous apprendrez à
- travailler avec des scripts Python
- configurer divers environnements de développement
- débuter avec GitHub

```{note}
Dans la suite, on suppose que vous disposez d'un environnement Anaconda opérationnel.

Vous voudrez peut-être [créer un nouvel environnement conda](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-with-commands) si ce n'est pas déjà fait.
```

## Travailler avec des fichiers Python

Les fichiers Python sont utilisés pour écrire de longs blocs de code réutilisables — par convention, ils portent le suffixe `.py`.

Commençons par travailler avec l'exemple suivant.

```{code-cell} ipython3
:caption: sine_wave.py
:lineno-start: 1

import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y = np.sin(x)

plt.plot(x, y)
plt.xlabel('x')
plt.ylabel('y')
plt.title('Sine Wave')
plt.show()
```

Comme il existe diverses façons d'exécuter le code, nous les explorerons dans le contexte de différents environnements de développement.

L'un des principaux avantages de l'utilisation des scripts Python réside dans le fait que vous pouvez « importer » des fonctionnalités depuis d'autres scripts vers votre script actuel ou votre notebook Jupyter.

Réécrivons le code précédent sous forme de fonction et écrivons-le dans un fichier appelé `sine_wave.py`.

```{code-cell} ipython3
:caption: sine_wave.py
:lineno-start: 1

%%writefile sine_wave.py

import matplotlib.pyplot as plt
import numpy as np

# Définition de la fonction plot_wave.
def plot_wave(title : str = 'Sine Wave'):
  x = np.linspace(0, 10, 100)
  y = np.sin(x)

  plt.plot(x, y)
  plt.xlabel('x')
  plt.ylabel('y')
  plt.title(title)
  plt.show()
```

```{code-cell} ipython3
:caption: second_script.py
:lineno-start: 1

import sine_wave # Import du script sine_wave
 
# Appel de la fonction plot_wave.
sine_wave.plot_wave("Sine Wave - Called from the Second Script")
```

Cela vous permet de découper votre code en morceaux et de mieux structurer votre base de code.

Renseignez-vous sur l'utilisation des [modules](https://docs.python.org/3/tutorial/modules.html) et des [paquets](https://docs.python.org/3/tutorial/modules.html#packages) pour en savoir plus sur l'importation de fonctionnalités.

## Environnements de développement

Un environnement de développement est un espace de travail unique où vous pouvez
- éditer et exécuter votre code
- tester et déboguer
- gérer les fichiers du projet

Ce cours vous fait découvrir le fonctionnement de deux environnements de développement.

## Un pas en avant par rapport aux notebooks Jupyter : JupyterLab

JupyterLab est un environnement de développement basé sur le navigateur, destiné aux notebooks Jupyter, aux scripts de code et aux fichiers de données.

Vous pouvez [essayer JupyterLab dans le navigateur](https://jupyter.org/try-jupyter/lab/) si vous souhaitez le tester avant de l'installer localement.

Vous pouvez installer JupyterLab à l'aide de pip

```
> pip install jupyterlab
``` 

et le lancer dans le navigateur, de manière similaire aux notebooks Jupyter.

```
> jupyter-lab
```

```{figure} /_static/lecture_specific/workspace/jupyter_lab_cmd.png
:figclass: auto
```

Vous pouvez voir que le serveur Jupyter s'exécute sur le port 8888 de localhost.

L'interface suivante devrait s'ouvrir automatiquement dans votre navigateur par défaut — sinon, faites CTRL + Clic sur l'URL du serveur.

```{figure} /_static/lecture_specific/workspace/jupyter_lab.png
:figclass: auto
```

Cliquez sur

- le bouton Python 3 (ipykernel) sous Notebooks pour ouvrir un nouveau notebook Jupyter
- le bouton Python File pour ouvrir un nouveau script Python (.py)

Vous pouvez toujours ouvrir cet onglet de lancement en cliquant sur le bouton « + » en haut.

Tous les fichiers et dossiers de votre répertoire de travail se trouvent dans l'explorateur de fichiers (onglet à gauche).

Vous pouvez créer de nouveaux fichiers et dossiers à l'aide des boutons disponibles en haut de l'onglet de l'explorateur de fichiers.

```{figure} /_static/lecture_specific/workspace/file_browser.png
:figclass: auto
```
Vous pouvez installer des extensions qui augmentent les fonctionnalités de JupyterLab en visitant l'onglet Extensions.

```{figure} /_static/lecture_specific/workspace/extensions.png
:figclass: auto
```
Pour revenir aux scripts d'exemple présentés plus tôt, il existe deux façons de travailler avec eux dans JupyterLab.

- En utilisant les commandes magiques
- En utilisant le terminal

### En utilisant les commandes magiques

Les notebooks Jupyter et JupyterLab prennent en charge l'utilisation de [commandes magiques](https://ipython.readthedocs.io/en/stable/interactive/magics.html) — des commandes qui étendent les capacités d'un notebook Jupyter standard.

La commande magique `%run` vous permet d'exécuter un script Python depuis un notebook.

C'est une manière pratique d'exécuter des scripts sur lesquels vous travaillez dans le même répertoire que votre notebook et de présenter les résultats au sein du notebook.

```{figure} /_static/lecture_specific/workspace/jupyter_lab_py_run.png
:figclass: auto
```

### En utilisant le terminal

Cependant, si vous cherchez simplement à exécuter le fichier `.py`, il est parfois plus facile d'utiliser le terminal.

Ouvrez un terminal depuis le lanceur et exécutez la commande suivante.

```
> python <path to file.py>
``` 

```{figure} /_static/lecture_specific/workspace/jupyter_lab_py_run_term.png
:figclass: auto
```

```{note}
Vous pouvez également exécuter le script ligne par ligne en ouvrant une console ipykernel, soit
- depuis le lanceur
- en faisant un clic droit dans le notebook et en sélectionnant Create Console for Editor

Utilisez Shift + Enter pour exécuter une ligne de code.
```

## Une visite guidée de Visual Studio Code

Visual Studio Code (VS Code) est un éditeur de code et un espace de travail de développement qui peut s'exécuter
- dans le [navigateur](https://vscode.dev/).
- en tant qu'[installation](https://code.visualstudio.com/docs/?dv=win) locale.

Les deux interfaces sont identiques.

Lorsque vous lancez VS Code, vous verrez l'interface suivante.

```{figure} /_static/lecture_specific/workspace/vs_code_home.png
:figclass: auto
```

Découvrez comment personnaliser VS Code selon vos goûts grâce aux visites guidées.

```{figure} /_static/lecture_specific/workspace/vs_code_walkthrough.png
:figclass: auto
```
Lorsque l'invite suivante s'affiche, installez toutes les extensions recommandées.

```{figure} /_static/lecture_specific/workspace/vs_code_install_ext.png
:figclass: auto
```
Vous pouvez également installer des extensions depuis l'onglet Extensions.

```{figure} /_static/lecture_specific/workspace/vs_code_extensions.png
:figclass: auto
```
Les notebooks Jupyter (fichiers `.ipynb`) peuvent être travaillés dans VS Code.

Assurez-vous d'installer l'extension Jupyter depuis l'onglet Extensions avant d'essayer d'ouvrir un notebook Jupyter.

Créez un nouveau fichier (dans l'onglet Explorateur de fichiers) et enregistrez-le avec l'extension `.ipynb`.

Choisissez un noyau/environnement dans lequel exécuter le notebook en cliquant sur le bouton Select Kernel dans le coin supérieur droit de l'éditeur.

```{figure} /_static/lecture_specific/workspace/vs_code_kernels.png
:figclass: auto
```

VS Code dispose également d'excellentes fonctionnalités de contrôle de version via l'onglet Source Control.

```{figure} /_static/lecture_specific/workspace/vs_code_git.png
:figclass: auto
```
Liez votre compte GitHub à VS Code pour envoyer (push) et récupérer (pull) des modifications vers et depuis vos dépôts.

D'autres discussions sur le contrôle de version se trouvent dans la section suivante.

Pour ouvrir un nouveau terminal dans VS Code, cliquez sur l'onglet Terminal et sélectionnez New Terminal.

VS Code ouvre un nouveau terminal dans le répertoire dans lequel vous travaillez — un PowerShell sous Windows et un Bash sous Linux.

Vous pouvez changer de shell ou ouvrir une nouvelle instance via le menu déroulant situé à l'extrémité droite de l'onglet du terminal.

```{figure} /_static/lecture_specific/workspace/vs_code_terminal_opts.png
:figclass: auto
```

VS Code vous aide à gérer les environnements conda sans utiliser la ligne de commande.

Ouvrez la palette de commandes (CTRL + SHIFT + P ou depuis le menu déroulant sous l'onglet View) et recherchez ```Python: Select Interpreter```.

Cela charge les environnements existants.

Vous pouvez également créer de nouveaux environnements en utilisant ```Python: Create Environment``` dans la palette de commandes.

Un nouvel environnement (dossier .conda) est créé dans le répertoire de travail actuel.

Pour en venir aux scripts d'exemple présentés plus tôt, il existe là encore deux façons de travailler avec eux dans VS Code.

- En utilisant le bouton d'exécution
- En utilisant le terminal

### En utilisant le bouton d'exécution

Vous pouvez exécuter le script en cliquant sur le bouton d'exécution dans le coin supérieur droit de l'éditeur.

```{figure} /_static/lecture_specific/workspace/vs_code_run.png
:figclass: auto
```

Vous pouvez également exécuter le script de manière interactive en sélectionnant l'option **Run Current File in Interactive Window** dans le menu déroulant.

```{figure} /_static/lecture_specific/workspace/vs_code_run_button.png
:figclass: auto
```
Cela crée une console ipykernel et exécute le script.

### En utilisant le terminal

La commande `python <path to file.py>` est exécutée sur la console de votre choix.

Si vous utilisez une machine Windows, vous pouvez utiliser soit l'invite Anaconda, soit l'invite de commande — mais, en général, pas PowerShell.

Voici une exécution du code précédent.

```{figure} /_static/lecture_specific/workspace/sine_wave_import.png
:figclass: auto
```

```{note}
Si vous souhaitez développer des paquets et créer des outils avec Python, vous voudrez peut-être vous renseigner sur [l'utilisation des conteneurs Docker et de VS Code](https://github.com/RamiKrispin/vscode-python).

Cependant, cela sort du cadre de ces cours.
```

## Mettez la main à la pâte avec Git

Cette section vous familiarisera avec git et GitHub.

[Git](https://git-scm.com/) est un *système de contrôle de version* — un logiciel utilisé pour gérer des projets numériques tels que des bibliothèques de code.

Dans de nombreux cas, les collections de fichiers associées — appelées *dépôts* — sont stockées sur [GitHub](https://github.com/).

GitHub est un pays des merveilles rempli de projets de codage collaboratif.

Par exemple, il héberge de nombreuses bibliothèques scientifiques que nous utiliserons plus tard, comme [celle-ci](https://github.com/pandas-dev/pandas).

Git est le logiciel sous-jacent utilisé pour gérer ces projets.

Git est un outil extrêmement puissant pour la collaboration distribuée — par exemple, nous l'utilisons pour partager et synchroniser tous les fichiers sources de ces cours.

Il existe deux principales variantes de Git

1. la version classique [Git en ligne de commande](https://git-scm.com/downloads/)
2. les diverses versions graphiques (GUI) à cliquer
    * Voir, par exemple, la [version GitHub](https://github.com/apps/desktop) ou l'interface graphique Git intégrée à votre IDE.

Si ce n'est pas déjà fait, essayez de

1. Installer Git.
1. Obtenir une copie de [QuantEcon.py](https://github.com/QuantEcon/QuantEcon.py) à l'aide de Git.

Par exemple, si vous avez installé la version en ligne de commande, ouvrez un terminal et saisissez.

```bash
git clone https://github.com/QuantEcon/QuantEcon.py
```
(C'est simplement `git clone` devant l'URL du dépôt)

Cette commande téléchargera tous les composants nécessaires pour reconstruire le cours que vous êtes en train de lire.

Comme deuxième tâche,

1. Inscrivez-vous sur [GitHub](https://github.com/).
1. Renseignez-vous sur le « forking » des dépôts GitHub (faire un fork signifie créer votre propre copie d'un dépôt GitHub, stockée sur GitHub).
1. Faites un fork de [QuantEcon.py](https://github.com/QuantEcon/QuantEcon.py).
1. Clonez votre fork dans un répertoire local, apportez des modifications, validez-les (commit) et renvoyez-les (push) vers votre dépôt GitHub forké.
1. Si vous avez apporté une amélioration précieuse, envoyez-nous une [pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests) !

Pour approfondir ces sujets et d'autres, essayez

* [La documentation officielle de Git](https://git-scm.com/doc).
* La lecture de la documentation sur [GitHub](https://docs.github.com/en).
* [Pro Git Book](https://git-scm.com/book) par Scott Chacon et Ben Straub.
* L'un des milliers de tutoriels Git disponibles sur le Net.