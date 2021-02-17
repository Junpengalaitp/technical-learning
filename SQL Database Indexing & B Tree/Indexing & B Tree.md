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


# 4. Organize of Records in a File
## Ordered File Organization vs Unordered File Organization
* Ordered File Organization
  * Each block in disk stores an order file records, can be search in O(logn) time
* Unordered File Organization
  * Faster on insertion