
# 1 Introduction
## 1.1 Motivation

One or multiple levels of indexes:
- Often helpful in speeding up queries
- But no clear strategy for data modifications yet; common problems:
	- Appropriate number of index levels
	- Adding blocks (e.g., using overflow blocks)
	- Almost empty blocks or moving entries

Idea of a more efficient structure, automatically maintaining
• Number of levels
• Length of individual paths from root to a leaves, i.e., balance
• Fill level of blocks One or multiple levels of indexes:


Multi-level indexes may basically run into similar problems as binary search trees
![[Pasted image 20251123161051.png]]


—> Use balanced tree: A binary tree is balanced if for every interior node, the heights of its two child trees differ by at most 1.


## 1.2 B-Tree

• B-Tree is a multi-level index with a variable number of levels
• Height, i.e., number of nodes from root to leaves, adapts to table size
• Designed for block-wise access
• Every block is between half used and completely full
• Always balanced: All paths from the root to a leaf are of equal length

![[Pasted image 20251123161134.png]]


## 1.3 B-Tree vs. Binary Search Tree

• B-Tree is a multi-level index with a variable number of levels
• Height, i.e., number of nodes from root to leaves, adapts to table size
• Designed for block-wise access
• Every block is between half used and completely full
• Always balanced: All paths from the root to a leaf are of equal length

![[Pasted image 20251123161144.png]]


## 1.4 B+-Tree vs. B-Tree

Difference of B+-tree to a B-tree
• Only the leaves point to the data
• A leaf points to the next leaf in sequence (to the right)

![[Pasted image 20251123161331.png]]


# 2 Block Structure

B-Tree structure with 3 levels (height/depth=3)

d is  height of tree 
Different definition: “B-tree of order d contains at most 2d keys and 2d+1 pointers”

- Tree of blocks with a common structure
- Paths from root to leaf have the same length (balanced)
- No overflow blocks needed
- Blocks are at least 50% full
- As many levels as necessary


Block structure (based on book)
==根据block 的尺寸 算一个block 中最多容纳多少个 key==
• up to n keys and n+1 pointers
• pick n as large as possible with regard to block size
• Example:
• 4096 bytes blocks; 4 bytes keys; 8 bytes pointers; no meta data (e.g., block header)
• 4n+8(n+1)≤4096 —> Choose n=340

![[Pasted image 20251123161511.png]]


![[Pasted image 20251123161639.png]]


![[Pasted image 20251123161650.png]]

![[Pasted image 20251123162136.png]]

K is the height of whole tree 

## 2.1 Leaves

![[Pasted image 20251123161705.png]]

- At least (n+1)/2 取下限 of pointers point to data records,
- At least half of the records pointer (i.e. keys) must be used 

## 2.2 Inner Nodes

At least (n+1)/2 取上限 of pointers point to blocks of lower level,
i.e., half of the pointers must point to blocks of lower levels; exception: if node is root, at least 2 pointers

![[Pasted image 20251123162052.png]]




# 3 B+-Tree — Example

n = 4
• All nodes: At most 4 keys and 5 pointers
• Root: At least 1 key and 2 pointers (exception: it can have less pointers in case of single level trees)
• Inner Nodes: At least 2 key and 3 pointers
• Leaves: At least 2 keys (i.e., 2 pointers to records)

![[Pasted image 20251123162431.png]]


# 4 Search/Insert/Delete

• For now:
• No duplicate keys at the leaves
• Assume a dense index, i.e., every search-key value that appears in the data file also appears at a leaf
—> Simpler operations, but concepts are the same


## 4.1 How to calculate/estimate the height of a tree?

How to calculate/estimate the height of a tree?
• Calculate number of key-pointer pairs per block (Consider fill level, e.g., all blocks are full)
• Use logarithm


Key-pointer pairs for a 4096 byte block
• 4096 bytes blocks;
4 bytes keys;
8 bytes pointers;
no meta data (e.g., block header)
• 4n+8(n+1)≤4096 —> n=340  一个block 一共有多少个 keys


Tree height for 100 * 106 records
• ⌈log340(100 * 106)⌉=4   求上限 



## 4.2 Search

• Find K (recursively)
• If we are at a leaf, search k on the leaf: if the ith key is k, the ith pointer points to the record

• If we are at an inner node with keys k1, k2, …, kn:
	• If k < k1 go to first child
	• If kn ≤ k go to last child
	• If k1 ≤ k < k2 go to second child

