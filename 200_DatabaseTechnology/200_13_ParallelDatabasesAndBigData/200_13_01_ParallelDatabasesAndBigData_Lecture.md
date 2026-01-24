

# 1 Introduction 

- One single instance of the DBMS on a single system manages all the data. 
- Query Execution is run by a single thread — no synchronization and communication necessary.
- Parallelism only on the I/O level possible through using multiple disks. 
- Only one CPU core involved in the processing.



Data volume keeps growing
- Data Warehouse sizes of about 1PB are not uncommon!
- Some businesses produce >1TB of new data per day!  
- Scientific scenarios are even larger (e.g., LHC experiment results in ~15PB / yr)

Some systems are required to support extreme throughput in transaction processing
-  Especially financial institutes

Analysis queries are becoming more and more complex
- Discovery of statistical patterns is highly compute intensive
- May require multiple passes over the data

—> Divide work over multiple machines /nodes! (Chapter 20 in textbook)

---

First Things First: Parallel vs. Distributed

- There are two basic approaches for dividing work across machines: Parallelization & Distribution
- In a parallel database, machines ...
    - ... are homogeneous, and tightly coupled (low-latency, high-bandwidth).
        - • Racks or Clusters with direct network connections.
        - • • Single multi-core / multi-processor system.
    - ... nodes work together, and cannot act autonomously.
    - —> Mainly optimized for performance
- In a distributed database, machines
    - ... can be heterogeneous, and are loosely coupled (high-latency, low-bandwidth):
        - Long network links, potentially over the Internet.  
    - • ... might act independently (e.g. to answer a local query without network cost)
    - —> Mainly optimized for reliability.



# 2 Background

## 2.1 Defining speedup  

Most important metric when discussing parallel algorithms: Speedup
Defined as:
• Runtime of the sequential program  
• Runtime of the parallel program, when run on p nodes

Describes:
how much faster we get by running the parallel algorithm on p nodes The highest achievable speedup is linear speedup:

The highest achievable speedup is linear speedup
- Doubling the number of nodes doubles the speed (halves the runtime).

---
What About Super-Linear Speedups?

![](image/Pasted%20image%2020260124123630.png)

Doubling the number of processors => Program runs more than twice as fast. Is this even possible?

It can indeed happen! Potential reasons:
Additional nodes usually also give us more memory, meaning we can keep larger parts of the data cached in memory.
In multi-core systems: Core 1 might access a piece of data, leading to it already being available in the L2 cache when Core 2 needs it later.  
In parallel dynamic programming: Threads might skip work if a different thread has already previously computed a required intermediate result.

---

## 2.2 Amdahl’s Law

The Speed Limits

The theoretically, optimally achievable speedup is linear.  
Amdahl’s Law: “The maximum speedup is determined by the non- parallelizable part of a program.”: 

P, fraction of the program that can be parallelized 1-P, fraction of the program that must be serial N, number of cores 

For problems with f=1.0 (embarrassingly parallel), we can actually achieve ideal (linear) speedup.  
However, for “real-world” problems f is usually < 1.0  
Strong scaling is defined as how the solution time varies with the number of processors for a fixed total problem size.


![](image/Pasted%20image%2020260124123700.png)


![](image/Pasted%20image%2020260124123709.png)

According to Amdahl's Law, in a best-case scenario, the theoretical speedup of a program is equal to the number of processors available.
![](image/Pasted%20image%2020260124192921.png)

## 2.3 Gustafson's Law

‣ Problem sizes tend to grow over time and with increasing available processing power

‣ Gustafson's law: More processors are usually added to solve a larger problem in the same time

‣ The larger the problem gets the larger the speed-up becomes

‣ A growing problem can be efficiently handled using parallelization

‣ Weak scaling is defined as how the solution time varies with the number of processors for a fixed problem size per processor.


![](image/Pasted%20image%2020260124123734.png)

