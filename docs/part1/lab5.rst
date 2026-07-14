Assignment 5: Operators
=======================

In this lab, you will implement physical database operators using the iterator model and complete the student-owned parts of a small query parser and executor.

Submission Contract
-------------------

Submit exactly one file named ``operators.cpp``. In the provided skeleton ZIP file, that file is located at ``student_lab_5/src/operators/operators.cpp``.

All headers and every other ``.cpp`` file are fixed infrastructure. Do not modify or submit them. The fixed headers define the required member variables, API signatures, and state invariants. You may add file-local helper functions or types inside ``operators.cpp``, but you may not change the provided interfaces or class layouts.

The provided ``operators.cpp`` skeleton compiles before you fill in its TODOs. Successful compilation alone does not mean the operators are complete.

What You Implement
------------------

The TODOs in ``operators.cpp`` are the complete assignment surface:

- ``Field`` comparisons: ``!=``, ``<``, ``>``, ``<=``, and ``>=``. Equality (``==``) is provided.
- ``PrintOperator``: its constructor, ``open()``, ``next()``, and ``close()``. Its fixed ``getOutput()`` method returns no tuple.
- ``ProjectOperator``, ``Sort``, ``HashJoin``, and ``HashAggregationOperator``: their constructors and iterator methods, including output handling.
- Set and bag operators: ``UnionOperator``, ``UnionAll``, ``Intersect``, ``IntersectAll``, ``Except``, and ``ExceptAll``. Their constructors are provided; implement ``open()``, ``next()``, ``close()``, and ``getOutput()``.
- The private ``RelationManager`` parsing hooks for ``FROM``, aggregate functions, ``JOIN``, and ``ORDER BY``.
- The private ``RelationManager`` planning hooks that construct join, aggregation, and sort operators.

The query hooks are deliberately narrower than ``parseQuery()`` and ``executeQuery()``. Their surrounding parsing and execution orchestration is provided code.

Provided / Invariants
---------------------

The following behavior is already implemented outside ``operators.cpp``:

- ``Field`` storage, serialization, cloning, hashing, parsing, and equality.
- Tuple, page, storage, and buffer-management behavior.
- ``SimplePredicate``, ``ComplexPredicate``, and comparison-operator parsing.
- ``InsertOperator``, ``ScanOperator``, and ``SelectOperator``.
- Constructors for all set and bag operators, plus ``PrintOperator::getOutput()``.
- CSV loading, relation lookup and population, and the fixed portions of ``RelationManager::parseQuery()`` and ``RelationManager::executeQuery()``.

You may refer to the provided implementations of ``InsertOperator``, ``ScanOperator``, ``SelectOperator``, ``SimplePredicate``, and ``ComplexPredicate`` for examples of iterator lifecycle, input and output handling, field cloning, and ownership conventions.

These definitions are fixed for the lab and are not part of the student submission.

Iterator Interface
------------------

Every operator follows this interface:

- ``open()`` initializes the operator and opens its input or inputs.
- ``next()`` advances to the next result and returns ``true`` when a result is available.
- ``getOutput()`` returns ``std::vector<std::unique_ptr<Field>>`` for the current result.
- ``close()`` closes the inputs and releases or resets operator state.

Call ``getOutput()`` only after ``next()`` returns ``true``. Operators return owned or cloned fields rather than pointers into a child's internal tuple.

``PrintOperator`` writes one tuple per line. It separates fields with a comma and one space (``", "``) and terminates each tuple with a newline.

Operator Semantics
------------------

The following physical operators are part of the assignment:

- **Projection**: emits cloned fields in the requested attribute-index order so the result does not alias the input tuple.
- **Selection**: filters tuples using ``==``, ``!=``, ``<``, ``<=``, ``>``, or ``>=``. A ``SimplePredicate`` can compare an attribute with another attribute or with a constant.
- **Sort**: materializes its input and applies its criteria in order. A later criterion is considered only when all earlier criterion fields compare equal, and each criterion's ``desc`` flag controls its direction.
- **HashJoin**: performs an inner equi-join using left-side hash buckets and preserves every matching pair when join keys repeat.
- **HashAggregationOperator**: groups tuples and computes ``COUNT``, ``SUM``, ``MIN``, and ``MAX`` aggregates.
- **UnionOperator**, **Intersect**, and **Except**: use set semantics and remove duplicate output tuples.
- **UnionAll**, **IntersectAll**, and **ExceptAll**: use bag semantics and retain the multiplicity required by the corresponding bag operation. ``UnionAll`` emits the left bag followed by the right bag.

Hash Join Semantics
-------------------

``HashJoin`` is an inner equi-join. ``open()`` builds a hash table from the left input with one bucket per join-key value, preserving every left tuple in each bucket. ``next()`` probes with the right input and drains the complete matching left bucket before advancing to the next right tuple. Tuples without a matching key are not emitted.

Repeated keys use full many-to-many multiplicity. If a key occurs ``L`` times in the left input and ``R`` times in the right input, the join emits ``L * R`` results for that key. Each result is ordered as ``[left_columns..., right_columns...]``.

Aggregation Semantics
---------------------

With no grouping attributes, all input tuples belong to one global group. With ``GROUP BY``, one result is emitted for each group.

