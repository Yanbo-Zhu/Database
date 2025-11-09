
# 1 Recap

1 What is the primary storage location for the DBMSs data, and why?
On Disk, because customers do not like to lose data  which can happen if it is only stored in DRAM

2 What is the primary optimization goal in DBMS implementation?
![[Pasted image 20251109105748.png]]


1. How does a DBMS actually organize its data on disk?  In regular files that  are written/read with system calls to the OS, 
2. What are the atomic units of data that are read/written? Hardware page (typically 4 KB, 16KB on Mac) granularity 



# 2 Pages and Tuples


## 2.1 Table

What types of data can we distinguish for attributes?

How do traditional DBMSs store these relations?

![[Pasted image 20251108225706.png]]

![[Pasted image 20251109110229.png]]

![[Pasted image 20251109110247.png]]



## 2.2 Why not just store them like this?

`| [Tuple1] [Tuple2] [Tuple3] [Tuple4] … |`



- What happens with variable-sized data?   
- What happens on an update like this?

```
UPDATE articles
SET title = 'Basics of SQL'
WHERE article_id = 103; 
```


![[Pasted image 20251109110342.png]]

# 3 How is a single tuple/row laid out in memory?


![[Pasted image 20251109110322.png]]

![[Pasted image 20251109110331.png]]


# 4 Alignment and Addressing


```
CREATE TABLE Student (
  Birthday DATE.
  ImmatrNumber INT, 
  Name CHAR(30), 
  Lastname CHAR(30)
  );
```

![[Pasted image 20251108225848.png]]

![[Pasted image 20251109110408.png]]


1. What is the size of an individual Record?
2. How many records fit on a Page with 4096 bytes?



3. What is the offest of the 33th tuple (assuming pages are filled in reverse order)
![[Pasted image 20251109110451.png]]



## 4.1 ##

What about variable size data?

```
CREATE TABLE Student (
  Birthday DATE.
  ImmatrNumber INT, 
  Name VARCHAR(200), 
  Lastname VARCHAR(200)
);
```

![[Pasted image 20251108225914.png]]
- What is the fixed size of an individual record?
- What is the average total size of a record, assuming VARCHARs are using half of the memory on average?
- How many average records fit into a page with 4096?



![[Pasted image 20251109110633.png]]


## 4.2 Can we store each attribute at an arbitrary offset

![[Pasted image 20251109110712.png]]


## 4.3 ##

Can we store each attribute at an arbitrary offset?
Looking at the following schema:
![[Pasted image 20251108230021.png]]

Can we align this differently to reduce the tuple size (CPU requires 4-byte alignment)?
![[Pasted image 20251109110833.png]]


How is memory addressed by the CPU - and how do we address a tuple on disk?

![[Pasted image 20251109110843.png]]




# 5 Storage Models

## 5.1 What is the difference between NSM, DSM and PAX storage models?

![[Pasted image 20251109110857.png]]



## 5.2 Calculate the number of pages accessed for both NSM and DSM:

![[Pasted image 20251108230130.png]]

- Each attribute consumes **4 bytes**.    
- Page size is **32 bytes**.

```
INSERT INTO R VALUES(9, 9, 330, '21/02/2022')
```

Given this query, what is the problem with pure DSM, and does PAX help here?
- Need to create 4 new pages, one for each column.
- PAX stores columns adjacent to each other within a page.
- In our example, the disk layout would be` [1 2 2 4 50 30 ‘12/05/2020’ ‘21/04/2021’]`
- With PAX, we would only need to read/write to a single page, where we create 4 new column chunks


![[Pasted image 20251109111033.png]]


![[Pasted image 20251109111132.png]]


# 6 Quiz 1: Storage, Data Layouts, Caching

## 6.1 ##

Consider the following relation R:

|employee_id|payment_id|amount|date|
|---|---|---|---|
|1|2|50|12/05/2020|
|2|4|30|21/04/2021|
|3|3|20|15/07/2020|
|4|6|100|30/06/2020|
|5|4|75|11/02/2020|
|6|5|90|21/03/2020|
|7|7|310|10/01/2021|
|8|7|145|25/10/2020|

Assume that:

