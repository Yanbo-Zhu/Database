

# 1 Cache 发生在哪里 

OS can do the caching totally transparent for you but DBMS’s want to have full access, e.g., when to load and when to evict.
OS Caching highly depends on the underlying kernel/ implementation and is not always comprehensible.

![[Pasted image 20251109120335.png]]



Responsibilities:
• Provide access to blocks on disk to database processes
• Read blocks into main memory
• Cache blocks for faster access
• The DBMS is responsible for it
• OS allocates it in virtual memory (e.g., malloc)


Architecture
• The DBMS is responsible for it
• OS allocates it in virtual memory


# 2 Buffer Manager


![[Pasted image 20251109121130.png]]



![[Pasted image 20251109120539.png]]



Benefits of Caching

![[Pasted image 20251109120633.png]]


# 3 Buffer Management Strategies


## 3.1 Buffer Manager Scope
What is cached by the buffer manager?
• Table pages
• Index pages
• Metadata pages

Note: The system requires further memory for query processing,
e.g, for building a hash table or sorting

## 3.2 Granularity of Buffers

• Records
- Not used because replacement and reloading too costly
• Blocks/pages
- Commonly used
- Either OS blocks or database blocks
• Chunks
- Combine blocks into larger “chunks”
- Can exploit sequentially placed blocks on disk
- Good for very large operations (large table joins or sorts)
• Whole Tables
- Fix all blocks of heavily used tables


## 3.3 Pinning

• Blocks/pages in use are “pinned” and forbidden to be evicted
• Buffered blocks can be pinned by multiple execution units
## 3.4 Eviction Policy

Eviction Policy - Things to Consider
• Age
- Time since block was loaded
- Last time accessed
• Accesses
- Number of accesses
• Trade-offs
- New blocks → low access counts, but involved in current operations
- Old blocks → high access counts, constant but potentially less frequent use
Mix of age and accesses Other factors: Pinned blocks, operation access pattern, overhead

Eviction Policies
Common strategies, also in other applications (e.g., operating system); Simple, low-overhead strategies are surprisingly good:
• First-In-First-Out (FIFO): Eviction based on position
• Least Recently Used (LRU): Eviction based on time
• Least Frequently Used (LFU): Eviction based on usage
• CLOCK (sometimes also called second-chance): commonly implemented, efficient approximation to LRU


### 3.4.1 
First In First Out
• Easy to implement; low overhead
• Does not consider access recency or frequency

Least Recently Used
• Considers access recency (which is intuitive)
• Requires maintenance of the access recency order (i.e., for every access)


Least Recently Used
• Good for temporal locality
• Poor for large sequential scans (cache pollution)
Good if you access data in a burst and then forget it like in this query plans

Most Recently Used
• Good for sequential and repeating patterns (e.g., table scans)
• Poor for hot-spot reuse
Good if parts are often accessed over and over again like the root of a tree

![[Pasted image 20251109121437.png]]


![[Pasted image 20251109121445.png]]


### 3.4.2 CLOCK also called Second Chance

General Idea: Improvement over FIFO; approximate LRU with less overhead. Take the accesses to a page between two points in time into account. Reduce the risk of evicting a page that is in use.

Algorithm
- Pointer to page in ring buffer
- Each buffer has a reference bit
- Insert new page if buffer is empty => Store at pointer
position, set reference bit to 1
- When referencing a cached page: (no pointer movement)
- If 0 => Set reference bit to 1
- If 1 => Keep it to 1
- Evict old page:
1. Rotate pointer clockwise through ring buffer
2. If page bit is 1 then set to 0 and continue rotation
3. If reference bit is 0, then evict page and load new page into buffer (no pointer movement after loading)

![[Pasted image 20251109121600.png]]

![[Pasted image 20251109121611.png]]

![[Pasted image 20251109121617.png]]

![[Pasted image 20251109121623.png]]


![[Pasted image 20251109121635.png]]




## 3.5 Prefetching

Prefetching Pages
• Load blocks not yet needed now, but hopefully soon
• Examples:
- If disk sector is requested, read entire track
- Read from other cylinders without moving head (data is stored
usually cylinder by cylinder)
- If a block from relation is requested, load next few blocks
• Using sequential and asynchronous, non-blocking reads, prefetching
often cost little and can save a lot of time



Semantic Caching
• Cache query result
- Q1: SELECT * FROM person WHERE birth_year > 1980
- Q2: SELECT name FROM person WHERE birth_year > 1990
- Q2 can be answered using result tuples from Q1
• Powerful but challenging technique
- When can we use results of one or more other queries?
- Query containment, “answering queries using views”
• Semantic caching is not used by any commercial database today
- Note: Normal caching can mimic semantic caching
- If Q2 executed after Q1, blocks from Q1 are in cache (“hot” cache)
- But: Computations need to be repeated (e.g., aggregation)

## 3.6 Extended Approaches

