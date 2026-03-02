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

如果一个调度只执行完一个事务的所有操作后才开始执行另一个事务，则该调度是串行的。或者，如果一个调度中来自不同事务的操作没有交错，则该调度是串行的。
如果一个调度中多个事务的操作以非顺序方式执行，则该调度是交错的。
如果一个调度对于事务集合 T 的结果等于 T 的某个串行调度的结果，则该调度称为可串行化的。


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

- **冲突**：调度中的一对操作，如果它们的顺序被改变，至少一个事务的行为会发生变化
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



# 5 Quiz


## 5.1 SQL grammer 


![](image/Pasted%20image%2020260124153731.png)


典型的 SQL 条件语法包括：

1. 比较：`<Attribute> = <Attribute>` 或 `<Attribute> = <Literal>` 等
    
2. 逻辑组合：`<Condition> AND <Condition>`
    
3. 模式匹配：SQL 中使用 `LIKE`，但题目中写的是 `FOUND IN` 或 `AVAILABLE`，这不是标准 SQL。
    

---

逐条分析：

1. **`<Condition> ::= <Attribute> FOUND IN <Pattern>`**
    
    - SQL 中没有 `FOUND IN` 语法。类似功能是 `LIKE` 或 `IN`，但不是 `FOUND IN`。  
        ⇒ **False**
        
2. **`<Condition> ::= <Attribute> = <Attribute>`**
    
    - 有效的比较条件，比如 `R.a = S.a`。  
        ⇒ **True**
        
3. **`<Condition> ::= <Attribute> AVAILABLE <Pattern>`**
    
    - SQL 中没有 `AVAILABLE` 操作符。  
        ⇒ **False**
        
4. **`<Condition> ::= <Condition> AND <Condition>`**
    
    - 有效的逻辑组合条件。  
        ⇒ **True**



## 5.2 **precedence graph**


好的，我们逐步分析这个调度并构建优先图（precedence graph）。

调度顺序（按步骤编号）：

1. \( W_2(C) \)
2. \( W_1(A) \)
3. \( R_3(B) \)
4. \( W_4(B) \)
5. \( R_2(C) \)
6. \( W_3(C) \)
7. \( R_3(A) \)
8. \( W_2(B) \)

---

涉及事务：T1, T2, T3, T4。

---

识别冲突（读写/写写/写读）并建立优先边**

**冲突规则**：两个不同事务的两个操作访问同一数据项，且至少有一个是写操作，并且它们在调度中出现的顺序会影响最终结果或另一个事务的读取结果。

---

**(A) 数据项 C**
- 操作：\( W_2(C)[1] \), \( R_2(C)[5] \), \( W_3(C)[6] \)
  同一事务 T2 的 W 和 R 不与其他事务冲突（除了自己的写写冲突没有）。
  冲突：
  - \( W_2(C)[1] \)（写） 和 \( W_3(C)[6] \)（写） ⇒ 写写冲突，T2 → T3
  - \( R_2(C)[5] \)（读） 和 \( W_3(C)[6] \)（写） ⇒ 读写冲突（读在写前），T2 → T3
  注意：同一个 T2 的 W→R 不是冲突边（但这里 T3 写 C 在 T2 读 C 之后？等等，T2 的 W₂(C) 在步骤1，R₂(C) 在步骤5，之间 W₃(C) 在步骤6，但 W₃(C) 在 R₂(C) 之后吗？不，看错！重新检查：
  顺序：1:W₂(C), 5:R₂(C), 6:W₃(C)
  所以 R₂(C) 在 W₃(C) 之前 ⇒ R₂(C) → W₃(C) 冲突 T2→T3
  W₂(C) 在 W₃(C) 之前 ⇒ W₂(C) → W₃(C) 冲突 T2→T3
  这两个冲突边是同一方向。

所以从 C 得到 T2 → T3。

---

**(B) 数据项 A**
- 操作：\( W_1(A)[2] \), \( R_3(A)[7] \)
  - W₁(A) 在 R₃(A) 之前 ⇒ T1 → T3（写读冲突）

---

**(C) 数据项 B**
- 操作：\( R_3(B)[3] \), \( W_4(B)[4] \), \( W_2(B)[8] \)
  冲突：
  1. \( R_3(B)[3] \) 和 \( W_4(B)[4] \) ⇒ 读写冲突（读在写前），T3 → T4
  2. \( W_4(B)[4] \) 和 \( W_2(B)[8] \) ⇒ 写写冲突，T4 → T2

---

