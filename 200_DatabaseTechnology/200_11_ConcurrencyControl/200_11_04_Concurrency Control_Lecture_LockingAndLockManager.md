

# 1 Locking 


Enforcing Serializability
• Precedence graphs are not a concurrency control mechanism!
    • We need to know the schedule in advance
    • What about interactive systems?
• Instead a scheduler ensures serializability by delaying operations or aborting transactions
• ==Pessimistic concurrency control – Locking==
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


- 通过**锁定数据库元素**来实施可串行化
- **两种类型的锁**
    - **读锁 / 共享锁**
    - **写锁 / 排他锁**
    - 读锁可以**升级**为排他锁
- 锁由事务**请求**
    - 也可能由调度器**隐式**请求
- 锁由**锁管理器**管理和授予
    - 全局**内存数据结构**
    - 是调度器的一部分

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


- 事务只有在满足以下条件时才能读写一个元素：
    - 它已获得该元素的锁
    - 它尚未释放该锁
- 如果事务持有一个锁，它必须在之后释放它
    - 例如，由调度器在提交或中止时隐式释放
- 调度中的额外操作：
    - **S(E)** – 请求元素 E 的共享锁
    - **X(E)** – 请求元素 E 的排他锁
    - **U(E)** – 释放元素 E 上持有的（任何）锁
- 带锁的调度示例：
    S₁(A), R₁(A), X₁(B), W₁(B), U₁(A), U₁(B)

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

两阶段加锁：先获得读锁，然后再获得排他锁
不能在不先获得读锁的情况下直接获得排他锁 ， 因为如果你先获得读锁，其他事务在写锁释放后仍然可以读取该数据
每个事务必须持有所有锁直到事务结束

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
• ==Read locks are compatible with other read locks==
    • Multiple TX can have a read lock on same element
• ==Write locks are not compatible with other locks==
    • A read lock and write lock cannot exist on the same element at the same time
• NOTE: If a transaction requests a lock that cannot be granted, the operations is delayed (or the transaction is aborted)


- 多个事务可以持有对同一元素的锁
- **读锁与其他读锁兼容**
    - 多个事务可以对同一元素持有读锁
- **写锁与其他锁都不兼容**
    - 读锁和写锁不能同时存在于同一元素上
- 注意：如果事务请求的锁无法被授予，则该操作被**延迟**（或事务被**中止**）

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


加锁阶段（Expanding/Growing Phase）：
事务可以获取锁，但不能释放任何锁
通常先获取读锁，之后可能升级为写锁

释放阶段（Shrinking Phase）：
事务可以释放锁，但不能获取任何新锁
所有锁在事务提交或中止时释放

为什么需要两阶段锁？
保证冲突可串行化
防止事务在释放锁后又被其他事务读取已修改但未提交的数据

![](image/Pasted%20image%2020250118204359.png)

![](image/Pasted%20image%2020260124012416.png)

![](image/Pasted%20image%2020250118204419.png)



严格两阶段锁（Strict 2PL）**不会**带来**最大并行度**和**最大吞吐量**，原因如下：
1. **锁持有时间过长**：在 Strict 2PL 下，事务持有的所有锁必须在事务提交后才释放。这意味着其他事务可能更长时间地等待锁，从而减少并发度。
2. **降低并行度**：由于锁被持有到事务结束，其他需要相同数据项的事务会被阻塞，无法并行执行。
3. **实际中的权衡**：数据库系统通常在**并发度**和**正确性/恢复简单性**之间权衡。
    - Strict 2PL 的优点主要是**简化恢复**（避免级联回滚）和**保证冲突可串行化**。
    - 为了更高并发，有时会使用较弱的隔离级别或更灵活的锁释放策略（如非严格 2PL 或多版本并发控制 MVCC）。
4. **现代实践**：许多高性能数据库（如 PostgreSQL、Oracle）使用 **MVCC** 而不是纯 Strict 2PL 来获得更高并发，因为读不阻塞写、写不阻塞读。

