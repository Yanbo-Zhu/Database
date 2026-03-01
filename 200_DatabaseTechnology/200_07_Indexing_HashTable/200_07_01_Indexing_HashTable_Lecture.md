
# 1 Indexing

An Index is a data structure that finds the record(s) efficiently given a search key

Index can be:  
- Ordered index: Based on a sorted ordering of values (e.g., B-tree) 
- Hash index: Based on a distribution of key to hash values, determined by a hash function

![](image/Pasted%20image%2020260119172302.png)




# 2 Hash Tables: Basic Principles

Hash table: data structure that implements an associative array, storing key-value pairs

Hash function h(K)  
• Input: Search key K (Hash key)  
• Output: Integer between 0 and B-1 , B is the number of buckets


Desired Properties
• Uniform distribution (across the hash space)
• Random distribution (low collision probability)

Simple examples  
• For integers: K % B
• For strings: Use an integer for every letter and sum these up  
In practice, more complex bit-shifting and operations (e.g., MurmurHash)

![](image/Pasted%20image%2020260119173418.png)


## 2.1 Collision Handling


Main problem is collisions, i.e., K1 ≠ K2, but h(K1) = h(K2)
Finding a good hash function
- bit-shuffling and operations “perfect hash function”

Birthday Paradox:
With only 23 persons, the probability that two people have the same birthday is 50%.

Use mechanism to handle collision (instead of huge hash tables)  because a collision-free function cannot be guaranteed for unseen data
- Separate chaining 
- Open addressing

![](image/Pasted%20image%2020260119173856.png)

Separate Chaining
use (often linked) list to store entries with the same hash value

Open Addressing
- use some probe sequence if slot is occupied, e.g., linear probing (below), quadratic probing, cuckoo hashing

![](image/Pasted%20image%2020260119173911.png)

![](image/Pasted%20image%2020260119201240.png)



### 2.1.1 Collision Handling: Separate Chaining

‣ Advantage:
    ‣ Easy to implement 
    ‣ Never fills up
‣Disadvantage:
    ‣ Following linked list is inefficient
        ‣ Worst case O(n) search time 
    ‣ Potential waist of storage:
        ‣Empty slots and link information 
    ‣ Possible data structures for storing:
        ‣ Linked list, dynamic sized array, trees, etc.

![](image/Pasted%20image%2020260119202142.png)


### 2.1.2 Collision Handling: Open Addressing

‣ Method to resolve hash collisions
‣ Searching through alternative locations (sequence) until value is found or unused slot is found (indicating entry does not exists).
‣ Common sequences are:
    ‣ Linear Probing: Intervall between consecutive probes is fixed, e.g., 1
    ‣ Quadratic Probing: Intervall between consecutive probes increases quadratically
    ‣ Double Hashing: Intervall between consecutive probes is determined by another hash function

通过备选位置序列进行搜索，直到找到目标值或找到未使用的槽位（表示该条目不存在）。

常见的探测序列包括：
线性探测：连续探测之间的间隔是固定的，例如 1
二次探测：连续探测之间的间隔呈二次方增长
双重哈希：连续探测之间的间隔由另一个哈希函数决定



![](image/Pasted%20image%2020260119202223.png)



![](image/Pasted%20image%2020260119202249.png)

![](image/Pasted%20image%2020260119202257.png)

### 2.1.3 Comparision

![](image/Pasted%20image%2020260119202304.png)


![](image/Pasted%20image%2020260119202311.png)

## 2.2 Static vs. Dynamic Hash Tables

Repeated insertions fill up hash tables (and deletions may lead to wasted space)

Chaining
- Pre-allocated number of buckets, theoretically unlimited number of possible entries due to chaining (overflow blocks)
- Degrading performance with increasing fill level due to increasing length of linked lists
预先分配固定数量的桶，由于可以使用链表（溢出块），理论上可存储的条目数量无上限
随着填充率升高，链表长度增加，性能下降


Open addressing: 
- Pre-allocated number of buckets, limited number of possible entries 
- Degrading performance with increasing fill level due to increasing number of required probes
预先分配固定数量的桶，可存储的条目数量有限
随着填充率升高，需要的探测次数增加，性能下降


