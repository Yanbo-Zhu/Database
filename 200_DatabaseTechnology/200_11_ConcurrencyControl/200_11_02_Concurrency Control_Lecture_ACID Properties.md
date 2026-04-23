


# 1 ACID Properties and transaction

Properties
• Multiple users at the same time
    • Improve utilization of resources – Multi-core processors, asynchronous I/O, multiple disks, etc.
    • Fairness to users
• Multiple users abstracted as transactions (TX)
• Transactions introduce two problems:
    • Concurrency: What happens when two transactions try to access the same object?
    • Recovery: What if the system fails in the middle of transaction execution?

Transactions
• Unit of work:
    • May contain multiple reads and writes of data
    • Manipulates internal state
    • Finishes with Commit or Abort
• Follows ACID principles
    • Must be processed as a single unit
    • Must appear to be processed in isolation
• Abort: If a transaction is aborted, i.e., it is incomplete, changes made to the database are not visible
• Commit: If a transaction is committed, i.e., it is complete, changes made to the database are visible


- **多用户同时访问**
    - 提高资源利用率——多核处理器、异步 I/O、多磁盘等
    - 对用户公平

- 多用户被抽象为**事务**

- 事务引入两个问题：
    - **并发**：当两个事务试图访问同一个对象时会发生什么？
    - **恢复**：如果系统在事务执行过程中发生故障怎么办？




- **工作单元：**
    - 可能包含多次数据的读和写
    - 操作内部状态
    - 以**提交**或**中止**结束

- **遵循 ACID 原则**
    - 必须作为一个整体单元处理
    - 必须在处理时看起来是隔离的

- **中止**：如果事务被中止，即未完成，其对数据库所做的更改不可见
- **提交**：如果事务被提交，即已完成，其对数据库所做的更改可见


## 1.1 ACID Properties

The **ACID Principle** outlines the core properties that transactions in database systems must adhere to in order to ensure data consistency and reliability. ACID stands for:

Benefits of the ACID Principle
- Ensures data integrity and security.
- Increases the reliability of database systems, especially in critical applications such as banking or inventory management.

Challenges
- Enforcing the ACID principle can reduce database performance (e.g., due to locking or synchronization).
- Modern distributed systems and NoSQL databases often follow the **BASE Principle** instead, which relaxes some ACID guarantees for better scalability and flexibility.

• Atomicity: Either all operations of a transaction complete or none of them complete (“all or nothing”) — Recovery
• Consistency: A transaction that is applied to a consistent database produces a consistent database — Integrity constraints
• Isolation: A transaction executes as if it is the only transaction running in the system — Concurrency control
• Durability: The effects of committed transactions are reflected in the database even after failures — Recovery


**ACID 原则**概述了数据库系统中的事务必须遵循的核心属性，以确保数据的一致性和可靠性。ACID 代表：

**ACID 原则的优点**
- 确保数据的完整性和安全性。
- 提高数据库系统的可靠性，特别是在银行或库存管理等关键应用中。

**挑战**
- 实施 ACID 原则可能会降低数据库性能（例如，由于锁定或同步）。
- 现代分布式系统和 NoSQL 数据库通常遵循 **BASE 原则**，该原则放宽了部分 ACID 保证，以获得更好的可扩展性和灵活性。

- **原子性（Atomicity）**：保证事务要么全做要么全不做，避免上面“只扣款不加款”的情况。
- **一致性（Consistency）**：**事务开始前**，数据库处于一个**一致性状态**（满足所有定义的规则）。**事务结束后**，数据库必须进入**另一个一致性状态**。
- **隔离性（Isolation）**：防止其他事务在中间状态读取数据（如读到总余额1300）。
- **持久性（Durability）**：事务提交后，一致性状态会持久保存。

![](image/Pasted%20image%2020250118195252.png)


### 1.1.1 Atomicity -- All or Nothing

A transaction is executed completely or not at all. There are no partial results.
- **Example**:  
    When transferring money from account A to account B, **both actions** (deducting from A and adding to B) must be completed fully, or neither should happen if an error occurs.

Can be dealt with by undoing the effects of failed transactions during recovery

![](image/Pasted%20image%2020250118195544.png)



### 1.1.2 Consistency - Integrity Constraints

After a transaction is completed, the database must transition from one consistent state to another. Rules like integrity constraints must never be violated.

**Example**:  
During a money transfer, the total balance across both accounts must remain the same before and after the transaction.

From a consistent state to a consistent state “Consistent” defined declaratively as integrity constraints

![](image/Pasted%20image%2020250118195623.png)

