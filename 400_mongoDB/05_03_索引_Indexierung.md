
https://juejin.cn/post/7272730422735929396#heading-16

https://juejin.cn/post/7073273393528700964#heading-30

PS：早版本的`MongoDB`中，索引底层默认使用`B-Tree`结构，而`4.x`版本后，`MongoDB`推出了`V2`版索引，默认使用变种`B+Tree`来作为索引的数据结构（和`MySQL`索引的数据结构相同）


# 1 初识MongoDB索引

在之前提到过，`MongoDB`会为每个集合生成一个默认的`_id`字段，该字段在每个文档中必须存在，可以手动赋值，如果不赋值则会默认生成一个`ObjectId`，该字段则是集合的主键，`MongoDB`会基于该字段创建一个默认的主键索引，后续基于`_id`字段查询数据时，会走索引来提升查询效率。

但当咱们基于其他字段查询时，由于未使用`_id`作为条件，这会导致`find`语句走全表查询，即从第一条数据开始，遍历完整个集合，从而检索到目标数据。当集合中的数据量，达到百万、千万、甚至更高时，意味着效率会直线下滑，在这种情况下，必须得由我们手动为频繁作为查询条件的字段建立索引。


![](images/Pasted%20image%2020250206223425.png)

```js
{
  "_id": ObjectId("52d02240eb01d67d71ad577"),
  "title": "First Blog Post",
  "comments": [
    ...., ....., .....
  ],
  "commentsCount": 12
}

db.posts.find({"commentsCount": {$gt: 10}})
```

- Datenbankindex: von der Datenstruktur getrennte Indexstruktur in einer Datenbank, welche die Suche und das Sortieren nach bestimmten Datenbankfeldern beschleunigt
- Anstelle alle Felder eines Dokumentes bei einer Anfrage zu überprüfen, liefert ein Index eine schnelle Referenz auf alle Dokumente, welche die Bedingungen der Anfrage erfüllen

- MongoDB-Datenbank zur Verwaltung von Einträgen eines Blogs: pro Dokument werden ein Blogeintrag und alle dazu eingegangenen Kommentare gespeichert
- Dediziertes Feld für den schnellen Zugriff auf die Anzahl der eingegangenen Kommentare
- Index auf dem Feld `**commentsCount**` macht Dokumente schnelle auffindbar



# 2 索引分类 

这里咱们先来说说索引的分类，同`MySQL`一样，站在不同维度上来说，可以衍生出不同的分类。

从字段数量的维度划分：
- 单列索引：基于一个字段建立的索引；
- 组合索引：也叫复合索引、多列索引、联合索引，是由多个字段组成的索引；
- 多键索引：和上面的不同，如果索引字段为数组类型，`MongoDB`会专门为每个元素生成索引条目；
- 部分索引：使用一个或多个字段的前`N`个字节创建出的索引；

从排序维度上划分：
- 升序索引：一或多个字段组成的索引，排序方式完全为升序（从小到大）；
- 降序索引：一或多个字段组成的索引，排序方式完全为降序（从大到小）；
- 多序索引：多个字段组成的索引，但其中一部分字段为升序，一部分字段为降序；

从功能维度划分：
- 主键索引：`MongoDB`中默认为`_id`字段且不能更改，用于维护聚簇索引树；
- 普通索引：一或多个字段组成的索引，没有任何特殊性，只为提升检索效率；
- 唯一索引：一或多个字段组成的索引，值不能重复，且只允许一个文档不插入索引字段；
- 全文索引：一或多个字段组成的索引，集合内只能有一个，可以基于索引字段进行全文检索；
- 空间索引：基于地理空间数据字段创建的索引，支持平面几何的`2D`索引和球形几何的`2D sphere`索引；

从索引性质维度划分：
- 稀疏索引：结合唯一索引使用，允许一个唯一索引字段，出现多个`null`值；
- TTL索引：类似于`Redis`的过期时间，为一个字段创建`TTL`索引后，超时会自动删除整个文档；
- 隐藏索引：在不删除索引的情况下，把索引隐藏起来，语句执行时不再走索引，相当于关闭索引；
- 通配符索引：给内嵌文档、且会发生动态变化的字段创建的索引；

