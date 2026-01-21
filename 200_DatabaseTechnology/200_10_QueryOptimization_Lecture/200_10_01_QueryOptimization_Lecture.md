



![](image/Pasted%20image%2020260121153550.png)

# 1 Parsing and Preprocessing



![](image/Pasted%20image%2020260121153647.png)

![](image/Pasted%20image%2020260121153722.png)


![](image/Pasted%20image%2020260121154031.png)


# 2 Logical Query Plan Selection 

Improve the logical query plan according to algebraic laws  “Pre-Optimization” (Heuristics)
- Push down selections (as early as possible)
- Push down projections (as early as possible) 
- Adjust duplicate elimination  
- Combine selection and cross product to join  
- Group set and join operators
—> Improved logical query plan

![](image/Pasted%20image%2020260121155908.png)

---

Transformation Rules (Algebraic Laws)

Transforming the internal presentation
- Without changing the semantics of the query
- For more efficient execution, particularly smaller intermediate results


Equivalent terms
- Two terms of the relational algebra are equivalent, if and only if:
    - They have the same operands (= Relations)
    - Always produce the same result relation


---

Commutativity and Associativity

![](image/Pasted%20image%2020260121160130.png)


---

![](image/Pasted%20image%2020260121160157.png)

---

![](image/Pasted%20image%2020260121160223.png)

![](image/Pasted%20image%2020260121160258.png)


# 3 Physical Query Plan Selection

From Logical to Physical Query Plans
Choices for physical query plans
- Order and grouping of associative and commutative operators,  e.g., join, union, intersection, selection
- Choice of physical implementation for each operator, e.g., join => nested loop join, merge join, hash join, index join
- Additional operators, which do not occur in the logical plan,  e.g., (table or index) scan, intermediate sort
- Mode of data transport between operators, e.g., materialize intermediate results, pipeline with iterators
Needed for the translation: Cost estimation for physical operators


Cost-Based Optimization/Enumeration
![](image/Pasted%20image%2020260121160921.png)


Conceptually: Generate many or all possible physical query plans 
Evaluate the costs of each physical plan with respect to a cost model: 
Logical (intermediate results) vs. physical cost model (e.g., number of I/Os)

Sizes of intermediate relations • • Use statistics

Used implementation (algorithmic costs)

Calibrated to particular computers,  e.g., sequential vs. random access

Optimization metric

• •

Maximize throughput Minimize response time

## 3.1 Estimating Sizes of Intermediate Relations

![](image/Pasted%20image%2020260121161508.png)

### 3.1.1 Cost Estimation: Projection (Bag Semantic) (select xx attribute)


![](image/Pasted%20image%2020260121161857.png)

### 3.1.2 Cost estimation: Selection (where )
![](image/Pasted%20image%2020260121161925.png)

![](image/Pasted%20image%2020260121162133.png)


![](image/Pasted%20image%2020260121162331.png)

![](image/Pasted%20image%2020260121162340.png)

![](image/Pasted%20image%2020260121162348.png)


### 3.1.3 Cost Estimation: Join 

![](image/Pasted%20image%2020260121162457.png)

![](image/Pasted%20image%2020260121162504.png)

![](image/Pasted%20image%2020260121162607.png)

![](image/Pasted%20image%2020260121162621.png)

![](image/Pasted%20image%2020260121162628.png)

### 3.1.4 Cost Estimation: Multiple Joins

![](image/Pasted%20image%2020260121162644.png)

![](image/Pasted%20image%2020260121162735.png)


## 3.2 Obtaining Estimates for Size Parameters

![](image/Pasted%20image%2020260121163044.png)



![](image/Pasted%20image%2020260121163232.png)

![](image/Pasted%20image%2020260121163249.png)


![](image/Pasted%20image%2020260121163257.png)

Considerations for Collecting Statistics
(Precise and incremental statistic collection possible but usually not done)
• Statistics are collected and maintained periodically
    • Statistics do not change radically in a short time
    • Even slightly inaccurate statistics are useful
    • Keeping statistics up-to-date can make them a hot-spot


