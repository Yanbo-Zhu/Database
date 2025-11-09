
# 1 Caching

**Question:** What are the benefits of caching?

- Reduced disk I/O
- Faster reads
- Faster writes
- Decrease latency


# 2 Buffer Manager

What are the steps for Buffer Manager lookups?

1. evict page . when Cache is full
2. only if dirty, the entry is modified, we need  cache update also in main momery
3. writeQueue(): process it sequentielly 


![[Pasted image 20251109112925.png]]


# 3 Cache Hit / Miss

- What is a cache hit?   Data is already in Cache (the Page we are looking for)
- What is a cache miss?  Data ist not in Cache, still in Memory

## 3.1 Memoization Example

Let’s take a look at the following example. We are adding up all the numbers in a range by using sum function, and then caching the result for that input.

Here, we are using a type of caching called [memoization](https://en.wikipedia.org/wiki/Memoization) to keep the result of the function in cache, in case if it is going to be called again with the same input.

- We first run the function with range `[0, 10m]`
- Then we run the function again with the same input.
- Finally, we run the function with range `[0, 15m]`

**Question:** How would you expect the runtime of these calls (t1, t2, t3), if you would compare?

![[Pasted image 20251109113419.png]]

![[Pasted image 20251109113429.png]]

```
import functools  # for caching
import timeit  # for measuring time

# cache results for function sum
sum = functools.cache(sum)

# run the function in range [0,10m]
# and print how long it takes
print(
    timeit.timeit(
        "sum(range(10_000_001))",
        globals=globals(),
        number=1,
    )
)

# run the function again with the same input
# and print how long it takes
print(
    timeit.timeit(
        "sum(range(10_000_001))",
        globals=globals(),
        number=1,
    )
)

# run the function with different input
# and print how long it takes
print(
    timeit.timeit(
        "sum(range(15_000_001))",
        globals=globals(),
        number=1,
    )
)
```

0.44990579783916473
3.039836883544922e-06
0.6597890555858612


# 4 Eviction

When there is no space in cache, we need to evict some data to replace it with the new ones.

There a varying eviction methods to choose the next page in the cache for replacement.

**Question:** When to use LRU, and when to use LFU?

![[Pasted image 20251109113529.png]]

LRU: Good if you access data in a burst and then forget it 
LFU: Good if parts are often accessed icer and over again




# 5 FIFO / Queue

Image we have the following list of numbers that we access in this order:
![[Pasted image 20251109112050.png]]

We have a cache with size of 5, and we use FIFO/queue as the eviction method.

**a)** How many cache hits?

**b)** How many cache misses?
```
|  |  |  |  |  | <- 12
|  |  |  |  |  | <- 26
|  |  |  |  |  | <- 34
|  |  |  |  |  | <- 26
|  |  |  |  |  | <- 15
|  |  |  |  |  | <- 18
|  |  |  |  |  | <- 12
|  |  |  |  |  | <- 19
|  |  |  |  |  | <- 12
|  |  |  |  |  | <- 26
|  |  |  |  |  |      
```

![[Pasted image 20251109113650.png]]

# 6 Least Recently Used (LRU)

Let’s try the same example with the same input, but now with LRU as eviction method.

**a)** How many cache hits?

**b)** How many cache misses?

```
|  |  |  |  |  | <- 12
|  |  |  |  |  | <- 26
|  |  |  |  |  | <- 34
|  |  |  |  |  | <- 26
|  |  |  |  |  | <- 15
|  |  |  |  |  | <- 18
|  |  |  |  |  | <- 12
|  |  |  |  |  | <- 19
|  |  |  |  |  | <- 12
|  |  |  |  |  | <- 26
|  |  |  |  |  |      
```

![[Pasted image 20251109113658.png]]


# 7 Least Frequently Used (LFU)

Let’s try the same example with the same input, but now with LFU as eviction method.

**a)** How many cache hits?

**b)** How many cache misses?

```
|  |  |  |  |  | <- 12
|  |  |  |  |  | <- 26
|  |  |  |  |  | <- 34
|  |  |  |  |  | <- 26
|  |  |  |  |  | <- 15
|  |  |  |  |  | <- 18
|  |  |  |  |  | <- 12
|  |  |  |  |  | <- 19
|  |  |  |  |  | <- 12
|  |  |  |  |  | <- 26
|  |  |  |  |  |      
```


![[Pasted image 20251109113639.png]]


# 8 CLOCK - Second Chance Algorithm

What if we use CLOCK (Second Chance) algorithm instead?

**a)** How many cache hits?
**b)** How many cache misses?

**Note:** In the schema below, * denotes Reference Bit is set to 1, and 0 otherwise.

```
        |  |          {12,26,34,26,15,18,12,19,12,26}
         |
  |  |   |    |  |


    |  |    |  |      hit: 0 miss: 0
--------------------
        |  |         

  |  |        |  |
           
            
    |  |    |  |     
--------------------
        |  |         

  |  |        |  |
           
            
    |  |    |  |
--------------------
        |  |         

  |  |        |  |
           
            
    |  |    |  |
--------------------
        |  |         

  |  |        |  |
           
            
    |  |    |  |
```

