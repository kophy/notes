# Zettabyte File System

A new file system with strong data integrity, simple administration and immense capacity.

**Features**
1. redesign of interface between file system and volume manager
2. pooled storage
  - multiple file systems share a pool of storage devices
  - decouple file systems from physical storage
3. immense capacity
  - 128-bit block addresses
  - scalable algorithms for directory lookup, block allocation, etc
4. transactional copy-on-write model
5. checksum all on-disk data

**Implementation**
![Traditional file system vs ZFS](images/block-diagram.jpg)
1. Overview
  - SPA handles block allocation and I/O; exports virtually addressed, explicitly allocated and freed blocks to DMU
  - DMU turns virtually addressed blocks into transactional object interface
  - ZPL implements posix file system on DMU objects; exports vnode operations to the system call layer

2. SPA
  - verify block checksum when reading the block update when writing (a block's checksum is stored in its parent block except uberblock)
  - use slab allocator to prevent memory fragmentation (copy-on-write file system needs big contiguous chunks to write new blocks)

3. DMU
  - on-disk data is stored in a tree of blocks. root is called uberblock and leaves are data blocks
  - when a block is written, allocate a new block and copy modified content into the new block
  - transaction is implemented by "rippling" from data block to uberblock
  - group transactions together so we can rewrite uberblock and indirect blocks only once for many data block writes
![Copy on write](images/copy-on-write.jpg)

4. ZPL
  - use intent log to avoid losing writes before system crashes
