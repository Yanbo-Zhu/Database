
# 1 B⁺ Trees

## 1.1 Explain the data structure from a B tree given below:



![](https://dima.gitlab-pages.tu-berlin.de/dbt/exercise-base/_images/btree-block.png)


![[Pasted image 20251123165840.png]]



## 1.2 What is the advantage of using B/B⁺ Trees over Binary Trees?

In B trees, only the leaves point to the data. Also in B+ Trees, a leaf points t the next leaf in sequence (to the right)
Binary tees have less condition checking but more I/O (disk accesses)
B+ Tree are much flatter, which means , they have more condition checks but less I/O accesses
Because the time spent on I/O is way more costly than the cost conditional branches 


![[Pasted image 20251123165609.png]]

## 1.3 How to choose parameter n in the context of B⁺ trees, and what is its relation to the number of keys and the number of pointers?

we have to choose n as large as possible which still allows n keys and n+1 pointers


## 1.4 Calculate n based on the following information:

- block size: 8192 bytes
- key size: 8 bytes
- pointer size: 8 bytes

![[Pasted image 20251123165732.png]]


# 2 B⁺ Tree Insertion

Given the following B Tree with a block size of 4, how does the B⁺ Tree look after inserting 17 and 16?
![[btrees_start.png]]

![[Pasted image 20251123164931.png]]

![[Pasted image 20251123165336.png]]

![[Pasted image 20251123165341.png]]

![[Pasted image 20251123165347.png]]

# 3 B+ Tree Deletion

Given a B⁺ Tree with a block size of 4, remove 24 and 30 from the following B⁺ Tree:
![[btrees_delete_start.png]]


![[Pasted image 20251123164941.png]]


![[Pasted image 20251123165356.png]]

![[Pasted image 20251123165405.png]]


# 4 Qizz


## 4.1 ##

In both B and B⁺ Trees, paths from root to leaf have the same length.: True
Leaves in a B⁺-Tree contain keys and pointers to the block containing the data.: True
The leaf nodes of both B and B⁺ Trees are connected to the adjacent right leaf node.: False
Leaves in a B⁺-Tree contain keys and values (i.e., complete tuples with a key).: False


## 4.2 B⁺ tree

Consider the B⁺ tree below with the following properties:

Each node (except the root) contains 
 keys.
Inner nodes: The keys in the subtree below the pointer 
 are less than the key 
; the keys in the subtree below the pointer 
 are greater or equal than the key 
.
Delete operations will first try to steal nodes from a direct sibling (first right sibling, then left sibling). If this is not possible, they will merge with a sibling (first right sibling, then left sibling).
Pointers 
 in leaves point to row IDs. Pointer 
 in leaves points to the next leaf.
Please read section 14.2.6 in "Database Systems The Complete Book" second edition to look for the details of the deletion algorithm when one or more of the above conditions are violated post deletion.


考虑下面的 B⁺ 树，它具有以下性质：

- 每个节点（根节点除外）包含 ⌈(n+1)/2⌉ -1 个键。
- 内部节点：指针 p 所指的子树中的键都小于键 k；指针 p 所指的子树中的键都大于或等于键 k。
- 删除操作将首先尝试从直接兄弟节点窃取节点（先右兄弟，后左兄弟）。如果这不可能，它们将与兄弟节点合并（先右兄弟，后左兄弟）。
- 叶子中的指针 p 指向行 ID。叶子中的指针 p_next 指向下一个叶子。
- 如果在删除后违反上述一个或多个条件，请阅读《Database Systems The Complete Book》第二版第 14.2.6 节以了解删除算法的细节。

![](image/Pasted%20image%2020260302132619.png)

Which of the following choices represents the B⁺ tree after deletion of the key 36?



答案是 

![](image/Pasted%20image%2020260302133051.png)



## 4.3 
Consider the B⁺ tree below with the following properties:

Each node (except the root) contains  keys.
Inner nodes: The keys in the subtree below the pointer  are less than the key  ; the keys in the subtree below the pointer   are greater or equal than the key 

Insert operations may only trigger node splits but not shifting of keys into bordering leafs.
When a node is split, the middle key is moved to the parent node.
Pointers  in leaves point to row IDs. Pointer  in leaves points to the next leaf.

---
请考虑下面的 B⁺ 树，它具有以下性质：

- 每个节点（根节点除外）包含 ⌈(n+1)/2⌉ - 1 个键。
- 内部节点：指针 pᵢ 所指的子树中的键都小于键 kᵢ；指针 pᵢ₊₁ 所指的子树中的键都大于或等于键 kᵢ。
- 插入操作可能只会触发节点分裂，而不会将键移到相邻的叶子中。
- 当节点分裂时，中间键被移到父节点。
- 叶子中的指针 p 指向行 ID。叶子中的指针 pₙₑₓₜ 指向下一个叶子。


![](image/Pasted%20image%2020260302133117.png)


Which of the following choices represents the B⁺-Tree after insertion of the key 43?
正确打印
因为插入后 中间 key为34， 则34 被挪到了上面 
![](image/Pasted%20image%2020260302133608.png)



错误的答案： 
![](image/Pasted%20image%2020260302133622.png)

