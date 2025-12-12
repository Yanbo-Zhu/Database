



# 1 Quiz


## 1.1 

Consider relation students(studentID, degree, courseID) with bitmap indexes on degree and courseID.

Bitmap index on degree:

ISM: 010101
CS: 101010


Bitmap index on courseID:

DBT: 101000
DBTLAB: 010010
ROC: 000100
DW: 000001

Which table matches these bitmap indexes?

Each bitmap is 6 bits long, so the table has 6 rows (students).

Bitmap index on degree
```
ISM: 010101
CS : 101010

```


| Row | ISM | CS | Degree |
| --- | --- | -- | ------ |
| 1   | 0   | 1  | CS     |
| 2   | 1   | 0  | ISM    |
| 3   | 0   | 1  | CS     |
| 4   | 1   | 0  | ISM    |
| 5   | 0   | 1  | CS     |
| 6   | 1   | 0  | ISM    |

So degree alternates CS, ISM, CS, ISM, CS, ISM.

---

Bitmap index on courseID

```
DBT    : 101000
DBTLAB : 010010
ROC    : 000100
DW     : 000001

```


逐行解释：

第 1 行：DBT=1 → DBT

第 2 行：DBTLAB=1 → DBTLAB

第 3 行：DBT=1 → DBT

第 4 行：ROC=1 → ROC

第 5 行：DBTLAB=1 → DBTLAB

第 6 行：DW=1 → DW

---

最终得到学生表（students）

1   CS     DBT
2   ISM    DBTLAB
3   CS     DBT
4   ISM    ROC
5   CS     DBTLAB
6   ISM    DW



## 1.2 ##

Consider the following extensible hash table:

Block capacity: 4 records
Current state:   i= 1
![](image/Pasted%20image%2020251212213821.png)

How does the hash table look after inserting key 0100?

当全局深度 i = 1 时，目录只看 哈希值的最前1 位。


因为当我们分裂桶 A 时，它的 local depth 从 1 → 2。

这代表这个桶不再只看 1 位，
而是要看 2 位 来决定分裂。

00
01
10
11

---

重新分配桶 A 中的原记录 + 新插入记录 0100

⚠ 注意：分裂只影响原桶的"位组"
原桶 A 是目录项 "0" → 目录 i=1 时，"0" 覆盖所有 00 与 01 位置。

因此分裂后两个桶应该是：

桶 A（对应 00）

桶 A'（对应 01）

所以重新分配规则：

取 尾 2 位：

尾 2 位 = 00 → 去桶 00

尾 2 位 = 01 → 去桶 01


| key       | 头2 位 | 去向      |
| --------- | ---- | ------- |
| 0010      | 00   | → 进入 00 |
| 0101      | 01   | → 01    |
| 0011      | ==   | → 进入 00 |
| 0110      | 01   | → 01    |
| 0100（新插入） | 00   | → 00    |

---

0011  ， 需要按原前缀 0 分 → 入 00  为什么 

桶分裂后，记录只能在原桶所属的目录范围内重新分组，而不能跑到其他目录。

也就是说：

原来桶 A 属于目录项 0（i=1），
扩展到 i=2 后，它只能变成：00 和 01

而目录项 10、11 属于另一个桶（桶 B），
是不能放原桶 A 的记录的。


| record | 最后 1 位                                                      |
| ------ | ----------------------------------------------------------- |
| 0010   | 0                                                           |
| 0011   | 1 ← 但题目给的桶中包含它，说明 hash 用的是最后 1 bit 之外的规则（如取低 k bits 的 hash） |
| 0110   | 0                                                           |
| 0101   | 1 ← 同上                                                      |

重点是：
无论如何，这些数据都属于目录项 0（桶 A）。

---


当桶 A 分裂（local depth变成 2）

目录扩展：

0 → 00 和 01
1 → 10 和 11


于是：

桶 A 原来属于目录项 0

扩展后，它能分给 00 和 01

