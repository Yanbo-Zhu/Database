
Why Do We Need Recovery?
It ensures database consistency, atomicity, and durability despite failures.  i.e., It attempts to reconstruct a state of the data after a failure

Recall: transactions introduce two problems:
•Concurrency: What happens when two transactions try to access the same object?
•Recovery: What if the system fails in the middle of execution?


# 1 Issues  

## 1.1 Types of Failure

Transaction failure:
- Logical Errors: A transaction cannot complete due to some internal error condition (e.g., integrity, constraint violation).
- Internal State Errors: The DBMS must terminate an active transaction due to an error condition (e.g., deadlock)

Media failure:
- Non-Repairable Hardware Failure: A head crash or similar disk failure destroys all or parts of non-volatile storage. Recover by using a backup of the DBMS.

System failure:
- Software Failure: There is a problem with the DBMS implementation (e.g., uncaught divide-by-zero exception), and the system has to halt.
- Hardware Failure: The computer hosting the DBMS crashes (e.g., the power plug gets pulled). We assume that non-volatile storage contents are not corrupted by a system crash.

**事务故障：**
- **逻辑错误**：由于某些内部错误条件（例如完整性约束违反），事务无法完成。
- **内部状态错误**：DBMS 必须因错误条件（例如死锁）而终止活动事务。

**介质故障：**
- **不可修复的硬件故障**：磁头损坏或类似的磁盘故障破坏了非易失性存储器的全部或部分内容。通过使用 DBMS 的备份进行恢复。

**系统故障：**
- **软件故障**：DBMS 实现存在问题（例如未捕获的除零异常），系统必须停止运行。
- **硬件故障**：运行 DBMS 的计算机崩溃（例如电源被拔掉）。我们假设系统崩溃不会损坏非易失性存储器的内容。



## 1.2 Buffer Pool Management Policies

‣ General problem: The buffer manager holds pages in memory that may diverge from the page content on disk during the crash
‣ To handle the problem, the DBMS has to ensure the following guarantees:
‣ The changes for any transaction are durable once the DBMS has told somebody that it committed everyehg upto e commit should
‣ No partial changes are durable if the transaction aborted

‣ **核心问题**：崩溃发生时，缓冲管理器在内存中持有的页面内容可能与磁盘上的页面内容不一致。
‣ **为解决此问题，DBMS 必须确保以下保证**：  
‣ **持久性保证**：一旦 DBMS 通知外部某个事务已提交，该事务的所有更改就必须是持久的。  
‣ **原子性保证**：如果事务中止，则其任何部分更改都不应是持久的。

## 1.3 Buffer Manager Policies

‣ The steal policy dictates whether the DBMS allows an uncommitted transaction to write to disk (push dirty pages if memory is full)
‣ Steal: is allowed  
‣ No-Steal: is not allowed (enforces atomicity)

‣ The force policy dictates whether the DBMS requires that all updates made by a transaction are reflected on disk before the transaction is allowed to commit.  
‣ Force: is required (enforces durability)  
‣ No-Force: is not required (pages may be written later)


‣ **Steal 策略** 决定 DBMS 是否允许未提交的事务将数据写入磁盘（在内存满时强制写出脏页）  
‣ **Steal**：允许（可能破坏原子性，需要额外的恢复机制）  
‣ **No-Steal**：不允许（强制保证原子性）

‣ **Force 策略** 决定 DBMS 是否要求事务的所有更新在事务提交前必须全部写入磁盘  
‣ **Force**：要求（强制保证持久性，但影响性能）  
‣ **No-Force**：不要求（页可以延迟写入，但需要日志来保证持久性）

‣ Steal: changes can be moved into the disk as soon as the Tx starts
- steal policy means that the database system **allows** a transaction to overwrite the most recent committed value of a database element on disk even though the transaction has not committed. 
‣ No-Steal: changes cannot be moved into the disk until Tx commits

‣ Force: changes should be moved into the disk no later than when Tx commits
‣ No-Force: changes can be moved into the disk, even after Tx commits

![](image/Pasted%20image%2020260124104805.png)


![](image/Pasted%20image%2020260124104829.png)


