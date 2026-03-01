
# 1 Basics

What kinds of multidimensional indices exist? Into which categories can they be grouped?

Compared to single dimension indices, which properties are given up by multidimensional indices to allow for multidimensional data?


# 2 Our cute Data Warehouse

Given is the following fact table of a data warehouse:

```
import matplotlib.pyplot as plt
import pandas as pd
from IPython.display import Markdown

# Read the CSV file
data = pd.read_csv("resources/07_multidimidx/dwh.csv")
```


```
data.rename(
    columns={
        "productID": "<ins>productID</ins>",
        "storeID": "<ins>storeID</ins>",
        "timestamp": "<ins>timestamp</ins>",
    }
).to_markdown()
```

```
'|    |   <ins>storeID</ins> | <ins>productID</ins>   |   <ins>timestamp</ins> |   Price |   Amount |\n|---:|---------------------:|:-----------------------|-----------------------:|--------:|---------:|\n|  0 |                  101 | K                      |                   0.27 |   12.99 |        5 |\n|  1 |                  112 | P                      |                   8.53 |    7.49 |        3 |\n|  2 |                  125 | T                      |                   5.18 |   19.99 |        2 |\n|  3 |                  138 | X                      |                   2.67 |    5.49 |       10 |\n|  4 |                  150 | Z                      |                   9.76 |   13.49 |        6 |\n|  5 |                  104 | K                      |                   1.02 |    7.29 |        4 |\n|  6 |                  117 | P                      |                   6.45 |   19.79 |        1 |\n|  7 |                  130 | T                      |                   3.89 |    5.69 |        8 |\n|  8 |                  143 | X                      |                   7.71 |   13.19 |        7 |\n|  9 |                  107 | Z                      |                   4.33 |    7.39 |        2 |\n| 10 |                  115 | K                      |                   0.59 |   12.59 |        5 |\n| 11 |                  120 | P                      |                   9.12 |    6.99 |        3 |\n| 12 |                  133 | T                      |                   2.38 |   18.99 |        2 |\n| 13 |                  140 | X                      |                   6.97 |    5.29 |       10 |\n| 14 |                  149 | Z                      |                   8.64 |   12.49 |        6 |'
```



```
data.rename(
    columns={
        "productID": "_productID_",
        "storeID": "_storeID_",
        "timestamp": "_timestamp_",
    }
)
```

![](image/Pasted%20image%2020260120154915.png)

# 3 Hash-based Indices

```python
# Plot the variables
plt.figure(figsize=(10, 6))
plt.scatter(data["productID"], data["storeID"], alpha=0.7, edgecolor="k")
plt.xlabel("productID")
plt.ylabel("storeID")
plt.grid(True)
plt.show()
```


![](image/Pasted%20image%2020260120154937.png)

## 3.1 Grid Files
Based on the Fact table given above, build a grid file on storeID and productID with at most 3 entries per bucket.
![](image/Pasted%20image%2020260120160219.png)


Add the point (T, 105). How does the grid change?
![](image/Pasted%20image%2020260120160230.png)



# 4 Tree-Based indices

```
# Plot the variables
plt.figure(figsize=(10, 6))
plt.scatter(data["timestamp"], data["storeID"], alpha=0.7, edgecolor="k")
plt.xlabel("timestamp")
plt.ylabel("storeID")
plt.grid(True)
plt.show()
```

![](image/Pasted%20image%2020260120155027.png)

## 4.1 kd-Trees and Quad Trees


For creating a Multidimensional Index between storeID and timestamp, how would a kd-Tree and a Quad Tree look like?
Assumptions:
- The Bucket size is 2
- The kd-Tree always splits on the median value. It splits such that less than goes to the left, and greater or equals goes to the right.
- The kd-tree first splits on timestamp.

![](image/Pasted%20image%2020260120160427.png)

