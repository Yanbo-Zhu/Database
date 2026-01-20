
# 1 Data Model 

Relational Data
Graph Data
Spatial Data
Temporal Data


# 2 OLTP and OLAP 

# 3 Accessing Stored Data


A Disk only has one dimension 

either, the block size was very large  
or, we did not care about the block size


# 4 Indexing and Partitioning

Strategies for arranging multi-dimensional Data on Disk


data can be accessed efficiently  
data is partitioned into several subregions

![](image/Pasted%20image%2020260119221947.png)


# 5 Object Storage

Nowadays, when storing large amounts of data, we use

Object Storage  
Object storage has higher latencies

data is accessed over the network

→ Disk Latency is less important  

We should not store all data in a single Bucket Therefore, we need to partition the data
But: We do not have a hard block size anymore


# 6 Dimensional Data

![](image/Pasted%20image%2020260119222054.png)

- Point Queries
- Orthogonal Range / Box / Window Queries
- Distance Queries
- K-Nearest-Neighbour Queries
- Join Queries
![](image/Pasted%20image%2020260119222614.png)

![](image/Pasted%20image%2020260119222620.png)


# 7 Queries in Data Warehouses

The same problem also exists with other rather independent variables, like those in a Data Warehouse

![](image/Pasted%20image%2020260119222806.png)


![](image/Pasted%20image%2020260119222815.png)


# 8 Using Multidimensional Indices

Group Task

With your neighbour(s): What possible ways can you imagine giving an order to a two- dimensional space. Which trade-offs are important for answering this question?

Key Considerations  
Are my dimensions qualitative or quantitative?  
Do I partition by data or by space?  
Do my queries query correlated datasets?  
Do my queries have a special structure  
how should my represented area be shaped? Are separate-connected areas fine?

![](image/Pasted%20image%2020260119222922.png)




# 9 Hash-based: Grid File Index

Overlay multidimensional space with a grid
• Partition every dimension into stripes
• Number of stripes can be different for each dimension
• Width of stripes can be different in each dimension
• CustomerDB of a store
• Customer(Age,Salary)


![](image/Pasted%20image%2020260119223032.png)

![](image/Pasted%20image%2020260120145052.png)

## 9.1 Search 


![](image/Pasted%20image%2020260120145301.png)


![](image/Pasted%20image%2020260120145344.png)


![](image/Pasted%20image%2020260120145356.png)

## 9.2 Insert

![](image/Pasted%20image%2020260120145436.png)



Search corresponding bucket If space: insert
If no space:
• Alternative 1: 
- Overflow block
• Sequences of overflow blocks should not get too long! • Alternative 2: Change grid (similar to dynamic Hashing)
• Add grid lines • Move grid lines • Problems:
Insert
• Moving or adding a line has impact on all buckets along the line • Optimal choice not always possible
• May create many empty buckets • Buckets may remain full


---

![](image/Pasted%20image%2020260120145510.png)


## 9.3 Efficiency

High Dimensionality: Exponential number of buckets – many empty 
• Already can happen with 2 dimensions. When?
    • If age and salary are correlated

Choice of grid lines such that 
• Bucket matrix fits into main memory
• Index for every dimension fits into main memory 
• Not too many overflow blocks


Point query (age and salary specified to a point) 
• 1 I/O to read bucket
• Insert/delete: 1 additional I/O



Partial match query (only age OR salary specified)
• I/O: All buckets of a row or columns of the bucket matrix

Range queries: specify multidimensional range • All buckets overlapping the range matter!
• 35≤age≤45
• 50 ≤ salary ≤ 100
• If a lot of buckets
    • Only few at the boundary
-Number of I/Os

![](image/Pasted%20image%2020260120145817.png)


