---
title: Higher-rank Polymorphism
date: 2016-07-31T13:10:38+00:00
layout: single
permalink: /2016/07/higher-rank-polymorphism/
classes: wide
---
On the [functional programming course I attended](http://www.mansourahmed.com/2016/07/afp-haskell-utrecht/), I learnt about some interesting parts of Haskell&#8217;s type system. I figured I&#8217;d write a blog post*.

(Standard) Haskell has what&#8217;s called [let-bound polymorphism](https://www.haskell.org/tutorial/pitfalls.html). This means that identifiers bound in a `let`, a `where` or top-level definitions can be polymorphic, whereas lambda-bound identifiers (function arguments) cannot. Here are some examples, using the (polymorphic) [_id_](http://hackage.haskell.org/package/base-4.9.0.0/docs/Prelude.html#v:id) function:

{% highlight haskell %}
-- Let (OK)
polymorphicLet :: (Integer, Char)
polymorphicLet = let f = id in (f 3, f 'a')

-- Where (OK)
polymorphicWhere :: (Integer, Char) 
polymorphicWhere = (f 3, f 'a') where f = id
 
-- Top-level (OK)
f :: a -> a 
f = id 
polymorphicTopLevel :: (Integer, Char) 
polymorphicTopLevel = (f 3, f 'a')
 
-- Lambda-bound (not OK)
polymorphicInlineLambda :: (Integer, Char) 
polymorphicInlineLambda = (\f -> (f 3, f 'a')) id
{% endhighlight %}

This last example does not work: it is not possible to determine the type of `f`, and no explicit type signature has been provided**. How do we fix this? We need to give the compiler more type information!

How do we describe the type of `f`? Well, it&#8217;s a polymorphic function of type `a -> a`. So the type of the inline lambda abstraction above is something like (`∀a.a -> a) -> (Integer, Char)`. This is an example of [higher-rank polymorphism](https://en.wikibooks.org/wiki/Haskell/Polymorphism#Higher_rank_types) &#8211; this is where a polymorphic type appears in another type; in this case the function argument itself is polymorphic (a so-called _rank-2_ _type_).

How do we express that in Haskell? Well, we need two [GHC extensions](https://ocharles.org.uk/blog/posts/2014-12-01-24-days-of-ghc-extensions.html); namely [ScopedTypeVariables](https://wiki.haskell.org/Scoped_type_variables) and [RankNTypes](https://wiki.haskell.org/Rank-N_types). `RankNTypes` allows us to express &#8216;for all&#8217; (and so describe higher-rank types), and ScopeTypeVariables allows us to extend the scope of type variables into the definition of a function.

{% highlight haskell %}
{-# LANGUAGE RankNTypes #-}
{-# LANGUAGE ScopedTypeVariables #-}
 
polymorphicInlineLambda' :: (Integer, Char)
polymorphicInlineLambda' = (\(f :: forall a. a -> a) -> (f 3, f 'a')) id
{% endhighlight %}

_et voila! _

[See here](https://github.com/ma489/afp/blob/master/src/AdvancedTypeSystems.hs) for the full example source code, and [here](https://github.com/ma489/afp/tree/master/src) for more Haskell.

*Oh, and if you&#8217;re a Haskeller and have noticed some mistake in this post, let me know, feedback welcome!

**I think GHC assumes `f` to be monomorphic because of the [monomorphism restriction](https://wiki.haskell.org/Monomorphism_restriction)