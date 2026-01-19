
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
Separate chaining 
Open addressing

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


![](image/Pasted%20image%2020260119202249.png)

![](image/Pasted%20image%2020260119202257.png)

![](image/Pasted%20image%2020260119202304.png)

![](image/Pasted%20image%2020260119202311.png)

### 2.1.2 Collision Handling: Open Addressing

‣ Method to resolve hash collisions
‣ Searching through alternative locations (sequence) until value is found or unused slot is found (indicating entry does not exists).
‣ Common sequences are:
    ‣ Linear Probing: Intervall between consecutive probes is fixed, e.g., 1
    ‣ Quadratic Probing: Intervall between consecutive probes increases quadratically
    ‣ Double Hashing: Intervall between consecutive probes is determined by another hash function


![](image/Pasted%20image%2020260119202223.png)

## 2.2 Static vs. Dynamic Hash Tables

Repeated insertions fill up hash tables (and deletions may lead to wasted space)


Open addressing: 
- Pre-allocated number of buckets, limited number of possible entries 
- Degrading performance with increasing fill level due to increasing number of required probes

Chaining
- Pre-allocated number of buckets, theoretically unlimited number of possible entries due to chaining (overflow blocks)
- Degrading performance with increasing fill level due to increasing length of linked lists

—> Dynamic resizing necessary or desirable
Simple approach: creating a new hash table and rehashing all entries Possible but computationally expensive

# 3 Secondary-Storage Hash Tables


So far: Hash tables in main memory

DBMS: Bucket array consists of blocks/pages  在这里 block就是等效于page.  一个 block/page 里面可以存若干个 record. 一个bucket 只有一个 block/page
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
• No support for range queries
(neither sequential disk access nor information which keys are stored)
• Number of buckets B is fixed (“static hashing”)
    • Long lists of overflow blocks or limited number of entries
    • Solution: Dynamic hash tables, which adapt their size dynamically



![[Pasted image 20251126104828.png]]



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
• • ... if room is available

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


Disadvantages:
- When i is large doubling bucket array is a lot of work
- Size of every bucket is fixed —> Non-uniform key distribution leads to many pointers pointing to the same blocks  and potentially empty blocks (i.e., a wast of storage)
- As bucket array doubles, it may not fit in memory (causing an additional I/O)


## 4.3 Linear Hash Table

Idea: Number of buckets grows (and shrinks) in a linear fashion  (i.e., one bucket at a time)
- Compared to extensible hashing, no bucket directory
- Overflows are handled by creating a chain of blocks,  but average number of overflow blocks per bucket is much smaller than 1 
- Choose number of buckets n such that the average number of records per bucket is a fixed fraction, e.g., 85%
- Use ceil(log2n) bits to identify a bucket, always use the rightmost (LSB) bits of the hash value

![](image/Pasted%20image%2020260119195753.png)

### 4.3.1 Inserting into Linear Hash Tables

![](image/Pasted%20image%2020260119195939.png)



i
If m < n: Insert record into bucket m
i: number of relevant bits n: number of buckets
r: number of records
1. Compute h(K)
2. Consider rightmost i bits; interpret as integer m
If m ≥ n: Bucket m does not exist yet; Compute bucket ID using i-1 bits 3. If no space for insertion:
•
3. Check r/n, if too high (e.g., ≥ 1.7 if blocks hold up to 2 records,  and an 85% fill level is targeted):
• •
4. If now n > 2i:
• • •
Create overflow block and insert
Create one new bucket n+1
If the ID of new bucket is 1xxx, rehash records of bucket 0xxx into 1xxx and 0xxx (xxx stands for the same bit pattern)
i = i +1
All buckets IDs are extended by one bit and start with 0 No physical change of the hash table

----

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
一种 **多路平衡搜索树**，所有数据都在叶子节点，叶子节点之间通过链表相连。  
常用于：数据库索引（MySQL、PostgreSQL）、文件系统（EXT4、NTFS）
• • •
Better worst-case complexity of operations (search, insert, delete)
Balance and space-efficiency guarantees
Support for range queries



**Hash Table**
一种通过 **哈希函数**把 key 映射到槽位(bucket)** 的数据结构。  
常用于：编程语言的 key-value 存储（Python dict、Java HashMap）
Faster look-ups, particularly for accessing single records

B+-Trees are more versatile and usually the default index structure for secondary indexes


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



