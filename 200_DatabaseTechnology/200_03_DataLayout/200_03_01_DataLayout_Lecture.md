
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

...map block references from the database address space to the virtual memory space

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