![](image/Pasted%20image%2020260124123741.png)

---

![](image/Pasted%20image%2020260124123748.png)


# 3 Database architectures  

‣ Homogeneous distributed database:
    ‣ All sites have identical software
    ‣ Are aware of each other and agree to cooperate in processing user requests
    ‣ Each site surrenders part of its autonomy in terms of right to change schemas or software
    ‣ Appears to user as a single system  

‣ Heterogeneous distributed database:
    ‣ Different sites may use different schemas and software  
    ‣ Difference in schema is a major problem for query processing  
    ‣ Difference in software is a major problem for transaction processing
‣ Sites may not be aware of each other and may provide only limited facilities for cooperation in transaction processing

‣ **同构分布式数据库**：  
‣ 所有节点具有相同的软件  
‣ 彼此感知并同意协作处理用户请求  
‣ 每个节点在修改模式或软件的权利上放弃部分自主权  
‣ 对用户表现为单一系统

‣ **异构分布式数据库**：  
‣ 不同节点可能使用不同的模式和软件  
‣ 模式差异是查询处理的主要问题  
‣ 软件差异是事务处理的主要问题  
‣ 节点之间可能彼此不感知，在事务处理中仅提供有限的协作功能

---

- **Shared-disk**：多个处理器通过共享磁盘连接（如某些集群或 SAN 系统），不适用于单个笔记本电脑。    
- **Shared-nothing**：每个处理器拥有独立的内存和磁盘，通过网络通信（如分布式集群），不适用于多核笔记本。

## 3.1 Shared Memory
Several CPUs share a single memory space and (multiple) disks Communication over a single common bus
Usually this is single node execution. (Exception new technology disaggregated memory)

![](image/Pasted%20image%2020260124125228.png)


## 3.2 Shared Disk

Several nodes with multiple CPUs, each node has its private memory Single attached disk (array): Often NAS, SAN, etc...  
Sever share the same data „source“

For example, cloud nodes with distributed file system like S3 for Amazon
![](image/Pasted%20image%2020260124125234.png)

## 3.3 Shared Nothing
Each node has it own set of CPUs, memory and disks attached Data needs to be partitioned over the nodes  
Data is exchanged through direct node-to-node communication

![](image/Pasted%20image%2020260124125300.png)



## 3.4 Issues for Selecting the Architecture

‣ Reliability
‣ Scalability
‣ Geographic distribution of data
‣ Performance 
‣ Cost


![](image/Pasted%20image%2020260124125323.png)
# 4 Modes of Parallelism in databases 

## 4.1 Modes of Hardware Parallelism

Instruction-level Parallelism:
- Different data can be processed independently  
- Each processing unit executes the same operations on its share of the input data Example: Utilizing vector units of the CPU (SSE, MMX)  
- —> Can often be automatically introduced by the compiler, but might need manual adjustments of the code

Data Parallelism:
- Different data can be processed independently  
- Each processing unit executes the same operations on its share of the input data
- Example: Utilizing vector units of the CPU (SSE, MMX)  
- —> Can often be automatically introduced by the compiler, but might need manual adjustments of the code

Task Parallelism:
- Tasks encapsulate work and/or data  
- Tasks are distributed among the processors/nodes  
- Each processor executes a different thread/process Example: Multi-threaded programs  
- —> Typically requires specifically implemented programs


**指令级并行（Instruction-level Parallelism）**：
- 不同数据可以独立处理
- 每个处理单元在其分配的输入数据上执行相同的操作
- 示例：利用 CPU 的向量单元（SSE、MMX）
- → 通常可由编译器自动引入，但可能需要手动调整代码

**数据并行（Data Parallelism）**：
- 不同数据可以独立处理
- 每个处理单元在其分配的输入数据上执行相同的操作
- 示例：利用 CPU 的向量单元（SSE、MMX）
- → 通常可由编译器自动引入，但可能需要手动调整代码

