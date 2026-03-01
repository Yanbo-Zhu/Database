

# 1 What is an index and why do we want to have it in our DB?

Each index is for one field/attribute 
- To find rows in our database!
- Everything is stored in files on disk, and we need to locate individual rows to update/delete/read them.
- Without it, we would need to do a full table scan for every operation that we do.    
- **Remember: we want to minimize disk I/O!**

```
SELECT *
FROM movies
WHERE studioName = ’Disney’
AND year = 1990 
```

|movieID|title|studioName|year|rating|boxOfficeMillions|
|---|---|---|---|---|---|
|1|The Little Mermaid II|Disney|1990|7.2|210.4|
|2|DuckTales the Movie|Disney|1990|6.9|18.1|
|3|The Rescuers Down Under|Disney|1990|6.8|47.4|
|4|Beauty and the Beast|Disney|1991|8.0|331.9|
|5|Aladdin|Disney|1992|8.1|504.0|
|6|The Lion King|Disney|1994|8.5|763.4|
|7|Pocahontas|Disney|1995|6.7|346.1|
|8|Toy Story|Pixar|1995|8.3|373.6|
|9|Hercules|Disney|1997|7.3|252.7|
|10|Mulan|Disney|1998|7.6|304.3|
|11|Tarzan|Disney|1999|7.3|448.2|
|12|Fantasia 2000|Universal|1999|7.2|90.9|

- Without index: scan 12 rows
- With index on `studio_name`: 10 rows
- With index on `year`: 3 rows



# 2 Index is stored on disk in files / pages like tuples 


in disk 因为不容易丢失 

Size： indexes can reach TBs for large tables
Recovery: would need to scan all data to rebuild index 
Co-location: primary / clustered index stored together with the actual data 



# 3 How does an index locate individual rows 


![[Pasted image 20251122172522.png]]



Split relations logically into pairs of (SearchKey, Value)
Create & maintain an auxiliary data structure that provides quick lookups when given a key 

# 4 What index types do you know?

…based on **key arrangement/lookup** mechanism

|movieID|title|studioName|year|
|---|---|---|---|
|1|The Little Mermaid II|Disney|1990|
|2|The Rescuers Down Under|Disney|1990|
|3|Beauty and the Beast|Disney|1991|
|4|Aladdin|Disney|1992|
|5|The Lion King|Disney|1994|
|6|Pocahontas|Disney|1995|
|7|Hercules|Disney|1997|

**Ordering**: Based on a sorted ordering of values (you can do binary search!).

```
                 (1994)
                /      \
          (1991)        (1997)
         /     \        /    
   (1990)     (1992)  (1995)
```

**Hashing**: Based on a distribution of keys to hash values, determined by a hash function.
`(1990) → [pointers → row 1, row 2]`
…based on **number of entries**

|movieID|title|studioName|year|
|---|---|---|---|
|1|The Little Mermaid II|Disney|1990|
|2|The Rescuers Down Under|Disney|1990|
|3|Beauty and the Beast|Disney|1991|
|4|Aladdin|Disney|1992|
|5|The Lion King|Disney|1994|
|6|Pocahontas|Disney|1995|
|7|Hercules|Disney|1997|

Dense Index (every row)
- [1990] → Row 1 (PageId, SlotId)
    
- [1990] → Row 2
    
- [1991] → Row 3
    
- [1992] → Row 4
    
- [1994] → Row 5
    
- [1995] → Row 6
    
- [1997] → Row 7
    

Sparse Index (per block)
- [1990] → Block1: [1990,1990]
    
- [1991] → Block2: [1991,1992]
    
- [1994] → Block3: [1994,1995]
    
- [1997] → Block4: [1997]



# 5 Type of Index 

不同 catagory 
primary / clustered / non Cluster index 
sparse / dense Index 
Hash Index 

sorted/ordered index
- Based on arrangement / lookup mechanism
	- Ordering: Based on a sorted ordering of values 
	- Hashing based on a distribution of keys to hash values, determined by a hash function
- Based on number of entries
	- Dense Index : for each row , on index 
	- Sparse index: only store the subset of key
		- Block consist of pages.  not restricts to pages
		- binary search in block to identity the location of the data you search for . 
		- In Sparce Index, we need to search in memory
- Based on value arrangement
	- Clustered Index (data sorted by year)
	- accesse the data sequenctielly , because they are save  in order 

# 6 Can a hash index be sparse 

in ordered data

Dense index : 1 2 2 4 5 6 6 
Sparse index:  1     4    6 ,  from index , we know 5 ist bewteen 4, 5

hash index 
0 -> (1, 5)    我们可以直接 定位到 5 
1 -> 
2 -> 

you can not tell the hash function to be ordered, 因为 hash function 计算出来的结果 就是无序的


…based on **value arrangement**

No 
We can not make assumptions for the location of  keys that are not in stored in the index 

原因：
哈希索引是基于 哈希函数 将键映射到桶（buckets）的结构。
- 哈希函数会把每一个 key 映射到一个桶的位置。
- 但是 桶的位置完全取决于哈希函数，不是按值的顺序排列的。
- 因此，我们无法推断一个没有出现在索引中的 key 在文件中的位置。


