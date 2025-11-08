Assignment 4: Operators
=======================

In this assignment, you will implement physical operators using the iterator model for your database system. This will involve building various operator classes that perform fundamental database operations such as selection, projection, sorting, joins, aggregation, and set operations.

Description
-----------

Your task is to develop physical operators that enable your database to execute SQL-like queries efficiently, and to extend a simple query layer that parses and executes queries over relations. These operators form the core of query execution, handling tasks like filtering data, combining datasets, sorting, and computing aggregations.

You will implement the following operations:

- **Print**: Outputs tuples with attributes separated by commas and tuples separated by newlines.
- **Projection**: Generates tuples containing a subset of attributes.
- **Select**: Filters tuples based on a predicate involving relational operators (``==``, ``!=``, ``<``, ``<=``, ``>``, ``>=``). Predicates compare an attribute (``l``) to another attribute or constant (``r``).
- **Sort**: Sorts tuples based on specified attributes and sort directions (ascending or descending).
- **HashJoin**: Performs an inner equi-join between two inputs on a specified attribute.
- **HashAggregation**: Groups tuples and computes aggregates (``SUM``, ``COUNT``, ``MIN``, ``MAX``) over groups or globally.
- **Union**: Computes the union of two inputs without duplicates (set semantics).
- **UnionAll**: Computes the union of two inputs including duplicates (bag semantics).
- **Intersect**: Retrieves common tuples from two inputs without duplicates (set semantics).
- **IntersectAll**: Retrieves common tuples from two inputs including duplicates (bag semantics).
- **Except**: Returns tuples in the first input not present in the second input without duplicates (set semantics).
- **ExceptAll**: Returns tuples in the first input not present in the second input including duplicates (bag semantics).

Additionally, extend the ``parseQuery`` and ``executeQuery`` methods to support:

- ``SELECT {i,j,...}`` attribute lists, or ``{*}``
- ``FROM {rel}``
- ``JOIN {rel2} ON {i} = {j}`` (inner equi-join)
- ``WHERE`` with one or more conditions combined by ``AND``; each condition uses ``{k} <op> <value>``
- ``GROUP BY {i, j, ...}`` (one or more attributes)
- Aggregate functions ``COUNT {i}``, ``SUM {i}``, ``MIN {i}``, ``MAX {i}``
- ``ORDER BY {i}`` (ascending)

Implementation Details
----------------------

You will work with the provided C++ skeleton code, which includes the base ``Operator`` class and derived classes for each operator, as well as a lightweight CSV-backed relation manager. Your implementation will involve:

1. **Field Comparison Logic**: Implement comparison operators for the ``Field`` class to support predicates in selection and other operations.

2. **Operator Classes**: Implement the ``open()``, ``next()``, and ``close()`` methods for each operator. Where applicable, implement the ``get_output()`` method to retrieve the current tuple.

   - ``open()``: Initialize the operator before execution.
   - ``next()``: Retrieve the next tuple. Return ``true`` if a tuple is available.
   - ``close()``: Release resources after execution.
   - ``get_output()``: Return the pointers to the current tuple's fields after a successful ``next()``.

You may add additional member functions and variables to support your implementation. Use the provided ``Scan`` and ``Select`` operators as references.

3. **Query parsing and execution (extended)**:

   Extend ``parseQuery`` to extract and store:

   - Selected attributes (1-based in queries -> 0-based internally)
   - Single-relation ``FROM`` and optional ``JOIN {rel2} ON {i} = {j}``
   - One or more ``WHERE`` conditions (combined by ``AND``). Each condition is ``{idx} <op> <literal>`` where literal may be int, float, or quoted string
   - ``GROUP BY`` with one or more attributes
   - One or more aggregate functions ``COUNT``, ``SUM``, ``MIN``, ``MAX``
   - ``ORDER BY {i}`` (ascending)

   Implement ``executeQuery`` to build the operator tree in this order:

   - Scan (and optional HashJoin) -> Select -> HashAggregation (if present) -> Project (only if no aggregation) -> Sort

   Index mapping rules you must follow:

   - Query indices are 1-based; convert to 0-based internally
   - After a join, the output tuple is ``[left_columns..., right_columns...]``. Indices in WHERE, GROUP BY, and aggregates refer to the tuple flowing at that point in the pipeline
   - When aggregation is present, the aggregation operator emits: ``[selected_columns_from_first_tuple..., aggregate_values...]`` (in the order aggregates were specified). ``ORDER BY`` refers to these emitted indices

