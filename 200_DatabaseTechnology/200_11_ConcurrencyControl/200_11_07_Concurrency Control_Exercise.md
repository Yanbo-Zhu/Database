# 1 ACID principles 

## 1.1 What are ACID principles and what do they stand for?

Solution

- **Atomicity:** Either all operations of a transaction complete or none of them complete (“all or nothing”)
    
- **Consistency:** A transaction that is applied to a consistent database produces a consistent database
    
- **Isolation:** A transaction executes as if it is the only transaction running in the system
    
- **Durability:** The effects of committed transactions are reflected in the database even after failures
    

## 1.2 Which of the ACID principles are directly related to Concurrency Control?
    
Solution

Isolation and Consistency

“How can we isolate transactions from each other, and ensure consistent database state?”

# 2 Isolation Levels


## 2.1 What are the typical concurrency problems?

- Lost Update
- Dirty Read
- Non-repeatable Read
- Phantom Read

difference between last two:
- A non-repeatable read occurs, when during the course of a transaction, a row is retrieved twice and the values within the row differ between reads.
- A phantom read occurs when, in the course of a transaction, two identical queries are executed, and the collection of rows returned by the second query is different from the first.

Simple example:

User A runs the same query twice. In between, User B runs a transaction and commits.
- Non-repeatable read: The x row that user A has queried has a different value the second time.
- Phantom read: All the rows in the query have the same value before and after, but different rows are being selected (because B has deleted or inserted some). Example: select sum(x) from table; will return a different result even if none of the affected rows themselves have been updated, if rows have been added or deleted.


![](image/Pasted%20image%2020260124094759.png)

![](image/Pasted%20image%2020260124094810.png)


![](image/Pasted%20image%2020260124094820.png)


## 2.2 Why do we need different Isolation Levels? Why not using the highest level if it is the safest?

Database isolation levels are important because they determine the degree of consistency and correctness in a multi-user database system. The isolation level to choose is dependent on the requirements of the application. Highest level of isolation guarantees that no concurrency problem is going to occur, but then performance becomes the bottleneck.



## 2.3 What are the Isolation Levels from the lecture?


- READ UNCOMMITTED
    
- READ COMMITTED
    
- REPEATABLE READ
    
- SERIALIZABLE


## 2.4 Which isolation levels protect from which concurrency problems?

- READ UNCOMMITED - {Lost Update}
    
- READ COMMITED - {Lost Update, Dirty Read}
    
- REPEATABLE READ - {Lost Update, Dirty Read, Non-repeatable Read}
    
- SERIALIZABLE - {All above + Phantom Read}


## 2.5 Given the following example, which Isolation Level should be used?


We need SERIALIZABLE, as tuples should not be deleted (Phantom Read) and we are reading same tuples repeatedly (Non-repeatable Read).

---
example 
We have a long-running transaction (5-7 mins), that repeatedly reads a certain set of tuples from a table during the execution. There is an assumption that tuples to be read remain same during the execution, and should not be deleted until the transaction commits. Performance is not a big deal for this scenario, since we already assume that the transaction execution is going to take so long.

我们有一个长时间运行的事务（5-7分钟），它在执行期间会反复读取表中的某组元组。我们假设要读取的元组在执行期间保持不变，并且在该事务提交前不应被删除。性能对于这个场景来说不是大问题，因为我们已经假设事务执行时间会很长。

 **场景特点分析：**
1. **长事务**：运行时间 5-7 分钟
2. **重复读取**：在事务执行期间反复读取同一组元组
3. **数据稳定性要求**：读取的元组在事务期间不能改变，也不能被删除
4. **性能不是首要考虑**：因为事务本身就很长

**需要解决的核心问题：**
- **读一致性**：确保事务期间读取的数据不变
- **防删除保护**：防止其他事务删除这些元组
- **避免幻读**：确保读取的元组集合不变


# 3 Serializability

## 3.1 Define serial, interleaved, and serializable schedules.

Solution
- A schedule is _serial_ if it only executes all the operations from one transaction before moving to another transaction. or A schedule is _serial_ if operations from different TX are not interleaved.
- A schedule is _interleaved_ if operations of the multiple transactions execute in a non-sequential manner.
- A schedule for a set of TX T is called _serializable_ if its result is equal to the result of a serial schedule of T.