```python
# solution

# first level

kd_tl = data[data["timestamp"] < data["timestamp"].median()]
kd_tr = data[data["timestamp"] >= data["timestamp"].median()]

print(f"median of timestamp for data is {data['timestamp'].median()}")
print(f"left leaf has size {kd_tl.shape[0]}")
print(f"right leaf has size {kd_tr.shape[0]}")

# second level

kd_tl_sl = kd_tl[kd_tl["storeID"] < kd_tl["storeID"].median()]
kd_tl_sr = kd_tl[kd_tl["storeID"] >= kd_tl["storeID"].median()]

print(f"median of storeID for left leaf is {kd_tl['storeID'].median()}")
print(f"left left leaf has size {kd_tl_sl.shape[0]}")
print(f"left right leaf has size {kd_tl_sr.shape[0]}")

kd_tr_sl = kd_tr[kd_tr["storeID"] < kd_tr["storeID"].median()]
kd_tr_sr = kd_tr[kd_tr["storeID"] >= kd_tr["storeID"].median()]

print(f"median of storeID for right leaf is {kd_tr['storeID'].median()}")
print(f"right left leaf has size {kd_tr_sl.shape[0]}")
print(f"right right leaf has size {kd_tr_sr.shape[0]}")

# third level

kd_tl_sl_tl = kd_tl_sl[kd_tl_sl["timestamp"] < kd_tl_sl["timestamp"].median()]
kd_tl_sl_tr = kd_tl_sl[kd_tl_sl["timestamp"] >= kd_tl_sl["timestamp"].median()]

print(f"median of timestamp for left left leaf is {kd_tl_sl['timestamp'].median()}")
print(f"left left left leaf has size {kd_tl_sl_tl.shape[0]}")
print(f"left left right leaf has size {kd_tl_sl_tr.shape[0]}")

kd_tl_sr_tl = kd_tl_sr[kd_tl_sr["timestamp"] < kd_tl_sr["timestamp"].median()]
kd_tl_sr_tr = kd_tl_sr[kd_tl_sr["timestamp"] >= kd_tl_sr["timestamp"].median()]

print(f"median of timestamp for left right leaf is {kd_tl_sr['timestamp'].median()}")
print(f"left right left leaf has size {kd_tl_sr_tl.shape[0]}")
print(f"left right right leaf has size {kd_tl_sr_tr.shape[0]}")

kd_tr_sl_tl = kd_tr_sl[kd_tr_sl["timestamp"] < kd_tr_sl["timestamp"].median()]
kd_tr_sl_tr = kd_tr_sl[kd_tr_sl["timestamp"] >= kd_tr_sl["timestamp"].median()]

print(f"median of timestamp for right left leaf is {kd_tr_sl['timestamp'].median()}")
print(f"right left left leaf has size {kd_tr_sl_tl.shape[0]}")
print(f"right left right leaf has size {kd_tr_sl_tr.shape[0]}")

kd_tr_sr_tl = kd_tr_sr[kd_tr_sr["timestamp"] < kd_tr_sr["timestamp"].median()]
kd_tr_sr_tr = kd_tr_sr[kd_tr_sr["timestamp"] >= kd_tr_sr["timestamp"].median()]

print(f"median of timestamp for right right leaf is {kd_tr_sr['timestamp'].median()}")
print(f"right right left leaf has size {kd_tr_sr_tl.shape[0]}")
print(f"right right right leaf has size {kd_tr_sr_tr.shape[0]}")
```


```python
median of timestamp for data is 5.18
left leaf has size 7
right leaf has size 8
median of storeID for left leaf is 115.0
left left leaf has size 3
left right leaf has size 4
median of storeID for right leaf is 132.5
right left leaf has size 4
right right leaf has size 4
median of timestamp for left left leaf is 1.02
left left left leaf has size 1
left left right leaf has size 2
median of timestamp for left right leaf is 2.525
left right left leaf has size 2
left right right leaf has size 2
median of timestamp for right left leaf is 7.49
right left left leaf has size 2
right left right leaf has size 2
median of timestamp for right right leaf is 8.175
right right left leaf has size 2
right right right leaf has size 2
```


---

