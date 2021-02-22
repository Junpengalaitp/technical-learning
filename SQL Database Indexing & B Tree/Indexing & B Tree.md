# 1. Introduction
## Database Structure
* Database(Relational) -> Tables(Files) -> Records -> Columns

## Hard Disk Storage
* Divided into equal sizes blocks, usually 512KB per block.
* SQL tables are divided into hard disk block sized blocks, and stored in the hard disk

## File System
* File System read and store data sequentially most of the time.
* SQL tables need to be perform random access, so file system is not used.

# 2. Organize of File in the Disk
## Blocking Factor, Ordered File Organization vs Unordered File Organization

### Blocking Factor of a File
* max number of records which can be placed in a disk block
* Blocking Factor = Disk Block Size / Record Size

### Spanned Organization
* A Record can span across more than one block

### Unspanned Organization
* A Record cannot span across multiple blocks

#### Data transfer from disk to memory only happen in terms of blocks
* with spanned organization, read one record may transfer multiple blocks, which is costly.
* with unspanned organization, read one record only one block is transfer.
* unspanned organization is used most commonly

### Fixed & Dynamic length of records
* fixed: Unspanned Organization
* dynamic: Spanned Organization
* record size > block size: Spanned Organization

## Examples on Spanned & Unspanned Organization


# 3. Organize of Records in a File
## Ordered File Organization vs Unordered File Organization
* Ordered File Organization
  * Each block in disk stores an order file records, can be search in O(logn) time
* Unordered File Organization
  * Faster on insertion

# 4. Indexing Technique in Databases
## What is indexing in DBMS
* Sort sorted index by a column in disk, which points to the actual table record address.

## Dense Index vs Sparse Indexing
* dense: has an entry for every record
* sparse: has entries for some record

## Classification of Indexing Techniques
* Single Level Index
  * primary key index (primary key + ordered)
  * cluster index (non key + ordered)
  * secondary index (key/non key index)
* Multi Level Index
  * B Trees
  * B+ Trees
  
# 5. Primary Index
## Introduction
* Indexing is done based on primary key of data file.
* The actual data file is ordered based on primary key

1. Primary index is an ordered file with
   * primary key of data file
   * block pointer
2. Index entry is created for first record of each block of data file (called as anchor record)
3. Primary Index is a sparse index 
4. Number of index entries in primary index = number of blocks
5. Number of block accesses to search for a record(worst case) logI + 1

* Binary search on indices and find the target data block, than load that block to memory and search the target record
* Fast in two ways compared to dense index(index for every record)
  * dense index has more indices to search (logR time, R = num of records) compared to (logI, I = num of indices)
  * dense index operates in the disk entirely compare to sparse index operates last step on memory

## Disadvantages
* to remain sorted order, in the worst case, all blocks will be altered when inserting new data.
* indices have to change accordingly


# 6. Clustering Index
## Introduction
* Clustering index is created on a data file which file records are physically ordered based on non-key field
* Clustering index is a sparse index, created on first occurrence of every value of the clustering field
* Searching >= logI + 1
* Insertion = (B + I)
* When searching when duplicated records exist, binary search on indices to find the block of first occurrence of the target, because of the records are sorted, we can continue search on the next block of the first occurrence block
* Need to maintain a pointer in each block, which points to the next block

# 7. Secondary Index (unordered + key/non-key)
* Secondary Key Index (table is not ordered by key)
  * have to create indices for every record in the table
  * dense index 
  * O(logI) searching time
* Secondary Non-key Index
  * create index for unique values in the table, but the block pointer also maintains the record pointer
  * sparse index
  
# 8. Multi Level Indexing
## Introduction
* create primary indices on secondary indices.
* because the secondary index keys is ordered, we can create primary index for the index block.
* we can repeat the process and create more levels of indexes until there is only one index.
* all the indices are primary indices except for the first level index which can be any kind of index.

## Time Complexity of Multi Level Index
* O(logfI), f = blocking factor, I = indices

## Disadvantages of static multi level index
* ISAM(Indexed sequential Access Method)
* insertion is too costly

# 9. Dynamic Multi Level Indexing: Introduction to B Trees
## Introduction
* Each node in B-Tree is of from <P1, <K1, Ptr1>, P2, <K2, Ptr2>,...,<K(q-1), Ptr(q-1)>Pq> where q <= p
  * Pi: tree pointer
  * Ki: Key
  * Ptri: Data Pointer

* With in each node, K1 < K2 < ... < K(q - 1)
* For all search values X in the subtree pointer by Pi, K(i-1) < X < Ki
* Each node has at most p tree pointers
* Each node except node and leaf, has at least ceil(p/2) tree pointers, root and leaf node have at least 2 tree pointers
* A node with q tree pointers will have q - 1 index entries.
* All leaf nodes will be at the same level

* B-Tree order: max tree pointers in a node

# 10. Insertion to B Trees
* always insert to a leaf node
* if the leaf node overflows(exceeds max index entry) after insertion, do a node split
  * take the mid index entry to parent and the left and right as its children
  * if the parent overflows, repeat this process
* on insertion, only overflow can happen