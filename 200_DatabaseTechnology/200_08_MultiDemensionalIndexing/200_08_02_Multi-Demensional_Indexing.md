


# 1 Quiz
## 1.1 

Given the following kd-tree, choose the corresponding partitions.



![](image/Pasted%20image%2020251212221441.png)

KD-tree 在不同深度按以下顺序选择维度：

第 0 层：按 Salary 切
第 1 层：按 Age 切
第 2 层：按 Salary 切
第 3 层：按 Age 切

… 交替下去


![](image/Pasted%20image%2020251212221705.png)

![](image/Pasted%20image%2020251212222142.png)




| 叶节点点集                  | Partition（二维范围）                     |
| ---------------------- | ----------------------------------- |
| **(18,26)**            | Salary < 30 AND Age < 50            |
| **(48,28), (88,24)**   | 30 ≤ Salary < 110 AND Age < 40      |
| **(97,42)**            | 30 ≤ Salary < 110 AND 40 ≤ Age < 50 |
| **(64,54)**            | Salary < 110 AND Age ≥ 50           |
| **(122,20)**           | 110 ≤ Salary < 130 AND Age < 35     |
| **(142,27)**           | Salary ≥ 130 AND Age < 35           |
| **(128,54), (152,42)** | 110 ≤ Salary < 160 AND Age ≥ 35     |
| **(210,74), (174,64)** | Salary ≥ 160 AND Age ≥ 35           |

![](image/Pasted%20image%2020251212222346.png)

## 1.2 ##


kd-trees branch on different indexed attributes at the same level.
Falsch


分裂属性在不同层次之间轮流切换，但同一层所有节点使用的属性相同。

例如：

第 0 层（root）：按 Salary 切

第 1 层：按 Age 切

第 2 层：按 Salary 切

第 3 层：按 Age 切

在第 1 层，即使有很多节点，它们都必须以 Age 为分裂属性。

绝不允许第 1 层一部分节点按 Age 切，另一部分按 Salary 切。


## 1.3 ##


Consider relation Customer(Age, PostalCode, City, TotalSpent) with a kd-tree index on the primary key.

The following query:

select avg(TotalSpent) from Customer
where Age between 45 and 60 and
PostalCode between 10629 and 13581


is an example of:
Frage 17Wählen Sie eine Antwort:

GIS Query
Point Query
Partial-Match Query
Range Query  wahr


这是对 两个维度（Age、PostalCode）同时给出的区间条件。

在 KD-tree 中，这属于：

多维范围查询（Range Query）

因为：

Age 给了一个范围

PostalCode 给了一个范围

查询的结果包括满足这些范围的所有点

并不是查某个特定值（Point Query）

也不是只指定部分维度（Partial-Match Query）

更不是 GIS Query（那针对地理坐标）


| 查询类型                    | 含义                                        |
| ----------------------- | ----------------------------------------- |
| **GIS Query**           | 地理空间查询，使用经纬度等，不符合本题                       |
| **Point Query**         | 查询一个完全确定的点，例如 Age=50 AND PostalCode=12000 |
| **Partial-Match Query** | 只给部分维度条件，例如 Age=50（PostalCode 不指定范围）      |
| **Range Query**         | 对一个或多个维度给定范围条件                            |


## 1.4 ##


Consider relation R(a, b, c, d) stored using partitioned hashing:

Total buckets: 65536 (16-bit hash function)
Bit allocation: 4 bits for a, 5 bits for b, 3 bits for c, 4 bits for d
How many buckets must be checked for a partial match query on a?

Example:
SELECT *
FROM R
WHERE a = 'y';



在 partitioned hashing 中：

哈希值分成不同段，每段对应一个属性
查询中 指定的属性对应的 hash bits 固定
未指定的属性 bits 可以是任意值 → 产生多个组合
需要检查的桶 = 未指定 bits 组合数


WHERE a = 'y'
仅指定了属性 a 的 4 bit。

表示：

a 的 4 bit 是固定值

b（5 bits）可以是任意 → 2⁵ 组合

c（3 bits）可以是任意 → 2³ 组合

d（4 bits）可以是任意 → 2⁴ 组合




需要检查的桶数：
2^(5+3+4)=212=4096