```python 
# solution

# first level

quad_sw = data[(data["timestamp"] < 5) & (data["storeID"] < 125)]
quad_se = data[(data["timestamp"] >= 5) & (data["storeID"] < 125)]
quad_nw = data[(data["timestamp"] < 5) & (data["storeID"] >= 125)]
quad_ne = data[(data["timestamp"] >= 5) & (data["storeID"] >= 125)]

print(f"splitting on point ({5}, {125})")
print(f"South West quadrant has size {quad_sw.shape[0]}")
print(f"South East quadrant has size {quad_se.shape[0]}")
print(f"North West quadrant has size {quad_nw.shape[0]}")
print(f"North East quadrant has size {quad_ne.shape[0]}")

# second level

quad_sw_sw = quad_sw[
    (quad_sw["timestamp"] < quad_sw["timestamp"].mean())
    & (quad_sw["storeID"] < quad_sw["storeID"].mean())
]
quad_sw_se = quad_sw[
    (quad_sw["timestamp"] >= quad_sw["timestamp"].mean())
    & (quad_sw["storeID"] < quad_sw["storeID"].mean())
]
quad_sw_nw = quad_sw[
    (quad_sw["timestamp"] < quad_sw["timestamp"].mean())
    & (quad_sw["storeID"] >= quad_sw["storeID"].mean())
]
quad_sw_ne = quad_sw[
    (quad_sw["timestamp"] >= quad_sw["timestamp"].mean())
    & (quad_sw["storeID"] >= quad_sw["storeID"].mean())
]

print(
    f"splitting on point ({quad_sw['timestamp'].mean()}, {quad_sw['storeID'].mean()})"
)
print(f"South West South West quadrant has size {quad_sw_sw.shape[0]}")
print(f"South West South East quadrant has size {quad_sw_se.shape[0]}")
print(f"South West North West quadrant has size {quad_sw_nw.shape[0]}")
print(f"South West North East quadrant has size {quad_sw_ne.shape[0]}")

quad_se_sw = quad_se[
    (quad_se["timestamp"] < quad_se["timestamp"].mean())
    & (quad_se["storeID"] < quad_se["storeID"].mean())
]
quad_se_se = quad_se[
    (quad_se["timestamp"] >= quad_se["timestamp"].mean())
    & (quad_se["storeID"] < quad_se["storeID"].mean())
]
quad_se_nw = quad_se[
    (quad_se["timestamp"] < quad_se["timestamp"].mean())
    & (quad_se["storeID"] >= quad_se["storeID"].mean())
]
quad_se_ne = quad_se[
    (quad_se["timestamp"] >= quad_se["timestamp"].mean())
    & (quad_se["storeID"] >= quad_se["storeID"].mean())
]

print(
    f"splitting on point ({quad_se['timestamp'].mean()}, {quad_se['storeID'].mean()})"
)
print(f"South East South West quadrant has size {quad_se_sw.shape[0]}")
print(f"South East South East quadrant has size {quad_se_se.shape[0]}")
print(f"South East North West quadrant has size {quad_se_nw.shape[0]}")
print(f"South East North East quadrant has size {quad_se_ne.shape[0]}")

quad_nw_sw = quad_nw[
    (quad_nw["timestamp"] < quad_nw["timestamp"].mean())
    & (quad_nw["storeID"] < quad_nw["storeID"].mean())
]
quad_nw_se = quad_nw[
    (quad_nw["timestamp"] >= quad_nw["timestamp"].mean())
    & (quad_nw["storeID"] < quad_nw["storeID"].mean())
]
quad_nw_nw = quad_nw[
    (quad_nw["timestamp"] < quad_nw["timestamp"].mean())
    & (quad_nw["storeID"] >= quad_nw["storeID"].mean())
]
quad_nw_ne = quad_nw[
    (quad_nw["timestamp"] >= quad_nw["timestamp"].mean())
    & (quad_nw["storeID"] >= quad_nw["storeID"].mean())
]

print(
    f"splitting on point ({quad_nw['timestamp'].mean()}, {quad_nw['storeID'].mean()})"
)
print(f"North West South West quadrant has size {quad_nw_sw.shape[0]}")
print(f"North West South East quadrant has size {quad_nw_se.shape[0]}")
print(f"North West North West quadrant has size {quad_nw_nw.shape[0]}")
print(f"North West North East quadrant has size {quad_nw_ne.shape[0]}")

quad_ne_sw = quad_ne[
    (quad_ne["timestamp"] < quad_ne["timestamp"].mean())
    & (quad_ne["storeID"] < quad_ne["storeID"].mean())
]
quad_ne_se = quad_ne[
    (quad_ne["timestamp"] >= quad_ne["timestamp"].mean())
    & (quad_ne["storeID"] < quad_ne["storeID"].mean())
]
quad_ne_nw = quad_ne[
    (quad_ne["timestamp"] < quad_ne["timestamp"].mean())
    & (quad_ne["storeID"] >= quad_ne["storeID"].mean())
]
quad_ne_ne = quad_ne[
    (quad_ne["timestamp"] >= quad_ne["timestamp"].mean())
    & (quad_ne["storeID"] >= quad_ne["storeID"].mean())
]

print(
    f"splitting on point ({quad_ne['timestamp'].mean()}, {quad_ne['storeID'].mean()})"
)
print(f"North East South West quadrant has size {quad_ne_sw.shape[0]}")
print(f"North East South East quadrant has size {quad_ne_se.shape[0]}")
print(f"North East North West quadrant has size {quad_ne_nw.shape[0]}")
print(f"North East North East quadrant has size {quad_ne_ne.shape[0]}")

# third level

quad_ne_ne_sw = quad_ne_ne[
    (quad_ne_ne["timestamp"] < quad_ne_ne["timestamp"].mean())
    & (quad_ne_ne["storeID"] < quad_ne_ne["storeID"].mean())
]
quad_ne_ne_se = quad_ne_ne[
    (quad_ne_ne["timestamp"] >= quad_ne_ne["timestamp"].mean())
    & (quad_ne_ne["storeID"] < quad_ne_ne["storeID"].mean())
]
quad_ne_ne_nw = quad_ne_ne[
    (quad_ne_ne["timestamp"] < quad_ne_ne["timestamp"].mean())
    & (quad_ne_ne["storeID"] >= quad_ne_ne["storeID"].mean())
]
quad_ne_ne_ne = quad_ne_ne[
    (quad_ne_ne["timestamp"] >= quad_ne_ne["timestamp"].mean())
    & (quad_ne_ne["storeID"] >= quad_ne_ne["storeID"].mean())
]

print(
    f"splitting on point ({quad_ne_ne['timestamp'].mean()}, {quad_ne_ne['storeID'].mean()})"
)
print(f"North East North East South West quadrant has size {quad_ne_ne_sw.shape[0]}")
print(f"North East North East South East quadrant has size {quad_ne_ne_se.shape[0]}")
print(f"North East North East North West quadrant has size {quad_ne_ne_nw.shape[0]}")
print(f"North East North East North East quadrant has size {quad_ne_ne_ne.shape[0]}")
```


