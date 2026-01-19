
# 1 What is a hash table?
- An unordered associative array that maps keys to values.
- Associative  it "associates" a value with a key.
- Keys and values could be anything (INTEGERS, STRINGS, DATES, …)

Uses a hash function to compute an offset into this array.
- hash(key: K) -> u64
- Example: hash(12) -> 2   - hash(k) = k % len(A) and len(A) = 10
- The value would be stored at index 2

Hash functions are deterministic: hash(k) will always produce the same result.

They should spread the keys evenly across the array offsets & be fast.


## 1.1 Hash Function and Mapping

The hash function (H) is the key itself. The array index is calculated using the modulo operator (Index = Key % Array Size).

| **Key (Integer)** | **Hash Value (Key)** | **Array Index (Key % 8)** | **Value (String)** |
| ----------------- | -------------------- | ------------------------- | ------------------ |
| `10`              | 10                   | 2                         | `Client A`         |
| `19`              | 19                   | 3                         | `Client B`         |
| `9`               | 9                    | 1                         | `Client C`         |
| `12`              | 12                   | 4                         | `Client D`         |
| `13`              | 13                   | 5                         | `Client E`         |
| `30`              | 30                   | 6                         | `Client F`         |
## 1.2 Array Contents (Buckets)

|**Array Index**|**Key**|**Value**|
|---|---|---|
|**0**|`NULL`|`NULL`|
|**1**|`9`|`Client C`|
|**2**|`10`|`Client A`|
|**3**|`19`|`Client B`|
|**4**|`12`|`Client D`|
|**5**|`13`|`Client E`|
|**6**|`30`|`Client F`|
|**7**|`NULL`|`NULL`|


# 2 What is the a) time and b) space complexity of a hash table?

Space Complexity: 

Time Complexity:

- Average:  - When?
    
- Worst:  - When?

![](image/Pasted%20image%2020260119203259.png)

# 3 What operations does a hash table support?

- `GET(k)`: given key `k`, retrieve the value
    
- `PUT(k, v)`: put value `v` into the table and associate with key `k`
    
- `DELETE(k)`: remove `k` and its associated value


## 3.1 hash table squeezes the key domain into a fixed amount of memory.

Tradeoff:
- Array of size 1  collisions on every `PUT`, linear search
- Array of size  for signed 32-bit integers, no collisions at all, exceeds available memory
-  for `u64`
-  for `String`

Reality:
- How many unique keys will we need?   - Index on column `age` () vs. primary `AUTOINCREMENT` index , what about `String`?
- We need `hash` to work with arbitrary byte sequences (e.g., `String`)

![](image/Pasted%20image%2020260119205052.png)

# 4 What is a hash collision?

Two keys hashing to the same offset in the array.

# 5 How can we deal with collisions?

Approach #1:
- **open addressing**
- Move the collided value to another bucket based on a **hashing scheme** (e.g., linear probing, quadratic probing, cuckoo hashing).
- Sequentially scanning for a given key is cheap, but needs rehashing & reorganizing when full.

Approach #2:
- **separate chaining**
- Store a pointer (pageID, offset) to the “next” value in the bucket.
- No rehashing and data shuffling (unlimited capacity), but more expensive lookups (follow pointers, random I/O) because overflow blocks are not stored adjacent.


# 6 What are the advantages/disadvantages of hash indices over B+Trees?

Advantages
- Average time complexity for locating a key is 
- Therefore: less disk I/O

Disadvantages
- No support for range queries
- Performance degradations on increasing **fill level**.
- Rehashing and moving data is computationally expensive.

![](image/Pasted%20image%2020260119205418.png)


# 7 What can we do if we don’t know the number of unique keys upfront?

Static hash tables:
- Double/halve the array, rehash all keys, shuffle values.

Dynamic hash tables:
- Increase/decrease number of buckets without rehashing everything.

# 8 Recall: Extensible Hash Table
- **Indirection** for buckets: array of pointers to blocks
- **Growing/shrinking** by updating pointer array
- Buckets can **share** blocks (multiple pointers point to the same block)
- Bucket numbers use a **prefix** of  of the hash value’s bits
- Bucket array has  elements

![](image/Pasted%20image%2020260119205529.png)

# 9 Extensible Hash Map Insertion (block size = 2)

- `PUT(1010)`
- `PUT(0000)`
- `PUT(1100)`
- `PUT(1111)`
- `PUT(0110)`
- `PUT(0011)`

![](image/Pasted%20image%2020260119205616.png)

![](image/Pasted%20image%2020260119205629.png)


![](image/Pasted%20image%2020260119205641.png)


![](image/Pasted%20image%2020260119205555.png)

# 10 Recall: Linear Hash Table

- Number of buckets grows/shrinks **linearly**
- No bucket directory
- Instead: chain of blocks on overflow
- Choose some buckets  such that the average number of records per bucket is a fixed fraction, e.g., 
-  bits to identify a bucket, use the rightmost (LSB) bits of the hash value

![](image/Pasted%20image%2020260119205750.png)


# 11 Linear Hash Map Insertion (block size = 2)

Given:

- A linear hash map with the following buckets: `0 : {0110, 1010}`, `1 : {0001}`
    
- LSB as hash function
    
- Maximum filling ratio of 85%
    
- `i`: number of relevant bits
    
- `n`: number of buckets
    
- `r`: number of records

What happens with after each operation:

- `PUT(0111)`
    
- `PUT(1110)`
    
- `PUT(0010)`

![](image/Pasted%20image%2020260119205820.png)

![](image/Pasted%20image%2020260119205858.png)

![](image/Pasted%20image%2020260119205917.png)



![](image/Pasted%20image%2020260119205811.png)