## 1.4 Recovery Methods Overview

![](image/Pasted%20image%2020260124105056.png)


# 2 Shadow Pages （Force+No-Steal. slowest, trivial）

Steel => Write Dirty Pages. changes cannot be moved into the disk until Tx commits
Force => All Tx updates pushed before commit. changes should be moved into the disk no later than when Tx commits

‣ Force + No-Steal: Easiest solution
‣ No-Steal: As only committed Tx are written, the DBMS does not need to undo aborted Txs
‣ For Steal: An Undo would be required during recovery if changes of uncommitted Tx are written to disk

‣ Force: As all data of committed Tx are written, the DBMS does not need to redo Txs
‣ For No-Force: A Redo would be required during recovery if changes from a transaction could be written to disk after the Tx commits

![](image/Pasted%20image%2020260124105334.png)


## 2.1 Shadow Paging Implementation

‣ A technique to avoid in-place updates  
‣ It minimizes the risk of data corruption and allows for a consistent view of data

‣ The DBMS maintains two separate copies of the database:
    ‣ Master: Contains only changes from committed Txs
    ‣ Shadow: Temporary database with changes made from uncommitted transactions

‣ Updates are only made to the shadow page

‣ If a transaction commits, the shadow is switched to become the new master


‣ **一种避免原地更新的技术**  
‣ **它能最小化数据损坏风险，并提供一致的数据视图**

‣ **DBMS 维护两份独立的数据库副本**：  
‣ **主副本**：仅包含已提交事务的更改  
‣ **影子副本**：包含未提交事务更改的临时数据库

‣ **更新仅作用于影子页面**

‣ **当事务提交时，影子副本切换成为新的主副本**

---

**简要说明**：  
影子分页是一种早期的事务原子性保证技术，通过维护两份副本来避免原地更新带来的恢复问题。提交时通过切换指针快速完成，但缺点是存储开销大、碎片化严重，现代数据库多采用 WAL（预写日志）替代此方案。


![](image/Pasted%20image%2020260124105841.png)

## 2.2 Shadow Pages Properties

‣ Pros:  
‣ No need for log records  
‣ No Undo/ Redo algorithm  
‣ Recovery is fast (no undo/redo required)

‣ Cons:
‣ Normal operation is slow (a lot of waiting on committed Tx and dirty pages)
‣ Data is fragmented or scattered
‣ Garbage collection problem. Database pages containing old versions of modified data need to be garbage-collected after every transaction
‣ Concurrent transactions are difficult to execute

‣ **优点**：  
‣ 不需要日志记录  
‣ 不需要 撤销/重做 算法  
‣ 恢复速度快（无需执行撤销/重做操作）

‣ **缺点**：  
‣ 正常操作速度慢（大量等待已提交事务和脏页）  
‣ 数据碎片化或分散存储  
‣ 垃圾回收问题。每次事务后都需要回收包含旧版本数据的数据库页面  
‣ 难以执行并发事务

## 2.3 崩溃后恢复时，DBMS 需要做什么？

**影子分页的恢复非常简单：**
- 不需要做任何事（或者只需要丢弃影子页表）
- 因为事务提交前，所有修改都在影子页表中，主页表未被修改
- 如果崩溃发生在提交前：直接丢弃影子页表，回到主版本
- 如果崩溃发生在提交后：主版本已经是新版本，影子页表可丢弃

**恢复步骤：**
- 检查当前使用的是主页表还是影子页表
- 如果是影子页表且事务未提交 → 丢弃影子页表
- 如果是影子页表且事务已提交 → 切换为主页表

## 2.4 影子分页实现了哪种缓冲池策略？

**影子分页实现的是 NO-STEAL + FORCE 策略**
- **NO-STEAL**：事务提交前，不允许将未提交的修改写回磁盘（修改写在影子页面中）
- **FORCE**：事务提交时，强制将所有修改的页面写回磁盘

这样做的结果是：
- 不需要 UNDO（因为未提交的修改不在磁盘上）
- 不需要 REDO（因为提交时所有修改已写回）

## 2.5 为什么影子分页会导致磁盘碎片？

