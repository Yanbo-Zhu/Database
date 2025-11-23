
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

Selectivity 
Selected Tuples/ Total Number Of Tuples
![[Pasted image 20251122163905.png]]

In a highly selective query, only a few elements qualify under the selection; most elements are discarded. -》 wahr 
(Note: Selectivity refers to the property of the index or query.) 
- **Only a small fraction of tuples satisfy the condition**
- **Most tuples are filtered out / discarded**

Selectiveness is used in natural language with the following meaning:
**更日常、更偏向人的行为。**   表示一个人或行为 **很挑、很有选择性、很偏好**
• An index or query is highly selective when it selects only a few values.
• An index or query is lowly selective when it selects a large number of values.

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


## 5.1 Exemplary Insert (Dense Index)

![[Pasted image 20251122171323.png]]


## 5.2 Exemplary Delete (Sparse Index)

![[Pasted image 20251122171358.png]]



