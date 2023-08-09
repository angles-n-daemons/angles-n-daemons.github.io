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
│   │   ├── cell.py
│   │   ├── node.py
│   │   ├── record.py
│   │   └── pager.py
│   ├── dbinfo.py
│   └── util.py
├── test
│   └── __init__.py
│   ... various test files
└── test.db
```

I should note that this section will lean extremely heavily on the [file format](https://www.sqlite.org/fileformat2.html) page in the SQLite documentation so I'll try to include locations to jump to as we start to write code.

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


It seems like the answer is yes. Creating a single user table with 2 rows yields an 8k file, which consulting the file format leads me to believe that the page contents are as follows:

 * Page 1. Header (.dbinfo) and Schema Table Leaf Node
 * Page 2. Table Leaf Node for the test table which I just created.

I assume that the first page is going to be more complicated to parse than the second page, and a lot of the process for parsing the second page can be reused for the first page.

From here, I figure the minumum testable codebase will require:
1. A pager for reading and writing to the database file.
2. An abstraction which takes database a database page and reads the header out of it.

After this is done, I'll be able to go further by reading out the btree nodes as well as the row values stored inside of them.

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

# Reading the database header, or .dbinfo

Now that we have a pager reading data from disk let's try to parse out the header. The file format documentation specifies the header with the following table:

> | Offset | Size | Description |
> | ------ | ---- | ----------- |
> | 0  |  16 |  The header string: "SQLite format 3\000"
> | 16 |  2  |  The database page size in bytes. Must be a power of two between 512 and 32768 inclusive, or the value 1 representing a page size of 65536.
> | 18 |  1  |  File format write version. 1 for legacy; 2 for WAL.
> | 19 |  1  |  File format read version. 1 for legacy; 2 for WAL.
> | 20 |  1  |  Bytes of unused "reserved" space at the end of each page. Usually 0.
> | 21 |  1  |  Maximum embedded payload fraction. Must be 64.
> | 22 |  1  |  Minimum embedded payload fraction. Must be 32.
> | 23 |  1  |  Leaf payload fraction. Must be 32.
> | 24 |  4  |  File change counter.
> | 28 |  4  |  Size of the database file in pages. The "in-header database size".
> | 32 |  4  |  Page number of the first freelist trunk page.
> | 36 |  4  |  Total number of freelist pages.
> | 40 |  4  |  The schema cookie.
> | 44 |  4  |  The schema format number. Supported schema formats are 1, 2, 3, and 4.
> | 48 |  4  |  Default page cache size.
> | 52 |  4  |  The page number of the largest root b-tree page when in auto-vacuum or incremental-vacuum modes, or zero otherwise.
> | 56 |  4  |  The database text encoding. A value of 1 means UTF-8. A value of 2 means UTF-16le. A value of 3 means UTF-16be.
> | 60 |  4  |  The "user version" as read and set by the user_version pragma.
> | 64 |  4  |  True (non-zero) for incremental-vacuum mode. False (zero) otherwise.
> | 68 |  4  |  The "Application ID" set by PRAGMA application_id.
> | 72 |  20 |  Reserved for expansion. Must be zero.
> | 92 |  4  |  The version-valid-for number.
> | 96 |  4  |  SQLITE_VERSION_NUMBER 

First wew need a utility function for turning bytes into an integer and then write the code that reads the node header:

```python
# src/util.py
def b2i(b: bytes) -> int:
    return int.from_bytes(b, 'big', signed=False)
```

Now we can start reading the above database info with the below minimal `dbinfo.py` file:

```python
# src/dbinfo.py
from enum import Enum

from src.util import b2i

class FileFormatVersion(Enum):
    LEGACY = 1
    WAL = 2

class DBInfo:
    def __init__(
        self,
        data: bytes,
    ):
        header_str = data[:16]
        if header_str != b'SQLite format 3\x00':
            raise Exception('header string not found, result is ', header_str)
        
        self.page_size = b2i(data[16:18])

        self.file_format_write_version = FileFormatVersion(data[18])
        self.file_format_read_version = FileFormatVersion(data[19])

    def _debug(self):
        print('page size', self.page_size)
        print('file format write version', self.file_format_write_version)
        print('file format read version', self.file_format_read_version)
```

We can create a simple main file to test this behavior:

```python
# main.py
from src.dbinfo import DBInfo
from src.backend.pager import Pager

