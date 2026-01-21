

# 1 Locking 


Enforcing Serializability
• Precedence graphs are not a concurrency control mechanism!
    • We need to know the schedule in advance
    • What about interactive systems?
• Instead a scheduler ensures serializability by delaying operations or aborting transactions
• Pessimistic concurrency control – Locking
• Optimistic concurrency control – Timestamps

![](image/Pasted%20image%2020250118203354.png)


## 1.1 Locking

• Enforce serializability by locking database elements
• Two types of locks
    • Read lock / sharable lock
    • Write lock / exclusive lock
    • Read lock can be upgraded to exclusive lock
• Locks are requested by a transaction
    • Maybe implicitly by the Scheduler
• Locks are managed/granted by the Lock Manager
    • Global in-memory data structure
    • Part of the Scheduler

![](image/Pasted%20image%2020250118203644.png)


# 2 sharable lock and exclusiv lock

- Read lock / sharable lock
    - 一个data 能有多个 sharable lock
    - more than 1 transaction  can  readit 
    - any can access it, but no one can write it 
- Write lock / exclusive lock
    - 一个data 能有一个 sharable lock
    - it can access it, and  can write it 



## 2.1 Transaction Schedules with Locks

• A TX can only read or write an element if
    • It was granted a lock for that element
    • It hasn’t released that lock yet
• If a TX holds a lock it must release it later
    • For example, implicitly by the scheduler at commit or abort
• Additional operations in a schedule
    • S(E) – Request shared lock for element E
    • X(E) – Request exclusive lock for element E
    • U(E) – Release (any) lock held on element E
• Schedule with locks:
    S1(A), R1(A), X1(B), W1(B), U1(A), U1(B)

```
T1: BEGIN
    
    R(A)
    R(B)
    if A == 0 then B = B + 1
    W(B)
    COMMIT

T2: BEGIN
    R(B)
    R(A)
    if B == 0 then A = A + 1
    W(A)
    COMMIT
```



```
T1: BEGIN
    这里添加 S(A)
    R(A)
    这里添加 S(B)
    R(B)
    if A == 0 then B = B + 1
    这里添加 X(B)
    W(B)
    COMMIT
    这里添加 U(A)
    这里添加 U(B)

同上添加   S(A),  S(B), X(B), U(A), U(B)
T2: BEGIN
    R(B)
    R(A)
    if B == 0 then A = A + 1
    W(A)
    COMMIT
```


## 2.2 strategy to create locks  (重要)
- two phase locking, first create read lock, then create exclusive lock.   
- never create exclusive lock directly without create read lock beforehand, because other transaction can read this date if you already create read lock once the write lock are released 
- every transaction hast to keep all locks until this transaction is finished 



## 2.3 Lock problem

Dead Locks
• Locking can lead to dead locks
    • T1: R1(A), W1(A)
    • T2: R2(A), W2(A)
![](image/Pasted%20image%2020250118203928.png)


Locking and Serializability
• Locking itself does not ensure serializability
![](image/Pasted%20image%2020250118203941.png)


## 2.4 Lock Compatibility

• Multiple TX can hold the locks on the same element
• Read locks are compatible with other read locks
    • Multiple TX can have a read lock on same element
• Write locks are not compatible with other locks
    • A read lock and write lock cannot exist on the same element at the same time
• NOTE: If a transaction requests a lock that cannot be granted, the operations is delayed (or the transaction is aborted)

![](image/Pasted%20image%2020250118204149.png)


## 2.5 Two-Phase Locking (2PL)

• Solution: 2PL guarantees serializability
• Before a TX can read/write element E, it must own a read/write lock for E
• Two TX cannot own incompatible locks
• Once a TX starts to release its locks, it cannot be granted new locks
    • Lock phase / Growing phase
    • Release phase / Shrinking phase
• All 2PL schedules are serializable
    • Proof omitted, see literature
• Dead locks can still happen

![](image/Pasted%20image%2020250118204359.png)


![](image/Pasted%20image%2020250118204419.png)


### 2.5.1 Delayed Operations by 2PL
• T1: R1(A), W1(A), R1(B), W1(B)
• T2: R2(A), W2(A), R2(B), W2(B)
• Order of requested operations: R1(A), W1(A), R2(A), W2(A), R2(B), W2(B), R1(B), W1(B)

![](image/Pasted%20image%2020250118204548.png)



---

### 2.5.2 Dead Locks with 2PL
Deadlocks are still possible

• Same example as before
• T1: R1(A), W1(A)
• T2: R2(A), W2(A)

![](image/Pasted%20image%2020250118204618.png)


---


### 2.5.3 Recoverable Schedules
• 2PL does not guarantee recoverable schedules
    • 2PL only requires lock all items before processing but it is not mandatory to commit
    • Recall: A schedule S is called recoverable, if, whenever a T1 reads an object X whose value was before written by a unfinished T2, then S contains a commit for T2 (at whatever place)
    • Ensures that if a transaction fails, no other transactions depend on its uncommitted changes.
    • When T2 starts, it may lock and write objects written by T1
    • Commit of T2 violates recoverable because T1 abort


