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
  title: Dépannage
  headings:
    Fixing Your Local Environment: Corriger votre environnement local
    Reporting an Issue: Signaler un problème
---

(troubleshooting)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Dépannage

Cette page s'adresse aux lecteurs rencontrant des erreurs lors de l'exécution du code des cours.

## Corriger votre environnement local

L'hypothèse de base des cours est que le code d'un cours doit s'exécuter dès lors que

1. il est exécuté dans un notebook Jupyter et
1. le notebook est exécuté sur une machine disposant de la dernière version d'Anaconda Python.

Vous avez bien installé Anaconda, n'est-ce pas, en suivant les instructions de {doc}`ce cours <getting_started>` ?

En supposant que vous l'ayez fait, la source de problèmes la plus fréquente chez nos lecteurs est que leur distribution Anaconda n'est pas à jour.

[Voici un article utile](https://www.anaconda.com/blog/keeping-anaconda-date)
sur la manière de mettre à jour Anaconda.

Une autre option consiste simplement à désinstaller Anaconda et à le réinstaller.

Vous devez également maintenir à jour les bibliothèques de code externes, telles que [QuantEcon.py](https://quantecon.org/quantecon-py/).

Pour cette tâche, vous pouvez soit

* utiliser conda upgrade quantecon en ligne de commande, soit
* exécuter !conda upgrade quantecon depuis un notebook Jupyter.

Si votre environnement local ne fonctionne toujours pas, vous pouvez faire deux choses.

Premièrement, vous pouvez utiliser une machine distante à la place, en cliquant sur l'icône Launch Notebook disponible pour chaque cours

```{image} _static/lecture_specific/troubleshooting/launch.png

```

Deuxièmement, vous pouvez signaler un problème, afin que nous puissions essayer de corriger votre configuration locale.

Nous apprécions les retours sur les cours, alors n'hésitez pas à nous contacter.

## Signaler un problème

Une façon de nous faire part de vos commentaires est de soumettre un problème via notre [outil de suivi des problèmes](https://github.com/QuantEcon/lecture-python-programming/issues).

Soyez aussi précis que possible. Indiquez-nous où se situe le problème et donnez-nous autant de détails que possible sur votre configuration locale.

Enfin, vous pouvez nous adresser directement vos retours à [contact@quantecon.org](mailto:contact@quantecon.org)