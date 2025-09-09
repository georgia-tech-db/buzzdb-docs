Assignment 2: Buffer Manager
============================


Description
-----------

For the second programming assignment, you will design and implement a buffer manager that handles page management for a database system. The buffer manager will allow pages to be loaded, locked, and accessed concurrently by multiple threads. The core functionality of the buffer manager includes fetching pages from disk, storing them in memory, managing page replacement using the 2Q caching policy, and ensuring efficient thread-safe access to pages. Your implementation should support concurrency and ensure the correct reading and writing of pages to disk.

This assignment involves a fair amount of code, and we **strongly recommend** starting early to ensure sufficient time for debugging and testing.


Key Requirements
-------------------------------

Locks and Concurrency
~~~~~~~~~~~~~~~~~~~~~~

- Use `std::shared_mutex` to handle shared/exclusive access to pages. Pages locked in shared mode can be accessed by multiple readers, while exclusive locks are required for write access.
- When fixing a page, increment the use counter to track how many threads are accessing the page. Decrement this counter when the page is unfixed.
- Ensure that locks are held for the shortest possible duration, especially during disk I/O operations, to improve concurrency.

2Q Replacement Strategy
~~~~~~~~~~~~~~~~~~~~~~~~

- Pages start in the FIFO queue when first loaded into memory. If they are accessed again while still in the FIFO queue, they are moved to the LRU queue.
- When evicting pages, attempt to evict pages from the FIFO queue first. If all FIFO pages are still in use, eviction should occur from the LRU queue.
- Dirty pages (modified pages) must be written back to disk before eviction. This operation should be done under an exclusive lock.

BufferFrame Dirty Flag
~~~~~~~~~~~~~~~~~~~~~~

- Modified pages should be marked as dirty. Dirty pages must be written to disk when evicted to prevent data loss. Track when a page is modified and handle the flushing of dirty pages during eviction.


Implementation Overview
-----------

We provide skeleton code required to implement the buffer manager. There is only a single C++ source file where you will complete your implementation. This file already contains a significant portion of working `buzzdb` components covered in the class lectures. The key classes include:

- **`BufferFrame`**: This class handles individual pages. It stores metadata such as page IDs, dirty flags, and whether the page is locked exclusively. However, it does not directly manage concurrency or page eviction.

- **`BufferManager`**: This class controls concurrency, page replacement, and the 2Q page replacement strategy. It is responsible for managing shared and exclusive locks, page eviction, and fixing/unfixing pages. You will implement these operations, ensuring proper thread safety and efficient page access.


Implementation Details
--------------------

The `fix_page` and `unfix_page` methods are the core of your buffer manager implementation. Here's a high-level guide for implementing them:

`fix_page(page_id, exclusive)`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Description:** Retrieves a page from the buffer pool, loading it from disk if necessary, and locks it for either shared (read) or exclusive (write) access. This mechanism ensures controlled, concurrent access to pages in a multi-threaded environment.

**Parameters:**

1. page_id: The unique identifier of the page to be accessed.
2. exclusive: Determines the lock mode: 

    true: Locks the page exclusively for write operations.

    false: Locks the page in shared mode for read operations.

**Returns:** A reference to the BufferFrame object corresponding to the requested page.

**Steps to follow:**

1. **Check the LRU Queue**:
    - If the page is found in the LRU queue, lock it (shared or exclusive), increment the use counter, and return the page.

2. **Check the FIFO Queue**:
    - If the page is in the FIFO queue, promote it to the LRU queue. Lock the page, increment the use counter, and return it.

3. **Page Not Found in Memory**:
    - If the page is not in memory, find a free slot in the buffer pool.
    - If the buffer is full, evict a page from the FIFO queue first. If no evictable pages are in the FIFO queue, attempt eviction from the LRU queue.
    - Ensure that the eviction process is thread-safe and does not interfere with other threads fixing or unfixing pages.

4. **Load Page from Disk**:
    - Load the page from disk using `StorageManager`.
    - Insert the page into the FIFO queue and initialize its metadata (e.g., page ID, lock status).

5. **Lock the Page**:
    - Lock the page based on the requested access type (shared or exclusive).
    - If the page is locked in shared mode, multiple threads may access it concurrently. If locked exclusively, only one thread may access it.
    - Increment the use counter for the page.

6. **Write Back Dirty Pages**:
    - If evicting a dirty page (i.e., a modified page), write it back to disk before removing it from memory.
    - This operation must be done under an exclusive lock to ensure no other threads modify the page while it's being written to disk.

`unfix_page(page, is_dirty)`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Description:** Releases a previously fixed page from the buffer pool, updating its status based on whether it was modified. This process is crucial for maintaining data consistency and managing buffer resources efficiently.

**Parameters:**

1. page: The page to be unfixed.
2. is_dirty: Indicates whether the page was modified during its fixed period:

    true: The page was modified and should be marked as dirty

    false: The page remains unmodified.

**Steps to follow:**

1. **Mark Page as Dirty**:
    - If the page has been modified (i.e., `is_dirty` is true), mark the page as dirty so it will be written to disk before eviction.

2. **Decrement Use Counter**:
    - Reduce the use counter for the page. If the counter reaches zero, the page can potentially be evicted.

3. **Unlock the Page**:
    - Release the lock on the page (shared or exclusive), based on how it was originally locked.


Implementation Clarifications
----------------------

- Use `StorageManager` methods to load or flush pages when necessary.
- Policy class: You may (but are not required to) create a new 2Q policy class that inherits from the provided one. It is up to you.
- Buffer eviction policy:

  1. Evict from **FIFO queue** first.
  2. If no evictable pages in FIFO, then evict from **LRU queue**.
  3. If neither queue has evictable pages (all pages are fixed), you must throw a **buffer_full_error**.

- FrameID: It can be treated as an identifier or index for a BufferFrame.


Implementation Guidelines
----------------------

- Ensure thread-safety by properly managing locks and atomic operations. You need read locks for reads and write locks for modifications to avoid concurrency issues.
- Don't miss to consistently update auxiliary data structures (e.g., page ID → frame ID map).


General Guidelines
----------------------

- We encourage you to complete the single-threaded implementation before moving to multi-threaded test cases.
- Your implementation should pass all tests. Tests with `Multithread` in their names have a 30-second timeout to prevent deadlocks from blocking the testing process.
- Passing earlier tests doesn’t guarantee correctness. Later cases might expose deeper flaws in eviction and synchronization logic.


Building the Code
-----------------

You can build the code with the following command:

```
g++ -fdiagnostics-color -std=c++17 -O3 -Wall -Werror -Wextra <file_name.cpp> -o <output_name.out>
```

Make sure that your project compiles without any warnings, as we treat warnings as errors.


Testing the Code
-----------------

To run a particular test case, use:

```
./<output_name.out> <test_number>
```

For example, `./a.out 2` runs the second test case. If no test number is provided, all test cases will be executed one by one.
