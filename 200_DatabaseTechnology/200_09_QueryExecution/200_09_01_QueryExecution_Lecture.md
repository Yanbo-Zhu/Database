
![](image/Pasted%20image%2020260302174356.png)

# 1 Relational Algebra


Basic Query execution
![](image/Pasted%20image%2020260120170011.png)


---
The five base operators of the relational algebra are selection, projection, difference, union, and cartesian product. 

![](image/Pasted%20image%2020260120170124.png)

5 base operators of the Relational Algebra (+ rename)
Selection σ  =where 
Projection π    = select 
Difference –
Union ∪
Cartesian Product X
ρ = Renaming

Derived operators, e.g., 
Join ⋈ (X and σ)
Intersect ∩ (𝑅∩𝑆=𝑅−(𝑅−𝑆)

Additional operators, e.g.,
aggregation function γ (SUM, AVG, MAX, etc.), grouping G

Translation from Bags to Sets (de-duplication)
Unique/distinct D


Difference: 
- **集合差 difference （A − B）** 的定义是：
    > 属于 A 但不属于 B 的元素
- 用集合运算表示为：  
    👉 **A − B = A ∩ ¬B**
关键点在于：  
➡️ **需要“补集（complement）”**

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


- **Join（连接）**  
    ✔️ **可以是 pipelining 的**（例如 nested-loop join、hash join 的 probe 阶段）
- **Grouping（分组 / GROUP BY）**  
    ❌ **通常是 blocking 的**
    - 需要先看到所有相关元组，才能完成分组和聚合
- **Sort（排序）**  
    ❌ **典型的 blocking operator**
    - 必须先读完全部输入才能输出第一个结果


# 2 Processing Models

‣ A processing models determines how a DBMS executes a given query plan 
‣ Today's DBMS use combinations of the following techniques:
    ‣ Execution method: Interpretation-based approaches vs. Compilation- based approaches (analogous to programming languages)
    ‣ Intra-operator granularity: Granularity at which tuples are processed within an operator: e.g., tuple, vector, column, data centric, etc.
    ‣ Inter-operator execution: Policy to define the control flow between operations in a query plan (see next slide)


好的，我把这段关于数据库执行模型的内容翻译成中文：

- **处理模型**决定了数据库管理系统如何执行给定的查询计划
- 当今的数据库管理系统使用以下技术的**组合**：
    - **执行方式**：基于解释的方法 vs. 基于编译的方法（类似于编程语言）
    - **操作内粒度**：在操作符内部处理元组的粒度，例如：元组、向量、列、数据为中心等
    - **操作间执行**：定义查询计划中操作符之间控制流的策略（见下一张幻灯片）


Upstream  (towards Source)  
pull model
- downstream operator request data times
- the request is passed upstream until source 

Downstream (towards sink)
pushed data
Upstream operator  pushed data
The request is pushed further downstream



![](image/Pasted%20image%2020260120171028.png)


## 2.1 Interpretation-based Approaches 

‣ Interpretation-based approaches "interpret" the query plan during runtime
‣ Query evaluation starts from the root in a pull-based model
‣ Each operator pulls the next tuple up, also called pull-based query execution
‣ The following models were proposed (in chronological order) with intra- operator granularities:
    ‣ Iterator model (volcano, tuple-at-a-time) Book stops here 
    ‣ Materialization model (operator-at-a-time, column-at-a-time)
    ‣ Vectorization model (vector-at-a-time, batch, block-wise)


- **基于解释的方法**在运行时"解释"执行查询计划
- 查询评估从根节点开始，采用**拉取模型**
- 每个操作符向上拉取下一个元组，也称为**基于拉取的查询执行**
- 以下模型按时间顺序提出，具有不同的**操作内粒度**：
    - **迭代器模型**（火山模型，一次一个元组）← 教材只讲到这里
    - **物化模型**（一次一个操作符，一次一列）
    - **向量化模型**（一次一个向量，一批，一块）


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


- 也称为**一次一个操作符模型**
- 每个操作符一次性处理完所有输入，然后一次性存储其输出（在一个缓冲区中）
- 操作符**消费和产生完整的列或表**（而不是一次一个元组）
- 操作符将其结果完全物化到主内存中的所谓**中间结果**
- **无法进行流水线处理**
- 所有操作符**从下到上按顺序运行**，每个操作符**只运行一次**




![](image/Pasted%20image%2020260121133541.png)


‣ Pros:
    ‣ Much fewer function calls, e.g., one per operator instead of one per tuple
    ‣ Only one operator is active at a time => lower instruction footprint
    ‣ Can be better optimized on modern hardware, e.g., due to tight loops
‣ Cons:
    ‣ No pipelining
    ‣ Potentially large intermediate results and thus high demand on memory footprint


这是对**物化模型（Operator-at-a-time）优缺点**的总结，我来翻译成中文：

---

**优点：**
- **函数调用少得多**，例如每个操作符只调用一次，而不是每个元组调用一次
- **一次只有一个操作符处于活动状态** → 指令占用空间更小
- 在**现代硬件上可以更好地优化**，例如由于**紧凑的循环**（tight loops）
- 实现简单
- 适合内存足够大的场景
- 对于需要多次扫描同一结果的操作（如排序、聚合）有利

**缺点：**
- **无法进行流水线处理**
- **中间结果可能非常大**，因此对内存占用要求高
- 中间结果可能非常大
- 内存开销大
- 无法利用流水线并行

---

**解释一下：**

**优点详解：**
1. **函数调用少**：在迭代器模型中，每个元组经过每个操作符都要调用一次`next()`，函数调用开销很大。物化模型每个操作符只调用一次，大幅减少开销。
2. **指令缓存友好**：只有一个操作符在运行，CPU可以专注于执行该操作符的紧凑循环，指令缓存命中率高。
3. **向量化优化**：可以更容易地利用SIMD指令进行批量处理。

**缺点详解：**
1. **无流水线**：必须等一个操作符完全执行完才能开始下一个，无法像迭代器模型那样边生产边消费。
2. **内存压力大**：中间结果需要完整存储在内存中，例如连接操作可能需要构建完整的哈希表或排序结果。

---

**适用场景：**
- 内存足够大的OLAP场景
- 需要对整个列进行批量计算的场景（如列存数据库）
- 操作符本身需要全量数据的场景（如排序、聚合）


### 2.1.4 Iterator vs. Materialized Model

‣ Materialized (operator-at-a-time) model is a two-edged sword: 
    ‣ Cache-efficient with respect to code and operator state
    ‣ Tight loops, optimizable code
‣ But each operator reads in and out everything:
    ‣ Data won't fit into cache and maybe not even in main memory
    ‣ Useless if intermediate results do not fit into main memory
    ‣ Maybe we can find gold in the middle ground between the two extremes


‣ 物化模型（一次一个算子模型）是一把双刃剑：
    ‣ 在代码和算子状态方面具有缓存友好性
    ‣ 紧凑的循环结构，代码可优化性强

‣ 但每个算子都需要完整地读入和写出所有数据：
    ‣ 数据可能无法放入缓存，甚至可能无法放入主内存
    ‣ 如果中间结果无法放入主内存，模型将失效
    ‣ 或许我们可以在两个极端之间找到平衡点（折中方案）


![](image/Pasted%20image%2020260121133802.png)


### 2.1.5 Vectorization model (vector-at-a-time, batch, block-wise), 也可以称为 block-oriented model

> The term "vector" is overloaded in DBMS, it is used interchangeable with batch and block-oriented


‣ Idea: Use volcano-style iteration
    ‣ But for each next() call return a batch of tuples instead of the entire column or only one tuple
‣ The batch is called vector
‣ Vector size vary based on hardware and query properties:
    ‣ Large enough to compensate for iteration overhead
    ‣ Small enough to not trash the data cache


**核心思想：使用火山风格的迭代**
- 但每个 `next()` 调用返回**一批元组**，而不是整个列或仅仅一个元组
- 这一批元组被称为**向量**
- 向量大小根据硬件和查询特性变化：
  - **足够大**以抵消迭代开销
  - **足够小**以避免数据缓存颠簸（cache trashing）

---

**向量化模型的特点：**

**工作方式：**
- 继承迭代器模型的**拉取式流水线**架构
- 但每个操作符每次返回**一个向量**（例如 100~1000 个元组）
- 操作符内部用**紧凑循环**处理整个向量

**优点：**
1. **流水线仍在** → 不需要物化完整中间结果
2. **函数调用减少** → 比一次一个元组少几十到几百倍
3. **缓存友好** → 向量大小适合L1/L2缓存
4. **SIMD友好** → 可以对向量进行批量操作

**缺点：**
- 实现比简单迭代器复杂
- 需要为不同硬件调整向量大小


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


这是对**向量化模型优缺点**的总结，我来翻译成中文：

---

**优点：**
- **集两者之长**（结合了迭代器模型和物化模型的优点）
- **减少每个操作符的调用次数**
- **支持批量处理**

**缺点：**
- 难以预先确定对所有工作负载都"最优"的向量大小
    - 只能使用简单的启发式方法，例如基于 L1 或 L2 缓存大小
- **总体而言：**
- 当今几乎所有**最先进的数据库管理系统**都使用这种模型

---

**为什么向量化模型是当今主流？**

1. **性能平衡**：
   - 相比一次一个元组：减少解释开销
   - 相比一次一列：保持流水线，避免大中间结果

2. **硬件适应性**：
   - 可以针对不同CPU缓存大小调整向量长度
   - 便于利用SIMD指令

3. **实践中的向量大小**：
   - MonetDB/X100: 约 1000 个元组
   - DuckDB: 2048 个元组（可配置）
   - ClickHouse: 块大小可配置，默认 65,536 行
   - 通常基于 L1/L2 缓存大小 + 元组大小估算


![](image/Pasted%20image%2020260121134723.png)


#### 2.1.5.2 Vectorized Execution in PostgreSQL

‣ Idea: Leave rest of system unchanged but switch from tuple- at-a-time to vector-at-a-time
‣ Solution: Buffer Operators between execution groups
‣ Buffer operator provides tuple-at- a-time interface to the outside but batches up tuples internally



---

**核心思想：保持系统其余部分不变，但从一次一个元组切换到一次一个向量**

**解决方案：在执行组之间使用**缓冲操作符****

- **缓冲操作符**对外部提供**一次一个元组**的接口
- 但在内部**将元组批量缓存**起来

---

**这样做的目的：**

1. **兼容性**：
   - 不需要重写整个系统
   - 可以逐步引入向量化

2. **模块化**：
   - 缓冲操作符作为适配器
   - 上游是向量化执行，下游是传统的一次一个元组

3. **性能提升**：
   - 关键路径（如扫描、过滤、投影）可以用向量化
   - 非关键部分可以保持原样

---

**例子：**
```
[向量化扫描] → [缓冲操作符] → [传统迭代器模型的上层操作]
                      ↓
                 一次返回一个元组
```

这种设计在**逐步改造遗留系统**时很常见。

![](image/Pasted%20image%2020260121135107.png)

## 2.2 Compilation-based Approaches  (Query Compilation)

==Query Compilation 通过将 SQL 编译为专用机器码，消除了解释执行中的函数调用与算子边界开销，从而显著提高 CPU 利用率和查询性能。==

‣ Ideally the query engine should spend all CPU cycles on useful work 
‣ Interpretation-based engines have some inherent overheads:
    ‣ Checking types, computing offsets, call a function for every vector, etc.
    ‣ Modern compilers cannot optimize this code as they cannot look across operator boundaries
‣ Solution: Query Compilation: Compiling a query in a declarative language like SQL down to machine code
    ‣ Executing the binary will produce the query result

理想情况下，查询引擎应该把所有 CPU 周期都用于"有用的工作"。

基于解释执行（interpretation-based）的引擎存在一些固有开销：
- 检查数据类型， 计算内存偏移， 对每个向量/元组都要进行函数调用等
- 现代编译器无法对这类代码进行充分优化， 因为它们无法跨越算子（operator）边界进行整体优化。

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
==Query Compilation 通过将 SQL 编译为专用机器码，消除了解释执行中的函数调用与算子边界开销，从而显著提高 CPU 利用率和查询性能。==



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


**主要缺点是编译生成代码所需的编译时间以及高复杂性：**

- **编译时间**在数百毫秒甚至秒级，对于实时分析来说可能太昂贵
    - 主要对**短查询**是个问题
- 对开发人员来说，由于**间接性**导致**高复杂性**：
    - 编写**生成代码**的代码，然后执行生成的代码

---

**编译时间问题：**
- 对于运行几秒钟的长查询，几百毫秒的编译开销可以接受
- 对于亚秒级的短查询（如OLTP、实时仪表盘），编译开销可能占主导

**复杂性问题：**
- 需要写**代码生成器**（如C++模板元编程、LLVM IR生成）
- 调试困难：运行时生成的代码很难追踪错误
- 维护成本高：需要同时维护逻辑和代码生成逻辑

**代表系统：**
- **Hyper**（现Tableau）：LLVM JIT编译
- **Umbra**：Hyper的继任者，优化编译速度
- **NoisePage**：DBOS系统的编译执行
- **SingleStore**（前MemSQL）：部分编译


### 2.2.3 Compilation for Databases (Produce/Consume Model)

‣ Instead of having open/next/close, each operator implements a produce and consume method, so-called Produce/Consume Model compilation     比 anay更耗时
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


# 3 Comparison of Approaches

‣ Only two combinations withstand the test of time:
‣ Interpretation-based, vectorized, pull-based execution
    ‣ Production systems: DuckDB, Microsoft SQL Server, DB2
    ‣ Research prototypes: MonetDB X100, Velox
‣ Compilation-based, data-centric, push-based execution
    ‣ Production systems: Cloudera Impala, Microsoft's Hekaton, MemSQL
    ‣ Research prototypes: Hyper, Umbra, Peloton


**只有两种组合经受住了时间的考验：**

1. **基于解释、向量化、基于拉取的执行**
   - **生产系统**：DuckDB、Microsoft SQL Server、DB2
   - **研究原型**：MonetDB X100、Velox

2. **基于编译、以数据为中心、基于推送的执行**
   - **生产系统**：Cloudera Impala、Microsoft's Hekaton、MemSQL
   - **研究原型**：Hyper、Umbra、Peloton

---

**补充说明：**

**第一种组合（解释+向量化+拉取）：**
- 继承火山模型的拉取式架构
- 每个操作符每次返回一批数据（向量）
- 平衡了灵活性和性能
- 代表：DuckDB（嵌入式分析）、SQL Server（批处理模式）

**第二种组合（编译+数据为中心+推送）：**
- 将查询编译成紧凑的循环代码
- 数据在操作符之间直接推送，减少物化
- 代表：Hyper（LLVM编译）、Umbra（Hyper继任者）
- 适合高性能OLAP

**Velox**是Facebook开源的C++向量化执行引擎，被Presto、Spark等使用。

需要我进一步解释**基于拉取（pull-based）**和**基于推送（push-based）**的区别吗？


![](image/Pasted%20image%2020260121143709.png)

### 3.1.1 Vectorization vs. Compilation

There is a single test system to compare compilation-based model (Typer) vs. a vectorization based model (Tectorwise)

![](image/Pasted%20image%2020260121143822.png)

![](image/Pasted%20image%2020260121143831.png)

![](image/Pasted%20image%2020260121143840.png)


Query compilation requires fewer CPU instructions and hides cache misses better than vectorized query processing.： wahr  
- **Query compilation** 会把整个查询编译成紧凑的机器码，  
    👉 减少函数调用、类型检查等**额外 CPU 指令**。
    
- 通过 **算子内联（operator fusion）**，形成长循环，  
    👉 **更好地隐藏 cache miss**（流水线执行、指令级并行）。
    
- 相比之下，**vectorized query processing** 仍然存在算子边界和批处理调度开销。


# 4 Parallel Execution

Concurrent queries may seriously affect each other's performance.

## 4.1 Parallelism in Volcano：Exchange Operators


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

### 4.1.1 Morsel-Driven Parallelism 分片驱动并行

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
