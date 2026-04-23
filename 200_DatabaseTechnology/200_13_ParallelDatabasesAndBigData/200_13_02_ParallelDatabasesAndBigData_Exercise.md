

# 1 #

why parallelism
- scalability
- better performance , lower latenzcy increase the trough output
- failure recovery, fault-tolerant

Why Speedup
 - time we can save 
 - maximaize speed up sequence
 - resource limitation: super linear problem, more Cache data 

Multiprocessor: multiple processor
Multicore: multiple core in one processor
Core in one processor: can shared the data in Cache 


**为什么需要并行计算**
- **可扩展性**
- **更佳性能**：降低延迟，提高吞吐量
- **故障恢复**：容错性


**为什么追求加速**
- 节省时间
- 最大化加速比（序列执行）
- 资源限制：超线性问题，更多缓存数据


**多处理器**：多个处理器
**多核心**：一个处理器中的多个核心
**单处理器中的核心**：可以在缓存中共享数据

# 2 Amdahl's Law

## 2.1 Explain what Amdahl's Law is and provide its formula.

Solution

Amdahl's Law explains the potential speedup in latency of a task using parallel computing, based on the proportion of the task that can be parallelized. It highlights the diminishing returns of adding more processors to a system.

The speedup S of a program using parallel computing is given by: $

![](image/Pasted%20image%2020260201204243.png)

Where:

S: The theoretical speedup of the program.

P: The fraction of the program that can be parallelized.

(1 - P): The fraction of the program that is sequential and cannot be parallelized.

N: The number of processors.

---

阿姆达尔定律解释了在使用并行计算时，基于任务可并行部分的比例，所能获得的潜在延迟加速效果。该定律揭示了为系统增加更多处理器所带来的收益递减规律。

使用并行计算时，程序的加速比 S 由以下公式给出：

$$ S = \frac{1}{(1 - P) + \frac{P}{N}} $$

其中：

- **S**：程序的理论加速比。
- **P**：程序中可以并行执行的部分所占的比例。
- **(1 - P)**：程序中必须串行执行、无法并行的部分所占的比例。
- **N**：处理器的数量。


## 2.2 

A program takes 10 hours to complete when executed on a single processor.

- 60% of the program is parallelizable (𝑃 = 0.6), while the rest must be executed sequentially.
- The program is executed on a system with 4 processors (𝑁 = 4).

Calculate the theoretical speedup 𝑆 using Amdahl's Law for 4 processors.


Solution

Total execution time on 1 processor:   T_single = 10 hours
Parallelizable portion: P = 0.6
Sequential portion: (1 - P) = 0.4

![](image/Pasted%20image%2020260201204405.png)

Answer: The theoretical speedup with 4 processors is approximately 1.82.

---

Determine the new execution time for the program.

![](image/Pasted%20image%2020260201204430.png)

## 2.3 ##

If the number of processors increases to 8, what is the new theoretical speedup 𝑆?


![](image/Pasted%20image%2020260201204501.png)

Answer: The theoretical speedup with 8 processors is approximately 2.1.

## 2.4 What is the maximum possible speedup for this program, regardless of the number of processors?

![](image/Pasted%20image%2020260201204527.png)



Solution
The maximum speedup occurs when the number of processors approaches infinity 
Substitute P = 0.6:

Answer: The maximum possible speedup for this program is approximately 2.5, regardless of the number of processors.


# 3 Gustafson's Law

## 3.1 Explain what Gustafson's Law is and provide its formula.

Solution

Gustafson's Law addresses the scalability of parallel computing by considering how the problem size grows with the number of processors. The law is expressed as:

![](image/Pasted%20image%2020260201204608.png)


A program has the following characteristics:
- The parallelizable portion is P = 0.8.
- The sequential portion is (1 - P) = 0.2.

## 3.2 The program is executed on a system with N = 8 processors.

Solution

Where:
- S: The speedup of the program.
- P: The proportion of the program that can be parallelized.
- (1 - P): The proportion of the program that is sequential.
- N: The number of processors.

Gustafson's Law contrasts with Amdahl's Law by assuming that the problem size increases as the number of processors grows, enabling better utilization of additional processors.


## 3.3 Use Gustafson's Law to calculate the speedup S.

![](image/Pasted%20image%2020260201204653.png)

Answer: The speedup with 8 processors is S = 6.6.



## 3.4 Calculate the speedup S with N = 16 processors.


Solution

Substitute P = 0.8 and N = 16:
Answer: The speedup with 16 processors is S = 13.0.

![](image/Pasted%20image%2020260201204717.png)

# 4 Compare Gustafson's law with Amdahl's law on scalability.


Solution

Gustafson's Law accounts for the fact that as the number of processors increases, the problem size can also grow, meaning more of the workload becomes parallelizable. 
This contrasts with Amdahl's Law, which assumes a fixed problem size and limits the speedup due to the sequential portion.

