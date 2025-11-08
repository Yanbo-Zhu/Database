
# 1 Memory Hierarchy

![[Pasted image 20251108230456.png]]


![[Pasted image 20251108230503.png]]


![[Pasted image 20251108230511.png]]

## 1.1 Cache 


CPU Caches:
- It brings data from main memory to the CPU memories
- Hardware controlled
File System Cache:
- It brings data from permanent storage to main memory
- OS controlled
DBMS Buffer Cache:
- It brings data from permanent storage to main memory
- DBMS controlled

![[Pasted image 20251108231306.png]]

![[Pasted image 20251108230631.png]]

---


Locality Access

![[Pasted image 20251108230802.png]]


## 1.2 Main Memory





# 2 Disks

![[Pasted image 20251108231218.png]]

![[Pasted image 20251108235201.png]]

## 2.1 Disk Latency 
Accessing (reading or writing) a block requires four steps:
1. Transferring blocks between main memory and disk controller
	— Communication delay (Typically ignored, PCI speed or faster, +11 GB/s)
2. Position head assembly at cylinder containing the track containing the block
	— Seek time
3. Wait until first sector of block rotates under read/write head
	— Rotational latency
4. Transfer bits passing under the head to disc controller (or vice versa)
	— Transfer time

Disk latency = Seek time + Rotational latency + Transfer time


![[Pasted image 20251108231829.png]]


![[Pasted image 20251108231838.png]]


![[Pasted image 20251108231847.png]]

![[Pasted image 20251108231916.png]]





# 3 Efficient Disk Operations

![[Pasted image 20251108232022.png]]



# 4 Access Acceleration

1. Store blocks that are processed together on same cylinder
	→ Reduced seek times
2. Distribute data on multiple disks
	→ Parallel reads and writes
3. Replicate data on multiple disks
	→ Parallel reads
4. Use a disk scheduling algorithm
	→ Reduced seek times
5. Prefetch blocks
	→ Reduced latencies
	


## 4.1 Disk Scheduling/ Elevator Algorithm

Key Idea: Disk controller reorders access requests
Useful when many small requests, each accessing few blocks
• Transactional workload (OLTP)
• Goal: Increases the throughput


Elevator Algorithm
**类比（Analogy）**
- 电梯在大楼中上下移动。
- 当有人要进入或离开时，电梯会停下。
- 如果在当前行进方向上没有等待的人，电梯就会改变方向。

**算法（Algorithm）**
- 磁盘的读/写磁头在磁盘上从内圈向外圈移动，然后再返回。
- 当磁头到达某个柱面（cylinder）时，会停下来处理该柱面的所有请求。
- 当当前方向上不再有待处理的请求时，磁头就会**掉头反向移动**。


![[Pasted image 20251108232628.png]]

![[Pasted image 20251108232658.png]]






# 5 Disk Terminology and Elevator Algorithm

Disk Sections Terminology
Current slides from the lecture contradicts itself because of the illustration from Disk Summary slide (at page: 36). We will remove that slide and update the presentation. Until then, you can ignore the Disk Summary slide. The terminology built between pages [24-35] is still valid, and you can take it as reference.

Elevator (SCAN) Algorithm
You can take the description from Database Systems The Complete Book (2nd edition) in Chapter 13.3.5 (Disk Scheduling and Elevator Algorithm, page 571) as reference. I also added a link to PDF of this book's 2nd edition in the [Book Reference](https://isis.tu-berlin.de/mod/page/view.php?id=2169570 "Book Reference") page.

In essence, the disk head moves toward a direction until there is no more request to process in that direction.

### 5.1.1 Small Example

Below is an illustration to clarify some possible scenarios asked during the exercises:

------(a)---->-----(b)-----(X)-----(c)------

">" is the head, moving towards right to process an existing request "X". "a","b", and "c" are potential spots for new requests to arrive. Following is some potential scenarios:

- A new request arrives at "a", i.e., it arrives at a position that the head already passed. Then this request has to wait until head processes all the request in its current direction and then come back.
- A new request arrives at "b", i.e., it arrives at a position that head is approaching but not passed yet (you have to do the math). Then head stops here and processes the new request, and then proceeds.
- A new request arrives at "c", i.e., it arrives at a position further than head is moving towards (X). Then this new request is scheduled to be processed after the first (X) one, as it is in the same direction.

If we ask a question about this algorithm, you will find all the details and assumptions in order to leave no room for confusion (e.g. start/stop time combined or separate).

If you have any questions, you can either ask here in replies or in-person next week (in Caching exercises).



# 6 Disk Crashes (Checksums, Stable Storage, RAID)


![[Pasted image 20251108232741.png]]

![[Pasted image 20251108232914.png]]


![[Pasted image 20251108232924.png]]

![[Pasted image 20251108232931.png]]



# 7 RAID


• Redundant Arrays of Inexpensive Disks
• Commodity hardware synthesized with software
• Looks externally like a single disk
• Data is stored in various ways ➜ RAID levels

In this lecture:
• Level 0: Block-level striping
• Level 1: Mirroring, Shadowing, or Duplexing
• Level 1+0: Striped Mirror
• Level 4: Block-level striping with parity disk
• Level 5: Block-level striping with distributed parity
• Level 6: Block-level striping with two distributed parity

![[Pasted image 20251108233229.png]]


![[Pasted image 20251108233237.png]]


![[Pasted image 20251108233244.png]]


![[Pasted image 20251108233251.png]]


![[Pasted image 20251108233258.png]]


![[Pasted image 20251108233306.png]]



![[Pasted image 20251108233318.png]]



