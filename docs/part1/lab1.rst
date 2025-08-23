Assignment 1: C++ Crash Course
==============================

Description
-----------

The goal of this assignment is to **refresh your C++ systems programming skills** by implementing a tiny flat-file social media application over three CSV files: `users.csv`, `posts.csv`, and `engagements.csv`. You will load data into memory, maintain simple indexes, support concurrent updates, and persist changes back to CSV atomically.

Implementation Details
----------------------

You are given skeleton code to finish and submit. Fill in the methods in the `FlatFile` class. You can add helper functions or small classes if they help, but **do not change any existing function signatures in `FlatFile`**. Here are some more details to help with this assignment:

1. **Input files:** Three CSV files are provided — `users.csv`, `posts.csv`, and `engagements.csv` — containing user details, posts, and engagements.

2. **Function signatures:** Do not change any function signatures. Implement the missing logic as indicated in the inline comments and tests (e.g., `loadFlatFile` should load CSV data into memory using appropriate data structures).

3. **Data structures:** You may choose any STL containers and supporting structures, unless a specific structure is explicitly required by the assignment.

4. **Single-file implementation:** All classes and logic are contained in a single C++ source file. The `main` function includes all test cases. **Review the test code carefully to understand the expected behavior of each function**.

5. **Class definitions:** Skeleton definitions for `User`, `Post`, and `Engagement` are provided; use them when loading and manipulating CSV data.

6. **Building the code:** Compile the single source file with the following
   command:
   ``g++ -fdiagnostics-color -std=c++17 -O3 -Wall -Werror -Wextra <file_name.cpp> -o <output_name.out>``
   Compiler warnings are treated as errors to encourage more careful systems programming.

7. **Testing Instructions**: Run a specific test with:
   ``./<output_name.out> <test_number>`` For example,
   ``./a.out 2`` runs the second test case. Running without an argument executes all tests. **Gradescope will use these same tests**.

8. **Uploading to Gradescope**: Just upload `buzzdb_lab1.cpp` to Gradescope. No Makefile or zip is required; the autograder will compile your single-file submission and provide the CSV inputs.

Functions to be Completed
-------------------------

You will need to complete the following methods in the `FlatFile` class (do not change any existing function signatures):

1. **Constructor**  
   `FlatFile(std::string users_csv_path, std::string posts_csv_path, std::string engagements_csv_path)`  
   Store the CSV paths and initialize internal state.

2. **Destructor**  
   `~FlatFile()`  
   Default cleanup is sufficient unless you add resources that need explicit release.

3. **loadFlatFile**  
   `void loadFlatFile()`  
   Single-threaded load of `users.csv`, `posts.csv`, and `engagements.csv` into the in-memory maps. Build/refresh any secondary indexes (e.g., username → user_id). Ignore empty lines; trim cells; parse numerics strictly.

4. **loadMultipleFlatFilesInParallel**  
   `void loadMultipleFlatFilesInParallel()`  
   Load the three CSVs concurrently (e.g., one thread per file), then atomically commit to the main maps under locks. Rebuild indexes and remove dangling engagement records that reference missing users/posts.

5. **updatePostViews**  
   `bool updatePostViews(int post_id, int views_count)`  
   Thread-safe increment of a post's `views`. Must be **durable**: rewrite the posts CSV using a temp file and an atomic rename in the same directory. Return `false` if `post_id` does not exist.

6. **addEngagementRecord**  
   `void addEngagementRecord(Engagement& record)`  
   Validate foreign-key-like constraints (`record.postId` exists and `record.username` is known). If valid, append to `engagements.csv` and update memory under appropriate locks.

7. **getAllUserComments**  
   `std::vector<std::pair<int, std::string>> getAllUserComments(int user_id)`  
   Return all `(postId, comment)` pairs for the specified user where `type == "comment"`, sorted by `(postId, comment)`.

8. **getAllEngagementsByLocation**  
   `std::pair<int, int> getAllEngagementsByLocation(std::string location)`  
   For all users in the given `location`, count engagements and return `{likes_count, comments_count}`.

9. **updateUserName**  
   `bool updateUserName(int user_id, std::string new_username)`  
   Update the username in memory and across **all three CSV files** (`users`, `posts`, `engagements`) consistently and durably (atomic rewrites). Preserve post/engagement counts; return `false` if `user_id` is not found.

Additional Guidance
-------------------

- **Thread safety & durability:**  
  Ensure `updatePostViews` is thread-safe (use `std::mutex` or equivalent) **and** durable. Persist updates with a temp-file rewrite followed by an atomic rename in the same directory so readers never observe partial writes.

- **CSV parsing:**  
  Use standard C++ I/O (`fstream`, `stringstream`). Trim ASCII whitespace, ignore empty lines, and parse integers strictly (reject malformed values). **Handle short or malformed rows gracefully by skipping them**.

- **Referential integrity:**  
  When adding an engagement, verify that both the `postId` exists and the `username` refers to a known user. During parallel loads, remove or ignore dangling engagements that reference missing users/posts.

- **Error handling & return values:**  
  Prefer clear status returns for expected issues (e.g., return `false` if a `post_id` or `user_id` is not found) instead of throwing. Maintain consistent in-memory state and secondary indexes after mutations.

- **Performance considerations:**  
  Build/refresh lightweight indexes (e.g., username → id) to keep queries efficient. Keep lock scopes minimal, and commit results atomically after parallel loading.

- **Testing expectations:**  
  Implementations are evaluated by the provided tests in `main`. Read the test code to understand required behaviors, edge cases, and ordering guarantees.

Learning Goals
--------------

- Learn about file I/O and CSV parsing  
- Practice using C++ STL containers and building simple indexes  
- Implement thread-safe and durable updates  
- Enforce basic referential integrity  
- Reason about performance (wall time, CPU, memory)

Input Data Model
----------------

Each CSV includes a header row:

- `users.csv`: `id,username,location`  
- `posts.csv`: `id,content,username,views`  
- `engagements.csv`: `id,postId,username,type,comment,timestamp`

Assumptions: ignore empty lines; numeric fields must be integers.

Test Overview 
--------------

- **Single-threaded load:** Counts/keys match expected.  
- **Parallel load speedup:** Parallel loading time must be less than that of serial loading time on multi-core; cardinalities must match.  
- **User comments:** Correct ordering and contents; stable across reload.  
- **Engagements by location:** Counts match, including after appends.  
- **Rename user:** Username updated across all files; counts preserved.  
- **Line-growth safety:** Rewrites still parse when numbers grow digits.  
- **Resource guardrails:** Under limits for wall time/CPU/RSS during load.  
- **Atomicity of post view updates:** Concurrent increments persist accurately.  
- **Invalid post id:** `updatePostViews` returns `false`.  
- **Reader/Writer isolation:** Readers never see broken state while a writer updates.  
- **Crash-during-write durability:** File remains parseable; value non-decreasing.  
- **Referential integrity:** No dangling `engagement.postId`.  
- **Type sanity:** Numeric columns always parse (no corruption).