![](image/Pasted%20image%2020250118204705.png)

---

### 2.5.4 Problem of Cascading Aborts

Cascading Abort:
• It does not lead to cascading rollbacks if a Tx fails
• Delay reading TxR of uncommitted data until writing TxW committed and thus TxR does not need to be rolled back

![](image/Pasted%20image%2020250118204812.png)


---


### 2.5.5 Strict 2PL

• Exclusive locks are released only after commit/abort is acknowledged by scheduler
• Ensures recoverable schedules
• But: less parallelization, less throughput
• Deadlocks may still happen but no cascading aborts
• In practice, locking-based systems implement Strict 2PL

![](image/Pasted%20image%2020250118204836.png)

![](image/Pasted%20image%2020250118204905.png)


## 2.6 Lock Granularities


• Lock database elements organized in a hierarchy to cope with different granularities, e.g., blocks, relations, tuples, etc.
1. To place S or X lock on any element, we must begin at the root
2. If we are at the element we want to lock, we request an S or X lock
3. Otherwise, if the element is below in the hierarchy, we request an IS or IX intention lock on the current element

![](image/Pasted%20image%2020250118205028.png)


## 2.7 Phantom Read Problem

• Occurs within a transaction when the same query produces different output at
different times.
• Example:
• TX 1: Every month we can distribute a total of 10,000 € to the employees of department X
    • Q1: T := SELECT COUNT(*) FROM Emp WHERE Dept = ‘X’
    • Q2: UPDATE Emp SET Salary = Salary + 10000 / T
• TX 2: Jane Doe starts working in department X
    • Q3: UPDATE Emp SET Dept = ‘X’ WHERE Name = ‘Jane Doe’
• If Q3 runs between Q1 and Q2, we have distributed too much money
    • TX 1 needs a write lock on the entire table
    • But TX 2 only needs a write lock on a single tuple
    • We can only lock existing items


## 2.8 Intention Locks Compatibility Matrix

• Intention locks IS and IX are compatible with each other
    • Allow conflict to be resolved at lower level
• If a TX wants to update a page P1, it first sets an IX lock on the table and then a X lock on P1
• If another TX wants to update a page P2, it also sets an IX lock on the table
    • The lock on P1 by TX1 does not concern TX2



![](image/Pasted%20image%2020250118205129.png)



# 3 Lock Manager


• In-memory DBMS-internal data structure
• Grants or blocks lock requests, deals with deadlocks
• Manages queues of blocked transactions
• Interface to transactions
    • acquireLock(T,X, mode)
    • releaseLock(T,X, mode)
• A table of logical lock data structures

• Logical lock
    • Lock mode (S, X, IS, IX, …)
    • Linked list of lock requests (granted or pending)
    • Latch (physical lock that protects data structure, more later)
    • Resource to lock


## 3.1 Locks and Latches
lock: logical things
latches: the implementation of logical things the reale Opearation 



![](image/Pasted%20image%2020250118205256.png)



## 3.2 Acquiring Locks

• Transaction attempts to acquire lock
• Follow hierarchical locking protocol to ensure that transaction holds intention locks at higher level
    • Resource hierarchy is fixed and hard-coded in data structures — e.g., a row knows its pageId
    • Recursively issue higher-level lock requests if needed
    • If transaction already holds a coarser-grained lock, grant request immediately
    • Otherwise, probe hash table to find lock

• Latch the lock and append request to queue
    • Transaction may block if request incompatible with current lock mode

## 3.3 Releasing Locks

• Transactions maintain pointers to held logical locks in request order
• Locks are released one by one in request order at end of transaction

• To release lock:
    • Latch lock and unlink corresponding request
    • Traverse request list to discover new lock mode and pending requests that may now be granted
    • Unlatch lock
    • Blocked transactions are notified and can proceed


## 3.4 Lock Manager Performance

Lock manager is a “hot spot” for contention, especially for locks high in the hierarchy or popular items
• E.g., insertions and searches at the “most recent” part of a time-ordered table


![](image/Pasted%20image%2020250118205450.png)

## 3.5 Optimizations

• Speculative lock inheritance
    • Transaction tries to inherit high-level locks from the previous transaction that executed in the same thread (Paper: Improving OLTP Scalability using Speculative Lock Inheritance)
• Data-oriented execution
    • Thread-per-data partition rather than thread-per-transaction (Paper: Data- Oriented Transaction Execution)
• Lightweight intent lock
    • Private lock table per transaction simplifies code paths for requesting and releasing locks from global lock table (Paper: Efficient Locking Techniques for Databases on Modern Hardware)
• Early lock release
    • It takes 0.01ms to run a transaction if all data is found in buffer pool
    • It takes 10ms to force the commit record to a hard disk (=1000x !!!)
    • Allow transactions to release their locks as soon as they receive allocated space in buffer pool for commit log record

## 3.6 Dealing with Deadlocks

