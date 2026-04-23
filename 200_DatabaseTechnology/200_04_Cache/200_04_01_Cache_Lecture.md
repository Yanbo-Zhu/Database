

Caching: General, DB System Caching, Buffer Manager

![](image/Pasted%20image%2020260301105955.png)



# 1 disk, buffer manager, main momery,  cpu register 之间的关系

在数据库或操作系统中，数据从 **磁盘到 CPU 处理** 通常经过一个层级结构。可以简单理解为 **越往上越快、越小、越贵**。👇

---
Disk（磁盘）

* **作用**：长期存储数据（数据库文件、日志等）
* **特点**：

  * 容量最大
  * 速度最慢
  * 按 **block/page** 读写
* 数据不能直接被 CPU 使用，必须先加载到内存。

---

Buffer Manager（缓冲管理器）
* 位于 **数据库系统内部的组件**。
* **作用**：
  * 管理磁盘页在内存中的缓存。
  * 决定：
    * 什么时候把数据从磁盘读到内存
    * 哪些页留在内存
    * 哪些页被替换（LRU 等策略）

简单理解：

```
Disk  ←→  Buffer Manager  ←→  Main Memory
```

它像一个"调度员"。

---

Main Memory（主存 / RAM）
* **CPU 可以直接访问的工作区**
* 数据库把磁盘页加载到这里进行操作。
* 比磁盘快很多，但容量更小。

例子：

```
磁盘中的表数据
   ↓
读入 RAM 中的 buffer page
```

---

CPU Register（CPU 寄存器）
* 位于 **CPU 内部**
* **速度最快**
* 用于：
  * 临时存储计算中的数据
  * 执行指令

数据处理时流程通常是：

```
Main Memory → CPU Register → 运算 → 写回 Memory
```

---

## 1.1 整体数据流

典型流程：

```
Disk
  ↓
Buffer Manager 把 page 读入
  ↓
Main Memory (Buffer Pool)
  ↓
CPU 读取到 Register
  ↓
执行计算
```

---


* **Disk**：永久存储
* **Buffer Manager**：管理磁盘和内存之间的数据交换
* **Main Memory**：运行时数据所在位置
* **CPU Register**：CPU 计算时的最快存储

速度排序：

```
CPU Register  >  Main Memory  >  Disk
```

---

如果你是在学 **Database Systems / Storage Hierarchy**，老师通常会画这个经典结构，我也可以帮你做一个 **考试用的记忆图**。


# 2 General Concept 

 General concept in computer science
‣ Memory hierarchy enables caching
‣ Caching improves access speed
‣ The improvement depends on many factors, e.g., access order, access types, cache size, etc

![](image/Pasted%20image%2020260301105143.png)



---

Cache 发生在哪里 

OS can do the caching totally transparent for you but DBMS’s want to have full access, e.g., when to load and when to evict.
OS Caching highly depends on the underlying kernel/ implementation and is not always comprehensible.

![](image/Pasted%20image%2020260301105221.png)






# 3 Database System Caching


• DBS benefits from hardware (and OS) caching
• Database buffer manager (focus of the lecture)

Query execution
• Intermediate results (e.g., for sorting)
• Auxiliary structures (e.g., hash table for join)

Overall query processing
• Query result cache (inside or outside of DBS)
• Logging
• Plan cache

Application-level cache may avoid DBS access




# 4 Database System Caching: Buffer Manager

Responsibilities:
• Provide access to blocks on disk to database processes
• Read blocks into main memory
• Cache blocks for faster access
• The DBMS is responsible for it
• OS allocates it in virtual memory (e.g., malloc)



Architecture
• The DBMS is responsible for it
• OS allocates it in virtual memory


![](image/Pasted%20image%2020260301110045.png)



## 4.1 Benefits of Caching

• Reduced disk I/O (load it once, use it often)
• Faster Reads (from higher memory hierarchy level)

• But also, faster writes:
    • Change only in memory until block is written to disk
    • Possibility of batching changes and cluster data
    No change is final until it has been persisted on disk (Either data or log => see later lecture)



## 4.2 Challenges 


![[Pasted image 20251109121130.png]]



![[Pasted image 20251109120539.png]]

# 5 Buffer Management Strategies


## 5.1 Buffer Manager Scope

What is cached by the buffer manager?
• Table pages
• Index pages
• Metadata pages

Note: The system requires further memory for query processing,
e.g, for building a hash table or sorting

## 5.2 Granularity of Buffers