- Each field in a record consumes 4 bytes (integers as well as the date).
- Block size as well as page size is 32 bytes.
- No extra storage overhead is required for any storage model (i.e., page and block are same sized and contain purely data.)
- For each query in the questions below, the disk head is above the first block to be read.
- No further information about the schema exists, especially no info on indices and constraints.

**How many blocks are retrieved from disk for the following query?**

```
select * from R
where employee_id = 1
```


- **N-ary storage model (row store):** **4 blocks**
    - Each record = 4 fields × 4 B = **16 B** ⇒ **2 records/block** (block = 32 B).
    - 8 records ⇒ **4 blocks total**.
    - No index / no uniqueness info ⇒ must scan all blocks to ensure all matches to `employee_id = 1` are found ⇒ **4 blocks read**.
- **Decomposition storage model (column store):** **4 blocks*
    - Each column value = 4 B ⇒ **8 values/block** (32 B).
    - With 8 rows, each column fits in **1 block**.
    - Steps: read `employee_id` column block to find positions (1 block), then fetch the matching row’s values from `payment_id`, `amount`, `date` columns (1 block each) ⇒ **1 + 3 = 4 blocks**.


---

**How many blocks are retrieved from disk for the following query?**

```
select employee_id from R
where date between 01/05/2020 and 01/07/2020
```

**N-ary storage model (row store):** **4 blocks**
- Record size = 4 fields × 4 B = **16 B** → **2 records per 32 B block**.
- 8 records → **4 blocks** total.
- No index on `date`, so we must scan all blocks to test the predicate and then project `employee_id` → **4 blocks read**.
**Decomposition storage model (column store):** **2 blocks**
- Each column value = 4 B → **8 values per 32 B block** → each column fits in **1 block**.
- Read `date` column to filter (1 block), then read `employee_id` column to fetch the qualifying rows (1 block) → **1 + 1 = 2 blocks**.


## 6.2 ##

Consider the following record structure:

![[Pasted image 20251109130627.png]]


The fields consume the following space:

|Field|Size (bytes)|
|---|---|
|pointer2schema|1|
|length|1|
|timestamp|1|
|name|28|
|age|2|
|state|6|

**How large is one record if the memory is 4-byte aligned?**

==Note: all fields are aligned separately.==


| Field          | Actual size | Aligned size (next multiple of 4) | Explanation                           |
| -------------- | ----------- | --------------------------------- | ------------------------------------- |
| pointer2schema | 1           | 4                                 | padded to 4                           |
| length         | 1           | 4                                 | padded to 4                           |
| timestamp      | 1           | 4                                 | padded to 4                           |
| name           | 28          | 28                                | already multiple of 4 (7×4)           |
| age            | 2           | 4                                 | padded to 4                           |
| state          | 6           | 8                                 | padded to 8 (next multiple of 4 is 8) |

4+4+4+28+4+8=52 bytes


## 6.3 ##

Assume the following variable length record structure:
![[Pasted image 20251109130905.png]]


A record consists of a **record header and record data**. 
A record header consists of 4 components: A, B, C, and D. 
A record data consists of: Name, Address.

The **record header** requires a fixed number of bytes (A and B in the figure above) for book-keeping tasks. It additionally requires C bytes for a pointer to the (variable-sized) **name** field and D bytes for a pointer to the (variable-sized) **address** field. The sizes required by the **header fields of the record** are as follows:

- A = 8 bytes
- B = 4 bytes
- C = 2 bytes
- D = 2 bytes



Consider the following data table. The numbers in parentheses indicate the number of bytes required to store the value.

|Name|Address|
|---|---|
|John Doe (8)|3734 Capitol Avenue (19)|
|Bruce Wayne (11)|461 Still Street (16)|
|George Cross (12)|1001 Clover Drive (17)|
|Johnny Riley (12)|4919 Modoc Alley (16)|
|Alfred Thaddeus Crane (21)|3895 Owagner Lane (17)|
|Beryl Hutchinson (16)|4558 Twin Willow Lane (21)|

Furthermore, assume that:

- Each variable sized field requires 2 additional bytes (apart from raw storage requirement)
- Each page consists of Page Header and Page Data, i.e multiple records that can be stored on a page.
- Page Header has a fixed size of 12 bytes
- The tuples are stored as row store or NSM
- The records can span multiple blocks, and do not require additional overhead (for example fragment bits etc) apart from the storage requirement for the record
- The records can be placed one after the other without any additional overhead (for example fragment pointers etc.) to maintain as shown below:

