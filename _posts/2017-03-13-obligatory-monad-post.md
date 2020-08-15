---
title: Obligatory Monad Post
date: 2017-03-13T22:46:10+00:00
layout: single
permalink: /2017/03/obligatory-monad-post/
classes: wide
---
<p style="text-align: left;">
  I&#8217;m taking a break from blogging for a while, but before I go, here&#8217;s a (<a href="https://byorgey.wordpress.com/2009/01/12/abstraction-intuition-and-the-monad-tutorial-fallacy/">now cliché</a>) blog post on <a href="https://en.wikipedia.org/wiki/Monad_(functional_programming)">monads</a>.
</p>

<p style="text-align: left;">
  There are far more precise, formal descriptions out there; however, I&#8217;ll give a practical, if imprecise, description of monads that I believe (!) makes them easier to think about:
</p>

> <p style="text-align: left;">
>   A monad is a datatype<sup>1</sup> that allows us to: represent computations<sup>2</sup> that have some structured<sup>3</sup> result, and chain such computations together<sup>4</sup>
> </p>

1. The datatype defines two operations, which (in Haskell parlance) are called <em>bind</em> (>>=) and <em>return</em>. <em>return</em> places a value in the monad, and <em>bind</em> is used to pass this monadic value to another function. Implementations of these operations must satisfy <a href="https://wiki.haskell.org/Monad_laws">certain laws</a>.
2. Some expression to be evaluated.
3. The result has some additional context. For example, that context could be that the computation may fail (Haskell&#8217;s <a href="https://en.wikibooks.org/wiki/Haskell/Understanding_monads/Maybe">Maybe</a> monad), or that it is non-deterministic (<a href="https://en.wikibooks.org/wiki/Haskell/Understanding_monads/List">List</a>), or that some interaction with the outside-world occurs (<a href="https://en.wikibooks.org/wiki/Haskell/Understanding_monads/IO">IO</a>).
4. Sequential composition. We can combine monadic values by passing them between functions that operate on them (using the bind operator)

[Here&#8217;s an example](https://github.com/ma489/haskell-sandbox/blob/master/examples/MaybeMonadExample.hs) in Haskell:

{% highlight haskell %}
-- Placing a value inside the context of the Maybe monad, using 'return'
x :: Maybe Int 
x = return 3
 
-- Let's define a function to operate on x
-- It takes on a non-monadic value and produce another monadic value
f :: Int -> Maybe String 
f a = return $ show a 
 
-- Lets apply function f to value 'inside' x using the bind operator
applyFtoX :: Maybe String 
applyFtoX = x >>= f
{% endhighlight %}

As a bonus, [here&#8217;s an example in Scala,](https://github.com/ma489/scala-sandbox/blob/master/src/main/scala/example/WriterExample.scala) inspired by some code I wrote recently at work that made use of the Writer monad from the [Scalaz](https://github.com/scalaz/scalaz) library. <span style="color: #000000;">The <a href="http://learnyouahaskell.com/for-a-few-monads-more#writer">Writer</a> monad is useful for computations that accumulate some additional output alongside the result (e.g. logging).</span>

Anyways, I hope this has been useful. If there is anything incorrect, don&#8217;t hesitate to mail me 🙂