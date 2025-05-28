Assignment 1: C++ Crash Course
==============================

Description
-----------

The goal of this assignment is to help you brush up on your C++
programming skills and exercise your skills in Data Structure and
Algorithm Design. In this assignment, you are to develop a program
written in C++ that will allow us to extract the required information
from the provided input CSV files. This is a variant of a flat-file
database implementation, and you are expected to implement the provided
empty functions to return the expected output.

Implementation Details
----------------------

For this programming assignment, you are provided with skeleton code
that you are expected to complete and submit. You are expected to fill
in the methods present in the ``FlatFile`` class. You are free to add
your own helper functions and classes to complete the assignment. Please
do not change the signature of any functions in the ``FlatFile`` class.
Here are some more details to help with this assignment:

1. **Input Files**: There are three input files in CSV format:
   ``users.csv``, ``posts.csv``, and ``engagements.csv``. These contain
   data related to user profiles, their social media posts, and
   engagements on those posts.

2. **Function Signatures**: Do not change the function signatures.
   Complete the missing implementations as instructed in the inline
   comments to get the desired output. For example, the ``loadFlatFile``
   function is expected to load the data present in the input CSV files
   into memory by creating objects of appropriate classes and storing
   them in relevant data structures for optimal updating and querying.
   Similarly, complete all the empty functions.

3. **Data Structures**: You are free to use any data structures as long
   as the implementation passes the provided test cases unless
   explicitly mentioned to use a specific data structure. In such cases,
   the submitted code will be manually checked for compliance.

4. **Single File Implementation**: To make understanding the assignment
   code simpler, all the classes and logic are implemented in a single
   C++ file. The main function contains all the test cases that you need
   to pass. Please review all test case code to understand what is being
   tested for each function. This will give you ideas about what is
   expected from the function implementations.

5. **Class Definitions**: The skeleton code contains definitions of
   ``User``, ``Post``, and ``Engagement`` classes. You are expected to
   use these while loading the CSV data into memory.

6. **Building the Code**: We only have one C++ file that contains all
   the code for this assignment. This can be built using the following
   command:
   ``g++ -fdiagnostics-color -std=c++17 -O3 -Wall -Werror -Wextra <file_name.cpp> -o <output_name.out>``
   We treat compiler warnings as errors. Your project will fail to build
   if there are any compiler warnings.

7. **Testing Instructions**: To run any particular test case, use:
   ``./<output_name.out> <test_number>`` For example,
   ``./a.out 2`` runs the second test case. Not providing any test
   number runs all the test cases one by one. We have provided all the
   test cases for this lab. Gradescope will only test your code against
   these test cases. Your implementation will also be checked for memory
   leaks. You can check for memory leaks using Valgrind with:
   ``valgrind --leak-check=yes <output_name.out> ...<args>``

Functions to be Completed
-------------------------

You need to complete the following functions in the ``FlatFile`` class:

1. **Constructor:**
   ``FlatFile(std::string users_csv_path, std::string posts_csv_path, std::string engagements_csv_path)``

   -  Initialize the class and load the CSV file paths.
   -  Example:
      ``FlatFile flatFile("users.csv", "posts.csv", "engagements.csv");``

2. **Destructor:** ``~FlatFile()``

   -  Properly clean up any resources used by the class.

3. **loadFlatFile:** ``void loadFlatFile()``

   -  Load the provided CSV files into the corresponding maps (Single
      threaded).

4. **loadMultipleFlatFilesInParallel:**
   ``void loadMultipleFlatFilesInParallel()``

   -  Load the provided CSV files into the corresponding maps using
      multiple threads

5. **incrementPostViews:**
   ``bool incrementPostViews(int post_id, int views_count)``

   -  Increase the views count for the post associated with the post_id
      by views_count. This method should be thread-safe.

6. **addEngagementRecord:**
   ``void addEngagementRecord(Engagement& record)``

   -  Insert a new engagement record into the engagements map.

7. **getAllUserComments:**
   ``std::vector<std::pair<int, std::string>> getAllUserComments(int user_id)``

   -  Return all the comments made by the user across all the posts,
      ordered by post ID and comment.

8. **getAllEngagementsByLocation:**
   ``std::pair<int, int> getAllEngagementsByLocation(std::string location)``

   -  Return the count of all engagements (likes and comments) for all
      users from a given location.

9. **updateUserName:**
   ``bool updateUserName(int user_id, std::string new_username)``

   -  Update the username for the given user_id across all files where
      it appears.

Additional Guidance
-------------------

-  **Thread Safety:** Ensure that the function
   ``incrementPostViews`` is thread-safe. You might want to use
   mutexes or other synchronization mechanisms to achieve this.

-  **CSV Parsing:** You can use standard C++ libraries such as
   ``fstream`` and ``sstream`` for reading and parsing CSV files. Make
   sure to handle edge cases such as empty fields or malformed CSV
   lines.

-  **Error Handling:** Implement robust error handling in your
   functions. For example, if an invalid ``post_id`` or ``user_id`` is
   provided, ensure that your functions return appropriate values or
   handle the error gracefully.

-  **Testing:** Thoroughly test your implementation against the provided
   test cases. Make sure to understand the test case requirements by
   reviewing the main function.

Good luck with your assignment!