—> Dynamic resizing necessary or desirable
Simple approach: creating a new hash table and rehashing all entries Possible but computationally expensive
简单方法：创建新的哈希表，将所有条目重新哈希
这种方法可行，但计算开销很大



# 3 Secondary-Storage Hash Tables

So far: Hash tables in main memory

DBMS: Bucket array consists of blocks/pages  。  ==在这里 block就是等效于page.  一个 block/page 里面可以存若干个 record. 一个bucket 只有一个 block/page==
• Records hashed to the same value are stored in the same block
• Overflow blocks can be used for collision handling

Bucket array
• Array of Bucket headers for Bucket chained lists

![](image/Pasted%20image%2020260119175000.png)


## 3.1 Inserting into a Hash Table

1. Compute hash  
2. If room, insert record into corresponding block
    1. Sort order in block does not matter
    2. Or store in an overflow block that has room  
3. If no room, create overflow block and insert into

![](image/Pasted%20image%2020260119175112.png)

![](image/Pasted%20image%2020260119175131.png)


## 3.2 Deleting from a Hash Table

1. Compute hash
    1. Search record/records in bucket  (including overflow blocks)
2. Delete it/them
3. Possibly reorganisation and removal of overflow block    

Example: Delete C, move G 

![](image/Pasted%20image%2020260119175219.png)

![](image/Pasted%20image%2020260119175208.png)


## 3.3 Analysis of “Static” Hash Tables


Ideally: Only one block/page per bucket   (就是下图中 一个 0,1,2,3, 代表一个bucket )
• Search: 1 I/O
• Insert/Delete: 2 I/Os
• Much better than dense or sparse indexes
• Better than B-trees


Disadvantages:
• No support for range queries (neither sequential disk access nor information which keys are stored)
• Number of buckets B is fixed (“static hashing”)
    • Long lists of overflow blocks or limited number of entries
Solution: Dynamic hash tables, which adapt their size dynamically

不支持范围查询（既无法顺序访问磁盘，也无法知道存储了哪些键）

桶的数量 B 是固定的（"静态哈希"）  要么会有很长的溢出块链表，要么可存储的条目数量受限

解决方案：动态哈希表，能够动态调整自身大小

# 4 Dynamic Hash Tables


## 4.1 Static vs. Dynamic Hash Tables

So far: number of buckets B was fixed (static hash tables)
- Open addressing 
- Chained bucket hashing

Several kinds of dynamic hash tables:
- number of buckets B allowed to vary  
- B ≈ Number of records / number of records per block

Here:
- Extensible hashing 
- Linear hashing

![](image/Pasted%20image%2020260119193434.png)


## 4.2 Extensible Hash Table

Idea: Create new buckets if overflow block would be necessary 

Indirection for buckets: Use an array of pointers to blocks  instead of array of blocks

Dynamic Growth
•Size of pointer array doubles when needed

Sharing
• Buckets can share data blocks... 
• ... if room is available

Hash function:
Computes a sequence of k bits for each key (e.g., k = 32) 
Bucket numbers use only i bits (i ≤ k)
Bucket array consists of 2^i buckets


![](image/Pasted%20image%2020260119193824.png)


![](image/Pasted%20image%2020260119193958.png)





### 4.2.1 Example:  Inserting into Extensible Hash Tables


![](image/Pasted%20image%2020260119194751.png)

![](image/Pasted%20image%2020260119194802.png)


![](image/Pasted%20image%2020260119194518.png)


![](image/Pasted%20image%2020260119194529.png)


![](image/Pasted%20image%2020260119194536.png)



### 4.2.2 Analysis of Extensible Hashing
Advantages:
Search never must look at more than one data block (No overflow blocks) 
Bucket array may fit into main memory at the beginning
Reorganization is performed on only one bucket at a time
搜索时最多只需要访问一个数据块（没有溢出块）
初始时桶数组可能可以放入主内存
每次只对一个桶进行重组织（reorganization）


Disadvantages:
- When i is large doubling bucket array is a lot of work
- Size of every bucket is fixed —> Non-uniform key distribution leads to many pointers pointing to the same blocks  and potentially empty blocks (i.e., a wast of storage)
- As bucket array doubles, it may not fit in memory (causing an additional I/O)

