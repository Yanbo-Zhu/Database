
# 1 Data Model 

Relational Data
Graph Data
Spatial Data
Temporal Data


# 2 OLTP and OLAP 

# 3 Accessing Stored Data


A Disk only has one dimension 

either, the block size was very large  
or, we did not care about the block size


# 4 Indexing and Partitioning

Strategies for arranging multi-dimensional Data on Disk


data can be accessed efficiently  
data is partitioned into several subregions

![](image/Pasted%20image%2020260119221947.png)


# 5 Object Storage

Nowadays, when storing large amounts of data, we use

Object Storage  
Object storage has higher latencies

data is accessed over the network

→ Disk Latency is less important  

We should not store all data in a single Bucket Therefore, we need to partition the data
But: We do not have a hard block size anymore


# 6 Dimensional Data

![](image/Pasted%20image%2020260119222054.png)

- Point Queries
- Orthogonal Range / Box / Window Queries
- Distance Queries
- K-Nearest-Neighbour Queries
- Join Queries
![](image/Pasted%20image%2020260119222614.png)

![](image/Pasted%20image%2020260119222620.png)


# 7 Queries in Data Warehouses

The same problem also exists with other rather independent variables, like those in a Data Warehouse

![](image/Pasted%20image%2020260119222806.png)


![](image/Pasted%20image%2020260119222815.png)


# 8 Using Multidimensional Indices

Group Task

With your neighbour(s): What possible ways can you imagine giving an order to a two- dimensional space. Which trade-offs are important for answering this question?

Key Considerations  
Are my dimensions qualitative or quantitative?  
Do I partition by data or by space?  
Do my queries query correlated datasets?  
Do my queries have a special structure  
how should my represented area be shaped? Are separate-connected areas fine?

![](image/Pasted%20image%2020260119222922.png)

## 8.1 Hash-based: Grid File

![](image/Pasted%20image%2020260119223032.png)

## 8.2 Tree-Based: Partitioned Hashing

![](image/Pasted%20image%2020260119223107.png)


![](image/Pasted%20image%2020260119223116.png)


## 8.3 Tree-based:  Multiple Keys

![](image/Pasted%20image%2020260119223250.png)


## 8.4 Tree-based:  Kd-Tree


![](image/Pasted%20image%2020260119223326.png)



## 8.5 Tree-based:  Quad-Tree

![](image/Pasted%20image%2020260119223356.png)



## 8.6 Tree-based:  R-Tree

![](image/Pasted%20image%2020260119223416.png)