• Records
- Not used because replacement and reloading too costly
• Blocks/pages
- Commonly used
- Either OS blocks or database blocks
• Chunks
- Combine blocks into larger “chunks”
- Can exploit sequentially placed blocks on disk
- Good for very large operations (large table joins or sorts)
• Whole Tables
- Fix all blocks of heavily used tables


## 5.3 Pinning

• Blocks/pages in use are “pinned” and forbidden to be evicted
• Buffered blocks can be pinned by multiple execution units


## 5.4 Eviction Policy

Eviction Policy - Things to Consider
• Age
- Time since block was loaded
- Last time accessed
• Accesses
- Number of accesses
• Trade-offs
- New blocks → low access counts, but involved in current operations
- Old blocks → high access counts, constant but potentially less frequent use
Mix of age and accesses Other factors: Pinned blocks, operation access pattern, overhead

驱逐策略——考量因素
• 存活时间

自数据块载入以来的时长

最近一次访问时间
• 访问次数

被访问的频次
• 权衡因素

新载入块 → 访问次数少，但参与当前操作

旧数据块 → 访问次数多，稳定但潜在使用频率较低
存活时间与访问次数的结合
其他因素：锁定块、操作访问模式、系统开销



Eviction Policies
No single policy is optimal; best depends on workload pattern
Common strategies, also in other applications (e.g., operating system); Simple, low-overhead strategies are surprisingly good:
• First-In-First-Out (FIFO): Eviction based on position
• Least Recently Used (LRU): Eviction based on time
• Least Frequently Used (LFU): Eviction based on usage
• CLOCK (sometimes also called second-chance): commonly implemented, efficient approximation to LRU

通用策略，亦见于其他应用场景（如操作系统）；简单且低开销的策略效果出乎意料地好：
• 先进先出：基于载入位置进行驱逐
• 最近最少使用：基于访问时间进行驱逐
• 最不常使用：基于使用频率进行驱逐
• CLOCK算法（亦称二次机会法）：广泛实现的算法，是对LRU的高效近似

![](image/Pasted%20image%2020260301113043.png)



### 5.4.1 FIFO, LRU, MRU, LFU
First In First Out
• Easy to implement; low overhead
• Does not consider access recency or frequency

**先进先出（FIFO）**  
• 易于实现，系统开销低  
• 不考虑访问的新近程度或访问频率  

----

Least Recently Used
• Considers access recency (which is intuitive)
• Requires maintenance of the access recency order (i.e., for every access)
• Good for temporal locality
• Poor for large sequential scans (cache pollution)
Good if you access data in a burst and then forget it like in this query plans

**最近最少使用（LRU）**  
• 考虑访问的新近程度（符合直观认知）  
• 需要维护访问新近程度的顺序（即每次访问都需要更新）  
• 对时间局部性友好  
• 对大型顺序扫描不友好（导致缓存污染）  
适用于突发访问后即丢弃数据的场景（如某些查询计划）  

生活例子：手机后台应用切换
你打开了微信、淘宝、抖音。然后你又切回微信看了一眼。这时候系统觉得你最可能继续用微信，所以如果内存不够了，它会优先关掉你很久没看的淘宝（最近最少使用），而不是微信。
缓存场景：
容量为 3，访问顺序：A， B， C， 然后再次访问 A，最后请求 D。
访问 A：[A]
访问 B：[A， B]
访问 C：[A， B， C]
再次访问 A：A 被使用了，挪到最新位置 [B， C， A] （B 变成了最久未使用的）
请求 D：容量满，淘汰最久未使用的 B，变成 [C， A， D]

----


Most Recently Used
- Good for sequential and repeating patterns , like table scans
- poor for hot-spot reuse
**最常使用（MRU）**  
- 对顺序扫描和重复模式友好（如表扫描）  
- 对热点数据重用不友好  

生活例子：翻杂志找图
你有一堆杂志叠在一起，你刚看完一本（刚使用过），短时间内你不会再看它了。你想看的是那些压在下面很久没翻的。所以你会把刚看完的这本丢到一边（移除 MRU）。

缓存场景（大表扫描）：
假设要扫描一个大表，数据块是 A， B， C， D， E...（顺序访问）。
如果是 LRU，刚扫完 A，A 被放到最近位置，但接下来要扫 B、C、D... 内存里存的全是刚扫过的旧数据（污染缓存），新数据反而进不来。
如果是 MRU，刚扫完 A（最近使用的），立刻把 A 淘汰掉，给 B 腾地方。这样缓存里永远存的是"将要"或"正在"使用的数据。


