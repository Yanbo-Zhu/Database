
# 1 Relational Algebra


Basic Query execution
![](image/Pasted%20image%2020260120170011.png)


---


![](image/Pasted%20image%2020260120170124.png)
•
Selection σ
• • •
Projection π Difference –
Relational Operators
5 base operators of the Relational Algebra (+ rename)
Union ∪
• • Cartesian Product X
•
Derived operators, e.g., • • Join ⋈ (X and σ)
• Intersect ∩ (𝑅∩𝑆=𝑅−(𝑅−𝑆)
Additional operators, e.g.,
• • aggregation function γ (SUM, AVG, MAX, etc.), grouping G
Translation from Bags to Sets (de-duplication)
•
Unique/distinct D

---

Logical Physical Mapping 

![](image/Pasted%20image%2020260120170258.png)

## 1.1 Pipelining vs. Blocking

‣ Every operator can be either pipelined or blocking
    ‣ Pipelining: The operator can produce the output tuples by just seeing an input tuple
        ‣ Example: Hash Probe, Selection
‣ Blocking: The operator has to see all its input tuples before the first output tuples can be generated
    ‣ Example: Sort, Grouping, Join
‣ Some operators have pipelinable and blocking sub-operators like joins:
    ‣ Blocking: Hash table build
    ‣ Pipelinable: Hash table probe

# 2 Processing Models

‣ A processing models determines how a DBMS executes a given query plan 
‣ Today's DBMS use combinations of the following techniques:
    ‣ Execution method: Interpretation-based approaches vs. Compilation- based approaches (analogous to programming languages)
    ‣ Intra-operator granularity: Granularity at which tuples are processed within an operator: e.g., tuple, vector, column, data centric, etc.
    ‣ Inter-operator execution: Policy to define the control flow between operations in a query plan (see next slide)
![](image/Pasted%20image%2020260120171028.png)


## 2.1 Interpretation-based Approaches 

‣ Interpretation-based approaches "interpret" the query plan during runtime
‣ Query evaluation starts from the root in a pull-based model
‣ Each operator pulls the next tuple up, also called pull-based query execution
‣ The following models were proposed (in chronological order) with intra- operator granularities:
    ‣ Iterator model (volcano, tuple-at-a-time) Book stops here 
    ‣ Materialization model (operator-at-a-time, column-at-a-time)
    ‣ Vectorization model (vector-at-a-time, batch, block-wise)


### 2.1.1 Comparison of Processing Models

![](image/Pasted%20image%2020260120171211.png)

## 2.2 Compilation-based Approaches 



## 2.3 Comparison of Approaches

# 3 Parallel Execution