**原因：**
- 每次更新页面时，不是就地更新，而是**写到一个新位置**（写时复制）
- 随着时间的推移，同一个逻辑页面在磁盘上有多个物理版本
- 旧的页面变成"空洞"，无法被有效重用
- 导致磁盘空间利用率下降，文件系统碎片增加
- 需要定期压缩或整理


## 2.6 为什么基于 WAL 的恢复优于影子分页？

| 方面 | 影子分页 | WAL（预写日志） |
|------|---------|----------------|
| **写开销** | 每次修改都写整个页面 | 只写日志记录（小得多） |
| **磁盘碎片** | 严重，需定期整理 | 无额外碎片 |
| **并发** | 页表交换需全局锁 | 细粒度锁，并发高 |
| **恢复速度** | 恢复快（几乎不需要） | 需要扫描日志，但可优化 |
| **空间利用率** | 低（多个版本） | 高（就地更新） |
| **实现复杂度** | 中等 | 较高 |

**WAL 的主要优势：**
- **写日志是顺序 I/O**，比随机写页面快得多
- **支持更细粒度的并发控制**
- **不会产生磁盘碎片**
- **恢复灵活**：可以用检查点控制恢复时间


**结论：**
虽然影子分页恢复简单，但**写时复制的开销、磁盘碎片和并发限制**使其在大规模系统中不如 WAL 流行。现代数据库（如 PostgreSQL, MySQL InnoDB, Oracle）都使用 WAL。



# 3 Write-Ahead-Logging and Algorithms for Recovery and Isolation Exploiting Semantics (WAL, ARIES)  （No-Force+Steal,  Fastest）

WAL use NoForce and steal to recover the data

‣ Force: has poor response time (waiting for disk)
‣ No-Steel: has poor throughput (waiting on unfinished TX)

‣ Alternative: ARIES (Algorithms for Recovery and Isolation Exploiting Semantics)

‣ No-Force + Steal  
‣ Provides more flexibility to the DBMS  
‣ Is implemented using write-ahead logging


----
Tx can be seen as committed once the WAL entry is written

- DBMS records all the changes made to the database in a log file (on stable storage) before the change is made to a disk page  
- Log contains sufficient information to perform the necessary undo and redo actions to restore the database after a crash
- DBMS must write the log entry to disk before the page is written to disk  
- Compared to non-continuous writes in Shadow Paging, the log can be written in a sequential fashion
- Optimization: Group commits to flush log in batches to amortize overhead

- **DBMS 在将更改应用到磁盘页面之前，会在稳定存储的日志文件中记录所有对数据库的更改**
- **日志包含足够的信息**，以便在崩溃后执行必要的撤销和重做操作来恢复数据库
- **DBMS 必须在页面写入磁盘之前将日志条目写入磁盘**
- **与影子分页的非连续写入相比**，日志可以按顺序方式写入
- **优化**：将多个提交分组，批量刷新日志，以分摊开销


![](image/Pasted%20image%2020260124112138.png)

---
- Force log record for an update before the data page is written to disk
    - Ensures atomicity, solves STEAL: Even if a dirty state is on stable storage, the log contains a consistent state
- Write all log records of a TX before TX commits
    - Ensures durability, solves NO-FORCE:   Even if Tx changes are not on stable storage at crash time, the log
- Why is it Ok to force log records but not data pages
    -  Log records are written sequentially to a separate disk
    - Log records are smaller than data pages (contain only diffs) —> many log records per page

Pro: Faster runtime than shadow paging  
Con: Slower recovery time (cause of log replay) than shadow paging

- **在数据页写入磁盘之前，强制写入更新的日志记录**
    - **确保原子性，解决 STEAL 策略问题**：即使脏数据已写入稳定存储，日志也包含一致状态
- **在事务提交之前，写入该事务的所有日志记录**
    - **确保持久性，解决 NO-FORCE 策略问题**：即使事务更改在崩溃时未写入稳定存储，日志仍保留记录
- **为什么强制写入日志记录可行，而强制写入数据页不可行？**
    - 日志记录按顺序写入单独的磁盘
    - 日志记录比数据页小（仅包含差异）→ 每页可容纳多条日志记录

