
# 1 The Problem 
• Isolation hurts concurrency
• Blocking can heavily impact response time of transaction
• 2PL: Increases duration of holding locks increases likelihood of locking due to lock contention
• Some applications can tolerate a bit of “dirtyness”
    • Full serializability is not always needed
• Solution: Trade isolation with concurrency in a controlled manner by defining levels (degrees) of isolation


# 2 Undesired Phenomena

P1: Dirty read: A transaction T1 modifies a data item. Another transaction T2 reads the same item before T1 commits or rolls back. If T1 rolls back, T2 has read a value that never existed.

P2: Non-repeatable (fuzzy) read: T1 reads a data item. T2 modifies or deletes the data item and commits. T1 attempts to reread the data item. It discovers another value or that the item has been deleted

P3: Phantom: T1 searches using a < X < b. T2 creates some items that fall in the range (or updates items in the range so they do not qualify anymore). T1 repeats its search, and  discovers a different set of items.

| 现象                                               | 中文解释                                      | SERIALIZABLE 是否允许？ |
| ------------------------------------------------ | ----------------------------------------- | ------------------ |
| **Dirty Read（脏读）**                               | 读取了其他事务还没提交的数据。                           | ❌ 不允许              |
| **Fuzzy Read/ Non-repeatable Read（不可重复读 / 模糊读）** | 同一事务中两次读到的同一行数据不一样（因为别的事务修改了它）。           | ❌ 不允许              |
| **Phantom（幻读）**                                  | 同一事务中按条件查询的结果集的行数不同（因为别的事务插入/删除了符合条件的新行）。 | ❌ 不允许              |
# 3 Isolation Levels

• READ UNCOMMITTED: Transactions can read data that has been written by not-yet-committed transactions.
    • Allows dirty reads, fuzzy reads, phantoms
• READ COMMITTED: Transactions only read data that have been updated by committed transactions.
    • Does not allow dirty reads
    • Allows fuzzy reads, phantoms
• REPEATABLE READ: Reads to individual items are repeatable.
    • Does not allow dirty reads or fuzzy reads
    • Allows phantoms
• SERIALIZABLE: Reads by predicate search are repeatable.
    • Does not allow dirty reads, fuzzy reads, phantoms



## 3.1 SERIALIZABLE

意思是：

> 在 SERIALIZABLE 隔离级别下，  
> 按条件（谓词）搜索得到的记录（比如 `SELECT * FROM users WHERE age > 30`）  
> 在同一个事务内再次执行时，**结果是可重复的**。

也就是说：

- 如果你在一个事务中查了所有 age > 30 的用户，
    
- 另一个事务不能在你提交前插入一个新用户 `age = 35`，
    
- 否则会影响你的查询结果。
    

所以 **谓词搜索 (predicate search)** 的结果在同一事务内不会变化。


# 4 Why do we need different Isolation Levels? Why not using the highest level if it is the safest?


Using the highest isolation level, such as **Serializable**, is indeed the safest because it ensures complete isolation of transactions, preventing concurrency issues like dirty reads, non-repeatable reads, and phantom reads. However, it is not always practical or optimal to use the highest isolation level due to several reasons. Here's why we need different isolation levels:


Performance Considerations
- High isolation levels, especially **Serializable**, require more resources and can result in significant performance overhead.
- They often involve locking or other mechanisms that can cause **contention** between transactions, leading to **slower response times** or even **deadlocks** in highly concurrent environments.

Trade-off Between Consistency and Throughput
- In many applications, absolute consistency is not always necessary. For example, in a shopping cart system, slight delays in reflecting inventory changes might be acceptable if it allows for higher throughput.
- Different isolation levels allow systems to balance the need for consistency against the demand for higher transaction throughput.

Use Case-Specific Requirements
- Some applications can tolerate certain anomalies, such as **non-repeatable reads** or **phantom reads**, without impacting the correctness of the application logic.
- Lower isolation levels, such as **Read Committed** or **Read Uncommitted**, are suitable for such cases, as they allow more concurrent access and faster execution.

Resource Utilization
- Higher isolation levels require more locks or additional overhead like maintaining version chains (in multi-version concurrency control, MVCC).
- This can lead to **higher memory and CPU usage**, which might be unsustainable in systems with limited resources.