此外，假设以下条件成立：
1. 每个可变长度字段（Name 或 Address）**额外需要 2 个字节**的开销（不包括原始数据本身的存储大小）。
2. 每个**页面（page）**由**页面头（Page Header）**和**页面数据（Page Data）**组成，页面数据可以包含多个记录。
3. 页面头的大小是**12 字节**。
4. 元组以**行存储格式（row store 或 NSM, N-ary Storage Model）**存储。
5. 记录可以**跨多个块（blocks）存储**，且不需要额外的开销（例如片段位 fragment bits）。
6. 各个记录可以**顺序排列存放**，且不需要额外的开销（例如记录之间的指针等），如下面所示：


![[Pasted image 20251109130913.png]]

---

**How many pages are required if the page size is 128 bytes?**



Here’s the size of each variable-length record with the given header and per-field overhead:

- Header = A(8) + B(4) + C(2) + D(2) = **16 bytes**
    
- Each variable field (Name, Address) adds **+2 bytes** on top of its raw length.
    

So per record:

1. **John Doe (8)**, **3734 Capitol Avenue (19)**  
    16 + (8+2) + (19+2) = **47 B**
    
2. **Bruce Wayne (11)**, **461 Still Street (16)**  
    16 + (11+2) + (16+2) = **47 B**
    
3. **George Cross (12)**, **1001 Clover Drive (17)**  
    16 + (12+2) + (17+2) = **49 B**
    
4. **Johnny Riley (12)**, **4919 Modoc Alley (16)**  
    16 + (12+2) + (16+2) = **48 B**
    
5. **Alfred Thaddeus Crane (21)**, **3895 Owagner Lane (17)**  
    16 + (21+2) + (17+2) = **58 B**
    
6. **Beryl Hutchinson (16)**, **4558 Twin Willow Lane (21)**  
    16 + (16+2) + (21+2) = **57 B**


|#|Name (字节)|Address (字节)|Header|每个字段+2字节|总大小|
|---|---|---|---|---|---|
|1|8|19|16|(8+2) + (19+2)|**47 B**|
|2|11|16|16|(11+2) + (16+2)|**47 B**|
|3|12|17|16|(12+2) + (17+2)|**49 B**|
|4|12|16|16|(12+2) + (16+2)|**48 B**|
|5|21|17|16|(21+2) + (17+2)|**58 B**|
|6|16|21|16|(16+2) + (21+2)|**57 B**|

|记录|大小（字节）|
|---|---|
|R1|47|
|R2|47|
|R3|49|
|R4|48|
|R5|58|
|R6|57|

**Total record bytes = 47 + 47 + 49 + 48 + 58 + 57 = 306 bytes.**

---

Pages required

Each page has a **12-byte page header**, and records may span pages with **no extra overhead**.  
If page size is **P**, the usable payload per page is **(P − 12)**, so the number of pages is:
每个页面由两部分组成
- 页面头（Page Header）：固定大小 **12 字节**
- 页面数据（Page Data）：存放记录的部分，可跨页存储（**允许跨页，无额外开销**）

因此：  
可用数据空间 = `页面大小 - 12 字节`


pages  =  ⌈306 P−12 ⌉\text{pages} \;=\; \left\lceil \dfrac{306}{\,P - 12\,} \right\rceilpages=⌈P−12306​⌉

![[Pasted image 20251109131428.png]]

Examples:

- **P = 64 B** → payload 52 B → ⌈306 / 52⌉ = **6 pages**
    
- **P = 128 B** → payload 116 B → ⌈306 / 116⌉ = **3 pages**
    
- **P = 256 B** → payload 244 B → ⌈306 / 244⌉ = **2 pages**
    

(You can plug in your actual page size if it’s different.)

|页面大小 (B)|可用空间 (B)|所需页数|
|---|---|---|
|64|52|⌈306 / 52⌉ = **6 页**|
|128|116|⌈306 / 116⌉ = **3 页**|
|256|244|⌈306 / 244⌉ = **2 页**|

