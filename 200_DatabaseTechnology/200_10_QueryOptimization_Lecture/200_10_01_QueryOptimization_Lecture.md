

![](image/Pasted%20image%2020260123233516.png)

![](image/Pasted%20image%2020260121153550.png)

# 1 Parsing and Preprocessing


![](image/Pasted%20image%2020260121153647.png)

![](image/Pasted%20image%2020260121153722.png)


Preprocessing
- Translate parse tree into a query expression tree (extended raletional algebra)
- Check conrretness of semantics 
- Resolve views and subqueries 
- initial logical query plan 

![](image/Pasted%20image%2020260121154031.png)


# 2 Logical Query Plan Selection 

Improve the logical query plan according to algebraic laws  “Pre-Optimization” (Heuristics)
- Push down selections (as early as possible)  where 
- Push down projections (as early as possible)   select
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

物理查询计划的选择：
可交换和可结合操作符的顺序和分组，例如：连接、并集、交集、选择
每个操作符的物理实现选择，例如：连接操作 → 嵌套循环连接、归并连接、哈希连接、索引连接
逻辑计划中不存在的额外操作符，例如：（表或索引）扫描、中间排序
操作符之间的数据传输方式，例如：物化中间结果、使用迭代器进行流水线处理

转换所需的条件：物理操作符的成本估算



Cost-Based Optimization/Enumeration
![](image/Pasted%20image%2020260121160921.png)


Conceptually: Generate many or all possible physical query plans 
Evaluate the costs of each physical plan with respect to a cost model:  Logical (intermediate results) vs. physical cost model (e.g., number of I/Os)
- Sizes of intermediate relations 
    - Use statistics
- Used implementation (algorithmic costs)
- Underlying hardware 
    - Calibrated to particular computers,  e.g., sequential vs. random access
- Optimization metric
    - Maximize throughput 
    - Minimize response time

## 3.1 Estimating Sizes of Intermediate Relations

![](image/Pasted%20image%2020260121161508.png)


- Substantial component of operator cost: Input and output cardinality (Output is input of the next operator)
    + Tuple size/width (in bytes)
+ Particularly: Does the data fit into main memory?
+ Hence: Cost model must estimate/predict the output cardinality (i.e., number of records produced by each operator)
    + • "Selectivity" of an operator with respect to its input
    + • Output cardinality = input cardinality * selectivity
    + • Also known as "filter factor" (selectivity factor)


- 操作符成本的重要组成部分：**输入和输出的基数**（输出是下一个操作符的输入）
    - **元组大小/宽度**（以字节为单位）
    - 特别重要的是：**数据是否能放入主内存？**
- 因此：**成本模型必须估计/预测输出基数**（即每个操作符产生的记录数）
    - 操作符相对于其输入的**"选择性"**
    - **输出基数 = 输入基数 × 选择性**
    - 也称为 **"过滤因子"**（选择率）


**补充说明：**

**为什么基数估计如此重要？**
1. **影响后续操作的成本**：
   - 如果一个操作输出很多行，下一个操作就要处理更多数据
   - 例如：过滤操作的选择性高（输出少）可以大大降低后续连接的成本

2. **决定内存使用**：
   - 估计输出大小可以判断是否需要物化到磁盘
   - 例如：哈希连接需要估计构建表是否能放入内存

3. **影响操作符选择**：
   - 如果估计输出很小，可以用嵌套循环连接
   - 如果估计输出很大，可能需要用哈希连接或归并连接

**选择性的例子：**
- 等值条件 `age = 30`：选择性 ≈ 1 / 不同值的数量
- 范围条件 `age > 30`：选择性 ≈ (max - 30) / (max - min)
- 连接条件 `T1.id = T2.id`：选择性 ≈ 1 / max(不同值的数量)

**常见估计方法：**
- 基于统计信息（直方图、不同值数量）
- 基于采样
- 基于假设（如均匀分布）

需要我解释一下**如何用直方图估计选择性**吗？