![[Pasted image 20251109113718.png]]


![[Pasted image 20251109113730.png]]


![[Pasted image 20251109113754.png]]

![[Pasted image 20251109113807.png]]


![[Pasted image 20251109113817.png]]


![[Pasted image 20251109113823.png]]


# 9 Quiz 1: Storage, Data Layouts, Caching


## 9.1 


The cache in the system can hold 3 pages in total and is initially empty. Consider the following sequence of page requests to the cache: 15, 4, 6, 15, 10, 11, 6, 9, 12, 1

How many cache hits will occur if the cache replacement policy is FIFO?

Additional information:

Cache Hit - A cache hit occurs when a requested page is found in the cache.

Cache Miss - A cache miss occurs when a requested page is not found in the cache. Writing on an empty cache is also considered as a cache miss.



|Step|Requested Page|Cache Before|Hit/Miss|Cache After (FIFO order)|
|---|---|---|---|---|
|1|15|[ ]|❌ Miss|[15]|
|2|4|[15]|❌ Miss|[15, 4]|
|3|6|[15, 4]|❌ Miss|[15, 4, 6]|
|4|15|[15, 4, 6]|✅ **Hit**|[15, 4, 6]|
|5|10|[15, 4, 6]|❌ Miss → evict oldest (15)|[4, 6, 10]|
|6|11|[4, 6, 10]|❌ Miss → evict oldest (4)|[6, 10, 11]|
|7|6|[6, 10, 11]|✅ **Hit**|[6, 10, 11]|
|8|9|[6, 10, 11]|❌ Miss → evict oldest (6)|[10, 11, 9]|
|9|12|[10, 11, 9]|❌ Miss → evict oldest (10)|[11, 9, 12]|
|10|1|[11, 9, 12]|❌ Miss → evict oldest (11)|[9, 12, 1]|

|Total requests|10|
|---|---|
|Cache hits|**2**|
|Cache misses|8|

## 9.2 ##
The cache in the system can hold 3 pages in total and is initially empty. Consider the following sequence of page requests to the cache: 15, 4, 6, 15, 9, 11, 6, 12, 15, 6

**How many cache misses will occur if the cache replacement policy is FIFO?**

Additional information:

**Cache Hit** - A cache hit occurs when a requested page is **found** in the cache.

**Cache Miss** - A cache miss occurs when a requested page is **not found** in the cache. Writing on an empty cache is also considered as a cache miss.

|Step|Requested Page|Cache Before|Hit/Miss|Cache After (FIFO order: oldest → newest)|
|---|---|---|---|---|
|1|15|[ ]|❌ Miss|[15]|
|2|4|[15]|❌ Miss|[15, 4]|
|3|6|[15, 4]|❌ Miss|[15, 4, 6]|
|4|15|[15, 4, 6]|✅ Hit|[15, 4, 6]|
|5|9|[15, 4, 6]|❌ Miss → evict oldest (15)|[4, 6, 9]|
|6|11|[4, 6, 9]|❌ Miss → evict oldest (4)|[6, 9, 11]|
|7|6|[6, 9, 11]|✅ Hit|[6, 9, 11]|
|8|12|[6, 9, 11]|❌ Miss → evict oldest (6)|[9, 11, 12]|
|9|15|[9, 11, 12]|❌ Miss → evict oldest (9)|[11, 12, 15]|
|10|6|[11, 12, 15]|❌ Miss → evict oldest (11)|[12, 15, 6]|


| Total requests   | 10             |
| ---------------- | -------------- |
| **Cache hits**   | 2 (steps 4, 7) |
| **Cache misses** | 8              |


## 9.3 ##

The cache in the system can hold 3 pages in total and is initially empty. Consider the following sequence of page requests to the cache: 15, 4, 6, 15, 9, 11, 6, 9, 12, 1

**How many cache hits will occur if the cache replacement policy is LRU?**

Additional information:

**Cache Hit** - A cache hit occurs when a requested page is **found** in the cache.

**Cache Miss** - A cache miss occurs when a requested page is **not found** in the cache. Writing on an empty cache is also considered as a cache miss.


|Step|Requested Page|Cache Before (MRU → rightmost)|Hit/Miss|Cache After (MRU → rightmost)|
|---|---|---|---|---|
|1|15|[ ]|❌ Miss|[15]|
|2|4|[15]|❌ Miss|[15, 4]|
|3|6|[15, 4]|❌ Miss|[15, 4, 6]|
|4|15|[15, 4, 6]|✅ Hit|[4, 6, 15]|
|5|9|[4, 6, 15]|❌ Miss → evict LRU (4)|[6, 15, 9]|
|6|11|[6, 15, 9]|❌ Miss → evict LRU (6)|[15, 9, 11]|
|7|6|[15, 9, 11]|❌ Miss → evict LRU (15)|[9, 11, 6]|
|8|9|[9, 11, 6]|✅ Hit|[11, 6, 9]|
|9|12|[11, 6, 9]|❌ Miss → evict LRU (11)|[6, 9, 12]|
|10|1|[6, 9, 12]|❌ Miss → evict LRU (6)|[9, 12, 1]|


