
# 1 The Problem 
• Isolation hurts concurrency
• Blocking can heavily impact response time of transaction
• 2PL: Increases duration of holding locks increases likelihood of locking due to lock contention
• Some applications can tolerate a bit of “dirtyness”
    • Full serializability is not always needed
• Solution: Trade isolation with concurrency in a controlled manner by defining levels (degrees) of isolation


 **核心问题**

- **隔离性损害并发性**  
    严格的隔离级别（如可串行化）通过锁机制防止数据异常，但限制了并发操作。
    
- **阻塞严重影响事务响应时间**  
    锁竞争导致事务长时间等待，延长响应时间。
    
- **两阶段锁（2PL）加剧锁持有时间**  
    2PL 要求在事务结束前持续持有锁，增加了锁竞争的可能性。
    

 **现实需求**
- **并非所有应用都需要完全的可串行化**  
    某些应用可以容忍一定程度的“数据不一致”（如一些读多写少的分析场景）。
- **灵活性与性能需求**  
    在不同业务场景下，可以在**正确性**和**性能**之间进行权衡。
    

**解决方案：隔离级别**

通过定义不同的**隔离级别（Degrees of Isolation）**，在**可控范围内**用隔离性换取并发性：
1. 降低隔离级别 → 提高并发性（减少锁竞争）
2. 提高隔离级别 → 增强隔离性（减少数据异常）
    

这样，应用程序可以根据自身对数据一致性的要求选择合适的隔离级别，在保证业务正确性的同时优化性能。



# 2 Undesired Phenomena

P1: Dirty read: A transaction T1 modifies a data item. Another transaction T2 reads the same item before T1 commits or rolls back. If T1 rolls back, T2 has read a value that never existed.
P2: Non-repeatable (fuzzy) read: T1 reads a data item. T2 modifies or deletes the data item and commits. T1 attempts to reread the data item. It discovers another value or that the item has been deleted
P3: Phantom: T1 searches using a < X < b. T2 creates some items that fall in the range (or updates items in the range so they do not qualify anymore). T1 repeats its search, and  discovers a different set of items.

| 现象                                               | 中文解释                                                                  |
| ------------------------------------------------ | --------------------------------------------------------------------- |
| **Lost Updates**                                 | 两个事务同时处理同一个数据， 两房都不知道对方， 一个事物更新了数据，复写了另一个事物的update. 另一个事务的 update被丢失了 |
| **Dirty Read（脏读）**                               | 读取了其他事务还没提交的数据。                                                       |
| **Fuzzy Read/ Non-repeatable Read（不可重复读 / 模糊读）** | 同一事务中两次读到的同一行数据不一样（因为别的事务修改了它）。                                       |
| **Phantom（幻读）**                                  | 同一事务中按条件查询的结果集的行数不同（因为别的事务插入/删除了符合条件的新行）。                             |

- **脏读**：读到未提交的数据（可能被回滚）
- **不可重复读**：同一行数据在事务内两次读取结果不同（被其他事务修改）
- **幻读**：同一范围查询在事务内两次返回不同行数（被其他事务插入/删除）

- **Lost Updates**
    - **Definition**: Occurs when two transactions modify the same data simultaneously, and one transaction's update overwrites the other without being aware of it.
    - **Prevention**: Requires locking data being updated to prevent concurrent modifications.
    - **Isolation Levels**: Prevented in **Repeatable Read** and **Serializable** (via row-level locks).
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



## 2.1 The Interleaving Problem

Interleaved execution of transactions may cause inconsistencies
• Which interleaved executions should be allowed?


![](image/Pasted%20image%2020250118195856.png)


serial Execution: t1先, 在t2
interleaved Execution: t1 pause, t2. t1 


## 2.2 **Lost Update**


![](image/Pasted%20image%2020250118195834.png)


Eine Transaktion überschreibt das Update der anderen Transaktion.

In a **Lost Update** scenario, two transactions concurrently modify the same data, and the update made by one transaction is overwritten by the other transaction, causing one of the updates to be lost. This happens when the transactions read and modify the same value without proper isolation.


Ergebnis
Das Ergebnis einer Transaktion wird durch die andere Transaktion verloren.
- **Reihenfolge**:
    - TA1: `10 - 4 = 6` (wird geschrieben).
    - TA2: `10 + 5 = 15` (überschreibt 6).
- Endwert: **15** (falsch, da das Update von TA1 verloren geht).


Scenario
We have a table `Inventory` that stores the number of items in stock.

Initial Data:
- Table: `Inventory`
    - `ItemID = 1, Stock = 10`