Practicality in Distributed Systems
- Achieving Serializable isolation in distributed systems is even more complex and costly due to network latency and synchronization requirements.
- Many distributed databases (e.g., NoSQL systems) relax isolation guarantees to achieve **better availability** and **partition tolerance** (as per the CAP theorem).

## 4.1 Summary of Isolation Levels and Use Cases

|**Isolation Level**|**Pros**|**Cons**|**Use Cases**|
|---|---|---|---|
|**Read Uncommitted**|Fastest; no locking|Dirty reads allowed|Logs, analytics, and applications with tolerable inconsistency|
|**Read Committed**|Prevents dirty reads|Allows non-repeatable reads|General-purpose workloads where consistency is moderately important|
|**Repeatable Read**|Prevents dirty and non-repeatable reads|Allows phantom reads|Banking systems where repeatable reads are essential|
|**Serializable**|Ensures complete isolation|High overhead, slower performance|Critical systems needing strong consistency (e.g., financial transactions)|

By using **different isolation levels**, database systems offer a way to fine-tune the balance between **data consistency**, **system performance**, and **resource efficiency**, depending on the needs of the application.


# 5 Which isolation levels protect from which concurrency problems?



| **Isolation Level**  | **Dirty Reads** | **Non-Repeatable Reads** | **Phantom Reads** | Lost Update                                |
| -------------------- | --------------- | ------------------------ | ----------------- | ------------------------------------------ |
| **Read Uncommitted** | Not prevented   | Not prevented            | Not prevented     | Not prevented                              |
| **Read Committed**   | Prevented       | Not prevented            | Not prevented     | Not prevented                              |
| **Repeatable Read**  | Prevented       | Prevented                | Not prevented     | **Prevented** (by locking rows for update) |
| **Serializable**     | Prevented       | Prevented                | Prevented         | Prevented                                  |


### 5.1.1 **Detailed Explanation**

1. **Read Uncommitted**
    - **Dirty Reads**: Allowed, as it does not enforce any locks.
    - **Non-Repeatable Reads**: Allowed, as rows can be modified during a transaction.
    - **Phantom Reads**: Allowed, as other transactions can insert/delete rows.
2. **Read Committed**
    
    - **Dirty Reads**: Prevented by ensuring that only committed changes are visible.
    - **Non-Repeatable Reads**: Allowed because rows can be modified after being read.
    - **Phantom Reads**: Allowed because no range locks are applied.
3. **Repeatable Read**
    
    - **Dirty Reads**: Prevented.
    - **Non-Repeatable Reads**: Prevented by holding read locks on rows until the transaction completes.
    - **Phantom Reads**: Allowed, as it does not lock ranges of rows.
4. **Serializable**
    
    - **Dirty Reads**: Prevented.
    - **Non-Repeatable Reads**: Prevented.
    - **Phantom Reads**: Prevented by locking the entire range of rows queried.


- **Dirty Reads**
    
    - **Definition**: Occurs when a transaction reads uncommitted changes from another transaction.
    - **Prevention**: Requires that transactions only see committed data.
    - **Isolation Levels**: Prevented in **Read Committed**, **Repeatable Read**, and **Serializable**.
- **Non-Repeatable Reads**
    
    - **Definition**: Occurs when a transaction reads the same row twice and sees different values because another transaction modified the data in between reads.
    - **Prevention**: Requires that data read by a transaction is locked to prevent modifications by other transactions.
    - **Isolation Levels**: Prevented in **Repeatable Read** and **Serializable**.
- **Phantom Reads**
    
    - **Definition**: Occurs when a transaction re-executes a query and gets a different set of rows because another transaction added or deleted rows matching the query conditions.
    - **Prevention**: Requires locking entire ranges of data or using predicate locks to ensure consistency.
    - **Isolation Levels**: Prevented only in **Serializable**.
- **Lost Updates**
    
    - **Definition**: Occurs when two transactions modify the same data simultaneously, and one transaction's update overwrites the other without being aware of it.
    - **Prevention**: Requires locking data being updated to prevent concurrent modifications.
    - **Isolation Levels**: Prevented in **Repeatable Read** and **Serializable** (via row-level locks).