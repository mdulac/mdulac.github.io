---
layout: post
title:  'Play with Maybe "monads !"'
date:   2014-02-01
categories: functionnal programming monads
---

Si tu as déjà touché à la programmation fonctionnelle, tu as probablement entendu parler de la notion de `monade`. Il existe pléthore d'articles sur Internet couvrant les monades, mais beaucoup d'entre eux sont teintés de notions mathématiques, plus précisément de théorie des catégories.

Pour définir la notion de monade en deux mots : `contexte` et `composition`.

_Les exemples de code qui suivent sont écrits en Haskell, un langage que j'affectionne particulièrement pour sa syntaxe concise, mais très expressive._

Imagine une fonction qui retourne, pour tout entier positif, le double de sa valeur. Comment représenter le fait que cette fonction puisse ne pas retourner de valeur utile, **dans le cas où le paramètre est négatif** ?
Il s'agit là d'un contexte **d'absence probable** de valeur utile, _et il y a une monade pour ça !_

# Maybe `a`
Haskell propose un type `Maybe a` pour représenter ce contexte, où `a` est un type paramétré. Ce type propose deux valeurs, `Just a` et `Nothing`. Ainsi, le type `Maybe Int` contient les valeurs `Just Int` et `Nothing`, le type `Maybe String` contient les valeurs `Just String` et `Nothing`, _etc_

{% highlight haskell %}
Just "hey you !" :: Maybe String
Just 3 :: Maybe Int
Nothing :: Maybe a
{% endhighlight %}

Ok, maintenant que nous avons découvert ce contexte, on va l'utiliser pour implémenter notre fonction, qui retournera dans un contexte `Just` le double de la valeur, si elle est positive, sinon `Nothing` :

{% highlight haskell %}
doublePositive :: Int -> Maybe Int
doublePositive x
	| x > 0 = Just (2 * x)
	| otherwise = Nothing
{% endhighlight %}

{% highlight haskell %}
doublePositive 2 -- returns Just 4
doublePositive 6 -- returns Just 12
doublePositive -1 -- returns Nothing
{% endhighlight %}

Il est d'usage d'utiliser la fonction `return` pour placer une valeur dans son contexte, _le contexte est inféré grace au prototype de la fonction_ :
{% highlight haskell %}
doublePositive :: Int -> Maybe Int
doublePositive x
	| x > 0 = return (2 * x)
	| otherwise = Nothing
{% endhighlight %}

C'est bien d'avoir une valeur contextualisée, mais encore faut il pouvoir l'exploiter.

# Usage et composition
Comment faire maintenant pour, par exemple, réappliquer la fonction sur la valeur retournée ?
{% highlight haskell %}
let r = doublePositive 4 in doublePositive r -- Ne compile pas !
{% endhighlight %}

Le code ci-dessus ne compile pas car il est impossible d'appliquer la fonction `doublePositive` à `Just Int` _(`Maybe Int` n'est pas un `Int`)_.
`Maybe a` propose la fonction `>>=` qui permet de réaliser une composition de valeurs contextualisées. De cette manière, les fonctions `a -> M a` et `a -> M b` peuvent être composées en une fonction `a -> M b`.
Le prototype de la fonction `>>=` est `M a -> (a -> M b) -> M b`

{% highlight haskell %}
doublePositive 3 -- cette fonction retourne un Maybe Int
doublePositive -- cette fonction est du type Int -> Maybe Int
doublePositive 3 >>= doublePositive -- returns Just 12
{% endhighlight %}

Tu viens d'appliquer la fonction `doublePositive` à `Just Int` ! Tu peux évidemment appliquer toute fonction de type Int -> Maybe Int !

{% highlight haskell %}
doublePositive 1 >>= doublePositive >>= doublePositive -- returns Just 8
doublePositive -4 >>= doublePositive >>= doublePositive -- returns Nothing
doublePositive 3 >>= ( \x -> return (x - 2) ) -- returns Just 4
doublePositive 3 >>= ( \x -> return (x - 10) ) >>= doublePositive -- returns Nothing
doublePositive 4 >>= ( \x -> return (show x) ) -- returns Just "8"
doublePositive 4 >>= ( \x -> return ("Hello" ++ show x) ) -- returns Just "hello 8"
{% endhighlight %}

Le prototype de la fonction `>>=` étant `M a -> (a -> M b) -> M b`, tu peux évidemment transformer ton Int en String, ou en tout autre type !

# Et la monade dans tout ça ?
Tu viens de voir que `Maybe a` est une monade _parmi d'autres_. Les deux fonctions `return` et `>>=` que l'on vient d'utiliser sont disponibles pour toutes les monades. Haskell dispose d'un `typeclass` `Monad` qui t'oblige à définir ces fonctions si tu instancies `Monad` (_Maybe est une instance de Monad, donc implémente ces fonctions_).

{% highlight haskell %}
class Monad Maybe where
	return = Just
	m >>= f = case m of
		Just x = f x
		_ = Nothing
{% endhighlight %}

Tu dois savoir que pour être une monade, un type doit respecter trois conditions :

- `return x >>= f` est équivalent à `f x`
- `m >>= return` est équivalent à `m`
- `(m >>= f) >>= g` est équivalent à `m >>= (\x -> f x >>= g)`

`Maybe` couvre ces trois conditions :
{% highlight haskell %}
-- Rule 1
let f = ( \x -> return (x + 5) )
return 5 >>= f -- returns Just 10
f 5 -- returns Just 10

-- Rule 2
Just 5 >>= return -- returns Just 5

-- Rule 3
let f = ( \x -> return (x + 5) )
let g = ( \x -> return (x + 10) )
(Just 5 >> f) >> g -- returns Just 20
Just 5 >>= ( \x -> f >>= g ) -- returns Just 20
{% endhighlight %}

Si tu veux en savoir plus sur Maybe, je te conseille de visiter le site [Hoogle][hoogle], et plus particulièrement la page sur le type [Maybe][hoogle-maybe].

[hoogle]: http://www.haskell.org/hoogle/
[hoogle-maybe]: http://hackage.haskell.org/package/base-4.6.0.1/docs/Prelude.html#t:Maybe