Two transactions are trying to update the `Stock` value for `ItemID = 1`:
- **Transaction A (TA1)** wants to decrease the stock by 4.
- **Transaction B (TA2)** wants to increase the stock by 5.

Breakdown:
1. **Transaction A** (TA1) starts and reads the current stock value from the table (which is `10`):
    1. `SELECT Stock FROM Inventory WHERE ItemID = 1; -- Result: Stock = 10`
2. **Transaction B** (TA2) starts and also reads the current stock value from the table (which is still `10`):
    1. `SELECT Stock FROM Inventory WHERE ItemID = 1; -- Result: Stock = 10`
3. **Transaction A** (TA1) decreases the stock by 4 and updates the table with the new value `6`:
    1. `UPDATE Inventory SET Stock = 6 WHERE ItemID = 1;`
4. **Transaction B** (TA2) increases the stock by 5 and updates the table with the new value `15`:
    1. `UPDATE Inventory SET Stock = 15 WHERE ItemID = 1;`

----

What Happens:
- **Transaction A** (TA1) writes `6` to the `Stock` column after subtracting 4 from 10.
- **Transaction B** (TA2) writes `15` to the `Stock` column after adding 5 to 10.
- Since **Transaction B** writes its result last, the update from **Transaction A** is **lost**, and the final `Stock` value will be `15` instead of the correct value (`6` + `5` = `11`).