----


Least Frequently Used
-Good if parts are often accessed over and over again like the root of a tree
**最不常使用（LFU）**  
- 适用于某些部分被频繁反复访问的场景（如树的根节点）

核心逻辑：移除使用次数最少的数据。
生活例子：图书馆的畅销书架
图书馆有一个小架子放热门书。如果书被借了 100 次（访问频率高），它就一直留在架子上。如果一本书只有人借过 1 次，新书来了，就把这本借 1 次的清走。

缓存场景：
容量为 3，访问顺序：A， A， B， B， C， 然后 D 进入。
访问计数：
A: 2次
B: 2次
C: 1次
请求 D：容量满，淘汰次数最少的 C。



![[Pasted image 20251109121437.png]]


![[Pasted image 20251109121445.png]]


### 5.4.2 CLOCK also called Second Chance


核心逻辑：给每个数据一个"二次机会"标志位。相当于一个简化的 LRU，避免每次访问都去调整链表顺序。

生活例子：饭店的旋转寿司
寿司在传送带上转（时钟指针在转）。每盘寿司有一个小旗子（使用位）。
如果你从传送带上拿了一盘（访问），服务生会把小旗子立起来（标志位=1）。
传送带不停转，服务生巡视。如果看到一盘寿司的小旗子立着（近期被访问过），就把旗子放倒（给一次机会），让它再转一圈。
如果看到一盘寿司的小旗子是倒着的（很久没人吃），就把它端走（淘汰）。


General Idea: Improvement over FIFO; approximate LRU with less overhead. Take the accesses to a page between two points in time into account. Reduce the risk of evicting a page that is in use.

Overview:
• Candidates to be replaced are considered in a round-robin manner
• A page that has been accessed between consecutive considerations will not be
removed
• Replace the page that has not been accessed since its last consideration
• An access is indicated by setting the second chance bit to one
• A page with the "second chance" bit set to 1 is never replaced during the first consideration and will only be replaced if all the other pages deserve a second chance too

• 以轮询方式考察待置换的候选页面
• 在两次连续考察consideration 之间被访问过的页面不会被移除
• 置换自上次考察后未被访问过的页面
• 通过将"二次机会位"设置为 1 来标记页面已被访问
• 二次机会位为 1 的页面在首次被考察时永远不会被置换，仅当所有其他页面都获得二次机会后，该页面才可能被置换



### 5.4.3 CLOCK Algorithm


Second Chance Algorithm: Think of buffers as arranged in a cicle


- 当某个页面被访问到，并且它**已经在缓存中**时：  
    → 它的 **引用位（reference bit）** 会被设置为 **1**。  在轮序的时候 其他的 page 的 reference bit 的值不会改变 
    - ⚠️ 此时 **时钟指针（clock hand）不会移动**，即保持原位。
- 时钟指针 **只在需要淘汰页面（即缓存缺页、要找替换页）** 时才会移动，  并按顺时针方向依次检查缓存中的页面。
	- 在轮序的时候 其他的 page 的 reference bit 的值会改变 
		-  如果原本是 1，则改成 0；
	    - 如果原本已经是 0，新页面被装 装到这里。
	- 当一个新页面被装入缓存时，它的 **引用位被设置为 1**。
	-  **时钟指针（clock hand）移动**， 指向新页面的下一个页面。


insertion
- Pointer to page in ring buffer
- Each buffer has a reference bit
- Insert new page if buffer is empty => Store at pointer position, set reference bit to 1

reference a existing page in cache
- When referencing a cached page: (no pointer movement)
- If 0 => Set reference bit to 1
- If 1 => Keep it to 1
通过指针指向环形缓冲区中的页面位置
每个缓冲区页面设有一个引用位\
当缓冲区为空时插入新页面 → 将页面存入指针当前位置，并将引用位设置为 1
当访问已缓存的页面时：（指针不移动）
若引用位为 0 → 将其设置为 1
若引用位为 1 → 保持为 1


Evict old page:
1. Rotate pointer clockwise through ring buffer
2. If page bit is 1 then set to 0 and continue rotation
3. If reference bit is 0, then evict page and load new page into buffer (no pointer movement after loading)

将指针沿环形缓冲区顺时针旋转
若当前页面的引用位为 1，则将其设置为 0 并继续旋转
若引用位为 0，则淘汰该页面并将新页面载入缓冲区（载入后指针不移动）


![[Pasted image 20251109121600.png]]


