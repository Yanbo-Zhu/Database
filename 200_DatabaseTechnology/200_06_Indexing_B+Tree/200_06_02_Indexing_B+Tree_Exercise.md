
# 1 B⁺ Trees

## 1.1 Explain the data structure from a B tree given below:
    

![](https://dima.gitlab-pages.tu-berlin.de/dbt/exercise-base/_images/btree-block.png)


![[Pasted image 20251123165840.png]]



## 1.2 What is the advantage of using B/B⁺ Trees over Binary Trees?

In B* trees, only the leaves point to the data. Also in B* Trees, a leaf points t the next leaf in sequence (to the right)

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

# 3 B⁺ Tree Deletion

Given a B⁺ Tree with a block size of 4, remove 24 and 30 from the following B⁺ Tree:
![[btrees_delete_start.png]]


![[Pasted image 20251123164941.png]]


![[Pasted image 20251123165356.png]]

![[Pasted image 20251123165405.png]]