当全局深度 i 较大时，桶数组翻倍的工作量很大
每个桶的大小是固定的 → 键分布不均匀会导致很多指针指向同一个块，同时可能存在空块（即浪费存储空间）
桶数组翻倍后，可能无法放入内存（导致额外的 I/O）

## 4.3 Linear Hash Table

Idea: Number of buckets grows (and shrinks) in a linear fashion  (i.e., one bucket at a time)
- Compared to extensible hashing, no bucket directory
- Overflows are handled by creating a chain of blocks,  but average number of overflow blocks per bucket is much smaller than 1 
- Choose number of buckets n such that the average number of records per bucket is a fixed fraction, e.g., 85%
- Use ceil(log2n) bits to identify a bucket, always use the rightmost (LSB) bits of the hash value


**核心思想：** 桶的数量以线性方式增长（和收缩）（即一次只增加一个桶）
- 与可扩展哈希相比，**没有桶目录**
- 溢出通过创建块链来处理，但每个桶的平均溢出块数**远小于 1**
- 选择桶的数量 n，使得每个桶的平均记录数是一个固定的比例，例如 **85%**
- 使用 **ceil(log₂n) 位**来标识一个桶，总是使用哈希值的**最右边（最低有效位，LSB）**位


![](image/Pasted%20image%2020260119195753.png)

### 4.3.1 Inserting into Linear Hash Tables

![](image/Pasted%20image%2020260119195939.png)




i: number of relevant bits n: number of buckets
n: number of buckets
r: number of records

1 Compute h(k)

2 Consider rightmost i bits; interpret as integer m
If m < n: Insert record into bucket m
If m ≥ n: Bucket m does not exist yet; Compute bucket ID using i-1 bits 3. 


3 If no space for insertion:
Create overflow block and insert

4  Check r/n, if too high (e.g., ≥ 1.7 if blocks hold up to 2 records,  and an 85% fill level is targeted):
Create one new bucket n+1
If the ID of new bucket is 1xxx, rehash records of bucket 0xxx into 1xxx and 0xxx (xxx stands for the same bit pattern)


5 If now n > 2i:
i = i +1
All buckets IDs are extended by one bit and start with 0 
No physical change of the hash table

---

**i：相关位数**
**n：桶的数量**
**r：记录的数量**

1. 计算哈希值 **h(k)**

2. 取最右边的 **i 位**，将其解释为整数 **m**
   - 如果 **m < n**：将记录插入桶 m
   - 如果 **m ≥ n**：桶 m 还不存在；使用 **i-1 位**计算桶 ID（即去掉最高位，用剩下的位决定实际桶号）

3. 如果插入时没有空间：
   - 创建溢出块，将记录插入溢出块中

4. 检查 **r/n**（平均每个桶的记录数）
   - 如果太高（例如每个块最多放 2 条记录，目标填充率 85%，则 r/n ≥ 1.7 时触发扩容）：
     - 创建一个新桶 **n+1**
     - 如果新桶的 ID 是 **1xxx**，则将桶 **0xxx** 中的记录重新哈希分配到 **0xxx** 和 **1xxx**（xxx 表示相同的低位模式）

1. 如果现在 **==n > 2ⁱ**==：
   - **i = i + 1**
   - 所有桶的 ID 扩展一位，**最高位补 0**
   - 哈希表物理结构不变（只是逻辑上的位数增加）

### 4.3.2 Example linear Hash Tables

![](image/Pasted%20image%2020260119200619.png)



![](image/Pasted%20image%2020260119200627.png)


![](image/Pasted%20image%2020260119200636.png)


![](image/Pasted%20image%2020260119200655.png)

![](image/Pasted%20image%2020260119200705.png)

![](image/Pasted%20image%2020260119200810.png)

consider the most right 2 bits: 0111 -> 11
m = 2^1 x 1+ 2^ 0 x1    =3 
m=3 >= n=3 , comput bucket ID with the most right i-1 bits, 0111 -> 1 , extend 1 -> 01
then insert records into Bucket with ID 01 

![](image/Pasted%20image%2020260119200832.png)


# 5 B+-Tree vs. Hash Table

Trees and hash tables can be optimised for block/page access and database operations. Both are useful index structures for database systems.


