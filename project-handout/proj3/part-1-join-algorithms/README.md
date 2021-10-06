# Part 1: Join Algorithms

![Datatape](../../../.gitbook/assets/datatape.png)

In this part, you will implement some join algorithms: block nested loop join, sort merge, and grace hash join. You can complete Task 1, Task 2 and Task 3 in **any order you want**. Task 4 is dependent on the completion of Task 3.

Aside from when the comments tell you that you can do something in memory, everything else should be **streamed**. You should not hold more pages in memory at once than the given algorithm says you are allowed to. Doing otherwise may result in no credit.

**Note on terminology**: in lecture, we sometimes use both block and page describe the unit of transfer between memory and disk. In the context of join algorithms, however, page refers to the unit of transfer between memory and disk, and block refers to a set of one or more pages. All uses of the word `block` in this part refer to this second definition \(a set of pages\).

**Convenient assumptions**:

* For all iterators that will be implemented in this project you can assume `hasNext()` will always be called before `next()`.
* Any Record object provided through an argument or as an element of a list or iterator will never be `null`.
* For testing purposes, we will **not** be testing behavior on invalid inputs \(`null` objects, negative buffer sizes or buffers too small to perform a join, invalid queries, etc...\). You can handle these inputs however you want, or not at all.
* Your join operators, sort operator, and query plans do not need to account for underlying relations being mutated during their execution.

## Your Tasks

### Task 1: Nested Loop Joins

**Update \(03/04/2021\)**: If you received your copy of the skeleton code before March 1st, then the following docstring for fetchNextRightPage is incorrect:

"rightPageIterator should be set to a backtracking iterator over up to one page of records from the left source, and leftRecord should be set to the first record in this block."

and should instead be:

"rightPageIterator should be set to a backtracking iterator over up to one page of records from **the right source.**" Note that leftRecord should not be set to the first record in the right page.

#### Simple Nested Loop Join \(SNLJ\)

SNLJ has already been implemented for you in `SNLJOperator`. You should take a look at it to get a sense for how the pseudocode in lecture and section translate to code, but you should **not** copy it when writing your own join operators. Although each join algorithm should return the same data, the order differs between each join algorithm, as does the structure of the code. In particular, SNLJ does not need to explicitly manage pages of data \(it only ever needs the next record of each table, and therefore can just use an iterator over all records in a table\), whereas all the algorithms you will be implementing in this part must explicitly manage when pages of data are fetched from disk.

#### Page Nested Loop Join \(PNLJ\)

PNLJ has already been implemented for you as a special case of BNLJ with B=3. Therefore, it will not function properly until BNLJ has been properly implemented. The test cases for both PNLJ and BNLJ in `TestNestedLoopJoin` depend on a properly implemented BNLJ.

#### Block Nested Loop Join \(BNLJ\)

You should read through the given skeleton code in `BNLJOperator`. The `next` and `hasNext` methods of the iterator have already been filled out for you, but you will need to implement the `fetchNextRecord` method, which should do most of the heavy lifting of the BNLJ algorithm.

There are also two suggested helper methods: `fetchNextLeftBlock`, which should fetch the next non-empty block of left table pages from `leftIterator`, and `fetchNextRightPage`, which should fetch the next non-empty page of the right table \(from `rightIterator`\). We suggest breaking up the problem into smaller subproblems, and adding more helper methods than the two suggested ones -- it will make debugging your code much easier.

The `fetchNextRecord` method should, as its name suggests, fetches the next record of the join output. When implementing this method there are 4 important cases you should consider:

* Case 1: The right page iterator has a value to yield
* Case 2: The right page iterator doesn't have a value to yield but the left block iterator does
* Case 3: Neither the right page nor left block iterators have values to yield, but there's more right pages
* Case 4: Neither right page nor left block iterators have values nor are there more right pages, but there are still left blocks

We've provided the following animation to give you a feel for how the blocks, pages, and records are traversed during the nested looping process. Identifying where each of these cases take place in the diagram may help guide on what to do in each case.

![](../../../.gitbook/assets/bnlj-slower.gif)