**优点**：运行时比影子分页更快  
**缺点**：恢复时间比影子分页慢（因为需要重放日志）

## 3.1 Logging Schemes

‣ Physical Logging:  
‣ Record the byte-level changes to a specific page
- least flexible

‣ Logical Logging:  
‣ Records the high-level operation executed by a query, e.g., UPDATE, INSERT, or DELETE
- less data written but recovery takes longer as queries have to be re-executed 

‣ Physiological Logging:  
‣ Physical-Logging-to-a-page and Logical-Logging-within-a-page
- more flexible 
- 不在再找物理位置了  ， 用 physical logging 中德位置 知道 tuples in page/disk 

![](image/Pasted%20image%2020260124112817.png)


## 3.2 Log Records

- Every log record contains a Log Sequence Number (LSN), a unique number identifying every log entry, which grows monotonically, allowing chronological order of log entires  
- The data page contains the PID (pageId) and the page LSN (pageLSN)
    - • pageLSN => the LSN of the last log record that updates the page
- Global system variable: flushedLSN
    - • flushedLSN => the maximum LSN flushed so far (end of stable log)
- WAL means that before a page is written, it applies that pageLSN <= flushedLSN  
- Log record consists of:
    - type, TID, prevLSN (LSN of previous log record of same TX to follow the chain)  
    - Additional for “update log records”: PID, length, offset, before, after Other types: ABORT, COMMIT, BOT, EOT, INSERT, DELETE, ...


- **每条日志记录包含一个日志序列号（LSN）**，这是一个唯一标识每条日志条目的单调递增数字，用于按时间顺序排列日志条目
- **数据页包含页面标识符（PID）和页面 LSN（pageLSN）**
    - **pageLSN** → 更新该页面的最后一条日志记录的 LSN
- **全局系统变量：flushedLSN**
    - **flushedLSN** → 当前已刷新到磁盘的最大 LSN（稳定日志的末尾）
- **预写日志（WAL）原则**：在页面写入磁盘前，必须确保该页面的 `pageLSN <= flushedLSN`
- **日志记录的组成**：
    - 类型、事务标识符（TID）、前一个 LSN（同一事务上一条日志记录的 LSN，用于链式追踪）
    - **针对“更新日志记录”的额外字段**：PID、长度、偏移量、修改前值、修改后值
    - 其他日志类型：中止（ABORT）、提交（COMMIT）、事务开始（BOT）、事务结束（EOT）、插入（INSERT）、删除（DELETE）等



redo 项记载的是: 如果要redo 的话 要做什么 
![](image/Pasted%20image%2020260124113103.png)

# 4 Recovery Process

## 4.1 Normal Operation
- A transaction is a series of reads and writes followed by a commit or an abort
- Buffer manager controls flushing to disk
- At page Update: 
    - Create a log record with a new LSN and set it as the pageLSN of the page
- At TX commit:
    - Write commit TX log record  
    - Force all log records up to TX’s largest LSN (lastLSN), i.e., writting all in- memory log entries to disk  
    - Write end TX (eTX) log record

**正常操作流程**
- **事务定义**：事务是一系列的读写操作，最后以提交或中止结束
- **缓冲区管理器控制**：负责将数据刷新到磁盘
- **页面更新时**：
    - 创建一条新的日志记录（带有新的 LSN - 日志序列号）
    - 将该 LSN 设置为页面的 `pageLSN`（记录页面上最新的更新）
- **事务提交时**：
    - 写入事务提交日志记录
    - **强制刷写**：将该事务所有日志记录（直到其最大的 LSN - `lastLSN`）写入磁盘
    - 写入事务结束日志记录（eTX）

**关键点说明**：
- **LSN（日志序列号）**：唯一标识每条日志记录，用于跟踪操作顺序
- **pageLSN**：每个数据页存储的最新更新对应的 LSN
- **强制刷写（Force）**：在事务提交时，确保所有相关日志记录已持久化到磁盘，这是保证**持久性（Durability）**的关键步骤
- **WAL（预写日志）原则**：日志记录必须在对应的数据页更改持久化之前写入磁盘

