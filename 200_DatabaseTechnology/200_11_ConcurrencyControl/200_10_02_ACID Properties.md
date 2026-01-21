


# 1 ACID Properties and transaction


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



![](image/Pasted%20image%2020250118195252.png)


### 1.1.1 Atomicity -- All or Nothing
A transaction is executed completely or not at all. There are no partial results.
- **Example**:  
    When transferring money from account A to account B, **both actions** (deducting from A and adding to B) must be completed fully, or neither should happen if an error occurs.

Can be dealt with by undoing the effects of failed transactions during recovery

![](image/Pasted%20image%2020250118195544.png)



### 1.1.2 Consistency - Integrity Constraints



After a transaction is completed, the database must transition from one consistent state to another. Rules like integrity constraints must never be violated.
- **Example**:  
    During a money transfer, the total balance across both accounts must remain the same before and after the transaction.

From a consistent state to a consistent state “Consistent” defined declaratively as integrity constraints

![](image/Pasted%20image%2020250118195623.png)



### 1.1.3 **Isolation**

Every transcation is excuted indepenedent 

Multiple transactions running concurrently should not interfere with each other. Each transaction should execute as if it were the only one running.
- **Example**:  
    If two users update the same stock inventory simultaneously, the updates must not conflict with one another.

![](image/Pasted%20image%2020250118195631.png)


### 1.1.4 **Durability** - Recovery



Transactions that reported that they finished must have their results durable.

Once a transaction is committed, its changes must be permanently stored in the database, even in the event of a system crash.
- **Example**:  
    After completing a money transfer and displaying confirmation, the data must remain safe even if there is a power outage.

![](image/Pasted%20image%2020250118195653.png)

# 2 BASE Principle

The **BASE Principle** is an alternative to the ACID Principle, often used in distributed and NoSQL databases to achieve high availability and scalability. BASE is designed for systems where perfect consistency (as in ACID) can be relaxed in favor of better performance and resilience. BASE stands for:

---

 **Basically Available**

The system guarantees availability, meaning it responds to every request, even if the response might not reflect the most recent data.

- **Example**:  
    A web application allows users to read and write data even during a system partition, but some data may be stale or inconsistent temporarily.

---

 **Soft State**

The state of the system may change over time, even without additional input. This happens due to eventual consistency, where the database synchronizes its state across nodes asynchronously.

- **Example**:  
    In a distributed shopping cart system, changes made on one node may take time to propagate to other nodes.

---

 **Eventually Consistent**

The system does not enforce strict consistency after every transaction. However, given enough time and the absence of new updates, all nodes will converge to the same state.

- **Example**:  
    Updates to a user’s profile might take some time to appear on all servers, but eventually, all servers will reflect the same data.

---

Benefits of the BASE Principle
- **High Scalability**: Optimized for large-scale, distributed systems.
- **High Availability**: Systems can continue to operate even during network partitions.
- **Flexibility**: Allows trade-offs between consistency and performance based on application needs.

Challenges
- **Eventual Consistency**: Not suitable for applications requiring strict consistency, like banking.
- **Complexity**: Developers must handle potential inconsistencies and conflicts in the application logic.


BASE vs. ACID

|**Aspect**|**ACID**|**BASE**|
|---|---|---|
|Consistency|Strong|Eventual|
|Availability|May sacrifice availability|High availability|
|Use Case|Relational databases, financial apps|Distributed systems, NoSQL databases|

BASE is particularly popular in systems like **Cassandra**, **DynamoDB**, and **MongoDB**, where the focus is on handling massive data and traffic with fault tolerance.