**B+ Tree**
- 一种 **多路平衡搜索树**，所有数据都在叶子节点，叶子节点之间通过链表相连。  
- 常用于：数据库索引（MySQL、PostgreSQL）、文件系统（EXT4、NTFS）
- Better worst-case complexity of operations (search, insert, delete)
- Balance and space-efficiency guarantees
- Support for range queries


**Hash Table**
- 一种通过 **哈希函数**把 key 映射到槽位(bucket)** 的数据结构。  
- 常用于：编程语言的 key-value 存储（Python dict、Java HashMap）
- Faster look-ups, particularly for accessing single records
- B+-Trees are more versatile and usually the default index structure for secondary indexes


![](image/Pasted%20image%2020260119202327.png)


## 5.1 性能比较 

|操作|B+ Tree|Hash Table|
|---|---|---|
|查找|O(log N)|O(1) 平均|
|插入|O(log N)|O(1) 平均|
|删除|O(log N)|O(1) 平均|
|范围查询|**O(log N + K)**|❌ 不支持（必须遍历整个表）|
|顺序遍历|**有序（叶子链表）**|❌ 无序|

## 5.2 B+ Tree 优缺点

**优点**
**支持范围查询（range scan）**
如 `age > 30 and age < 50`  
B+ 树是有序的，叶子节点串成链表，非常适合：
- BETWEEN 查询
- ORDER BY
- LIMIT + OFFSET
- 范围统计

**磁盘友好（IO 优化）**
数据库页通常是 **4KB 或 16KB**，B+ 树每个节点存大量 key，因此：
- 树高度非常低（一般 3 层可支持百万级数据）
- 查询只需少量磁盘 I/O

**稳定性高**
插入、删除都能保持平衡，不会退化。

---

**缺点**
1. 查询速度比哈希表慢（O(log N)）
2. 单 key-value point lookup 不如 Hash 快
3. 实现复杂


---

**为什么 B+-Tree 每个节点能存大量 key？**
**核心原因：B+-Tree 的节点大小通常按“磁盘页（page size）”设计**
数据库和文件系统处理的数据不在内存里，而在 **磁盘** 上。
- 读写磁盘的最小单位是 **page**（例如 4 KB、8 KB、16 KB）
- **B+-Tree 的一个节点大小就等于一个 page 的大小**
**→ 这意味着每次访问 B+-Tree 的一个节点，就是一次磁盘 IO**  
→ 所以节点内尽量放更多 key，减少树的高度，就能减少 IO。

## 5.3 Hash Table 优缺点

**优点**
1. **查询速度快（平均 O(1)）**  
    对于 k->v 映射非常高效。
2. 插入、删除也很快
3. 实现简单

----

**缺点**
**无法做范围查询**

因为 key 是无序的，例如要找 `10 < key < 30`，只能：
❌ 遍历所有 bucket  
❌ 性能极差

**哈希冲突（collision）**
需要：链表拉链 / 开放寻址  
可能导致退化性能

**扩容成本高**
增长到一定容量必须 rehash，代价很大。

**不适合外存结构**
Hash Table 对磁盘 **非常不友好**：
- 随机访问太多
- 容易导致大量 random IO


## 5.4 为什么数据库更喜欢 B+ Tree，而不是 Hash 表 

|需求|Hash Table|B+ Tree|
|---|---|---|
|单点查询|快|较快|
|范围查询|❌ 不支持|✔️ 支持|
|排序|❌ 不支持|✔️ 支持|
|磁盘结构优化|❌ 差|✔️ 优秀|
|高并发写入|❌ rehash 成本高|✔️ 平稳可预期|

数据库（如 MySQL、PostgreSQL）需要：
- ORDER BY
- GROUP BY
- BETWEEN
- LIMIT
- 排序
- 全表范围扫描
- 磁盘顺序读
=> **只有 B+ 树能满足**

因此：
- **MySQL InnoDB 索引 = B+ Tree**
- **PostgreSQL 索引 = B+ Tree**
- **文件系统目录 = B+ Tree**
- **LMDB、LevelDB = B+ Tree / LSM-Tree 变体**

Hash 索引只在少数场景使用，例如：
- Redis 内存哈希
- PostgreSQL Hash Index（不常用）
- 内存 KV 引擎