def test_dbinfo():
    pager = Pager('test.db') # the test database we created earlier
    data = pager.get_page(1) # the first page of the database
    dbinfo = DBInfo(data)
    dbinfo._debug(self)

if __name__ == '__main__':
    test_dbinfo()
```

And if we run it we should see the following output:

```
➜  python3 main.py
page size 4096
file format write version FileFormatVersion.LEGACY
file format read version FileFormatVersion.LEGACY
```

Great! For the sake of brevity I've omitted the rest of the `DBInfo` definition, but feel free to look at the source link at the bottom of the page to see what the rest of it looks like.

## Interfacing with the BTree

SQLite like many other relational databases stores it's data in b-trees, a balanced tree which is designed to work best on disk. The simple reason for this is that each Node has a lot of children, because each node fits in a single page read. If a page is 4096 bytes, that means there can be a lot of child pointers in that page.

There's lots of literature on how btrees work and why they're used by databases, so I'll omit that here but feel free to do your own research on this topic if you're interested. I'll dive right into reading the data.

### Reading the Node

Now that we have an abstraction for reading data to and from a database file, we'll be able to parse out some of the properties of the individual btree nodes. We'll start with the node header, which is the first 8 or 12 bytes of the page - depending on whether the page is an internal or leaf node.

The header is laid out bytewise like this:

> | Offset | Size | Description |
> | ------ | ---- | ----------- |
> | 0  | 1  | The one-byte flag at offset 0 indicating the b-tree page type.
> | 1  | 2  | The two-byte integer at offset 1 gives the start of the first freeblock on the page, or is zero if there are no freeblocks.
> | 3  | 2  | The two-byte integer at offset 3 gives the number of cells on the page.
> | 5  | 2  | The two-byte integer at offset 5 designates the start of the cell content area. A zero value for this integer is interpreted as 65536.
> | 7  | 1  | The one-byte integer at offset 7 gives the number of fragmented free bytes within the cell content area.
> | 8  | 4  | The four-byte page number at offset 8 is the right-most pointer. This value appears in the header of interior b-tree pages only and is omitted from all other pages.

And the values for the page type byte:
> * A value of 2 (0x02) means the page is an interior index b-tree page.
> * A value of 5 (0x05) means the page is an interior table b-tree page.
> * A value of 10 (0x0a) means the page is a leaf index b-tree page.
> * A value of 13 (0x0d) means the page is a leaf table b-tree page.
> * Any other value for the b-tree page type is an error.

So we can create a node parser, which includes a function for parsing the header as well as debug utility which allows us to see the values of the node.

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

    def is_leaf(self, node_type: NodeType = None) -> bool:
        node_type = node_type or self.node_type
        return node_type in (NodeType.TABLE_LEAF, NodeType.INDEX_LEAF)

    def _debug(self):
        print('node type', self.node_type)
        print('first freeblock', self.first_freeblock)
        print('cell content start', self.cell_offset)
        print('num fragmented bytes', self.num_fragmented_bytes)
        print('right pointer', self.right_pointer)
        print('\n')
```

And finishing this step up with a simple main file which allows us to test this behavior:

```python
# main.py
from src.backend.node import Node
from src.backend.pager import Pager

def test_dbinfo():
    ...

+ def test_btree():
+     pager = Pager('test.db') # the test database we created earlier
+     data = pager.get_page(2) # read the second page, the one with our test table
+     node = Node(data)
+     node._debug(self)

if __name__ == '__main__':
-    test_dbinfo()
+    test_btree()
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

# Reading cell pointers and content

In the btree literature, each record is referred to as a btree "cell". Each btree node has a list of cell pointers immediately following the header, of which each points to some cell content. The cell content grows leftward from the end of the page, which means that the center of the page is occupied by empty space. The cell pointers are ordered, and the cell content isn't necessarily in the same order as the pointers. The layout of the page generally looks as follows:

```
[h, cp1, cp2, cp3..., 0, 0, 0, 0, 0, ...cc3, cc2, cc1]

h -> header section
cpi -> cell pointer i, two byte pointer which points to the offset in the page where cell i's content lies
cci -> cell content i, variable sized payload for the actual information housed by cell i
```

Let's get started by reading the cell pointers and then follow up later by reading the full cell content. We can create a minimal TableLeafCell implementation which holds only the pointer to the cell content as well as a debug function itself:

```python
# src/backend/cell.py

