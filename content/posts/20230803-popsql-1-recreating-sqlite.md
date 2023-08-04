---
title: "popSQL part 1: a DB education through reverse-engineering"
date: 2023-08-03T11:41:42-04:00
tags: ["sql", "python", "sqlite"]
categories: ["software"]
---

# A liking to database systems

Databases have long been a close interest of mine. Since taking CS 445 Information Systems at UMass, I've held a special interest in how databases were used and how they worked under the hood. I found things like relational algebra fascinating, and I always enjoyed how clean working with SQL felt. I leafed through Michael Stonebraker's [Red Book](https://redbook.io) and developed a decent understanding about how to design, interact with and debug production databases.

For a long while this interest was merely a surface level curiosity, but at a certain point I wanted to dive below that surface - to really understand how SQL statments turned into data, and how the data systems I was using operated so efficiently. The resources on creating databases from scratch were slim (and still seem to be). Books, courses and academia make it easy to interact with existing databases, but not to design one on your own.

They all seem to miss the sweet spot of being approachable, practical, and sufficiently true to the principles of production systems that one can better appreciate and understand what's happening under the hood. I treat [Crafting Interpreters](https://craftinginterpreters.com/contents.html) my personal gold standard for what I look for in technical literature, and in databases there wasn't anything equivalent. Luckily, I happened upon the [SQLite documentation](https://www.sqlite.org/arch.html) - which in some ways had what I was looking for.

I decided to try to replicate the behavior of SQLite using the documentation, but ran headfirst into a roadblock. It seemed that while the documentation was thorough and quite easy to follow - it glossed over some of the details as to how SQLite actually worked under the hood. Questions like "how does the virtual machine interact with the b-tree?" and "how is a scan performed at a programmatic level?" can only be answered by reading the source code. For the benefit of the end users, and the detriment of a curious subscriber like myself the SQLite source is anything but simple to follow. It's tens of thousands of lines of code which has been tuned for optimization rather than readability. Take for example parsing variable length integers (varints), something described in the documentation with one paragraph. The [source code](https://github.com/sqlite/sqlite/blob/master/src/util.c#L1183) is nearly 300 lines of C code, with loops unrolled, bitmasks applied, variables stripped down so they represent nothing other than the address space which they occupy. If you compare this to the [15 lines of python code](https://github.com/angles-n-daemons/popsql/blob/master/pypopsql/util.py#L17) it takes to do this clearly and comprehsibly - you come to the same conclusion I did, that SQLite was designed to be a production system, and not an educational resource.

Being discouraged, I abandoned this journey in favor of simpler, lower hanging endeavours. Three years later however, my interest was once again piqued, and SQLite despite my above caveats still remained one of the closest things to an educational resource that I could look for.

With all this in mind, I'm trying once again to replicate a good bit of SQLite functionality in python by taking it slower, and really digging into the source code bit by bit. My goals in this endeavor are to create a good-enough sqlite replication which covers some of the core functionality employed by SQLite.

It should be noted that as this is an educational exercise, performance is an afterthought. You'll note this as my code runs lots of transformations to and from byte arrays, which SQLite ignores in the name of speed. I still haven't come to a conclusion as to how much of the source implementation I intend to replicate, whether I'll replicate the lemon parser generator, if I plan to replicate the bytecode engine. I figure I'll take it week by week and try to get the testable bits working and see where things go from there.

# Getting started

My approach this time will be to start from the absolute bottom of the system, the file format. I chose this as a starting place because it's easy to test in addition to being well documented. From there I'll design a close-enough-to the source set of abstractions which are unit tested fairly comprehsively.

In approaching the build out this time, I figured the easiest thing to test against the functionality with SQLite was the file format, mainly because it would be simple to read and compare what SQLite actually writes. It's simple enough to create a database with a single table and then see what the file looks like on a byte, by byte level.

```
$ sqlite test.db
> CREATE TABLE test(col1 VARCHAR(2), col2 INTEGER);
> INSERT INTO test(col1, col2) VALUES ('hi', 1), ('yo', 2);
```

Once the database has been created you can also see how large it is. One question I had that didn't seem immediately answered by the documentation was would a database with almost no data still fill up entire pages.

```
➜  budlightlime git:(bdillmann/popsql-1) ✗ ls -lh ~/projects/popsql/pypopsql/test.db
-rw-r--r--  1 godzilla  staff   8.0K Aug  2 19:12 /Users/godzilla/projects/popsql/pypopsql/test.db
```

It seems like the answer is yes. Creating a single user table with 2 rows yields an 8k file, which consulting the [file format](https://www.sqlite.org/fileformat2.html) leads me to believe that the page contents are as follows:

Page 1. Header (.dbinfo) and Schema Table Leaf Node
Page 2. Table Leaf Node for the test table which I just created.

I assume that the first page is going to be more complicated to parse than the second page, and a lot of the process for parsing the second page can be reused for the first page.

From here, I figure the minumum testable codebase will require:
1. A pager for reading and writing to the database file.
2. An abstraction which takes database pages and reads the Node header from them.

### The Pager

The reason we use a pager is because reading and writing to disk often means doing so in fixed-size blocks, regardless of the size data passed. If the fixed-size is 512 bytes, reading 4 bytes will still at a mechanical level read 512 bytes from disk, writing 513 bytes will still write 1024 bytes because of how computers work. By adding a pager, which reads and writes in these fixed sizes - we optimize the IO bandwidth of our application.

See the below description on minimum addressible units and sectors wrt reading and writing from disk by ChatGPT.

> The minimum addressable unit on a disk refers to the smallest unit of data that can be read from or written to the disk. It is the basic building block of data storage and represents the smallest chunk of data that the disk controller or operating system can access and manipulate.
> 
> In most modern computer systems, the minimum addressable unit on a disk is known as a "sector." A sector is a fixed-size portion of the disk's surface, and it is typically 512 bytes or 4 kilobytes (KB) in size. The sector size is determined by the disk's hardware and firmware and is an inherent property of the disk.
> 
> When data is read from or written to the disk, it must be done in multiples of the sector size. For example, if you want to store 1 KB of data on a disk with 512-byte sectors, the data will still occupy one whole sector, leaving some unused space. Similarly, if you want to store 4.5 KB of data, it will occupy nine sectors (4 KB) with some unused space within the last sector.
> 
> The concept of the minimum addressable unit on the disk is essential because it impacts how data is organized and stored on the storage medium. It also affects disk I/O operations and can influence the efficiency and performance of data access.
> 
> It's worth noting that advancements in storage technology, like Solid State Drives (SSDs) and newer disk formats, have introduced changes in the concept of the minimum addressable unit and the sector size. For instance, some modern SSDs have larger physical page sizes (e.g., 4KB or 8KB) for internal data management, even though the standard sector size presented to the operating system might still be 512 bytes or 4KB.

With this in mind, a simple enough pager would look like the following:

```
class Pager:
    def __init__(
        self,
        file_name: str,
        page_size: int = 4096,
    ):
        self.file_name = file_name
        self.page_size = page_size

    def get_page(
        self,
        page_number: int,
    ) -> bytes:
        with open(self.file_name, 'rb+') as file:
            file.seek(self.get_offset(page_number))
            return file.read(self.page_size)

    def get_offset(
        self,
        page_number: int,
    ):
        return (page_number - 1) * self.page_size

```

Something interesting to note is that page numbers are 1-indexed in the sqlite implementation, so I created a function for calculating the offset of the file so that calculating the offset is straightforward for any function requiring it in the future.

As we continue to expand our implementation, there will also be a caching layer at the level of the Pager which will cache these blocks of data in memory for increased performance.

### Reading the Node

Now we get to the fun part, seeing if we're at all on the right track by parsing the bytes into types that our system recognizes.


# Reading the test table node

<visualization of byte format for nodes>

# Reading varints

<visualization of byte format for varint>

 - Mention documentation https://sqlite.org/forum/info/11fc7308d5e607f2

# Reading the header

# Reading records