因此，说 Strict 2PL 带来“最大并行度和最大吞吐量”是错误的，它是以降低并发为代价来保证严格的可串行化和简单恢复。


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
    

---

**避免方法**：

- 让事务按相同顺序申请锁（例如都先申请 A 锁，再申请 B 锁）。
    
- 使用超时机制或死锁检测来解除死锁。

---


### 2.5.3 Recoverable Schedules
• 2PL does not guarantee recoverable schedules
    • 2PL only requires lock all items before processing but it is not mandatory to commit
    • Recall: A schedule S is called recoverable, if, whenever a T1 reads an object X whose value was before written by a unfinished T2, then S contains a commit for T2 (at whatever place)
    • Ensures that if a transaction fails, no other transactions depend on its uncommitted changes.
    • When T2 starts, it may lock and write objects written by T1
    • Commit of T2 violates recoverable because T1 abort


- **两阶段锁不保证可恢复调度**
    - 两阶段锁只要求在访问前锁定所有数据项，但不强制事务提交
    - 回顾：如果每当 T₁ 读取了某个由未完成的 T₂ 写入的 X 的值，调度 S 中包含 T₂ 的提交（在任何位置），则调度 S 称为**可恢复的**
    - 确保如果一个事务失败，没有其他事务依赖于其未提交的更改
    - 当 T₂ 开始时，它可能锁定并写入 T₁ 写入的对象
    - T₂ 的提交违反可恢复性，因为 T₁ 可能中止


![](image/Pasted%20image%2020250118204705.png)


我来帮你解释这段话的意思，特别是为什么 **2PL 不保证可恢复调度**。

---

背景：什么是可恢复调度？

**可恢复调度（Recoverable Schedule）** 的定义：
> 如果事务 T₁ 读取了事务 T₂ 写入的数据，并且 T₂ 还没有提交，那么 T₁ 必须在 T₂ **提交之后**才能提交。

换句话说：  
如果一个事务读到了另一个**未提交事务**写的数据，那么它必须等到那个事务提交后才能提交。  
这样做的目的是：如果 T₂ 后来**中止**了，那么 T₁ 读到的数据就是"脏数据"，T₁ 也必须中止，否则数据库就不一致。

---

为什么 2PL 不保证可恢复调度？

**2PL 只要求：**
- 在访问数据前加锁
- 在事务结束后释放锁
- 但不要求事务的提交顺序

**问题出在哪里？**

看一个例子：

```
T1: W1(A)  (写 A，未提交)
T2: R2(A)  (读 T1 写的 A)
T2: 提交
T1: 中止
```

**按 2PL 规则：**
- T1 写 A 前加写锁
- T2 读 A 前加读锁（如果 T1 还没释放写锁，T2 会等）
- 假设 T1 先释放写锁（比如在提交前释放），T2 就能读到 A
- 然后 T2 提交
- 最后 T1 中止

**结果：**
- T2 读到了 T1 未提交的数据（脏读）
- T2 提交了，但 T1 后来中止了
- 这就违反了**可恢复调度**的要求：  
  T2 读到了未提交的 T1 的数据，T2 提交了，但 T1 中止了 → 不可恢复

---

 2PL 缺少什么？

2PL 只控制了**并发访问的顺序**（通过锁），但没有控制**事务提交的顺序**。

要保证可恢复调度，还需要一个额外的规则：
> 如果 T₂ 读到了 T₁ 写的数据，那么 T₂ 必须在 T₁ 提交之后才能提交。

这叫做 **"先提交依赖事务，再提交依赖者"**。

---

- **2PL 保证冲突可串行化** ✅
- **2PL 不保证可恢复调度** ❌  
  因为事务的提交顺序不受 2PL 控制
- 要保证可恢复调度，需要在 2PL 基础上加一个规则：  
  **"读未提交数据的事务，必须在被读事务提交后才能提交"**

这就是为什么实际系统中常用 **"严格两阶段锁（Strict 2PL）"**，它要求所有锁在事务提交后才释放，避免了脏读和不可恢复调度。


