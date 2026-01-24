

# 1 Question 1

Given the following relational operators:

Selection

Projection

Join

Aggregation

Explain their functionality and associated physical implementations.

![](image/Pasted%20image%2020260121145407.png)


# 2 Question 2

Given the following SQL query and its logical query tree, determine two different physical execution plans in the form of a query tree.

Important assumptions:
- You do not need to wory about how the data schema / table looks like.
- We do not care about the optimal plan, i.e. the performance. This is part of the next lecture “Query Optimization”.

SQL:
```
SELECT T1.id, AVG(T2.value) as avg_value
FROM T1
JOIN T2 ON T1.id = T2.id
WHERE T1.status = 'active'
GROUP BY T1.id;
```


![](image/Pasted%20image%2020260121145551.png)

---

Solution
- Physical query tree 1 (hash-based execution):
    - Selection: Index Scan (filter status = ‘active’).
    - Join: Hash-Join ([T1.id](http://t1.id/) = T2.t1_id).
    - Aggregation: Hash-based Aggregation (handles group by [T1.id](http://t1.id/)).
    - Projection: Tuple Reconstruction.




![](image/Pasted%20image%2020260121145621.png)

---

Solution
- Physical query tree 2 (sort-based execution):
    - Selection: Sequential Scan (scans the entire table).
    - Join: Sort-Merge Join (requires sorted input).
    - Aggregation: Sort-based Aggregation (sorts data before aggregation).
    - Projection: Tuple Reconstruction.


![](image/Pasted%20image%2020260121145720.png)

# 3 Question 3

Explain what is a **pipelined** operator and what is a **blocking** operator.


Solution

- Pipelined: The operator can start producing output tuples as soon as it sees an input tuple, without waiting for the entire input to be processed.
    - Example: Hash Probe, Selection.
- Blocking: The operator has to process all input tuples before it can start producing output tuples.
    - Example: Sort, Grouping, Join (build phase of Hash Join).
- Some operators combine both pipelinable and blocking sub-operators, e.g. Joins:
    - Blocking: The build phase (building the hash table).
    - Pipelined: The probe phase (matching tuples from the hash table).  探索阶段 


# 4 Question 4


Consider the following properties of the operators `Source`, `FilterEven`, and `Sort`.

![](image/Pasted%20image%2020260121150358.png)

----


Run the code chunks below to understand the phenomenon of pipelined and blocking operators.
```
import time


class Source:
    def __init__(self, n: int):
        self.current = 0
        self.n = n

    def __iter__(self):
        return self

    def __next__(self):
        if self.current >= self.n:
            raise StopIteration
        value = self.current
        self.current += 1
        time.sleep(0.5)
        return value


source = Source(10)

print("(Source) --> (print)")
for it in source:
    print(it)
```


![](image/Pasted%20image%2020260121150639.png)



---

```
class FilterEven:
    def __init__(self, child):
        self.child_iter = iter(child)

    def __iter__(self):
        return self

    def __next__(self):
        for value in self.child_iter:
            if value % 2 == 0:
                return value
        raise StopIteration


source = Source(10)
filt = FilterEven(source)

print("(Source) --> (FilterEven) --> (print)")
for it in filt:
    print(it)
```

![](image/Pasted%20image%2020260121150736.png)


----

```
class SortDescending:
    def __init__(self, child):
        self.child = child
        self.buffer = None
        self.pos = 0

    def _materialize_and_sort(self):
        if self.buffer is None:
            self.buffer = list(self.child)
            self.buffer.sort(reverse=True)

    def __iter__(self):
        return self

    def __next__(self):
        self._materialize_and_sort()

        if self.pos >= len(self.buffer):
            raise StopIteration

        value = self.buffer[self.pos]
        self.pos += 1
        return value


source = Source(10)
sort_des = SortDescending(source)

print("(Source) --> (SortDescending) --> (print)")
for x in sort_des:
    print(x)
```

![](image/Pasted%20image%2020260121150753.png)

---

```
source = Source(10)
filt = FilterEven(source)
sort_des = SortDescending(filt)

print("(Source) --> (FilterEven) --> (SortDescending) --> (print)")
for x in sort_des:
    print(x)
```

![](image/Pasted%20image%2020260121150810.png)


# 5 Question 5

Given two SQL queries and their execution plans, identify the pipelining and blocking operators for each, and describe why it is classified like this.

**Query 1**

SQL:
SELECT T1.id, T2.value
FROM T1
JOIN T2 ON T1.id = T2.id
WHERE T1.status = 'active';

Query execution tree:
Output (Projection: T1.id, T2.value)
└── Join (Hash Join: Build phase and Probe phase)
    ├── Sequential Scan on T2
    └── Selection (Filter T1.status = 'active')
        └── Sequential Scan on T1


- Query 1:
    - Pipelining:
        - Sequential Scan on T2: Produces output as it scans through the input. Subsequent operators do not have to wait for the scanning operator to complete its entire operation.
        - Selection (T1.status = ‘active’): This is pipelined because it can filter each tuple from T1 as it is read.
        - Join (Probe phase): The probe phase is pipelined because it processes each tuple from T2 as it arrives and matches it with the hash table built from T1.
    - Blocking:
        - Join (Build phase): The build phase of the hash join is blocking because the hash table must be built using all the tuples from T1 before any tuples from T2 can be processed.



----


**Query 2**

SQL:
SELECT T1.id, AVG(T2.value) as avg_value
FROM T1
JOIN T2 ON T1.id = T2.t1_id
GROUP BY T1.id;

Query execution tree:
Output (Projection: T1.id, avg_value)
└── Sort-based Aggregation (Group by T1.id, AVG(T2.value) as avg_value)
    └── Merge Join
        ├── Sort by T1.id
        |   └── Sequential Scan on T1
        └──  Sort by T2.id
            └── Sequential Scan on T2


- Pipelining:
    - Sequential Scan on T1 and T2: Produces output as it scans through the input.
    - Merge Join: Produces output as it merges tuples from T1 and T2.
- Blocking:    
    - Sort on [T1.id](http://t1.id/) and [T2.id](http://t2.id/): This is a blocking operation because both input tables must be fully sorted before the merge join operator can process this data.


# 6 Question 6

How does each processing model handle tuples within a query operator?

- Volcano Model
    
- Materialization Model
    
- Vectorization Model

Solution
- Volcano Model (Iterator-Based Processing):
    - Operators implement an iterator interface with methods like open(), next(), and close().
    - Each operator calls next() on its child operator to pull one tuple at a time.
    - Processing is **tuple-at-a-time**, i.e. a single tuple is processed through an operator and then passed to the parent.
- Materialization Model:
    - Each operator processes all tuples in its input, applies its logic (e.g., filtering, joining), and outputs a new intermediate relation.
    - The parent operator processes the materialized result.
- Vectorization Model:
    - Tuples are processed in batches (vectors) rather than individually.
    - A “vector” is a block of multiple tuples, typically sized to fit the CPU cache.


- **火山模型（迭代器式处理）：**
    - 算子实现迭代器接口，包含 `open()`、`next()` 和 `close()` 等方法。
    - 每个算子调用其子算子的 `next()` 方法，每次拉取一个元组。
    - 处理方式是**一次一个元组**，即单个元组在一个算子中处理后传递给父算子。
- **物化模型：**
    - 每个算子处理其所有输入元组，应用其逻辑（如过滤、连接等），并输出一个新的中间关系（结果集）。
    - 父算子再处理该物化后的结果。
- **向量化模型：**
    - 元组按批次（向量）处理，而不是逐个处理。
    - 一个“向量”是由多个元组组成的块，其大小通常设计为可放入 CPU 缓存。


---

| 维度         | **火山模型** (Volcano) | **物化模型** (Materialization) | **向量化模型** (Vectorization) |
| ---------- | ------------------ | -------------------------- | ------------------------- |
| **处理单元**   | 一次一个元组             | 一次一个算子（处理全部输入）             | 一次一个批次（向量）                |
| **数据流方向**  | 拉式（Pull）           | 拉式或物化后再传递                  | 拉式，但按批次传递                 |
| **函数调用开销** | 高（每元组多次调用）         | 中（每算子一次调用）                 | 低（每批次一次调用）                |
| **内存使用**   | 低（流水线处理）           | 高（中间结果全物化）                 | 中（批次缓存友好）                 |
| **缓存效率**   | 差（代码跳跃频繁）          | 中（算子代码集中）                  | 优（数据局部性好）                 |
| **适合场景**   | OLTP、通用执行引擎        | 内存数据库、简单查询                 | OLAP、分析型负载                |
|            |                    |                            |                           |
|            |                    |                            |                           |

1. **数据规模**：小数据→火山，大数据→向量化
2. **查询复杂度**：简单→物化，复杂→向量化/火山
3. **硬件特性**：多核SIMD→向量化，内存受限→火山
4. **负载类型**：OLTP→火山，OLAP→向量化


# 7 Question 7: alculate the number of invocations

Given the following SQL query and its excution steps, calculate the number of invocations for each operator on each processing model. For Materialization Model, assume that intermediate results are fully materialized.


Query:
SELECT SUM(col2)
FROM T
WHERE col1 > 50;

Execution steps:
1. Scan T (produces tuples [col1, col2]).
2. Filter col1 > 50 (only rows satisfying the condition are passed).
3. Aggregate SUM(col2).

Assumptions:
- Data characteristics:
    - The table T contains 10 million rows.
    - The condition `col1 > 50` is satisfied for 10% of the rows (1 million rows pass the filter).
- For vector-at-a-time:
    - The batch size is 1000 tuples, i.e. an operator processes 1000 tuples-at-time.
- For materialization:
    - All the table can fit into memory.



---


Solution

1. Volcano Model:
    - Scan: 10 million rows → 10 million invocations.
    - Filter: 10 million rows → 10 million invocations.
    - Aggregate: 1 million rows → 1 million invocations.
    - Total: 21 million operator invocations.
2. Materialization Model:
    - Scan: 1 invocation (entire table processed in bulk).
    - Filter: 1 invocation (all rows passed in bulk).
    - Aggregate: 1 invocation (all filtered rows passed in bulk).
    - Total: 3 operator invocations.
3. Vectorization Model:
    - Scan: 10 mio / 1000 = 10k invocations.
    - Filter: 10 mio / 1000 = 10k invocations.
    - Aggregate: 1 mio / 1000 = 1000 invocations.
    - Total: 21k operator invocations.


![](image/Pasted%20image%2020260121151935.png)

# 8 Quiz

## 8.1 Calculate the number of invocations

Given the following SQL query and execution steps, calculate the number of invocations per operator for each processing model (Volcano, Materialization, Vectorization).

**SQL:**

```
SELECT MIN(e.salary)
FROM employee e
WHERE e.age > 35;
```

**Execution steps:**

1. Scan table `employee e`.
2. Filter results on `e.age > 35`.
3. Aggregate tuples on `MIN(e.salary)`.

**Important assumptions:**

- Table employee consists of 500000 rows.
- The predicate (`e.age > 35`) has a selectivity factor of 0.5 (50% pass this filter).
- The vectorization model uses a batch size of 2000 tuples (processes 2000 tuples at a time).
- For the materialization model, intermediate results are fully materialized.

 
 ----

 **1. 火山模型（Tuple-at-a-time / Volcano）**

- 每个算子一次处理一个元组，通过 `next()` 调用传递。
    
- **Scan**：被调用 500,000 次（每个元组一次）。
    
- **Filter**：被调用 500,000 次（每个元组一次）。
    
- **Aggregate**：调用次数取决于通过 filter 的元组数量。  
    通过 filter 的元组数 = 500,000 × 0.5 = 250,000。  
    Aggregate 每次收到一个元组时更新一次 MIN，所以是 **250,000 次**。

---

**物化模型（Operator-at-a-time / Materialization）**

- 每个算子一次性处理全部输入，并物化完整中间结果。
    
- **Scan**：调用 **1 次**，读取全表并输出所有 500,000 个元组。
    
- **Filter**：调用 **1 次**，接收 500,000 个元组，输出 250,000 个元组（完全物化）。
    
- **Aggregate**：调用 **1 次**，接收 250,000 个元组，计算 MIN。


**向量化模型（Vector-at-a-time / Vectorization）**

- 每次处理一批元组，批量大小 = 2000 个元组。
    
- 批次数 = 总行数 / 批量大小（向上取整）。
    
- 需要按照流水线传播来计算各算子的调用次数。


批次数 = ceil(500,000 / 2000) = ceil(250) = 250 批
Scan 调用次数 = 250 次


Filter 接收来自 Scan 的 250 批，每批最多 2000 个元组（最后一批可能少些）。  
Filter 需要输出批次给 Aggregate，但这里的 Aggregate 只需要一个值，通常是向量化模型会逐批更新状态（例如每批计算部分 MIN，最后合并）。  
对于向量化模型，通常 Aggregate 会接收来自 Filter 的输出批次并逐批更新聚合状态，但这里需要看 Filter 的输出批次数。

Filter 不会改变批次数（它过滤掉元组，但输出的批次仍然是连续的，除非整批都被过滤掉才会产生空批，但向量化实现中通常还是会传一个空批或者跳过？）。

实际上，更常见的设计：

- Scan 输出 250 批 → Filter 对每批应用条件，输出可能是较少的元组，但批次仍以相同批次数量传递给 Aggregate，只是某些批次可能为空。
    
- 因此 Filter 调用次数 = 接收批次数 = 250 次。
    
- Aggregate 调用次数也等于 Filter 输出批次数 = 250 次（因为向量化 Aggregate 每批更新一次局部 MIN，最后合并）。  不应该这样计算
    - 500000/2/2000  = 125

Scan:     250 次
Filter:   250 次
Aggregate: 125 次