**任务并行（Task Parallelism）**：
- 任务封装了工作和/或数据
- 任务被分配到不同的处理器/节点
- 每个处理器执行不同的线程/进程
- 示例：多线程程序
- → 通常需要专门实现的程序支持



## 4.2 Modes of Query Parallelism (Inter-Query and Intra-Query  Parallelism )



Inter-Query Parallelism (multiple concurrent queries)
- Necessary for efficient resource utilization: While one query waits (e.g. for I/O), another one executes.
- Requires concurrency control (locking mechanisms) to gurantee transactional properties (then "I" in ACID)
- —> Important for highly transactional scenarios (OLTP).

Intra-Query Parallelism (parallel processing of a single query)
- I/O Parallelism: Concurrent reading from multiple disks  Hidden: Hardware RAID
- Data Parallelism: Multiple threads work on the same operator. Example: Parallel Sort.  
- Pipeline Parallelism: Multiple pipelined parts of the plan run in parallel.  
- —> Important for complex analytical tasks (OLAP).


**查询间并行（多个并发查询）**

- **必要性**：实现高效资源利用——当一个查询等待时（例如等待I/O），另一个查询可以执行。
    
- **要求**：需要并发控制（锁机制）来保证事务属性（即ACID中的“I”——隔离性）。
    
- **→ 适用场景**：对高事务性场景（OLTP）非常重要。
    

**查询内并行（单个查询的并行处理）**

- **I/O并行**：从多个磁盘并发读取
    
    - 隐藏实现：硬件RAID
        
- **数据并行**：多个线程在同一操作符上工作。例如：并行排序。
    
- **流水线并行**：查询计划中多个流水线部分并行运行。
    
- **→ 适用场景**：对复杂分析任务（OLAP）非常重要。


## 4.3 Pipeline Parallelism (Inter-operator parallelism)

![](image/Pasted%20image%2020260124130043.png)

**操作符间并行（Inter-operator Parallelism）** 是指查询计划中的**不同操作符**同时执行。常见的实现方式包括：

1. **流水线并行（Pipeline Parallelism）**：一个操作符的输出直接作为下一个操作符的输入，两者同时运行。
    
    - 例如：`Scan → Filter → Join` 中，Scan 产生数据时 Filter 就开始处理，同时 Join 可能也在处理之前已通过 Filter 的数据。
        
2. **独立分支并行**：查询计划有多个独立分支时，可以同时执行。
    

因此，操作符间并行允许多个操作符并发运行，从而提升查询性能。


- Pipeline Parallelism is also called Inter-Operator Parallelism:
    - Inter Operator because the parallelism is between the operators
    - - Inter-operator parallelism allows multiple operators to run concurrently.
- Execute multiple pipelines simultaneously:
    - Limited in its applicability, only if multiple pipelines are present and not totally dependent on each other
    - Enables task parallelism  
- Problems
    - High synchronization overhead between the different parts of the pipeline
    - Mostly limited to lower degree of parallelism (not too many pipelines per query) 
    - Can easily lead to imbalanced parallelism (Thread 1: Build Hashtable for 1TB relation, Thread 2: Build Hashtable for 1MB relation)

- Due to these problems, Pipeline Parallelism is often only applicable to a certain, small degree


- **流水线并行也被称为操作符间并行**：
    
    - 称为“操作符间”是因为并行发生在不同的操作符之间
        
- **同时执行多个流水线**：
    
    - 适用性有限，仅当存在多个流水线且不完全相互依赖时可行
        
    - 支持任务并行
        
- **存在的问题**：
    
    - 不同流水线部分之间同步开销大
        
    - 并行度通常受限（每个查询不能有太多流水线）
        
    - 容易导致并行不均衡（例如：线程1为1TB的关系构建哈希表，线程2为1MB的关系构建哈希表）
        
- 由于这些问题，流水线并行通常只适用于较低程度的并行化