古斯塔夫森定律考虑了这样一个事实：随着处理器数量的增加，问题规模也可以随之扩大，这意味着更多的工作负载变得可并行化。这与阿姆达尔定律形成对比，后者假设问题规模是固定的，因此加速效果受限于程序的串行部分。


# 5 Parallel Database Architecture - Shared Disk vs. Shared Nothing

## 5.1 Explain the main characteristics, advantages and disadvantages of Shared Disk and Shared Nothing architectures.

Solution

Parallel database architectures are designed to improve the performance of large-scale data processing by distributing the workload across multiple nodes.
**并行数据库架构**旨在通过将工作负载分布到多个节点上，来提升大规模数据处理的性能。

---

Shared Disk Architecture
- All nodes share access to a central storage system.
- Each node has its own CPU and memory but relies on the shared disk for data.
- Advantages:
    - Simplifies data consistency and synchronization.
    - Suitable for workloads requiring shared access to data.
    - consistency
    - no management/ write data into different replicas
- Disadvantages:
    - Disk contention can become a bottleneck.
    - Performance decreases as the number of nodes increases.

**共享磁盘架构**
- 所有节点共享对中央存储系统的访问。
- 每个节点拥有自己的CPU和内存，但依赖共享磁盘获取数据。
- **优点**：
    - 简化了数据一致性和同步。
    - 适用于需要共享数据访问的工作负载。
    - 数据一致性
    - 无需管理/将数据写入不同的副本
- **缺点**：
    - 磁盘争用可能成为瓶颈。
    - 性能随着节点数量的增加而下降。


---

Shared Nothing Architecture
- Each node has its own CPU, memory, and storage.
- Nodes communicate over a network to coordinate operations.
- Advantages:
    - Scales better by distributing both computation and storage.
    - Eliminates contention on shared resources.
    - Fault tolerant
    - most scalable
- Disadvantages:
    - Data shuffling and replication can introduce overhead.
    - More complex to manage and ensure consistency.
    - increase the network workload 

**无共享架构**
- 每个节点拥有自己的CPU、内存和存储。
- 节点通过网络通信来协调操作。
- **优点**：
    - 通过同时分布计算和存储实现更好的扩展性。
    - 消除了对共享资源的争用。
    - 容错性
    - 最具可扩展性
- **缺点**：
    - 数据混洗和复制会引入开销。
    - 管理和确保一致性更为复杂。
    - 增加网络负载
## 5.2 ##

A company is evaluating which architecture to use for their parallel database system. Two types of queries are considered:
- Query A: Reads a significant amount of shared data and requires frequent disk access.
- Query B: Performs independent computations on separate partitions of data.

The company also wants to understand how scaling the system (adding more nodes) will affect performance.

Query A: shared Disk
Query B:  shared nothing, because the computation of sequence portion is indepent to rach other 

一家公司正在评估为其并行数据库系统采用哪种架构。需考虑两种类型的查询：
- 查询 A：读取大量共享数据，且需要频繁访问磁盘。
- 查询 B：对独立的数据分区执行计算。

该公司还想了解扩展系统（增加更多节点）将如何影响性能。

查询 A：共享磁盘架构
查询 B：无共享架构，因为各个部分的计算彼此独立。


### 5.2.1 Analyze the performance of Query A on both Shared Disk and Shared Nothing architectures. Which is better suited for this query, and why?


Shared Disk Architecture:
- Since all nodes share a common disk, accessing shared data is straightforward.
- However, contention for the disk increases as more nodes try to access data simultaneously.
- The shared disk becomes a bottleneck.

Shared Nothing Architecture:
- Each node has its own disk, so shared data must be replicated or shuffled across nodes.
- This introduces additional overhead for data movement.

Conclusion:
Query A performs better on a Shared Disk Architecture because it avoids the overhead of data shuffling and replication.

**共享磁盘架构**：
- 由于所有节点共享一个共同的磁盘，访问共享数据非常直接。
- 然而，随着更多节点尝试同时访问数据，磁盘争用会增加。
- 共享磁盘会成为瓶颈。

**无共享架构**：
- 每个节点有自己的磁盘，因此共享数据必须在节点间复制或混洗。
- 这会引入额外的数据移动开销。

**结论**：
查询 A 在共享磁盘架构上表现更好，因为它避免了数据混洗和复制的开销。

### 5.2.2 Analyze the performance of Query B on both architectures. Which is better suited for this query, and why?

Solution

- Shared Disk Architecture:
    - Independent computations still require access to a shared disk.
    - Disk contention 磁盘挣用 can slow down performance as the number of nodes increases.
- Shared Nothing Architecture:
    - Each node processes its own partition of data locally, with no shared resource contention.
    - The system scales well as more nodes are added.

Conclusion:
Query B performs better on a Shared Nothing Architecture because it eliminates disk contention and leverages local storage.

