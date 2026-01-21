
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


### 2.1.2 iterator model 迭代模型 (volcano, tuple-at-a-time) Book stops here 

• First processing model also called volcano style execution model (1994) or tuple-at-a-time
• Every operator has three functions and keeps its own state:
• Open():
    • Opens iterator and initializes data structures 
    • Calls Open for all its input operators
    • Note: No records are produced in this call
• GetNext():
    • Fetches/Computes the next result record
    • Calls GetNext for input operator(s) and compute results 
    • If there is no further tuple to get: returns NotFound
• Close():
    • Stops and closes iterator
    • Calls Close for all input operator(s)

每个算子都有三个函数并维护自己的状态：
Open()：
- 打开迭代器并初始化数据结构
- 调用其所有输入算子的 Open() 函数
- 注意：此调用中不产生任何记录
GetNext()：
- 获取/计算下一条结果记录
- 调用输入算子的 GetNext() 函数并进行结果计算
- 如果没有更多元组可获取：返回 NotFound
Close()：
- 停止并关闭迭代器

![](image/Pasted%20image%2020260121094954.png)

![](image/Pasted%20image%2020260121095106.png)

‣ Pros:
    ‣ Easy to implement
    ‣ No scheduler is needed => the model by "itself" evaluates the plan
    ‣ It enables pipelining and processing can start without that all data is in memory
‣ Cons:
    ‣ All operators run interleaved
    ‣ potentially large instruction footprint and thus many instruction cache misses ‣ Every operator call the next operator for every tuples
    ‣ potentially many function calls with significant overhead


‣ 优点：
实现简单：每个算子只需实现 open()、next()、close() 接口。
无需调度器：数据流通过算子间的 next() 调用自然驱动执行。
支持流水线处理：可以逐步处理数据，无需等待所有数据加载到内存。

‣ 缺点：
算子交叉执行：所有算子交替运行，频繁切换上下文。
指令缓存缺失风险大：大量算子代码交替执行，可能导致指令缓存效率低。
每个元组都需调用下级算子：对每个元组都产生一次函数调用。
函数调用开销大：大量函数调用可能带来显著的性能开销。



### 2.1.3 Materialization model 物化模型(operator-at-a-time, column-at-a-time)

‣ Also called Operator-at-a-time model
‣ Each operator processes its input all at once and then stores its output all at once (in one buffer)
‣ Operators consume and produce full columns or tables (instead of one tuple at a time)
‣ Operators fully materialize their results into so- called intermediate results in main-memory
‣ No pipelining is possible
‣ All operators run in a sequence from bottom to top and each operator runs exactly once


![](image/Pasted%20image%2020260121133541.png)


‣ Pros:
    ‣ Much fewer function calls, e.g., one per operator instead of one per
tuple
    ‣ Only one operator is active at a time => lower instruction footprint
    ‣ Can be better optimized on modern hardware, e.g., due to tight loops
‣ Cons:
    ‣ No pipelining
    ‣ Potentially large intermediate results and thus high demand on memory footprint


### 2.1.4 Iterator vs. Materialized Model

‣ 物化模型（一次一个算子模型）是一把双刃剑：
‣ 在代码和算子状态方面具有缓存友好性
‣ 紧凑的循环结构，代码可优化性强

‣ 但每个算子都需要完整地读入和写出所有数据：
‣ 数据可能无法放入缓存，甚至可能无法放入主内存
‣ 如果中间结果无法放入主内存，模型将失效
‣ 或许我们可以在两个极端之间找到平衡点（折中方案）


‣ Materialized (operator-at-a-time) model is a two-edged sword: 
    ‣ Cache-efficient with respect to code and operator state
    ‣ Tight loops, optimizable code
‣ But each operator reads in and out everything:
    ‣ Data won't fit into cache and maybe not even in main memory
    ‣ Useless if intermediate results do not fit into main memory
    ‣ Maybe we can find gold in the middle ground between the two extremes

![](image/Pasted%20image%2020260121133802.png)


### 2.1.5 Vectorization model (vector-at-a-time, batch, block-wise), 也可以称为 block-oriented model