## 4.2 Situation after Failure

• TAs like T11 are winner transactions: they must be replayed completely  
‣TAs like T2 are loser transactions: they must be undone

![](image/Pasted%20image%2020260124113349.png)


## 4.3 Three Pass Recovery Process

1 Analysis pass
Figure out which TX have been committed, aborted, or failed
Determine winners and losers

2 Redo pass
Read the log forward and redo all actions: “Repeating history”  
Restores the database to a state where all TXs changes have been made stable  
All operations contained in the log are applied to the database instance in the original order

3 Undo pass
Read the log backwards for uncommitted TXs only  
Undo actions by failed TXs  
The operations of loser transactions are undone in the database instance in reverse order

**1 分析阶段（Analysis Pass）**  
找出哪些事务已提交、已中止或失败  
确定胜者事务（已提交）和败者事务（未提交或失败）

**2 重做阶段（Redo Pass）**  
向前读取日志并重做所有操作：“重演历史”  
将数据库恢复到所有事务更改已持久化的状态  
日志中包含的所有操作按原始顺序应用到数据库实例
- **重做阶段（Redo Pass）**：**向前（forward）** 扫描日志，从最后一个检查点开始，重做所有已提交和未提交的事务操作，将数据库恢复到崩溃前的状态。    

**3 撤销阶段（Undo Pass）**  
仅针对未提交事务向后读取日志  
撤销失败事务的操作  
败者事务的操作按**逆序**在数据库实例中撤销
- **撤销阶段（Undo Pass）**：**向后（backward）** 扫描日志，仅针对未提交事务（loser transactions）撤销它们的操作。


![](image/Pasted%20image%2020260124113528.png)

### 4.3.1 Analysis Phase
‣ The log contains BOT, commit, and abort entries 
‣ The log is scanned sequentially to identify all TAs 
‣ When a commit is seen, the TA is a winner  
‣ When an abort is seen, the TA is a loser
‣ TAs that neither commit nor abort are implicitly losers
‣ Overall: Winner have to be preserved, the losers have to be undone

**分析阶段（Analysis Phase）**

‣ **日志包含**：事务开始（BOT）、提交（commit）和中止（abort）记录  
‣ **扫描过程**：按顺序扫描日志以识别所有事务  
‣ **识别胜者**：当看到提交记录时，该事务为胜者  
‣ **识别败者**：当看到中止记录时，该事务为败者  
‣ **隐式败者**：既无提交也无中止记录的事务默认为败者  
‣ **总体原则**：胜者的更改必须保留，败者的更改必须撤销

### 4.3.2 Redo Phase

‣ Redo brings the DB into a consistent state:  
    ‣ some changes might still be in main memory at the crash 
    ‣ changes can be incomplete (e.g., B-tree split)  
    ‣ but the log contains everything
‣ Redo is done by one forward pass
    ‣ all log entries contain the affected page (and how they changed)
    ‣ the pages contain LSN entries (LSN that changed this page last)
    ‣ if the LSN of the page is less than the LSN of the entry, the operation must be applied
‣ Afterwards the DB has a known state


‣ **重做使数据库达到一致状态**：  
‣ 崩溃时某些更改可能仍在内存中  
‣ 更改可能不完整（例如 B 树分裂操作）  
‣ 但日志包含所有必要信息

‣ **重做通过一次前向扫描完成**  
‣ 所有日志条目都包含受影响的页面（以及更改方式）  
‣ 页面包含 LSN 标记（最近更改该页面的 LSN）  
‣ 如果页面的 LSN **小于**日志条目的 LSN，则必须应用该操作


The redo pass in recovery goes forward in the log from the maximum flushed log sequence number so far. -> falsch 

在数据库恢复（如 ARIES 算法）中：
- **重做阶段（Redo Pass）** 的起始点通常是**最后一个检查点（last checkpoint）** 或检查点记录中记录的 **RedoLSN**（即检查点时刻最旧的脏页对应的日志记录 LSN），而不是“当前最大的已刷新 LSN（flushedLSN）”。    
- **flushedLSN** 是系统正常运行时跟踪的、已刷新到磁盘的最大 LSN，用于保证 WAL。但在崩溃恢复时，重做阶段必须从**可能需重做的最早操作**开始，这由检查点信息决定，不一定等于 flushedLSN。
    