汇总优先边**
1. T2 → T3  (来自 C 的 W₂(C)→W₃(C) 和 R₂(C)→W₃(C))
2. T1 → T3  (来自 A 的 W₁(A)→R₃(A))
3. T3 → T4  (来自 B 的 R₃(B)→W₄(B))
4. T4 → T2  (来自 B 的 W₄(B)→W₂(B))

---

画优先图**
节点：T1, T2, T3, T4

有向边：
- T1 → T3
- T2 → T3
- T3 → T4
- T4 → T2

这形成了 **T2 → T3 → T4 → T2** 的环，因此调度不是冲突可串行化。

---

如果你需要“从下拉菜单选择正确条目”的具体界面答案，那就是按这个关系：

**T1 → T3**  
**T2 → T3**  
**T3 → T4**  
**T4 → T2**

在 1 3 2 4 的下拉菜单里，1 指 T1，2 指 T2，3 指 T3，4 指 T4。

所以在题目给的“1 3 2 4”四个节点之间的箭头：
- 从 1 到 3 有边（T1→T3）
- 从 2 到 3 有边（T2→T3）
- 从 3 到 4 有边（T3→T4）
- 从 4 到 2 有边（T4→T2）

---

这样**优先图就完整了**，并且检测到环。


![](image/Pasted%20image%2020260124155505.png)

## 5.3 判断 schedule is conflict-serializable:

The following schedule is conflict-serializable:

R2(B), R4(B), R4(A), W3(C), W2(A), W1(B), W2(B), R3(A)


我们来判断这个调度是否冲突可串行化（conflict-serializable）。

**调度顺序**（已编号）：
1. R₂(B)
2. R₄(B)
3. R₄(A)
4. W₃(C)
5. W₂(A)
6. W₁(B)
7. W₂(B)
8. R₃(A)

事务：T1, T2, T3, T4。

---

识别冲突边**

**数据项 A**：
- R₄(A)[3] 和 W₂(A)[5] ⇒ R→W 冲突，T4 → T2
- W₂(A)[5] 和 R₃(A)[8] ⇒ W→R 冲突，T2 → T3

**数据项 B**：
- R₂(B)[1] 和 W₁(B)[6] ⇒ R→W 冲突，T2 → T1
- R₄(B)[2] 和 W₁(B)[6] ⇒ R→W 冲突，T4 → T1
- W₁(B)[6] 和 W₂(B)[7] ⇒ W→W 冲突，T1 → T2
- R₂(B)[1] 和 W₂(B)[7] 是同一事务，不算冲突边

**数据项 C**：
只有 W₃(C)[4]，没有其他冲突（除非有其他事务读写 C，这里没有）

---

汇总冲突边**
1. T4 → T2  (来自 R₄(A)→W₂(A))
2. T2 → T3  (来自 W₂(A)→R₃(A))
3. T2 → T1  (来自 R₂(B)→W₁(B))
4. T4 → T1  (来自 R₄(B)→W₁(B))
5. T1 → T2  (来自 W₁(B)→W₂(B))

---


从边中我们可以看到：
- T1 → T2 → T3 （无环）
- T1 → T2 → T1? 没有直接的 T2→T1，但有 T2→T1 来自边3，所以 T2→T1 与 T1→T2 构成环吗？
  检查：
  - T1 → T2 (边5)
  - T2 → T1 (边3)  
  确实 T1 → T2 且 T2 → T1 ⇒ **环 T1 ↔ T2**。

存在环，所以**不是冲突可串行化**。

---

**答案**：
\[
\boxed{\text{Falsch}}
\]


## 5.4 **Increment (I) lock und Multiplication lock **

Consider that in addition to the **shared (S)** and **exclusive (X)** locks, your database system has two more locks:

1. **Increment (I)** - increments a database element by a constant, and
2. **Multiplication (M)** - multiplies a database element by a constant.

Fill in the appropriate value for the drop-downs in the compatibility matrix:

(N.A. stands for not applicable. Only the drop-down in the table is graded.)

![](image/Pasted%20image%2020260124190707.png)




1. **共享锁 (S)**：允许多个事务同时读取，但禁止写入。
    
2. **排他锁 (X)**：禁止其他任何锁。
    
3. **增量锁 (I)**：
    
    - 如果增量操作是**可交换和可结合**的（例如 `A = A + c`），那么多个增量锁可能兼容（例如两个事务同时对同一数据项做加法，顺序不影响最终结果）。
        
    - 但通常增量锁与**排他锁不兼容**，因为排他锁要覆盖整个值。
        
    - 增量锁与**共享锁**可能不兼容，因为读取的值在增量过程中会变化（除非是快照隔离）。
        