![[Pasted image 20251123162600.png]]


![[Pasted image 20251123162746.png]]

![[Pasted image 20251123162739.png]]


![[Pasted image 20251123162717.png]]


![[Pasted image 20251123162729.png]]


B+-Tree — Cost of Searching
Search (including range queries) requires one block access per level to lookup value
Range queries may require accessing multiple leaves (in sequence)
Additional block access(es) to retrieve record(s)


## 4.3 Insert

![[Pasted image 20251123172801.png]]

Consider the B⁺ tree below with the following properties:
- Each node (except the root) contains keys.
- Inner nodes: The keys in the subtree below the pointer are less than the key ; the keys in the subtree below the pointer are greater or equal than the key- .
- Insert operations may only trigger node splits but not shifting of keys into bordering leafs.
- When a node is split, the middle key is moved to the parent node.
- Pointers in leaves point to row IDs. Pointer in leaves points to the next leaf.

每个节点（根节点除外）包含键。
内部节点：指针所指的子树中的键都小于该键；指针所指的子树中的键都大于或等于该键。
插入操作可能只会触发节点分裂，而不会将键移到相邻的叶子中。
当节点分裂时，中间键被移到父节点。
叶子中的指针指向行 ID。叶子中的指针指向下一个叶子。



Search corresponding leaf
- If room, insert key and pointer
- If no room: Split
	- Split leaf in two parts and distribute keys equally
	- Split requires inserting a new key-pointer pair in parent node
- Recursively ascend the tree
- Exception: If no space in root
	- Split root
	- Create new root (with only one key and two children)
	- Height increases by one

查找对应的叶子节点  
- 如果有空间，插入键和指针  
- 如果没有空间：分裂  
  - 将叶子节点分成两部分，并平均分配键  
  - 分裂需要在父节点中插入一个新的键-指针对  
- 递归向上处理树  
- 例外情况：如果根节点没有空间  
  - 分裂根节点  
  - 创建新的根节点（只有一个键和两个孩子）  
  - 树的高度增加 1  

如果键的数量是奇数，则将多的一个放在左边或右边  
对于内部节点分裂：将中间键移到父节点
If the number of keys is uneven, put +1 left or right0
For inner node split: move middle key to the parent

![[Pasted image 20251123163252.png]]

![[Pasted image 20251123163514.png]]


![[Pasted image 20251123163523.png]]


![[Pasted image 20251123163535.png]]


• Space available in leaf (best case): height reads + 1 write (best case)

• Recursive maintenance up to the root (worst case):
	• Operations per level:
		• Read old block (1 I/O)
		• Create a new block (split, write two blocks): 2 I/O
		• Go level up
	• + writing a new root (1 I/O) —> Total cost: height + 2 × height + 1 = 3 × height + 1


## 4.4 Delete

Consider the B⁺ tree below with the following properties:

- Each node (except the root) contains
- keys.
- Inner nodes: The keys in the subtree below the pointerare less than the key ; the keys in the subtree below the pointer are greater or equal than the key- .
- Delete operations will first try to steal nodes from a direct sibling (first right sibling, then left sibling). If this is not possible, they will merge with a sibling (first right sibling, then left sibling).
- Pointersin leaves point to row IDs. Pointer- in leaves points to the next leaf.
- **Please read section 14.2.6 in "Database Systems The Complete Book" second edition to look for the details of the deletion algorithm when one or more of the above conditions are violated post deletion.**

考虑下面的 B⁺ 树，它具有以下性质：
- 每个节点（根节点除外）包含键。
- 内部节点：指针所指的子树中的键都小于该键；指针所指的子树中的键都大于或等于该键。
- 删除操作将首先尝试从直接兄弟节点窃取节点（先右兄弟，后左兄弟）。如果这不可能，它们将与兄弟节点合并（先右兄弟，后左兄弟）。
- 叶子中的指针指向行 ID。叶子中的指针指向下一个叶子。
- **如果在删除后违反上述一个或多个条件，请阅读《Database Systems The Complete Book》第二版第 14.2.6 节以了解删除算法的细节。**

![[Pasted image 20251123172012.png]]


Based on your description, the B⁺ tree deletion process follows a specific priority: first, it attempts to "steal" (redistribute) an entry from a sibling; if that fails, it performs a merge with a sibling. The order for both attempts is the right sibling first, then the left sibling.

----


• Search corresponding leaf