> The term "vector" is overloaded in DBMS, it is used interchangeable with batch and block-oriented



‣ Idea: Use volcano-style iteration
    ‣ But for each next() call return a batch of tuples instead of the entire column or only one tuple
‣ The batch is called vector
‣ Vector size vary based on hardware and query properties:
    ‣ Large enough to compensate for iteration overhead
    ‣ Small enough to not trash the data cache


![](image/Pasted%20image%2020260121134552.png)

![](image/Pasted%20image%2020260121134606.png)

![](image/Pasted%20image%2020260121134626.png)


---

#### 2.1.5.1 Vectorized Model Properties


‣ Pros:
    ‣ Use the best of both worlds (iterator and materialized model) 
    ‣ Reduced number of invocation per operator
    ‣ Enable batch processing
‣ Cons:
    ‣ Hard to determine the "optimal" vector size for all workloads in advance
        ‣ Simple heuristics like L1 or L2 cache size ‣ Overall:
‣ Almost all state-of-the-art DBMS use this model nowadays

![](image/Pasted%20image%2020260121134723.png)


#### 2.1.5.2 Vectorized Execution in PostgreSQL

‣ Idea: Leave rest of system unchanged but switch from tuple- at-a-time to vector-at-a-time
‣ Solution: Buffer Operators between execution groups
‣ Buffer operator provides tuple-at- a-time interface to the outside but batches up tuples internally


![](image/Pasted%20image%2020260121135107.png)

## 2.2 Compilation-based Approaches 


‣ Ideally the query engine should spend all CPU cycles on useful work 
‣ Interpretation-based engines have some inherent overheads:
    ‣ Checking types, computing offsets, call a function for every vector, etc.
    ‣ Modern compilers cannot optimize this code as they cannot look across operator boundaries
‣ Solution: Query Compilation: Compiling a query in a declarative language like SQL down to machine code
    ‣ Executing the binary will produce the query result

理想情况下，查询引擎应该把所有 CPU 周期都用于"有用的工作"。

基于解释执行（interpretation-based）的引擎存在一些固有开销：
检查数据类型
计算内存偏移
对每个向量/元组都要进行函数调用等

现代编译器无法对这类代码进行充分优化，
因为它们无法跨越算子（operator）边界进行整体优化。

解决方案：查询编译（Query Compilation）
将类似 SQL 这样的声明式查询语言
直接编译成机器码（machine code）
执行生成的二进制程序即可直接得到查询结果。


1 
传统数据库执行查询时，通常是：
```
一条 SQL
→ 查询计划
→ 一个一个算子（scan / filter / join）
→ 每处理一行数据就：
   - 检查类型
   - 调函数
   - 跳转到下一个算子

```

问题是：

CPU 大量时间花在：
    if 判断
    函数调用
    指针跳转
而不是 真正做计算（比较、加法、过滤）
👉 CPU 没被"榨干"

2 
为什么编译器帮不上忙？
因为：
每个算子都是一个"黑盒函数"
编译器看不到：
filter 之后一定是 join
join 之后一定是 aggregate

👉 无法做跨算子的优化
（比如循环融合、消除中间结果）


3
Query Compilation 在干嘛？
Query Compilation 的思想是：
不要"解释"SQL，而是"编译"SQL

流程变成：
SQL
→ 生成一段专用的 C / LLVM / Machine Code
→ 直接跑这段代码

这段代码：
没有多余的函数调用
没有类型检查
没有虚函数跳转
所有算子都被"内联"进一个紧凑循环

👉 CPU 每个周期都在干正事


4
Query Compilation 通过将 SQL 编译为专用机器码，消除了解释执行中的函数调用与算子边界开销，从而显著提高 CPU 利用率和查询性能。



### 2.2.1 Code Generation Example

![](image/Pasted%20image%2020260121135944.png)

![](image/Pasted%20image%2020260121135953.png)

![](image/Pasted%20image%2020260121140000.png)


### 2.2.2 Code Generation Disadvantages