|Total requests|10|
|---|---|
|**Cache hits**|2 (at steps 4 and 8)|
|**Cache misses**|8|


## 9.4 CLOCK (a.k.a. Second Chance) algorithm

Consider the CLOCK (a.k.a. Second Chance) algorithm. The system cache has space for 8 pages, and the current content of the cache is shown in the figure below.
![[Pasted image 20251109131936.png]]

Assume that when a page is accessed and it is already in the cache, its reference bit is set to 1 (if it is 0, it is changed to 1; if it is 1, then it is left as 1). The clock hand does not move in such a case (i.e., it stays where it was before this access). The clock hand only moves when the accessed page is not in the cache and the cache is looking for a page to evict by moving clockwise. When a new page is brought into the cache its reference bit is set to 1.

假设如下规则适用于页面缓存（Cache）系统：

- 当某个页面被访问到，并且它**已经在缓存中**时：  
    → 它的 **引用位（reference bit）** 会被设置为 **1**。  在轮序的时候 其他的 page 的 reference bit 的值不会改变 
    - ⚠️ 此时 **时钟指针（clock hand）不会移动**，即保持原位。
- 时钟指针 **只在需要淘汰页面（即缓存缺页、要找替换页）** 时才会移动，  并按顺时针方向依次检查缓存中的页面。
	- 在轮序的时候 其他的 page 的 reference bit 的值会改变 
		-  如果原本是 0，则改成 1；
	    - 如果原本已经是 1，则保持不变。
	- 当一个新页面被装入缓存时，它的 **引用位被设置为 1**。
	-  **时钟指针（clock hand）移动**， 指向新页面的下一个页面。


Also assume that in the first figure; red box indicates that the reference bit is set to 1, and orange box indicates that reference bit is set to 0.

- **红色框（red box）** 表示该页的引用位 = 1 **橙色框（orange box）** 表示该页的引用位 = 0

Consider the following sequence of page requests to the cache:

> 20, 15, 36, 6, 42, 3, 61, 89

Fill in the pages of the clock with regards to the page numbers after the last request has been completed (after all the pages in the series are accessed).


---

Initial clockwise order from the hand (at **6**) is:

`[6(1), 28(0), 34(0), 7(1), 36(0), 70(0), 52(0), 12(1)]`



**Final pages (clockwise from the original hand position):**  
`[6, 20, 15, 61, 89, 42, 3, 12]`

**Reference bits (same order):**  
`[0, 0, 0, 1, 1, 1, 1, 0]`

(After inserting the last page 89, the clock hand rests at the next frame, i.e., on the frame holding 42.)


---

Requests: 20, 15, 36, 6, 42, 3, 61, 89

**1) Request 20 — MISS**

- Hand @ 6(1) → set 0, move to 28(0) → evict 28 → insert 20(1).
    
- Hand stops at next frame (34).  
    State: `[6(0), 20(1), 34(0), 7(1), 36(0), 70(0), 52(0), 12(1)]` (hand @ **34**)
    

**2) Request 15 — MISS**

- Hand @ 34(0) → evict → insert 15(1).
    
- Hand → next (7).  
    State: `[6(0), 20(1), 15(1), 7(1), 36(0), 70(0), 52(0), 12(1)]` (hand @ **7**)
    

**3) Request 36 — HIT**

- Set 36’s bit 0→1. Hand stays @ 7.  
    State: `[6(0), 20(1), 15(1), 7(1), 36(1), 70(0), 52(0), 12(1)]` (hand @ **7**)
    

**4) Request 6 — HIT**

- Set 6’s bit 0→1. Hand stays @ 7.  
    State: `[6(1), 20(1), 15(1), 7(1), 36(1), 70(0), 52(0), 12(1)]` (hand @ **7**)
    

**5) Request 42 — MISS**

- Scan from 7(1) → set 0 → 36(1) → set 0 → 70(0) → evict → insert 42(1).
    
- Hand → next (52).  
    State: `[6(1), 20(1), 15(1), 7(0), 36(0), 42(1), 52(0), 12(1)]` (hand @ **52**)
    

**6) Request 3 — MISS**

- Hand @ 52(0) → evict → insert 3(1).
    
- Hand → next (12).  
    State: `[6(1), 20(1), 15(1), 7(0), 36(0), 42(1), 3(1), 12(1)]` (hand @ **12**)
    

**7) Request 61 — MISS**

- 12(1) → set 0 → 6(1) → set 0 → 20(1) → set 0 → 15(1) → set 0 → 7(0) → evict → insert 61(1).
    
- Hand → next (36).  
    State: `[6(0), 20(0), 15(0), 61(1), 36(0), 42(1), 3(1), 12(0)]` (hand @ **36**)
    

**8) Request 89 — MISS**

- 36(0) → evict → insert 89(1).
    
- Hand → next (42).  
    Final state: `[6(0), 20(0), 15(0), 61(1), 89(1), 42(1), 3(1), 12(0)]`  
    (hand now sits on **42**)

![[Pasted image 20251109133356.png]]