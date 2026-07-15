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
  title: "POO II\_: Construire des classes"
  headings:
    Overview: Vue d'ensemble
    OOP Review: Révision de la POO
    OOP Review::Key Concepts: Concepts clés
    OOP Review::Why is OOP Useful?: "Pourquoi la POO est-elle utile\_?"
    Defining Your Own Classes: Définir vos propres classes
    'Defining Your Own Classes::Example: A Consumer Class': "Exemple\_: une classe Consommateur"
    'Defining Your Own Classes::Example: A Consumer Class::Usage': Utilisation
    'Defining Your Own Classes::Example: A Consumer Class::Self': Self
    'Defining Your Own Classes::Example: A Consumer Class::Details': Détails
    'Defining Your Own Classes::Example: The Solow Growth Model': "Exemple\_: le modèle de croissance de Solow"
    'Defining Your Own Classes::Example: A Market': "Exemple\_: un marché"
    'Defining Your Own Classes::Example: Chaos': "Exemple\_: le chaos"
    Special Methods: Méthodes spéciales
    Exercises: Exercices
---

(python_oop)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# {index}`POO II : Construire des classes <single: POO II: Building Classes>`

```{index} single: Python; Programmation orientée objet
```

## Vue d'ensemble

Dans un {doc}`cours précédent <oop_intro>`, nous avons appris quelques fondements de la programmation orientée objet.

Les objectifs de ce cours sont les suivants

* aborder la POO plus en profondeur
* apprendre à construire nos propres objets, spécialisés selon nos besoins

Par exemple, vous savez déjà comment

* créer des listes, des chaînes de caractères et d'autres objets Python
* utiliser leurs méthodes pour modifier leur contenu

Imaginez maintenant que vous vouliez écrire un programme avec des consommateurs, qui peuvent

* détenir et dépenser de l'argent
* consommer des biens
* travailler et gagner de l'argent

Une solution naturelle en Python consisterait à créer des consommateurs sous forme d'objets ayant

* des données, telles que l'argent disponible
* des méthodes, telles que `buy` ou `work` qui agissent sur ces données

Python rend cela facile, en vous fournissant des **définitions de classes**.

Les classes sont des plans qui vous aident à construire des objets selon vos propres spécifications.

Il faut un peu de temps pour s'habituer à la syntaxe, c'est pourquoi nous fournirons de nombreux exemples.

Nous utiliserons les importations suivantes :

```{code-cell} ipython
import numpy as np
import matplotlib.pyplot as plt
```

## Révision de la POO

La POO est prise en charge dans de nombreux langages :

* JAVA et Ruby sont des langages relativement purement orientés objet.
* Python prend en charge à la fois la programmation procédurale et orientée objet.
* Fortran et MATLAB sont principalement procéduraux, avec quelques ajouts récents de POO.
* C est un langage procédural, tandis que C++ est du C avec la POO ajoutée par-dessus.

Abordons les concepts généraux de la POO avant de nous spécialiser sur Python.

### Concepts clés

```{index} single: Programmation orientée objet; Concepts clés
```

Comme nous l'avons vu dans un {doc}`cours précédent <oop_intro>`, dans le paradigme de la POO, les données et les fonctions sont **regroupées** en « objets ».

Un exemple est une liste Python, qui non seulement stocke des données mais sait aussi comment se trier elle-même, etc.

```{code-cell} python3
x = [1, 5, 4]
x.sort()
x
```

Comme nous le savons maintenant, `sort` est une fonction qui « fait partie » de l'objet liste — et donc appelée une *méthode*.

Si nous voulons créer nos propres types d'objets, nous devons utiliser des définitions de classes.

Une *définition de classe* est un plan pour une classe particulière d'objets (par exemple, des listes, des chaînes de caractères ou des nombres complexes).

Elle décrit

* Quel type de données la classe stocke
* Quelles méthodes elle possède pour agir sur ces données

Un *objet* ou *instance* est une réalisation de la classe, créée à partir du plan