• Deadlock prevention:
• Using timeouts: Abort transaction if it is waiting too long
• May result in false positives. How to pick timeout parameter?
• Deadlock detection:
    • T1 ➜ T2 if T1 is blocked waiting for T2 to release a lock
    • Cycles mean deadlocks
    • Roll back transaction if waits-for graph contains cycle (rollback releases locks automatically)
    • High computational overhead: Check status of all transactions and probe lock queues

![](image/Pasted%20image%2020250118205622.png)


## 3.7 practical 例子 

python 的语法
db.x("transactionName", "databaseName")

在sql 语句中, 执行他的时候, lock 是自动生成的, 不用特意去屑 

---

In the following tasks we used a simple database with a lock manager. Given two database objects, A and B, and two transactions T0 and T1.
1. Design a 2PL-compliant schedule.
2. Design an illegal schedule.
3. Design a schedule with two transactions that leads to a deadlock.

Insert necessary locks to correct locations.

```python
from resources.scripts.lockmanager import *

db = Datenbank()

db.write("T0", "A")

db.read("T1", "B")

del db
restartkernel()
```

Error: Transaction T0 does not have a write lock on object A!
Error: Transaction T1 does not have a lock on object B!
Success: All locks have been released!

--

```python
# Copyright (c) 2020 Clemens Lutz, German Research Center for Artificial Intelligence
# Author: Clemens Lutz <clemens.lutz@dfki.de>
#
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#     * Redistributions of source code must retain the above copyright
#       notice, this list of conditions and the following disclaimer.
#     * Redistributions in binary form must reproduce the above copyright
#       notice, this list of conditions and the following disclaimer in the
#       documentation and/or other materials provided with the distribution.
#     * Neither the name of the <organization> nor the
#       names of its contributors may be used to endorse or promote products
#       derived from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
# ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
# WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
# DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
# (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
# LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
# ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
# SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

from IPython.display import display_html


def restartkernel():
    display_html("<script>Jupyter.notebook.kernel.restart()</script>", raw=True)


class Datenbank:
    shared_locks = dict()
    exclusive_locks = dict()

    def read(self, transaktion, objekt):
        status = False
        if (
            objekt in self.shared_locks
            and transaktion in self.shared_locks[objekt]
            or objekt in self.exclusive_locks
            and self.exclusive_locks[objekt] == transaktion
        ):
            print(f"Success: Transaction {transaktion} has read object {objekt}!")
            status = True
        else:
            print(
                f"Error: Transaction {transaktion} does not have a lock on object {objekt}!"
            )
            status = False

        return status

    def write(self, transaktion, objekt):
        status = False
        if (
            objekt in self.exclusive_locks
            and self.exclusive_locks[objekt] == transaktion
        ):
            print(f"Success: Transaction {transaktion} has written object {objekt}!")
            status = True
        else:
            print(
                f"Error: Transaction {transaktion} does not have a write lock on object {objekt}!"
            )
            status = False

        return status

    def sl(self, transaktion, objekt):
        status = False
        if objekt in self.exclusive_locks:
            status = False
        elif objekt in self.shared_locks:
            self.shared_locks[objekt].add(transaktion)
            status = True
        else:
            self.shared_locks[objekt] = set([transaktion])
            status = True

        if status:
            print(
                f"Success: Transaction {transaktion} has shared lock on object {objekt}!"
            )
        else:
            print(
                f"Error: Transaction {transaktion} could not acquire shared lock on object {objekt}!"
            )

        return status

    def xl(self, transaktion, objekt):
        status = False
        if objekt in self.exclusive_locks:
            status = False
        elif objekt in self.shared_locks:
            status = False
        else:
            self.exclusive_locks[objekt] = transaktion
            status = True

        if status:
            print(
                f"Success: Transaction {transaktion} has exclusive lock on object {objekt}!"
            )
        else:
            print(
                f"Error: Transaction {transaktion} could not acquire exclusive lock on object {objekt}!"
            )

        return status

    def ul(self, transaktion, objekt):
        status = False
        success = f"Success: Transaction {transaktion} has unlocked object {objekt}!"
        failure = f"Error: Object {objekt} was not locked!"

        try:
            if objekt in self.exclusive_locks:
                if self.exclusive_locks[objekt] != transaktion:
                    raise KeyError
                else:
                    del self.exclusive_locks[objekt]
                    print(success)
                    status = True
            elif objekt in self.shared_locks:
                if (
                    transaktion in self.shared_locks[objekt]
                    and len(self.shared_locks[objekt]) == 1
                ):
                    del self.shared_locks[objekt]
                else:
                    self.shared_locks[objekt].remove(transaktion)

                print(success)
                status = True
            else:
                print(failure)
                status = False
        except KeyError:
            print(
                f"Error: Transaction {transaktion} does not have a lock on object {objekt}!"
            )
            status = False

        return status

    def __init__(self):
        return

    def __del__(self):
        if len(self.shared_locks) != 0 or len(self.exclusive_locks) != 0:
            print("Error: Not all locks have been released!")
        else:
            print("Success: All locks have been released!")
        return
```