从存储方式维度划分：
- 聚簇索引：索引数据和文档数据存储在一起的索引；
- 非聚簇索引：索引数据和文档数据分开存储的索引；

从数据结构维度划分：
- B+树索引：索引底层采用`B+Tree`结构存储；
- 哈希索引：索引底层采用`Hash`结构存储；

显而易见，这里又出现了一堆索引相关的名词，不过好在其中大部分概念，和`MySQL`索引的概念相同，下面展开聊聊。

# 3 索引基础API

> db.colltectionName.getIndexes()
--- 查看集合索引
> db.colltectionName.totalIndexSize()
--- 查看集合索引大小
> db.colltectionName.dropIndexes()
--- 删除集合所有索引
> db.colltectionName.dropIndex("索引名称")
--- 删除指定索引



# 4 创建索引 

先来看看`MongoDB`中创建索引的命令，如下：

```JavaScript
db.collection.createIndex(<key and index type specification>, <options>);

> db.colltectionName.createIndex({"title":1})
--- 创建一个title字段的索引，为升序创建；
> db.colltectionName.createIndex({"title":1, "description":-1})
--- 创建复合索引 title 和 description ，分别为升序和降序；
> db.colltectionName.createIndex({open: 1, close: 1}, {background: true})
--- 创建索引时指定需要的参数

```

前面提到的所有索引，都是通过这一个方法创建，不同类型的索引，通过里面的参数和选项来区分，下面说明一下参数和可选项。

第一个参数主要是传字段，以及索引类型，这里可以传一或多个字段，用于表示单列/复合索引。
Key 值为你要创建的索引字段，1 为指定按升序创建索引，如果你想按降序来创建索引指定为 -1 即可；





第二个参数表示可选项，如下：
- `background`：是否以后台形式创建索引，因为创建索引会导致其他操作阻塞；
- `unique`：是否创建成唯一索引；
- `name`：指定索引的名称；
- `sparse`：是否对集合中不存在的索引字段的文档不启用索引；
- `expireAfterSeconds`：指定存活时间，超时后会自动删除文档；
- `v`：指定索引的版本号；
- `weights`：指定索引的权重值，权值范围是`1~99999`，当一条语句命中多个索引时，会根据该值来选择；

![](image/Pasted%20image%2020250226215845.png)




---


接着还是以之前的`xiong_mao`集合为例，来阐述索引相关内容，数据如下：

```JavaScript
db.xiong_mao.insert([
    {_id:1, name:"肥肥", age:3, hobby:"竹子", color:"黑白色"},
    {_id:2, name:"花花", color:"黑白色"},
    {_id:4, name:"黑熊", age:3, food:{name:"黄金竹", grade:"S"}},
    {_id:5, name:"白熊", age:4, food:{name:"翠绿竹", grade:"B"}},
    {_id:6, name:"棕熊", age:3, food:{name:"明月竹", grade:"A"}},
    {_id:7, name:"红熊", age:2, food:{name:"白玉竹", grade:"S"}},
    {_id:8, name:"粉熊", age:6, food:{name:"翡翠竹", grade:"A"}},
    {_id:9, name:"紫熊", age:3, food:{name:"烈日竹", grade:"S"}},
    {_id:10, name:"金熊", age:6, food:{name:"黄金竹", grade:"S"}}
```


## 4.1 单列索引

```JavaScript
db.xiong_mao.createIndex({name: -1}, {name: "idx_name"});
```

这代表着基于`name`字段，创建一个名为`idx_name`的单列普通索引，`-1`代表降序，不过对于单字段的索引而言，排序方式并不重要，因为索引底层默认是`B+Tree`，每个文档之间会有双向指针，为此，`MongoDB`基于单字段索引查询时，既可以向前、也可以向后查找数据，示意图如下：