‣ Major drawback is the compilation time needed to compile the generated code and the high complexity:
‣ Compilation time in the order of hundreds of milliseconds or seconds are maybe too expensive for real-time analytics
    ‣ Mainly a problem for short running queries
‣ High complexity for developers due to indirection:
    ‣ Writecode that generates code that is executed

### 2.2.3 Compilation for Databases (Produce/Consume Model)

‣ Instead of having open/next/close, each operator implements a produce and consume method, so-called Produce/Consume Model compilation比 anay更耗时
‣ Code generation is conceptually done in two phases:
‣ Produce Phase ("Down-Phase"): Divides the query plan into pipelines
‣ Consume Phase ("Up-Phase"): Fills current pipeline with operators

生产阶段（"向下阶段"）：将查询计划划分为多个流水线（pipelines）
消费阶段（"向上阶段"）：向当前流水线中填入算子代码

----

Produce Phase
‣ Divides the query plan into pipelines
‣ Current operator is instructed to produce its result tuples
‣ Non-pipeline breaker: call the produce function of the child operator
‣ Pipeline-breaker: Start a new pipeline
    ‣ Create a new code generator and compile sub-pipeline before returning

‣ 将查询计划划分为多个流水线
‣ 指示当前算子生产其结果元组
‣ 若当前算子不是流水线阻断器：调用子算子的 produce 函数继续向下传递
‣ 若当前算子是流水线阻断器：启动一个新的流水线
    ‣ 创建一个新的代码生成器
    ‣ 先编译该子流水线的代码，再返回

---


Consume Phase
‣ Fills current pipeline with operators
‣ Tuples retrieved by the producer are pushed towards the consuming operator
‣ Produce function of an operator calls its consume function after calling the produce function of the child operators
    ‣ As a result, operators at the bottom of the query plan are inserted first
‣ Non-pipeline breaker: call consume function of the parent operator
‣ Pipeline breaker: will stop after calling their own consume function

‣ 向当前流水线中填入算子
‣ 由生产者获取的元组会被向上推送（push）给消费算子
‣ 算子的 produce 函数在调用子算子的 produce 函数之后，会调用它自身的 consume 函数
‣ 因此，查询计划底部的算子会最先被插入（代码生成顺序自底向上）
‣ 若当前算子不是流水线阻断器：调用父算子的 consume 函数继续向上传递
‣ 若当前算子是流水线阻断器：在调用自身的 consume 函数之后停止（不再向上传递）

---

Example

![](image/Pasted%20image%2020260121142040.png)

![](image/Pasted%20image%2020260121142048.png)

![](image/Pasted%20image%2020260121142130.png)


![](image/Pasted%20image%2020260121142744.png)

![](image/Pasted%20image%2020260121142753.png)

### 2.2.4 Compilation-based Query Execution

‣ The compilation-based approach compiles one binary during deployment
‣ Query evaluation starts from the bottom in a push-based model
‣ Each operator pushes its tuple to the next operator, also called push-based query execution
‣ Fuses multiple operators into pipelines
‣ The intra-operator granularity is usually data-centric instead of the previous operator-centric approach
    ‣ Data-centric: Pick one tuple, apply all operations
    ‣ Operator-centric: Pick one operator, apply it for all tuples

‣ 基于编译的方法会在部署时生成一个二进制执行文件
‣ 查询执行从底层算子开始，采用推式模型
‣ 每个算子将其处理后的元组推送给下一个算子，因此也称为推式查询执行
‣ 将多个算子融合为流水线
‣ 算子内部的执行粒度通常是数据中心的，而非传统算子中心的

数据中心（Data-Centric）：取一个元组，对其应用所有操作（在一个紧密循环中完成）
算子中心（Operator-Centric）：取一个算子，将其应用到所有元组上（传统的火山模型）


![](image/Pasted%20image%2020260121143217.png)


#### 2.2.4.1 How to actually compile code?

![](image/Pasted%20image%2020260121143301.png)


## 2.3 Comparison of Approaches

‣ Only two combinations withstand the test of time:
    ‣ Interpretation-based, vectorized, pull-based execution
        ‣ Production systems: DuckDB, Microsoft SQL Server, DB2
        ‣ Research prototypes: MonetDB X100, Velox