Triggering statistics collection
- After a fixed period of time
- After a fixed number of updates、
- By a database administrator or user


Calculating statistics is costly ： Solution: Sampling


## 3.3 Approaches to Enumerate Physical Plans 

- Complete Enumeration (exhaustive search)
    - Considering all choices for physical query plans
        - Order and grouping of associative and commutative operators
        - Choice of physical implementation for each operator 
        - Additional operators, which do not occur in the logical plan 
        - Mode of data transport between operators
    - Cost calculation for every plan
    - Select the plan with the least possible costs 
    - Problem: Too many plans (see join ordering)

There are several faster methods
Heuristics (e.g., greedy search, hill climbing), branch-and-bound, dynamic programming, Selinger-Style

![](image/Pasted%20image%2020260121164156.png)


---

![](image/Pasted%20image%2020260121164557.png)


•
• • •
•
Using Heuristics
Idea: Use heuristics, which try to find good solutions quickly  e.g., greedy search for join ordering
First determine join pair with the smallest intermediate result
Then join with the relation that in turn produces the least intermediate result And so on ...
Further heuristics
• • • • •
Use index-scan when selection on indexed attribute
Apply multiple selections on same relation at the same time Use index-join if there is an index on the join attribute
Use sort-merge-join if one relation is sorted on the join attribute Compute union and intersection for smallest relations first



---

![](image/Pasted%20image%2020260121164924.png)



---

![](image/Pasted%20image%2020260121164953.png)

•
Idea: Bottom-up approach for building operator trees
•
•
Keep best partial plan for each subexpression; 
use best partial plans to build more complex partial plan
Dynamic Programming
•
Idea: Enhancement of dynamic programming
• •
Do not only memorize the best plan
• Also keep track of miscellaneous sorting variants (interesting orders)
Selinger-Style
• • May cost more but can have benefits later
Does not affect cardinalities of intermediate results but I/O costs


### 3.3.1 Join Ordering - Dynamic Programming


Join is usually most expensive operator
Here: Only join order, but parallelization optimization may be important in practice! Order and tree shape matters!
Many algorithms
• • •
•
Greedy search
•
Dynamic programming

![](image/Pasted%20image%2020260121165130.png)


---


![](image/Pasted%20image%2020260121165254.png)


![](image/Pasted%20image%2020260121165350.png)

---

![](image/Pasted%20image%2020260121170000.png)

![](image/Pasted%20image%2020260121170130.png)


![](image/Pasted%20image%2020260121170138.png)


![](image/Pasted%20image%2020260121170201.png)


### 3.3.2 Dynamic Programming — Example

![](image/Pasted%20image%2020260121170228.png)

![](image/Pasted%20image%2020260121170331.png)

![](image/Pasted%20image%2020260121170435.png)

### 3.3.3 Dynamic Programming — Interesting Orders

![](image/Pasted%20image%2020260121170801.png)

•
•
When choosing the best partial plan:
• •
Cost comparison is not sufficient Sort orders must be considered
Solution:
•
Save several "interesting sort orders" for each combination of relations: • Interesting orders for later plans
• Also keep an unsorted plan
• • Dynamic programming tables become "wider" Also: 
Memorize best join and sort operations which produce the order


![](image/Pasted%20image%2020260121170840.png)


## 3.4 Completing the Physical Query Plan

• Choosing a physical implementation
    • If not already done (e.g., with dynamic programming) 
    • Examples: Selection and Join
• Pipelining vs. materialization of intermediate results
- Access path (access method) for each table

Choice of Physical Operators
![](image/Pasted%20image%2020260121163959.png)

•
•
•
How should joins be executed?
• • •
How should relations be accessed?
• • •
Table Scan
Clustered Index Scan Secondary Index Scan(s)
Hash Join
(Sort) Merge Join
(Index) Nested Loop Join
Answer depends on:
• • •
Size of operator input Amount of available memory Data properties
• Is data already sorted?
• • Will sorted data be helpful l


