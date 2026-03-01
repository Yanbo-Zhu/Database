
# 1 Data Representation:


## 1.1 Records and Data types


How Stoage Devices Store Relations 
![[Pasted image 20251109094132.png]]



## 1.2 Storing Data in DBMS
‣Where to store data => Database File Storage
‣How to store data on a page => Page Layout
‣How to store a record => Record Layout


### 1.2.1 Database Files

‣Usually a database is stored on multiple files
‣DBMS organizes these files into a collection of pages
‣Files are stored on top of a regular file system

![[Pasted image 20251109094608.png]]


### 1.2.2 Pages

‣Pages are fixed-sized blocks of data
‣They have a header and a data portion
‣Header could contain various information (checksum etc.)
‣Data contains the tuples or records
‣Each page is also assigned a unique identifier called PageID


![[Pasted image 20251109094649.png]]


### 1.2.3 Data Types

![[Pasted image 20251109094718.png]]


#### 1.2.3.1 Memory Alignment (bits Padding)


![[Pasted image 20251109094920.png]]


![[Pasted image 20251109095047.png]]




## 1.3 Addressing

![](image/Pasted%20image%2020260301010630.png)



### 1.3.1 Addressing Fixed-Length Records


![[Pasted image 20251109095217.png]]


Linear Addressing: offset = pageSize - tupleSize*(slotID+1)

### 1.3.2 Variable-Length Attributes


![[Pasted image 20251109095409.png]]

![[Pasted image 20251109095444.png]]


## 1.4 Spanned Records

Storing across pages: One records is
• slightly larger than half of a page
• larger than a page


More complex for more than two possible fragments


### 1.4.1 Pointer Swizzling

==Pointer swizzling and unswizzling are used to.map block references from the database address space to the virtual memory space==
就是用 pointer in Memory 取代 id_nummer in disk when load data form disk the momery 




"In computer science, pointer swizzling is the conversion of references based on name or position to direct pointer references. It is typically performed during the deserialization (loading) of a relocatable
object from disk, such as an executable file or pointer-based data structure. The reverse operation, replacing pointers with position-independent symbols or positions, is sometimes referred to as unswizzling, and is performed during serialization (saving)."

![[Pasted image 20251109100742.png]]



Data Storge in Disk: No Pointer, No Jump, id_number of the next node 
Data Storge in main Momery: use Pointer to point nex location in other location 

![[Pasted image 20251109103255.png]]




## 1.5 Updates

Record Insertion
![[Pasted image 20251109103320.png]]


Record Deletion
![[Pasted image 20251109103336.png]]

# 2 Data Layouts:
## 2.1 Row- and Column-Layout

Row stores have high tuple reconstruction costs.  false 
Disk access of analytical queries is reduced when using column stores.  wahr 
Column stores, in general, yield better performance when doing point queries (e.g., reading a single tuple) compared to row stores. false
Vertical partitioning aims at reducing the I/O cost for every single incoming query.  Falsch

  





Linearizing Tuples 
### 2.1.1 N-ary Storage Model (NSM)   Column-Layout

![[Pasted image 20251109103435.png]]



### 2.1.2 Decomposition Storage Model (DSM)  Column-Layout 

![[Pasted image 20251109103449.png]]


### 2.1.3 Comparision

![[Pasted image 20251109104423.png]]


![[Pasted image 20251109103600.png]]

## 2.2 Hybrid Layouts


### 2.2.1 Fractured Mirrors

Which of the following statements is true for "fractured mirroring"?

The data is mirrored using the RAID technique for replicating data for enhanced fault tolerance in case of broken disks.
The data is replicated, twice in row layout (NSM) and twice in column layout (DSM) format, with the goal of handling disk failures。
The data is stored twice, once in row layout (NSM) and once in column layout (DSM) format to adaptively pick the suitable layout depending on the workload.  (wahr)
The data is stored once, but some attributes are stored in row layout (NSM) and some in column layout (DSM).


It can handle both OLAP and OLTP workloads
It requires twice storage space
Maintain both replicas up-to-date

The data is stored twice, once in row layout (NSM) and once in column layout (DSM) format to adaptively pick the suitable layout depending on the workload.

![[Pasted image 20251109103718.png]]


### 2.2.2 Vertical Partitioning
The data is stored once, but some attributes are stored in row layout (NSM) and some in column layout (DSM).

Vertical partitioning aims at reducing the I/O cost for every single incoming query.: Falsch 


Vertical partitioning (vertikale Partitionierung) bedeutet,
dass eine Tabelle in Spalten bzw. Spaltengruppen aufgeteilt wird
— also ähnlich wie in einem Column Store.

➡️ Ziel ist nicht, die I/O-Kosten für jede einzelne Anfrage zu reduzieren,
sondern für bestimmte Arten von Abfragen (typisch: analytische Queries, die nur wenige Spalten benötigen).



![[Pasted image 20251109103956.png]]

### 2.2.3 PAX Layout: Partioning Across Attributes