## 4.4 Data Parallelism

Data is divided into several sub-sets:
- Most operations don't need a complete view of the data
    - E.g., "Filter" looks only at a single tuple at a time
- Subsets can be processed independently and hence in parallel 

Degree of Parallelism as high as the number of possible subsets
•   For "Filter": As high as the number of tuples

Some operations possibly need a view of larger portions of the data:
- E.g. Grouping/Aggregation operation needs all tuples with the same grouping key
- Different operators need different sets

**数据被划分为多个子集：**

- 大多数操作不需要数据的完整视图
    
    - 例如：“筛选”操作一次只查看单个元组
        
- 子集可以独立处理，因此能够并行执行
    

**并行度最高可达可能子集的数量**  
• 对于“筛选”操作：最高可达元组数量

**某些操作可能需要更大范围的数据视图：**

- 例如：分组/聚合操作需要所有具有相同分组键的元组
    
- 不同操作符需要不同的数据子集

---

- Client send an SQL query to one of the cluster nodes  
    - Node becomes the "coordinator“: 
- Coordinator compiles the query: 
    - Parsing, Checking, Optimization • Parallelization
- Sends partial plans to the other cluster nodes that describes their tasks:
    - Coordinator also excutes the partial plan on his part of the data 
- Collects partial results and finalizes them (see next slide)



![](image/Pasted%20image%2020260124130256.png)

---

• For shared-nothing & shared-disk systems:
- Multiple instances of a sub-plan areexecuted on different computers
- The instances operate on different splits or partition of the data
- At some points, results from the sub-plans are collected
- For more complex queries, results are not collected but re-distributed, for further parallel processing

![](image/Pasted%20image%2020260124131127.png)

# 5 Distributed Data Storage  



Challenges for Distribution

‣ Deciding what data goes where highly depend on the access pattern of the data 两个Repliacate 先弄那个

‣ Replication: System maintains multiple copies of data, stored in different sites, for faster retrieval and fault tolerance

‣ Fragmentation: Relation is partitioned into several fragments stored in distinct sites

‣ Replication and fragmentation can be combined: Relation is partitioned into several fragments: system maintains several identical replicas of each such fragment

‣ **数据放置决策高度依赖于数据的访问模式**  
（这里原文 “两个Repliacate 先弄那个” 可能是中文输入错误，推测意思是 “决定先复制哪个数据”）

‣ **复制（Replication）**：系统在多个节点维护数据副本，以实现更快的检索和容错

‣ **分片（Fragmentation）**：关系被划分成多个片段，存储在不同节点

‣ **复制与分片可以结合使用**：关系被划分成多个片段，系统为每个片段维护多个相同的副本


----

Data Replication

‣ A relation or fragment of a relation is replicated if it is stored redundantly in two or more sites  
‣ Full replication of a relation is the case where the relation is stored at all sites  
‣ Fully redundant databases are those in which every site contains a copy of the entire database 

‣ Advantages of Replication:
    ‣ Availability: failure of site containing relation R does not result in unavailability of R is replicas exist 
    ‣ Parallelism: queries on R may be processed by several nodes in parallel  
    ‣ Reduced data transfer: relation R is available locally at each site containing a replica of R
‣Disadvantages of Replication:  
    ‣ Increased cost of updates: each replica of relation R must be updated
    ‣ Increased complexity of concurrency control: concurrent updates to distinct replicas may lead to inconsistent data unless special concurrency control mechanisms are implemented
        ‣ One solution: choose one copy as primary copy and apply concurrency control operations on primary copy


**数据复制**

‣ 如果一个关系或关系的片段冗余地存储在两个或多个节点上，则称其被复制  
‣ **完全复制**：关系被存储在所有节点  
‣ **完全冗余数据库**：每个节点都包含整个数据库的副本