- Delete key
    - If at least minimal number of keys in node: Nothing else to do
    - If too few keys in node:
        - Steal a key from a sibling node (left or right) if possible, i.e., a sibling has more than the minimum number of keys (also redistribution of keys with sibling as optimization possible)
            - adjust keys of parent (and delete one key-pointer pair); recursively apply deletion algorithm if necessary
        - If stealing is not possible:
            - two siblings with minimal and sub-minimal number of keys can be merged
            - adjust keys of parent if required and delete one key-pointer pair. recursively apply deletion algorithm if necessary


- 删除键  
    - 如果节点中的键数量至少为最小值：无需其他操作  
    - 如果节点中的键数量太少：  
        - 如果可能，从兄弟节点（左或右）偷一个键，即兄弟节点的键数大于最小值（也可以与兄弟节点重新分配键，作为优化手段）  
            - 调整父节点中的键（并删除一个键-指针对）；如有必要，递归应用删除算法  
        - 如果偷取不可行：  
            - 两个键数分别为最小值和次最小值的兄弟节点可以合并  
            - 如果需要，调整父节点中的键，并删除一个键-指针对；如有必要，递归应用删除算法  

---  

如果你需要我根据这里的规则画图演示 B⁺ 树删除过程，或者解释某一特定步骤，也可以告诉我。

![[Pasted image 20251123163800.png]]


![[Pasted image 20251123163835.png]]

![[Pasted image 20251123163842.png]]


![[Pasted image 20251123163855.png]]

![[Pasted image 20251123163905.png]]


![[Pasted image 20251123163912.png]]

![[Pasted image 20251123163923.png]]

![[Pasted image 20251123163933.png]]

![[Pasted image 20251123163940.png]]


Deletion Costs
• Enough keys: h reads + 1 write
• When stealing from first siblings: (h+1) reads + 2 writes
• When merging with sibling: (h+2) reads + 3 writes
	• Read left and right sibling
	• Write block and sibling
	• Write parent
• When merging up to the root (worst case):
	• Search: depth of the tree
	• For every level: Must potentially check both siblings before merging


**删除成本**
- 键数量足够：h 次读取 + 1 次写入  
- 从第一个兄弟节点窃取时：(h+1) 次读取 + 2 次写入  
- 与兄弟节点合并时：(h+2) 次读取 + 3 次写入  
    - 读取左兄弟和右兄弟  
    - 写入本块和兄弟块  
    - 写入父节点  
- 向上合并直到根节点（最坏情况）：  
    - 查找：树的深度  
    - 每一层：合并前可能必须检查两个兄弟节点  



# 5 Efficiency

• Search, insert, and delete should require as few I/Os as possible
• The larger n, the fewer block splits or merges are required

• Search:
	• Number of I/Os corresponds to the height of the tree (log(n))
	• + 1 I/O on the data file for search
	• + 2 I/Os on the data file for insert or delete
• Typical height of a B-tree: 3 How many records can we index?
	• 340 key-pointer pairs per block
	• 255 pointers in root => 255 children => 255² = 65025 leaves = 255³ record pointers
	• Assumption: in average filled with 255 pointers
	• Overall 16.6 million records

Recap
4096 bytes blocks; 4 bytes keys; 8 bytes pointers; no meta data (e.g., block header)
• 4n+8(n+1)≤4096 —> Choose n=340   . 一个block 一共有多少个 keys



# 6 Bulk Loading  (构造 B+ Tree )
Insertion variations (LOAD):
• Inserting larger data sets into an existing B-tree
• Creating a new B-tree on existing data
• Iterative insertion of every record is inefficient
• Always need to search through the root


• Better for large data sets: Sort the data and exploit the order
• Step 1: Create key-pointer pairs for all blocks
• Step 2: Sort pairs on key
• Step 3: Insert pairs iteratively


Bulk Loading with Filling Degree
• Step 1: Sort pairs on sort key
• Step 2: Create key-pointer pairs for all blocks
• Step 3: Create filled/full B-tree leaves
• Step 4: Construct inner nodes based on leaves
• Result: Perfect filing degree


Why is this efficient?
• ==Only access index blocks in cache==
• Cost: Write each block once


插入操作的变体（加载）：
- 将较大的数据集插入到现有的 B 树中
- 基于现有数据创建新的 B 树
- 逐条插入每条记录效率低下
- 每次都需要从根节点开始搜索


- 对于大数据集更优的方法：对数据进行排序并利用其顺序
- 步骤 1：为所有数据块创建键-指针对
- 步骤 2：按键对键-指针对进行排序
- 步骤 3：迭代插入这些键-指针对


