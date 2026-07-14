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
  title: À propos de ces cours
  headings:
    Overview: Vue d'ensemble
    Overview::Can't I Just Use LLMs?: "Ne puis-je pas simplement utiliser des LLM\_?"
    Overview::Isn't MATLAB Better?: "MATLAB n'est-il pas meilleur\_?"
    Introducing Python: Présentation de Python
    Introducing Python::Common Uses: Usages courants
    Introducing Python::Relative Popularity: Popularité relative
    Introducing Python::Features: Fonctionnalités
    Introducing Python::Syntax and Design: Syntaxe et conception
    Introducing Python::The AI Connection: Le lien avec l'IA
    Scientific Programming with Python: Programmation scientifique avec Python
    Scientific Programming with Python::NumPy: NumPy
    Scientific Programming with Python::NumPy Alternatives: Alternatives à NumPy
    Scientific Programming with Python::SciPy: SciPy
    Scientific Programming with Python::Graphics: Graphiques
    Scientific Programming with Python::Networks and Graphs: Réseaux et graphes
    Scientific Programming with Python::Other Scientific Libraries: Autres bibliothèques scientifiques
---

(about_py)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

```{index} single: python
```

# À propos de ces cours

```{epigraph}
« Python est devenu suffisamment redoutable pour que nous ne descendions plus jusqu'à R. Désolé, les gens de R. J'étais des vôtres autrefois, mais nous ne descendons plus jusqu'à R. » -- Chris Wiggins
```

## Vue d'ensemble

Cette série de cours vous apprendra à utiliser Python pour le calcul scientifique, en mettant l'accent sur l'économie et la finance.

La série s'adresse aux novices en Python, bien que les utilisateurs expérimentés y trouveront également du contenu utile dans les cours ultérieurs.

Dans ce cours, nous allons

* présenter Python,
* mettre en valeur certaines de ses capacités,
* expliquer pourquoi Python est notre langage préféré pour le calcul scientifique, et
* vous indiquer les étapes suivantes.

Vous n'avez **pas** besoin de tout comprendre dans ce cours -- nous détaillerons les points progressivement plus tard dans la série de cours.


### Ne puis-je pas simplement utiliser des LLM ?

Non !

Bien sûr, il est tentant de penser qu'à l'ère de l'IA, nous n'avons plus besoin d'apprendre à coder.

Et oui, nous aimons parfois être paresseux nous aussi.

De plus, nous convenons que les IA sont d'excellents outils de productivité pour les programmeurs.

Mais les IA ne peuvent pas résoudre de manière fiable de nouveaux problèmes qu'elles n'ont jamais rencontrés auparavant.

Vous devrez être l'architecte et le superviseur -- et pour ces tâches, vous devez être capable de lire, d'écrire et de comprendre du code informatique.

Cela dit, un bon LLM est un compagnon utile pour ces cours -- essayez de copier-coller du code de cette série et de demander une explication.


### MATLAB n'est-il pas meilleur ?

Non, non, et cent fois non.

