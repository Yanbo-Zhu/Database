

Storage: Memory Hierarchy, Disks, Efficient Disk Operations, Access Acceleration, Disk Crashes, RAID

# 1 Memory Hierarchy

![[Pasted image 20251108230456.png]]


![[Pasted image 20251108230503.png]]


![[Pasted image 20251108230511.png]]

## 1.1 Cache 

Idea: move important data from lower to higher tier

Principle of locality
CPU Caches:
- It brings data from main memory to the CPU memories
- Hardware controlled
File System Cache:
- It brings data from permanent storage to main memory
- OS controlled
DBMS Buffer Cache:
- It brings data from permanent storage to main memory
- DBMS controlled

----

Locality  access
Two kinds of localities
- Temporal locality - data currently used is likely to be reused
- Spatial Locality - data close to each other are likely to be used together
- Data without locality is bad
Goal: increase the ratio cache-hit-ratio
- Cache hit - requested data is in the cache
- Cache miss - requested data is not in the cache 
- Cache-hit-ratio -   nb_hits/(nb_hits — nb_misses)
Replacement strategies
- LRU: Least Recently Used
- FIFO: First In, First Out
- And More 

![[Pasted image 20251108231306.png]]


----

CPU Caches

Small (from KBs to few MBs), Fast (in ns), and extremely expensive
Under CPU control:
- One can influence contents only via memory access patterns and prefetch instructions
- ====
- Cache Coherency Protocol assures consistency between cores (read more here: https://www.geeksforgeeks.org/cache-coherence/)
- Cache Inclusion Policy determines which content of which cache is also present in a different cache (read more here: https:// en.wikipedia.org/wiki/Cache_inclusion_policy)
- Current Intel consumer CPUs: L1 (32 kB), L2 (256 kB), LLC (≥ 8 MB)


![[Pasted image 20251108230631.png]]

![](image/Pasted%20image%2020260228194056.png)


The "Stride" (distance between two accesses) impacts the performance.
![](image/Pasted%20image%2020260228194104.png)


## 1.2 Main Memory


Main Memory
100 ns – 100x slower than CPU registers Random access and (typically) byte-addressable

Access time does not depend on location:
- some architectures can only access data at 4-byte or 8-byte boundaries
- It depends on access pattern
- CPUs typically transfer entire cache lines (Intel: 64 bytes)
- Strided access pattern (every 64th byte) is slower than sequential access pattern (every byte)
- How to layout two-dimensional data?

![](image/Pasted%20image%2020260228194247.png)

## 1.3 Secondary Storage

Typically magnetic disks (hard disks/ HDDs) or solid state drives (SSDs)

Access for a single byte:
1 – 20 million times slower than CPU registers


Data transfer rate:
~125 MB/s (7200 RPM HDD)
GBs/s (SSD)
Hide single byte access latency through data locality
Arrange data so that locality is maximized
Under the control of the application (OS, DBMS)

## 1.4 Tertiary Storage

Storage on magnetic tape (cassettes) — human vs robot store (10x faster)
Access time:
• seconds/minutes – Too expensive (maintenance, electricity, machines, space)

Comparison tertiary storage — secondary storage
I/O-times much higher 
Storage capacity much higher
- Many Terabytes (10^12 Bytes) of sales data
- Many Petabytes (10^15 Bytes) of scientific data
- Cost per Byte much lower

No random access — access times depend largely on the position of a data set


## 1.5 Volatile vs Non-Volatile


Volatile 易变的，动荡不定的，反复无常的；（情绪）易变的，易怒的，突然发作的；（液体或固体）易挥发的，易气化的；（计算机内存）易失的

Volatile: "Forgets" contents when power is turned off
- CPU registers, caches, main memory (DRAM)

Non-volatile: Keeps contents permanently, even without power
- secondary storage, tertiary storage
- Non-volatile main memory (NVM, NVRAM)

Durability:
- Changes are not considered final until they are stored on permanent, non- volatile, secondary storage
- Significant part of database complexity
- Keyword: Logging (→ later lectures)


## 1.6 Virtual Memory

Application typically execute in a virtual address space
x86: 32-bit address space —> 232 addresses can be represented , Each byte has its own address —> max. 4 GB of main memory
64-bit address space —> 256 TB

Virtual memory exceeds physical main memory
Allows programs to use more memory than physically available Solution: Page (or swap) data to disk
Managed by operating system

Database systems typically have their own memory management
Some rely on the OS
Main-memory DBMSes are common today

Moore's Law: exponential growth of many parameters that double about every 18 month

![](image/Pasted%20image%2020260228195018.png)



# 2 Disks


## 2.1 Magnetic Hard Disks
Composed of three main components
Disk assembly – multiple homogeneously rotating circular platters 
Head assembly – One read/write head for each platter surface
Disk controller – Controls position and operation of read/write heads


### 2.1.1 Mechanics of Disk 

![](image/Pasted%20image%2020260228200148.png)


![](image/Pasted%20image%2020260228200211.png)


![](image/Pasted%20image%2020260228200241.png)


![](image/Pasted%20image%2020260228200252.png)

![](image/Pasted%20image%2020260228200306.png)


![](image/Pasted%20image%2020260228200317.png)



### 2.1.2 Head assembly

Composed of three main components
Disk assembly – multiple homogeneously rotating circular platters 
Head assembly – One read/write head for each platter surface
Disk controller – Controls position and operation of read/write heads

One head per surface
They all move together
It never touches the surface Read and write magnetic pattern

Heads typically cannot read in parallel

### 2.1.3 Disk controller
Controls position and operation of read/write heads

It manages one or more disks
It controls read/write heads
It selects surface and sector to access
It transfers between main memory and disk It caches data
It performs integrity checks


### 2.1.4 Example — Megatron 747 Disk

![](image/Pasted%20image%2020260228201141.png)






### 2.1.5 Disk Summary


![[Pasted image 20251108231218.png]]

![[Pasted image 20251108235201.png]]


## 2.2 Disk Access Characteristics

### 2.2.1 Disk Latency 
Accessing (reading or writing) a block requires four steps:
1. Transferring blocks between main memory and disk controller
	— Communication delay (Typically ignored, PCI speed or faster, +11 GB/s)
2. Position head assembly at cylinder containing the track containing the block
	— Seek time
3. Wait until first sector of block rotates under read/write head
	— Rotational latency
4. Transfer bits passing under the head to disc controller (or vice versa)
	— Transfer time

Disk latency = Seek time + Rotational latency + Transfer time


### 2.2.2 Disk Access Characteristics — Example

Characteristics
8 disks with 16 surfaces (diameter: 3,5") 
216 = 65,536 tracks per surface
On average 28 = 256 sectors per track 
212 = 4,096 Byte per sector
16 KB block consists of four 4 KB sectors

Access properties
7200 rpm → 1 rotation every 8.33 ms
1 ms to start and stop the head assembly
1 ms for every 4000 cylinders travelled  
→ Move one track in 1 / 4000 + 1 = 1.00025 ms 
→ Move from innermost to outermost track in 65536 / 4000 + 1 = 17.38 ms Gaps occupy 10% of tracks


![](image/Pasted%20image%2020260228201248.png)


### 2.2.3 Best Case  

Head is already positioned above correct cylinder 
Beginning of correct sector is under the head 
Seek time + rotational latency is 0
Latency caused by transfer time alone


![[Pasted image 20251108231829.png]]


### 2.2.4 worst case 

Head is outermost cylinder, block is innermost cylinder


![[Pasted image 20251108231838.png]]


### 2.2.5 Average Case

Average seek distance is 1/3 of tracks 

![[Pasted image 20251108231847.png]]



## 2.3 Write and Modifying Blocks

Writing of blocks
Algorithm and latency: like reading
Can use checksum to verify successful write
Has to wait a rotation for verification


Changing of blocks
- Not directly possible 
    - Read block into main memory  
    - Modify the data 
    - Write block back to disk 
    - Check for success
- Latency = tr + tw



![[Pasted image 20251108231916.png]]


## 2.4 SSD

SSDs have maximum write cycles between 10K - 100K and are on average more expensive


![](image/Pasted%20image%2020260228202933.png)


## 2.5 Persistent Memory

![](image/Pasted%20image%2020260228203733.png)


## 2.6 New Storage Hierarchy

![](image/Pasted%20image%2020260228203750.png)


# 3 Efficient Disk Operations


1 I/O Model of Computation

Assumptions
DBMS answering queries or performing modifications 
One CPU, one disk
Database too large to fit into main memory
Parts of database are buffered in main memory

假设条件
数据库管理系统执行查询或修改操作
采用单CPU、单磁盘配置
数据库规模过大，无法完全存入主内存
数据库部分数据通过缓冲区暂存于主内存

Dominance of I/O cost
Time required for disk access (1 – 20 ms) >> time required to manipulate data in main memory (100 ns) 
- [4 – 5 orders of magnitude larger]
For each disk access the CPU can perform millions of instructions 
Disk I/Os should be minimized
I/O成本占主导地位
磁盘访问时间（1-20毫秒）远超内存数据处理时间（100纳秒）
——两者相差4-5个数量级——
每次磁盘访问期间，CPU可执行数百万条指令
因此必须最大限度减少磁盘I/O操作

---

2 RAM Model vs. I/O Model

Usual assumption when analyzing algorithms:
RAM model of computation
Access to data costs nothing
Only operations on data count: comparison, arithmetic, counting, ...
采用RAM计算模型
数据访问不计成本
仅对数据的操作计入成本：比较、算术运算、计数等


Assumption when implementing a database:
I/O model of computation
Operations costs nothing Only access to data counts
采用I/O计算模型
操作不计成本，仅数据访问计入成本

Main memory databases:
I/O model for main memory
Operations on data in CPU cache costs nothing 
Moving data from main memory to cache counts
采用面向主存的I/O模型
CPU缓存内的数据操作不计成本
将数据从主内存调入缓存的过程计入成本

---


3 I/O Model: Indexes Example

Looking for a tuple t in relation R with key k (index on key attribute)
- Alternative A only returns in which block t is located
- Alternative B also provides information where in block t is located 
- Alternative B requires more space,
    - Question: does it offer benefits during search?
    - Answer
        - On average 11 ms to transfer 16 KB Block
            - During this time: CPU can process many million instructions
        - Searching for k in a block costs thousands of CPU instructions
            - even with linear search

==Time to search in a block << Time to read block==

在关系R中查找键值为k的元组t（基于键属性建立索引）  

- 方案A仅返回元组t所在的磁盘块  
- 方案B额外提供元组t在磁盘块内的具体位置  
- 方案B需要更多存储空间  
  - 问题：在查找过程中能否带来性能提升？    能， B方案更好 
  - 答案  
    - 传输16KB磁盘块平均耗时11毫秒  
      - 在此期间：CPU可执行数百万条指令  
    - 在磁盘块内搜索键值k仅需数千条CPU指令  
      - 即便采用线性查找也是如此

![](image/Pasted%20image%2020260301000327.png)
# 4 Access Acceleration

1. Store blocks that are processed together on same cylinder
	→ Reduced seek times
2. Distribute data on multiple disks
	→ Parallel reads and writes
3. Replicate data on multiple disks
	→ Parallel reads
4. Use a disk scheduling algorithm
	→ Reduced seek times
5. Prefetch blocks
	→ Reduced latencies


## 4.1 Increase Locality


Recall heads typically cannot read in parallel

Assumptions
- Relation R requires 1024 blocks 
- Average read latency: 10,76 ms 
- Average seek time: 6,46 ms 
- One rotation: 8,33 ms


Blocks distributed randomly on disk
- 1024 blocks * 10,76 ms / block = 11 s


Blocks stored close together
- 1024 blocks is one cylinder on Megatron 747 
- Megatron 747 has 16 surfaces
- One seek + 1 rotation per surface surfaces 
- 6,46 ms + 16 * 8,33 ms = 139 ms → ~ 80x faster


## 4.2 Disk Scheduling/ Elevator Algorithm

Key Idea: Disk controller reorders access requests
Useful when many small requests, each accessing few blocks
• Transactional workload (OLTP)
• Goal: Increases the throughput


Elevator Algorithm
Analogy
Elevator moves up and down through a building
Stops whenever someone wants to enter or leave
Changes direction if no one is waiting on floor in direction of travel

Algorithm
R/W-head moves over disk from inside to outside and back 
Stops at a cylinder to process all requests for that cylinder 
Turns around when there are no more requests in that direction


**类比（Analogy）**
- 电梯在大楼中上下移动。
- 当有人要进入或离开时，电梯会停下。
- 如果在当前行进方向上没有等待的人，电梯就会改变方向。

**算法（Algorithm）**
- 磁盘的读/写磁头在磁盘上从内圈向外圈移动，然后再返回。
- 当磁头到达某个柱面（cylinder）时，会停下来处理该柱面的所有请求。
- 当当前方向上不再有待处理的请求时，磁头就会**掉头反向移动**。

![](image/Pasted%20image%2020260228224738.png)


![[Pasted image 20251108232628.png]]


### 4.2.1 Elevator Algorithm Benifits

随等待请求数量增加而提升

等待请求数 ≤ 磁道数时
• 每次寻道仅跨越少量磁道
• 平均寻道时间缩短

等待请求数 > 磁道数时
• 每个磁道对应多个请求
• 旋转延迟仅需承担一次


Increases with the number of waiting requests

Number of waiting requests <= number of cylinders
• Every seek only moves over few cylinders
• Average seek time is reduced

More requests > cylinders
• Several requests per cylinder
• Rotational delay incurred only once


Wait time for an individual request can be larger!

![[Pasted image 20251108232658.png]]






### 4.2.2 Disk Terminology and Elevator Algorithm

Disk Sections Terminology
Current slides from the lecture contradicts itself because of the illustration from Disk Summary slide (at page: 36). We will remove that slide and update the presentation. Until then, you can ignore the Disk Summary slide. The terminology built between pages [24-35] is still valid, and you can take it as reference.

Elevator (SCAN) Algorithm
You can take the description from Database Systems The Complete Book (2nd edition) in Chapter 13.3.5 (Disk Scheduling and Elevator Algorithm, page 571) as reference. I also added a link to PDF of this book's 2nd edition in the [Book Reference](https://isis.tu-berlin.de/mod/page/view.php?id=2169570 "Book Reference") page.

In essence, the disk head moves toward a direction until there is no more request to process in that direction.


磁盘分区术语
当前讲稿因"磁盘概要"幻灯片（第36页）的图示存在自相矛盾之处，我们将移除该幻灯片并更新演示文稿。在此之前，请忽略"磁盘概要"幻灯片。第24-35页建立的专业术语体系仍然有效，可将其作为参考依据。

电梯（SCAN）算法
可参考《数据库系统全书（第二版）》第13.3.5节"磁盘调度与电梯算法"（第571页）中的描述。我已在本课程的[书籍参考]页面添加该书第二版PDF的链接。
该算法的核心机制是：磁盘臂持续朝某个方向移动，直至该方向上再无待处理的请求。

#### 4.2.2.1 Small Example

Below is an illustration to clarify some possible scenarios asked during the exercises:

------(a)---->-----(b)-----(X)-----(c)------

">" is the head, moving towards right to process an existing request "X". "a","b", and "c" are potential spots for new requests to arrive. Following is some potential scenarios:

- A new request arrives at "a", i.e., it arrives at a position that the head already passed. Then this request has to wait until head processes all the request in its current direction and then come back.
- A new request arrives at "b", i.e., it arrives at a position that head is approaching but not passed yet (you have to do the math). Then head stops here and processes the new request, and then proceeds.
- A new request arrives at "c", i.e., it arrives at a position further than head is moving towards (X). Then this new request is scheduled to be processed after the first (X) one, as it is in the same direction.

If we ask a question about this algorithm, you will find all the details and assumptions in order to leave no room for confusion (e.g. start/stop time combined or separate).

If you have any questions, you can either ask here in replies or in-person next week (in Caching exercises).

以下图示用于阐明习题中可能出现的几种典型场景：

------(a)----->-----(b)-----(X)-----(c)------

符号">"代表磁头，此时正在向右移动处理现有请求"X"。"a"、"b"、"c"是可能产生新请求的潜在位置。可能出现的几种情况如下：

若新请求出现在"a"处（即磁头已越过的位置），则此请求需等待磁头处理完当前方向所有请求后，折返时才能被处理。

若新请求出现在"b"处（即磁头正在接近但尚未经过的位置，需通过计算判定），磁头将在此处停顿处理新请求，随后继续原方向移动。

若新请求出现在"c"处（即磁头移动方向更远处的目标点之后），该新请求将被调度为紧随首个请求"X"之后处理（因同属当前行进方向）。



# 5 Disk Crashes (Checksums, Stable Storage, RAID)

## 5.1 Disk Scheduling

Intermittent Failure   
- Unsuccessful read or write attempt to sector 
- Repeated attempts succeed
使用 Checksums


----

Media decay
Corrupt bits of sector(s) on disk
Repeated read attempts are not successful

Write failure
Write request to a sector cannot be completed 
Power failure a possible cause

使用 Checksums + Stable Storage 解决上面的两个文 

---
Disk crash
Whole disk cannot be read anymore 
Suddenly and permanently
使用RAID解决这个问题 


![[Pasted image 20251108232741.png]]



## 5.2 Intermittent Failure

Idea: every sector contains some bits for storing a checksum
Checksum depends on the data stored in the sector

核心理念：每个扇区预留若干比特位用于存储校验和
校验和数值取决于该扇区存储的数据内容


Parity bits
Computed for a collection of bits
Number of set bits in original collection and parity bit is even
    Odd number of 1s: 01101000, add parity bit 1 at the end → 011010001 
    Even number of 1s: 11101110, add parity bit 0 at the end → 111011100

奇偶校验位
针对一组比特位计算生成
原始数据组与校验位组合后，保证"1"的个数为偶数
• 奇数个"1"（例：01101000），末尾添加校验位"1" → 011010001
• 偶数个"1"（例：11101110），末尾添加校验位"0" → 111011100

More than 1 bit can be corrupted
50% chance that an error will not be detected
Keep several parity bits
n independent parity bits → Probability of undetected error is 1/2n

局限性说明
可能出现多位数据损坏
存在50%概率无法检测到错误
解决方案：采用多个校验位
设置n个独立校验位 → 未检测到错误的概率降至1/2ⁿ



![[Pasted image 20251108232914.png]]



## 5.3 Stable Storage

==Checksums cannot be used to correct the error==

Media decay: What was the original data? 衰变 / 衰减
Write failure: Old data is destroyed, new data is not fully written

Idea: contents of sector X is stored in two sectors (XL, XR)
==The idea is to keep two copies of every block (often called XL and XR) and verify them with checksums.==
• Checksums can discover read and write errors in XL and XR

![[Pasted image 20251108232924.png]]



Writing protocol
1. Write value for X to XL and check correctness using checksum
2. If checksum is incorrect: Repeat write n times
3. If still not correct → media error; allocate new sector for XL and go to 1
4. Repeat steps 1–3 for XR

**Goal:** Ensure that at least one correct copy of data `X` is safely stored.

**Steps**

1. **Write X to XL**
    1. Write the value `X` into the first physical block **XL**.
    2. Compute and store a **checksum**.
2. **Verify**
    1. Immediately read the block and verify the checksum.
3. **Retry if corrupted**
    1. If the checksum fails, rewrite the block up to **n times**.
4. **Handle media failure**
    1. If it still fails after `n` retries:
        1. Assume a **bad sector**.
        2. Allocate a **new sector** for XL.
        3. Repeat the process.
5. **Repeat for XR**
    1. Do the exact same procedure for the second copy **XR**.

✔ Result: two independently verified copies exist.

----


Reading protocol
- Alternatively read XL and XR until good value is returned
- If no good value is returned after n times, X is truly unreadable

**Goal:** Retrieve a correct version even if one copy is corrupted.
1. Read **XL** and check checksum.
2. If valid → return value.
3. If invalid → read **XR**.
4. Repeat up to **n times** if needed.

If neither XL nor XR returns a valid checksum after `n` attempts:

➡ The data **X is considered permanently lost/unreadable**.


![[Pasted image 20251108232931.png]]

This describes a **stable storage protocol** used in databases and operating systems to make data reliable even when disks or sectors fail. The idea is to keep **two copies of every block** (often called **XL** and **XR**) and verify them with checksums.


### 5.3.1 Why This Works

This protocol protects against:
• Torn writes
• Bad sectors
• Partial disk failures
• Transient read errors

Because:
* Two physically separate blocks exist.
* Each write is verified.
* Failed sectors are replaced.

Probability that **both copies fail simultaneously is extremely small**.


Think of it like saving an important file:

* Save **two copies**
* Verify both
* If USB sector is bad → move to a new one
* When reading → take whichever copy still works

### 5.3.2 Small Example

Writing value `X = 42`

```
Disk block XL → write 42 + checksum → verify OK
Disk block XR → write 42 + checksum → verify OK
```

Later reading:

```
Read XL → checksum fails
Read XR → checksum OK
Return 42
```

System still works even though one copy was corrupted.


If you want, I can also show:

* why databases need **stable storage for WAL logging**, or
* how this appears in **ARIES recovery** exam questions.


# 6 RAID Redundant Arrays of Inexpensive Disks

Goal: Make disks faster, bigger, and more reliable
Ideas: Combine multiple disks Store data redundantly

RAID
• Redundant Arrays of Inexpensive Disks
• Commodity hardware synthesized with software
• Looks externally like a single disk
• Data is stored in various ways ➜ RAID levels

In this lecture:
• Level 0: Block-level striping
• Level 1: Mirroring, Shadowing, or Duplexing
• Level 1+0: Striped Mirror
• Level 4: Block-level striping with parity disk
• Level 5: Block-level striping with distributed parity
• Level 6: Block-level striping with two distributed parity

Others:
Block-level striping with multiple parity disks 
Bitwise striping


----

Evaluation Metrics

Assumptions
RAID array consisting of N identical disks 
Each disk has B blocks

Metrics
Min Disks
Storage Efficiency % 
Cost
Read Performance 
Write Performance 
Protection

---

为了帮你系统地理解，我把 RAID 的标准级别（即前5层）加上常用的混合层（RAID 10），一共 6 个主要级别分别解释一下。这涵盖了最常见的 RAID 0 到 RAID 6 以及 RAID 10。

在阅读下表前，先明确两个基础概念：
Strip 条带化：把数据切成小块，分散存到不同盘，提升速度。
Mirror 镜像：数据原封不动地复制一份到另一块盘，保证安全。
checksum 校验：通过算法计算出一串校验码，如果某块盘坏了，可以用剩下的数据和校验码反推出丢失的数据。
Parity Block: Parity blocks（校验块）是 RAID 中用于实现数据容错的一种特殊数据块。它不存储原始的文件内容，而是存储通过数学计算（通常是异或算法 XOR）生成的校验信息。 简单来说，如果数据块是账本上的数字，校验块就是这些数字的总和。只要知道总和（校验块）和其中一部分数字（剩余的数据块），就能推算出丢失的那个数字（损坏的硬盘数据）。

| RAID 层级     | 核心原理                                     | 容量利用率                         | 优点                                       | 缺点                                           | 常见用途                               |
| :---------- | :--------------------------------------- | :---------------------------- | :--------------------------------------- | :------------------------------------------- | :--------------------------------- |
| **RAID 0**  | **条带化** <br>数据分散写入所有磁盘。                  | **100%** <br>（N块盘总容量）         | **读写速度极快**（并行读写），无容量损失。                  | **无容错能力** <br>任何一块盘损坏，所有数据全丢。                | 追求极致速度且不在乎数据丢失风险的场景，如视频剪辑缓存盘、游戏盘。  |
| **RAID 1**  | **镜像** <br>数据同时写入两块盘，互为备份。               | **50%** <br>（2块盘总容量的一半）       | **高安全性**（坏一块盘数据完好无损），读取速度有所提升（可从两块盘同时读）。 | **写入速度略慢**（需同时写两份），**成本高**（利用率仅50%）。         | 对数据安全性要求高的场景，如操作系统盘、数据库日志盘。        |
| **RAID 5**  | **带校验的条带化** <br>数据和校验码分散存储在所有盘。          | **(N-1)/N** <br>（相当于只损失一块盘容量） | **兼顾性能与容量**（读取快，利用率高），**允许单块盘损坏**。       | **写入较慢**（需计算校验码），**重建数据时风险高**（重建期间若再坏一盘则全丢）。 | 最常见的通用服务器配置，性价比高。                  |
| **RAID 6**  | **双重校验的条带化** <br>在RAID 5基础上增加一份校验码。      | **(N-2)/N** <br>（损失两块盘容量）     | **极高的安全性** <br>**允许同时坏两块盘**。             | **写入性能更差**（双重校验计算开销大），**重建速度慢**，成本高于RAID 5。  | 大容量存储、关键数据存储，尤其是使用大容量硬盘（如10TB以上）时。 |
| **RAID 10** | **镜像+条带化** <br>先镜像（RAID 1），再条带化（RAID 0）。 | **50%** <br>（和RAID 1一样）       | **兼顾速度与安全**（读写性能好，安全性高），重建速度快（只需复制镜像）。   | **成本高** <br>一半的容量用于镜像。                       | 高性能数据库服务器、高频交易系统等需要高IO和高可靠性的场景。    |


你问的"一共6层"通常指的是上述这些。但在严格的定义中，RAID 还有两个很少用到的层级：

-   **RAID 2**、**RAID 3**、**RAID 4**：这些在历史上存在过，但因为实现复杂或效率问题，在现代实际应用中基本已被 RAID 5 和 RAID 6 取代，因此通常不列入日常讨论的"主要层级"中。

总结一下选择倾向：

-   要**速度**且不怕丢数据：选 **RAID 0**
-   要**安全**且不在乎成本：选 **RAID 1**
-   要**性价比**且只允许坏一块盘：选 **RAID 5**
-   要**极致安全**且容量大：选 **RAID 6**
-   要**又快又安全**且预算充足：选 **RAID 10**

---


![[Pasted image 20251108233318.png]]




## 6.1 Reliability Metrics

Mean Time To Failure (MTTF) - average lifespan on one disk
Time after which 50% of disks fail
10 years for modern disks
Most disks fail either early due to manufacturing defects or late due to normal wear and tear
MTTF = nb_hours_per_disk/N => hours for a disk failure

Mean Time To Data Loss (MTTDL) — for an entire system as a collection of disks
Time after which data is lost irrevocably 

Without RAID:
If one disk fails the entire system fails
50.000 hours (MTTF per disk) / 100 disks => MTTDL = 500 hours 
Common data center has thousands of disks!



这段内容讲的是磁盘可靠性指标，以及在大规模存储系统中，如果不做保护（如RAID），数据会变得多么脆弱。以下是详细的解释：

MTTF：平均无故障时间
-   **定义**：指一块磁盘的平均寿命。
-   **统计意义**：它不是一个精确的到期时间，而是指 statistically, after that time, about 50% of the disks have failed. 也就是说，MTTF 是故障发生的中位时间。
-   **现代磁盘**：现代磁盘的 MTTF 通常标注为 **10年** 左右（约 87600 小时，实际标注通常为 100万到200万小时，但那是在实验室条件下的平均数，这里按通俗逻辑简化）。
-   **故障规律**：磁盘故障率通常遵循浴缸曲线。
    1.  **早期**：因制造缺陷，故障率较高。
    2.  **中期**：稳定期，故障率低。
    3.  **晚期**：因磨损，故障率再次升高。


从 MTTF 到故障率
这里有一个关键换算公式：
`MTTF = 总运行时间 / 故障次数`

推导出：**故障率 = 磁盘数量 / MTTF**
即：`nb_hours_per_disk / N`


MTTDL：平均数据丢失时间
-   **定义**：针对**整个存储系统**而言，数据发生不可逆丢失的平均时间。
-   **核心区别**：MTTF 看的是单块盘什么时候坏；MTTDL 看的是整个系统什么时候彻底完蛋。

### 6.1.1 无 RAID 时的可怕数学
假设一个数据中心的场景：
-   **参数**：
    -   单盘 MTTF = 50,000 小时。
    -   系统有 100 块盘。
-   **计算**：
    MTTDL = MTTF / 磁盘数量 = 50,000 小时 / 100 = **500 小时**。

**这意味着什么？**
-   500 小时大约是 **20.8 天**。
-   在没有 RAID 的情况下，虽然有 100 块盘，但因为数据没有冗余，**任何一块盘坏了，整个系统的数据就全丢了**。
-   结论：**平均每 21 天，这个系统就会彻底崩溃一次。**

### 6.1.2 扩展思考
你提供的最后一句也提到了重点：**Common data center has thousands of disks!**

-   如果是一个有 5000 块盘的数据中心：
    MTTDL = 50,000 / 5000 = **10 小时**。
-   这意味着，如果不做任何冗余保护（RAID或复制），这个数据中心平均每 10 小时就会发生一次全量数据丢失的重大事故——这是完全不可接受的。

**总结：**
MTTF 看的是单盘寿命，MTTDL 看的是系统整体可靠性。**磁盘越多，整体系统越脆弱**，这就是为什么大规模存储系统必须引入 RAID 或多副本机制来显著延长 MTTDL。


## 6.2 RAID 0 : Block-level striping

Stripe = blocks distribution across all disks

![[Pasted image 20251108233229.png]]



## 6.3 RAID 1: Mirroring, Shadowing, or Duplexing

Stripe = block replication

![[Pasted image 20251108233237.png]]


## 6.4 RAID  1+0: Striped Mirror 

Block replication + distribution

![[Pasted image 20251108233244.png]]


## 6.5 RAID4: Block-level Striping with Parity Disk 

Single parity disks for multiple data disks

![[Pasted image 20251108233251.png]]


## 6.6 RAID5: Block-level Striping with Distributed Parity 

Rotate the parity blocks across all disks

![[Pasted image 20251108233258.png]]


## 6.7 RAID 6: Block-level Striping with Doubled-Distributed Parity 

Rotate the 2x parity blocks across all disks

![[Pasted image 20251108233306.png]]


