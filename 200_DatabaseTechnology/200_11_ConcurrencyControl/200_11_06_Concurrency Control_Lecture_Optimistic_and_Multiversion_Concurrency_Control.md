
# 1 Disadvantages of Locking

• Pessimistic approach to concurrency control
    • Imposes a lot of overhead, including
        • Locks for read-only transactions
        • deadlock detection
• Optimistic approaches are based on the hope that conflicts will be rare
    • Fix things if conflicts happen, do not preempt conflicts
    • If something goes wrong, abort and restart transaction

Possible solutions:
    • Timestamp ordering protocol
    • Multi-Version Concurrency Control


• **悲观并发控制方法**  
• 带来大量开销，包括：  
• 为只读事务加锁  
• 死锁检测

• **乐观方法**基于冲突稀少的假设  
• 如果发生冲突再处理，而非预先防止冲突  
• 如果出现问题，中止并重启事务

可能的解决方案：  
• 时间戳排序协议  
• 多版本并发控制（MVCC）


# 2 Time Stamp Ordering Protocol

‣ Idea: order transactions based on their timestamp
‣ Protocol has to make sure that for each item accessed by Conflicting Operations in the schedule, the order in which the item is accessed does not violate the ordering
‣ Tool: add two timestamp values to each database item:
    ‣ W_TS(X) is the largest timestamp of any transaction that executed write(X) successfully on this item
    ‣ R_TS(X) is the largest timestamp of any transaction that executed read(X) successfully on this item
    ‣ TS(Ti) => every Tx gets a timestamp based on when it enters the system

‣ **核心思想**：基于事务的时间戳对它们进行排序  
‣ **协议要求**：确保调度中每个被冲突操作访问的数据项，其访问顺序不违反时间戳顺序  
‣ **工具**：为每个数据库项添加两个时间戳值：  
‣ **W_TS(X)**：表示对该数据项成功执行 write(X) 的事务中最大的时间戳  
‣ **R_TS(X)**：表示对该数据项成功执行 read(X) 的事务中最大的时间戳  
‣ **TS(Ti)**：每个事务在进入系统时都会获得一个时间戳




# 3 Simple TSO Algorithm

该算法通过比较事务时间戳与数据项的时间戳来保证可串行化，无需加锁，但会导致某些旧事务被中止。

‣ Case 1: Whenever a Transaction T issues a write(X) operation, check the following conditions:
    ‣ If R_TS(X) > TS(T) then abort (a more recent thread is already relying on the old value)
    ‣ If W_TS(X) > TS(T), then skip (Thomas rule state a later write will anyway overwrite this so ignore it)
    ‣ Else: Execute W_item(X) operation of T and set W_TS(X) to TS(T).
‣Case 2: Whenever a Transaction T issues a read(X) operation, check the following conditions:
    ‣ If W_TS(X) > TS(T) then abort (a more recent thread has overwritten the value)
    ‣ Else: execute the R_item(X) operation of T and set R_TS(X) to the larger of TS(T) and current R_TS(X).
‣ Whenever two conflicting operations occur in incorrect order, it reject the latter of the two by aborting the transaction

 **情况 1：当事务 T 发出 write(X) 操作时：**
- **如果 R_TS(X) > TS(T)**：则**中止 T**（因为已经有一个更晚的事务读了这个值，T 的写入可能会破坏其可见性）
- **如果 W_TS(X) > TS(T)**：则**跳过这个写操作**（根据 Thomas 规则：既然后面已经有事务写了新值，T 的写就没必要执行，可直接忽略）
- **否则**：执行 T 的 W_item(X) 操作，并将 W_TS(X) 设置为 TS(T)

---

**情况 2：当事务 T 发出 read(X) 操作时：**
- **如果 W_TS(X) > TS(T)**：则**中止 T**（因为有一个更晚的事务已经修改了 X，T 应该读到的是被覆盖前的旧值，但已不存在）
- **否则**：执行 T 的 R_item(X) 操作，并将 R_TS(X) 更新为 max(TS(T), 当前 R_TS(X))

---

**核心原则：**
- 当两个冲突操作（读写、写读、写写）以**错误的时间戳顺序**发生时，系统会**拒绝后者，并中止对应的事务**，以保证调度的时间戳顺序与执行顺序一致。
- **Thomas 规则** 用于优化写操作：如果一个旧事务的写被更晚事务的写覆盖，则可安全忽略旧事务的写，避免不必要的中止。


# 4 Multiversion Concurrency Control

• Keep multiple versions of data items
• Each transaction operates on private copy of data, copies are merged later
• It can in addition support transaction-time temporal databases and time-travel queries

![](image/Pasted%20image%2020250118202446.png)




Snapshot Isolation
• A transaction T executing in SI reads data from a snapshot of the committed data as of the time the transaction started
• Start-Timestamp(T): any time before the transaction’s first read
• T’s reads are never blocked as long as the snapshot can be maintained
• T’s writes are reflected in the snapshot, and are read again
• Updates by transactions that have not committed before Start- Timestamp(T) are invisible

**快照隔离（Snapshot Isolation）**

- **工作原理**：在 SI 下执行的事务 T，会从一个**事务开始时已提交的数据快照**中读取数据。
    
- **起始时间戳**：Start-Timestamp(T) 可以是事务开始之前、第一次读操作之前的任意时刻。
    
- **读不阻塞**：只要能够维护快照，T 的读取操作永远不会被阻塞。
    
- **写可见性**：T 自己的写入会反映在快照中，并可以被自己再次读取。
    
- **其他事务更新不可见**：在 Start-Timestamp(T) 之前尚未提交的其他事务的更新，对 T 是不可见的。

快照隔离是一种多版本并发控制（MVCC）的实现方式，它让每个事务看到数据库在某一历史时刻的一致性视图，从而避免了读-写冲突和大部分阻塞，适合读多写少的场景。

---



Multiversion Concurrency Control for SI
• When T1 is ready to commit, it obtains a Commit-Timestamp
    • Larger than any other ST or CT
• T1 commits if there is no T2 with CT(T2) in [ST(T1),CT(T1)]
• “First committer wins” rule
    • It is not possible to have two concurrently active transactions that both commit and modify the same data item
• SI allows more concurrency than S2PL, especially for read-mostly transactions
• Implemented in many systems
• Algorithm can be extended to allow serializability with modest overhead
(Paper: M. J. Cahill et al. Serializable isolation for snapshot databases. SIGMOD 2008)

  

**快照隔离的多版本并发控制机制**

- **提交时间戳获取**：当 T1 准备好提交时，它会获取一个**提交时间戳（Commit-Timestamp）**
    
    - 该时间戳比任何已有的起始时间戳（ST）或提交时间戳（CT）都大
        
- **提交条件**：T1 仅在**不存在**满足 CT(T2) 在 [ST(T1), CT(T1)] 区间内的其他事务 T2 时才能提交
    
- **“先提交者获胜”规则**：
    
    - 不可能存在两个并发活跃事务同时提交并修改同一个数据项的情况
        
- **相比严格两阶段锁（S2PL）的优势**：快照隔离允许更高的并发度，尤其适合以读为主的事务
    
- **实际应用**：已在许多数据库系统中实现
    
- **扩展能力**：该算法可以扩展以支持可串行化隔离级别，且开销可控
    
    - （参考论文：M. J. Cahill 等人，《快照数据库的可串行化隔离》，SIGMOD 2008）
