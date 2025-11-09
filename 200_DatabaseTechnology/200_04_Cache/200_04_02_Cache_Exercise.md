
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