```
splitting on point (5, 125)
South West quadrant has size 4
South East quadrant has size 3
North West quadrant has size 3
North East quadrant has size 5
splitting on point (1.5525, 106.75)
South West South West quadrant has size 2
South West South East quadrant has size 0
South West North West quadrant has size 1
South West North East quadrant has size 1
splitting on point (8.033333333333333, 116.33333333333333)
South East South West quadrant has size 0
South East South East quadrant has size 1
South East North West quadrant has size 1
South East North East quadrant has size 1
splitting on point (2.9800000000000004, 133.66666666666666)
North West South West quadrant has size 1
North West South East quadrant has size 1
North West North West quadrant has size 1
North West North East quadrant has size 0
splitting on point (7.651999999999999, 141.4)
North East South West quadrant has size 2
North East South East quadrant has size 0
North East North West quadrant has size 0
North East North East quadrant has size 3
splitting on point (8.703333333333333, 147.33333333333334)
North East North East South West quadrant has size 1
North East North East South East quadrant has size 0
North East North East North West quadrant has size 1
North East North East North East quadrant has size 1
```


## 4.2 Insertion into an R-Tree

Given is the following R-Tree between timestamp and storeID with each leaf containing at most 4 children.

We want to insert a point at (9.3, 143). What steps need to happen in order to insert it correctly?

![](image/Pasted%20image%2020260120155203.png)


![](image/Pasted%20image%2020260120160542.png)

Timestamp: 0 bis 5
StoreID 100 bis 140 

Timpstamp 4 bis 10 
StoreID 105 bis 150 

# 5 I/O Operations on a Bitmap index

For the Fact table given above, assume that
The table consists of 200 Mio tuples.
storeID consists of 1000 different values.
productID consists of 50 different values.
timestamp consists of 150 Mio different values.

## 5.1 

Also, you are given a page size of 4096 byte.
What is the size of the bitmap indices for each of the variables? How many blocks do they cover? (assume that two bitmaps cannot share a block)
Assume now the table is clustered on productID and the index is compressed using the following run length encoding, how big is the bitmap index for productID?

已知页面大小为 4096 字节。
每个变量的位图索引的大小是多少？它们占用多少个块？（假设两个位图不能共享同一个块）
现在假设表已按 productID 聚集，并且索引使用以下游程长度编码进行压缩，那么 productID 的位图索引有多大？


事实表有 200 Mio 行（200,000,000 行）
storeID：1000 个不同值
productID：50 个不同值
timestamp：150 Mio 个不同值（虽然行数只有200M，timestamp可以有重复但实际不同值很多）
页大小：4096 字节 = 32768 比特
两个位图不能共享一个块（即一个位图如果跨块，不能和另一个位图的前半部分放在同一块）

1 原始位图索引大小（未压缩）
每个位图长度 = 行数 = 200 M bit

一个位图占用的比特数：200,000,000 bit
换算成字节：
200,000,000 bit ÷ 8 = 25,000,000 字节
关键点：位图长度等于表的行数，每个位代表一行。



2 storeID
不同值：1000 个
每个位图大小：200 M bit = 25 MB
总大小 = 1000 × 25 MB = 25,000 MB = 25 GB