### 2.5.4 Problem of Cascading Aborts

Cascading Abort:
• It does not lead to cascading rollbacks if a Tx fails
• Delay reading TxR of uncommitted data until writing TxW committed and thus TxR does not need to be rolled back

级联中止：

如果一个事务失败，不会导致级联回滚
延迟读取未提交数据的事务，直到写事务提交，这样读事务就不需要回滚

![](image/Pasted%20image%2020250118204812.png)


---


### 2.5.5 Strict 2PL

• Exclusive locks are released only after commit/abort is acknowledged by scheduler
• Ensures recoverable schedules
• But: less parallelization, less throughput
• Deadlocks may still happen but no cascading aborts
• In practice, locking-based systems implement Strict 2PL

排他锁只有在提交/中止被调度器确认后才释放
确保可恢复调度
但：并行度降低，吞吐量下降
死锁仍可能发生，但不会出现级联中止
在实践中，基于锁的系统实现的是严格两阶段锁

![](image/Pasted%20image%2020250118204836.png)

![](image/Pasted%20image%2020250118204905.png)


## 2.6 Lock Granularities


• Lock database elements organized in a hierarchy to cope with different granularities, e.g., blocks, relations, tuples, etc.
1. To place S or X lock on any element, we must begin at the root
2. If we are at the element we want to lock, we request an S or X lock
3. Otherwise, if the element is below in the hierarchy, we request an IS or IX intention lock on the current element
![](image/Pasted%20image%2020250118205028.png)



这是关于**多粒度锁（Hierarchical Locking）**的规则，我来翻译成中文：

---

- 为了应对不同粒度（例如块、关系、元组等），将数据库元素按层次结构组织锁定
1. 要在任何元素上放置 S 或 X 锁，我们必须从根节点开始
2. 如果当前元素就是我们想要锁定的元素，则请求 S 或 X 锁
3. 否则，如果目标元素在当前元素的下层，则对当前元素请求 **IS 或 IX 意向锁**

---



## 2.7 Phantom Read Problem

• Occurs within a transaction when the same query produces different output at
different times.
• Example:
• TX 1: Every month we can distribute a total of 10,000 € to the employees of department X
    • `Q1: T := SELECT COUNT(*) FROM Emp WHERE Dept = ‘X’`
    • Q2: UPDATE Emp SET Salary = Salary + 10000 / T
• TX 2: Jane Doe starts working in department X
    • Q3: UPDATE Emp SET Dept = ‘X’ WHERE Name = ‘Jane Doe’
• If Q3 runs between Q1 and Q2, we have distributed too much money
    • TX 1 needs a write lock on the entire table
    • But TX 2 only needs a write lock on a single tuple
    • We can only lock existing items


- **发生情况**：在同一事务中，相同的查询在不同时间点返回不同的结果集。
- **示例**：
    **事务 TX1**：每月我们可以向 X 部门员工发放总额 10,000 €
    - `Q1：T := SELECT COUNT(*) FROM Emp WHERE Dept = 'X'`
    - Q2：`UPDATE Emp SET Salary = Salary + 10000 / T`
    **事务 TX2**：Jane Doe 开始在 X 部门工作
    - Q3：`UPDATE Emp SET Dept = 'X' WHERE Name = 'Jane Doe'`
- **如果 Q3 在 Q1 和 Q2 之间执行**，会导致**发放的总金额过多**：
    - TX1 需要对整个表加写锁
    - 但 TX2 只需要对单个元组加写锁
    - **我们只能对已存在的记录加锁**（无法锁住将来可能插入的、满足条件的记录）



**核心原因**：  
TX1 在 Q1 时统计到 N 个员工，但在 Q2 更新前，TX2 插入/更新了一个新记录，使得实际满足条件的员工变为 N+1 个，导致 10000/(N+1) 被算成 10000/N，多发钱。  
**本质**：**范围查询**无法通过行级锁防止新记录的插入，从而引发数据一致性问题。




