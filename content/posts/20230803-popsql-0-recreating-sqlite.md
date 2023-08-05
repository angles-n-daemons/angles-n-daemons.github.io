---
title: "popSQL part 0: a DB education through reverse-engineering"
date: 2023-08-03T11:41:42-04:00
tags: ["sql", "python", "sqlite"]
categories: ["software"]
---

# A liking for database systems

Databases have long been a special interest of mine. Since taking CS 445 Information Systems at UMass, I've held a unique curiosity for how databases were used and how they worked under the hood. I found topics like relational algebra fascinating, and I always enjoyed how clean SQL felt. I distinctly remember how enjoyable it was leafing through Michael Stonebraker's [Red Book](https://redbook.io) and over time I developed a decent understanding about how to design, interact with and debug production databases.

For a long while this interest was merely a surface level curiosity, but at a certain point I wanted to dive below that surface - to really understand how SQL statments turned into data, and how the data systems I was using operated so efficiently. The resources on creating databases from scratch were slim (and still seem to be). Books, courses and academia make it easy to interact with existing databases, but not to design one on your own.

They all seem to miss the sweet spot of being approachable, practical, and true enough to the principles of production systems that one could better understand what's happening under the hood. I treat [Crafting Interpreters](https://craftinginterpreters.com/contents.html) my personal gold standard for what I look for in technical literature of this nature, and in databases there wasn't anything equivalent. Luckily, I happened upon the [SQLite documentation](https://www.sqlite.org/arch.html) - which in some ways had these properties.

I decided to try to replicate the behavior of SQLite using the documentation, but quickly ran headfirst into a roadblock. While the documentation was thorough, it was laid out in a way that very detailed questions about how the system works could be answered quickly, and not in a way that someone could easily learn top to bottom what is actually happening. For the benefit of the end users, and the detriment of a curious subscriber like myself the SQLite source is a bit too complex to easily follow. It's tens of thousands of lines of code which has been tuned for optimization rather than readability. Take for example parsing variable length integers (varints), something described in the documentation with one paragraph. The [source code](https://github.com/sqlite/sqlite/blob/master/src/util.c#L1183) is nearly 300 lines of C code, with loops unrolled, bitmasks applied, variables stripped down so they represent nothing other than the address space which they occupy. If you compare this to the [15 lines of python code](https://github.com/angles-n-daemons/popsql/blob/master/pypopsql/util.py#L17) it takes to do this in a slower, but more digestable format - you come to the reasonable conclusion that SQLite was designed to be a production system and not an educational resource.

Being discouraged, I abandoned this journey in favor of simpler, lower hanging endeavours. Three years later however, my interest was once again piqued, and SQLite despite my above caveats still remained one of the closest things to an educational resource that I could look for.

With all this in mind, I'm trying once again to replicate a good bit of SQLite functionality in python by taking it slower, and really digging into the source code bit by bit. My goals in this endeavor are to create a good-enough sqlite replication which covers some of the core functionality employed by SQLite.

It should be noted that as this is an educational exercise, performance is an afterthought. You'll note this as my code runs lots of transformations to and from byte arrays, which SQLite ignores in the name of speed. I still haven't come to a conclusion as to how much of the source implementation I intend to replicate, whether I'll replicate the lemon parser generator, if I plan to replicate the bytecode engine. I figure I'll take it week by week and try to get the testable bits working and see where things go from there.

Feel free to read the following blog posts to see how this journey goes, and take a look at the [source repository](https://github.com/angles-n-daemons/pypopsql) to see what the result of this adventure looks like.
