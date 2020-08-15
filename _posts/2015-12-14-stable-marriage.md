---
title: Hackathon/Stable Marriage Problem
date: 2015-12-14T18:23:33+00:00
layout: single
permalink: /2015/12/stable-marriage/
classes: wide
---
I recently volunteered at a [hackathon](https://en.wikipedia.org/wiki/Hackathon) at my workplace, where university students were invited to team up to [solve problems for charities and NGOs](https://github.com/CFGLondon), which also doubled as a thinly-veiled recruitment exercise (everybody wins!). I was on hand to answer technical questions that the participants had; it was an enjoyable couple days working with some bright, young, talented people. And of course, they had plenty of snacks to keep them going!

![hackathon-snacks](/assets/img/20151204_174134-1-768x563.jpg)

One thing I noticed was that many of the problems were variations on [the Stable Marriage problem](https://en.wikipedia.org/wiki/Stable_marriage_problem) &#8211; the problem of &#8220;finding a stable matching between two equally sized sets of elements given an ordering of preferences for each element&#8221;, which has many important applications, including matching users to servers in [CDNs](https://en.wikipedia.org/wiki/Content_delivery_network) (where proximity determines preference).

A solution to this problem is to use [the Gale-Shaply algorithm](https://en.wikipedia.org/wiki/Stable_marriage_problem#Solution), so I had a go at coding it up myself. As ever, [source code here!](https://github.com/ma489/stable-marriage)