4. **Aggregation semantics**:

   - Without ``GROUP BY``: emit a single row containing the aggregate value(s); or, if select attributes are provided to aggregation, emit those columns of the (first) input tuple followed by the aggregate value(s)
   - With ``GROUP BY``: for each group, emit the selected columns from the first tuple of the group (or all columns if no select list was provided to aggregation), then append aggregate value(s)
   - Support INT/FLOAT semantics where applicable; MIN/MAX on strings use lexicographic order

5. **Join semantics**:

   - Implement an inner equi-join via HashJoin
   - Build the hash table on the left input; probe using the right input
   - The joined tuple order is: all columns from the left tuple, followed by all columns from the right tuple

**Iterator Model**: Operators should interact using the iterator model, where each operator pulls data from its child operator(s).

Testing and Validation
----------------------

Your implementation should pass all provided test cases to ensure correctness. Focus on:

- Correctness of each operator's logic.
- Correct execution order.
- Correct index mapping after joins and projections.
- Correct output schema for aggregation.
- Proper handling of edge cases.
- Memory management and resource cleanup.

Building the Code
-----------------

Compile your code using:

.. code-block:: bash

   g++ -fdiagnostics-color -std=c++17 -O3 -Wall -Werror -Wextra <file_name.cpp> -o <output_name.out>

Ensure your code compiles without warnings or errors, as warnings are treated as errors.

FAQs
----

**How do I implement comparison logic for the ``Field`` class?**

You need to overload the comparison operators (``==``, ``!=``, ``<``, ``<=``, ``>``, ``>=``) for the ``Field`` class to compare field values correctly. This is essential for the ``Select`` operator and any operation that relies on comparing attribute values.

**What is the iterator model used in this assignment?**

The iterator model is a way of processing data where each operator requests data from its child by calling ``next()``. If the child returns ``true``, the operator can then process the data. This allows for efficient, pipeline-style execution with minimal memory overhead.

**How should I handle sorting in the ``Sort`` operator?**

In the ``Sort`` operator's ``open()`` method, you can read all tuples from the child operator and store them in a data structure like a vector. Then, sort the vector based on the specified attributes and sort directions. In the ``next()`` method, return tuples from the sorted vector one at a time.

**What's the difference between set semantics and bag semantics?**

- **Set Semantics**: Duplicate tuples are not included in the result. Operations like ``Union``, ``Intersect``, and ``Except`` eliminate duplicates.
- **Bag Semantics**: Duplicates are included in the result. Operations like ``UnionAll``, ``IntersectAll``, and ``ExceptAll`` retain duplicates based on their occurrence in the inputs.

**How do I extend ``parseQuery`` and ``executeQuery`` for SQL-like queries?**

Parse the supported clauses (``SELECT``, ``FROM``, optional ``JOIN``, optional multi-condition ``WHERE`` separated by ``AND``, optional multi-attribute ``GROUP BY``, one or more aggregates, optional ``ORDER BY``) and construct the operator tree in the required order. Ensure index mapping is correct after joins and projections.

**Can I add additional helper methods or variables?**

Yes, you are encouraged to add helper methods and member variables as needed to support your implementation, as long as they adhere to the assignment's requirements.

**How do I handle multiple aggregates in ``HashAggregation``?**

You can use a data structure (e.g., a map) to group tuples based on the grouping attributes. Then, compute the aggregates for each group by iterating over the grouped data and calculating the required aggregate functions.

**Do I need to handle complex predicates in ``Select``?**

For this assignment, each predicate consists of a single relational operator between an attribute and another attribute or constant. You are not required to handle compound predicates involving logical operators like ``AND`` or ``OR``.

**What resources can I refer to for this assignment?**

- The skeleton code and provided operator implementations (``Scan``, ``Select``).
- Lecture notes or textbooks covering database operators and the iterator model.
- Online resources or documentation on database physical operator implementations.

**What are the expected join and aggregation outputs?**

- Join output: ``[left_columns..., right_columns...]``.
- Aggregation output: ``[selected_columns_from_first_tuple..., aggregate_values...]`` (or all columns from the first tuple if no select list was given to aggregation).

---