Nearest Neighbor Query (Given point P) 
• Start with bucket for P
• If d(P, P') > bucket boundary
    • Search requires neighboring buckets

Example: (50, 200)
• Candidate (50,120) - same bucket - has distance 80
• Lower boundary: No problem
• Upper, left, and right boundary are closer than candidate

![](image/Pasted%20image%2020260120145826.png)





# 10 Hash-Based: Partitioned Hashing Index

![](image/Pasted%20image%2020260120150210.png)

![](image/Pasted%20image%2020260120150224.png)




![](image/Pasted%20image%2020260119223107.png)


![](image/Pasted%20image%2020260119223116.png)



• Nearest-Neighbor-Query?
    • No concept of spatial proximity (aka "clustering")
    • Solution: Suitable Hash function to map small values to small hash values —> Grid Files!!!

• Distribution better than Grid Files
    • Correlation among dimensions irrelevant
    • Fewer overflow blocks



# 11 Tree-based:  Multiple Keys Index 

![](image/Pasted%20image%2020260120150529.png)

![](image/Pasted%20image%2020260120150540.png)




![](image/Pasted%20image%2020260119223250.png)

![](image/Pasted%20image%2020260120150630.png)


![](image/Pasted%20image%2020260120150641.png)

# 12 Tree-based:  K diemensional-Tree


![](image/Pasted%20image%2020260119223326.png)

• Generalization of binary search trees for more (namely k) dimensions 
• Inner nodes
    • Attribute and separator that partitions values for the attribute 
        • Left: smaller values
        • Right: larger values
• Attributes alternate through levels (every level is different attribute) • Two pointers to children (left and right child)

• Leaf nodes
    • Disk blocks with place for (many) records (as many as the block can hold)

![](image/Pasted%20image%2020260120150806.png)

## 12.1 Operations

Search like in binary search trees
• Insert
    • Search corresponding leaf
    • If space: Insert
    • If no space: Split

Example
• Insert (35,500)
• Split necessary

![](image/Pasted%20image%2020260120150915.png)

## 12.2 Query 

![](image/Pasted%20image%2020260120151116.png)


![](image/Pasted%20image%2020260120151128.png)


## 12.3 Kd_trees On Disk

• Problem 1: Path length to leaf much longer than in B-trees • Binary vs. n-ary
• Problem 2: Inner nodes of kd-tree are tiny • Only one (attribute, value) pair
• Idea 1: n-ary inner nodes (multi-branches in inner nodes)  • Each inner nodes stores n values and partitions value range into n+1 ranges
• Idea 2: Group several inner nodes in one disk block • One node and all descendants up to a certain level


![](image/Pasted%20image%2020260120151336.png)

# 13 Tree-based:  Quad-Tree

![](image/Pasted%20image%2020260119223356.png)


![](image/Pasted%20image%2020260120151352.png)

![](image/Pasted%20image%2020260120151537.png)


![](image/Pasted%20image%2020260120151923.png)



# 14 Tree-based:  R-Tree

Analogues to B-Trees
• B-Tree partitions a line into 1d intervals
• Partitioning simplifies search: Only one child must be searched.

R-tree separates 2- or multidimensional spaces into regions
• Can be any shape, mostly rectangles (multidimensional intervals) • Hierarchy of regions and subregions
• Subregions do not cover complete space • Subregions may overlap
• Goal: Overlap should be minimal.

![](image/Pasted%20image%2020260120152240.png)

![](image/Pasted%20image%2020260120152248.png)

![](image/Pasted%20image%2020260120152257.png)


![](image/Pasted%20image%2020260120152305.png)

![](image/Pasted%20image%2020260120152315.png)


## 14.1 R-Tree Properties

• Extension of B-tree to multidimensional space.
• Can support both point data and data with spatial extent (e.g., rectangles)
• Group objects into possibly overlapping clusters (rectangles in the example)
• Search of a range query proceeds along all paths that overlap with the query.
• Parents store the min and max values from all child nodes

## 14.2 Operations
## 14.3 Typical query: Where am I? (point query)
• Start with root (= whole region)
    • For every subregion check if point is contained in region.
    • If not: done
    • If contained in some regions: Recursively descend through all overlapping subregions to leaves 
• In leaves: record of region or pointer to records


![](image/Pasted%20image%2020260120152638.png)

## 14.4 Insertion
• Find region with space  — In general, more than one exists
• Possibly extend a subregion — Keep expansion as small as possible 
• Recursively to the leaves
    • Possibly split a leaf
    • Keep new regions as small as possible 
    • However, regions must cover all records
• New leaves must be represented in parent nodes

![](image/Pasted%20image%2020260119223416.png)

![](image/Pasted%20image%2020260120152809.png)

![](image/Pasted%20image%2020260120152817.png)


![](image/Pasted%20image%2020260120152944.png)


## 14.5 R-Trees — Split Node

MBR is an n-dimensional Minimal Bounding Rectangle used in R trees • It is the minimal bounding n-dimensional rectangle that bounds its corresponding objects
MBR face property: Every face of any MBR contains at least one point of some object in the DB

## 14.6 Nearest Neighbour Search

Retrieve the nearest neighbour of query point Q 

Simple Strategy:
• Convert the nearest neighbour search to range search
• Guess a range around Q that contains at least one object say O
    • if the current guess does not include any answer, increase range size until an object found
• Compute distance d' between Q and O
• Re-execute the range query with the distance d' around Q
• Compute distance of Q from each retrieved object. The object at minimum distance is the nearest neighbor
• Issues: how to guess range, the retrieval may be sub-optimal if incorrect range guessed. Becomes a problem in high dimensional spaces.

![](image/Pasted%20image%2020260120153623.png)

## 14.7 R-Tree Variations

Guttman's R-trees sparked much follow-up work
• Can we do better splits? R*-Tree
• What about static datasets (no ins/del/upd)? Hilbert-Packing
• What about other bounding shapes? Cell-Tree, P-Tree, hB-Tree, TV-Tree, SR-Tree, 


# 15 R*-Tree

R* tree combined numerous improvements to Guttman's R-tree


Better splits
• consider both area and perimeter during split
• Higher tree levels split based on area, lower nodes based on perimeter

defer splits?
shrink overflowing MBR, and re-insert those entries • Issues:
• Which ones to re-insert? 

• How many?
• Approx. 30%


![](image/Pasted%20image%2020260120153855.png)

---

## 15.1 Hilbert R-Trees

Can we improve for static datasets
• Q: What is the best way to pack points into rectangles?
A1: plane-sweep, great for queries on 'x' ; terrible for 'y'
A2: plane sweep on the Hilbert-curve


Dynamic ('Hilbert R-tree):
• each point has an 'h' -value (hilbert value) • insertions: like a B-tree on the h-value
• but also store MBR, for searches

![](image/Pasted%20image%2020260120154039.png)

## 15.2 R-Tree Variations for Other Bounding Shapes

What about other bounding shapes 
• A1: arbitrary-orientation lines (cell-tree, [Guenther])
• A2: P-trees (polygon trees) (MB polygon: 0, 90, 45, 135 degree lines) • A3: L-shapes; holes (hB-tree [Lomet])
• A4: TV-trees [Lin+, VLDB Journal 1994]
• A5: SR-trees [Katayama+, SIGMOD97]

# 16 bitmap indices

## 16.1 位图索引的基本概念

In a Bitmap Index, to each value of a Column in a cell, we create a one-dimensional vector specifying whether the value is present in a given row or not
位图索引的核心思想是：为数据表中某个列的每个可能取值创建一个位向量。

假设有一个"性别"列，只有两个可能值：M（男）和 F（女）。
```
行号 | 性别
1   | M
2   | F
3   | M
4   | F
5   | M
```

为"性别"列建立位图索引：
位图 for M: 1 0 1 0 1 （第1、3、5行是男性）
位图 for F: 0 1 0 1 0 （第2、4行是女性）


![](image/Pasted%20image%2020260120153715.png)

## 16.2 Features of Bitmap Indices
Bitmap indices work well for qualitative Data. they do not really work well for quantitative data though
Bitmap indices are hard to maintain
Bitmap indices can be easily compressed, e.g., using run-length encoding: 11100000110 → 31502110 (or a similar )

(1) 适用于定性数据
定性数据：分类/离散值（如性别、国家、状态码）
可能值的数量有限
每个值对应一个位图
例如：性别（2个值）、颜色（红黄蓝绿等）

不适用于定量数据
定量数据：数值型、连续取值（如年龄、价格、温度）
如果年龄从0到100，需要101个位图 → 占用空间大，查询效率低
但可以通过"分桶"将定量数据转为定性数据处理（如年龄分组 0-18, 19-30, ...）


(2) 维护困难
插入/更新/删除数据时，需要修改多个位图（每个值对应一个位图）
若频繁更新，维护开销很大
对于大量数据更新，位图索引可能不适用


(3) 易于压缩
位图中常出现长串的 0 或 1
行程长度编码 (RLE)：把连续的相同数字压缩成 "数字+出现次数"
    示例：11100000110
    RLE 编码后可能为：3个1, 5个0, 2个1, 1个0 → 数字表示为 3 5 2 1 等
其他压缩方式：BBC、WAH、EWAH 等位图压缩算法




## 16.3 Querying Bitmap Indices
Point queries: Count bits, then retrieve correct block
Partial Match Queries: Fetch all blocks where the Bitmap is 1
Range Queries: Apply AND on Bitmaps then retrieve correct block Fetch all blocks where the Bitmap is 1
kNN-Queries: 1 Requires many Bitmaps per attribute. 2 Thus many ops to execute


(1) 点查询 (Point Query)
查询：性别 = 'M'
直接找到"M"对应的位图，统计 1 的个数 → 得到匹配的行数
根据位图找到数据块位置去读取数据


(2) 部分匹配查询 (Partial Match Queries)
在多个列上查询，例如 性别 = 'M' AND 城市 = '北京'
操作：
取"M"位图 → 10101
取"北京"位图 → 11000
按位与 (AND) → 10000
结果表示只有第1行同时满足条件

(3) 范围查询 (Range Queries)
对于定性数据来说，"范围"可能指的是多个值的"或"操作
例如查询 颜色 IN ('红','黄','蓝')
取"红"位图 OR "黄"位图 OR "蓝"位图
得到结果位图
检索对应行

对于数值型数据（已分桶）：
例如年龄分组 0-18, 19-30, 31-50, 50+
查询 年龄 >= 31
取31-50和50+的位图做 OR 操作


(4) k-最近邻查询 (kNN-Queries)
难点：需要为每个属性值都创建位图
例如对于数值属性，如果直接为每个数值建位图，数量会非常多
查询时需要合并多个位图，操作量大

示例（简化）：
查询"找到与某个点最近的k个点"
如果每个坐标值都有一个位图，需要组合这些位图判断距离
实际操作复杂，位图索引并不适合此类查询


## 16.4 优势与劣势总结

优势：
对于低基数（唯一值少）的列，存储和查询效率高
多条件查询时，通过位运算快速合并结果
易于压缩，节省空间

劣势：
不适用于高基数列（如ID、时间戳）
数据更新时维护代价高
不适合数值型范围查询和kNN查询


## 16.5 实际应用场景
数据仓库中的维度表（如产品类别、地区、时间维度等）
OLAP（联机分析处理）系统中的快速聚合查询
商业智能报表中的多条件筛选