因为每个不同的 storeID 值都需要一个独立的位图。
如果有 1000 个不同的 storeID 值，就需要：
storeID=1 的位图
storeID=2 的位图
...
storeID=1000 的位图



3 timestamp
不同值：150 M 个
每个位图大小：25 MB
总大小 = 150,000,000 × 25 MB = 3.75 × 10^9 MB = 3.75 EB（艾字节）
（显然这个索引本身比原表大得多，不现实，说明 timestamp 不适合用普通位图索引）


4 
占用的块数
一个块 4096 字节 = 32768 比特。
每个位图长度 200 M bit，所以：

一个位图占的块数 =
200,000,000 bit ÷ 32,768 bit/块 ≈ 6103.5156 块
因为位图不能跨块共享，所以每个位图必须完整占用若干块，最后不足一块的也要单独占一块。

每个块 4096 字节，一个位图大小 = 6104 × 4096 字节 ≈ 25,001,984 字节（与 25,000,000 基本一致，多了一点点尾部填充）

4 
总块数：
storeID：1000 × 6104 = 6,104,000 块
productID：50 × 6104 = 305,200 块
timestamp：150M × 6104 块 → 天文数字，不实际


![](image/Pasted%20image%2020260120155331.png)



## 5.2 Run length encoding 表按 productID 聚类后的压缩

We store values in one list using integers and the corresponding run length (as integers) in a second.

Example:
Input: 1111123333334444
values: [1, 2, 3, 4]
run-lengths: [5, 1, 6, 4]


关键点：表按 productID 聚类，意味着同一个 productID 的所有行在物理存储上是连续的。

1 
假设 productID 值分布均匀：
每个 productID 出现的行数 = 200M ÷ 50 = 4M 行。

连续存储 → 位图中对应一个 productID 的位图会是：
000...111...000... 这样的模式，具体是连续 4M 个 1，其它地方是 0。

但是我们要看所有 50 个 productID 的位图总大小。注意由于表按 productID 排序，对于一个 productID 的位图，它会在某个连续区间全是 1，其它地方全是 0。因此，每个位图可以行程长度编码成很少的几段。

2 
productID 0 的行：第 0 到 3,999,999 行是 1，其余是 0
productID 1 的行：第 4,000,000 到 7,999,999 行是 1，其余是 0
…
productID 49 的行：第 196,000,000 到 199,999,999 行是 1


3 
每个 productID 位图用 RLE 存储（题目说：values 列表 和 run-lengths 列表）

例：productID 0 的位图（假设比特 1 代表该行是这个 productID）：
200M 行中，前 4M 行是 1，后面 196M 行是 0。

所以 RLE 表示：
values = [1, 0]
run-lengths = [4,000,000, 196,000,000]

每个列表两个整数，每个整数占多少字节？看怎么存，一般整数用 4 字节。


4 
每个列表大小：
values 列表：2 个整数 × 4 字节 = 8 字节

run-lengths 列表：2 个整数 × 4 字节 = 8 字节
加起来 16 字节 就可以表示整个位图。

4.1
类似地，productID k 的位图 RLE 表示：
values = [0, 1, 0]
run-lengths = [4M×k, 4M, 200M - 4M×(k+1)]

即对 k=1：前 4M 行是 0，中间 4M 行是 1，后面 196M-4M=192M 行是 0（这里仔细算一下）。

验证 k=1：
run-lengths = [4,000,000, 4,000,000, 192,000,000]
values = [0,1,0]

这个需要 3 个值，所以
values: 3 × 4 = 12 字节
run-lengths: 3 × 4 = 12 字节
共 24 字节。

5 
产品 ID 从 0 到 49，中间的产品 ID（非第一个非最后一个）都是三段。
产品 ID 0：2 段（values 1,0）
产品 ID 49：2 段（values 0,1）
其余 48 个：3 段（values 0,1,0）


5 
总大小计算：
ID 0：16 字节
ID 49：16 字节
中间 48 个：每个 24 字节
总大小 = 16 + 16 + 48×24 = 32 + 1152 = 1184 字节。

是的，因为聚类存储使得位图极度规则，RLE 压缩后每个位图只要几十字节，总共 1KB 多就够了。

6 
最终答案：

原始大小（未压缩）：
storeID: 25 GB，6,104,000 块
productID: 1.25 GB，305,200 块
timestamp: 不可行（太大）

压缩后（聚类 + RLE）productID 位图大小：约 1184 字节（不到一个块）。


![](image/Pasted%20image%2020260120155339.png)




