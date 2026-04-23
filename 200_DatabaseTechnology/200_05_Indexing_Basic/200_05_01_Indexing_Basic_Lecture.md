

![](image/Pasted%20image%2020260301155128.png)

# 1 Index 

• Data structure that finds the record(s) efficiently
• Given a value of one or more fields find records with that value
• The field(s) on whose value the index is based is a search key


![[Pasted image 20251122163539.png]]


---

When to use an Index or Scan 
**更学术、更正式、更常用。**   表示：**系统、机制、工具或过程** 的 **选择性 / 过滤程度**。
- Depends on “how” many records will be in the answer of a query
- Usually this is defined by the term selectivity
- Selectivity and Selectiveness refer to different views of the same thing

1 Selectivity 
Selected Tuples/ Total Number Of Tuples
![[Pasted image 20251122163905.png]]

In a highly selective query, only a few elements qualify under the selection; most elements are discarded. -》 wahr 
(Note: Selectivity refers to the property of the index or query.) 
- **Only a small fraction of tuples satisfy the condition**
- **Most tuples are filtered out / discarded**

• An index or query is highly selective when it selects only a few values.
• An index or query is lowly selective when it selects a large number of values.


2 Selectiveness
Selectiveness is used in natural language with the following meaning:
**更日常、更偏向人的行为。**   表示一个人或行为 **很挑、很有选择性、很偏好**


![[Pasted image 20251122164104.png]]

# 2 Use Cases

![[Pasted image 20251122164134.png]]


# 3 Index Types, Properties, and Goals

Index can be:
• Ordered index: Based on a sorted ordering of values
• Hash index: Based on a distribution of key to hash values, determined by a hash function

Index can be:
• Dense: One index entry per record/tuple
• Sparse: Only some of the entries (i.e., a strict subset) are represented in the index Often one index entry per block (only useful if records are sorted)

Index can be:
- Clustered/Primary: Index determines the order of the records in the files, i.e., order of index entries same as data entries (often on primary key of the relation) Records are sorted by index
	- 索引决定文件中记录的存储顺序，也就是说：**索引条目的顺序与数据条目的顺序相同**。
	- （通常建立在关系的主键上）    
	- 记录按照索引排序。
-  Secondary: Index does not determine the order of records, i.e., order of index entries not the same as data entries (index on any other attributes except the primary key) Sorted attributes with locations of records in the files
	- 索引不决定记录的存储顺序，也就是说：**索引条目的顺序与数据条目的顺序不同**。
	- （建立在除主键外的所有其它属性上）
	- 属性值按索引排序，同时存储文件中记录的位置。


---

Index Properties:
• Supported access types (e.g., point vs. range look-up)
• Access time/complexity
• Insertion time/complexity (also creation, bulk insert, grow and shrink)
• Deletion time/complexity
• Space overhead/complexity

Index Goals: Improve overall performance for a given workload and database 
There are further physical design aspects for performance tuning, e.g., materialised views, partitioning, small materialised aggregates, data encoding/compression


# 4 Indexes on Sequential Files

Sequential file is created by sorting tuples by a key


## 4.1 Dense index: 
One entry per record/tuple (can be used for any search key)

![[Pasted image 20251122165417.png]]

Speedup
• Index file much smaller than data file
• (Can use binary search on both structures)

![[Pasted image 20251122165439.png]]


## 4.2 Sparse Index

Sparse index: 
Only some of the entries (i.e., a strict subset) are represented in the index 
Often one index entry per block (only useful if records are sorted)

![[Pasted image 20251122165522.png]]

Speedup
• Smaller index than dense -> Less I/O during search, but has to scan the entire page with records
• (Can use binary search on both structures)

![[Pasted image 20251122165548.png]]

## 4.3 Multi-Level Index


Multi-level index: Use an index on the index (can be used for any search key)
Second or higher-level indexes must be sparse