For every group, the aggregation operator preserves the first input tuple as the representative tuple. Its output schema is:

``[selected_fields_from_first_tuple..., aggregate_values...]``

If the aggregation receives no selected attributes, it emits all fields from the representative tuple before the aggregate values. Aggregate values appear in the order in which their functions occur in the query.

``COUNT`` starts at one for the first tuple in a group and increments once for each additional tuple. ``SUM`` is initialized from the first value and supports integer and floating-point fields. ``MIN`` and ``MAX`` are initialized from the first value; they support integer and floating-point fields and use lexicographic ordering for strings.

Set and Bag Semantics
---------------------

The set operators emit each qualifying tuple at most once. ``UnionOperator`` emits tuples present in either input, ``Intersect`` emits tuples present in both inputs, and ``Except`` emits tuples present in the left input but absent from the right input.

For the bag operators, let ``L`` and ``R`` be a tuple's multiplicities in the left and right inputs. ``UnionAll`` emits ``L + R`` copies, with all left tuples materialized before all right tuples. ``IntersectAll`` emits ``min(L, R)`` copies. ``ExceptAll`` emits ``max(L - R, 0)`` copies.

Query Syntax
------------

Queries use the parenthesized Lab 5 syntax used by the tests. For example:

.. code-block:: text

   SELECT (1,2,3,5) FROM (users) JOIN (posts) ON (2) = (3) ORDER BY (5)

A query may contain the following forms:

- ``SELECT (i,j,...)``
- ``FROM (relation)``
- ``JOIN (relation2) ON (i) = (j)`` for an optional inner equi-join
- ``WHERE (k) <op> literal`` with one or more conditions separated by ``AND``
- ``GROUP BY (i,j,...)``
- ``COUNT(i)``, ``SUM(i)``, ``MIN(i)``, and ``MAX(i)``; multiple aggregate functions may occur in one query
- ``ORDER BY (i)`` for ascending order

Clause keywords are expected in uppercase. The fixed ``WHERE`` parser accepts ``AND`` or ``and`` between conditions. The query grammar does not support ``OR``. A ``WHERE`` literal may be an integer, floating-point value, or quoted string.

When implementing the parsing hooks, remember that parentheses are special regular-expression characters. Escape the literal parentheses in the query syntax. For example, a C++ raw-string pattern for ``FROM (relation)`` can use ``R"(FROM \((\w+)\))"``. Unescaped parentheses create capture groups instead of matching the parentheses present in the query text.

Query Planning and Index Mapping
--------------------------------

``executeQuery()`` uses this operator order:

``Scan (and optional HashJoin) -> Select -> HashAggregationOperator (if needed) -> ProjectOperator (only without aggregation) -> Sort``

Use the following index-mapping rules:

- Query attribute positions are one-based and must be converted to zero-based indexes internally. Indexes passed directly to operator constructors are already zero-based.
- In ``JOIN (relation2) ON (i) = (j)``, ``i`` indexes the left input and ``j`` independently indexes the right input.
- After a join, the output tuple is ``[left_columns..., right_columns...]``. ``WHERE``, ``GROUP BY``, and aggregate indexes refer to the tuple flowing at that point in the pipeline.
- When aggregation has selected attributes, its output is ``[selected_columns_from_first_tuple..., aggregate_values...]``. Without selected attributes, all columns from the first tuple precede the aggregate values. Aggregate values retain query order, and ``ORDER BY`` refers to indexes in this emitted aggregation schema.
- Without aggregation, ``ORDER BY`` uses the source attribute position and the planner maps it through projection as needed.

Testing and Validation
----------------------

Your implementation should pass all provided tests. Pay particular attention to:

- Correctness of each operator's logic.
- Correct execution order.
- Correct index mapping after joins and projections.
- Correct output schema for aggregation.
- Proper handling of edge cases.
- Memory management and resource cleanup.

Build and Test
--------------

From the ``student_lab_5`` directory, configure and build the project with CMake:

.. code-block:: bash

   cmake -S . -B build
   cmake --build build
   cd build
   ./test

Run one public test with ``./test N``, where ``N`` is from 1 through 12. Only the public test executable is included in the provided skeleton ZIP file.

Frequently Asked Questions
--------------------------

**Do I implement equality for** ``Field`` **?**

No. ``operator==`` is provided. Implement the five comparison operators marked with TODOs in ``operators.cpp``.

**Do I implement** ``SelectOperator`` **or compound query predicates?**

No. ``SelectOperator``, ``SimplePredicate``, ``ComplexPredicate``, and the fixed ``WHERE`` parsing are provided. Your other operators use their output through the normal iterator interface. The query layer supports multiple ``WHERE`` conditions joined by ``AND``, but not ``OR``.

**May I add helper methods or member variables?**

Do not change the fixed headers or class layouts. You may add file-local helper functions and types inside ``operators.cpp``.

**Does an** ``All`` **operator simply concatenate its inputs?**

Only ``UnionAll`` concatenates both bags. ``IntersectAll`` emits the minimum multiplicity from the two inputs, and ``ExceptAll`` subtracts right-side multiplicity from left-side multiplicity without going below zero.
