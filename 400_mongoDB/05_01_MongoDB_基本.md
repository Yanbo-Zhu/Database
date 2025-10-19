
[`MongoDB`](https://link.juejin.cn?target=https%3A%2F%2Fwww.mongodb.com%2F "https://www.mongodb.com/")是数据库家族中的一员，是一款专为扩展性、高性能和高可用而设计的数据库，它可以从单节点部署扩展到大型、复杂的多数据中心架构，也能提供高性能的数据读写操作；而且提供了数据复制、无感知的故障自动选主等功能，从而实现数据节点高可用。

但`MongoDB`并不是一款关系型数据库，而是一款基于“**分布式存储**”的非关系型数据库（`NoSQL`），由`C++`编写而成。它与`Redis、Memcached`这类传统`NoSQL`不同，`MongoDB`具有半结构化特性，啥意思呢？下面仔细聊聊。

  

# 1 Eine Dokumentenorientierte Datenbank

![](images/Pasted%20image%2020250206220244.png)

Merkmale
- Dokumentenorientierte Datenbank, d.h. Daten werden in Dokumenten gespeichert und in Kollektionen verwaltet
- Dokumente entsprechen Zeilen einer Tabelle in einer relationalen Datenbank
- Kollektionen entsprechen Tabellen in einer relationalen Datenbank
- MongoDB ist schemafrei - Dokumente einer Kollektion müssen nicht einem bestimmten, vorgegebenen Schema folgen

BSON
- _**Binary JSON**_: binäre codierte Serialisierung von JSON
- Zugrunde liegendes Dokumentenformat
- Unterstützung der JSON-Datenformate, plus zusätzlicher Formate (z.B. `**date**` oder `**byte**` array)


`MongoDB`的数据存储格式非常松散，采用“无模式、半结构化”的思想，通过`BSON`格式来存储数据。

> [`BSON`](https://link.juejin.cn?target=https%3A%2F%2Fbsonspec.org%2F "https://bsonspec.org/")的全称为`Binary JSON`，翻译过来是指二进制的`JSON`格式，在存储和扫描效率高于原始版`JSON`，但是对空间的占用会更高。同时，`BSON`在`JSON`基础之上，多加了一些类型及元数据描述，具体可参考[《MongoDB官网-BSON类型》](https://link.juejin.cn?target=https%3A%2F%2Fwww.mongodb.com%2Fdocs%2Fmanual%2Freference%2Fbson-types%2F "https://www.mongodb.com/docs/manual/reference/bson-types/")。

之所以说`MongoDB`具有半结构化特性，这是由于`BSON`拥有着`JSON`的特性，`JSON`天生具备结构性，可以便捷的存储复杂的结构数据。但是`Mongo`的数据结构又特别灵活，没有关系型数据库那种“强结构化”的规范。

> 强结构化：所有数据入库前，库中必须先定义好结构，并且入库的数据，每个值要与定义好的字段一一对应，当结构发生变更时，再按原有结构插入数据则会报错，必须同步修改插入的数据，使用起来特别“繁琐”。

`MongoDB`中用`BSON`来存储数据，而`BSON`是一种文档，所以它也被称之为：**文档型数据库**，文档由一或多个`K-V`键值对组成，其中的`Key`只能由字符串表示，值则可以是任意类型，如数组、字符串、数值……。简单来说，你可以把`MongoDB`中的一个文档，看做成一个`JSON`对象。


|      MySQL      |      MongoDB       |
| :-------------: | :----------------: |
| 数据库（`DataBase`） |  数据库（`DataBase`）   |
|  数据表（`Table`）   | 数据集合（`Collection`） |
|   数据行（`Row`）    |  数据文档（`Document`）  |
| 列/字段（`Column`）  |    字段（`Field`）     |
|   索引（`Index`）   |    索引（`Index`）     |



# 2 Lokale MongoDB versus Cloud MongoDB


![](images/Pasted%20image%2020250206221208.png)

mongod:  mongo daemon


# 3 库

- 查询所有数据库：`show dbs;`或`show databases;`
- 切换/创建数据库：`use 库名;`
- 查看目前所在的数据库：`db;`
- 查看`MongoDB`目前的连接信息：`db.currentOp();`
- 查看当前数据库的统计信息：`db.stats();`
- 删除数据库：`db.dropDatabase("库名");`

## 3.1 show databases; | show dbs;

- 从打印的信息来看一共有三个库；
    
    --- admin 0.00GB
    
    --- config 0.00GB
    
    --- local 0.00GB
    

> **⚠️注意:**
> 
> `admin` 、`config`、`local` 这三个库是MongoDB的保留库。不要删除也不要随便对其进行操作；
> 
> admin：从权限角度来看，这是 `root` 库。如果将一个用户添加到这个数据库当中，这个用户将会自动继承所有的数据库的权限。一些特定的服务器端命令也只能从这个数据库运行。
> 
> ​ 比如：列出所有的数据库或者关闭服务器；
> 
> config：当MongoDB用于分片设置时。config数据库在内部使用，用于保存分片的相关信息；
> 
> local： 这个数据永远不会被复制，可以用来储存限于本地单台服务器的任意集合；

  
# 4 集合的命令 
- 查看库中的所有集合：`show collections;`或`show tables;`
- 显式创建集合：`db.createCollection("集合名");`
- 查看集合统计信息：`db.集合名.stats();`
- 查看集合的数据大小：`db.集合名.totalSize();`
- 删除指定集合：`db.集合名.drop();`

## 4.1 创建集合

`db.createCollection("collectionName", [options]);`

创建集合又两个参数，第一个参数是填写 集合的名称，第二个参数是填写设置。`如果不填写options则使用默认配置`； 

**Options部分参数：**

|字段|类型|描述|
|---|---|---|
|capped|布尔|（可选）如果为true，则创建固定集合。固定集合是指有着固定大小的集合；  <br>如果参数达到了最大值，它会自动覆盖最早的文档。**当该值为true时，必须制定size参数**；|
|size|数值|（可选）为固定集合制定一个指定的最大值，即字节数；  <br>**如果capped为true时，必须制定该字段的参数**；|
|max|数值|（可选）指定固定集合中包含文档的最大数量；|

```
db.createCollection("collectionNmae", {max: 100, capped: true, size:1000})
```






# 5 数据类型 

关于`CRUD`操作相关的命令咱们就此打住，掌握上述那些基本够用了，下面来看看`MongoDB`中的数据类型。

看完前几个阶段的内容大家会发现，`MongoDB`的命令，用的其实就是`JavaScript`的语法，所以在数据类型这块，原生`JS`中支持的`MongoDB`几乎都支持，这里就简单列一下：

- `String`：存储变长的字符串；
- `Number`：存储整数或浮点型小数；
- `Boolean`：存储`true、flase`两个布尔值；
- `Date`：存储日期和时间；
- `Array`：可以存储由任意类型组成的数组；
- `Embedded-Document`：可以将其他结构的文档，嵌套到一个文档的字段中；
- `Binary`：存储由`0、1`组成的二进制数据，如图像、音/视频等；
- `Code`：`MongoDB`支持直接存储`JavaScript`代码；
- `Timestamp`：存储格林威治时间（`GMT`）的时间戳；
- `GeoJSON`：存储地理空间数据；
- `Text`：存储大文本数据；

从上面列出的类型不难发现，其实这些数据，的的确确都是原生`JS`支持的类型~

  
# 6 事务

`MongoDB`作为数据库家族的一员，自然也支持事务机制，只不过相较于`InnoDB`的事务机制而言，`MongoDB`事务方面并没有那么强大，这倒不是因为官方技术欠缺，而是由于`MongoDB`的定位是：**大数据、高拓展、高可用、分布式**，因此在实现事务时，不仅仅要考虑单机事务，而且需要考虑分布式事务，复杂度上来之后，自然无法做到`MySQL-InnoDB`那种单机事务的强大性。

这里也列出`MongoDB`事务方面的改进过程，如下：

- `3.0`版本中，引入`WiredTiger`存储引擎，开始支持单文档事务；
- `4.0`版本中，开始支持多文档事务，以及副本集（主从复制）架构下的事务；
- `4.2`版本中，开始支持分片集群、分片式多副本集架构下的事务。

再次类比`MySQL`，`MongoDB`支持了分片集群中的事务，而`MySQL`只支持主从集群下的事务，并不支持分库环境下的事务，在这方面，`MongoDB`强于`MySQL`，下面来看看事务怎么使用：

```js
// 开启一个会话
var session = db.getMongo().startSession({readPreference:{mode: "primary"}});
// 开启事务：指定读模式为快照读，写模式为半同步，即写入半数以上节点后再返回成功
session.startTransaction({readConcern: {level:"snapshot"}, writeConcern:{w: "majority"}});
// 获取要操作的集合对象
var trx_coll = session.getDatabase("库名").getCollection("集合名");

// 要在事务里执行的CRUD操作
......

// 回滚事务命令
session.abortTransaction();
// 提交事务命令
session.commitTransaction();
// 关闭会话命令
session.endSession();
  
```


大家一眼看下来，会发现使用起来非常麻烦，首先需要先拿到一个会话，接着开启事务时，还要指定一堆参数，回滚/提交事务的命令也不一样……，为此，如若之前没接触过的小伙伴，看着或许会很别扭。

这里重点解释一下开启事务的`startTransaction()`方法的参数，以及可选项，如下：

- `readConcern`：指定事务的读取模式
    - `level`：指定一致性级别
        - `local`：读取最近的数据，可能包含未提交的事务更改；
        - `available`：读取已提交的数据，可能包含尚未持久化的事务更改；
        - `snapshot`：读取事务开始时的一致快照，不包含未提交的事务更改；
- `writeConcern`：指定事务的写入模式
    - `w`：指定写操作的确认级别（同步模式）
        - `[number]`：写操作在写入指定数量的节点后，返回写入成功；
        - `majority`：写操作在写入大多数节点（半数以上）后，返回写入成功；
        - `tagSetName`：写操作在写入指定标签的节点后，返回写入成功；
    - `j`：写入是否应被持久化到磁盘；
    - `wtimeout`：指定写入确认的超时时间；
- `readPreference`：定义读操作的节点优先级和模式
    - `mode`：指定读取模式
        - `primary`：只从主节点读取数据；
        - `secondary`：只在从节点上读取数据；
        - `primaryPreferred`：优先从主节点读取，主节点不可用，转到从节点读取；
        - `secondaryPreferred`：优先在从节点读取，从节点不可用，转到主节点读取；
        - `nearest`：从可用节点中选择最近的节点进行读取；
    - `tagSets`：在带有指定的标签的节点上执行读取操作

OK，对于`MongoDB`的事务机制，了解上述命令即可，毕竟`MongoDB`本身就不适用于强事务的场景，原因如下：

- ①`MongoDB`的事务必须在`60s`内完成，超时将自动取消（因为要考虑分布式环境）；
- ②涉及到事务的分片集群中，不能有仲裁节点（后面会解释仲裁节点）；
- ③事务会影响集群数据同步效率、节点数据迁移效率；
- ④多文档事务的所有操作，必须在主节点上完成，包括读操作；

综上所述，就算`MongoDB`支持事务，可实际使用起来也会有诸多限制，因此在不必要的情况下，不建议使用其事务机制。


# 7 锁

接着再来说说`MongoDB`的锁机制，由于其天生的分布式特性，所以内部的锁机制尤为复杂，`MySQL`中有的锁概念，`MongoDB`中几乎都有，为此，详细、深入的内容可参考：[MongoDB锁机制](https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F644323120 "https://zhuanlan.zhihu.com/p/644323120")，这里只简单列出手动操作锁的命令，如下：

  ```JavaScript
# 获取锁
db.collection.fsyncLock();
# 释放锁
db.collection.fsyncUnlock();
```





