
# 1 Hash Tables: Basic Principles
## 1.1 Collision Handling

## 1.2 Static vs. Dynamic Hash Tables


# 2 Secondary-Storage Hash Tables


So far: Hash tables in main memory

DBMS: Bucket array consists of blocks/pages  在这里 block就是等效于page.  一个 block/page 里面可以存若干个 record. 一个bucket 只有一个 block/page
• Records hashed to the same value are stored in the same block
• Overflow blocks can be used for collision handling

Bucket array
• Array of Bucket headers for Bucket chained lists


## 2.1 Analysis of “Static” Hash Tables
Ideally: Only one block per bucket
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

# 3 Dynamic Hash Tables




# 4 B+-Tree vs. Hash Table

**B+ Tree**
一种 **多路平衡搜索树**，所有数据都在叶子节点，叶子节点之间通过链表相连。  
常用于：数据库索引（MySQL、PostgreSQL）、文件系统（EXT4、NTFS）

**Hash Table**
一种通过 **哈希函数**把 key 映射到槽位(bucket)** 的数据结构。  
常用于：编程语言的 key-value 存储（Python dict、Java HashMap）


## 4.1 性能比较 

|操作|B+ Tree|Hash Table|
|---|---|---|
|查找|O(log N)|O(1) 平均|
|插入|O(log N)|O(1) 平均|
|删除|O(log N)|O(1) 平均|
|范围查询|**O(log N + K)**|❌ 不支持（必须遍历整个表）|
|顺序遍历|**有序（叶子链表）**|❌ 无序|

## 4.2 B+ Tree 优缺点

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

## 4.3 Hash Table 优缺点

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


## 4.4 为什么数据库更喜欢 B+ Tree，而不是 Hash 表 

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