具有填充度的批量加载
- 步骤 1：按排序键对键-指针对进行排序
- 步骤 2：为所有数据块创建键-指针对
- 步骤 3：创建填满/填充好的 B 树叶子节点
- 步骤 4：基于叶子节点构建内部节点
- 结果：完美的填充度


为什么这样高效？
- **只需访问缓存中的索引块**
- 成本：每个块只写一次

## 6.1 Example 

![[Pasted image 20251123164440.png]]

![[Pasted image 20251123164448.png]]

![[Pasted image 20251123164516.png]]


![[Pasted image 20251123164529.png]]


![[Pasted image 20251123164536.png]]


## 6.2 普通逐条插入 vs 批量加载

### 6.2.1 普通逐条插入（低效）
每插入一条记录：
1. 从根节点开始搜索
2. 读取路径上的所有节点到内存
3. 找到叶子节点位置
4. 如果叶子节点满 → 分裂 → 向上递归调整
5. 每次插入可能触发多次磁盘 I/O

**问题：**
- 10 万条记录 → 10 万次从根到叶子的路径搜索
- 每次插入的节点可能不在缓存中 → 大量随机 I/O
- 树的高度 h → 每次插入至少 h 次读 + 写


### 6.2.2 批量加载（高效）
步骤图解：

Step 1：创建键-指针对
```
数据块 1: [key1, key2] → 指针 P1
数据块 2: [key3, key4] → 指针 P2
...
```
每个数据块对应一个键-指针对（稀疏索引）

Step 2：按键排序
```
排序后：
(k1, P1), (k2, P2), (k3, P3), ...
```

Step 3：创建叶子节点（填满）
假设每个叶子节点可装 3 个索引项：
```
叶子节点 L1: [(k1,P1), (k2,P2), (k3,P3)]
叶子节点 L2: [(k4,P4), (k5,P5), (k6,P6)]
```
**关键：** 一次性按顺序创建叶子节点，每个叶子节点写一次磁盘，不再修改。

Step 4：构建内部节点
基于叶子节点构建上层节点：
```
内部节点 I1: [k3, k6]  
指向 L1, L2, L3
```
同样自底向上，每个内部节点只写一次。


## 6.3 为什么"只需访问缓存中的索引块"？

普通插入：
每次插入需要读一个叶子节点，可能它不在缓存 → 磁盘 I/O

批量加载：
1. **顺序创建叶子节点**  
   - 创建 L1 → 写入磁盘  
   - 创建 L2 → 写入磁盘  
   - 创建时只需在内存中填充，写回磁盘后不再读它

2. **构建内部节点时**  
   - 只需要读取叶子节点的第一个键（在内存中已排序）
   - 内部节点也是顺序创建，一次写入

**结果：**
- 每个索引块（叶子或内部节点）只被写入一次
- 不需要从磁盘读取已有的索引块（因为是从零构建）
- 所以"只需访问缓存中的索引块"指的是：
  - 创建时在内存中组装
  - 写回磁盘后不再需要随机读取

---

## 6.4 成本对比

| 操作 | 读 I/O | 写 I/O | 是否随机 |
|------|--------|--------|----------|
| 逐条插入 | h 次/记录 | 1~3 次/记录 | 随机读 |
| 批量加载 | 0（仅数据排序） | 每个块 1 次 | 顺序写 |

---

## 6.5 总结
批量加载的核心优势：
1. **利用排序** → 避免随机搜索
2. **自底向上构建** → 每个节点只写一次
3. **无节点分裂** → 无额外 I/O
4. **完美填充度** → 空间利用率高

所以在大数据量建索引时，批量加载比逐条插入快几个数量级。


# 7 Further Aspects

• If search key is a string => large memory consumption
• Solution: Store common prefixes (i.e., single or multiple characters) on the path and construct value by them

![[Pasted image 20251123164632.png]]

B-Tree — Further Aspects
• Variable-length fields (further techniques)
• Concurrent access
• Duplicate keys
• …
Douglas Comer: “The Ubiquitous B-Tree”, ACM Comput. Surv. 11(2): 121-137,1979
Goetz Graefe: “Modern B-Tree Techniques”, Found. Trends Databases 3(4): 203-402, 2011
Goetz Graefe: “More Modern B-Tree Techniques”, Found. Trends Databases 13(3): 169-249, 2024