## 3.2 Draw a _serial schedule_, an _interleaved schedule_ and a _serializable schedule_.

Schedule 1: Serial


![](image/Pasted%20image%2020260124094443.png)

Schedule 2: Interleaved

![](image/Pasted%20image%2020260124094458.png)


## 3.3 When and how can two operations cause a conflict?
![](image/Pasted%20image%2020260124094600.png)

Two operations are said to be conflicting if all conditions are satisfied:
- C1: They belong to different transactions
- C2: They operate on the same data item
- C3: At Least one of them is a write operation



Two operations are said to be conflicting if all conditions are satisfied:

- They belong to different transactions
    
- They operate on the same data item
    
- At Least one of them is a write operation

![](image/Pasted%20image%2020260124095256.png)

## 3.4 When are two schedules Conflict Equivalent and When is a Schedule Conflict-Serializable?
    

Solution

- A conflict is a pair of operations in a schedule that if their order is changed, the behaviour of at least one TX changes
    
- Non-conflicting operations: When two operations operate on separate data items or the same data item but at least one of them is a read operation, they are said to be non-conflicting.
    
- **Conflict Equivalent** If a schedule S can be transformed into a schedule S´ by a series of swaps of non-conflicting instructions, we say that S and S´ are conflict equivalent. We say that a schedule S is conflict serializable if it is conflict equivalent to a serial schedule.
    
- A schedule is conflict-serializable if a conflict-equivalent serial schedule exists. Conflict serializable is a subset of serializable, so just because a schedule is conflict serializable does mean it is serializable.

- **冲突**：调度中的一对操作，如果它们的顺序被改变，至少一个事务的行为会发生变化。

- **非冲突操作**：当两个操作作用于不同的数据项，或作用于同一数据项但至少有一个是读操作时，称它们为非冲突操作。
    
- **冲突等价**：如果调度 S 可以通过一系列非冲突指令的交换转变为调度 S´，则称 S 和 S´ 是冲突等价的。如果一个调度与某个串行调度冲突等价，则称该调度是**冲突可串行化**的。
    
- 如果一个调度存在冲突等价的串行调度，则该调度是冲突可串行化的。冲突可串行化是可串行化的子集，因此一个调度是冲突可串行化的，并不直接意味着它是可串行化的（在广义上）——但**冲突可串行化是可串行化的充分条件**（在并发控制理论中通常认为冲突可串行化 ⇒ 可串行化，除非有边缘情况如谓词读写）。

![](image/Pasted%20image%2020260124095314.png)

# 4 Locking

## 4.1 
1. Consider the following two transactions.
    1. Add lock and unlock instructions to transactions T1 and T2 so that they observe the two-phase locking protocol.
    2. Can the execution of these transactions result in a deadlock?

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


---

Solution to 1

```
T1: BEGIN                                
    S-LOCK(A)                            
    R(A)                                 
    X-LOCK(B)                            
    R(B)                                 
    if A == 0 then B = B + 1             
    W(B)
    UNLOCK(A)
    UNLOCK(B)
    COMMIT                               
    
T2: BEGIN                      
    S-LOCK(B)                  
    R(B)                       
    X-LOCK(A)                  
    R(A)                       
    if B == 0 then A = A + 1   
    W(A)
    UNLOCK(B)
    UNLOCK(A)
    COMMIT    
```



---

Solution to 2

Execution of these transactions can result in deadlock. For example, consider the following partial schedule:

```

T1: BEGIN                   T2:
    S-LOCK(A)
                                BEGIN
                                S-LOCK(B)
                                R(B)
    R(A)
    X-LOCK(B)
                                X-LOCK(A)
```

![](image/Pasted%20image%2020260124095644.png)

**步骤分析：**

1. **T1 先开始**：
    
    - `S-LOCK(A)` → T1 获得 A 的**共享锁**。
        
    - `R(A)` → 读取 A（可以执行，因为有 S 锁）。
        
2. **T2 同时或交错运行**：
    
    - `S-LOCK(B)` → T2 获得 B 的**共享锁**。
        
    - `R(B)` → 读取 B。
        
3. **接下来**：
    
    - T1 执行 `X-LOCK(B)` → 申请 B 的**排他锁**。  
        但此时 T2 持有 B 的共享锁，不兼容 → **T1 等待 T2 释放 B 的锁**。
        
