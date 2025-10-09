Assignment 3: B+-Tree Implementation
=====================================

For the third programming assignment, you will implement a B+-Tree for your database system. This data structure will enhance the database's ability to efficiently query, insert, and delete data by using page IDs and a buffer manager for managing node access.

Description
-----------

In this assignment, you will develop a B+-Tree that supports the following operations:

- **lookup**: Return the value associated with a specific key or indicate that the key is not found.
- **insert**: Add a new key-value pair to the tree.
- **erase**: Remove a specified key from the tree.
- **rangeQuery**: Retrieve all key-value pairs within a specified key range (inclusive of low and high).

You will be utilizing a buffer manager, which simplifies node access via page IDs instead of direct pointers, allowing you to focus solely on the B+-Tree logic.

Implementation Details
----------------------

Your B+-Tree implementation should be designed as a C++ template accommodating key type, value type, comparator, and page size as parameters. This design requires compile-time constants for page size.

Operations
~~~~~~~~~~

The B+-Tree has two types of Nodes: **LeafNodes** and **InnerNodes**. The stubs for both are provided in the template, and you are expected to implement their methods as described below.

**LeafNode:**
1. **insert(const KeyT &key, const ValueT &value)**  
   Inserts or updates a key-value pair in sorted order.  

2. **erase(const KeyT &key)**  
   Removes the given key (and its associated value) from the leaf.  
   You do not need to handle rebalance or merge operations for this assignment.  

**InnerNode:**
1. **find_child_index(const KeyT &key)**  
   Determines which child pointer should be followed for a given key using binary search.  

2. **insert(const KeyT &key, uint64_t child_page_id)**  
   Inserts a key and child pointer into the inner node (typically after a lower-level split).

The BTree class has a private method splitNode.

**splitNode:**
This helper function is responsible for splitting a full node (leaf or inner) and updating parent pointers accordingly. It should create a new node, redistribute keys (and values or child pointers), and propagate the key upward. Ensure that parent references remain consistent. If the parent is full, the updates could cascade toward the root.

Structure
~~~~~~~~~

- **Node**: Base structure with common properties like ID and parent references.
- **InnerNode**: Manages keys and child node pointers, handling node traversal and splits.
- **LeafNode**: Stores actual key-value pairs and handles direct data operations.

Key Methods
~~~~~~~~~~~

- **lookup(KeyT)**: Retrieves a value by its key.
- **insert(KeyT, ValueT)**: Inserts a key-value pair.
- **erase(KeyT)**: Deletes a key.
- **rangeQuery(KeyT, KeyT)**: Retrieves all key-value pairs within a specified range (inclusive of low and high).

Building the Code
-----------------

.. code-block:: bash

   g++ -fdiagnostics-color -std=c++17 -O3 -Wall -Werror -Wextra <file_name.cpp> -o <output_name.out>

Make sure that your project compiles without any warnings, as we treat warnings as errors.

Testing and Validation
----------------------

Test your implementation against provided unit tests in the main function. These tests instantiate your B+-Tree as `BTree<uint64_t, uint64_t, std::less<uint64_t>, 1024>` to ensure functionality across different scenarios.

FAQs
----

1. **Can you clarify the terminologies used in this assignment?**

   - **Key**: Key of an element you want to insert/lookup/erase from the dictionary.
   - **Value**: Corresponding value associated with the key.
   - **Page ID**: Same as what we had in Assignment 2. This can be thought of as an identifier for a page and helps to determine the offset for a page on the db file.
   - **Node ID**: This is the same as Page ID because each node is stored in a separate page.

2. **What are differences between the split methods for inner node and leaf node?**

   - Both the inner and leaf nodes' split methods should create one new node. The node undergoing the split will reduce in size and transfer some keys and child pointers/values to the newly created node.
   - Assigning the parent pointer can be handled internally (since both the original and the newly created nodes will share the same parent). However, updating the child pointers in the parent should be done externally, utilizing the key returned from the split method.

3. **Will the node's keys array have duplicate values?**

   - Repeated inserts with the same key should be treated as "updates". For example, if a (K1, V1) already exists in the tree and we call insert(K1, V2), then lookup(K1) should return V2.

4. **How to fix a page?**

   - BufferManager provides a method to access individual nodes via page ID. In our setup, each node is stored on a distinct page, effectively creating a one-to-one mapping between nodes and pages. To access a node within the tree, you must request the BufferManager to fix the page using the page ID associated with that node.

5. **Is it mandatory to use binary search?**

   - Yes, it is mandatory to use binary search. 
   - You can also use std::lower_bound/std::upper_bound from the C++ STL.