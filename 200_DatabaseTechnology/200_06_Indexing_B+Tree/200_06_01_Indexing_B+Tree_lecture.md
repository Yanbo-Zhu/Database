
# 1 Introduction
## 1.1 Motivation

![[Pasted image 20251123160843.png]]

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


- •At least (n+1)/2 of pointers point to data records,
- At least half of the records pointer (i.e. keys) must be used 


## 2.2 Inner Nodes

At least (n+1)/2 of pointers point to blocks of lower level,
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

If the number of keys is uneven, put +1 left or right

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

![[Pasted image 20251123172012.png]]


----


• Search corresponding leaf

• Delete key
	• If at least minimal number of keys in node: Nothing else to do
	• If too few keys in node:
		• Steal a key from a sibling node (left or right) if possible, i.e., a sibling has more than the minimum number of keys
	• If stealing is not possible:
• adjust keys of parent (and delete one key-pointer pair);
recursively apply deletion algorithm if necessary
• two siblings with minimal and sub-minimal number of keys can be merged
• If too few keys in node:
• Steal a key from a sibling node (left or right) if possible,
i.e., a sibling has more than the minimum number of keys
• adjust keys of parent if required

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
• 4096 bytes blocks;
4 bytes keys; 8 bytes pointers;
no meta data (e.g., block header)
• 4n+8(n+1)≤4096 —> Choose n=340   . 一个block 一共有多少个 keys



## 5.1 Bulk Loading  (构造 B+ Tree )
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
• Step 2: Sort pairs on sort key
• Step1: Create key-pointer pairs for all blocks
• Step 3: Create filled/full B-tree leaves
• Step 4: Construct inner nodes based on leaves
• Result: Perfect filing degree


Why is this efficient?
• ==Only access index blocks in cache==
• Cost: Write each block once



![[Pasted image 20251123164440.png]]

![[Pasted image 20251123164448.png]]

![[Pasted image 20251123164516.png]]


![[Pasted image 20251123164529.png]]


![[Pasted image 20251123164536.png]]

# 6 Further Aspects

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