Animations of SNLJ and PNLJ can be found [here](../../../common/misc/nested-loop-join-animations.md). Loaded left records are highlighted in blue, while loaded orange records are highlighted in orange. The dark purple square represents which pair of records are being considered for the join, while light purple shows which pairs have already been considered.

Once you have implemented `BNLJOperator`, all the tests in `TestNestedLoopJoin` should pass.

### Task 2: Hash Joins

#### Simple Hash Join \(SHJ\)

We've provided an implementation of Simple Hash Join which can be found in `SHJOperator.java`. Simple Hash Join performs a single pass of partitioning on only the left records before attempting to join. Read the code for SHJ carefully as it should give you a good idea of how to implement Grace Hash Join.

#### Grace Hash Join \(GHJ\)

Everything you will need to implement will be done in `GHJOperator.java`. You will need to implement the functions `partition`, `buildAndProbe`, and `run`. Additionally, you will have to provide some inputs in `getBreakSHJInputs` and `getBreakGHJInputs` which will be used to test that Simple Hash Join fails but Grace Hash Join passes \(tested in `testBreakSHJButPassGHJ`\) and that GHJ breaks \(tested in `testGHJBreak`\) respectively.

The file `Partition.java` in the `query/disk` directory will be useful when working with partitions. Read through the file and get a good idea what methods you can use.

Once you have implemented all the methods in `GHJOperator.java`, all tests in `TestGraceHashJoin.java` will pass. There will be **no hidden tests** for Grace Hash Join. Your grade for Grace Hash Join will come solely from the released public tests.

### Task 3: External Sort

The first step in Sort Merge Join is to sort both input relations. Therefore, before you can work on implementing Sort Merge Join, you must first implement an external sorting algorithm.

Recall that a "run" in the context of external mergesort is just a sequence of sorted records. This is represented in `SortOperator` by the `Run` class \(located in `query/disk/Run.java`\). As runs in external mergesort can span many pages \(and eventually span the entirety of the table\), the `Run` class does not keep all its data in memory. Rather, it creates a temporary table and writes all of its data to the temporary table \(which is materialized to disk at the buffer manager's discretion\).

You will need to implement the `sortRun`, `mergeSortedRuns`, `mergePass`, and `sort` methods of `SortOperator`.

* `sortRun(run)` should sort the passed in data using an in-memory sort \(Pass 0 of external mergesort\).
* `mergeSortedRuns(runs)` should return a new run given a list of sorted runs.
* `mergePass(runs)` should perform a single merge pass of external mergesort, given a list of all the sorted runs from the previous pass.
* `sort()` should run external mergesort from start to finish, and return the final run with the sorted data

Each of these methods may be tested independently, so you **must** implement each one as described. You may add additional helper methods as you see fit.

Once you have implemented all four methods, all the tests in `TestSortOperator` should pass.

### Task 4: Sort Merge Join

Now that you have a working external sort, you can now implement Sort Merge Join \(SMJ\).

For simplicity, your implementation of SMJ should _not_ utilize the optimization discussed in lecture in any case \(where the final merge pass of sorting happens at the same time as the join\). Therefore, you should use `SortOperator` to sort during the sort phase of SMJ.

You will need to implement the `SortMergeIterator` inner class of `SortMergeOperator`.

Your implementation in `SortMergeOperator` and your implementation of `SortOperator` may be tested independently. You **must not** use any method of `SortOperator` in `SortMergeOperator`, aside from the public methods given in the skeleton \(in other words: don't add a new public method to `SortOperator` and call it from `SortMergeOperator`\).

Once you have implemented `SortMergeIterator`, all the tests in `TestSortMergeJoin` should pass.

## Submission

Follow the submission instructions [here](../submitting-the-assignment.md) for the Project 3 Part 1 assignment on Gradescope. If you completed everything you should be passing all the tests in the following files:

* `database.query.TestNestedLoopJoin`
* `database.query.TestGraceHashJoin`
* `database.query.TestSortOperator`
* `database.query.TestSortMergeJoin`

