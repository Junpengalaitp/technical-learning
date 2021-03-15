# 1. Introduction
## Database Structure
* Database(Relational) -> Tables(Files) -> Records -> Columns

## Hard Disk Storage
* Divided into equal sizes blocks, usually 512KB per block.
* SQL tables are divided into hard disk block sized chunks, and are stored in the hard disk

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
* with unspanned organization, read one record only takes one block access.
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
* The actual data file is ordered based on primary keys.

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

# 9. Dynamic Multi Level Indexing: Introduction to B-Trees
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

# 10. Insertion in B-Trees
* always insert to a leaf node
* if the leaf node overflows(exceeds max index entry(order - 1)) after insertion, do a node split
  * take the mid index entry to parent and the left and right as its children
  * if the parent overflows, repeat this process
* on insertion, only overflow can happen

# 11. Deletion in B-Trees
### Delete a leaf node
 * No underflow(index entries is less than the min index entry ceil(p/2) - 1), do nothing
 * underflow
  * borrow from sibling(left or right rotate)
  * if both siblings are not available, merge with sibling and bring the parent entry down to merged node
    * this may lead to parent underflow, in this case, merge the parent with one of parent sibling
    * after parent merge, parent node will have one more tree pointer, bring down one index entry from grandparent
* if root node is empty, remove the root node

### Delete in a non-leaf node
* replace the index entry with its inorder predecessor or successor(always on leaf nodes), pick predecessor only or successor only, do not need to compare, this is because we want minimize the hard disk operations which is costly
* if its leads to underflow to the leaf node, deal it with the same method as leaf node deletion

# 12. B+ Tree 
* all data pointers exist in leaf nodes, non-leaf nodes only contain keys.
* all leaf nodes have a pointer points to the next leaf node, so the leaf nodes are forming a linked list

# 13. Insertion in B+ Tree
* left node degree and non-leaf node degree
* when overflow, do a node split similar to B-Tree, but only pull mid value key to parent and keep the mid <key, value> in one of the children
* Select * from table where id >= 5;
  * find the leaf node of 5 and use left pointers to get all data that great than 5 without traverse the whole tree
* Left biasing & Right biasing
  * determines the one extra data going to left or right node when splitting odd number of nodes
  * in left(right) biasing, the largest(smallest) value in left(right) node is the same with the key value

# 14. Deletion in B+ Tree
* need to maintain the key-children same property
  * change the parent key with the largest value in the deletion node
* underflow
  * borrow
  * merge
  * no rotation on leaf node
  * rotation on non-leaf node

* advantages of B+ Tree over B-Tree
  * B+ Tree only store key value in non-leaf node, so it can store more tree pointers in a single level, and have shorter height
  * linked list leaves, range query execution is faster