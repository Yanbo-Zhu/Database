

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

Gustafson's Law accounts for the fact that as the number of processors increases, the problem size can also grow, meaning more of the workload becomes parallelizable. This contrasts with Amdahl's Law, which assumes a fixed problem size and limits the speedup due to the sequential portion.




# 5 Parallel Database Architecture - Shared Disk vs. Shared Nothing

## 5.1 Explain the main characteristics, advantages and disadvantages of Shared Disk and Shared Nothing architectures.

Solution

Parallel database architectures are designed to improve the performance of large-scale data processing by distributing the workload across multiple nodes.

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


## 5.2 ##

A company is evaluating which architecture to use for their parallel database system. Two types of queries are considered:
- Query A: Reads a significant amount of shared data and requires frequent disk access.
- Query B: Performs independent computations on separate partitions of data.

The company also wants to understand how scaling the system (adding more nodes) will affect performance.


Query A: shared Disk
Query B:  shared nothing, because the computation of sequence portion is indepent to rach other 

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

### 5.2.2 Analyze the performance of Query B on both architectures. Which is better suited for this query, and why?

Solution

- Shared Disk Architecture:
    - Independent computations still require access to a shared disk.
    - Disk contention can slow down performance as the number of nodes increases.
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



Vertical Partitioning:

Advantages:
- Improves performance for queries that access a subset of columns frequently (e.g., querying Name and Department only).
- Reduces storage requirements for partitions when certain columns are less frequently accessed.

Disadvantages:
- Joins between partitions are required when queries access columns from multiple partitions, adding overhead.
- Partitioning scheme must be carefully planned to avoid frequent joins.



## 6.5 The main challenge of distributed databse  comparing with single node data 

- Synchronisation Gap/Consistency
- node start to work / Network workload
    - commit transaction is challenging 
        - two commit phase: prepare , let it know commit, then real commit 

Problem if one node fails -> blocking , another node always waiting 
address the problem:
-  blocking: set time one, choose new leader
- third phase commit: 
    - add phase: new pre commit phase: if fails, leader will handle the pre-commit 
