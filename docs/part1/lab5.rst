Assignment 5: R-Tree
====================

In this assignment, you will implement the core logic of an on-disk R-Tree on top of a buffer manager. Your tree will support inserting 2D rectangles (MBRs), splitting pages when they overflow, maintaining parent bounding boxes, and answering spatial queries such as window (rectangle) queries and k-nearest neighbor queries.

Description
----------------------
  
You are given C++ code that already provides:

* A `StorageManager` that reads/writes fixed-size pages (4KB) to a `.dat` file.
* A pin-aware LRU `BufferManager` that lets you load a page, edit it, and flush it.
* An on-disk R-Tree node layout: a header followed by either leaf entries (MBR + rid) or inner entries (MBR + child page id).
* Utilities for basic Minimum Bounding Rectangle (MBR) math.
* A test harness that inserts points of interest (POIs) near Georgia Tech.

Your task is to complete the R-Tree parts so that:

1. Inserts go to the correct leaf.
2. Overfull nodes are split using the R-Tree quadratic split strategy.
3. Splits are propagated upward, creating a new root if needed.
4. Parent MBRs are kept consistent with their children.
5. Window queries return all objects overlapping a query rectangle.
6. kNN queries return the k closest objects to a point.
7. The provided tests pass.

All coordinates can be treated as plain 2D values; no special lat/long handling is required.

What You Will Implement
----------------------
You will complete the following R-Tree operations (names may match the starter code):

* **Insert**

  * `insert(const MBR& m, uint64_t rid)`
    Locate the appropriate leaf, insert the entry, and handle overflow.

* **Subtree choice**

  * `choose_subtree(PageID start, const MBR& m)`
    Starting from the root, at each inner node pick the child needing the *smallest enlargement* to contain `m` (tie-break by area, then by count), until you reach a leaf.

* **Insert into leaf + split**

  * `insert_in_leaf(PageID pid, const MBR& m, uint64_t rid)`
    If there is room in the leaf, append. If not, run the quadratic split on the leaf entries.

* **Quadratic split (leaf and inner)**

  Use the standard R-Tree quadratic split:

  * pick two “seed” entries that would waste the most area if grouped together,
  * assign remaining entries to the group that needs the least enlargement,
  * respect min-fill.

* **Split propagation**

  * `adjust_after_split(PageID original, PageID sibling)`
    If the original node had a parent, insert the new sibling into the parent and split again if the parent overflows. If there was no parent (we split the root), create a new root with two children.

  * `insert_subtree_entry(const MBR& m, PageID child, uint32_t child_level)`
    Insert a newly created child page into the correct tree level.

  * `choose_subtree_to_level(PageID start, const MBR& m, uint32_t want_level)`
    Variant of subtree choice that stops at a particular level (used when inserting an internal node).

* **MBR maintenance**

  * `MBR compute_node_mbr(PageID pid)`
    Recompute the bounding box of a node from all of its entries.

  * `void update_ancestors(PageID start)`
    After a change to a node, walk up the parent chain and refresh the MBRs stored in ancestor entries.

* **Queries**

  * `window_query(const MBR& q, Fn visit)`
    Depth-first search: for every node whose entry overlaps `q`, recurse (if inner) or emit (if leaf).

  * `knn(double x, double y, size_t k)`
    Best-first search using a priority queue keyed by distance from the query point to an entry’s MBR; output k objects.

The harness already has a radius query that is built on top of window query, so once your window query works, radius query should work too.

Implementation Details
----------------------
  
You will work with the provided C++ skeleton, which already defines:

* page size and node layout,
* helper casts to interpret a page as a header + leaf/inner entries,
* functions to allocate a new page from the underlying file,
* parameterization of max/min entries per node based on the page size.

Your implementation should follow these steps during insertion:

1. Start at the root and call the subtree-choice routine to find a leaf.
2. Insert into the leaf. If the leaf overflows:

   * run quadratic split (produces two nodes),
   * propagate this split upward using the adjust routine.
3. After any change, recompute MBRs where needed and update ancestors.

For queries, always check MBR overlap before recursing; this prunes the search.

Building the Code
-----------------

.. code-block:: bash

   g++ -fdiagnostics-color -std=c++17 -O3 -Wall -Werror -Wextra <file_name.cpp> -o <output_name.out>

Make sure that your project compiles without any warnings, as we treat warnings as errors.

Testing and Validation
----------------------

Test your implementation against provided unit tests in the main function. There are few additional test cases beyond the handout on Gradescope.

FAQs
----------------------
  
1. **Do I need to handle latitude/longitude or real earth distance?**

   No. Treat all coordinates as simple 2D numbers. The dataset looks geographic, but your R-Tree logic only needs axis-aligned MBRs.

2. **How do I pick which child to descend into on insert?**

   Use the R-Tree rule: choose the child whose MBR needs the *least enlargement* to contain the new entry. If tied, pick the one with smaller area; if still tied, pick the one with fewer entries.

3. **What if a node overflows after I insert?**

   Run the quadratic split on that node. Then insert the new sibling into the parent. If the parent overflows, repeat the process upward. If you split the root, create a new root with two children.

4. **Why do I have to recompute MBRs?**

   Because parent nodes store bounding boxes for their children. After you insert or split, those boxes may have changed. If they are not updated, window and kNN queries may miss results.

5. **What’s the difference between window query and kNN?**

   Window query returns *all* objects that overlap a rectangle. kNN returns the *k closest* objects to a point and stops after k, using a best-first traversal.

6. **Where do page IDs come from?**

   Use the provided page-allocation function in the R-Tree. It extends the underlying file and returns a new page id.

7. **Can I write small helpers inside the R-Tree?**

   Yes, as long as you keep the on-disk layout (header + entries) unchanged and keep using the supplied buffer manager API.