全访问一遍 看有没有空位， 在一个个访问的时候 把1  set to 0 
![[Pasted image 20251109121611.png]]

![[Pasted image 20251109121617.png]]

![[Pasted image 20251109121623.png]]

----


Implementation
• In general: Eviction policies are implemented on top of common data structures, e.g., queues
• Some implementations of these data structures (e.g., circular array vs. linked list) are more suited for some policies (e.g., functionality or performance-wise)


Implementation: PostgreSQL Buffer Manager
Implements (an extended) clock algorithm
• Buffer pool: array of data pages (with an implicit buffer_id)
• Buffer descriptors: array of descriptors with metadata (tag/content_id, is dirty, usage count, pins) of stored pages
• Buffer table: hash table for efficiently looking up stored pages (based on tag/content_id)

实现：PostgreSQL 缓冲区管理器
采用（扩展版的）时钟算法
• 缓冲池：数据页数组（包含隐式的 buffer_id）
• 缓冲区描述符：存储页面元数据的描述符数组（包括标签/内容标识、脏页标记、使用计数、固定计数）
• 缓冲区表：基于标签/内容标识高效查找已存储页面的哈希表


![](image/Pasted%20image%2020260301114558.png)



## 5.5 Prefetching

Prefetching Pages
• Load blocks not yet needed now, but hopefully soon
• Examples:
- If disk sector is requested, read entire track
- Read from other cylinders without moving head (data is stored usually cylinder by cylinder)
- If a block from relation is requested, load next few blocks
• Using sequential and asynchronous, non-blocking reads, prefetching
often cost little and can save a lot of time

• 加载当前尚未需要但预计不久将用到的数据块
• 示例场景：
    当请求某磁盘扇区时，读取整条磁道
    在不移动磁头的情况下读取其他柱面（数据通常按柱面顺序存储）
    当请求关系中的某数据块时，加载后续若干数据块
• 采用顺序异步非阻塞读取方式，预取通常开销极小且能显著节省时间

---


Semantic Caching 语义缓存

• Cache query result
- Q1: SELECT * FROM person WHERE birth_year > 1980
- Q2: SELECT name FROM person WHERE birth_year > 1990
- Q2 can be answered using result tuples from Q1
• Powerful but challenging technique
- When can we use results of one or more other queries?
- Query containment, “answering queries using views”
• Semantic caching is not used by any commercial database today
- Note: Normal caching can mimic semantic caching
- If Q2 executed after Q1, blocks from Q1 are in cache (“hot” cache)
- But: Computations need to be repeated (e.g., aggregation)

缓存查询结果
查询 Q1：SELECT * FROM person WHERE birth_year > 1980
查询 Q2：SELECT name FROM person WHERE birth_year > 1990
Q2 可利用 Q1 的结果元组来回答


能强大但具有挑战性的技术
何时能够利用一个或多个其他查询的结果？
涉及查询包含关系、"基于视图回答查询"等复杂问题

目前尚无商业数据库采用语义缓存
注：传统缓存可模拟语义缓存的效果
若 Q2 在 Q1 之后执行，Q1 涉及的数据块仍保留在缓存中（"热"缓存）
但计算过程需重复执行（如聚合操作）




## 5.6 Buffer Manager Challenges of Today

What is the best granularity?
How to handle references efficiently? 
What is the best replacement strategy? 
How to evict pages efficiently?
How do we handle synchronization? 
How to best translate addresses?

## 5.7 Extended Approaches

Lean Store Overview
Key ideas:
In-Memory performance for SSD-based DBMS
Low overhead implementation (e.g., pointer tagging instead of hash lookup) 
Modify common buffer managers in many places


----

Homework for Today

Read LeanStore Paper: https://db.in.tum.de/~leis/papers/leanstore.pdf (cutting-edge buffer manager) 
Read VMCache Paper: https://www.cs.cit.tum.de/fileadmin/w00cfj/dis/_my_direct_uploads/vmcache.pdf (Conceptual comparison of state-of-the-art buffer manager)

Try to implement a queue and a circular buffer on your own in your preferred language
- Claim: Only if you have implemented a data structure once on your own, you really understand how it works
- Suggestion: start with an empty template and try first by your own, only if you cannot proceed, look into the examples:
    - Queue: https://www.geeksforgeeks.org/queue-data-structure/
    - Circular Buffer: https://www.geeksforgeeks.org/introduction-to-circular-queue/ 
- Expert Task: Build an LRU cache on your own (hints: https://redis.io/glossary/lru-cache/)
