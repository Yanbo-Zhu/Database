

# 1 Memory Hierarchy

1. Examine the memory hierarchy, and explain how the following factors change when we go down in the hierarchy.
- Speed
- Capacity
- Cost per byte
![[memory_hierarchy.png]]

![[Pasted image 20251108233750.png]]

---


2. What is caching? Explain the principal of locality, and name two kind of localities with their meanings.

Principal locality is the tendency of a processor to access the same set of memory locations repetitively over a short period of time 
Spatial locality 
Temporal locality 

- **Spatial locality（空间局部性）**：  
    当程序访问某个内存地址时，它**很可能会访问与该地址相邻的地址**。  
    换句话说，数据在空间上彼此接近时，更有可能被连续访问。  
    👉 例子：遍历数组时，访问 `A[i]` 后很快会访问 `A[i+1]`。
    
- **Temporal locality（时间局部性）**：  
    当程序访问某个数据时，它**在不久的将来很可能会再次访问同一个数据**。  
    👉 例子：循环中多次使用同一个变量或缓存的数据。


---


3. What is a “cache hit” and a “cache miss”?
    
    - If a cache has 90% hit rate, what does that mean?
        
    - Calculate the cache hit ratio if 10 cache misses are observed during 50 memory access attempts.

![[Pasted image 20251108234100.png]]


---

4. What is the typical cache hierarchy of today’s computers?
    How does the **size** and **speed** of these caches influence overall system performance?

![[Pasted image 20251108234118.png]]

---

5. Examine the properties of two different CPUs, and explain their differences.
![[Pasted image 20251108225226.png]]


---


6. What is virtual memory, and how does it function within a computer system?

Solution: Virtual Memory is a Momery management technique that allows a computer to run more application than it's physical RAM would normally allow 

When the system, runs out of physical RAM , it moves less used data from RAM to a speical file on the storage drive , a process called swapping or paging 

# 2 Disk Operations

1. Describe the internal structure of a hard disk.
- What are the roles of components such as platters, tracks, cylinders, sectors, and the read/write head?

![[Pasted image 20251108234318.png]]


---

2. Explain the disk access characteristics? What a disk latency consists of?
- How best case and worst case disk latency looks like?

![[Pasted image 20251108235250.png]]

---

3. A hard disk consists of 10 disks with 2 surfaces each. Each surface contains tracks, and each track holds an average of 512 sectors. Each sector is 4,096 bytes in size.
- Calculate the overall storage capacity of the hard disk.


---


4. Following latency information is given for an access to a specific part of a disk:
- Full Seek Time: 12 ms
- Rotational Latency: 2 ms	
- Transfer Time: 0.4 ms
- Compute the **minimum time (best case)** it would take to access and read the data from the disk.
- Compute the **average time** it would take to access and read the data from the disk.

![[Pasted image 20251108235338.png]]
## 2.1 Average Seek

Examine the SSD architecture given below. Explain each component and define their roles.

![[Pasted image 20251108235409.png]]



![[Pasted image 20251108235401.png]]


# 3 I/O Model of Computation

1. Why is it important to minimize disk I/Os in a DBMS?
![[Pasted image 20251108235645.png]]

---

2. What is the key difference between the **RAM model** and the **I/O model** of computation in databases

![[Pasted image 20251108235906.png]]


---

3. When using different types of indexes, why is it generally more efficient to fetch an entire block of data from disk rather than just the specific relevant information?    
Note: Remember the example below from the lecture.
Looking for a tuple t in relation R with key k (index on key attribute)
- Alternative A only returns in which block t is located
- Alternative B also provides information where in block t is located

![[Pasted image 20251109000052.png]]


# 4 Elevator Algorithm

1. How does the elevator algorithm schedule requests for block accesses on a hard disk?
![[Pasted image 20251108235619.png]]


![[Pasted image 20251109000222.png]]

---


2. When the Elevator algorithm cannot be used?




3. When is the final request completed (in milliseconds) if the request are processed in an order determined by the elevator algorithm?
- Average transfer time for a block: 0.13 ms
- Average rotational latency: 4.17 ms
- Time to start/stop the read/write head: 1 ms
- Speed of the head = 4000 tracks per ms
- Head starts at 32000

![[Pasted image 20251108225333.png]]


![[Pasted image 20251109000320.png]]

# 5 RAID
1. What is RAID 0? What are the benefits of RAID 0?
2. What is RAID 1? What are the benefits of RAID 1?

![[Pasted image 20251109000340.png]]



