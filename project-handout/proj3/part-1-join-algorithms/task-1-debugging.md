# Task 1 Debugging

**Update \(03/05/2021\):** If you received your copy of the skeleton code before March 6th, then the [following line](https://github.com/berkeley-cs186/sp21-rookiedb/commit/0f9805a8c7e3417f9dcd4eba873c9b6d51af7b03) in ExtraNLJTests should be changed. This doesn't effect the way the tests run, but will make print debugging less confusing.

We put together some extra tests with detailed error outputs that should give you some hints as to what might be go wrong with your BNLJ implementation. They're meant to be easier to reason about than the main BNLJ tests since each page only has 4 records instead of 400. **These tests are ungraded**. They're just meant to help you track down bugs in the nested loop join tests in `TestNestedLoopJoin`.

## Overview

These tests are designed to give you visualizations that might hint as to where you're going wrong. **You should try to get the test cases working in order**, that is, start with the 1x1 PNLJ tests, followed by the 2x2 PNLJ tests, and then finally the 2x2 BNLJ tests. When you fail a test it should give you a detailed description of why you failed. Here's some example output from failing `testPNLJ1x1Full`:

```text
edu.berkeley.cs186.database.query.QueryPlanException:
== MISSING OR EXTRA RECORDS ==
         +---------+
 Left  0 | ? ? ? ? |
 Page  0 | x x x x |
 #1    0 | x x x x |
       0 | x x x x |
         +---------+
           0 0 0 0
           Right
           Page #1

You either excluded or included records when you shouldn't have. Key:
 - x means we expected this record to be included and you included it
 - + means we expected this record to be excluded and you included it
 - ? means we expected this record to be included and you excluded it
 - r means you included this record multiple times
 - a blank means we expected this record to be excluded and you excluded it
```

In this example we expect every single record in the left table to be joined with every single table in the right table. The question marks on the top row of the box tell you that you're missing 4 records. A likely reason for why this is the case is that your join logic exits too early, before the last left record is ever compared against the right records. The exact cause of this particular problem is stopping iteration as soon as `!this.leftRecordIterator.hasNext()`, before considering the last left record against any right records.

Here's a more complicated case that we see in office hours a lot in testPNLJ2x2Full :

```text
edu.berkeley.cs186.database.query.QueryPlanException:
== MISMATCH ==
         +---------+---------+
 Left  0 |         |         |
 Page  0 |         |         |
 #2    0 |         |         |
       0 |         |         |
         +---------+---------+
 Left  0 | x x x x | A       |
 Page  0 | x x x x |         |
 #1    0 | x x x x |         |
       0 | x x x x | E       |
         +---------+---------+
           0 0 0 0    0 0 0 0
           Right      Right
           Page #1    Page #2

You had 1 or more mismatched records. The first mismatch
was at record #17. The above shows the state of
the join when the mismatch occurred. Key:
 - x means your join properly yielded this record at the right time
 - E was the record we expected you to yield
 - A was the record that you actually yielded
```

This example found a record returned in the wrong order. To help you debug we give the position of where we expected the next record to be, and where it actually was. Can you spot the bug? We were expecting the first record on right page \#2 to be compared with the first record in left page \#1. It appears that leftRecord was still set to the last record on page \#1. The mistake was that the leftRecord wasn't reset back to the first record in the left page. Many students will remember to call `leftIterator.reset()`, but forget to do `leftRecord = leftIterator.next()` afterwards, causing this issue.

## Animations

Here's some animations of how we expect each test format to be traversed.

### PNLJ 1x1

![](../../../.gitbook/assets/1x1%20%283%29%20%284%29.gif)

### PNLJ 2x2

![](../../../.gitbook/assets/2x2pnlj%20%281%29%20%281%29.gif)

### BNLJ 2x2 \(B=4\)

![](../../../.gitbook/assets/2x2bnlj%20%284%29%20%284%29%20%283%29.gif)

## Cases

Here's examples of the cases mentioned in the spec look like in the PNLJ 2x2 cases \(block size of 1\). The dark purple square is the most recently considered record. The red arrow points to the next pair records that should be considered for the join.

![](../../../.gitbook/assets/cases%20%281%29.png)

Try to think about what should be advanced and what should be reset in each case. As a reminder:

* Case 1: The right page iterator has a value to yield
* Case 2: The right page iterator doesn't have a value to yield but the left block iterator does
* Case 3: Neither the right page nor left block iterators have values to yield, but there's more right pages
* Case 4: Neither right page nor left block iterators have values nor are there more right pages, but there are still left blocks

## Common Errors

### PNLJ 1x1 Full

```text
== MISSING OR EXTRA RECORDS ==
         +---------+
 Left  0 | x ? x ? |
 Page  0 | x ? x ? |
 #1    0 | x ? x ? |
       0 | x ? x ? |
         +---------+
           0 0 0 0
           Right
           Page #1

You either excluded or included records when you shouldn't have. Key:
 - x means we expected this record to be included and you included it
 - + means we expected this record to be excluded and you included it
 - ? means we expected this record to be included and you excluded it
 - r means you included this record multiple times
 - a blank means we expected this record to be excluded and you excluded it
```

The above case is likely happening because you're calling `rightRecordIterator.next()` more often than you should, and losing every other value. Make sure whenever you call `rightRecordIterator.next()` that you compare the result to the current left record and set it as the next record if there's a match.

```text
== MISMATCH ==
         +---------+
 Left  0 |         |
 Page  0 |         |
 #1    0 | E       |
       0 | A x x x |
         +---------+
           0 0 0 0
           Right
           Page #1

You had 1 or more mismatched records. The first mismatch
was at record #5. The above shows the state of
the join when the mismatch occurred. Key:
 - x means your join properly yielded this record at the right time
 - E was the record we expected you to yield
 - A was the record that you actually yielded
```

The above case is mostly likely caused by failing to advance the left record in case 2. Remember that even if you call `leftRecordIterator.next()`, if you don't set the result to leftRecord then leftRecord won't get updated.

```text
edu.berkeley.cs186.database.query.QueryPlanException:
== MISSING OR EXTRA RECORDS ==
         +---------+
 Left  0 | ? ? ? ? |
 Page  0 | x x x x |
 #1    0 | x x x x |
       0 | x x x x |
         +---------+
           0 0 0 0
           Right
           Page #1

You either excluded or included records when you shouldn't have. Key:
 - x means we expected this record to be included and you included it
 - + means we expected this record to be excluded and you included it
 - ? means we expected this record to be included and you excluded it
 - r means you included this record multiple times
 - a blank means we expected this record to be excluded and you excluded it
```

The above case likely caused by stopping iteration too early, specifically as soon as !leftRecordIterator.hasNext\(\). Remember that even if there isn't another left record, you still have to compare the current left record against every right record in the rightRecordIterator.

```text
edu.berkeley.cs186.database.query.QueryPlanException:
== MISMATCH ==
         +---------+
 Left  0 |         |
 Page  0 |         |
 #1    0 | A       |
       0 | x x x E |
         +---------+
           0 0 0 0
           Right
           Page #1

You had 1 or more mismatched records. The first mismatch
was at record #4. The above shows the state of
the join when the mismatch occurred. Key:
 - x means your join properly yielded this record at the right time
 - E was the record we expected you to yield
 - A was the record that you actually yielded

== MISSING OR EXTRA RECORDS ==
         +---------+
 Left  0 | x x x ? |
 Page  0 | x x x ? |
 #1    0 | x x x ? |
       0 | x x x ? |
         +---------+
           0 0 0 0
           Right
           Page #1

You either excluded or included records when you shouldn't have. Key:
 - x means we expected this record to be included and you included it
 - + means we expected this record to be excluded and you included it
 - ? means we expected this record to be included and you excluded it
 - r means you included this record multiple times
 - a blank means we expected this record to be excluded and you excluded it
```

In the above case you're probably handling case 2 too early, before you ever compare the last right record to the current left record. Make sure that when you handle case 2 that you've already handled case 1 for the the last right record.

### PNLJ 2x2 Full

```text
== MISMATCH ==
         +---------+---------+
 Left  0 |         |         |
 Page  0 |         |         |
 #2    0 |         |         |
       0 |         |         |
         +---------+---------+
 Left  0 | x x x x | A       |
 Page  0 | x x x x |         |
 #1    0 | x x x x |         |
       0 | x x x x | E       |
         +---------+---------+
           0 0 0 0    0 0 0 0
           Right      Right
           Page #1    Page #2

You had 1 or more mismatched records. The first mismatch
was at record #17. The above shows the state of
the join when the mismatch occurred. Key:
 - x means your join properly yielded this record at the right time
 - E was the record we expected you to yield
 - A was the record that you actually yielded
```

In the above case you're probably not handling case 3 properly. In particular, make sure that when you run out of both left records and right records for a given left block and right page respectively that you call `leftIterator.reset()` AND assign `leftRecord` to the first record of the current page. Many students forget to reassign left record.

```text
         +---------+---------+
 Left  0 |         |         |
 Page  0 |         |         |
 #2    0 |         | A       |
       0 | E       |         |
         +---------+---------+
 Left  0 | x x x x | x x x x |
 Page  0 | x x x x | x x x x |
 #1    0 | x x x x | x x x x |
       0 | x x x x | x x x x |
         +---------+---------+
           0 0 0 0    0 0 0 0
           Right      Right
           Page #1    Page #2

You had 1 or more mismatched records. The first mismatch
was at record #33. The above shows the state of
the join when the mismatch occurred. Key:
 - x means your join properly yielded this record at the right time
 - E was the record we expected you to yield
 - A was the record that you actually yielded
```

In the above case make sure that by the end of case 4 you've set rightRecordIterator to be an iterator over right page \#1.

```text
edu.berkeley.cs186.database.query.QueryPlanException:
== MISMATCH ==
         +---------+---------+
 Left  0 |         |         |
 Page  0 |         |         |
 #2    0 |         |         |
       0 | E       |         |
         +---------+---------+
 Left  0 | A x x x | x x x x |
 Page  0 | x x x x | x x x x |
 #1    0 | x x x x | x x x x |
       0 | x x x x | x x x x |
         +---------+---------+
           0 0 0 0    0 0 0 0
           Right      Right
           Page #1    Page #2

You had 1 or more mismatched records. The first mismatch
was at record #33. The above shows the state of
the join when the mismatch occurred. Key:
 - x means your join properly yielded this record at the right time
 - E was the record we expected you to yield
 - A was the record that you actually yielded
```

In the above case make sure that by the end of case 4 you've set leftRecordIterator to be an iterator over left page \#2.

```text
== MISSING OR EXTRA RECORDS ==
         +---------+---------+
 Left  0 | ? ? ? ? | ? ? ? ? |
 Page  0 | ? ? ? ? | ? ? ? ? |
 #2    0 | ? ? ? ? | ? ? ? ? |
       0 | ? ? ? ? | ? ? ? ? |
         +---------+---------+
 Left  0 | x x x x | x x x x |
 Page  0 | x x x x | x x x x |
 #1    0 | x x x x | x x x x |
       0 | x x x x | x x x x |
         +---------+---------+
           0 0 0 0    0 0 0 0
           Right      Right
           Page #1    Page #2

You either excluded or included records when you shouldn't have. Key:
 - x means we expected this record to be included and you included it
 - + means we expected this record to be excluded and you included it
 - ? means we expected this record to be included and you excluded it
 - r means you included this record multiple times
 - a blank means we expected this record to be excluded and you excluded it
```

In the above case you're probably doing something wrong in case 4. In particular make sure that your code resets your right record iterator to be an iterator over the first page of the right relation. Remember that you'll need to reset your `rightIterator` to do this!

```text
== MISSING OR EXTRA RECORDS ==
         +---------+---------+
 Left  0 | ? ? ? ? | ? ? ? ? |
 Page  0 | ? ? ? ? | ? ? ? ? |
 #2    0 | ? ? ? ? | ? ? ? ? |
       0 | ? ? ? ? | ? ? ? ? |
         +---------+---------+
 Left  0 | x x x x | ? ? ? ? |
 Page  0 | x x x x | ? ? ? ? |
 #1    0 | x x x x | ? ? ? ? |
       0 | x x x x | ? ? ? ? |
         +---------+---------+
           0 0 0 0    0 0 0 0
           Right      Right
           Page #1    Page #2

You either excluded or included records when you shouldn't have. Key:
 - x means we expected this record to be included and you included it
 - + means we expected this record to be excluded and you included it
 - ? means we expected this record to be included and you excluded it
 - r means you included this record multiple times
 - a blank means we expected this record to be excluded and you excluded it
```

In the above case you likely have a problem in your implementation of case 3, and your code is terminating too early. One possible cause of this is forgetting to mark the beginning of your leftRecordIterator in fetchNextLeftBlock. This could cause problems when you try to reset leftRecordIterator in your case 3, and causing you to throw a no such element exception earlier than you intend to.

