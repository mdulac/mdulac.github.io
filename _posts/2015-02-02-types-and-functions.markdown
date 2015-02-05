---
layout: post
title:  'Types, valeurs et fonctions dans la programmation fonctionnelle'
date:   2014-02-05
categories: functionnal programming type types value values function functions
---

#Généralités
Les types et les fonctions devraient être connus de tous les développeurs,
car ils sont largement manipulés dans la quasi-totalité des programmes informatiques. À ce moment, je t'entends déjà répondre
_"je me fiche des types car j'utilise un langage faiblement typé, ou un langage typé dynamiquement !"._
À ce genre de remarque, il existe plusieurs réponses :

* Les types apportent une sémantique aux programmes. Ainsi, par exemple, il y a peu de sens à mettre autre chose qu'un booléen dans
un bloc conditionnel. Et qu'est ce qu'un booléen... ?
* Un développeur doit savoir adapter ses outils et ses solutions à un problème donné, et sera donc très certainement amené à manipuler
plus d'un langage, et pas forcément avec un système de types dynamiques.

_Dans ce billet, je ne parlerai pas des avantages d'un système de types statiques par rapport à un typage dynamique._

Bien que ces notions me paraissent fondamentales, je me rends compte qu'il existe des développeurs qui confondent certaines de ces
notions à partir du moment où ils manipulent un langage purement fonctionnel.

_Dans la suite de cet article, les exemples de code seront en Haskell_

Un type `A` est une donnée d'un programme informatique qui définie :

* un ensemble de valeurs de `A`
* un ensemble d'opérations applicables sur les valeurs de `A`

Un type est donc un **Set** de valeurs de `A`, sur lesquelles peuvent être appliquées des fonctions.

> Par exemple, le type `Boolean` est un Set de deux valeurs : `True` et `False`

#Un peu de pratique
Je te propose maintenant de réécrire le type Booléen, que l'on appellera `Boo`, l'ensemble des valeurs `T` (true) et `F` (false) :
{% highlight haskell %}
data Boo = T | F
{% endhighlight %}

et la fonction `if_then_else` qui prendra en arguments un Boo, et deux valeurs typées génériquement `a`, et :
{% highlight haskell %}
if_then_else :: Boo -> a -> a -> a
if_then_else T x _ = x
if_then_else F _ y = y
{% endhighlight %}

La première ligne est le prototype de la fonction, c'est à dire sa signature, et ne comporte donc que des types !
Les autres lignes sont l'implémentation de cette même fonction, en fonction de la **valeur** de ses arguments (pattern-matching).
Si la valeur du `Boo` passée à la fonction est `T`, on retourne x (de type `a`), sinon on retourne `y`(de type `a`).

Ainsi, on peut utiliser la fonction de cette manière :
{% highlight haskell %}
if_then_else T (2 + 4) (4 + 5) -- returns 6
if_then_else F (2 + 4) (4 + 5) -- returns 9
{% endhighlight %}

Mais qu'est-ce que ce type `a` dans la signature de fonction ? Il s'agit en fait d'un type générique, c'est à dire n'importe quel type !
{% highlight haskell %}
if_then_else T ("Hello") ("World") -- returns "Hello"
if_then_else F ("Hello") ("World") -- returns "World"
{% endhighlight %}

Evidemment, tous les types `a` doivent être le même type au moment de l'appel de la fonction. Ainsi ce code résultera en une erreur de
compilation :
{% highlight haskell %}
if_then_else T (2 + 4) ("World")
{% endhighlight %}

car les valeurs (2 + 3) et "World" ne sont pas du même type !

#Et les fonctions dans tout ça ?
_In progress_