不能分到 10 或 11（这是目录项 1 的范围）



## 1.3 

When splitting a block in an extensible hash table and distributing keys into new blocks, one of the new blocks can be empty.  

Falsch


在可扩展哈希中：
只有当一个块已经满了（装不下新的 key）时，才会触发分裂（split）。
这个满的块至少已经装有 1 条记录（实际上通常接近容量上限）。
分裂时，块的局部深度（local depth）增加 1，然后根据"多看的一个哈希位"把记录重新分配到两个新块。




重点来了：

❗ 因为原块中是满的，所以里面至少有一些记录
在重新分配时：
不可能两个新块都为空
至少有一个新块中必须有一些记录
另一个新块可能记录很少（甚至只有 1 条）

⚠️ 但在理论和考试标准中，新产生的块"完全为空"被视为不可能




因为：

原块中 所有记录至少会进入两个新块之一。
不可能把所有记录都分给一个块、另一个块完全没用。

（除非哈希函数非常异常，但可扩展哈希算法不允许这种情况被视为"正确分裂"）



## 1.4 ##



在 grid file 中，split 一个 partition = 把原来的一个分区拆成两个分区。

因此 Page 分区数：


120
4×3×10=120



## 1.5 ##

1 A linear hash table searches the bucket array using the least significant bits.	
 ture  就是用的最低位 
线性哈希是一种 动态哈希技术，用于数据库中的索引结构。其设计目标是：

随着数据增加自动扩容
不需要像可扩展哈希那样维护一个目录（directory）
保持平均 O(1) 的查询时间
尽量减少溢出桶（overflow bucket）



2 An extensible hash table is optimized for an even distribution of data across blocks.	
可扩展哈希（Extensible Hashing）被设计用来确保数据在所有桶之间均匀分布。

👉 False（错误）

解释：
Extensible Hashing 的主要目标是：

避免 overflow chain（溢出链）

可以根据需要动态扩展目录

使查找保持稳定成本

它 依赖于良好的哈希函数 才能分布均匀，但 自身不是为了优化均匀分布而设计的，只是为"扩展"设计的。



3 An extensible hash table has a constant search cost.	
虽然 Extensible Hashing 尽量 保持 O(1) 查找，但：

目录可能变大

内存可能不命中

需要访问目录 + 桶

分裂前可能有溢出块（overflow block）

所以查找成本 不是严格常数，只是 平均接近常数。





4 A linear hash table searches the bucket array using the most significant bits.
Linear Hashing 只使用最低有效位 LSB，随着扩展增长 k，使用最后 k bits：

解释：
Linear Hashing 只使用最低有效位 LSB，随着扩展增长 k，使用最后 k bits：

如：
h(key) = 10101101
若 k = 3 → 使用 10101101 的最低 3 bit：101

绝不会用最高位。



## 1.6 ##


When adding a new block to extend a linear hash table, we always have to rehash one existing bucket.
wahr

1 
When adding a new block to extend a linear hash table, we always have to rehash one existing bucket.

在线性哈希（Linear Hashing）中，当需要扩展哈希表时，做的事情是：

按顺序分裂（rehash）现有桶中的一个 bucket，并将新桶加入表中。

也就是说：

线性哈希不会一次性重新分配所有桶（像可扩展哈希那样）。

它采用 渐进扩展（incremental growth） 的方式。

每次扩展哈希表时，都会 rehash（重哈希）一个现有的桶。


2 

🔍 过程回顾：线性哈希扩展的步骤

假设当前 level 为 i，有 2^i 个桶。
当桶满了，需要扩展时：

增加一个新桶（bucket k）

取出 某个旧桶（按顺序，如 bucket split_pointer 指向的桶）

对该桶中的所有记录重新 hash（rehash）
根据新的有效位（i+1 位）分到：

原来的桶

新增的桶

split_pointer++

只有当所有 2^i 个桶被分裂后，才会进入下一 level。