4. **乘法锁 (M)**：
    
    - 乘法操作通常**不兼容**，因为 `A = A * c1` 和 `A = A * c2` 的顺序会影响结果（除非特殊约束如 c1=c2=1），所以乘法锁之间通常互斥。
        
    - 乘法锁与排他锁不兼容。
        
    - 乘法锁与共享锁不兼容（因为读的值在变化）。


## 5.5 strict two-phase locking 

The following schedule is possible under strict two-phase locking (S2PL): R1(D), R2(C), R2(A), R1(A), R3(B), R3(B), R1(B), R3(B)

**严格两阶段锁（S2PL）规则**

1. 事务在读取数据项前必须获得共享锁（S-lock），在写入前必须获得排他锁（X-lock）。
2. 事务持有的所有锁必须在事务提交后才释放（严格性质 ⇒ 锁保持到事务结束）。
3. 锁必须按**两阶段**获取：增长阶段（只能加锁） → 收缩阶段（只能释放锁），但在严格 2PL 中，没有单独的收缩阶段，因为释放锁在提交时一次性完成。
4. 不同事务对同一数据项的锁必须兼容（S 与 S 兼容，S 与 X 不兼容，X 与 X 不兼容）。

---

**数据项 A**
- 步骤3：R₂(A) ⇒ T2 获得 A 的 S 锁。
- 步骤4：R₁(A) ⇒ T1 要获得 A 的 S 锁。
    - S 与 S 兼容，所以 T1 可以获得 S 锁，没问题。
    - 但 T2 在 R₂(A) 后没有立即释放锁（严格 2PL 要等到提交才释放），所以 T1 和 T2 同时持有 A 的 S 锁，兼容，没问题。

 **数据项 B**
- 步骤5：R₃(B) ⇒ T3 获得 B 的 S 锁。
- 步骤6：R₃(B)（同一事务再次读 B）⇒ 已持有 S 锁，没问题。
- 步骤7：R₁(B) ⇒ T1 要获得 B 的 S 锁。
    - S 与 S 兼容，所以 T1 可以同时持有 B 的 S 锁，没问题。
- 步骤8：R₃(B) ⇒ T3 再次读 B，已持有 S 锁，没问题。

这里也没有冲突，因为所有对 B 的操作都是读。

 
 **数据项 C**
- 步骤2：R₂(C) ⇒ T2 获得 C 的 S 锁，没问题。

**数据项 D**
- 步骤1：R₁(D) ⇒ T1 获得 D 的 S 锁，没问题。


---


这个调度**只有读操作**，没有写操作。  
在严格 2PL 下，所有事务只请求共享锁，共享锁之间完全兼容，因此任何顺序的读操作都是允许的。

没有锁冲突，也没有提前释放锁的问题（因为严格 2PL 只是要求锁保持到事务结束，这里没有写锁，不会造成阻塞或死锁）。



## 5.6 Strict Two-phase locking and Snapshot Isolation

Strict Two-phase locking allows more concurrency than Snapshot Isolation (SI).   ->  false 

**解释**：

**严格两阶段锁（Strict 2PL）** 与 **快照隔离（Snapshot Isolation，SI）** 在并发性方面的比较：

1. **Strict 2PL**：
    - 读操作需要共享锁（S-lock），写操作需要排他锁（X-lock）。
    - 锁保持到事务结束，因此：
        - **读阻塞写**（如果事务持有 S 锁，其他事务无法获取 X 锁）。
        - **写阻塞读**（如果事务持有 X 锁，其他事务无法获取 S 锁）。
    - 容易导致阻塞和死锁，限制并发度。
2. **快照隔离（SI）** 
    - 读操作基于事务开始时的数据快照，不需要加锁。
    - 写操作在提交时检查写-写冲突（“先提交者获胜”规则）。
    - 因此：
        - **读不阻塞写**，**写不阻塞读**。
        - 只有并发写同一数据项时才可能冲突导致中止。
    - 在读多写少的场景下，并发度显著高于 Strict 2PL。
        

---

**结论**：  
快照隔离通常比严格两阶段锁**允许更高的并发度**，因为读操作完全不加锁，不会阻塞其他事务。因此，说“Strict 2PL 允许比 SI 更多的并发”是错误的。

## 5.7 

![](image/Pasted%20image%2020260124200615.png)


我们按时间戳调度算法（严格时间戳排序 + Thomas 写规则）来填表。

---

**已知**：
- 时间戳：TS(T₁)=250, TS(T₂)=275, TS(T₃)=175
- 初始 RT(X)=0, WT(X)=0, C(X)=false（或表示未提交的写？书上“commit bit”可能指该元素最后被已提交的事务写过，这里我们按书 18.8.4 规则推演）

