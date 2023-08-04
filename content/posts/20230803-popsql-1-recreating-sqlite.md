---
title: "popSQL part 1: a DB education through reverse-engineering"
date: 2023-08-03T11:41:42-04:00
tags: ["sql", "python", "sqlite"]
categories: ["software"]
---

# A liking to database systems

Databases have long been a close interest of mine. Since taking CS 445 Information Systems at UMass, I've held a special interest in how databases were used and how they worked under the hood. I found things like relational algebra fascinating, and I always enjoyed how clean the SQL specification felt. I leafed through Michael Stonebraker's [Red Book](https://redbook.io) and developed a decent understanding about how to design, interact with and debug production databases.

For a long while this interest was merely a surface level curiosity, but at a certain point I wanted to dive below that surface - to really understand how SQL statments turned into data, and how the data systems I was using operated so efficiently. The resources on creating databases from scratch were slim (and still seem to be). Books, courses and academia make it easy to interact with existing databases, but not to design one on your own.

They all seem to miss the sweet spot of being approachable, practical, and sufficiently true to the principles of production systems that one can better appreciate and understand what's happening under the hood. I treat [Crafting Interpreters](https://craftinginterpreters.com/contents.html) my personal gold standard for what I look for in technical literature, and in databases there wasn't anything equivalent. Luckily, I happened upon the [SQLite documentation](https://www.sqlite.org/arch.html) - which in some ways had what I was looking for.

I decided to try to replicate the behavior of SQLite using the documentation, but ran headfirst into a roadblock. It seemed that while the documentation was thorough and quite easy to follow - it glossed over some of the details as to how SQLite actually worked under the hood. Questions like "how does the virtual machine interact with the b-tree?" and "how is a scan performed at a programmatic level?" can only be answered by reading the source code. For the benefit of the end users, and the detriment of a curious subscriber like myself the SQLite source is anything but simple to follow. It's tens of thousands of lines of code which has been tuned for optimization rather than readability. Take for example parsing variable length integers (varints), something described in the documentation with one paragraph. The [source code](https://github.com/sqlite/sqlite/blob/master/src/util.c#L1183) is nearly 300 lines of C code, with loops unrolled, bitmasks applied, variables stripped down so they represent nothing other than the address space which they occupy. If you compare this to the [15 lines of python code](https://github.com/angles-n-daemons/popsql/blob/master/pypopsql/util.py#L17) it takes to do this clearly and comprehsibly - you come to the same conclusion I did, that SQLite was designed to be a production system, and not an educational resource.

Being discouraged, I abandoned this journey in favor of simpler, lower hanging endeavours. Three years later however, my interest was once again piqued, and SQLite despite my above caveats still remained one of the closest things to an educational resource that I could look for.

With all this in mind, I'm trying once again to replicate a good bit of SQLite functionality in python by taking it slower, and really digging into the source code bit by bit. My goals in this endeavor are to create a good-enough sqlite replication which covers some of the core functionality employed by SQLite.

It should be noted that as this is an educational exercise, performance is an afterthought. You'll note this as my code runs lots of transformations to and from byte arrays, which SQLite ignores in the name of speed. I still haven't come to a conclusion as to how much of the source implementation I intend to replicate, whether I'll replicate the lemon parser generator, if I plan to replicate the bytecode engine. I figure I'll take it week by week and try to get the testable bits working and see where things go from there.

# Getting started

My approach this time will be to start from the absolute bottom of the system, the file format. I chose this as a starting place because it's easy to test in addition to being well documented. From there I'll design a close-enough-to the source set of abstractions which are unit tested fairly comprehsively.

```
$ sqlite test.db
> CREATE TABLE test(col1 VARCHAR(2), col2 INTEGER);
> INSERT INTO test(col1, col2) VALUES ('hi', 1), ('yo', 2);
```

# Reading the test table node

<visualization of byte format for nodes>

# Reading varints

<visualization of byte format for varint>

 - Mention documentation https://sqlite.org/forum/info/11fc7308d5e607f2

# Reading the header

# Reading records
