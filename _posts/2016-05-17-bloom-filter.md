---
title: Bloom filter
date: 2016-05-17T15:11:38+00:00
layout: single
permalink: /2016/05/bloom-filter/
classes: wide
---
A cool data structure that I&#8217;ve come across in the past, but have not yet had a chance to use directly is a [Bloom filter](https://en.wikipedia.org/wiki/Bloom_filter). It&#8217;s a probabilistic data structure that can act as a set, and is both space and time efficient (constant!). Bloom filters have found many useful applications, typically in preventing the occurrence of some more expensive lookup or access.

Naturally, I&#8217;ve had a go at implementing it myself. You can find the [source code here](https://github.com/ma489/python-sandbox/blob/master/datastructures/bloom_filter.py) (Python).

<img class="aligncenter" src="https://upload.wikimedia.org/wikipedia/commons/a/ac/Bloom_filter.svg" width="649" height="233" />

The general idea: to insert an element, run it through a number of hash functions, each generating an index into a bit array, and set the bits at those positions in the array. Then, to query for an element (i.e. is a given element a member of this set?), run it through those same hash functions and check whether all the bits at those positions in the array are set. Note: Bloom filters come with two key caveats: (i) when querying, _you may get false_ _positives_, and (ii) _items cannot be removed_ (which would introduce false negatives).

A wonderful explanation of Bloom Filters [can be found here](https://www.jasondavies.com/bloomfilter/) and a neat trick to generate several hash functions [can be found here](http://willwhim.wpengine.com/2011/09/03/producing-n-hash-functions-by-hashing-only-once/).

&nbsp;