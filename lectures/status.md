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
  title: Statistiques d'exécution
---

# Statistiques d'exécution

Ce tableau contient les statistiques d'exécution les plus récentes.

```{nb-exec-table}
```

(status:machine-details)=

Ces cours sont compilés sur des instances `linux` via `github actions`.

Ces cours utilisent la version de python suivante

```{code-cell} ipython
!python --version
```

et les versions de paquets suivantes

```{code-cell} ipython
:tags: [hide-output]
!conda list
```

Cette série de cours a accès au GPU suivant

```{code-cell} ipython
!nvidia-smi
```

Vous pouvez vérifier le backend utilisé par JAX avec :

```{code-cell} ipython3
import jax
# Vérifier si JAX utilise le GPU
print(f"JAX backend: {jax.devices()[0].platform}")
```