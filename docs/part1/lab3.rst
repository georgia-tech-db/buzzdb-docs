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

You will be utilizing a buffer manager, which simplifies node access via page IDs instead of direct pointers. 

**Persistent B-Tree Requirement:** 
Your `BTree` must be **persistent**. This means that the tree is backed by the buffer manager's underlying database file, allowing it to "remember" its state across different executions or database instances. If the buffer manager loads an existing database file containing a previously populated B-Tree, your constructor (`BTree(BufferManager& bm)`) must dynamically reconstruct the tree's state (i.e., locate the `root` node). You can achieve this by fetching a known existing node (e.g., using your `next_page_id` allocation logic) and tracing the `parent_node_id` pointers upward until you reach the root.

Implementation Details
----------------------

Your B+-Tree implementation should be designed as a C++ template accommodating key type, value type, comparator, and page size as parameters. You are explicitly tasked with calculating the maximum capacity for `InnerNode` and `LeafNode` using `sizeof` arithmetic to compute `kCapacity`.

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

**splitNode(std::vector<std::shared_ptr<Node>> path, std::shared_ptr<Node> node):**
This helper function is responsible for splitting a full node (leaf or inner) and updating parent pointers accordingly. It should create a new node, redistribute keys (and values or child pointers), and propagate the key upward. By utilizing the provided `path` vector of ancestor nodes, you should handle parent updates and any necessary cascading splits internally within this method.

Structure
~~~~~~~~~

- **Node**: Base structure with common properties like ID and parent references (fully provided).
- **InnerNode**: Manages keys and child node pointers, handling node traversal and splits. The memory layout is strictly defined.
- **LeafNode**: Stores actual key-value pairs and handles direct data operations. The memory layout is strictly defined.

Key Methods
~~~~~~~~~~~

- **lookup(KeyT)**: Retrieves a value by its key.
- **insert(KeyT, ValueT)**: Inserts a key-value pair.
- **erase(KeyT)**: Deletes a key.
- **rangeQuery(KeyT, KeyT)**: Retrieves all key-value pairs within a specified range (inclusive of low and high).

Building the Code
-----------------

The project uses CMake for building. You can build the codebase using standard CMake commands:

.. code-block:: bash

   mkdir build && cd build
   cmake ..
   make

Make sure that your project compiles without any warnings, as we treat warnings as errors.

Testing and Validation
----------------------

Test your implementation using the provided test suite. The unit tests have been separated from the main source code into dedicated external test files (e.g., `test/test.cpp`). These tests instantiate your B+-Tree as `BTree<uint64_t, uint64_t, std::less<uint64_t>, 1024>` to ensure functionality across different scenarios.

FAQs
----

1. **Can you clarify the terminologies used in this assignment?**

   - **Key**: Key of an element you want to insert/lookup/erase from the dictionary.
   - **Value**: Corresponding value associated with the key.
   - **Page ID**: Same as what we had in Assignment 2. This can be thought of as an identifier for a page and helps to determine the offset for a page on the db file.
   - **Node ID**: This is the same as Page ID because each node is stored in a separate page.

2. **What are differences between the split methods for inner node and leaf node?**

   - Both the inner and leaf nodes' split methods should create one new node. The node undergoing the split will reduce in size and transfer some keys and child pointers/values to the newly created node.
   - Assigning the parent pointer and updating the parent's child pointers should be handled entirely internally within `splitNode`, utilizing the `path` vector to trace ancestry and cascade splits if necessary.

3. **Will the node's keys array have duplicate values?**

   - Repeated inserts with the same key should be treated as "updates". For example, if a (K1, V1) already exists in the tree and we call insert(K1, V2), then lookup(K1) should return V2.

4. **How to fix a page?**

   - BufferManager provides a method to access individual nodes via page ID. In our setup, each node is stored on a distinct page, effectively creating a one-to-one mapping between nodes and pages. To access a node within the tree, you must request the BufferManager to fix the page using the page ID associated with that node.

5. **Is it mandatory to use binary search?**

   - Yes, it is mandatory to use binary search. 
   - You can also use std::lower_bound/std::upper_bound from the C++ STL.