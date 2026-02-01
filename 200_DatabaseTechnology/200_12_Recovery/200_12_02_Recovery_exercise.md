
# 1 Introduction

## 1.1 What types of storage devices do DBMSs use and for what purpose?

Disk (HDD, SDD):  Primary Storage, large capacity, cheap, durable in case of a power outage
Memory (Dram ):  For Updates, smaller capacity, fast, more expensive, volatile 

## 1.2 What are transactions?


Sequences of logically related operations in a database that run as a single unit 

## 1.3 What are the fundamental properties of transactions?


1. Run atomically, either Commit or Abort
2. DB is consistent before and after the txn (to the user)
3. Do not affect each other (to the user)
4. Effects are durable

## 1.4 How do we manage data between the two storage devices?What can happen when a crash occurs? 

- The Buffer manager moves data pages between disk and memory as required 
- Over time, the content of the pages deviated from their copy on disk (dirty pages)
- In the event of a crash, updated are lost

Butter page data stored in Pages Between Disk and main momery



## 1.5 Given his overview of a transaction managed by a DBMS using a BufferPool. What problems can be caused here?

![](image/Pasted%20image%2020260124115244.png)





## 1.6 DBMS crashes at point .What should happen to the in-flight transactions?

![](image/Pasted%20image%2020260124115322.png)


# 2 Buffer Mangement (Steal and Force )


STEAL: allow uncommitted txn to overwrite committed value and flush.

NO-STEAL: do not allow a page modified by an uncommitted txn to be flushed.

FORCE: require all pages modified by a txn to be flushed to disk before commit.

NO-FORCE: do not require to flush every modified page to disk before commit.

## 2.1 What is the problem here?

![](image/Pasted%20image%2020260124114717.png)



- We run tow txns modifying the same page concurrently
- txn 2 wants to commit, we want those changes durable
- txn 1 will later abort, we do not want those changes durable 

## 2.2 Are we allowed to flush the page under the no-steal policy?

No, No-steal prevents flushing pages with uncommitted changes 

## 2.3 Do we need to flush the page under the force policy?

Yes, the force policy requires that, before committing, the dirty pages affected by the txn be flushed disk 

# 3 ARIES  basic principle:



---


What is the basic principle of a Write-Ahead Log (WAL)?

- Use WAL during txn execution
- Flush WAL to disk before dirty pages
- Use WAL on restart to restore state before crash via undo & redo

---

Which buffer pool policy does WAL-based recovery implement?
Append all changes made by transactions to the DB to a log file.
The DBMS must flush all relevant log records corresponding to changes that made a page dirty to disk before it can flush the page itself 


# 4 Aries Example 


![](image/Pasted%20image%2020260124114748.png)

## 4.1 Why do we want to store the prevLSN in the log entries?

When traversing the log in reverse direction during UNDO, we may want to skip irrelevant records and follow only a specific txn 

![](image/Pasted%20image%2020260124120120.png)


## 4.2 If we crash now already, is this a problem? Why or why not?

Not a problem, because 
- No txn has committed yet. Therefore, no redo is necessary
- No dirty pages of uncommitted transactions have been flushed to disk, so no undo is necessary either

![](image/Pasted%20image%2020260124120201.png)


## 4.3 The buffer manager wants to evict page with Disk PID=1. What steps are required to ensure consistency?

![](image/Pasted%20image%2020260124120248.png)


![](image/Pasted%20image%2020260124120303.png)


## 4.4 flushedLSN tells us the last LSN that was successfully flushed to disk.

- Flush the log until LSN 4. only then, in case of a failure, can we undo the change. 
- Flush the page itself to disk.
- Remove the entry from DPT; the page is not dirty anymore

![](image/Pasted%20image%2020260124120331.png)


## 4.5 TXN1 wants to commit, What is requered for this 

![](image/Pasted%20image%2020260124120425.png)


# 5 Aries Example: If a crash happens now, what happens to our data structures?

10 and 11  and two Pages table with PID=2 or 3 will be lost 

Which txns need to be aborted
1. WAL tail in memory, pages2 and 3 in memory, ATT and DPT are lost. Everything else is durable
2. T2 did not commit, and therefore needs to abort. T1's commit log entry is persisted on disk

![](image/Pasted%20image%2020260124121105.png)


## 5.1 T1 committed, but what about its updates?

![](image/Pasted%20image%2020260124121129.png)


## 5.2 How do we find out what we need to redo?

- Analyse the log in forward direction and populate ATT and DPT
- Oldest recLSN in DPT tells us where to start REDO

