

# 1 What is index and why 

each index is for one field/  attribute 


# 2 Index is stored on disk in files / pages like tuples 


in disk 因为不容易丢失 

Size： indexes can reach TBs for large tables
Recovery: would need to scan all data to rebuild index 
Colocation: primary / clustered index stored together with the actual data 





# 3 How does an index locate individual rows 



# 4 Type of Index 

不同 catagory 
primary / clustered / non Cluster index 
sparse / dense Index 
Hash Index 

sorted/ordered index

- Based on arrangement / lookup mechanism
	- Ordering: Based on a sorted ordering of values 
	- Hashing based on a distribution of keys to hash values, determined by a hash function
- Based on number of entries
	- Dense Index : for each row , on index 
	- Sparse index: only store the subset of key
		- Block consist of pages.  not restricts to pages
		- binary search in block to identity the location of the data you search for . 
		- In Sparce Index, we need to search in memory
- Based on value arrangement
	- Clustered Index (data sorted by year)
	- accesse the data sequenctielly , because they are save  in order 






# 5 Can a hash index be sparse 

in ordered data

Dense index : 1 2 2 4 5 6 6 
Sparse index:  1     4    6 ,  from index , we know 5 ist bewteen 4, 5

hash index 
0 -> (1, 5)    我们可以直接 定位到 5 
1 -> 
2 -> 

you can bot tell the hash function to be ordered, 因为 hash function 计算出来的结果 就是无序的



# 6 Cluster and uncluster Index 

You can only habe one cluster index. 因为 cluster index hast to be ordered and your data can be order by sole attribute 
cluster index 中 必须是有序存入的 ， 比如说  year 这个 attribute，  在建立一个 cluster  index  的时候， 这个 cluster index 中的数据都是 ordered by year 


## 6.1 Can we have multiple clustered indexes for a single relation 

Solution: 
 -

# 7 When should we use indices

Solution:
 - For highly selective queries
	 - Highly selective: query 



# 8 Space of Dense vs Sparse Index 


Given:
- Tuple: 150M 

how much space would a dense or sparse index need? 

IndexTupleSize:  SearchKeySize + RecordPointerSIze 


# 9 

numIndexTuples:   `4.6875*(10^6)`

We assume we only have one index entry for each page 

to caculate the amount of pages to store the index 


# 10 Binary Search Dense vs Sparse Index 



It is not end,   because if we find the right page , we also need to search xx in the page  inside 



# 11 Multilevel Index on Dense Index : 


MultiLevel Index:   index of index 

Sparse index -> Dense Index 





