## 2.8 Intention Locks Compatibility Matrix

Concept: Intention Locks

‣ Lock database elements organized in a hierarchy to cope with different granularities, e.g., blocks, relations, tuples, etc.

‣ Intention locks allow a higher-level node to be locked in shared mode or exclusive mode without having to check all descendant nodes.

‣ If a node is in an intention mode, then explicit locking is being done at a lower level in the tree.

‣ Hierarchical locks are useful in practice, as each TX only needs a few locks.

‣ Intention locks help improve concurrency:


![](image/Pasted%20image%2020260124014909.png)

**意向锁（Intention Locks）概念**

**‣ 背景**：  
数据库元素（如表、块、行等）以层次结构组织，支持不同粒度的锁定。

**‣ 作用**：  
意向锁允许在较高层级节点上加共享锁或排他锁，而无需检查其所有后代节点的锁定状态。

**‣ 原理**：  
如果一个节点处于“意向模式”，说明该节点的**较低层级正在被显式加锁**。

**‣ 实际价值**：  
层次化锁很实用，因为每个事务通常只需少量锁。

**‣ 优势**：  
意向锁通过**减少锁检查开销**和**允许更细粒度的并发控制**，显著提升并发性能。


----


Intention-Shared (IS): Indicates explicit locking at a lower level with shared locks. Intention-Exclusive (IX): Indicates explicit locking at a lower level with exclusive or shared locks

1. To place S or X lock on any element, we must begin at the root
    
2. If we are at the element we want to lock, we request an S or X lock
    
3. Otherwise, if the element is below in the hierarchy, we request an IS or IX intention lock on the current element

**加锁规则（自顶向下）**

1. **从根节点开始**：  
    要对任何元素加 S 锁或 X 锁，必须从层次结构的根节点开始。
2. **到达目标元素**：  
    如果当前节点就是要加锁的元素，则直接请求 **S 锁** 或 **X 锁**。
3. **在路径中间节点**：  
    如果目标元素在当前节点的**下层**，则对当前节点请求 **IS 锁** 或 **IX 锁**（意向锁）。



---

**补充说明：**

**意向锁（Intention Locks）的含义：**
- **IS（意向共享锁）**：表示当前节点的事务打算在其下层节点上加共享锁
- **IX（意向排他锁）**：表示当前节点的事务打算在其下层节点上加排他锁
- **SIX（共享意向排他锁）**：当前节点加共享锁，同时有意图在其下层加排他锁

**锁的兼容性矩阵（简化版）：**

| 已有锁 | 请求锁 | 是否兼容 |
|--------|--------|----------|
| IS     | IS     | ✅ |
| IS     | IX     | ✅ |
| IX     | IX     | ✅ |
| S      | IS     | ✅ |
| S      | IX     | ❌ |
| X      | 任何   | ❌ |

**为什么需要层次锁？**
- 如果事务要锁一个元组，不需要锁整个表
- 但如果事务要锁整个表，需要知道有没有事务在锁其中的元组
- 意向锁解决了这个问题：表上的意向锁表示"下面有东西被锁了"

**例子：**
```
事务要锁元组 R1.A1：
1. 在数据库根节点加 IX 锁
2. 在表 R1 加 IX 锁
3. 在元组 A1 加 X 锁
```
这样，另一个事务想锁整个表 R1 时，看到表上有 IX 锁，就知道不能加 S 或 X 锁。

需要我画出**层次锁的兼容性矩阵**吗？


----
Example 


• Intention locks IS and IX are compatible with each other
    • Allow conflict to be resolved at lower level
• If a TX wants to update a page P1, it first sets an IX lock on the table and then a X lock on P1
• If another TX wants to update a page P2, it also sets an IX lock on the table
    • The lock on P1 by TX1 does not concern TX2


意向锁 IS 和 IX 之间是相互兼容的
    允许冲突在更低层级解决