![](image/Pasted%20image%2020260124121144.png)

![](image/Pasted%20image%2020260124121151.png)


## 5.3 Replay the log to restore state for all committed (winner) txns

## 5.4 Do we really need to redo everything?

![](image/Pasted%20image%2020260124121323.png)

# 6 Aries Example: Undo 

1. What txns do we need to UNDO?
2. Where do we start to UNDO?: redo the coperationen which do undo 
3. what actions ado we need to UNDO here


![](image/Pasted%20image%2020260124121431.png)


# 7 Aries Example:  Checkpoint 
What if the database has been running for a year without failure?How could we improve on recovery performance?

use checkpoint. This point, the excution are complete, gerabge the log into checkpoint and make a snapshot 


# 8 Overview ARIES Algorithm
1. AnalysisPhase
2. RedoPhase  
3. UndoPhase

Why do we need to start REDO at the smallest recLSN ? Do we need to continue logging during recovery?

![](image/Pasted%20image%2020260124120808.png)

## 8.1 Shadow Paging

Maintain two versions of the database incl. page table, master and shadow Copy pages on write to shadow page table  
On txn commit, flush pages and swap shadow with master

1. What does the DBMS need to do on recovery after a crash?
2. Which buffer pool policies does shadow paging implement?
3. Why does shadow paging lead to fragmentation on disk?
4. 4. Why does WAL-based recovery outperform shadowpaging?

# 9 Quiz

## 9.1 Aries 

![](image/Pasted%20image%2020260124202633.png)


![](image/Pasted%20image%2020260124202641.png)


我们先明确已知信息：

- **初始检查点**位于 LSN 7 (BEGIN_CHECKPOINT)，此时事务表(TT)和脏页表(DPT)为：
  - TT: T1(lastLSN=4), T2(lastLSN=6), T3(lastLSN=5)
  - DPT: PID=3(recLSN=4)
- 检查点结束在 LSN 12 (END_CHECKPOINT)。
- 恢复时从 LSN 7 开始分析阶段（因为检查点开始时的事务/脏页信息已知，但仍需从该点扫描到日志尾来更新状态）。

---

**1. 分析阶段（从 LSN 7 扫描到日志尾）**

初始（LSN 7 时）：
- TT = {T1:4, T2:6, T3:5}
- DPT = {3:4}

扫描日志：

**LSN 8**: T1 UPDATE (2,4,"d","D")
- TT: T1.lastLSN = 8（更新）
- DPT: 新增页 2？recLSN 应为此页第一次变脏时的 LSN，此前 DPT 无页 2，因此添加 2:8

**LSN 9**: T3 UPDATE (1,5,"e","E")
- TT: T3.lastLSN = 9
- DPT: 新增页 1，recLSN=9

**LSN 10**: T1 COMMIT
- TT: 移除 T1（因为提交后事务不再需要恢复）

**LSN 11**: T1 END_TRANSACTION（通常只是记录，不改变分析阶段状态，可能忽略）

**LSN 12**: END_CHECKPOINT（无影响）

**LSN 13**: T2 UPDATE (3,6,"f","F")
- TT: T2.lastLSN = 13
- DPT: 页 3 已经在 DPT 中（recLSN=4），不需更改

**LSN 14**: T2 UPDATE (2,7,"g","G")
- TT: T2.lastLSN = 14
- DPT: 页 2 已在 DPT 中（recLSN=8），不需更改

扫描到日志尾后（LSN 14），T2 和 T3 未提交，T1 已提交并从 TT 移除。

---

**2. 分析阶段结束时的表**

**事务表 (TT)：**
- T2: lastLSN = 14
- T3: lastLSN = 9
- T1 已提交 → 不在表中

**脏页表 (DPT)：**
- 页 3: recLSN = 4（初始就有）
- 页 2: recLSN = 8（LSN 8 第一次变脏）
- 页 1: recLSN = 9（LSN 9 第一次变脏）

---

**3. 填写答案**

Transaction table:
| TID | lastLSN |
|---|---|
| 1   | No Entry |
| 2   | 14       |
| 3   | 9        |

Dirty page table:
| PID | recLSN |
|---|---|
| 1   | 9       |
| 2   | 8       |
| 3   | 4       |

---

如果你需要，我可以把每个 LSN 处理后的 TT 和 DPT 变化详细列出。

## 9.2 Aries

**Select the action taken during the undo passes for the UPDATE actions in the drop-down menus in the log above.**

![](image/Pasted%20image%2020260124203022.png)

![](image/Pasted%20image%2020260124203031.png)



