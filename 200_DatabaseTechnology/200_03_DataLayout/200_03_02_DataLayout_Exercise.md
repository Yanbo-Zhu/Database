
# 1 Recap

What is the primary storage location for the DBMSs data, and why?

What is the primary optimization goal in DBMS implementation?

1. How does a DBMS actually organize its data on disk?
    
2. What are the atomic units of data that are read/written?

# 2 Pages and Tuples


## 2.1 Table

What types of data can we distinguish for attributes?

How do traditional DBMSs store these relations?

![[Pasted image 20251108225706.png]]


## 2.2 Why not just store them like this?

`| [Tuple1] [Tuple2] [Tuple3] [Tuple4] … |`

- What happens with variable-sized data?
    
- What happens on an update like this?

```
UPDATE articles
SET title = 'Basics of SQL'
WHERE article_id = 103; 
```


# 3 How is a single tuple/row laid out in memory?







# 4 Alignment and Addressing


```
CREATE TABLE Student (
  Birthday DATE.
  ImmatrNumber INT, 
  Name CHAR(30), 
  Lastname CHAR(30)
  );
```

1. What is the size of an individual Record?
    
2. How many records fit on a Page with 4096 bytes?
    
3. What is the offest of the 33th tuple (assuming pages are filled in reverse order)

![[Pasted image 20251108225848.png]]


## 4.1 ##

What about variable size data?

```
CREATE TABLE Student (
  Birthday DATE.
  ImmatrNumber INT, 
  Name VARCHAR(200), 
  Lastname VARCHAR(200)
);
```

![[Pasted image 20251108225914.png]]
- What is the fixed size of an individual record?
- What is the average total size of a record, assuming VARCHARs are using half of the memory on average?
- How many average records fit into a page with 4096?


## 4.2 ##

Can we store each attribute at an arbitrary offset?
Looking at the following schema:
![[Pasted image 20251108230021.png]]

Can we align this differently to reduce the tuple size (CPU requires 4-byte alignment)?
How is memory addressed by the CPU - and how do we address a tuple on disk?


# 5 Storage Models

What is the difference between NSM, DSM and PAX storage models?

Calculate the number of pages accessed for both NSM and DSM:

![[Pasted image 20251108230130.png]]

- Each attribute consumes **4 bytes**.    
- Page size is **32 bytes**.

```
INSERT INTO R VALUES(9, 9, 330, '21/02/2022')
```

Given this query, what is the problem with pure DSM, and does PAX help here?

- Need to create 4 new pages, one for each column.
    
- PAX stores columns adjacent to each other within a page.
    
- In our example, the disk layout would be` [1 2 2 4 50 30 ‘12/05/2020’ ‘21/04/2021’]`
    
- With PAX, we would only need to read/write to a single page, where we create 4 new column chunks

