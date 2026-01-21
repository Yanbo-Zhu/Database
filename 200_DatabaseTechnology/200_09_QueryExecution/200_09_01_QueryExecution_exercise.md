

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


# 7 Question 7 

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