class TableLeafCell:
    def __init__(
        self,
        data: bytes,
        pointer: int,
    ):
        self.pointer = pointer

    def _debug(self):
        print('cell at index', self.pointer)
        print('\n')
```

And then read the cells from our Node implementation. To do this we add a function `read_cells`, and add reading and storing the cells to the Node constructor, and modify our debug utility to print out the cell pointer values. For this step we'll only read the cell pointers to see that we're getting the right values.


```python
# src/backend/node.py
...
from src.backend.cell import TableLeafCell

class Node:
    def __init__(self...)
        ...

        self.cells = self.read_cells(data)

    def read_cells(
        self,
        data: bytes,
    ) -> List[any]:
        page_header_len = 8 if self.is_leaf() else 12

        cells = []

        for i in range(self.num_cells):
            offset = page_header_len + (i * 2)
            p = b2i(data[offset:offset + 2])
            cell = TableLeafCell(data, p)
            cells.append(cell)

        return cells

    def is_leaf(self, node_type: NodeType = None) -> bool:
        ...

    def _debug(self):
        ...

        for i, cell in enumerate(self.cells):
            cell._debug()
```

Testing our main function again, we should see the following output:

```
➜  python3 main.py
node type NodeType.TABLE_LEAF
first freeblock 0
cell content start 4069
num fragmented bytes 0
right pointer None


cell at index 4077


cell at index 4069
```

Now that we're ready to read the cell content, we need to add a utility to parse varint values. Varints are a variable length integer format described in the sqlite documentation. The varints in the simplest terms are integers that have a length of 1-9 bytes where the first bit of each byte describes whether the integer has terminated, and the last 7 bits are the integer payload. This excepts the 9th byte if included which is always filled with 8 bits of payload information as the integer cannot carry on past the 9th byte.

Varints are used extensively in the sqlite data format, and will be required to parse the cell headers.

We add it here to the `util.py` file:

```python
# src/util.py
from typing import Tuple

def b2i(b: bytes) -> int:
    ...

def varint(b: bytes, cursor: int) -> Tuple[int, int]:
    """
    varint reads a variable length int from a byte string

    it takes the following parameters
     - b: sequence of bytes, generally a full table page
     - cursor: starting index of the varint

    and returns a tuple containing
    - the value of the integer
    - index of the cursor after the last byte of the varint
    """

    result = 0

    for j in range(8):
        i = cursor + j

        # read the next byte into an integer
        byte_num = b[i]

        # result shifts left 7 bits, then the first 7 bits of byte_num are appended
        result = (result << 7) | (byte_num & 0x7f)

        # check the first bit of byte_num to see if we should continue reading
        continue_reading = byte_num & 0x80

        if not continue_reading:
            return result, cursor + j + 1

    # read last byte, use all 8 bytes to fill the remaining spaces
    byte_num = b[cursor + 8]
    result = (result << 8) | byte_num

    return result, cursor + 9
```

In starting this exercise I modeled my tests after the comments in the source and found a typo which was nice. I was surprised how quick and responsive the community was to [this inquiry](https://sqlite.org/forum/info/11fc7308d5e607f2).

With this utility, we can begin to parse the cell payload for a table leaf as defined by the file format:

 * A payload size varint, which includes itself in the total definition
 * A rowid value as a varint
 * The actual row data, which is encoded in a later defined format


```python
# src/backend/cell.py

class TableLeafCell:
    def __init__(
        self,
        data: bytes,
        pointer: int,
    ):
        self.pointer = pointer
+       cursor = pointer
+
+       self.payload_size, cursor = varint(data, cursor)
+       self.row_id, cursor = varint(data, cursor)
+
+       self.payload_data = data[cursor:cursor + self.payload_size]

    def _debug(self):
        print('cell at index', self.pointer)
        print('payload size', self.payload_size)
        print('row id', self.row_id)
        print('\n')
```

Running our main file one more time gives us a little more information:

```
➜  python3 main.py
node type NodeType.TABLE_LEAF
first freeblock 0
cell content start 4069
num fragmented bytes 0
right pointer None


cell at index 4077
payload size 5
row id 1
record data b'\x03\x11\thi'


