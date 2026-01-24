

# 1 Introduction

## 1.1 What types of storage devices do DBMSs use and for what purpose?
    
## 1.2 What are transactions?
    
## 1.3 What are the fundamental properties of transactions?
    
## 1.4 How do we manage data between the two storage devices?What can happen when a crash occurs? 

Butter page data stored in Pages Between Disk and main momey
    
## 1.5 Given his overview of a transaction managed by a DBMS using a BufferPool. What problems can be caused here?

![](image/Pasted%20image%2020260124115244.png)

## 1.6 DBMS crashes at point .What should happen to the in-flight transactions?

![](image/Pasted%20image%2020260124115322.png)


# 2 Buffer Mangement 

## 2.1 Steal and Force 

## 2.2 #

![](image/Pasted%20image%2020260124114717.png)

STEAL: allow uncommitted txn to overwrite committed value and flush.

NO-STEAL: do not allow a page modified by an uncommitted txn to be flushed.

FORCE: require all pages modified by a txn to be flushed to disk before commit.

NO-FORCE: do not require to flush every modified page to disk before commit.

## 2.3 Are we allowed to flush the page under the no-steal policy?
    
## 2.4 Do we need to flush the page under the force policy?

# 3 ARIES ARIES basic principle:

Use WAL during txn execution

Flush WAL to disk before dirty pages

Use WAL on restart to restore state before crash via undo & redo

# 4 Aries Example 

![](image/Pasted%20image%2020260124114748.png)

## 4.1 Why do we want to store the prevLSN in the log entries?

![](image/Pasted%20image%2020260124120120.png)


## 4.2 If we crash now already, is this a problem? Why or why not?

![](image/Pasted%20image%2020260124120201.png)


## 4.3 The buffer manager wants to evict page with Disk PID=1. What steps are required to ensure consistency?

![](image/Pasted%20image%2020260124120248.png)


![](image/Pasted%20image%2020260124120303.png)


## 4.4 flushedLSN tells us the last LSN that was successfully flushed to disk.

![](image/Pasted%20image%2020260124120331.png)


## 4.5 TXN1 wants to commit, What is requered for this 

![](image/Pasted%20image%2020260124120425.png)


# 5 Aries Example: If a crash happens now, what happens to our data structures?

10 and 11  and two Pages table with PID=2 or 3 will be lost 
![](image/Pasted%20image%2020260124121105.png)


## 5.1 T1 committed, but what about its updates?

![](image/Pasted%20image%2020260124121129.png)


## 5.2 How do we find out what we need to redo?

![](image/Pasted%20image%2020260124121144.png)

![](image/Pasted%20image%2020260124121151.png)


## 5.3 Replay the log to restore state for all committed (winner) txns

## 5.4 Do we really need to redo everything?

![](image/Pasted%20image%2020260124121323.png)

# 6 Undo 

1. What txns do we need to UNDO?
2. Where do we start to UNDO?: redo the coperationen which do undo 
3. what actions ado we need to UNDO here


![](image/Pasted%20image%2020260124121431.png)




# 7 Checkpoint 
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