Final Result:
- **Endwert**: **15** (The result of TA1 is lost, and TA2's update overwrites it.)

Key Problem
- **Lost Update** occurs because **Transaction A**'s update is overwritten by **Transaction B**'s update. The update by TA1 (`6`) is lost because TA2 (`15`) was written later without considering the first update.

Solution
To prevent a **Lost Update**, the system should use proper **transaction isolation**. This can be achieved by using higher isolation levels like **Serializable** or **Repeatable Read**, which will prevent both transactions from reading and modifying the same data concurrently.

Alternatively, **optimistic concurrency control** can be used, where the system checks if the data was modified by another transaction before committing an update. If the data was changed, the transaction can be aborted or retried.


## 2.3 Dirty Read

![](image/Pasted%20image%2020250118195822.png)

R: read the value of a , write the value intp t 


Eine Transaktion liest einen Wert, der von einer anderen Transaktion noch nicht festgeschrieben wurde.

A **Dirty Read** occurs when one transaction reads data that has been written by another transaction, but the second transaction has not yet committed. If the second transaction rolls back, the data read by the first transaction becomes invalid or "dirty."

- **Ergebnis**:  
    TA2 liest den Zwischenwert von TA1, bevor TA1 abgeschlossen ist.
    - **Reihenfolge**:
        - TA1: `10 - 4 = 6` (Zwischenwert)
        - TA2 liest `6 + 5 = 11`.
    - Endwert: **11** (falsch, da TA1 möglicherweise zurückgerollt wird).

Scenario:
We have a table `Account` that stores balances for different customers.

Initial Data:
- Table: `Account`
    - `AccountID = 1, Balance = 1000`

Two transactions are working on the same data:
- **Transaction A (TA1)** wants to decrease the balance by 200.
- **Transaction B (TA2)** wants to increase the balance by 300.

---

Breakdown:
1. Transaction A (TA1) starts and reads the current balance for AccountID = 1 (which is 1000):
    1.  `SELECT Balance FROM Account WHERE AccountID = 1; -- Result: Balance = 1000`
2. **Transaction B** (TA2) starts and updates the balance by increasing it by `300`, but hasn't yet committed the change:
    1. `UPDATE Account SET Balance = 1300 WHERE AccountID = 1; -- The balance is now 1300, but TA2 has not yet committed.`
3. **Transaction A** (TA1), still running, reads the balance again after the update by **Transaction B**, but **TA1** has not seen the `commit` of **TA2**. It reads the **dirty value** (`1300`):
    1. `SELECT Balance FROM Account WHERE AccountID = 1; -- Result: Balance = 1300 (even though Transaction B has not committed yet) `
4. **Transaction B** (TA2) rolls back the transaction due to some issue or error:
    1. `ROLLBACK; -- The balance reverts to the original value of 1000.`
5. **Transaction A** (TA1) proceeds with its operation, assuming the balance is 1300, and decreases it by 200:
    1. `UPDATE Account SET Balance = 1100 WHERE AccountID = 1;`

---

What Happens:
- **Transaction A** (TA1) performs a **Dirty Read** by reading the balance value `1300` that was written by **Transaction B** (TA2) before **TA2** had committed.
- Since **Transaction B** rolled back, the **actual balance** is still `1000`, but **Transaction A** operates as if the balance is `1300`, leading to incorrect updates.

Final Result:
- **Endwert**: `1100` (which is incorrect because **Transaction A** worked with invalid data).

Key Problem:
- **Dirty Read** occurs because **Transaction A** reads uncommitted data from **Transaction B**, which later rolled back. The data that **Transaction A** relied on was not valid.

Solution:
To avoid **Dirty Reads**, the database should use the **Read Committed** isolation level or higher, which ensures that a transaction can only read data that has been committed by other transactions. This way, **Transaction A** would not have read the uncommitted `1300` value from **Transaction B**.


## 2.4 **Non-Repeatable Read**

A **Non-Repeatable Read** occurs when a transaction reads the same data multiple times, but the value has been changed by another transaction in between those reads. This creates an inconsistency where the same data is read twice with different results.

每次transcation 读取同一个参数的值不一样, 因为中途被改变了

Eine Transaktion liest einen Wert mehrfach, aber der Wert wurde zwischen den Lesevorgängen durch die andere Transaktion verändert.

- **Ergebnis**:  
    TA2 liest den ursprünglichen Wert `10` und addiert 5, während TA1 zwischenzeitlich den Wert verändert hat.
    - **Reihenfolge**:
        - TA2 liest `10`.
        - TA1: `10 - 4 = 6`.
        - TA2: `10 + 5 = 15`.
    - Endwert: **15** (falsch, da TA2 den ursprünglichen Wert statt den aktualisierten Wert verwendet).

--- 

Example of Non-Repeatable Read:

A Non-Repeatable Read occurs when a transaction reads the same data multiple times, but the value has been changed by another transaction in between those reads. This creates an inconsistency where the same data is read twice with different results.

 Scenario:
- **Transaction A** reads the value of `StockQuantity` from a product and starts processing.
- **Transaction B** updates the `StockQuantity` after Transaction A's first read but before Transaction A's second read.

Initial Data:
- `StockQuantity` = 10

Step-by-Step Breakdown:
1. **Transaction A** reads the value of `StockQuantity` (which is 10).
    - **Transaction A (Step 1)**:  
        Reads `StockQuantity = 10`.
2. **Transaction B** updates `StockQuantity` (adds 5).
    - **Transaction B (Update)**:  
        `StockQuantity` is updated from 10 to 15.
3. **Transaction A** reads the value of `StockQuantity` again (but it now reads 15, which is different from its first read).
    - **Transaction A (Step 2)**:  
        Reads `StockQuantity = 15`.



Problem:
- **Transaction A** performed two reads of `StockQuantity`, expecting it to remain the same between reads. However, the value of `StockQuantity` changed after the first read due to **Transaction B**.
- The data was inconsistent for **Transaction A**, which is the essence of a **Non-Repeatable Read**.


Possible Result:
- **Transaction A** initially saw `StockQuantity = 10` and expected that same value for its second read, but it now sees `StockQuantity = 15`. This inconsistency can lead to errors, as **Transaction A** was not able to "repeat" its read with the same result.

This problem can be avoided by using higher isolation levels, such as **Serializable** or **Repeatable Read**, which prevent the underlying data from being changed by other transactions during the execution of the transaction.


## 2.5 **Phantom Read**

Hier treten Phantom-Werte auf, wenn neue Zeilen hinzugefügt oder gelöscht werden. Dies betrifft aber in diesem Fall keine Zeilenoperationen, daher ist **Phantom Read** nicht relevant.

A **Phantom Read** occurs when a transaction reads a set of rows that match a certain condition, but another transaction concurrently inserts, deletes, or updates rows that cause the result set of the first transaction to change unexpectedly before it completes.

---

Scenario:
- **Transaction A** performs a query to count how many products have a `Price` greater than 100.
- **Transaction B** concurrently inserts new products with a `Price` greater than 100, affecting the result of **Transaction A**'s query.

Initial Data:
- Product table:
    - `ProductID: 1, Price: 50`
    - `ProductID: 2, Price: 120`
    - `ProductID: 3, Price: 80`

Step-by-Step Breakdown:
1. **Transaction A** starts and executes the following query:
    1. `SELECT COUNT(*) FROM Products WHERE Price > 100;
    2. **Transaction A (Step 1)**:   The result set at this point contains only one product with `Price = 120` (i.e., Product 2). Therefore, the query returns `1`.
2. **Transaction B** concurrently inserts a new product with `Price = 150`:
    1. **Transaction B (Insert)**: `INSERT INTO Products (ProductID, Price) VALUES (4, 150);`
3. **Transaction A** executes the same query again, expecting the same result as the first time, but now the result set includes the new product inserted by **Transaction B**.
    1. **Transaction A (Step 2)**: The query now returns `2` because the newly inserted product (Product 4) also has a `Price > 100`.

---

Problem:
- **Transaction A**'s query initially returned `1` but later returned `2`, even though the same condition (`Price > 100`) was applied both times.
- The data has changed in between the two queries due to **Transaction B**'s insert, causing a **Phantom Read**.

Result:
- **Transaction A** sees inconsistent results because the set of rows that match its query criteria has changed between its reads. The appearance of new rows (the "phantoms") is the essence of a **Phantom Read**.

Solution:
- **Phantom Reads** can be avoided by using higher isolation levels such as **Serializable**, which locks the range of data being queried and prevents new rows from being added or removed during the transaction.


# 3 Isolation Levels

- **高并发、可容忍临时不一致** → READ COMMITTED
- **需要事务内读一致性** → REPEATABLE READ
- **强一致性要求（如金融交易）** → SERIALIZABLE
- **极少使用** → READ UNCOMMITTED（风险高）

**典型隔离级别**（从低到高）：

- **读未提交（Read Uncommitted）**：并发性最高，隔离性最低
- **读已提交（Read Committed）**：平衡选择
- **可重复读（Repeatable Read）**：较高隔离性
- **可串行化（Serializable）**：隔离性最高，并发性最低


---

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

**1. 读未提交（READ UNCOMMITTED）**
- **定义**：事务可以读取其他**未提交事务**写入的数据。
- **适用场景**：对数据一致性要求极低，追求最大并发的场景（如统计分析近似值）。

**2. 读已提交（READ COMMITTED）**
- **定义**：事务只能读取**已提交事务**更新的数据。
- **最常见级别**：多数数据库的默认隔离级别（如 Oracle、PostgreSQL）。
    

 **3. 可重复读（REPEATABLE READ）**
- **定义**：对**单个数据项**的多次读取结果一致。
- **实现方式**：通常通过行级锁保持读一致性（MySQL InnoDB 默认级别）。

**4. 可串行化（SERIALIZABLE）**
- **定义**：基于**谓词（范围）查询**的多次读取结果一致。
- **实现方式**：通过范围锁、表锁或乐观并发控制实现完全隔离。




# 4 Which isolation levels protect from which concurrency problems?

| **Isolation Level**  | Lost Update                                | **Dirty Reads** | **Non-Repeatable Reads/Fuzzy Read** | **Phantom Reads** |
| -------------------- | ------------------------------------------ | --------------- | ----------------------------------- | ----------------- |
| **Read Uncommitted** | prevented                                  | Not prevented   | Not prevented                       | Not prevented     |
| **Read Committed**   | Not prevented                              | Prevented       | Not prevented                       | Not prevented     |
| **Repeatable Read**  | **Prevented** (by locking rows for update) | Prevented       | Prevented                           | Not prevented     |
| **Serializable**     | Prevented                                  | Prevented       | Prevented                           | Prevented         |

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

## 4.1 SERIALIZABLE

意思是：

> 在 SERIALIZABLE 隔离级别下，  
> 按条件（谓词）搜索得到的记录（比如 `SELECT * FROM users WHERE age > 30`）  
> 在同一个事务内再次执行时，**结果是可重复的**。

也就是说：

- 如果你在一个事务中查了所有 age > 30 的用户，
    
- 另一个事务不能在你提交前插入一个新用户 `age = 35`，
    
- 否则会影响你的查询结果。
    

所以 **谓词搜索 (predicate search)** 的结果在同一事务内不会变化。


# 5 Why do we need different Isolation Levels? Why not using the highest level if it is the safest?


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

## 5.1 Summary of Isolation Levels and Use Cases

|**Isolation Level**|**Pros**|**Cons**|**Use Cases**|
|---|---|---|---|
|**Read Uncommitted**|Fastest; no locking|Dirty reads allowed|Logs, analytics, and applications with tolerable inconsistency|
|**Read Committed**|Prevents dirty reads|Allows non-repeatable reads|General-purpose workloads where consistency is moderately important|
|**Repeatable Read**|Prevents dirty and non-repeatable reads|Allows phantom reads|Banking systems where repeatable reads are essential|
|**Serializable**|Ensures complete isolation|High overhead, slower performance|Critical systems needing strong consistency (e.g., financial transactions)|

By using **different isolation levels**, database systems offer a way to fine-tune the balance between **data consistency**, **system performance**, and **resource efficiency**, depending on the needs of the application.