有一个**完整性约束**：  
**所有账户的总余额在转账前后必须保持不变**。这时数据库就处于不一致状态，而且这个不一致是**持久化**的。
1. **数据库系统**：保证内置约束（主键、外键、非空等）不被违反。  
    → 如果违反，DBMS会自动回滚事务。
2. **应用程序/开发者**：保证业务逻辑约束（如总金额不变）在事务中正确实现。  
    → DBMS 不会自动检查这类约束，需要开发者通过代码确保。


### 1.1.3 **Isolation**

Every transcation is excuted indepenedent 

Multiple transactions running concurrently should not interfere with each other. Each transaction should execute as if it were the only one running.
- **Example**:  
    If two users update the same stock inventory simultaneously, the updates must not conflict with one another.

![](image/Pasted%20image%2020250118195631.png)


### 1.1.4 **Durability** - Recovery

Transactions that reported that they finished must have their results durable.

Once a transaction is committed, its changes must be permanently stored in the database, even in the event of a system crash.

**Example**:  
After completing a money transfer and displaying confirmation, the data must remain safe even if there is a power outage.

![](image/Pasted%20image%2020250118195653.png)

# 2 BASE Principle

The **BASE Principle** is an alternative to the ACID Principle, often used in distributed and NoSQL databases to achieve high availability and scalability. BASE is designed for systems where perfect consistency (as in ACID) can be relaxed in favor of better performance and resilience. BASE stands for:

**BASE 原则**是 ACID 原则的替代方案，常用于分布式和 NoSQL 数据库，以实现高可用性和可扩展性。BASE 专为可以放宽完美一致性（如 ACID 中要求的那样）以换取更好性能和弹性的系统而设计。BASE 代表：

---

 **Basically Available**

The system guarantees availability, meaning it responds to every request, even if the response might not reflect the most recent data.

- **Example**:  
    A web application allows users to read and write data even during a system partition, but some data may be stale or inconsistent temporarily.



 **基本可用**

系统保证可用性，即它对每个请求都有响应，即使响应可能未反映最新的数据。

- **示例**：  
    即使在网络分区期间，一个 Web 应用程序也允许用户读写数据，但某些数据可能暂时过时或不一致。

---

 **Soft State**

The state of the system may change over time, even without additional input. This happens due to eventual consistency, where the database synchronizes its state across nodes asynchronously.

- **Example**:  
    In a distributed shopping cart system, changes made on one node may take time to propagate to other nodes.


 **软状态**

系统的状态可能随时间变化，即使没有额外输入。这是由于最终一致性造成的，数据库在不同节点之间异步同步其状态。

- **示例**：  
    在一个分布式购物车系统中，在一个节点上所做的更改可能需要时间才能传播到其他节点。



---

 **Eventually Consistent**

The system does not enforce strict consistency after every transaction. However, given enough time and the absence of new updates, all nodes will converge to the same state.

- **Example**:  
    Updates to a user’s profile might take some time to appear on all servers, but eventually, all servers will reflect the same data.


 **最终一致**

系统不在每次事务后强制实施严格的一致性。但是，只要有足够的时间并且没有新的更新，所有节点最终将收敛到相同的状态。

- **示例**：  
    对用户个人资料的更新可能需要一些时间才能出现在所有服务器上，但最终所有服务器都将反映相同的数据。


---

Benefits of the BASE Principle
- **High Scalability**: Optimized for large-scale, distributed systems.
- **High Availability**: Systems can continue to operate even during network partitions.
- **Flexibility**: Allows trade-offs between consistency and performance based on application needs.

Challenges
- **Eventual Consistency**: Not suitable for applications requiring strict consistency, like banking.
- **Complexity**: Developers must handle potential inconsistencies and conflicts in the application logic.

BASE 原则的优点
- **高可扩展性**：为大规模分布式系统进行了优化。
- **高可用性**：即使在网络分区期间，系统也能继续运行。
- **灵活性**：允许根据应用需求在一致性和性能之间进行权衡。

挑战
- **最终一致性**：不适合需要严格一致性的应用，如银行业务。
- **复杂性**：开发人员必须在应用逻辑中处理潜在的不一致和冲突。


## 2.1 BASE vs. ACID

|**Aspect**|**ACID**|**BASE**|
|---|---|---|
|Consistency|Strong|Eventual|
|Availability|May sacrifice availability|High availability|
|Use Case|Relational databases, financial apps|Distributed systems, NoSQL databases|

BASE is particularly popular in systems like **Cassandra**, **DynamoDB**, and **MongoDB**, where the focus is on handling massive data and traffic with fault tolerance.