我们需要进行恢复的三个阶段（分析、重做、撤销）。  
题中给的表格最后一列 “Undo Pass” 已打钩的是需要在撤销阶段处理的日志记录。  
我们需要确定撤销阶段处理这些记录的顺序。

---
**已知信息**
- 检查点在 LSN 7 开始，LSN 12 结束。
- 恢复从 LSN 7（BEGIN_CHECKPOINT）开始分析阶段。
- 初始事务表 (TT) 和脏页表 (DPT) 已经在检查点开始时给出：
  - TT: T1:6, T2:5, T3:3
  - DPT: 页1:4, 页2:5
- 分析阶段会更新这些表。
- 撤销阶段根据分析阶段结束后的 TT 来决定哪些事务要撤销，然后按 LSN 从大到小撤销这些事务的所有更新记录。

---

**1. 分析阶段（从 LSN 7 扫描到日志尾 LSN 14）**

初始状态（LSN 7）：
TT = {T1:6, T2:5, T3:3}  
DPT = {1:4, 2:5}

扫描：

**LSN 8**：T1 UPDATE (1,4,"d","D")
- TT: T1.lastLSN = 8
- DPT: 页1已在表中，recLSN=4（不变）

**LSN 9**：T2 UPDATE (3,5,"e","E")
- TT: T2.lastLSN = 9
- DPT: 页3新增，recLSN=9

**LSN 10**：T2 COMMIT
- TT: 移除 T2（提交完成）

**LSN 11**：T2 END（不影响TT/DPT）

**LSN 12**：END_CHECKPOINT（无影响）

**LSN 13**：T3 UPDATE (2,6,"f","F")
- TT: T3.lastLSN = 13
- DPT: 页2已在表中，recLSN=5（不变）

**LSN 14**：T3 UPDATE (3,7,"g","G")
- TT: T3.lastLSN = 14
- DPT: 页3已在表中，recLSN=9（不变）

**分析阶段结束时**：
- TT = {T1:8, T3:14}  （T2 已提交移除）
- DPT = {1:4, 2:5, 3:9}

---
**2. 重做阶段**
从 DPT 中最小的 recLSN = 4 开始向前扫描到日志尾，重做所有更新记录（无论事务是否提交），但已打钩的 Undo Pass 列表示撤销阶段也要处理，我们先不关心。

---

**3. 撤销阶段**
败者事务（losers） = TT 中未提交的事务 = {T1, T3}  
按 lastLSN 从大到小依次撤销（从 T3 开始，因为 T3.lastLSN=14 > T1.lastLSN=8）。

**T3 的更新链**（通过 prevLSN 反向）：
- lastLSN=14 (UPDATE g) → prevLSN=13 (UPDATE f) → prevLSN=3 (BEGIN)，没有其他更新。
  所以 T3 要撤销的 LSN 顺序：14（先），13（后）。
  撤销时写入 CLR。

**T1 的更新链**：
- lastLSN=8 (UPDATE d) → prevLSN=6 (UPDATE c) → prevLSN=4 (UPDATE a) → prevLSN=1 (BEGIN)。
  撤销顺序：8, 6, 4。

---

表中“Undo Pass”列打钩的记录是 LSN=6,8,9,13,14。  
但 LSN 9 是 T2 的更新，T2 已提交，为什么打钩？可能题目的表是“恢复过程中可能需要撤销的记录”预判，但实际撤销阶段只撤销败者事务（T1,T3）的记录，所以 LSN 9（T2）不应该被撤销。  
可能是表格排版错位，或者他们假设所有未在检查点提交的事务的更新都要撤销（但 T2 在检查点开始时未提交，在 LSN 10 提交了，所以提交后不用撤销）。  
从分析阶段结果看，撤销的更新记录应该是：
T3: LSN 14, 13  
T1: LSN 8, 6, 4

表中 LSN 4,5 没打钩，但 4 是 T1 的更新要撤销，5 是 T2 的更新（已提交）不撤销。  
所以可能题目给的“Undo Pass”列并不完全准确，或者他们只考虑未提交事务的最后部分记录。  
但是根据检查点开始的 TT 和最终分析结果，确实 T1 和 T3 是败者。

---

**所以实际撤销顺序**（按事务，每个事务内按 LSN 降序）：
1. T3: LSN 14 → LSN 13  
2. T1: LSN 8 → LSN 6 → LSN 4

---

如果你需要，我可以画出撤销过程的具体 LSN 顺序，或者解释为什么表中 LSN 9 被标记了 Undo Pass（可能是标记错误，或者 T2 在检查点时未提交但在恢复时可能因为某些原因也算败者？但按标准分析阶段它在 LSN 10 提交了，所以不是败者）。