‣ **复制的优点**：  
‣ **高可用性**：包含关系 R 的节点故障不会导致 R 不可用（如果存在副本）  
‣ **并行性**：对 R 的查询可以由多个节点并行处理  
‣ **减少数据传输**：每个有 R 副本的节点都可以在本地访问 R

‣ **复制的缺点**：  
‣ **更新成本增加**：关系 R 的每个副本都必须更新  
‣ **并发控制复杂性增加**：对不同副本的并发更新可能导致数据不一致，除非采用特殊的并发控制机制  
‣ 一种解决方案：选择一个副本作为**主副本**，并在主副本上执行并发控制操作




---


Data Fragmentation

- Also called Partitioning the data means creating multiple disjoint sub-sets
    - Example: Sales data, every year gets its own partition  
- For shared-nothing, data must be partitioned across nodes
- Partitioning with certain characteristics has more advantages:
    - Some queries can be limited to operate on certain sets only, if it is provable that all relevant data (passing the predicates) is in that partition  
    - Partitions can be simply dropped as a whole (data is rolled out) when it is no longer needed (e.g. discard old sales)

**数据分片**

- 也称为**数据分区**，即创建多个互不相交的子集
    
    - 示例：销售数据，每年拥有自己的分区
        
- 对于**无共享架构**，数据必须在不同节点间进行分区
    
- **具有特定特性的分区会带来更多优势**：
    
    - 如果可以证明所有相关数据（满足谓词条件的数据）都在某个分区中，那么某些查询可以仅在该分区上执行
        
    - 当分区数据不再需要时（例如丢弃旧的销售数据），可以**直接整体删除分区**（数据被移出）


## 5.1 Horizontal Partitioning

Partition table in multiple horizontal slices
Each slice has the same schema  
Each tuple is in exactly one partition  
Allow for parallel processing of partitions  
Allows to place partitions where there are most often read

![](image/Pasted%20image%2020260124131606.png)


Typical Modes of Horizontal Partitioning
Round robin:
Each set gets a tuple in a round
all sets have guaranteed equal amount of tuples 
no apparent relationship between tuples in one set

Hash Partitioned:
Define a set of partitioning columns
Generate a hash value over those columns to decide the target set  
All tuples with equal values in the partitioning columns are in the same set

- **轮循分区（Round Robin）**：确实保证每个分区获得大致相等数量的元组（严格相等，如果元组总数可被分区数整除）。
- **哈希分区（Hash-based Partitioning）**：**不保证**每个分区的元组数量相等。  
    哈希分区的分布取决于**哈希键的分布**。如果某些哈希值出现频率更高，对应分区就会包含更多元组，从而导致数据倾斜（data skew）。

Range Paritioned 
Define a set of partitioning columns
Split the domain of those columns into ranges 
The range determines the target set  
All tuples on one set are in the same range

**轮循分区（Round robin）**：
- 每个分区按轮循方式依次获得一个元组
- 保证所有分区拥有相同数量的元组
- 同一分区内的元组之间没有明显关联

**哈希分区（Hash Partitioned）**：
- 定义一组分区列
- 基于这些列生成哈希值以决定目标分区
- 所有在分区列上具有相同值的元组位于同一分区

**范围分区（Range Partitioned）**：
- 定义一组分区列
- 将这些列的值域划分为多个范围
- 根据范围决定目标分区    
- 同一分区内的所有元组属于同一值域范围


## 5.2 Vertical Partitioning

Partition table in multiple vertical slices
For reproducibility each partition needs a key (either primary or surrogate) 
Useful if full table is infrequently used  
Extreme form: column store  
Allows for processing only attributes that are required

Allows to place Attributes where they are most often read

![](image/Pasted%20image%2020260124131702.png)

  

**垂直分区（将表划分为多个垂直切片）**
- 为实现可恢复性，每个分区需要一个键（主键或代理键）
- 适用于整个表不常被完整使用的情况
- 极端形式：列存储（column store）
- 允许仅处理所需的属性列