cell at index 4069
payload size 6
row id 2
record data b'\x03\x11\x01yo\x02'
```

# Reading records

At this point we're able to start reading the meat of what the database stores, the rows themselves also referred to as records.

The payload mentioned above in the cells is going to be the home for the record values.

The record starts with a header as well, which specifies how many bytes the row header will be.

The rest of the record header is a sequence of varints, each one corresponding with the type of the column.

The types are defined in the file format, which you can reference before looking at the example row header below:

```
0x03 0x11 0x01
 |     |    |
 |     |    |
 |     |    |
 |     |     - second column a 8 bit twos-complement integer
 |     |
 |      - > first column is a string of length (17-13) / 2 = 2 bytes long
 |
  - > size of the header is 4 bytes (including this byte)
```

We can create an Column class as well as a ColumnType class for reading this information effectively. It's a bit meaty so I've omitted some of the values for brevity.

We'll also add a minimal Record class which takes the cell payload and parses out the column values.

```python
# src/backend/record.py
from enum import Enum
from typing import Tuple
from dataclasses import dataclass

from src.util import b2i, varint

class ColumnType(Enum):
    TINYINT = 1
    TEXT = 13
    ...

    @classmethod
    def _missing_(cls, value: int):
        if value < 12:
            return cls(value)
        elif value % 2 == 0:
            return cls(12)
        else:
            return cls(13)

@dataclass
class Column:
    def __init__(self, column_type: ColumnType, length: int=None):
        self.type = column_type
        self.length = length

    def __repr__(self):
        return f'column: {self.type}, {self.length}'

    @classmethod
    def from_int(cls, value: int):
        column_type = ColumnType(value)
        length = None

        # calculate length for BLOB and TEXT types as documented
        if column_type == ColumnType.BLOB:
            length = (value - 12) // 2
        elif column_type == ColumnType.TEXT:
            length = (value - 13) // 2

        return cls(column_type, length)

class Record:
    def __init__(
        self,
        data: bytes,
        cursor: int,
    ):
        self.data = data
        self.columns, cursor = self.read_column_types(data, cursor)
        self.cursor = cursor

    def read_column_types(
        self,
        data: bytes,
        cursor: int,
    ):
        columns = []
        cursor_start = cursor
        num_bytes_header, cursor = varint(data, cursor)

        while cursor - cursor_start < num_bytes_header:
            column_type_int, cursor = varint(data, cursor)
            columns.append(Column.from_int(column_type_int))

        return columns, cursor

    def _debug(self):
        for i, column in enumerate(self.columns):
            print('column type', column.type)
```

If we add the Record type's debug call to our TableLeafCell implementation we should be able to test the column reading:

```python
# src/backend/cell.py
+ from src.backend.record import Record
from src.util import varint

class TableLeafCell:
    def __init__(
        self,
        data: bytes,
        pointer: int,
    ):
        ...

-       self.payload_data = data[cursor:cursor + self.payload_size]
+       self.record = Record(data, cursor)

    def _debug(self):
        ...
-       print('record data', self.payload_data)
+       self.record._debug()
        print('\n')
```

If we rerun our main function again we should see the column types:

```
➜  python3 main.py
node type NodeType.TABLE_LEAF
first freeblock 0
cell content start 4069
num fragmented bytes 0
right pointer None


cell at index 4077
payload size 5
row id 1
column type ColumnType.TEXT
column type ColumnType.ONE


cell at index 4069
payload size 6
row id 2
column type ColumnType.TEXT
column type ColumnType.TINYINT
```

Great! So now that we have the column types, we only need to parse the row values into the record and we should have our btree node parsing a full vertical slice of the page.


```python
# src/backend/record.py
...
class Record:
    def __init__(
        self,
        data: bytes,
        cursor: int,
    ):
        self.columns, cursor = self.read_column_types(data, cursor)
+       self.values, cursor = self.read_values(data, cursor)
        self.cursor = cursor

    def read_column_types(
        self,
        data: bytes,
        cursor: int,
    ):
        ...

+    def read_values(
+        self,
+        data: bytes,
+        cursor: int,
+    ):
+        values = []
+        for column in self.columns:
+            value, cursor = self.read_value(column.type, data, cursor, column.length)
+            values.append(value)
+        return values, cursor
+
+    @staticmethod
+    def read_value(
+        column_type: ColumnType,
+        data: bytes,
+        cursor: int,
+        length: int = None,
+    ) -> Tuple[any, int]:
+        ... # other if statements omitted
+        if column_type == ColumnType.TINYINT:
+            return int(data[cursor]), cursor + 1
+        elif column_type == ColumnType.TEXT:
+            return data[cursor: cursor + length].decode('utf-8'), cursor + length
+        else:
+            raise Exception(f'cannot parse column type {column_type}')

    def _debug(self):
        for i, column in enumerate(self.columns):
            print('column type', column.type)