因此，说“重做阶段从当前最大 flushedLSN 开始向前扫描”是不准确的。


### 4.3.3 Undo Phase
‣ Eliminates all changes by loser transactions  
    ‣ During analysis, DBMS remembers last LSN of each transaction
    ‣ Transactions that aborted on their own can be ignored (no “last operation”, all undone)
    ‣ Only active TAs have to be rolled back 
‣ Log is read backwards
    ‣ All encountered operations are undone  
    ‣ Might produce new log entries (redo the undo) (see next slide)

‣ **撤销消除所有败者事务的更改**  
‣ 分析阶段，DBMS 记录每个事务的最后一个 LSN  
‣ 已自行中止的事务可忽略（无“最后操作”，已全部撤销）  
‣ 仅需回滚活动事务

‣ **向后读取日志**  
‣ 遇到的所有操作均被撤销  
‣ 可能产生新的日志条目（记录撤销操作）（见下页）

---

Redo the Undo

‣ Initial Undo: A transaction fails, and the database uses its undo log to reverse its changes (e.g., deleting a record)

‣ Compensation Log Record (CLR): During this undo process, the system creates new log entries (CLRs) that say, "I just undid that deletion, so now I'm inserting it back"

‣ "Redoing" the Undo: If the system crashes while undoing, the CLRs allow the recovery manager to "redo" these compensation actions, ensuring the data is clean and consistent, effectively redoing the undo

**重做撤销**
‣ **初始撤销**：事务失败时，数据库使用其撤销日志来反转其更改（例如删除一条记录）
‣ **补偿日志记录（CLR）**：在此撤销过程中，系统创建新的日志条目（CLR），内容是“我刚刚撤销了那个删除操作，所以现在我把记录插回去”
‣ **“重做”撤销操作**：如果系统在撤销过程中崩溃，CLR 允许恢复管理器“重做”这些补偿操作，确保数据干净一致，从而有效地重做撤销过程

# 5 Summary

![](image/Pasted%20image%2020260124114027.png)


# 6 Optimization Checkpointing

BIT : Beginning of transcation
EOT: End of Transaction

Before Check point: change is in log and main memeory, but not insered into Disk 
After Check Points: change 不再存在于log, logs are refreshed and new started. changes is insered into disk, not in main momery before 

![](image/Pasted%20image%2020260124114051.png)



- Starting at the beginning of the log every time is not practical 
- Need to consolidate changes periodically into checkpoints  
- An ideal checkpoint:
    - Do not slow down regular TX processing
    - Do not introduce unacceptable latency spikes
    - Do not require excessive memory overhead
- Tunable option that depends on the application recovery time requirements  
    - Trade-off between recovery time and runtime impact Dish

Common criteria: 
- Time-based: wait for a fixed period of time after the last checkpoint has complete
- Log File Size Thresholf: Begin checkpoint after a certain amount of data has been written to the log file 
- On Shutdown(Mandatory): Perform a checkpoint when the system shuts down


- 每次都从日志开头开始恢复是不现实的
- 需要定期将变更合并到检查点中
- 理想的检查点：
    - 不拖慢常规事务处理
    - 不引入不可接受的延迟峰值
    - 不需要过高的内存开销
- 可根据应用程序的恢复时间要求进行调整的选项
    - 在恢复时间和运行时影响之间权衡

常见标准：
- 基于时间：在上一个检查点完成后等待固定时间段
- 日志文件大小阈值：当日志文件写入一定量数据后开始检查点
- 关闭时（强制）：系统关闭时执行检查点


![](image/Pasted%20image%2020260124114104.png)


use the checkpoint record as the startying point for analyzing the WAL 

==Any txn that committed before the checkpoint is ignored (TA)==

T1 + T3 did not commit before the last checkpoint
- Need to redo T2 because it committed after checkpoint
- Need to undo T3 because it did not commit before the crash 

![](image/Pasted%20image%2020260124114331.png)