**优势**：允许将属性放置在最常被读取的位置


## 5.3 Allocation

Assignment of partitioned table to nodes 
- Each partition can be assigned to one or more nodes
- Decision depends on relation size and access patterns

![](image/Pasted%20image%2020260124131801.png)

# 6 Distributed Transactions  

‣ Transaction may access data at several sites  
‣ Each site has a local transaction manager responsible for:
    ‣ Maintaining a log for recovery purposes  
    ‣ Participating in coordinating the concurrent execution of the transactions executing at that site

‣ Each site has a transaction coordinator, which is responsible for:
    ‣ Starting the execution of transactions that originate at the site
    ‣ Distributing sub-transactions at appropriate sites for execution
    ‣ Coordinating the termination of each transaction that originates at the site, which may result in the transaction being committed at all sites or aborted at all sites

‣ **事务可能访问多个节点的数据**  
‣ **每个节点有一个本地事务管理器，负责：**  
- 维护日志用于恢复  
- 参与协调在该节点执行的事务的并发执行

‣ **每个节点有一个事务协调器，负责：**  
- 启动在该节点发起的事务的执行  
- 将子事务分发到适当的节点执行  
- 协调在该节点发起的每个事务的终止（提交或中止），这可能导致事务在所有节点提交或所有节点中止

---
## 6.1 System Failure Modes

‣ Some failures are unique to distributed systems:
    ‣ Failure of one site
    ‣ Message loss, usually handled by TCP-IP
    ‣ Failure of communication link, usually handled by the network protocol by re-routing the messages
    ‣ Network partitioning: happens when the network is split into two or more sub-systems that lack connection between them

‣ **以下故障是分布式系统特有的：**  
- 单节点故障  
- 消息丢失（通常由 TCP/IP 协议处理）  
- 通信链路故障（通常由网络协议通过消息重路由处理）  
- 网络分区：指网络被分割成两个或多个无法相互通信的子系统


## 6.2 Distributed Commit Problem

‣ The commit now involves local parts and global coordination which makes consistency more complex

‣ Solution: Two-Phase Commit (2PC)

![](image/Pasted%20image%2020260124132425.png)

ask if it is ready for commit in paraticipated node
put the result into , the commit place in node can be aborted 

## 6.3 Two Phase commit 

![](image/Pasted%20image%2020260124132638.png)

![](image/Pasted%20image%2020260124132730.png)


Two-Phase Commit Algorithm

1. VoteCollectionPhase:Coordinator contacts all the participants by sending the value proposed by the initiator and request their response.

2. VoteCollectionPhase:Afterreceivingall the responses, the coordinator makes a decision to commit if all participants agreed upon the value or abort if someone disagrees.

3. DecisionPhase:Coordinatorcontactsall participants again and communicates the commit or abort decision.


  ![](image/Pasted%20image%2020260124132744.png)



**两阶段提交协议（2PC）的核心流程：**

1. **投票收集阶段**：协调器联系所有参与者，发送发起者提议的值，并要求参与者响应。
    
2. **投票收集阶段**：在收到所有响应后，如果所有参与者都同意该值，协调器做出提交决定；如果有人不同意，则做出中止决定。
    
3. **决策阶段**：协调器再次联系所有参与者，并通知提交或中止的决定。


# 7 Distributed Query Processing

‣ Idea: This is just an extension of centralized query processing. (System R* et al. in the early 80s)

‣ What is different?  
    ‣ Extend physical algebra: send and receive operators  
    ‣ Exploit replication  
    ‣ Consider fragmentation
    ‣ Less predictability in cost model due to unpredictable network congestion, especially if the network is not used exclusively for the DBMS

![](image/Pasted%20image%2020260124132809.png)

‣ **核心思想**：这只是**集中式查询处理的扩展**（源于 80 年代初的 System R* 等系统）