+           print('value', self.values[i])
```

Testing again we should see the row values:

```
cell at index 4077
payload size 5
row id 1
column type ColumnType.TEXT
value hi
column type ColumnType.ONE
value 1


cell at index 4069
payload size 6
row id 2
column type ColumnType.TEXT
value yo
column type ColumnType.TINYINT
value 2
```

We've now completed reading a full btree node that was written by sqlite.

# Reading the schema page

From *2.6. Storage Of The SQL Database Schema* in the SQLite file format document:

> Page 1 of a database file is the root page of a table b-tree that holds a special table named "sqlite_schema". This b-tree is known as the "schema table" since it stores the complete database schema.

And in *1.3. The Database Header*

> The first 100 bytes of the database file comprise the database file header.

Which means that for the first page of the database, we should be parsing both the header and a b-tree page. To account for the header offset we modify the Node file:


```python
# src/backend/node.py
class Node:
    def __init__(
        self,
        data: bytes,
+       db_header: bool=False,
    ):
        self.data = data
        self.page_size = len(data)
+       self.db_header = db_header

        self.node_type, \
        self.cell_offset, \
        self.num_cells, \
        self.right_pointer, \
        self.first_freeblock, \
-        self.num_fragmented_bytes = self.read_header_bytes(data)
+        self.num_fragmented_bytes = self.read_header_bytes(data, db_header)

-       self.cells = self.read_cells(data)
+       self.cells = self.read_cells(data, db_header)

    def read_cells(
        self,
        data: bytes,
+       db_header: bool=False,
    ) -> List[any]:
        page_header_len = 8 if self.is_leaf() else 12
        db_header_len = 100 if db_header else 0

        cells = []

        for i in range(self.num_cells):
-           offset = page_header_len + (i * 2)
+           offset = db_header_len + page_header_len + (i * 2)
            ...


    def read_header_bytes(
        self,
        data: bytes,
+       db_header: bool=False,
    ) -> Tuple[
        NodeType,
        int, # num cells
        int, # cell_offset
        int, # right_pointer
        int, # first_freeblock
        int, # num_fragmented_bytes
    ]:
+       offset = 100 if db_header else 0
        ...
```

```python
# main.py

...
def test_btree():
    ...

def test_schema_page():
    pager = Pager('test.db')
    data = pager.get_page(1) # read the first page this time

    dbinfo = DBInfo(data)
    dbinfo._debug()

    node = Node(data, True) # pass in the db_header paramater to the Node constructor
    node._debug()

if __name__ == '__main__':
-   test_btree()
+   test_schema_page()
```

And we should see the following output:

```
➜  python3 main.py
page size 4096
file format write version FileFormatVersion.LEGACY
file format read version FileFormatVersion.LEGACY
page reserved space 12
maximum embedded payload fraction 64
minimum embedded payload fraction 32
leaf payload fraction 32
file change counter 2
database size in pages 2
first freelist trunk page 0
number of freelist pages 0
schema cookie 1
schema format number SchemaFormat.FORMAT_4
default page cache size 0
page of largest btree root 0
user version 0
text encoding TextEncoding.UTF_8
incremental_vacuum_mode 0
application id 0
version valid for 2
sqlite version number Version(major=3, minor=39, patch=5)
node type NodeType.TABLE_LEAF
first freeblock 0
cell content start 4014
num fragmented bytes 0
right pointer None


cell at index 4014
payload size 68
row id 1
column type ColumnType.TEXT
value table
column type ColumnType.TEXT
value test
column type ColumnType.TEXT
value test
column type ColumnType.TINYINT
value 2
column type ColumnType.TEXT
value CREATE TABLE test(col1 VARCHAR(2), col2 INTEGER)
```

So by this point we should be able to fully parse each byte of the database we created. This ended up being a bit more work than I had expected, but you can see the full codebase on [github](https://github.com/angles-n-daemons/pypopsql/tree/pypopsql/1/reading-the-db-file).

Next week we'll get started writing the database files to disk.