* Chaque instance possède ses propres données uniques.
* Les méthodes définies dans la définition de classe agissent sur ces données (et d'autres).

En Python, les données et les méthodes d'un objet sont collectivement appelées *attributs*.

Les attributs sont accessibles via la « notation d'attribut par point »

* `object_name.data`
* `object_name.method_name()`

Dans l'exemple

```{code-cell} python3
x = [1, 5, 4]
x.sort()
x.__class__
```

* `x` est un objet ou une instance, créé à partir de la définition des listes Python, mais avec ses propres données particulières.
* `x.sort()` et `x.__class__` sont deux attributs de `x`.
* `dir(x)` peut être utilisé pour afficher tous les attributs de `x`.

(why_oop)=
### Pourquoi la POO est-elle utile ?

La POO est utile pour la même raison que l'abstraction est utile : pour reconnaître et exploiter la structure commune.

Par exemple,

* *une chaîne de Markov* consiste en un ensemble d'états, une distribution de probabilité initiale sur les états, et une collection de probabilités de passage d'un état à un autre
* *une théorie de l'équilibre général* consiste en un espace de biens, des préférences, des technologies, et une définition d'équilibre
* *un jeu* consiste en une liste de joueurs, des listes d'actions disponibles pour chaque joueur, les gains de chaque joueur en fonction des actions de tous les autres joueurs, et un protocole de timing

Ce sont toutes des abstractions qui regroupent des « objets » du même « type ».

Reconnaître une structure commune nous permet d'employer des outils communs.

En théorie économique, il peut s'agir d'une proposition qui s'applique à tous les jeux d'un certain type.

En Python, il peut s'agir d'une méthode utile pour toutes les chaînes de Markov (par exemple, `simulate`).

Lorsque nous utilisons la POO, la méthode `simulate` est commodément regroupée avec l'objet chaîne de Markov.

## Définir vos propres classes

```{index} single: Programmation orientée objet; Classes
```

Construisons quelques classes simples pour commencer.

(oop_consumer_class)=
Avant de le faire, afin d'illustrer une partie de la puissance des classes, nous allons définir deux fonctions que nous appellerons `earn` et `spend`.

```{code-cell} python3
def earn(w,y):
    "Consumer with inital wealth w earns y"
    return w+y

def spend(w,x):
    "consumer with initial wealth w spends x"
    new_wealth = w -x
    if new_wealth < 0:
        print("Insufficient funds")
    else:
        return new_wealth
```

La fonction `earn` prend la richesse initiale d'un consommateur $w$ et y ajoute ses gains actuels $y$.

La fonction `spend` prend la richesse initiale d'un consommateur $w$ et en déduit ses dépenses actuelles $x$.

Nous pouvons utiliser ces deux fonctions pour suivre la richesse d'un consommateur au fur et à mesure qu'il gagne et dépense.

Par exemple

```{code-cell} python3
w0=100
w1=earn(w0,10)
w2=spend(w1,20)
w3=earn(w2,10)
w4=spend(w3,20)
print("w0,w1,w2,w3,w4 = ", w0,w1,w2,w3,w4)
```

Une *classe* regroupe un ensemble de données liées à une *instance* particulière avec une collection de fonctions qui opèrent sur ces données.

Dans notre exemple, une *instance* sera le nom d'une *personne* particulière dont les *données d'instance* consistent uniquement en sa richesse.

(Dans d'autres exemples, les *données d'instance* consisteront en un vecteur de données.)

Dans notre exemple, deux fonctions `earn` et `spend` peuvent être appliquées aux données d'instance actuelles.

Prises ensemble, les données d'instance et les fonctions sont appelées *attributs*.

Ceux-ci sont facilement accessibles de la manière que nous allons décrire maintenant.

### Exemple : une classe Consommateur

Nous allons construire une classe `Consumer` avec

* un attribut `wealth` qui stocke la richesse du consommateur (données)
* une méthode `earn`, où `earn(y)` augmente la richesse du consommateur de `y`
* une méthode `spend`, où `spend(x)` soit diminue la richesse de `x`, soit renvoie une erreur si les fonds sont insuffisants

Certes un peu artificiel, cet exemple de classe nous aide à intérioriser une syntaxe particulière.

Voici comment nous mettons en place notre classe Consommateur.

```{code-cell} python3
class Consumer:

    def __init__(self, w):
        "Initialize consumer with w dollars of wealth"
        self.wealth = w

    def earn(self, y):
        "The consumer earns y dollars"
        self.wealth += y

    def spend(self, x):
        "The consumer spends x dollars if feasible"
        new_wealth = self.wealth - x
        if new_wealth < 0:
            print("Insufficent funds")
        else:
            self.wealth = new_wealth
```

Il y a une syntaxe spéciale ici, alors examinons-la attentivement

* Le mot-clé `class` indique que nous construisons une classe.

La classe `Consumer` définit les données d'instance `wealth` et trois méthodes : `__init__`, `earn` et `spend`

* `wealth` est une *donnée d'instance* car chaque consommateur que nous créons (chaque instance de la classe `Consumer`) aura ses propres données de richesse.

Les méthodes `earn` et `spend` déploient les fonctions que nous avons décrites précédemment et qui peuvent potentiellement être appliquées aux données d'instance `wealth`.

La méthode `__init__` est une *méthode constructeur*.

Chaque fois que nous créons une instance de la classe, la méthode `__init_` sera appelée automatiquement.

L'appel de `__init__` met en place un « espace de noms » pour contenir les données d'instance — nous y reviendrons bientôt.

Nous discuterons également en détail ci-dessous du rôle du particulier dispositif de gestion `self`.

#### Utilisation

Voici un exemple dans lequel nous utilisons la classe `Consumer` pour créer une instance d'un consommateur que nous nommons affectueusement $c1$.

Après avoir créé le consommateur $c1$ et l'avoir doté d'une richesse initiale de $10$, nous appliquerons la méthode `spend`.

```{code-cell} python3
c1 = Consumer(10)  # Create instance with initial wealth 10
c1.spend(5)
c1.wealth
```

```{code-cell} python3
c1.earn(15)
c1.spend(100)
```

Nous pouvons bien sûr créer plusieurs instances, c'est-à-dire plusieurs consommateurs, chacun avec son propre nom et ses propres données

```{code-cell} python3
c1 = Consumer(10)
c2 = Consumer(12)
c2.spend(4)
c2.wealth
```

```{code-cell} python3
c1.wealth
```

Chaque instance, c'est-à-dire chaque consommateur, stocke ses données dans un dictionnaire d'espace de noms distinct

```{code-cell} python3
c1.__dict__
```

```{code-cell} python3
c2.__dict__
```

Lorsque nous accédons aux attributs ou les définissons, nous modifions en réalité simplement le dictionnaire maintenu par l'instance.

#### Self

Si vous regardez à nouveau la définition de la classe `Consumer`, vous verrez le mot self tout au long du code.

Les règles d'utilisation de `self` lors de la création d'une classe sont les suivantes

* Toute donnée d'instance doit être préfixée par `self`
    * par exemple, la méthode `earn` utilise `self.wealth` plutôt que simplement `wealth`
* Une méthode définie dans le code qui définit la classe doit avoir `self` comme premier argument
    * par exemple, `def earn(self, y)` plutôt que simplement `def earn(y)`
* Toute méthode référencée dans la classe doit être appelée comme `self.method_name`

Il n'y a pas d'exemples de la dernière règle dans le code précédent, mais nous en verrons quelques-uns sous peu.

#### Détails

Dans cette section, nous examinons quelques détails plus formels liés aux classes et à `self`

* Vous pourriez souhaiter passer directement à {ref}`la section suivante <oop_solow_growth>` lors de votre première lecture de ce cours.
* Vous pourrez revenir à ces détails après vous être familiarisé avec davantage d'exemples.

Les méthodes vivent en réalité à l'intérieur d'un objet de classe formé lorsque l'interpréteur lit la définition de la classe

```{code-cell} python3
print(Consumer.__dict__)  # Show __dict__ attribute of class object
```

Notez comment les trois méthodes `__init__`, `earn` et `spend` sont stockées dans l'objet de classe.

Considérons le code suivant

```{code-cell} python3
c1 = Consumer(10)
c1.earn(10)
c1.wealth
```

Lorsque vous appelez `earn` via `c1.earn(10)`, l'interpréteur passe l'instance `c1` et l'argument `10` à `Consumer.earn`.

En fait, les deux suivants sont équivalents

* `c1.earn(10)`
* `Consumer.earn(c1, 10)`

Dans l'appel de fonction `Consumer.earn(c1, 10)`, notez que `c1` est le premier argument.

Rappelez-vous que dans la définition de la méthode `earn`, `self` est le premier paramètre

```{code-cell} python3
def earn(self, y):
     "The consumer earns y dollars"
     self.wealth += y
```

Le résultat final est que `self` est lié à l'instance `c1` à l'intérieur de l'appel de fonction.

C'est pourquoi l'instruction `self.wealth += y` à l'intérieur de `earn` finit par modifier `c1.wealth`.

(oop_solow_growth)=
### Exemple : le modèle de croissance de Solow

```{index} single: Programmation orientée objet; Méthodes
```

Pour notre prochain exemple, écrivons une classe simple pour implémenter le modèle de croissance de Solow.

Le modèle de croissance de Solow est un modèle de croissance néoclassique dans lequel le stock de capital par habitant $k_t$ évolue selon la règle

```{math}
:label: solow_lom

k_{t+1} = \frac{s z k_t^{\alpha} + (1 - \delta) k_t}{1 + n}
```

Ici

* $s$ est un taux d'épargne donné de manière exogène
* $z$ est un paramètre de productivité
* $\alpha$ est la part du capital dans le revenu
* $n$ est le taux de croissance de la population
* $\delta$ est le taux de dépréciation

Un **état stationnaire** du modèle est un $k$ qui résout {eq}`solow_lom` lorsque $k_{t+1} = k_t = k$.

Voici une classe qui implémente ce modèle.

Quelques points intéressants dans le code sont

* Une instance conserve un enregistrement de son stock de capital actuel dans la variable `self.k`.
* La méthode `h` implémente le membre de droite de {eq}`solow_lom`.
* La méthode `update` utilise `h` pour mettre à jour le capital conformément à {eq}`solow_lom`.
    * Remarquez comment, à l'intérieur de `update`, la référence à la méthode locale `h` est `self.h`.

Les méthodes `steady_state` et `generate_sequence` sont assez explicites

```{code-cell} python3
class Solow:
    r"""
    Implements the Solow growth model with the update rule

        k_{t+1} = [(s z k^α_t) + (1 - δ)k_t] /(1 + n)

    """
    def __init__(self, n=0.05,  # population growth rate
                       s=0.25,  # savings rate
                       δ=0.1,   # depreciation rate
                       α=0.3,   # share of labor
                       z=2.0,   # productivity
                       k=1.0):  # current capital stock

        self.n, self.s, self.δ, self.α, self.z = n, s, δ, α, z
        self.k = k

    def h(self):
        "Evaluate the h function"
        # Unpack parameters (get rid of self to simplify notation)
        n, s, δ, α, z = self.n, self.s, self.δ, self.α, self.z
        # Apply the update rule
        return (s * z * self.k**α + (1 - δ) * self.k) / (1 + n)

    def update(self):
        "Update the current state (i.e., the capital stock)."
        self.k =  self.h()

    def steady_state(self):
        "Compute the steady state value of capital."
        # Unpack parameters (get rid of self to simplify notation)
        n, s, δ, α, z = self.n, self.s, self.δ, self.α, self.z
        # Compute and return steady state
        return ((s * z) / (n + δ))**(1 / (1 - α))

    def generate_sequence(self, t):
        "Generate and return a time series of length t"
        path = []
        for i in range(t):
            path.append(self.k)
            self.update()
        return path
```

Voici un petit programme qui utilise la classe pour calculer des séries temporelles à partir de deux conditions initiales différentes.

L'état stationnaire commun est également tracé à des fins de comparaison

```{code-cell} ipython
s1 = Solow()
s2 = Solow(k=8.0)

T = 60
fig, ax = plt.subplots(figsize=(9, 6))

# Trace la valeur commune de l'état stationnaire du capital
ax.plot([s1.steady_state()]*T, 'k-', label='état stationnaire')

# Trace les séries temporelles pour chaque économie
for s in s1, s2:
    lb = f'série du capital à partir de l\'état initial {s.k}'
    ax.plot(s.generate_sequence(T), 'o-', lw=2, alpha=0.6, label=lb)

ax.set_xlabel('$t$', fontsize=14)
ax.set_ylabel('$k_t$', fontsize=14)
ax.legend()
plt.show()
```

### Exemple : un marché

Ensuite, écrivons une classe pour un marché concurrentiel dans lequel les acheteurs et les vendeurs sont tous deux preneurs de prix.

Le marché consiste en les objets suivants :

* Une courbe de demande linéaire $Q = a_d - b_d p$
* Une courbe d'offre linéaire $Q = a_z + b_z (p - t)$

Ici

* $p$ est le prix payé par l'acheteur, $Q$ est la quantité et $t$ est une taxe unitaire.
* Les autres symboles sont des paramètres de demande et d'offre.

La classe fournit des méthodes pour calculer diverses valeurs d'intérêt, y compris le prix et la quantité d'équilibre concurrentiel, les recettes fiscales collectées, le surplus du consommateur et le surplus du producteur.

Voici notre implémentation.

(Elle utilise une fonction de SciPy appelée quad pour l'intégration numérique — un sujet dont nous parlerons davantage plus tard.)

```{code-cell} python3
from scipy.integrate import quad

class Market:

    def __init__(self, ad, bd, az, bz, tax):
        """
        Set up market parameters.  All parameters are scalars.  See
        https://lectures.quantecon.org/py/python_oop.html for interpretation.

        """
        self.ad, self.bd, self.az, self.bz, self.tax = ad, bd, az, bz, tax
        if ad < az:
            raise ValueError('Insufficient demand.')

    def price(self):
        "Compute equilibrium price"
        return  (self.ad - self.az + self.bz * self.tax) / (self.bd + self.bz)

    def quantity(self):
        "Compute equilibrium quantity"
        return  self.ad - self.bd * self.price()

    def consumer_surp(self):
        "Compute consumer surplus"
        # == Compute area under inverse demand function == #
        integrand = lambda x: (self.ad / self.bd) - (1 / self.bd) * x
        area, error = quad(integrand, 0, self.quantity())
        return area - self.price() * self.quantity()

    def producer_surp(self):
        "Compute producer surplus"
        #  == Compute area above inverse supply curve, excluding tax == #
        integrand = lambda x: -(self.az / self.bz) + (1 / self.bz) * x
        area, error = quad(integrand, 0, self.quantity())
        return (self.price() - self.tax) * self.quantity() - area

    def taxrev(self):
        "Compute tax revenue"
        return self.tax * self.quantity()

    def inverse_demand(self, x):
        "Compute inverse demand"
        return self.ad / self.bd - (1 / self.bd)* x

    def inverse_supply(self, x):
        "Compute inverse supply curve"
        return -(self.az / self.bz) + (1 / self.bz) * x + self.tax

    def inverse_supply_no_tax(self, x):
        "Compute inverse supply curve without tax"
        return -(self.az / self.bz) + (1 / self.bz) * x
```

Voici un exemple d'utilisation

```{code-cell} python3
baseline_params = 15, .5, -2, .5, 3
m = Market(*baseline_params)
print("equilibrium price = ", m.price())
```

```{code-cell} python3
print("consumer surplus = ", m.consumer_surp())
```

Voici un court programme qui utilise cette classe pour tracer une courbe de demande inverse avec des courbes d'offre inverse avec et sans taxes

```{code-cell} python3
# Baseline ad, bd, az, bz, tax
baseline_params = 15, .5, -2, .5, 3
m = Market(*baseline_params)

q_max = m.quantity() * 2
q_grid = np.linspace(0.0, q_max, 100)
pd = m.inverse_demand(q_grid)
ps = m.inverse_supply(q_grid)
psno = m.inverse_supply_no_tax(q_grid)

fig, ax = plt.subplots()
ax.plot(q_grid, pd, lw=2, alpha=0.6, label='demande')
ax.plot(q_grid, ps, lw=2, alpha=0.6, label='offre')
ax.plot(q_grid, psno, '--k', lw=2, alpha=0.6, label='offre sans taxe')
ax.set_xlabel('quantité', fontsize=14)
ax.set_xlim(0, q_max)
ax.set_ylabel('prix', fontsize=14)
ax.legend(loc='lower right', frameon=False, fontsize=14)
plt.show()
```

Le programme suivant fournit une fonction qui

* prend une instance de `Market` comme paramètre
* calcule la perte sèche résultant de l'imposition de la taxe

```{code-cell} python3
def deadw(m):
    "Computes deadweight loss for market m."
    # == Create analogous market with no tax == #
    m_no_tax = Market(m.ad, m.bd, m.az, m.bz, 0)
    # == Compare surplus, return difference == #
    surp1 = m_no_tax.consumer_surp() + m_no_tax.producer_surp()
    surp2 = m.consumer_surp() + m.producer_surp() + m.taxrev()
    return surp1 - surp2
```

Voici un exemple d'utilisation

```{code-cell} python3
baseline_params = 15, .5, -2, .5, 3
m = Market(*baseline_params)
deadw(m)  # Show deadweight loss
```

### Exemple : le chaos

Examinons un exemple de plus, lié à la dynamique chaotique dans les systèmes non linéaires.

Une règle de transition simple qui peut générer des trajectoires temporelles erratiques est la carte logistique

```{math}
:label: quadmap2

x_{t+1} = r x_t(1 - x_t) ,
\quad x_0 \in [0, 1],
\quad r \in [0, 4]
```

Écrivons une classe pour générer des séries temporelles à partir de ce modèle.

Voici une implémentation

```{code-cell} python3
class Chaos:
  """
  Models the dynamical system :math:`x_{t+1} = r x_t (1 - x_t)`
  """
  def __init__(self, x0, r):
      """
      Initialize with state x0 and parameter r
      """
      self.x, self.r = x0, r

  def update(self):
      "Apply the map to update state."
      self.x =  self.r * self.x *(1 - self.x)

  def generate_sequence(self, n):
      "Generate and return a sequence of length n."
      path = []
      for i in range(n):
          path.append(self.x)
          self.update()
      return path
```

Voici un exemple d'utilisation

```{code-cell} python3
ch = Chaos(0.1, 4.0)     # x0 = 0.1 and r = 0.4
ch.generate_sequence(5)  # First 5 iterates
```

Ce morceau de code trace une trajectoire plus longue

```{code-cell} python3
ch = Chaos(0.1, 4.0)
ts_length = 250

fig, ax = plt.subplots()
ax.set_xlabel('$t$', fontsize=14)
ax.set_ylabel('$x_t$', fontsize=14)
x = ch.generate_sequence(ts_length)
ax.plot(range(ts_length), x, 'bo-', alpha=0.5, lw=2, label='$x_t$')
plt.show()
```

Le morceau de code suivant fournit un diagramme de bifurcation

```{code-cell} python3
fig, ax = plt.subplots()
ch = Chaos(0.1, 4)
r = 2.5
while r < 4:
    ch.r = r
    t = ch.generate_sequence(1000)[950:]
    ax.plot([r] * len(t), t, 'b.', ms=0.6)
    r = r + 0.005

ax.set_xlabel('$r$', fontsize=16)
ax.set_ylabel('$x_t$', fontsize=16)
plt.show()
```

Sur l'axe horizontal se trouve le paramètre $r$ dans {eq}`quadmap2`.

L'axe vertical est l'espace d'états $[0, 1]$.

Pour chaque $r$, nous calculons une longue série temporelle puis traçons la queue (les 50 derniers points).

La queue de la séquence nous montre où la trajectoire se concentre après s'être stabilisée dans une sorte d'état stationnaire, si un état stationnaire existe.

Le fait qu'elle se stabilise, et le caractère de l'état stationnaire vers lequel elle se stabilise, dépendent de la valeur de $r$.

Pour $r$ entre environ 2,5 et 3, la série temporelle se stabilise en un point fixe unique tracé sur l'axe vertical.

Pour $r$ entre environ 3 et 3,45, la série temporelle se stabilise en oscillant entre les deux valeurs tracées sur l'axe vertical.

Pour $r$ un peu plus élevé que 3,45, la série temporelle se stabilise en oscillant entre les quatre valeurs tracées sur l'axe vertical.

Remarquez qu'il n'y a aucune valeur de $r$ qui conduit à un état stationnaire oscillant entre trois valeurs.

## Méthodes spéciales

```{index} single: Programmation orientée objet; Méthodes spéciales
```

Python fournit des méthodes spéciales qui s'avèrent pratiques.

Par exemple, rappelez-vous que les listes et les tuples ont une notion de longueur et que cette longueur peut être interrogée via la fonction `len`

```{code-cell} python3
x = (10, 20)
len(x)
```

Si vous souhaitez fournir une valeur de retour pour la fonction `len` lorsqu'elle est appliquée à votre objet défini par l'utilisateur, utilisez la méthode spéciale `__len__`

```{code-cell} python3
class Foo:

    def __len__(self):
        return 42
```

Maintenant nous obtenons

```{code-cell} python3
f = Foo()
len(f)
```

(call_method)=
Une méthode spéciale que nous utiliserons régulièrement est la méthode `__call__`.

Cette méthode peut être utilisée pour rendre vos instances appelables, tout comme des fonctions

```{code-cell} python3
class Foo:

    def __call__(self, x):
        return x + 42
```

Après l'exécution nous obtenons

```{code-cell} python3
f = Foo()
f(8)  # Exactly equivalent to f.__call__(8)
```

L'exercice 1 fournit un exemple plus utile.

## Exercices

```{exercise-start}
:label: oop_ex1
```

La [fonction de répartition empirique (ecdf)](https://en.wikipedia.org/wiki/Empirical_distribution_function) correspondant à un échantillon $\{X_i\}_{i=1}^n$ est définie comme

```{math}
:label: emdist

F_n(x) := \frac{1}{n}  \sum_{i=1}^n \mathbf{1}\{X_i \leq x\}
  \qquad (x \in \mathbb{R})
```

Ici $\mathbf{1}\{X_i \leq x\}$ est une fonction indicatrice (un si $X_i \leq x$ et zéro sinon) et donc $F_n(x)$ est la fraction de l'échantillon qui tombe en dessous de $x$.

Le théorème de Glivenko-Cantelli stipule que, à condition que l'échantillon soit i.i.d., l'ecdf $F_n$ converge vers la véritable fonction de répartition $F$.

Implémentez $F_n$ sous forme d'une classe appelée `ECDF`, où

* Un échantillon donné $\{X_i\}_{i=1}^n$ sont les données d'instance, stockées sous `self.observations`.
* La classe implémente une méthode `__call__` qui renvoie $F_n(x)$ pour tout $x$.

Votre code devrait fonctionner comme suit (au hasard près)

```{code-block} python3
:class: no-execute

from random import uniform

samples = [uniform(0, 1) for i in range(10)]
F = ECDF(samples)
F(0.5)  # Evaluate ecdf at x = 0.5
```

```{code-block} python3
:class: no-execute

F.observations = [uniform(0, 1) for i in range(1000)]
F(0.5)
```

Visez la clarté, pas l'efficacité.

```{exercise-end}
```

```{solution-start} oop_ex1
:class: dropdown
```

```{code-cell} python3
class ECDF:

    def __init__(self, observations):
        self.observations = observations

    def __call__(self, x):
        counter = 0.0
        for obs in self.observations:
            if obs <= x:
                counter += 1
        return counter / len(self.observations)
```

```{code-cell} python3
# == test == #

from random import uniform

samples = [uniform(0, 1) for i in range(10)]
F = ECDF(samples)

print(F(0.5))  # Evaluate ecdf at x = 0.5

F.observations = [uniform(0, 1) for i in range(1000)]

print(F(0.5))
```

```{solution-end}
```


```{exercise-start}
:label: oop_ex2
```

Dans un {ref}`exercice précédent <pyess_ex2>`, vous avez écrit une fonction pour évaluer des polynômes.

Cet exercice est une extension, où la tâche consiste à construire une classe simple appelée `Polynomial` pour représenter et manipuler des fonctions polynomiales telles que

```{math}
:label: polynom

p(x) = a_0 + a_1 x + a_2 x^2 + \cdots a_N x^N = \sum_{n=0}^N a_n x^n
    \qquad (x \in \mathbb{R})
```

Les données d'instance de la classe `Polynomial` seront les coefficients (dans le cas de {eq}`polynom`, les nombres $a_0, \ldots, a_N$).

Fournissez des méthodes qui

1. Évaluent le polynôme {eq}`polynom`, renvoyant $p(x)$ pour tout $x$.
1. Différencient le polynôme, en remplaçant les coefficients d'origine par ceux de sa dérivée $p'$.

Évitez d'utiliser toute instruction `import`.

```{exercise-end}
```

```{solution-start} oop_ex2
:class: dropdown
```

```{code-cell} python3
class Polynomial:

    def __init__(self, coefficients):
        """
        Creates an instance of the Polynomial class representing

            p(x) = a_0 x^0 + ... + a_N x^N,

        where a_i = coefficients[i].
        """
        self.coefficients = coefficients

    def __call__(self, x):
        "Evaluate the polynomial at x."
        y = 0
        for i, a in enumerate(self.coefficients):
            y += a * x**i
        return y

    def differentiate(self):
        "Reset self.coefficients to those of p' instead of p."
        new_coefficients = []
        for i, a in enumerate(self.coefficients):
            new_coefficients.append(i * a)
        # Remove the first element, which is zero
        del new_coefficients[0]
        # And reset coefficients data to new values
        self.coefficients = new_coefficients
        return new_coefficients
```

```{solution-end}
```