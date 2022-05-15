---
title: Listify in Haskell (Part I)
date: 2018-03-02T00:05:36+00:00
layout: single
permalink: /2018/03/rewriting-listify-in-haskell-part-i/
classes: wide
toc: true
---
I'm rewriting [Listify](https://www.mansourahmed.com/2016/06/spotify-playlister/) in [Haskell](https://www.haskell.org/), with a few changes. As a reminder, the original Listify (henceforth [Listify-js](https://github.com/ma489/listify-js)) is a web app that [scrapes](https://en.wikipedia.org/wiki/Web_scraping) the BBC radio website for tracklisting and program schedule information, and allows users to browse these tracklists and create Spotify playlists from them. The new version (henceforth [Listify-hs](https://github.com/ma489/listify-hs)) aims to build on this by introducing [VCR](https://en.wikipedia.org/wiki/Videocassette_recorder)-like functionality (or [TiVo Series Link](http://support.virginmedia.ie/app/answers/print/a_id/154) for the young'uns among you) of being able to 'follow' a show and have each episode's corresponding playlist delivered to your [Spotify](https://en.wikipedia.org/wiki/Spotify) account, as scheduled.

I'm rewriting it for a number of reasons:
1. The original version was written in [NodeJS](https://nodejs.org/en/)/[Express](https://expressjs.com/), and I&#8217;m finding that JavaScript code hard to maintain over time. 
2. I'm using it as a learning exercise through which to better understand certain Haskell concepts. 
3. There are parts of the application that I think are better suited to being implemented with Haskell (e.g. web scraping), and as an engineer one should always looks for better, simpler, more elegant solutions.

This is the first in a short series of blog posts that I&#8217;ll write as I build the system. This first post covers the scraping component, but keep an eye out for future posts, and also follow the implementation here on [GitHub](https://github.com/ma489/listify-hs).

## Overview

The purpose of this part of the system is to scrape playlist information from the BBC website (unfortunately there is no API available), and persist it to a database for later use. This is a batch process that will run daily. The components of interest are highlighted in red in the diagram below:

<img class="wp-image-526 size-large aligncenter" style="border: 1px solid #000000;" src="/assets/img/ls.png" alt="listify-hs" width="819" height="460"/>

## Project setup

I&#8217;m developing using the [Atom](https://atom.io/) text editor with the [IDE-Haskell](https://atom.io/packages/ide-haskell) plugin.

The project is built using [Stack](https://docs.haskellstack.org/en/stable/README/), with dependencies defined in a [Cabal file](https://github.com/ma489/listify-hs/blob/master/listify.cabal). Git commits are subject to CI builds [on Travis](https://travis-ci.org/ma489/listify-hs), which also runs [a small suite](https://github.com/ma489/listify-hs/blob/master/test/ScrapeSpec.hs) of [HSpec](https://hspec.github.io/) integration tests.

The project structure/layout was inspired by [this informative talk](https://jaspervdj.be/posts/2017-12-07-getting-things-done-in-haskell.html) on how to structure larger Haskell projects, given at [Haskell eXchange 2017](https://skillsmatter.com/conferences/8522-haskell-exchange-2017).

The main file of entry to follow the code for the scraping component is [here](https://github.com/ma489/listify-hs/blob/master/lib/Listify/Scraping/Main.hs).

## Scraping

The actual web scraping is done using [the Scalpel library](https://hackage.haskell.org/package/scalpel). Scalpel allows you to [declaratively](https://en.wikipedia.org/wiki/Declarative_programming) describe the scraping to be performed, resulting in [more elegant code](https://github.com/ma489/listify-hs/blob/master/lib/Scraping/Sources/BBC/Tracklist.hs) when compared to the [original Listify-js implementation](https://github.com/ma489/listify/blob/master/src/lib/radio-service.js).

The central type in the Scalpel library is [Scraper](https://hackage.haskell.org/package/scalpel-0.5.1/docs/Text-HTML-Scalpel.html#g:5). The _Scraper_&#8216;s job is to look certain HTML tags in a webpage that match some criteria, and return the contents therein. `Scraper` is made more useful by being an instance of (among others) two type classes: [Applicative](https://wiki.haskell.org/Typeclassopedia#Applicative) and [Alternative](https://wiki.haskell.org/Typeclassopedia#Failure_and_choice:_Alternative.2C_MonadPlus.2C_ArrowPlus).

### Alternative

[Alternative](https://wiki.haskell.org/Typeclassopedia#Failure_and_choice:_Alternative.2C_MonadPlus.2C_ArrowPlus) allows us to represent a choice of computation. We can use the `<|>` operator between two possible Scrapers that have different matching criteria, to have both criteria evaluated and return the combined result. This is useful when parsing HTML where there is some alternative or missing tags. For example, [from here](https://github.com/ma489/listify-hs/blob/master/lib/Scraping/Sources/BBC/Tracklist.hs#L30) (simplified below), we may not always have an album tag, so we should check both ways (with and without):

{% highlight haskell %}
playlistItemScraper :: Scraper ... Track 
playlistItemScraper = hasAlbumScraper <|> noAlbumScraper
{% endhighlight %}

### Applicative

[Applicative](https://wiki.haskell.org/Typeclassopedia#Applicative) allows us to use multi-argument functions in a more general way, i.e. by applying them to values in a context. First, some background: the `Applicative` type class has less structure than [Monad](https://wiki.haskell.org/Monad), but more than [Functor](https://wiki.haskell.org/Functor). The [fmap](https://hackage.haskell.org/package/base-4.10.1.0/docs/Data-Functor.html#v:fmap) function in Functor allows us to apply a unary (single-argument) function to a value in a Functor context (by [lifting](https://wiki.haskell.org/Lifting) that function into a Functor). For example, below we have the single-argument function `(+3)` - a [section](https://wiki.haskell.org/Section_of_an_infix_operator) - and a value in the [Maybe](https://hackage.haskell.org/package/base-4.10.1.0/docs/Data-Maybe.html) Functor, `Just 3`:

{% highlight haskell %}
example :: Maybe Int
example = fmap (1+) (Just 3)  -- results in 'Just 4'
{% endhighlight %}

However, its more useful to be able to lift _multiple-argument_ functions into a context. Monad supports this with the various [liftMn](https://hackage.haskell.org/package/base-4.10.1.0/docs/Control-Monad.html#v:liftM2) functions (where n is the number of arguments the function takes). In the example below, we lift the binary (two-argument) function `(+)`, using the `liftM2` function, to the Maybe Monad:

{% highlight haskell %}
import Control.Monad

example :: Maybe Int
example =  liftM2 (+) (Just 1) (Just 3)  -- results in 'Just 4'
{% endhighlight %}

However, `Monad` brings excess functionality than what is required to solve just this problem of function application. Thats where Applicative comes in. By using the [<$>](https://hackage.haskell.org/package/base-4.10.1.0/docs/Control-Applicative.html#v:-60--36--62-) and [<*>](https://hackage.haskell.org/package/base-4.10.1.0/docs/Control-Applicative.html#v:-60--42--62-) operators from Applicative, we can lift multi-argument functions to an Applicative and then apply it to values in that Applicative context, respectively. Example below with the Maybe Applicative:

{% highlight haskell %}
example :: Maybe Int
example = (+) <$> (Just 1) <*> (Just 3)  -- results in 'Just 4'
{% endhighlight %}

The scraping code makes use of this. For example we can write an expression like the one below (taken from [here](https://github.com/ma489/listify-hs/blob/master/lib/Scraping/Sources/BBC/Tracklist.hs#L33)), which in my opinion is quite elegant (type signature simplified):

{% highlight haskell %}
playlistItemScraper' :: Scraper ... Track 
playlistItemScraper' = Track <$> artistScraper <*> titleScraper <*> albumScraper
{% endhighlight %}

`Track` here is a multi-argument [data constructor](https://wiki.haskell.org/Constructor#Data_constructors_as_first_class_values) (i.e. a function) whose type is: `Text -> Text -> Text -> Track.` The values `artistScraper`, `titleScraper`, `albumScraper` are of type: `Scraper ... Text` (think of this as a `Text` value in the `Scraper` Applicative context). The `Track` function gets lifted to the `Scraper` Applicative and applied to the `artistScraper`, `titleScraper`, `albumScraper` values.

For comparison, the Applicative expression above is the [desugared](https://en.wiktionary.org/wiki/desugar#Verb_2) version of the less tidy [do-notation](https://wiki.haskell.org/Do_notation_considered_harmful) equivalent:

{% highlight haskell %}
playlistItemScraper' :: Scraper Text Track
playlistItemScraper' = do
  artist <- artistScraper
  titl <- titleScraper
  album <- albumScraper
  return $ Track titl artist album
{% endhighlight %}

## Persistence

The results of the scraping are persisted to a MySQL database, for later use by other components of the Listify application (coming soon). Interaction with the database is done via the [mysql-simple](https://hackage.haskell.org/package/mysql-simple) library, created by the guy who [literally wrote the book](http://book.realworldhaskell.org/) on how to use Haskell in the real world.

The type representing a SQL query is [Query](https://hackage.haskell.org/package/mysql-simple-0.4.4/docs/Database-MySQL-Simple.html#g:2). My only (minor) frustration is that there is only really one way to create a Query (from a literal [ByteString](https://hackage.haskell.org/package/bytestring-0.10.8.2/docs/Data-ByteString.html)); so one cannot, for example, read the query in from a separately maintained .sql file and use that.

That's it for Part I - look at for Part II coming soon! And as ever, if you spot anything incorrect, feedback is welcome.