如果一个事务想要更新页面 P1，它首先在表上设置 IX 锁，然后在 P1 上设置 X 锁
如果另一个事务想要更新页面 P2，它也在表上设置 IX 锁
    TX1 在 P1 上的锁与 TX2 无关

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


**• 位置与作用**：  
锁管理器是数据库管理系统内部的**内存数据结构**，负责：
- 授予或阻塞锁请求
- 处理死锁
- 管理被阻塞事务的等待队列

**• 事务接口**：  
向事务提供两个主要操作：
- `acquireLock(T, X, mode)`：事务 T 请求对资源 X 加指定模式的锁
- `releaseLock(T, X, mode)`：事务 T 释放对资源 X 的锁

**• 核心结构**：  
锁管理器维护一张**逻辑锁表**，每个表项是一个**逻辑锁数据结构**，包含：
1. **锁模式**（S, X, IS, IX, …）
2. **锁请求链表**（已授予或等待中的请求）
3. **闩锁（latch）**：保护该数据结构本身的**物理锁**（防止并发修改）
4. **被锁定的资源标识**



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

**• 事务尝试获取锁**  
当事务请求对某个资源加锁时，遵循以下步骤：

**1. 遵循层次化锁协议**  
确保事务在高层级节点上持有相应的**意向锁**：
- 资源层次结构是固定的，并硬编码在数据结构中（例如：一行数据知道它所属的页ID `pageId`）
- 如果需要，**递归地发出高层级的锁请求**（自顶向下）
- 如果事务已经持有**更粗粒度的锁**，则立即授予当前请求  
    例如：已持有表级 X 锁时，请求该表某行的 X 锁直接成功
- 否则，通过**哈希表**查找对应的锁对象
    

**2. 闩锁保护与队列管理**
- **闩住（latch）** 该逻辑锁结构，防止并发修改
- 将锁请求**追加到队列**中
- 如果请求的锁模式与当前已授予的锁**不兼容**，事务将**进入阻塞状态**


### 3.2.1 **流程举例**

事务 T1 请求对行 R1 加 X 锁：

1. 根据层次结构（数据库 → 表 → 页 → 行），递归请求：
    
    - 对数据库加 IX 锁
        
    - 对表加 IX 锁
        
    - 对页加 IX 锁
        
2. 检查是否已持有表级 X 锁？否 → 继续
    
3. 通过哈希表找到行 R1 的锁结构
    
4. 闩住该锁结构，检查兼容性：
    
    - 若无冲突 → 授予锁，加入 granted 链表
        
    - 若冲突（如已有其他事务的 S 锁） → 加入 waiting 链表，T1 阻塞
        

---

### 3.2.2 **关键设计点**

- **递归请求**：自动确保意向锁的完整性
    
- **哈希加速**：快速定位锁对象
    
- **队列化**：公平处理等待事务
    
- **阻塞机制**：不兼容时事务挂起，避免忙等待


## 3.3 Releasing Locks

• Transactions maintain pointers to held logical locks in request order
• Locks are released one by one in request order at end of transaction

• To release lock:
    • Latch lock and unlink corresponding request
    • Traverse request list to discover new lock mode and pending requests that may now be granted
    • Unlatch lock
    • Blocked transactions are notified and can proceed

**• 事务如何管理持有的锁**  
事务按照**请求顺序**维护指向所持有逻辑锁的指针。  
在事务结束时，锁按**相同顺序**逐个释放。

---

**• 释放锁的步骤**
1. **闩住锁并移除对应请求**
    - 闩住（latch）该逻辑锁结构
    - 从请求队列中**解除链接（unlink）** 对应的锁请求        
2. **更新锁状态并检查可授予的请求**
    - 遍历请求链表，**重新计算当前有效的锁模式**
    - 检查是否有因本次释放而**可被授予的等待请求**
3. **释放闩锁并通知等待事务**
    - 解锁（unlatch）逻辑锁结构        
    - **通知被阻塞的事务**，它们现在可以继续执行



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


---