‣ Compilation-based, data-centric, push-based execution
    ‣ Production systems: Cloudera Impala, Microsoft's Hekaton, MemSQL
    ‣ Research prototypes: Hyper, Umbra, Peloton


![](image/Pasted%20image%2020260121143709.png)

### 2.3.1 Vectorization vs. Compilation



There is a single test system to compare compilation-based model (Typer) vs. a vectorization based model (Tectorwise)

![](image/Pasted%20image%2020260121143822.png)

![](image/Pasted%20image%2020260121143831.png)

![](image/Pasted%20image%2020260121143840.png)




# 3 Parallel Execution

Concurrent queries may seriously affect each other's performance.






## 3.1 Parallelism in Volcano：Exchange Operators


‣ Exchange Operators encapsulate all logic necessary for the parallel execution (e.g., managing of threads), other operators are "parallelization-unaware"
‣ Rest of the system remain unchanged
‣ It is "just" another operator in the query plan ‣ Instantiates one query operator plan per thread
‣ Bad performance due to decisions at compilation time not at runtime:
‣ Cannot handle load imbalances
‣ Degree of parallelism cannot be changed during runtime

‣ 交换算子（Exchange Operators） 封装了并行执行所需的所有逻辑（例如线程管理），其他算子则 "对并行化无感知"
‣ 系统的其余部分保持不变
‣ 它 "仅仅" 是查询计划中的一个特殊算子
‣ 会为每个线程实例化一份查询算子计划

‣ 性能不佳，因为决策在编译时确定而非运行时：
‣ 无法处理负载不均衡
‣ 并行度无法在运行时动态调整

这是数据库并行执行模型中常见的一种设计，称为 "交换算子模型"（也称为 Bushy Tree Parallelism 或 Exchange Operator Model）。

核心思想：
在传统查询计划树中插入 Exchange 算子作为并行化边界
Exchange 负责数据的分发（partition）、收集（gather）、合并（merge）
并行执行时，每个线程运行 Exchange 之间 的子计划，相互之间通过 Exchange 交换数据
其他普通算子（如 Scan、Join、Aggregate）不需要知道自己在并行执行，保持原有逻辑



![](image/Pasted%20image%2020260121144301.png)

### 3.1.1 Morsel-Driven Parallelism 分片驱动并行

‣ Break Operator into chunks or around 100K tuples, called "morsels"
‣ Fixed number of operators per thread
‣ Morsels are dispatched to threads dynamically during runtime (using a task queue)
‣ Operators are aware of the parallelism

‣ 将算子拆分为若干块，每块约 10 万行元组，称为"分片"（Morsels）
‣ 每个线程固定分配一组算子
‣ 分片在运行时动态调度给线程（使用任务队列）
‣ 算子本身对并行化是有感知的

1. 分片（Morsels）
数据被逻辑上分成较小的、大小均匀的块（例如 10 万行）
分片是 任务分配的基本单位，而不是整个表或整个分区
这样可以将一个大任务拆成许多小任务，便于动态调度

2. 固定算子，动态数据
每个线程有自己的一组算子实例（例如每个线程有自己的哈希表、局部聚合器等）
但处理哪个分片是由 中央任务队列 动态分配的
线程空闲时就去任务队列取下一个分片处理

3. 运行时动态调度
使用任务队列实现 工作窃取（Work-Stealing）
某个线程处理完自己的分片后，可以去"偷"其他线程还未处理的分片
从而自动实现 负载均衡，即使数据分布倾斜

4. 算子感知并行
与传统 Exchange 模型不同，这里的算子知道自己在并行执行

例如：
哈希连接：每个线程先构建本地哈希表，最后再合并
聚合：每个线程先做局部聚合，最后全局合并
算子内部逻辑会做相应的并行优化



![](image/Pasted%20image%2020260121144353.png)


---

Each pipeline is parallelized individually using all threads

![](image/Pasted%20image%2020260121144607.png)


![](image/Pasted%20image%2020260121144654.png)