## 5.3 Discuss how adding more nodes would impact the performance of each architecture.

adding more node: add more contention . hard to keep the consistent

争用（Contention）是一种随机发生的访问，即在任何时刻和任何节点之间可能发生的多个节 点竞争使用一个线路，节点在自己决定的时间自由的发送报文，...


Solution

Shared Disk Architecture:
- Adding more nodes increases CPU and memory resources but exacerbates contention for the shared disk.
- Performance gains diminish as the shared disk becomes the bottleneck.

Shared Nothing Architecture:
- Adding more nodes increases the system's ability to handle larger datasets.
- Performance improves linearly (in ideal scenarios) as each node processes its own data.

Conclusion:
Shared Nothing architecture scales better with more nodes compared to Shared Disk.


**共享磁盘架构**：
- 即使进行独立计算，仍然需要访问共享磁盘。
- 随着节点数量增加，磁盘争用可能会降低性能。

**无共享架构**：
- 每个节点在本地处理自己的数据分区，不存在共享资源争用。
- 随着更多节点的加入，系统能够很好地扩展。

**结论**：
查询 B 在无共享架构上表现更好，因为它消除了磁盘争用，并利用了本地存储的优势。

# 6 Distributed Data Storage – Horizontal vs. Vertical Partitioning

A table named Employees is given with the following schema and sample data:

![](image/Pasted%20image%2020260201205824.png)

## 6.1 ##

Data parallelism: 
- Vertical and horizontal Partitioning
- pipeline parallelism: operator parallelism

horizontal partitioning
- Hash based 
- range based
- rock and around 

## 6.2 Perform horizontal partitioning of the Employees table by range on the HireDate column:

Partition 1: Employees hired before 2020-01-01.
Partition 2: Employees hired on or after 2020-01-01.

![](image/Pasted%20image%2020260201205842.png)


## 6.3 Perform vertical partitioning of the Employees table:

Partition 1: Columns EmployeeID, Name, and Department.
Partition 2: Columns Salary and HireDate.

Solution

Note: EmployeeID is used as primary key here.

Partition 1: EmployeeID, Name, and Department
![](image/Pasted%20image%2020260201205917.png)


Partition 2: Salary and HireDate
![](image/Pasted%20image%2020260201205928.png)



## 6.4 Discuss the advantages and disadvantages of both partitioning methods for this dataset and their impact on query performance.

Horizontal Partitioning:

Advantages:
- Distributes rows based on ranges, which can improve query performance for range-based queries (e.g., hire date ranges).
- Useful for distributing data across nodes in a cluster.

Disadvantages:
- Queries that need data from multiple partitions (e.g., aggregates across all employees) can involve significant overhead.
- Partitioning by range may lead to uneven distribution if data is not uniformly distributed.

**水平分区**：

**优点**：
- 基于范围分布行，这可以提高基于范围的查询（例如，雇佣日期范围）的性能。
- 有助于在集群中的节点间分布数据。

**缺点**：
- 需要从多个分区获取数据的查询（例如，所有员工的聚合计算）可能涉及大量开销。
- 如果数据分布不均匀，按范围分区可能导致分布不均。

---

Vertical Partitioning:

Advantages:
- Improves performance for queries that access a subset of columns frequently (e.g., querying Name and Department only).
- Reduces storage requirements for partitions when certain columns are less frequently accessed.

Disadvantages:
- Joins between partitions are required when queries access columns from multiple partitions, adding overhead.
- Partitioning scheme must be carefully planned to avoid frequent joins.


**垂直分区**：

**优点**：
- 提高频繁访问列子集的查询（例如，仅查询姓名和部门）的性能。
- 当某些列访问频率较低时，可以减少分区的存储需求。

**缺点**：
- 当查询需要访问多个分区中的列时，需要在分区之间进行连接，增加了开销。
- 必须仔细规划分区方案，以避免频繁的连接操作。


## 6.5 The main challenge of distributed databse  comparing with single node data 

- Synchronisation Gap/Consistency
- node start to work / Network workload
    - commit transaction is challenging 
        - two commit phase: prepare , let it know commit, then real commit 


- **同步差距/一致性**
- **节点开始工作 / 网络负载**
    - **提交事务具有挑战性**
        - **两阶段提交**：准备阶段，通知提交，然后实际提交

---


Problem if one node fails -> blocking , another node always waiting 
address the problem:
-  blocking: set time one, choose new leader
- third phase commit: 
    - add phase: new pre commit phase: if fails, leader will handle the pre-commit 


**问题**：如果一个节点失败 -> 阻塞，其他节点一直等待

**解决问题的方法**：
- **解决阻塞问题**：设置超时时间，选举新的领导者
- **三阶段提交**：
    - **增加阶段**：新增预提交阶段：如果失败，领导者将处理预提交