在时间戳调度中：
1. 读操作 Rᵢ(X)：
   - 如果 TS(Tᵢ) < WT(X) 且 C(X)=true → 回滚 Tᵢ
   - 否则允许读，RT(X) = max(RT(X), TS(Tᵢ))
2. 写操作 Wᵢ(X)：
   - 如果 TS(Tᵢ) < RT(X) → 回滚 Tᵢ（更年轻的事务已经读过旧值）
   - 如果 TS(Tᵢ) < WT(X) 且 C(X)=true → 根据 Thomas 写规则可忽略此写（旧事务的写被覆盖）
   - 否则允许写，设置 WT(X)=TS(Tᵢ)，C(X)=false（提交时再设 true）

---

初始状态**
RT(A)=RT(B)=RT(C)=0  
WT(A)=WT(B)=WT(C)=0  
C(A)=C(B)=C(C)=false（假设最初有已提交的值？题目说“初始读写时间戳为 0”，可能认为 WT=0 是已提交的初始写入，所以 C=true？这会影响 W1(B) 的判断。  
但在书 18.8.4 例子里，初始 C(X)=true，表示已有提交的值。我们按 C(X)=true 开始。）

---

逐个操作分析

**(1) R₁(B)**
TS(T₁)=250, WT(B)=0, C(B)=true, 250 ≥ 0 → 允许，RT(B) = max(0,250)=250 ✅ 表中已给 RT(B)=250

**(2) R₂(A)**
TS(T₂)=275, WT(A)=0, C(A)=true, 275 ≥ 0 → 允许，RT(A) = max(0,275)=275 ✅ 表中已给 RT(A)=275

**(3) R₃(C)**
TS(T₃)=175, WT(C)=0, C(C)=true, 175 ≥ 0 → 允许，RT(C) = max(0,175)=175 ✅ 表中已给 RT(C)=175

---

**(4) W₁(B)**  
TS(T₁)=250  
- RT(B)=250 → TS(T₁) < RT(B)? 250 < 250? 否  
- WT(B)=0, C(B)=true → TS(T₁)=250 > WT(B)=0，不是更年轻的事务写旧值（其实 250>0），所以不触发 Thomas 忽略  
- 允许写，更新 WT(B)=TS(T₁)=250，C(B)=false（因为 T₁ 尚未提交）

**填表**：WT(B)=**250** , C(B)=**false**

---

**(5) W₁(A)**
TS(T₁)=250  
- RT(A)=275 → TS(T₁)=250 < RT(A)=275 ⇒ **ROLLBACK T₁** ✅ 表中已填 ROLLBACK

T₁ 回滚后，W₁(B) 写入的 B 值会撤销，并且 WT(B) 恢复？但算法中回滚后事务终止，之前设置 WT(B)=250 和 C(B)=false 应作废。但题目表里 W1(B) 行已经让我们填 WT(B) 和 C(B)，我们只能按当时操作成功填（即使后来回滚，当时还是发生了写并改动了 WT 和 C）。不过可能因为回滚，C(B) 在系统里会被恢复为 true？  
但题目只要求按当时操作填。

---

**(6) W₂(C)**
TS(T₂)=275  
- RT(C)=175 → 275 ≥ 175 ✓  
- WT(C)=0, C(C)=true, 275 > 0 → 允许写  
- 设置 WT(C)=275, C(C)=false

**填表**：WT(C)=**275** , C(C)=**false**

---

**(7) W₃(A)**
TS(T₃)=175  
- RT(A)=275 → 175 < 275 ⇒ **ROLLBACK T₃** ✅ 表中已填 ROLLBACK

---

**最终填表答案**：

| Operation | Action    |
|---|---|
| R1(B)    | RT(B)=250    |
| R2(A)    | RT(A)=275    |
| R3(C)    | RT(C)=175    |
| W1(B)    | WT(B)=**250** , C(B)=**false** |
| W1(A)    | ROLLBACK    |
| W2(C)    | WT(C)=**275** , C(C)=**false** |
| W3(A)    | ROLLBACK    |

---

如果你需要，我可以把整个调度过程的时间戳更新和回滚原因再详细列出。


好的，我们来详细逐步推演这个调度过程，并说明每个操作的处理逻辑和时间戳变化。

---

**初始状态**
事务时间戳：
- TS(T₁) = 250  
- TS(T₂) = 275  
- TS(T₃) = 175  