这四种技术分别从不同角度优化锁机制：
1. **锁继承** → 减少锁请求开销
2. **数据导向** → 改变执行模型避免锁竞争
3. **轻量锁结构** → 降低锁管理开销
4. **早期释放** → 缩短锁持有时间
    

共同目标是：**在高并发OLTP场景下，减少锁带来的性能瓶颈**。

---


**1. 推测性锁继承（Speculative Lock Inheritance）**
- **原理**：在同一个线程中执行的后继事务，尝试继承前一个事务持有的高层级锁（如表级锁）。
- **目的**：避免重复的锁请求开销，提高线程内事务连续执行的效率。
- **应用场景**：线程池模型中，同一线程可能连续处理多个相似事务。
- **论文**：《Improving OLTP Scalability using Speculative Lock Inheritance》

---

**2. 数据导向执行（Data-oriented Execution）**
- **原理**：采用“**每个数据分区一个线程**”，而非传统的“每个事务一个线程”。
- **目的**：减少线程间锁竞争和上下文切换，提高数据局部性。
- **效果**：同一分区内的事务由同一线程串行执行，自然避免锁冲突。
- **论文**：《Data-Oriented Transaction Execution》

---

**3. 轻量级意向锁（Lightweight Intent Lock）**
- **原理**：每个事务维护**私有的锁表**，简化向全局锁表请求和释放锁的代码路径。
- **目的**：减少全局锁表的争用，提高多核环境下的并发性能。
- **现代硬件适配**：针对多核CPU缓存一致性优化，减少锁管理开销。
- **论文**：《Efficient Locking Techniques for Databases on Modern Hardware》
    

---

 **4. 早期锁释放（Early Lock Release）**

- **背景**：事务执行时间与提交时间严重不匹配：
    - 执行时间（数据在缓冲池）：约 **0.01 ms**
    - 强制提交记录到磁盘：约 **10 ms**（相差 **1000倍**）
- **原理**：事务在**为提交日志记录分配缓冲池空间后**立即释放锁，无需等待日志刷盘完成。
- **优势**：显著减少锁持有时间，提高并发吞吐量。
- **风险**：需确保事务提交的原子性和持久性不受影响（通过其他机制保证）。


## 3.6 Dealing with Deadlocks

• Deadlock prevention:
• Using timeouts: Abort transaction if it is waiting too long
• May result in false positives. How to pick timeout parameter?
• Deadlock detection:
    • T1 ➜ T2 if T1 is blocked waiting for T2 to release a lock
    • Cycles mean deadlocks
    • Roll back transaction if waits-for graph contains cycle (rollback releases locks automatically)
    • High computational overhead: Check status of all transactions and probe lock queues

**死锁预防（Deadlock Prevention）**
- **超时机制**：如果事务等待时间过长，则中止该事务。
    - **缺点**：可能导致**误判**（假阳性）——事务可能只是等待较久而非死锁。
    - **难点**：如何设定合适的超时参数？设置太短会误杀，太长则死锁响应延迟。

 **死锁检测（Deadlock Detection）**

- **等待图构建**：  
    若事务 T₁ 因等待 T₂ 释放锁而被阻塞，则添加一条边 **T₁ → T₂**。
    
- **环检测**：  
    图中存在**环**即表示死锁。
    
- **解决方式**：  
    一旦检测到环，选择一个事务进行**回滚**（回滚会自动释放其持有的锁）。
    
- **开销问题**：  
    需要检查所有事务状态并探测锁队列，**计算开销较高**。


![](image/Pasted%20image%2020250118205622.png)



**典型检测算法**
1. 定期构建**等待图（waits-for graph）**
2. 使用 DFS 或拓扑排序检测环
3. 选择“牺牲者”事务（通常基于：年龄最小、持有锁最少、已做工作最少等策略）
4. 回滚牺牲者事务并释放其资源
    

 **性能权衡**
- **检测频率高**：死锁发现快，但系统开销大
- **检测频率低**：开销小，但死锁持续时间长
- **混合策略**：根据系统负载动态调整检测频率


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

---


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