DSM inside an NSM page
NSM across Pages, DSM within pages

![[Pasted image 20251109104122.png]]

• PAX => Partition Attributes Across
• Try to maximize inter-record spatial locality within each column within each page (same attribute of different tuples are stored next to each other)
• Try to minimize record construction costs (limited within a page not across pages)
• Data remains on its original NSM page. NSM and PAX storage requirements are on par.
• Attribute values are grouped into minipages.

- **PAX ⇒ Partition Attributes Across（跨属性分区）**
    
- 尝试在每一页（page）中**最大化列内不同记录之间的空间局部性**，  
    即：**不同元组中相同属性的值被存储在相邻的位置**。
    
- 尝试**最小化记录构建的开销**（仅在单个页内优化，而非跨页）。
    
- 数据仍保存在其原始的 **NSM（N-ary Storage Model）** 页面中。  
    **NSM** 与 **PAX** 在存储空间需求上是**相当的**。
    
- 各个属性的值被分组存储在称为 **minipage（迷你页）** 的小页中。

![[Pasted image 20251109104317.png]]


![[Pasted image 20251109104337.png]]


## 2.3 OLTP OLAP


![[Pasted image 20251109104500.png]]

![[Pasted image 20251109104518.png]]



**OLAP** 与 **OLTP** 是两种主要的数据库处理类型，它们的核心区别在于**服务对象**和**处理方式**。

简单来说：**OLTP 系统负责处理"现在"的交易，OLAP 系统负责分析"过去"的数据以支撑未来的决策。**

以下是基于你之前关注的数据库、磁盘I/O、存储等计算机基础背景的详细对比：

-   **OLTP （Online Transaction Processing，在线事务处理）**：
    -   **目标**：处理高并发、短小精悍的日常交易。
    -   **例子**：你在淘宝下单、银行转账、刷地铁卡。每一次操作都是一个"事务"。
-   **OLAP （Online Analytical Processing，在线分析处理）**：
    -   **目标**：处理复杂的查询，用于数据分析和决策支持。
    -   **例子**：CEO查看"上个月哪个省份的销售额最高"，或者"去年购买A产品的用户今年还买了什么"。

## 2.4 详细区别对比表

| 特性 | OLTP （联机事务处理） | OLAP （联机分析处理） |
| :--- | :--- | :--- |
| **用户** | 一线操作人员（如客服、收银员、终端用户） | 数据分析师、经理、决策者 |
| **操作** | **读/写**（增删改查，以短事务为主） | **主要是读**（复杂的查询，生成报表） |
| **查询复杂度** | 简单、预定义的查询（如：SELECT * FROM orders WHERE id = 123） | 极其复杂（如多表连接、分组聚合、大量的数据扫描：SELECT SUM（...） FROM ... GROUP BY ...） |
| **响应时间** | 极快（毫秒级），要求实时 | 通常较慢（秒级、分钟级甚至小时级），取决于数据量 |
| **数据量** | 处理当前时刻的一小部分数据（GB级别） | 处理历史海量数据（TB、PB级别） |
| **数据模型** | **规范化**（3NF，即第三范式），消除冗余，保证更新一致性 | **反规范化**（星型模型、雪花模型），存在冗余，方便查询 |
| **索引** | 精准索引（为了快速找到特定那一行） | 大量索引、位图索引（为了快速汇总计算） |
| **I/O 特点** | **随机读写**（频繁修改小数据块，结合你之前的知识点：会产生大量的随机I/O） | **顺序读写**（全表扫描、批量写入，通常是将数据批量加载进去） |

## 2.5 结合你之前的关注点
结合你刚刚复习的磁盘和 RAID 知识，可以这么理解：

-   **OLTP 的 I/O 模式**：
    -   **特点**：**随机访问 + 频繁写入**。
    -   **存储挑战**：会产生大量的小文件随机读写。如果使用机械硬盘，这会非常慢（因为需要频繁寻道）。
    -   **硬件倾向**：为了应对这种"写惩罚"，RAID 10 是 OLTP 的常见选择（因为写入速度快且安全）。同时强烈依赖内存缓存（Buffer Pool）来减少磁盘 I/O。

-   **OLAP 的 I/O 模式**：
    -   **特点**：**顺序读写 + 海量数据扫描**。
    -   **存储挑战**：经常需要读取整个表或索引。即使有索引，也可能因为查询范围太大而退化为全表扫描。
    -   **硬件倾向**：RAID 5 或 RAID 6 比较适合 OLAP，因为它们可以提供较高的吞吐量，而且顺序读写对校验的计算开销相对不那么敏感。

## 2.6 简单类比
-   **OLTP** 就像**超市收银台**：操作快，每次处理一点点东西，不能出错，随时有人排队等着。
-   **OLAP** 就像**超市年度盘点**：动作慢，要把所有库存翻出来算一遍，找出规律（比如哪个卖得好），不需要实时响应，但计算量巨大。