![[Pasted image 20251122170357.png]]



Speedup
• Usage of higher-level indexes is more efficient than binary (or interpolation) search
• Useful for large indexes

![[Pasted image 20251122170424.png]]


## 4.4 Dense Index for Non-Unique Search Keys

![[Pasted image 20251122170523.png]]


## 4.5 Clustered/Primary Index

Clustered/Primary index: index determines the order of the records in the files, i.e., order of index entries same as data entries
It can be dense or sparse
Record data can also be stored in the index



![[Pasted image 20251122170549.png]]

## 4.6 Secondary Index


Secondary Index: Index does not determine the order of records,
i.e., order of index entries not the same as data entries 
• It can only be dense

![[Pasted image 20251122170937.png]]


# 5 Data Manipulation

Changes in data file (insertions, updates, deletes) must be reflected in index
• Dense index must always be updated
• Sparse index must be updated when key in first record changes
• Updates to index can maybe be done efficiently in main memory
• Index file blocks can be treated in the same ways as data file blocks, i.e., different approaches (and combinations) are possible
	• Move entries/records
	• Use tombstones, i.e., markers for deleted records. Delete Data is not a real deletion, but invalidate the records and delete the entry index table 
	• Free space to accommodate insertions
	• Overflow pages
	• Additional pages in sequence

**索引维护与数据文件变更的对应关系**
当数据文件发生插入、更新、删除操作时，这些变更必须同步反映在索引中，以确保数据的一致性和查询的准确性。不同类型的索引对数据变更的响应策略有所不同：
1.  **稠密索引必须始终更新**：由于稠密索引为数据文件中的每条记录都建立一个索引项，因此任何记录的插入、删除或关键字的更新，都必须立即且同步地在索引中添加或删除对应的索引项。
2.  **稀疏索引仅在首个记录关键字变更时更新**：稀疏索引只为数据文件的每个数据块（或每页）建立一条索引项，指向该块中关键字最小的记录。因此，只有当某个数据块中的最小关键字记录发生变化（例如被删除或该记录的关键字被更新为更大的值，导致块内最小值改变）时，才需要更新索引项。其他记录的变更不影响稀疏索引的结构。


**索引维护的实现机制**
索引的更新操作本身也需要高效的存储和访问策略。为了在内存中高效执行更新，并将变更持久化到磁盘，索引文件块可以采用与数据文件块相同的管理方式。以下是几种常见的方法及其组合：
*   **移动条目/记录**：在索引块内，为了保持索引项的有序性，插入或删除操作可能需要移动块内的现有条目，为新的条目腾出空间或填补删除后留下的空隙。
*   **使用墓碑标记**：删除操作不必立即物理清除索引项。可以引入"墓碑"标记，即标识记录已删除的特殊标志。对于数据文件，删除数据并非真正的物理擦除，而是将记录标记为无效；对于索引表，则删除对应的索引项，或者也采用标记的方式。
*   **预留空闲空间**：在构建索引块时，预先在每个块内（例如在页的内部）预留一部分空间。这样，后续的插入操作就可以在当前块内完成，而无需立即进行块的分裂，从而优化写入性能。
*   **溢出页**：当向一个已满的索引块插入新条目，且无法通过预留空间解决时，可以使用溢出页。即分配一个新的磁盘页，将部分条目（通常是原块中的一部分和新条目）移动到该溢出页中，并通过指针将原块与溢出页链接起来，以维持逻辑顺序。
*   **序列中的附加页**：对于按序组织的索引结构（如B+树的叶子节点层），当需要插入大量新数据时，可以在节点序列的末尾或适当位置附加新的页面来容纳数据，并通过调整指针来维护整个序列的有序链接。

## 5.1 Exemplary Insert (Dense Index)

![[Pasted image 20251122171323.png]]


## 5.2 Exemplary Delete (Sparse Index)

![[Pasted image 20251122171358.png]]



