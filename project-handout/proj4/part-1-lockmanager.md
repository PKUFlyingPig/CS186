# Part 1: Queuing

![Datarace](../../.gitbook/assets/datarace%20%281%29%20%281%29%20%282%29%20%282%29%20%283%29%20%283%29.png)

In this part you will implement some helpers functions for lock types and the queuing system for locks. The `concurrency` directory contains a partial implementation of a lock manager \(`LockManager.java`\), which you will be completing.

Note on terminology: in this project, "children" and "parent" refer to the resource\(s\) directly below/above a resource in the hierarchy. "Descendants" and "ancestors" are used when we wish to refer to all resource\(s\) below/above in the hierarchy.

## Task 1: LockType

Before you start implementing the queuing logic, you need to keep track of all the lock types supported, and how they interact with each other. The [`LockType`](https://github.com/berkeley-cs186/sp21-rookiedb/blob/master/src/main/java/edu/berkeley/cs186/database/concurrency/LockType.java) class contains methods reasoning about this, which will come in handy in the rest of the project.

For the purposes of this project, a transaction with:

* `S(A)` can read A and all descendants of A.
* `X(A)` can read and write A and all descendants of A.
* `IS(A)` can request shared and intent-shared locks on all children of A.
* `IX(A)` can request any lock on all children of A.
* `SIX(A)` can do anything that having `S(A)` or `IX(A)` lets it do, except requesting S, IS, or SIX\*\* locks on children of A, which would be redundant.

\*\* This differs from how its presented in lecture, where SIX\(A\) allows a transaction to request SIX locks on children of A. We disallow this in the project since the S aspect of the SIX child would be redundant.

You will need to implement the `compatible`, `canBeParentLock`, and `substitutable` methods:

* [`compatible(A, B)`](https://github.com/berkeley-cs186/sp21-rookiedb/blob/master/src/main/java/edu/berkeley/cs186/database/concurrency/LockType.java#L14) checks if lock type A is compatible with lock type B -- can one transaction have lock A while another transaction has lock B on the same resource? For example, two transactions can have S locks on the same resource, so `compatible(S, S) = true`, but two transactions cannot have X locks on the same resource, so `compatible(X, X) = false`.
* [`canBeParentLock(A, B)`](https://github.com/berkeley-cs186/sp21-rookiedb/blob/master/src/main/java/edu/berkeley/cs186/database/concurrency/LockType.java#L48) returns true if having A on a resource lets a transaction acquire a lock of type B on a child. For example, in order to get an S lock on a table, we must have \(at the very least\) an IS lock on the parent of table: the database. So `canBeParentLock(IS, S) = true`.
* [`substitutable(substitute, required)`](https://github.com/berkeley-cs186/sp21-rookiedb/blob/master/src/main/java/edu/berkeley/cs186/database/concurrency/LockType.java#L61) checks if one lock type \(`substitute`\) can be used in place of another \(`required`\). This is only the case if a transaction having `substitute` can do everything that a transaction having `required` can do. Another way of looking at this is: let a transaction request the required lock. Can there be any problems if we secretly give it the substitute lock instead? For example, if a transaction requested an X lock, and we quietly gave it an S lock, there would be problems if the transaction tries to write to the resource. Therefore, `substitutable(S, X) = false`.

Once you complete this task you should be passing all the tests in [`TestLockType.java`](https://github.com/berkeley-cs186/sp21-rookiedb/blob/master/src/test/java/edu/berkeley/cs186/database/concurrency/TestLockType.java). This semester we've provided a [Gradescope assignment](https://www.gradescope.com/courses/225521/assignments/1097657) that you can complete to check your logic for the cases that aren't given.

## Task 2: LockManager

**Note \(Spring 2021\):** To keep workload balanced with the upcoming midterm, your Part 1 submission will only need to pass the following tests from this task for full credit:

* TestLockManager.testSimpleAcquireLock
* TestLockManager.testSimpleAcquireLockFail
* TestLockManager.testSimpleReleaseLock
* TestLockManager.testReleaseUnheldLock
* TestLockManager.testSimpleConflict

These tests have been selected to overlap with the material you'll be studying for the midterm. The remaining tests will be tested in the Part 2 submission.

The [`LockManager`](https://github.com/berkeley-cs186/sp21-rookiedb/blob/master/src/main/java/edu/berkeley/cs186/database/concurrency/LockType.java) class handles locking for individual resources. We will add multigranularity constraints in Part 2.

**Before you start coding**, you should understand what the lock manager does, what each method of the lock manager is responsible for, and how the internal state of the lock manager changes with each operation. The different methods of the lock manager are not independent: trying to implement one without consideration of the others will cause you to spend significantly more time on this project. There is a fair amount of logic shared between methods, and it may be worth spending a bit of time writing some helper methods.

A simple example of a blocking acquire call is described at the bottom of this section -- you should understand it and be able to describe any other combination of calls before implementing any method.

You will need to implement the following methods of `LockManager`:

* [`acquireAndRelease`](https://github.com/berkeley-cs186/sp21-rookiedb/blob/master/src/main/java/edu/berkeley/cs186/database/concurrency/LockManager.java#L156): this method atomically \(from the user's perspective\) acquires one lock and releases zero or more locks. This method has priority over any queued requests \(it should proceed even if there is a queue, and it is placed in the front of the queue if it cannot proceed\).
* [`acquire`](https://github.com/berkeley-cs186/sp21-rookiedb/blob/master/src/main/java/edu/berkeley/cs186/database/concurrency/LockManager.java#L183): this method is the standard `acquire` method of a lock manager. It allows a transaction to request one lock, and grants the request if there is no queue and the request is compatible with existing locks. Otherwise, it should queue the request \(at the back\) and block the transaction. We do not allow implicit lock upgrades, so requesting an X lock on a resource the transaction already has an S lock on is invalid.
* [`release`](https://github.com/berkeley-cs186/sp21-rookiedb/blob/master/src/main/java/edu/berkeley/cs186/database/concurrency/LockManager.java#L208): this method is the standard `release` method of a lock manager. It allows a transaction to release one lock that it holds.
* [`promote`](https://github.com/berkeley-cs186/sp21-rookiedb/blob/master/src/main/java/edu/berkeley/cs186/database/concurrency/LockManager.java#L238): this method allows a transaction to explicitly promote/upgrade a held lock. The lock the transaction holds on a resource is replaced with a stronger lock on the same resource. This method has priority over any queued requests \(it should proceed even if there is a queue, and it is placed in the front of the queue if it cannot proceed\). We do not allow promotions to SIX, those types of requests should go to `acquireAndRelease`. This is because during SIX lock upgrades, it is possible we might need to also release redundant locks, so we need to handle these upgrades with `acquireAndRelease`.
* [`getLockType`](https://github.com/berkeley-cs186/sp21-rookiedb/blob/master/src/main/java/edu/berkeley/cs186/database/concurrency/LockManager.java#L256): this is the main way to query the lock manager, and returns the type of lock that a transaction has on a specific resource., which was implemented in the previous step.

### Queues

Whenever a request for a lock cannot be satisfied \(either because it conflicts with locks other transactions already have on the resource, or because there's a queue of requests for locks on the resource and the operation does not have priority over the queue\), it should be placed on the queue \(at the back, unless otherwise specified\) for the resource, and the transaction making the request should be blocked.

The queue for each resource is processed independently of other queues, and must be processed after a lock on the resource is released, in the following manner:

* The request at the front of the queue is considered, and if it doesn't conflict with any of the existing locks on the resource, it should be removed from the queue and:
  * the transaction that made the request should be given the lock
  * any locks that the request stated should be released are released
  * the transaction that made the request should be unblocked
* The previous step should be repeated until the first request on the queue cannot be satisfied or the queue is empty.

### **Synchronization**

`LockManager`'s methods have `synchronized` blocks to ensure that calls to `LockManager` are serial and that there is no interleaving of calls. You may want to read up on [synchronized methods](https://docs.oracle.com/javase/tutorial/essential/concurrency/syncmeth.html) and [synchronized statements](https://docs.oracle.com/javase/tutorial/essential/concurrency/locksync.html) in Java. You should make sure that all accesses \(both queries and modifications\) to lock manager state in a method is inside **one** synchronized block, for example:

```java
// Correct, use a single synchronized block
void acquire(...) {
    synchronized (this) {
        ResourceEntry entry = getResourceEntry(name); // fetch resource entry
        // do stuff
        entry.locks.add(...); // add to list of locks
    }
}

// Incorrect, multiple synchronized blocks
void acquire(...) {
    synchronized (this) {
        ResourceEntry entry = getResourceEntry(name); // fetch resource entry
    }
    // first synchronized block ended: another call to LockManager can start here
    synchronized (this) {
        // do stuff
        entry.locks.add(...); // add to list of locks
    }
}

// Incorrect, doing work outside of the synchronized block
void acquire(...) {
    ResourceEntry entry = getResourceEntry(name); // fetch resource entry
    // do stuff
    // other calls can run while the above code runs, which means we could
    // be using outdated lock manager state
    synchronized (this) {
        entry.locks.add(...); // add to list of locks
    }
}
```

Transactions block the entire thread when blocked, which means that you cannot block the transaction inside the `synchronized` block \(this would prevent any other call to `LockManager` from running until the transaction is unblocked... which is never, since the `LockManager` is the one that unblocks the transaction\).

To block a transaction, call `Transaction#prepareBlock` **inside** the synchronized block, and then call `Transaction#block` **outside** the synchronized block. The `Transaction#prepareBlock` needs to be in the synchronized block to avoid a race condition where the transaction may be dequeued between the time it leaves the synchronized block and the time it actually blocks.

If tests in `TestLockManager` are timing out, double-check that you are calling `prepareBlock` and `block` in the manner described above, and that you are not calling `prepareBlock` without `block`. \(It could also just be a regular infinite loop, but checking that you're handling synchronization correctly is a good place to start\).

**Example**

Consider the following calls \(this is what `testSimpleConflict` tests\):

```java
// initialized elsewhere, T1 has transaction number 1,
// T2 has transaction number 2
Transaction t1, t2;

LockManager lockman = new LockManager();
ResourceName db = new ResourceName("database");

lockman.acquire(t1, db, LockType.X); // t1 requests X(db)
lockman.acquire(t2, db, LockType.X); // t2 requests X(db)
lockman.release(t1, db); // t1 releases X(db)
```

In the first call, T1 requests an X lock on the database. There are no other locks on database, so we grant T1 the lock. We add X\(db\) to the list of locks T1 has \(in `transactionLocks`\), as well as to the locks held on the database \(in `resourceLocks`\). Our internal state now looks like:

```text
transactionLocks: { 1 => [ X(db) ] } (transaction 1 has 1 lock: X(db))
resourceEntries: { db => { locks: [ {1, X(db)} ], queue: [] } }
    (there is 1 lock on db: an X lock by transaction 1, nothing on the queue)
```

In the second call, T2 requests an X lock on the database. T1 already has an X lock on database, so T2 is not granted the lock. We add T2's request to the queue, and block T2. Our internal state now looks like:

```text
transactionLocks: {1 => [X(db)]} (transaction 1 has 1 lock: X(db))
resourceEntries:  {db => {locks: [{1, X(db)}], queue: [LockRequest(T2, X(db))]}}
    (there is 1 lock on db: an X lock by transaction 1, and 1 request on
     queue: a request for X by transaction 2)
```

In the last call, T1 releases an X lock on the database. T2's request can now be processed, so we remove T2 from the queue, grant it the lock by updating `transactionLocks` and `resourceLocks`, and unblock it. Our internal state now looks like:

```text
transactionLocks: { 2 => [ X(db) ] } (transaction 2 has 1 lock: X(db))
resourceEntries: { db => { locks: [ {2, X(db)} ], queue: [] } }
    (there is 1 lock on db: an X lock by transaction 2, nothing on the queue)
```

## Submission

After this, you should pass all the tests we have provided to you in [`database.concurrency.TestLockType`](https://github.com/berkeley-cs186/sp21-rookiedb/blob/master/src/test/java/edu/berkeley/cs186/database/concurrency/TestLockType.java) and [`database.concurrency.TestLockManager`](https://github.com/berkeley-cs186/sp21-rookiedb/blob/master/src/test/java/edu/berkeley/cs186/database/concurrency/TestLockUtil.java). Remember that for the Part 1 submission you only need to pass the first 5 tests of TestLockManager for full credit.

Note that you may **not** modify the signature of any methods or classes that we provide to you, but you're free to add helper methods. Also, you should only modify code in the `concurrency` directory for this part.