⭐ 稀疏索引（sparse index）只有在“值是有序的”情况下才有效

例如：
B+ Tree 索引
排序文件的主索引 / 辅助索引

它们都依赖 值的顺序（value arrangement） 来推断范围或位置。

因为 Hash 索引特点是：
- **无序**
- **值之间没有大小关系**
- **桶的位置不可预测**
- **没有办法根据一个值推断另一个值的位置**

因此：
👉 _要想查找一个 key 的记录，必须在哈希表中有它的条目。_  
👉 不能只索引部分 key（稀疏），否则剩下的 key 完全找不到。


# 7 Cluster and uncluster Index 

You can only have one cluster index. 因为 cluster index hast to be ordered and your data can be order by sole attribute 
cluster index 中 必须是有序存入的 ， 比如说  year 这个 attribute，  在建立一个 cluster  index  的时候， 这个 cluster index 中的数据都是 ordered by year 


## 7.1 Clustered Index (data sorted by year)

|year|movieID|title|pointer (physical order)|
|---|---|---|---|
|1990|1|The Little Mermaid II|→ next (1990)|
|1990|2|The Rescuers Down Under|→ next (1991)|
|1991|3|Beauty and the Beast|→ next (1992)|
|1992|4|Aladdin|→ next (1994)|
|1994|5|The Lion King|→ next (1995)|
|1995|6|Pocahontas|→ next (1997)|
|1997|7|Hercules|→ end|

## 7.2 Unclustered (Non-Clustered) Index

|year|row pointers (to heap)|
|---|---|
|1990|→ RowID 1, RowID 2|
|1991|→ RowID 3|
|1992|→ RowID 4|
|1994|→ RowID 5|
|1995|→ RowID 6|
|1997|→ RowID 7|

## 7.3 Heap File (unsorted data file)

| RowID | movieID | title                   | studioName | year |
| ----- | ------- | ----------------------- | ---------- | ---- |
| 1     | 5       | The Lion King           | Disney     | 1994 |
| 2     | 2       | The Rescuers Down Under | Disney     | 1990 |
| 3     | 4       | Aladdin                 | Disney     | 1992 |
| 4     | 7       | Hercules                | Disney     | 1997 |
| 5     | 3       | Beauty and the Beast    | Disney     | 1991 |
| 6     | 6       | Pocahontas              | Disney     | 1995 |
| 7     | 1       | The Little Mermaid II   | Disney     | 1990 |



# 8 Can we have multiple clustered indexes for a single relation 

Solution: 
- Without replicating the relation, no
- We can not order an already sorted relation by a second key without destroying the sorted order on the first key

**Clustered index = 表的物理存储顺序，而表只能有一种物理顺序，所以只能有一个 clustered index。**

**Clustered index 的本质：物理排序**

Clustered index（聚簇索引）**决定了数据表在磁盘上的物理存储顺序**。
- 当你为某个属性（例如 `id`）创建 clustered index 时，数据库会按照 `id` 的顺序把整个表的数据重新排列。
- 即：**表文件本身就是按照该索引排序的。**

👉 既然表在磁盘上只能以 **一种顺序存放**，就意味着它 **只能有一个排序方式**  
也就是说：

> 一个表只能按照一种键排序（物理顺序），因此也只能有一个 clustered index。


1
解释： 除非你复制出多份表（每份各自按不同的键排序），否则一个表无法同时按两个字段排序 → 所以不能有多个 clustered indexes。
即:  一个物理表文件只能有一种物理顺序。


2 
如果你已经按某个属性排序，再按另一个属性排序，就会破坏第一次排序的顺序。
两种顺序无法同时满足。
因此 不可能让同一份物理文件同时按两个字段排序。

# 9 When should we use indices

Solution:
 - For highly selective queries
	 - Highly selective: query only selects a few tuples
	 - 如果一个查询只需要几条数据，那么使用索引可以直接定位数据，避免扫描整张表。
 - Query requires in-order data
	 - 索引天然是 **排序结构（如 B+ 树）**。
	 - 当查询需要 **排序结果** 的时候，索引可以避免昂贵的排序操作。
 - (Unique-) Constraints
	 - 要维护 UNIQUE 或 PRIMARY KEY 约束，数据库需要一种快速检测重复值的方式。 DBMS 自动使用索引来实现 **主键** 和 **唯一性** 检查。数据库会给 `username` 创建一个唯一索引，以便快速判断是否存在重复条目。
 - Indexed Nested Loop Join (索引嵌套循环连接)
	 - 在 join 操作中，如果被连接的表在连接键上有索引，性能可以极大提升。
 - Shortcut for aggregates
	 - 一些聚合函数可以利用索引加速，例如：`MIN`, `MAX`, 因为按排序存储的数据结构（如 B+ 树）的 **最左和最右节点** 可以直接获得最小最大值。


