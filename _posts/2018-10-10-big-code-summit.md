---
title: Big Code Summit
date: 2018-10-10T19:02:00+00:00
layout: single
permalink: /2018/10/big-code-summit/
classes: wide
---
I've just returned from the inaugural 'Big Code Summit'. A two-day conference on the very interesting topic of *Big Code*, which featured a mix of academic and industry speakers. [See here](https://bigcodesummit.splashthat.com/) for a link to the full programme. 

By analogy to Big Data, *Big Code* is the research area on how to manage large amounts of code in an organization and how best to deal with the complexities that come with that, as well as how to treat your code as data and leverage that large corpus to gain insights into how engineers can work better.

Below are summary notes on some of the talks that I found to be personal highlights:

- [Neural code search](https://dl.acm.org/citation.cfm?id=3211353)
  - Using NLP and IR techniques to search through code
- [Learning to find Bugs](http://software-lab.org/publications/oopsla2018_DeepBugs.pdf)
  - Using ML to detect bugs in code. This is done by formalizing bug-finding as a classification problem: is this code buggy or non-buggy? In contrast, static analysis bug detectors fall short as they are coded to look for specific patterns.
  - [DeepBugs](https://github.com/michaelpradel/DeepBugs) is trained on a labeled code corpus (given snippet of code, label is buggy/non-buggy), and so can learn more general rules. Fortunately, these labels come naturally from source version histories: if a given commit has a bug, then there will be a dual, follow-up commit that fixes the bug. It can even suggest fixes to the bug, based on information from source version history (e.g. commit x had a bug, commit y fixes the bug, that delta is the bug-fix strategy).
- [Learning for Programs: Connecting code, statistics, semantics, and language](https://homepages.inf.ed.ac.uk/csutton/talks/learning4programs/)
  - A broad-ranging talk, that touched on solving some interesting problems such as generating code comments from some given source snippet, or detecting redundant/inaccurate comments by comparison to their adjacent source code.
- [Generate program UIs from design layouts](https://pavol-bielik.github.io/data/slides/BigCodeWorkshop'18.pdf)
  - Given an image from a UI designer, use ML to generate code that creates that UI. This is to save developers time/effort in producing UIs, and reduce inconsistencies from eye-balling.
- [Neural sketch learning](https://arxiv.org/pdf/1703.05698.pdf)
  - All about generating source code given some conditions (syntactic/semantic constraints) that the desired target source code must satisfy. Check out ([Bayou](http://askbayou.com/)) ([more info](https://info.askbayou.com/how-to-use-bayou/)); which can be used to generate Java code that follows API-calling idioms.