### 3.1.1 Cost Estimation: Projection (Bag Semantic) (select xx attribute)


![](image/Pasted%20image%2020260121161857.png)

### 3.1.2 Cost estimation: Selection (where )
![](image/Pasted%20image%2020260121161925.png)

selection 不相等
T(R)x  (1 -  1/V(R,V） )


---

小于x  等于 选1/3 个 原来的 tuples 

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


General case: S = R1 ⋈ R2 ⋈ … ⋈ Rn
• Notation: V(Ri, A) = vi
• Attribute A appears in k relations
• Let v1 ≤ v2 ≤ … ≤ vk
• Given: 1 tuple of each relation
• Need to compute: Probability that they have the same A value
    • Inclusion assumption: Each A-Value of tuples of R1 occurs in the other relations
    • The probability of one tuple of Ri matching a g
    • Together 1/(v2 v3 ... vk)

Overall approach:
• Start with product of all cardinalities
• and apply join selectivities: For each attribute that occurs more than once: Divide by product of all vi except for smallest (v1)

![](image/Pasted%20image%2020260121162735.png)


## 3.2 Obtaining Estimates for Size Parameters

![](image/Pasted%20image%2020260121163044.png)


Statistics necessary to calculate the size of intermediate results (cardinality)
• T(R) and V(R,A) are particularly important
    • T(R) estimation stored in statistics; could be computed after scanning R
    • V(R,A) estimation stored in statistics; for each (single) attribute
• B(R)
    • Estimation stored in statistics
    • Calculating/estimating (T(R) / (max) tuples per block)


Notation
B(R) number of blocks for relation R
T(R) number of tuples of relation R
V(R,A) distinct values of attribute A in relation R


![](image/Pasted%20image%2020260121163232.png)

![](image/Pasted%20image%2020260121163249.png)


Further statistics used in practice:
• Most common values (MCVs) and frequencies
• NULL value freq
• Extended statistics on demand (functional dependencies, multi-attribute distinct values and MCVs)
• For read-only data: small materialized aggregates, e.g., min, max (per partition)


----

![](image/Pasted%20image%2020260121163257.png)


• Histograms represent value distributions efficiently

Idea:
• Collect groups of values (ordered ranges) in buckets
• Equi-width, equal-depth
• Assumes uniform distribution within bucket

Advantages
• Fewer estimation errors (uniform ONLY within bucket)
• Low memory consumption (single value for range/group


-----

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


统计信息是定期收集和维护的
统计信息不会在短时间内发生剧烈变化
即使稍微不准确的统计信息也是有价值的
保持统计信息的实时更新可能会使其成为热点

触发统计信息收集的时机：
经过固定的时间段后
经过固定数量的更新后
由数据库管理员或用户手动触发


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

- **考虑物理查询计划的所有选择**
    - 可交换和可结合操作符的顺序和分组
    - 每个操作符的物理实现选择
    - 逻辑计划中不存在的额外操作符
    - 操作符之间的数据传输方式
- 计算**每个计划的成本**
- 选择**成本最低的计划**
- **问题：计划数量太多**（参见连接顺序问题）


--- 

There are several faster methods
Heuristics (e.g., greedy search, hill climbing), 
branch-and-bound, 
dynamic programming, 
Selinger-Style

![](image/Pasted%20image%2020260121164156.png)


### 3.3.1 Heuristics (greedy search)

![](image/Pasted%20image%2020260121164557.png)


Using Heuristics
Idea: Use heuristics, which try to find good solutions quickly  e.g., greedy search for join ordering
First determine join pair with the smallest intermediate result
Then join with the relation that in turn produces the least intermediate result And so on ...

Further heuristics
Use index-scan when selection on indexed attribute
Apply multiple selections on same relation at the same time 
Use index-join if there is an index on the join attribute
Use sort-merge-join if one relation is sorted on the join attribute 
Compute union and intersection for smallest relations first


---

**核心思想：** 使用启发式规则，尝试快速找到好的解决方案  
例如：连接顺序的**贪心搜索**

- 首先确定产生**最小中间结果**的连接对
- 然后与产生**次小中间结果**的关系进行连接
- 依此类推...

**其他启发式规则：**
- 如果选择条件在索引属性上，**使用索引扫描**
- 对同一关系的多个选择操作**同时应用**
- 如果连接属性上有索引，**使用索引连接**
- 如果一个关系在连接属性上已排序，**使用排序归并连接**
- 对于并集和交集操作，**先计算最小的关系**

---

**补充说明：**

**贪心连接顺序的缺点：**
- 可能找不到全局最优解
- 只考虑当前最优，忽略未来影响
- 但通常能快速得到"足够好"的计划

**启发式规则的依据：**
- **尽早过滤数据**（选择下推）
- **利用已有结构**（索引、排序）
- **减少中间结果大小**（先连接小表）

**实际应用：**
- 许多数据库先应用启发式规则生成初始计划
- 然后在此基础上进行有限的成本估算优化
- 或者用启发式规则剪枝搜索空间

**例子：**
```
T1(100行) JOIN T2(10000行) JOIN T3(1000000行)
启发式：先连接 T1 和 T2（假设产生 100 行）
再连接 T3（最终 100 行）
避免先连接 T2 和 T3（产生 10000 行）
```

需要我进一步解释**选择下推**和**投影下推**等常见的启发式优化规则吗？


### 3.3.2 branch-and-bound and hill climbing

![](image/Pasted%20image%2020260121164924.png)



- **分支定界**：像一个聪明的旅行推销员，先估算一个最短路径的上限，然后系统性地尝试所有路线，一旦某条路线部分已经超过上限就放弃，最终找到最短路径。
- **爬山法**：像一个登山者，只往更高的方向走，直到山顶。但可能错过更高的山峰，因为当前所在的山不是最高的。

**总结**：
- **分支定界**：追求**全局最优**，但可能慢
- **爬山法**：追求**快速找到好解**，但不保证最优




Branch and Bound 分支定界
• Idea: Use heuristics to find a good plan
    • Costs of that plan establish an upper bound (even for partial plans)
    • Enumerate plans for various parts of query
        • Ignore partial plans whose costs are higher than the upper bound
    • Lower the upper bound if a better complete plan is found
• Advantage: Optimization can be aborted anytime

- **核心思想：** 使用启发式方法找到一个好的计划
    - 该计划的成本建立一个**上界**（即使对于部分计划也适用）
    - 枚举查询各个部分的计划
        - **忽略成本高于上界的部分计划**
    - 如果找到更好的完整计划，**降低上界**
- **优点：** 可以**随时中止**优化过程


1. **先用启发式**找一个不错的计划，得到**上界** `U`
2. **枚举部分计划**，计算其成本
3. 如果部分计划的成本已经 > `U`，**直接剪枝**（不继续扩展）
4. 如果找到更好的完整计划，**更新 `U`**
5. 继续枚举，直到所有可能被考虑或剪枝完毕

**特点**：  
- 保证找到最优解（如果枚举完）
- 可以随时停止，得到当前最好解
- 适合中等规模的搜索空间

-----

Hill-Climbing 爬山法
• Idea: Use heuristics to find a good plan
    • Find similar plans with lower costs step by step
        • Similar: Only one change (operator, order, etc.)
• If no similar plans is better: terminate
• Disadvantage: Local optimum vs. Global optimum
• Variants for improvement:
    • Iterative Improvement: Start with 10 different starting plans
        • Simulated Annealing: Also allow deterioration


- **核心思想：** 使用启发式方法找到一个好的计划
    - **逐步寻找成本更低的相似计划**
        - 相似：只改变一个方面（操作符、顺序等）
    - 如果没有更好的相似计划：**终止**
- **缺点：** 可能陷入**局部最优**而非**全局最优**
- **改进变体：**
    - **迭代改进：** 从 10 个不同的起始计划开始
    - **模拟退火：** 也允许接受更差的计划（以跳出局部最优）


1. **随机或启发式**选择一个起始计划
2. **考察邻居计划**（只改变一个操作符、顺序等）
3. 如果邻居中有更优的，**移动到那个邻居**
4. 重复直到没有更好的邻居
5. 可能从多个起点重新开始（迭代改进）

**特点**：  
- 速度快，但可能陷入局部最优
- 不保证最优解
- 适合超大搜索空间，无法穷举的场景

---

**补充说明：**

**分支定界的优点：**
- 保证找到最优解（如果枚举完整）
- 可以通过上界剪枝大量无效搜索
- 可随时停止，得到当前最好计划

**爬山法的局限：**
- 容易陷入局部最优
- 对起始点敏感
- 但执行速度快

**改进策略：**
- **迭代改进**：多个随机起点，取最优
- **模拟退火**：以一定概率接受更差解，避免局部最优
- **遗传算法**：种群进化搜索

**实际应用：**
- 商业数据库常结合多种方法
- 先启发式生成初始计划
- 再用分支定界或动态规划精细优化
- 复杂查询可能用随机搜索


---


| 特性       | 分支定界            | 爬山法           |
| -------- | --------------- | ------------- |
| **搜索方式** | 系统性枚举 + 剪枝      | 局部搜索 + 迭代改进   |
| **最优性**  | 可保证全局最优（如果枚举完整） | 可能陷入局部最优      |
| **搜索方向** | 全局搜索，但用上界剪枝     | 只在当前解的"邻居"中搜索 |
| **终止条件** | 枚举完所有可能或用户中止    | 找不到更好的邻居      |
| **中间结果** | 维护全局上界          | 只维护当前解        |


| 方面 | 分支定界 | 爬山法 |
|------|----------|--------|
| **解的质量** | 高（可最优） | 中低（局部最优） |
| **搜索时间** | 可能很长（指数级） | 短（线性级） |
| **内存使用** | 需要维护搜索树和上界 | 只需维护当前解 |
| **可中断性** | 可随时中断，得到当前最好 | 只能得到最终解 |
| **适用场景** | 中等规模连接查询 | 极大规模连接查询 |

---

许多商业数据库**结合两者**：
1. **先用爬山法**快速得到一个"足够好"的计划
2. **用这个计划的成本作为上界**，启动分支定界
3. 在分支定界中，如果找到更好的计划，更新上界
4. 如果时间有限，可以随时返回当前最好计划

### 3.3.3 Dynamic Programming and Selinger Style 

![](image/Pasted%20image%2020260121164953.png)

Dynamic Programming 
Idea: Bottom-up approach for building operator trees
Keep best partial plan for each subexpression;  use best partial plans to build more complex partial plan

**核心思想：** 自底向上构建操作符树
- 为每个子表达式**保留最优的部分计划**
- 使用这些最优部分计划来构建更复杂的部分计划


---

Selinger-Style
Idea: Enhancement of dynamic programming
Do not only memorize the best plan
    • Also keep track of miscellaneous sorting variants (interesting orders)
    • May cost more but can have benefits later
Does not affect cardinalities of intermediate results but I/O costs


**核心思想：** 对动态规划的增强
- **不仅记住最优计划**
    - 还跟踪各种排序变体（**感兴趣的顺序**）
    - 虽然可能成本更高，但**后续可能带来好处**
- 不影响中间结果的基数，但影响 **I/O 成本**

---

**补充说明：**

**动态规划的优势：**
- 利用**最优子结构**：最优计划由最优子计划组成
- 避免重复计算相同子表达式的成本
- 典型应用：System R 优化器

**Selinger 风格的改进：**
1. **感兴趣的顺序（Interesting Orders）**
   - 例如：一个子计划可能产生按 `id` 排序的结果
   - 虽然这个排序本身有成本，但后续的归并连接可以直接利用
   - 优化器会保留多个排序变体，不只是成本最低的

2. **例子：**
   ```
   子计划 A：扫描 T1，成本 100，结果无序
   子计划 B：扫描 T1 + 排序，成本 150，结果按 id 排序
   
   如果后续有归并连接需要按 id 排序，
   选择 B 可能整体成本更低
   ```

3. **不影响基数但影响 I/O：**
   - 排序变体不影响结果的行数
   - 但影响如何读取数据（顺序 vs 随机 I/O）
   - 也影响后续操作能否流水线执行

**System R 优化器的影响：**
- 现代数据库优化器大多基于这个框架
- PostgreSQL、DB2、Oracle 等都采用类似方法

### 3.3.4 Dynamic Programming


Join is usually most expensive operator
Here: Only join order, but parallelization optimization may be important in practice! 
Order and tree shape matters!

Many algorithms
Greedy search
Dynamic programming

![](image/Pasted%20image%2020260121165130.png)


---


![](image/Pasted%20image%2020260121165254.png)

Idea of Dynamic Programming
– Here: "Backward Calculation"

Precondition: Principle of optimality
– Partial plan of an optimal plan is also optimal

Idea
– Based on the target node backward oriented , stepwise calculation of best partial paths
– F(X) := Minimal costs from X to J


![](image/Pasted%20image%2020260121165350.png)

---

![](image/Pasted%20image%2020260121170000.png)


• Optimal algorithm
• Requirements
    • Principle of optimality must apply
    • Breakdown of the problem into partial problems
• Complexity can be exponential
• Classic applications
    • Knapsack Problem
    • Traveling Salesman Problem
    • Machine Scheduling
    • Transportation Problem
• Now: Application to query planning


- **最优算法**
- **要求**
    - 必须满足**最优性原理**
    - 问题可以分解为**子问题**
- **复杂度可能是指数级的**
- **经典应用**
    - 背包问题
    - 旅行商问题
    - 机器调度
    - 运输问题
- **现在：应用于查询计划**


----


• Heuristic limitation of the search space
    • No cross products
        • Only if cross product is explicit part of query
    • Only left-deep trees

![](image/Pasted%20image%2020260121170130.png)


![](image/Pasted%20image%2020260121170138.png)


![](image/Pasted%20image%2020260121170201.png)


### 3.3.5 Dynamic Programming — Example

![](image/Pasted%20image%2020260121170228.png)

![](image/Pasted%20image%2020260121170331.png)

![](image/Pasted%20image%2020260121170435.png)

### 3.3.6 Dynamic Programming — Interesting Orders

• Repetition: Principle of optimality: When two plans differ in one partial plan, then the plan with the better partial plan is the better one
• Counter example?
    • R(A,B) ⋈ S(A,C) ⋈ T(A,D)
    • Best (local) plan for R ⋈ S: Hash-Join
    • Best (global) overall plan:
        • 1. Sort-Merge Join over R and S
        • 2. Sort-Merge Join with T

• Why might this be so?
    • The intermediate result of R ⋈sort-mergeS is ordered by join attribute A
    • This is an interesting order, that can be exploited later:
        • Later sort-merge joins
        • Grouping (GROUP BY)
        • Sorting (ORDER BY)
        • Duplicate Elimination (DISTINCT)


When choosing the best partial plan:
Cost comparison is not sufficient 
Sort orders must be considered

Solution:
• Save several "interesting sort orders" for each combination of relations: • Interesting orders for later plans
• Also keep an unsorted plan
• Dynamic programming tables become "wider" 
Also:  Memorize best join and sort operations which produce the order


这是关于**动态规划的局限性**以及**为什么需要保留多种排序变体**的解释，我来翻译成中文：

---
最优性原理的反例
- **回顾：最优性原理**：当两个计划在一个子计划上不同时，拥有更好子计划的计划就是更好的整体计划
- **反例：**
    - R(A,B) ⋈ S(A,C) ⋈ T(A,D)
    - R ⋈ S 的**最佳（局部）计划**：哈希连接
    - **最佳（全局）整体计划**：
        - 1. 对 R 和 S 使用**排序归并连接**
        - 2. 再与 T 进行排序归并连接

- **为什么会出现这种情况？**
    - R ⋈排序归并 S 的中间结果**按连接属性 A 排序**
    - 这是一个**感兴趣的顺序**，可以在后续被利用：
        - 后续的排序归并连接
        - 分组（GROUP BY）
        - 排序（ORDER BY）
        - 去重（DISTINCT）

---

选择最佳子计划时：
**成本比较是不够的**
**排序顺序必须考虑**

**解决方案：**
- 为每个关系组合保存多个**"感兴趣的排序顺序"**：
    - 为后续计划保留有用的排序
    - 同时保留**无序的计划**
- 动态规划表变得更"宽"
- 同时：记住能产生该顺序的**最佳连接和排序操作**

---

**补充说明：**

**为什么局部最优不等于全局最优？**
- 哈希连接对 R⋈S 可能成本更低
- 但它产生**无序**的结果
- 排序归并连接虽然成本稍高，但产生**有序**结果
- 这个有序结果可以让后续连接**避免再次排序**
- 整体成本反而更低

**动态规划的扩展：**
- 传统动态规划：每个子集只保留**成本最低**的计划
- Selinger 风格：每个子集保留**多种排序变体**的计划
- 例如：{R,S} 可能保留：
    - 无序计划（哈希连接）
    - 按 A 排序的计划（排序归并连接）
    - 按 B 排序的计划（如果后续 GROUP BY B）

**实际应用：**
- PostgreSQL、DB2、Oracle 等都采用这种多计划保留策略
- 需要在**搜索空间**和**优化质量**之间权衡

需要我画图说明**为什么排序归并连接在后续可以利用**吗？



## 3.4 Completing the Physical Query Plan

• Choosing a physical implementation
    • If not already done (e.g., with dynamic programming) 
    • Examples: Selection and Join
• Pipelining vs. materialization of intermediate results
- Access path (access method) for each table

Choice of Physical Operators
• How should relations be accessed?
    • Table Scan
    • Clustered Index Scan
    • Secondary Index Scan(s)
• How should joins be executed?
    • Hash Join
    • (Sort) Merge Join
    • (Index) Nested Loop Join
• Answer depends on:
    • Size of operator input
    • Amount of available memory
    • Data properties
        • Is data already sorted?
        • Will sorted data be helpful later?
    • Can we leverage pipelining?


选择物理实现
- 如果尚未完成（例如通过动态规划）
- 示例：选择和连接操作
- **流水线 vs. 物化**中间结果
- 每个表的**访问路径（访问方法）**

---

物理操作符的选择

**如何访问关系？**
- 表扫描
- 聚簇索引扫描
- 二级索引扫描

**如何执行连接？**
- 哈希连接
- （排序）归并连接
- （索引）嵌套循环连接

**答案取决于：**
- **操作符输入的大小**
- **可用内存量**
- **数据特性**
    - 数据是否已排序？
    - 排序后的数据后续是否有用？
- **能否利用流水线？**

---

**补充说明：**

**访问方法选择：**
- **表扫描**：适合大表全量读取，无索引可用时
- **聚簇索引扫描**：按索引顺序读取，适合范围查询
- **二级索引扫描**：可能需要回表，适合选择性高的条件

**连接方法选择：**

| 连接方法 | 适用场景 | 优点 | 缺点 |
|---------|---------|------|------|
| 哈希连接 | 等值连接，一大一小 | 只需一次扫描 | 需要内存建哈希表 |
| 归并连接 | 已排序或可排序 | 可利用已有顺序 | 需要排序开销 |
| 索引嵌套循环 | 小表驱动，有索引 | 可流水线执行 | 大表驱动时慢 |

**影响因素详解：**
1. **输入大小**：
   - 小表 → 嵌套循环
   - 大表 → 哈希或归并
2. **内存**：
   - 内存不足 → 避免哈希连接（可能溢出到磁盘）
3. **数据排序**：
   - 已排序 → 归并连接优先
   - 需要排序 → 权衡排序成本
4. **流水线**：
   - 能否边读边处理？影响内存和延迟