![MongoDB索引](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a7ae058ce734307b847ffc48cb7545e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

创建完成后，可以通过`db.xiong_mao.getIndexes()`命令查询索引，如下：

```json
[
  {v: 2, key: { _id: 1 }, name: '_id_'},
  {v: 2, key: { name: -1 }, name: 'idx_name'}
]
```

这里可以看到，`MongoDB`默认给`_id`字段创建的索引，第二个则是咱们手动创建的索引，索引的版本为`2`（如果不指定索引名，会按“字段名+下划线+排序方式”规则默认生成）。

再来看一种情况，例如现在将集合中的爱好字段，变为一个数组：

```Json
{_id:1, name:"肥肥", age:3, hobby:["竹子", "睡觉"], color:"黑白色"}
```

现在给`hobby`字段创建一个索引，这时叫啥索引？多键索引！因为这里是基于单个数组类型的字段在建立索引，所以`MongoDB`会为数组中的每个元素，都生成索引的条目（即索引键），由于一个文档的数组字段，拥有多个元素，因此会创建多个索引键，这也是“多键索引”的名字由来。


----
更多的索引类型 见 https://juejin.cn/post/7272730422735929396#heading-16


## 复合索引

主要是由两个字段共同维护的一个索引；

```js
db.colltectionName.createIndex(
	{
  	name: 1,
  	age: -1
  },
  {
  	name: "name_age_index"
  }
);

```


# 5 explain执行计划

在之前的[《SQL优化篇》](https://juejin.cn/post/7164652941159170078#heading-22 "https://juejin.cn/post/7164652941159170078#heading-22")中，曾详细说到过`MySQL`的`explain`工具，通过该命令，能有效帮咱们分析语句的执行情况，而`MongoDB`同样也提供了这个命令，在上面的索引阶段，也简单使用过，命令格式如下：

```JavaScript
db.<collection>.find().explain(<verbose>);
```

`explain`方法同样有三个模式可选，这里简单列出来：

- `queryPlanner`：返回执行计划的详细信息，包括查询计划、集合信息、查询条件、最佳计划、查询方式、服务信息等（默认模式）；
- `exectionStats`：列出最佳执行计划的执行情况和被拒绝的计划等信息（即语句最终执行的方案）；
- `allPlansExecution`：选择并执行最佳执行计划，同时输出其他所有执行计划的信息；

一般排查`find()`查询缓慢问题时，可以先指定第二个模式，查看最佳执行计划的信息；如果怀疑`MongoDB`没选择好索引，则可以再指定第三个模式，查看其他执行计划，

## 5.1 hint 强制指定要使用的索引

如果的确是因为走错了索引，这时你可以通过`hint`强制指定要使用的索引，如下：

```JavaScript
db.集合名.find(查询条件).hint(索引名);
```

OK，接着来了解一下`explain`命令输出的信息含义，大家可以自行去`mongosh`里执行一下`explain`命令，会发现它输出了特别多的信息，咱们主要关注`stage`这个值，这是最重要的字段，就类似于`MySQL-explain`的`type`字段，代表着本次语句的查询类型，该字段可能会出现以下值：

- `COLLSCAN`：扫描整个集合进行查询；
- `IXSCAN`：通过索引进行查询；
- `COUNT_SCAN`：使用索引在进行`count`操作；
- `COUNTSCAN`：没使用索引在进行`count`操作；
- `FETCH`：根据索引键去磁盘拿具体的数据；
- `SORT`：执行了`sort`排序查询；
- `LIMIT`：使用了`limit`限制返回行数；
- `SKIP`：使用了`skip`跳过了某些数据；
- `IDHACK`：通过`_id`主键查询数据；
- `SHARD_MERGE`：从多个分片中查询、合并数据；
- `SHARDING_FILTER`：通过`mongos`对分片集群执行查询操作；
- `SUBPLA`：未使用索引的`$or`查询；
- `TEXT`：使用全文索引进行查询；
- `PROJECTION`：本次查询指定了返回的结果集字段（投影查询）；

这里咱们只需要带`SCAN`后缀的，因为其他都属于命令执行的“阶段”，并不属于具体的类型，`explain`会将一条语句执行的每个阶段，都详细列出来，每个阶段都会有`stage`字段，我们要做的，就是确保每个阶段都能用上索引即可，如果某一阶段出现`COLLSCAN`，在数据量较大的情况下，都有可能导致查询缓慢。

其次，咱们需要关心`keysExamined、docsExamined`两个字段的值（`exectionStats`模式下才能看到），前者代表扫描的索引键数量，后者代表扫描的文档数量，前者越大，代表索引字段值的离散性太差，后者的值越大，一般代表着没建立索引。

OK，这里暂时了解这么多，关于`explain`其余的输出信息，很多跟集群有关系，为此，等后续把`MongoDB`集群过了之后，有机会再详细讲述~





