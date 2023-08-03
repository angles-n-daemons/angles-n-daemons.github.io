---
title: "popSQL part 1: learning through reverse-engineering"
date: 2023-08-03T11:41:42-04:00
tags: ["sql", "python", "sqlite"]
categories: ["software"]
---

# A liking to database systems

Databases have long been a close interest of mine. Since taking CS 445 Information Systems at UMass, I've held a special interest in how databases were used and how they worked under the hood. I found things like relational algebra fascinating, and I always enjoyed how clean the surface of the SQL specification felt. I leafed through Michael Stonebraker's [Red Book](https://redbook.io) and developed a decent understanding about how to design, interact with and debug production databases.

For a long while that interest was merely a surface level relationship, but at a certain point I wanted to dive below that surface - to really understand how SQL statments turned into data, and how the data systems I was using operated so efficiently. Unfortunately, the resources for building databases from scratch is relatively sparse.

Lots of educational resources on building databases focus on using a database system and building within it (a challenging thing to google around)

Missing the sweet spot - practical, approachable, true to the ideas

How I use crafting interpreters as the gold standard for technical resources

- Crafting interpreters for creating programming languages
- Build your own database (published now)

Talk about the SQLite documentation, talk about how I figured how easy it would be.

I spend a few months starting a go implementation of the SQLite internals and abandoned it, as just reading the documentation didn't iron out some of the more challenging questions. I also wanted to simplify my implementation but I realized that as soon as I diverged from the SQLite implementation that I would be in the dark as to how to design my solutions.

A few years later and I'm here, trying to start again using the sqlite documentation but this time starting from the data format. I'm going to try to match the sqlite specification the best I can, which will be difficult because SQLite is a monster (quote LOC). It also requires reading the source code which can be quite challenging as it's been designed in a way where comprehensibility has been sacrificed for performance (give example of parsing varints using sqlite link and my link). Give example on lemon parser generator, how many loc it is

What are my goals:
 - To provide a sql interface which is compatible with the sqlite file format
 - To make the codebase comprehensive to someone trying to read it

It should be noted that as this is an educational exercise, performance is an afterthought. You'll note this as my code runs lots of transformations to and from byte arrays, which SQLite ignores in the name of speed.

# Getting started

My approach this time will be to start from the absolute bottom of the database, the file format. I chose this as a starting place because it's testable and well documented. From there I'll design a close-enough-to the source set of abstractions which are unit tested fairly comprehsively.

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