Nirvana était formidable (et Soundgarden [était meilleur](https://www.youtube.com/watch?v=3mbBbFH9fAg&list=RD3mbBbFH9fAg)) mais il est temps de tourner la page des années 90.

Pour la plupart des problèmes modernes, les bibliothèques scientifiques de Python sont désormais bien en avance sur les capacités de MATLAB.

C'est particulièrement le cas dans des domaines en croissance rapide tels que l'apprentissage profond et l'apprentissage par renforcement.

De plus, tous les grands LLM sont plus compétents pour écrire du code Python que du code MATLAB.

Nous discuterons des mérites relatifs des bibliothèques de Python tout au long de cette série de cours, ainsi que dans notre série ultérieure sur [JAX](https://jax.quantecon.org/intro.html).



## Présentation de Python

[Python](https://www.python.org) est un langage de programmation à usage général conçu en 1989 par [Guido van Rossum](https://en.wikipedia.org/wiki/Guido_van_Rossum).

Python est gratuit et [open source](https://en.wikipedia.org/wiki/Open_source), avec un développement coordonné par la [Python Software Foundation](https://www.python.org/psf-landing/).

C'est important parce que cela

* nous fait économiser de l'argent,
* signifie que Python est contrôlé par la communauté des utilisateurs plutôt que par une entreprise à but lucratif, et
* encourage la reproductibilité et la [science ouverte](https://en.wikipedia.org/wiki/Open_science).


### Usages courants

{index}`Python <single: Python; common uses>` est un langage à usage général utilisé dans presque tous les domaines d'application, notamment

* l'IA et l'informatique
* d'autres formes de calcul scientifique
* la communication
* le développement web
* CGI et interfaces graphiques
* le développement de jeux
* la planification des ressources
* le multimédia
* etc.

Il est utilisé et largement soutenu par de grandes entreprises technologiques, notamment

* [Google](https://www.google.com/)
* [OpenAI](https://openai.com/)
* [Netflix](https://www.netflix.com/)
* [Meta](https://opensource.fb.com/)
* [Amazon](https://www.amazon.com/)
* [Reddit](https://www.reddit.com/)
* etc.


### Popularité relative

Python est l'un des -- si ce n'est le -- [langages de programmation les plus populaires](https://www.tiobe.com/tiobe-index/).

Des bibliothèques Python comme [pandas](https://pandas.pydata.org/) et [Polars](https://pola.rs/) remplacent des outils familiers comme Excel et VBA en tant que compétence essentielle dans les domaines de la finance et de la banque.

De plus, Python est extrêmement populaire au sein de la communauté scientifique -- en particulier celle liée à l'IA.

Par exemple, le graphique suivant issu de Stack Overflow Trends montre comment la popularité d'une seule bibliothèque Python d'apprentissage profond ([PyTorch](https://pytorch.org/)) a augmenté au cours des dernières années.


```{figure} /_static/lecture_specific/about_py/pytorch_vs_matlab.png
```
Pytorch n'est que l'une des nombreuses bibliothèques Python pour l'apprentissage profond et l'IA.



### Fonctionnalités

Python est un [langage de haut niveau](https://en.wikipedia.org/wiki/High-level_programming_language), ce qui signifie qu'il est relativement facile à lire, à écrire et à déboguer.

Il possède un noyau de langage relativement petit et facile à apprendre.

Ce noyau est soutenu par de nombreuses bibliothèques, que l'on peut étudier au besoin.

Python est flexible et pragmatique, prenant en charge plusieurs styles de programmation (procédural, orienté objet, fonctionnel, etc.).


### Syntaxe et conception

```{index} single: Python; syntax and design
```

L'une des raisons de la popularité de Python est sa conception simple et élégante.

Pour vous en donner une idée, examinons un exemple.

Le code ci-dessous est écrit en [Java](https://en.wikipedia.org/wiki/Java_(programming_language)) plutôt qu'en Python.

Vous n'avez **pas** besoin de lire et de comprendre ce code !


```{code-block} java

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class CSVReader {
    public static void main(String[] args) {
        String filePath = "data.csv"; 
        String line;
        String splitBy = ",";
        int columnIndex = 1; 
        double sum = 0;
        int count = 0;

        try (BufferedReader br = new BufferedReader(new FileReader(filePath))) {
            while ((line = br.readLine()) != null) {
                String[] values = line.split(splitBy);
                if (values.length > columnIndex) {
                    try {
                        double value = Double.parseDouble(
                            values[columnIndex]
                        );
                        sum += value;
                        count++;
                    } catch (NumberFormatException e) {
                        System.out.println(
                            "Skipping non-numeric value: " + 
                            values[columnIndex]
                        );
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        if (count > 0) {
            double average = sum / count;
            System.out.println(
                "Average of the second column: " + average
            );
        } else {
            System.out.println(
                "No valid numeric data found in the second column."
            );
        }
    }
}

```

Ce code Java ouvre un fichier imaginaire appelé `data.csv` et calcule la moyenne des valeurs de la deuxième colonne.

Voici un code Python qui fait la même chose.

Même si vous ne connaissez pas encore Python, vous pouvez voir que le code est bien plus simple et facile à lire.

```{code-cell} python3
:tags: [skip-execution]

import csv

total, count = 0, 0
with open('data.csv', mode='r') as file:
    reader = csv.reader(file)
    for row in reader:
        try:
            total += float(row[1])
            count += 1
        except (ValueError, IndexError):
            pass
print(f"Average: {total / count if count else 'No valid data'}")

```



### Le lien avec l'IA

L'IA est en train de prendre en charge de nombreuses tâches actuellement effectuées par des humains, tout comme d'autres formes de machines l'ont fait au cours des derniers siècles.

De plus, Python joue un rôle considérable dans les progrès de l'IA et de l'apprentissage automatique.

Cela signifie que les entreprises technologiques investissent massivement dans le développement de bibliothèques Python extrêmement puissantes.

Même si vous ne prévoyez pas de travailler sur l'IA et l'apprentissage automatique, vous pouvez tirer profit de l'apprentissage de certaines de ces bibliothèques pour vos propres projets en économie, en finance et dans d'autres domaines scientifiques.

Ces cours vous expliqueront comment.


## Programmation scientifique avec Python

```{index} single: scientific programming
```

Nous avons déjà discuté de l'importance de Python pour l'IA, l'apprentissage automatique et la science des données.

Python est également l'un des acteurs dominants dans

* l'astronomie
* la chimie
* la biologie computationnelle
* la météorologie
* le traitement du langage naturel
* etc.

L'utilisation de Python est également en hausse en économie, en finance et dans des domaines connexes comme la recherche opérationnelle -- qui étaient auparavant dominés par MATLAB / Excel / STATA / C / Fortran.

Cette section présente brièvement quelques exemples d'utilisation de Python pour la programmation scientifique générale.


### NumPy

```{index} single: scientific programming; numeric
```

L'une des parties les plus importantes du calcul scientifique consiste à travailler avec des données.

Les données sont souvent stockées dans des matrices, des vecteurs et des tableaux.

Nous pouvons créer un simple tableau de nombres en Python pur comme suit :

```{code-cell} python3
a = [-3.14, 0, 3.14]                    # Une liste Python
a
```

Ce tableau est très petit, il est donc acceptable de travailler avec Python pur.

Mais lorsque nous voulons travailler avec de plus grands tableaux dans de vrais programmes, nous avons besoin de plus d'efficacité et d'outils supplémentaires.

Pour cela, nous devons utiliser des bibliothèques pour travailler avec des tableaux.

Pour Python, la bibliothèque de traitement de matrices et de tableaux la plus importante est la bibliothèque [NumPy](https://numpy.org/).

Par exemple, construisons un tableau NumPy comportant 100 éléments

```{code-cell} python3
import numpy as np                     # Charger la bibliothèque

a = np.linspace(-np.pi, np.pi, 100)    # Créer une grille régulière de -π à π
a
```

Transformons maintenant ce tableau en lui appliquant des fonctions.

```{code-cell} python3
b = np.cos(a)                          # Appliquer le cosinus à chaque élément de a
c = np.sin(a)                          # Appliquer le sinus à chaque élément de a
```

Nous pouvons maintenant facilement calculer le produit scalaire de `b` et `c`.

```{code-cell} python3
b @ c
```

Nous pouvons également effectuer de nombreuses autres tâches, comme

* calculer la moyenne et la variance de tableaux
* construire des matrices et résoudre des systèmes linéaires
* générer des tableaux aléatoires pour la simulation, etc.

Nous en discuterons les détails plus tard dans la série de cours, où nous couvrons NumPy en profondeur.


### Alternatives à NumPy

Bien que NumPy soit toujours le roi du traitement de tableaux en Python, il existe désormais d'importants concurrents.

Des bibliothèques telles que [JAX](https://github.com/jax-ml/jax), [Pytorch](https://pytorch.org/) et [CuPy](https://cupy.dev/) disposent également de types de tableaux et d'opérations sur les tableaux intégrés qui peuvent être très rapides et efficaces.

En fait, ces bibliothèques sont meilleures pour exploiter la parallélisation et le matériel rapide, comme nous l'expliquerons plus tard dans cette série.

Cependant, vous devriez tout de même apprendre NumPy en premier parce que

* NumPy est plus simple et fournit une base solide, et
* des bibliothèques comme JAX étendent directement les fonctionnalités de NumPy et sont donc plus faciles à apprendre lorsque vous connaissez déjà NumPy.

Cette série de cours vous fournira un solide bagage sur NumPy.

### SciPy

La bibliothèque [SciPy](https://scipy.org/) est construite au-dessus de NumPy et offre des fonctionnalités supplémentaires.

(tuple_unpacking_example)=
Par exemple, calculons $\int_{-2}^2 \phi(z) dz$ où $\phi$ est la densité de la loi normale centrée réduite.

```{code-cell} python3
from scipy.stats import norm
from scipy.integrate import quad

ϕ = norm()
value, error = quad(ϕ.pdf, -2, 2)  # Intégrer à l'aide de la quadrature de Gauss
value
```

SciPy comprend de nombreuses routines standard utilisées en

* [algèbre linéaire](https://docs.scipy.org/doc/scipy/reference/linalg.html)
* [intégration](https://docs.scipy.org/doc/scipy/reference/integrate.html)
* [interpolation](https://docs.scipy.org/doc/scipy/reference/interpolate.html)
* [optimisation](https://docs.scipy.org/doc/scipy/reference/optimize.html)
* [distributions et techniques statistiques](https://docs.scipy.org/doc/scipy/reference/stats.html)
* [traitement du signal](https://docs.scipy.org/doc/scipy/reference/signal.html)

Vous pouvez toutes les voir [ici](https://docs.scipy.org/doc/scipy/reference/index.html).

Plus tard, nous discuterons de SciPy plus en détail.


### Graphiques

```{index} single: Matplotlib
```

L'un des grands atouts de Python est la visualisation de données.

La bibliothèque Python la plus populaire et la plus complète pour créer des figures et des graphiques est [Matplotlib](https://matplotlib.org/), dont les fonctionnalités comprennent

* des tracés, des histogrammes, des images de contours, des graphiques 3D, des diagrammes à barres, etc.
* une sortie dans de nombreux formats (PDF, PNG, EPS, etc.)
* l'intégration de LaTeX

Exemple de tracé 2D avec des annotations LaTeX intégrées

```{figure} /_static/lecture_specific/about_py/qs.png
:scale: 75
```

Exemple de tracé de contour

```{figure} /_static/lecture_specific/about_py/bn_density1.png
:scale: 70
```

Exemple de tracé 3D

```{figure} /_static/lecture_specific/about_py/career_vf.png
```

D'autres exemples se trouvent dans la [galerie de vignettes de Matplotlib](https://matplotlib.org/stable/gallery/index.html).

Parmi les autres bibliothèques graphiques figurent

* [Plotly](https://plotly.com/python/)
* [seaborn](https://seaborn.pydata.org/) --- une interface de haut niveau pour matplotlib
* [Altair](https://altair-viz.github.io/)
* [Bokeh](https://docs.bokeh.org/en/latest/)

Vous pouvez visiter la [Python Graph Gallery](https://python-graph-gallery.com/) pour plus d'exemples de tracés réalisés à l'aide de diverses bibliothèques.


### Réseaux et graphes

L'étude des [réseaux](https://networks.quantecon.org/) devient une partie importante du travail scientifique en économie, en finance et dans d'autres domaines.

Par exemple, nous nous intéressons à l'étude

* des réseaux de production
* des réseaux de banques et d'institutions financières
* des réseaux d'amitié et sociaux
* etc.

Python dispose de nombreuses bibliothèques pour étudier les réseaux et les graphes.

```{index} single: NetworkX
```

Un exemple bien connu est [NetworkX](https://networkx.org/).

Ses fonctionnalités comprennent, entre autres :

* des algorithmes de graphes standard pour analyser les réseaux
* des routines de tracé

Voici un exemple de code qui génère et trace un graphe aléatoire, avec la couleur des nœuds déterminée par la longueur du plus court chemin depuis un nœud central.

```{code-cell} ipython
import networkx as nx
import matplotlib.pyplot as plt
rng = np.random.default_rng(1234)

# Générer un graphe aléatoire
p = dict((i, (rng.uniform(0, 1), rng.uniform(0, 1)))
         for i in range(200))
g = nx.random_geometric_graph(200, 0.12, pos=p)
pos = nx.get_node_attributes(g, 'pos')

# Trouver le nœud le plus proche du point central (0.5, 0.5)
dists = [(x - 0.5)**2 + (y - 0.5)**2 for x, y in list(pos.values())]
ncenter = np.argmin(dists)

# Tracer le graphe, en coloriant selon la longueur du chemin depuis le nœud central
p = nx.single_source_shortest_path_length(g, ncenter)
plt.figure()
nx.draw_networkx_edges(g, pos, alpha=0.4)
nx.draw_networkx_nodes(g,
                       pos,
                       nodelist=list(p.keys()),
                       node_size=120, alpha=0.5,
                       node_color=list(p.values()),
                       cmap=plt.cm.jet_r)
plt.show()
```


### Autres bibliothèques scientifiques

Comme indiqué ci-dessus, il existe littéralement des milliers de bibliothèques scientifiques pour Python.

Certaines sont petites et effectuent des tâches très spécifiques.

D'autres sont énormes en termes de lignes de code et d'investissement de la part des programmeurs et des entreprises technologiques.

Voici une courte liste de quelques bibliothèques scientifiques importantes pour Python non mentionnées ci-dessus.

* [SymPy](https://www.sympy.org/) pour l'algèbre symbolique, y compris les limites, les dérivées et les intégrales
* [statsmodels](https://www.statsmodels.org/) pour les routines statistiques
* [scikit-learn](https://scikit-learn.org/) pour l'apprentissage automatique
* [Keras](https://keras.io/) pour l'apprentissage automatique
* [Pyro](https://pyro.ai/) et [PyStan](https://pystan.readthedocs.io/en/latest/) pour l'analyse bayésienne de données
* [GeoPandas](https://geopandas.org/en/stable/) pour l'analyse de données spatiales
* [Dask](https://docs.dask.org/en/stable/) pour la parallélisation
* [Numba](https://numba.pydata.org/) pour faire tourner Python à la même vitesse que le code machine natif
* [CVXPY](https://www.cvxpy.org/) pour l'optimisation convexe
* [scikit-image](https://scikit-image.org/) et [OpenCV](https://opencv.org/) pour le traitement et l'analyse de données d'images
* [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) pour extraire des données de fichiers HTML et XML


Dans cette série de cours, nous apprendrons à utiliser bon nombre de ces bibliothèques pour des tâches de calcul scientifique en économie et en finance.