4. **T2 继续**：
    
    - 执行 `X-LOCK(A)` → 申请 A 的**排他锁**。  
        但此时 T1 持有 A 的共享锁，不兼容 → **T2 等待 T1 释放 A 的锁**。

**此时形成环路等待**：

- T1 等待 T2 释放 B 的锁。
    
- T2 等待 T1 释放 A 的锁。
    

两个事务互相等待对方持有的资源，都无法继续执行，这就是**死锁**。



**死锁条件满足**：

1. **互斥**：锁是互斥资源。
    
2. **持有并等待**：T1 持有 A 锁等 B 锁，T2 持有 B 锁等 A 锁。
    
3. **不可剥夺**：锁不能强行剥夺。
    
4. **循环等待**：T1 → B（被 T2 持有） → T2 → A（被 T1 持有） → T1。

## 4.2 Example with Lock Manager

![](image/Pasted%20image%2020260124095654.png)


![](image/Pasted%20image%2020260124095929.png)

In the following tasks we used a simple database with a lock manager. Given two database objects, A and B, and two transactions T0 and T1.

1. Design a 2PL-compliant schedule.
    
2. Design an illegal schedule.
    
3. Design a schedule with two transactions that leads to a deadlock.
    

Insert necessary locks to correct locations.

You can play around with this Lockmanager by downloading the [Lockmanager script](https://git.tu-berlin.de/dima/dbt/dbt-wise25-26/-/blob/main/lockmanager/lockmanager.py?ref_type=heads).

```
from resources.scripts.lockmanager import *

db = Datenbank()

db.write("T0", "A")

db.read("T1", "B")

del db
restartkernel()
```

---

```
# solution 1 1. Design a 2PL-compliant schedule.

from resources.scripts.lockmanager import *

db = Datenbank()
db.xl("T0", "A")
db.write("T0", "A")

db.sl("T1", "B")

db.ul("T0", "A")

db.read("T1", "B")
db.ul("T1", "B")
del db
restartkernel()



---

Success: Transaction T0 has exclusive lock on object A!
Success: Transaction T0 has written object A!
Success: Transaction T1 has shared lock on object B!
Success: Transaction T0 has unlocked object A!
Success: Transaction T1 has read object B!
Success: Transaction T1 has unlocked object B!
Success: All locks have been released!


```

---

```
# solution 2  Design an illegal schedule.

from resources.scripts.lockmanager import *

db = Datenbank()
db.sl("T0", "C")
db.write("T0", "A")

db.read("T1", "B")

del db
restartkernel()


---
Success: Transaction T0 has shared lock on object C!
Error: Transaction T0 does not have a write lock on object A!
Error: Transaction T1 does not have a lock on object B!
Error: Not all locks have been released!

```


---

```
# solution 3  Design a schedule with two transactions that leads to a deadlock.

from resources.scripts.lockmanager import *

db = Datenbank()
db.xl("T0", "X")
db.read("T0", "A")
db.read("T1", "B")
db.xl("T1", "X")
db.ul("T1", "X")
db.ul("T1", "X")
del db
restartkernel()

---

Success: Transaction T0 has exclusive lock on object X!
Error: Transaction T0 does not have a lock on object A!
Error: Transaction T1 does not have a lock on object B!
Error: Transaction T1 could not acquire exclusive lock on object X!
Error: Transaction T1 does not have a lock on object X!
Error: Transaction T1 does not have a lock on object X!
Error: Not all locks have been released!


```


---

```
# solution 4

from resources.scripts.lockmanager import *

db = Datenbank()
db.xl("T0", "A")
db.write("T0", "A")

db.xl("T1", "B")
db.read("T1", "B")

db.xl("T0", "B")
db.xl("T1", "A")

db.ul("T0", "A")
db.ul("T0", "B")
db.ul("T1", "A")
db.ul("T1", "B")
del db
restartkernel()

---

Success: Transaction T0 has exclusive lock on object A!
Success: Transaction T0 has written object A!
Success: Transaction T1 has exclusive lock on object B!
Success: Transaction T1 has read object B!
Error: Transaction T0 could not acquire exclusive lock on object B!
Error: Transaction T1 could not acquire exclusive lock on object A!
Success: Transaction T0 has unlocked object A!
Error: Transaction T0 does not have a lock on object B!
Error: Object A was not locked!
Success: Transaction T1 has unlocked object B!
Error: Not all locks have been released!
```