|使用场景|解释|
|---|---|
|**高度选择性查询**|返回的元组很少，用索引很快|
|**需要排序的数据**|索引已经按顺序存储，避免额外排序|
|**唯一性约束**|索引帮助快速检查重复值|
|**带索引的嵌套循环 Join**|索引使 Join 更高效|
|**MIN/MAX 等聚合**|索引能直接定位最小最大值|


## 9.1 **Indexed Nested Loop Join（索引嵌套循环连接）**

在 join 操作中，如果被连接的表在连接键上有索引，性能可以极大提升。
示例：
```
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.id;
```

如果 `customers.id` 上有索引（通常这是主键）：
- 订单表（orders）中的每一行都可以通过索引快速找到对应的 customer    
- 效率比扫描整个 customers 表快得多

索引帮助 JOIN 更快进行，尤其是 **Nested Loop Join**。


# 10 Space of Dense vs Sparse Index 

Given
- Tuples: 150 M,
- RecordSize: 2 KiB, 
- PageSize: 64 KiB
- SearchKeySize: 8 B,
- RecordPointerSize: 12 B,
- Index entries don’t span blocks/pages on disk.

## 10.1 how much space would a dense or sparse index need? 

IndexTupleSize:  SearchKeySize + RecordPointerSIze 
1. Tuples/Page 一页能存多少个 tuples
2.  Pages: 150M tuples 一共需要多少页.   numIndexTuples:   `4.6875*(10^6)`
3. IndexTuples/Page: 一页能够装多少index 

![[Pasted image 20251123153800.png]]

Dense Index :  We assume we only have one index entry for each page 
Sparse Index:  We assume 一页 （用来储存原本数据的） 对应有 一个index 

To calculate the amount of pages to store the index 
Dense Index 
- IndexPages  一共需要多少页 去装index, 当一个tuple of data 就有一个index的时候

Sparse Index 
- IndexSize:  一共需要多少页 去装index ， 这些页数 乘以每一页的大小 



![[Pasted image 20251123153912.png]]

## 10.2 In the worst case, how many pages would we need to access in a binary search if the dense/sparse index were just the attribute values sorted?



![[Pasted image 20251123154007.png]]

It is not end,   because if we find the right page , we also need to search xx in the page inside 


## 10.3 Multilevel Index on Dense Index: How many paged would we need for a second-level index on the dense index

![[Pasted image 20251123154145.png]]


MultiLevel Index:   index of index 

先建立 Dense Index : 这个是 first-level sparse index 
然后再建立这个dense index 的 sparse index ， 这个  sparse index  面向 外部 .  second-level sparse index 


## 10.4 Disk Access Multilevel Index: How many pages on disk do we access to retrieve a tuple via a multilevel index on disk, if the dense or sparse index was just the attribute values sorted? 



# 11 Quiz 2: Indexing

## 11.1 

Given the following properties:
**Data file:** 
* 10 million tuples 
* Each tuple requires 2048 bytes
**Disk:** 
* Block size: 4096 bytes
**Index:** 
* Search key size: 8 bytes 
* Pointer size: 12 bytes 
* Index entries do not span blocks

---
Dense Index :  We assume we only have one index entry for each page 
Sparse Index:  We assume 一页 （用来储存原本数据的） 对用有一个index 


1 How many blocks are required to store this sparse index? (Given the following properties:
**Data file:** 
* 10 million tuples 
* Each tuple requires 2048 bytes
**Disk:** 
* Block size: 4096 bytes
**Index:** 
* Search key size: 8 bytes 
* Pointer size: 12 bytes 
* Index entries do not span blocks

---
Dense Index :  We assume we only have one index entry for each page 
Sparse Index:  We assume 一页 （用来储存原本数据的） 对用有一个index 


1 How many blocks are required to store this sparse index? (First-level sparse index)
(Give your answer as a single integer.)


这里 block就是 page
1. Tuples/block, 1个block能存多少个 tuples     → Each block holds **4096 / 2048 = 2 tuples**
2.  10M tuples 一共需要多少block.： numIndexTuples:   10M/2 = 5M.   
3. A sparse index has **one entry per data block**, so it needs **5,000,000 index entries**.
4. IndexTuples/block: 一block 能够装多少index   4096/20 = 204 
5. 一共需要多少 block: 5M/204 =24510 

---

2 How many blocks are required for storing a second-level sparse index if the first-level index is also sparse?

If the first-level index is also sparse, then the second-level index has 1 entry per block of the first-level index.

Number of first-level index blocks = 24510.  So second-level index has 24510 entries.
Blocks for second-level sparse index: 24510/204 = 121


## 11.2 ##

Given the following properties of the data file, the disk, and index structures.

Data file:
- The data file contains 10 million tuples.
- Each tuple requires 512 bytes on disk.

Disk properties:
- The size of a disk block is 8192 bytes.

Index properties:
- The size of a search key is 4 bytes.
- The size of a pointer to a block is 8 bytes.
- Index entries do not span blocks on disk.

----

1 **How many blocks are required to store this dense index?**
one disk block can contains 8192/512 tuple = 16
one disk block can contains 8192/12 indexTuple = 682 

10M/682  ≈14664