![](image/Pasted%20image%2020260124203911.png)

## 9.3 Aries

![](image/Pasted%20image%2020260124203959.png)

![](image/Pasted%20image%2020260124204007.png)



我们先明确分析阶段的任务：从 LSN 7（BEGIN_CHECKPOINT）扫描日志到末尾（LSN 14），更新事务表和脏页表。

已知 LSN 7 时初始状态：

- **TT**：T1=4, T2=6, T3=3
    
- **DPT**：页1=4, 页2=5
    

---

### 9.3.1 **逐条分析更新**


**LSN 8**：T2 UPDATE (1, 4, "d", "D")

- 事务表：T2.lastLSN 从 6 更新为 8
    
- 脏页表：页1 已在 DPT 中（recLSN=4），所以不更新 recLSN
    

此时：  
TT: T1=4, T2=8, T3=3  
DPT: 页1=4, 页2=5

---

 **LSN 9**：T2 UPDATE (3, 5, "e", "E")

- TT: T2.lastLSN 从 8 更新为 9
    
- DPT: 页3 不在表中，所以加入 **页3 = 9**（第一次变脏的 recLSN 就是当前 LSN）
    

此时：  
TT: T1=4, T2=9, T3=3  
DPT: 页1=4, 页2=5, 页3=9

---

**LSN 10**：T2 COMMIT

- TT: 移除 T2（因为提交完成，不需要再恢复它）
    

此时：  
TT: T1=4, T3=3  
DPT: 页1=4, 页2=5, 页3=9

---

**LSN 11**：T2 END_TRANSACTION

- 对 TT/DPT 无影响
    

---

**LSN 12**：END_CHECKPOINT

- 无影响
    

---

**LSN 13**：T3 UPDATE (2, 6, "f", "F")

- TT: T3.lastLSN 从 3 更新为 13
    
- DPT: 页2 已在表中（recLSN=5），不更新
    

此时：  
TT: T1=4, T3=13  
DPT: 页1=4, 页2=5, 页3=9

---

**LSN 14**：T3 UPDATE (3, 7, "g", "G")

- TT: T3.lastLSN 从 13 更新为 14
    
- DPT: 页3 已在表中（recLSN=9），不更新
    

最终：  
TT: T1=4, T3=14  
DPT: 页1=4, 页2=5, 页3=9

---

**分析阶段结束时的表**：

**Transaction table**：

|TID|lastLSN|
|---|---|
|1|4|
|3|14|

**Dirty page table**：

|PID|recLSN|
|---|---|
|1|4|
|2|5|
|3|9|

---

然后重做阶段从 DPT 的最小 recLSN = 4 开始到日志尾 LSN 14，对每个 UPDATE 检查 pageLSN < LSN（题设成立），所以所有 UPDATE 都要重做。
所有 UPDATE（LSN 4,5,6,8,9,13,14）在重做阶段都要执行重做。


- 重做阶段从 DPT 中最小的 recLSN 开始向前扫描到日志尾，对每个 UPDATE 记录，检查是否需要重做（即 pageLSN < LSN 时重做）。
    
- 题目假设 **pageLSN 总是小于 LSN**，这意味着**每个 UPDATE 记录在重做阶段都需要重做**，因为 pageLSN < LSN 始终成立（pageLSN 表示该页最后被更新的 LSN，这里假设总是比当前日志 LSN 小）。

DPT 中最小 recLSN = 4（页1），所以重做从 LSN=4 开始扫描到 LSN=14。

需要判断的记录是 **UPDATE 动作**（LSN 4,5,6,8,9,13,14）。  
按顺序：

1. **LSN 4** (T1, UPDATE (1,1,"a","A"))：pageLSN < LSN → 重做
    
2. **LSN 5** (T2, UPDATE (2,2,"b","B"))：pageLSN < LSN → 重做
    
3. **LSN 6** (T2, UPDATE (1,3,"c","C"))：pageLSN < LSN → 重做
    
4. **LSN 8** (T2, UPDATE (1,4,"d","D"))：pageLSN < LSN → 重做
    
5. **LSN 9** (T2, UPDATE (3,5,"e","E"))：pageLSN < LSN → 重做
    
6. **LSN 13** (T3, UPDATE (2,6,"f","F"))：pageLSN < LSN → 重做
    
7. **LSN 14** (T3, UPDATE (3,7,"g","G"))：pageLSN < LSN → 重做
    

所以 **所有 UPDATE 记录在重做阶段都要重做**。