数据库元素初始状态（假设 0 表示一个已提交的初始写入）：
| 元素 | RT   | WT   | C（提交位） |
|------|------|------|------------|
| A    | 0    | 0    | true       |
| B    | 0    | 0    | true       |
| C    | 0    | 0    | true       |

---

 **1. R₁(B)**
- TS(T₁) = 250
- 检查：WT(B) = 0, C(B) = true
- 条件判断：
  - TS(T₁) ≥ WT(B)？ 250 ≥ 0 → 是
  - 允许读
- 更新：RT(B) = max( RT(B), TS(T₁) ) = max(0, 250) = 250
- **操作后状态**：  
  RT(B) = 250，其余不变

表中已填：RT(B)=250 ✅

---

**2. R₂(A)**
- TS(T₂) = 275
- WT(A) = 0, C(A) = true
- 检查：TS(T₂) ≥ WT(A) → 275 ≥ 0 → 是
- 允许读
- 更新：RT(A) = max(0, 275) = 275
- **状态**：  
  RT(A) = 275

表中已填：RT(A)=275 ✅

---

**3. R₃(C)**
- TS(T₃) = 175
- WT(C) = 0, C(C) = true
- 检查：TS(T₃) ≥ WT(C) → 175 ≥ 0 → 是
- 允许读
- 更新：RT(C) = max(0, 175) = 175
- **状态**：  
  RT(C) = 175

表中已填：RT(C)=175 ✅

---

**4. W₁(B)**
- TS(T₁) = 250
- **规则检查**：
  1. **TS(T₁) < RT(B)** ？ RT(B)=250 → 250 < 250 ? 否
  2. **TS(T₁) < WT(B) 且 C(B) = true** ？ WT(B)=0 → 250 < 0 ? 否
- 允许写（不触发 Thomas 忽略）
- **更新**：
  - WT(B) = TS(T₁) = 250
  - C(B) = false（因为 T₁ 还未提交，写入是未提交状态）
- **状态**：  
  WT(B)=250, C(B)=false

**表中填空**：WT(B)=**250** , C(B)=**false**

---
**5. W₁(A)**
- TS(T₁) = 250
- **规则检查**：
  1. **TS(T₁) < RT(A)** ？ RT(A)=275 → 250 < 275 → **是**  
      → **触发回滚**，因为已经有更年轻的事务（T₂，TS=275）读过 A，T₁ 的写会破坏 T₂ 读的一致性。
- 因此 **T₁ 回滚**。
- 回滚后：T₁ 的所有写操作（这里只写了 B）被撤销。但 WT(B) 和 C(B) 会被恢复吗？  
  实际系统中，回滚会恢复 B 到之前的值，WT(B) 可能回退到之前的值（这里之前 WT(B)=0, C(B)=true），但题目表中只要求填调度器在操作执行时的动作，不是最终状态，所以 W₁(B) 行已经按当时情况填过了。

表中已填：ROLLBACK ✅

---
 **6. W₂(C)**
- TS(T₂) = 275
- **规则检查**：
  1. **TS(T₂) < RT(C)** ？ RT(C)=175 → 275 < 175 ? 否
  2. **TS(T₂) < WT(C) 且 C(C)=true** ？ WT(C)=0 → 275 < 0 ? 否
- 允许写
- **更新**：
  - WT(C) = TS(T₂) = 275
  - C(C) = false（T₂ 未提交）
- **状态**：  
  WT(C)=275, C(C)=false

**表中填空**：WT(C)=**275** , C(C)=**false**

---
**7. W₃(A)**
- TS(T₃) = 175
- **规则检查**：
  1. **TS(T₃) < RT(A)** ？ RT(A)=275 → 175 < 275 → **是**  
      → 触发回滚，因为更年轻的事务 T₂ 读过 A。
- 因此 **T₃ 回滚**。

表中已填：ROLLBACK ✅

---

**最终完整表格**

| Operation | Action                |
|-----------|-----------------------|
| R1(B)     | RT(B)=250             |
| R2(A)     | RT(A)=275             |
| R3(C)     | RT(C)=175             |
| W1(B)     | WT(B)=**250**, C(B)=**false** |
| W1(A)     | ROLLBACK              |
| W2(C)     | WT(C)=**275**, C(C)=**false** |
| W3(A)     | ROLLBACK              |

---

**注意**：在真实系统中，T₁ 回滚后 WT(B) 和 C(B) 会被恢复（可能 WT(B)=0, C(B)=true），但此表记录的是每个操作**执行时**的调度器动作，所以 W1(B) 那行的填写就是操作执行时的修改，后续回滚不影响该行内容。