‣ **分布式查询处理的不同之处**：  
‣ **扩展物理代数**：增加发送（send）和接收（receive）操作符  
‣ **利用数据复制**  
‣ **考虑数据分片**  
‣ **成本模型可预测性降低**：由于网络拥塞难以预测，特别是当网络并非专用于 DBMS 时

## 7.1 Distributed Join Strategies


‣ R ⋈ S ⋈ T, where R is on S1, S is on S2, and T is on S3

‣ Query issued to S1 and result has to be on S1 at the end ‣ Strategy 1: Ship all to to S1

‣ Ship copies of S and T relations to site S1 and choose a strategy for processing the entire locally at site S1

‣ Strategy 2: Ship one after the other  
‣ Ship a copy of S to S3 and join S ⋈ T, then ship the result to S1 and join R ⋈ S ⋈ T

‣ Many strategies possible, they have to consider: ‣ Amount of data being shipped, size of relations ‣ Cost of shipping data between sites  
‣ Processing capabilities at each node

‣ **查询示例**：R ⋈ S ⋈ T，其中 R 在节点 S1，S 在节点 S2，T 在节点 S3  
‣ **查询发往 S1，最终结果必须返回 S1**

‣ **策略 1：全部发送到 S1**  
‣ 将关系 S 和 T 的副本发送到节点 S1，并在 S1 本地选择策略处理整个查询

‣ **策略 2：依次传递连接**  
‣ 将关系 S 的副本发送到 S3，在 S3 上计算 S ⋈ T，然后将结果发送到 S1，再计算 R ⋈ (S ⋈ T)

‣ **可能存在多种策略，需考虑以下因素**：  
‣ 传输的数据量、关系的大小  
‣ 节点间传输数据的成本  
‣ 每个节点的处理能力

## 7.2 General Transfer Strategies
Initial Placement: R1, R2 on Node 1 T1, T2, T3, T4 on Node 2

![](image/Pasted%20image%2020260124133002.png)



![](image/Pasted%20image%2020260124133038.png)


## 7.3 The concept of Locality

‣ High locality => Nodes own partitions exclusively
‣ Low locality => All nodes own equal portions of each partition


![](image/Pasted%20image%2020260124133102.png)


## 7.4 Outlook: new fast Networks

‣ Distributed DBMS Mantra: Data- Locality first!

‣ Complex partitioning schemes to leverage data-locality

‣ Complex communication avoiding schemes

‣ But: modern networks make it possible to achieve network bandwidth similar to the main memory bandwidth


![](image/Pasted%20image%2020260124133130.png)

---

![](image/Pasted%20image%2020260124133142.png)



## 7.5 Limits in Parallel Databases

Database clusters tend to scale until 64 or 128 nodes:
- Afterwards the speedup increase curve flattens
- Communication overhead eats speedup 
- Hard limit example: 1000 nodes for DB2

Shared Disk does not scale infinitely, as bus and synchronization become overhead:
- For Updates: Cache Coherency Problem
- For Reads: I/O bandwidth limits

Shared Nothing cannot compensate loss of a node easily:
- In large clusters, failures and outages are most common
- Loss of a node means loss of data 
    - Unless: Data is replicated
    - But: Replicated data must be kept consistent! Has a high overhead...


**数据库集群通常最多扩展到 64 或 128 个节点**：

- 之后加速比的增长曲线趋于平缓
    
- 通信开销抵消了加速收益
    
- 硬件限制示例：DB2 最多支持 1000 个节点
    

**共享磁盘架构无法无限扩展**，因为总线和同步会成为瓶颈：

- 对于更新：存在**缓存一致性**问题
    
- 对于读取：受 **I/O 带宽**限制
    

**无共享架构难以轻松应对节点失效**：

- 在大型集群中，故障和中断非常常见
    
- 节点丢失意味着**数据丢失**
    
    - 除非：数据被复制
        
    - 但：**复制数据必须保持一致性**！这会带来很高的开销...
