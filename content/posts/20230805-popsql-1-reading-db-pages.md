---
title: "popSQL part 1: reading the sqlite database file"
date: 2023-08-05T10:26:00-04:00
tags: ["sql", "python", "sqlite"]
categories: ["software"]
---

# Reading the database files

My approach this time will be to start from the absolute bottom of the system, the file format. I chose this as a starting place because it's easy to test in addition to being well documented. From there I'll design a close-enough-to the source set of abstractions which are unit tested fairly comprehsively.

Before we write any code, let's look at the directory for the code this section. You'll notice a pager, node, and cell in the backend - a util file, a main file, and a test suite. At this point, we create any folders and `__init__.py` files needed to support the structure. You'll also notice a `test.db` which we'll create using sqlite itself, and ideally be parsing out using our own code by the end of this exercise.

```
/pypopsql
.
├── main.py
├── src
│   ├── backend
│   │   ├── __init__.py
│   │   ├── node.py
│   │   └── pager.py
│   └── util.py
├── test
│   └── __init__.py
└── test.db
```

# Getting started

In approaching the build out this time, I figured the easiest thing to test against the functionality with SQLite was the file format, mainly because it would be simple to read and compare what SQLite actually writes. It's simple enough to create a database with a single table and then see what the file looks like on a byte, by byte level.

```
$ sqlite3 test.db
> CREATE TABLE test(col1 VARCHAR(2), col2 INTEGER);
> INSERT INTO test(col1, col2) VALUES ('hi', 1), ('yo', 2);
```

Once the database has been created you can also see how large it is. One question I had that didn't seem immediately answered by the documentation was would a database with almost no data still fill up entire pages.

```
➜  budlightlime git:(bdillmann/popsql-1) ✗ ls -lh ~/projects/popsql/pypopsql/test.db
-rw-r--r--  1 godzilla  staff   8.0K Aug  2 19:12 /Users/godzilla/projects/popsql/pypopsql/test.db
```


It seems like the answer is yes. Creating a single user table with 2 rows yields an 8k file, which consulting the [file format](https://www.sqlite.org/fileformat2.html) leads me to believe that the page contents are as follows:

 * Page 1. Header (.dbinfo) and Schema Table Leaf Node
 * Page 2. Table Leaf Node for the test table which I just created.

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

```python
# pager.py
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

## Interfacing with the BTree

SQLite like many other relational databases stores it's data in b-trees, a balanced tree which is designed to work best on disk. The simple reason for this is that each Node has a lot of children, because each node fits in a single page read. If a page is 4096 bytes, that means there can be a lot of child pointers in that page.

There's lots of literature on how btrees work and why they're used by databases, so I'll omit that here but feel free to do your own research on this topic if you're interested. I'll dive right into reading the data.

### Reading the Node

Now that we have an abstraction for reading data to and from a database file, we'll be able to parse out some of the properties of the individual btree nodes. We'll start with the node header, which is the first 8 or 12 bytes of the page - depending on whether the page is an internal or leaf node.

The header is laid out bytewise like this:

```
[ a, b, b, c, c, d, d, e, (f, f, f, f)]

a -> one byte node type identifier
b -> two byte integer pointer representing the first freeblock
c -> two byte integer pointer identifying the start of the cell content
d -> one byte count of fragmented free bytes within the cell content area
f -> right pointer for this node (only included in internal nodes)
```

Using this information, we can write a utility function for turning bytes into an integer and then write the code that reads the node header.

```python
# util.py
def b2i(b: bytes):
    return int.from_bytes(b, 'big', signed=False)
```

And the node parser, which includes a function for parsing the header as well as debug utility which allows us to see the values of the node.

```python
# btree/node.py
from enum import Enum
from typing import Tuple

from util import b2i

class NodeType(Enum):
    INDEX_INTERIOR = 2
    TABLE_INTERIOR = 5
    INDEX_LEAF = 10
    TABLE_LEAF = 13

class Node:
    def __init__(
        self,
        data: bytes,
    ):
        self.data = data
        self.page_size = len(data)
        
        self.node_type, \
        self.cell_offset, \
        self.num_cells, \
        self.right_pointer, \
        self.first_freeblock, \
        self.num_fragmented_bytes = self.read_header_bytes(data)


    def read_header_bytes(
        self,
        data: bytes,
    ) -> Tuple[
        NodeType,
        int, # num cells
        int, # cell_offset
        int, # right_pointer
        int, # first_freeblock
        int, # num_fragmented_bytes
    ]:
        offset = 0

        node_type = NodeType(b2i(data[offset + 0: offset + 1]))

        first_freeblock = b2i(data[offset + 1: offset + 3])

        num_cells = b2i(data[offset + 3: offset + 5])

        cell_offset = b2i(data[offset + 5: offset + 7])

        num_fragmented_bytes = data[offset + 7]

        right_pointer = None
        if not self.is_leaf(node_type):
            right_pointer_bytes = data[offset + 8: offset + 12]
            right_pointer = b2i(right_pointer_bytes)

        return (
            node_type,
            cell_offset,
            num_cells,
            right_pointer,
            first_freeblock,
            num_fragmented_bytes,
        )

    def is_leaf(self, node_type: NodeType = None):
        node_type = node_type or self.node_type
        return node_type in (NodeType.TABLE_LEAF, NodeType.INDEX_LEAF)

    def _debug_print_header(self):
        print('node type', self.node_type)
        print('first freeblock', self.first_freeblock)
        print('cell content start', self.cell_offset)
        print('num fragmented bytes', self.num_fragmented_bytes)
        print('right pointer', self.right_pointer)
```

And finishing this step up with a simple main file which allows us to test this behavior:

```python
def test_btree():
    pager = Pager('test.db') # the test database we created earlier
    data = pager.get_page(2) # read the second page, the one with our test table
    node = Node(data)
    node._debug_print_header(self)

if __name__ == '__main__':
    test_btree()
```

If we've done everything correctly, then running our main file should look like this:

```
➜  python3 main.py
node type NodeType.TABLE_LEAF
first freeblock 0
cell content start 4069
num fragmented bytes 0
right pointer None
```

We should be able to read the row data now that we have the header being read appropriately.

# Reading the test table node

<visualization of byte format for nodes>

``````


# Reading varints

<visualization of byte format for varint>

 - Mention documentation https://sqlite.org/forum/info/11fc7308d5e607f2

# Reading the header

# Reading records
