

In der vergangenen Woche haben wir angefangen, uns mit Datenströmen auseinanderzusetzen. Dabei haben wir gesagt, dass Datenströme potenziell unendlich lang sind und dass die Gültigkeit dieser Daten nur kurzlebig ist. Wenn wir jedoch Anfragen stellen wollen, bevor die Daten ihre Gültigkeit verlieren, ein Datenstrom aber potenziell unendlich lang ist, so müssen wir diesen unterteilen und Anfragen auf einzelnen Abschnitten eines Datenstroms stellen. Für diese Partitionierung führen wir heute das sogenannte Windowing ein, welches anhand der Window Definition den Datenstrom partitioniert. Die Anfragen werden dann auf einer pro Window Basis ausgeführt. Deswegen ist das Datenstrom-Paradigma durch wiederkehrende Anfragen geprägt. Somit werden im Data Streaming die Rollen der Daten und Anfragen vertauscht. Während in einer relationalen Datenbank die Daten ruhen und sich generell nicht viel verändern, sind die Anfragen “in Bewegung”, da diese nur ad-hoc/spontan gestellt werden. Im Data Streaming bewegen sich stattdessen die Daten mit einer hohen Geschwindigkeit, aber die Anfragen ruhen, da diese einmal gestellt werden und dann wiederkehrend ausgeführt werden.

Des Weiteren haben wir in der vergangenen Woche nur Tuple basierte Streams kennengelernt. In dieser Woche haben wir dies durch zeitbasierte Streams ergänzt, welche wir in diesem Tutorium entsprechend wiederholen. Zuletzt wurden dann auch noch approximative Methoden (Synopsen) vorgestellt, welche ebenfalls in diesem Tutorium wiederholt und geübt werden.


下面我们将探讨**近似数据方法**，英文中称为 **Synopses（概要）**。具体来说，这些方法被划分为四类：**直方图（Histograms）**、**抽样（Samples）**、**草图（Sketches）** 和 **小波（Wavelets）**。这些方法非常流行，尤其在 **数据流（Data Streaming）** 领域中有广泛的应用——在这些场景中，数据量可能大到必须**牺牲准确性以换取计算效率**。



# 1 Einführung

Wir verwenden erneut die eigens definierte Streaming-API der letzten Woche. Dieses Mal ergänzen wir die imports um die zuvor erwähnten zeitbasierten Streams, als auch die Synopsen (engl. Synopses). Für eine tatsächliche Echtzeitverarbeitung bräuchte man dedizierte Systeme wie z.B. Apache Flink, die mit einer größeren Komplexität einhergehen.

Die Klassen und Methoden der API sind ausführlich dokumentiert. Siehe: [https://dima.gitlab-pages.tu-berlin.de/isda/isda-streaming/isda_streaming.html](https://dima.gitlab-pages.tu-berlin.de/isda/isda-streaming/isda_streaming.html)


```
import random

from isda_streaming.data_stream import DataStream, TimedStream
from isda_streaming.synopsis import BloomFilter, CountMinSketch, ReservoirSample
```


# 2 Windowing


In den folgenden Beispielen wollen wir uns mit Windowing beschäftigen. Als Windowing bezeichnen wir den Prozess der Diskretisierung eines Datenstroms mit Fenstern. Aber wieso benötigen wir dies überhaupt?

Als erstes initialisieren wir einen Datenstrom.

```
ds = DataStream().from_collection([1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12])
print(ds)
```

```
DataStream([1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12])
```

Nun wollen wir den Stream in einzelne Windows unterteilen. Es gibt viele unterschiedliche Windowarten, aber in der Vorlesung haben Sie die folgenden kennengelernt:

- Landmark Windows: Alle Windows haben einen gemeinsamen Startpunkt, jedoch wird das Window Ende späterer Windows linear um den übergebenen Dauer Parameter länger.
    - **Landmark Window** 是一种**固定起点的窗口机制**，窗口的起始时间是固定的（landmark），而结束时间是动态或人为触发的。它会**收集从 landmark 到当前时间的所有数据**。
    - ✅ **窗口大小会增长**（直到人为或系统触发重新设置）
    - ✅ 用于 **从某一事件或时间点开始，持续观测到某一时刻** 的应用
- Tumbling Windows: Erneut wird eine Dauer als Parameter übergeben. Dieses Mal ist dies die gesamte Länge des Windows, wobei das Ende eines Windows der Beginn des nächsten ist. Windows überlappen sich hier also nie.
    
- Sliding Windows: Beim Sliding Window gibt es zwei Parameter: die Dauer und den Slide Faktor, welcher angibt, um wie viel sich ein Window Anfang bzw. Ende verschiebt. Wenn der Slide der Dauer entspricht, so entspricht das Sliding Window dem Tumbling Window.

In der unten stehenden Codezelle können Sie das Verhalten genauer analysieren.

```
w1 = ds.landmark_window(4)
print("Landmark Windows:\n", w1)
w2 = ds.tumbling_window(4)
print("Tumbling Windows:\n", w2)
w3 = ds.sliding_window(4, 2)
print("Sliding Windows:\n", w3)
```



```
Landmark Windows:
 WindowedStream([
	start(0) [1, 2, 3, 4] end(3),
	start(0) [1, 2, 3, 4, 5, 6, 7, 8] end(7),
	start(0) [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12] end(11)
])
Tumbling Windows:
 WindowedStream([
	start(0) [1, 2, 3, 4] end(3),
	start(4) [5, 6, 7, 8] end(7),
	start(8) [9, 10, 11, 12] end(11)
])
Sliding Windows:
 WindowedStream([
	start(0) [1, 2, 3, 4] end(3),
	start(2) [3, 4, 5, 6] end(5),
	start(4) [5, 6, 7, 8] end(7),
	start(6) [7, 8, 9, 10] end(9),
	start(8) [9, 10, 11, 12] end(11)
])
```



Es ist uns nun möglich, mittels `reduce`, `aggregate` oder `apply` auf dem Windowed Stream weiterzuarbeiten. Mehr über deren Funktionsweise findet man in der `isda_streaming` Dokumentation.

```
# Example: reduce functions on windowed streams
def sum_all(value1, value2):
    return value1 + value2


result = w1.reduce(sum_all)
print(result)
result = w2.reduce(sum_all)
print(result)
result = w3.reduce(sum_all)
print(result)
```


```
DataStream([(10, 0, 3), (36, 0, 7), (78, 0, 11)])
DataStream([(10, 0, 3), (26, 4, 7), (42, 8, 11)])
DataStream([(10, 0, 3), (18, 2, 5), (26, 4, 7), (34, 6, 9), (42, 8, 11)])
```


---

```
# Example: aggregate on windowed streams
result = w1.aggregate("mean")
print(result)
result = w2.aggregate("mean")
print(result)
result = w3.aggregate("mean")
print(result)
```


```
DataStream([(np.float64(2.5), 0, 3), (np.float64(4.5), 0, 7), (np.float64(6.5), 0, 11)])
DataStream([(np.float64(2.5), 0, 3), (np.float64(6.5), 4, 7), (np.float64(10.5), 8, 11)])
DataStream([(np.float64(2.5), 0, 3), (np.float64(4.5), 2, 5), (np.float64(6.5), 4, 7), (np.float64(8.5), 6, 9), (np.float64(10.5), 8, 11)])
```

---

```
# Example: custom apply functions on windowed streams
def get_first_element(window: list):
    return window[0]


result = w1.apply(get_first_element)
print(result)
result = w2.apply(get_first_element)
print(result)
result = w3.apply(get_first_element)
print(result)
```


```
DataStream([(1, 0, 3), (1, 0, 7), (1, 0, 11)])
DataStream([(1, 0, 3), (5, 4, 7), (9, 8, 11)])
DataStream([(1, 0, 3), (3, 2, 5), (5, 4, 7), (7, 6, 9), (9, 8, 11)])
```


---

Es ist auch möglich, einen Stream zeitbasiert zu unterteilen. Hierzu brauchen wir eine Instanz von `TimedStream`, bei der jeder Wert mit einem zugehörigen Timestamp versehen ist.

```
data = [0, 1, 2, 3, 4, 5]
timestamps = [0.0, 1.0, 2.0, 3.0, 4.0, 5.0]
timed_stream = TimedStream().from_collection(data, timestamps)
print(timed_stream)

w1 = timed_stream.tumbling_time_window(3.0)
print(w1)
```


```
TimedStream([(0, 0.0), (1, 1.0), (2, 2.0), (3, 3.0), (4, 4.0), (5, 5.0)])
WindowedStream([
	start(0.0) [0, 1, 2] end(3.0),
	start(3.0) [3, 4, 5] end(6.0)
])
```



---


Hierbei hängt die Einteilung der Fenster nun nicht mehr von der Anzahl der Tupel ab, sondern von den Zeitstempeln der Tupel, Zeitintervall und den Timestamps ab. Um dies zu verdeutlichen, definieren wir einen zweiten Zeit basierten Datenstrom mit den gleichen Daten wie zuvor, aber mit anderen Zeitstempeln.


```
data = [0, 1, 2, 3, 4, 5]
timestamps = [0.0, 2.0, 4.0, 6.0, 8.0, 10.0]
timed_stream = TimedStream().from_collection(data, timestamps)
print(timed_stream)

w1 = timed_stream.tumbling_tuple_window(3)
print("Tuple-based Tumbling Windows:\n", w1)
w2 = timed_stream.tumbling_time_window(3.0)
print("Time-based Tumbling Windows:\n", w2)
```


```
TimedStream([(0, 0.0), (1, 2.0), (2, 4.0), (3, 6.0), (4, 8.0), (5, 10.0)])
Tuple-based Tumbling Windows:
 WindowedStream([
	start(0) [0, 1, 2] end(2),
	start(3) [3, 4, 5] end(5)
])
Time-based Tumbling Windows:
 WindowedStream([
	start(0.0) [0, 1] end(3.0),
	start(3.0) [2] end(6.0),
	start(6.0) [3, 4] end(9.0),
	start(9.0) [5] end(12.0)
])
```



# 3 Synopsen

Im Folgenden beschäftigen wir uns mit approximativen Datenmethoden, die im Englischen als Synopses bezeichnet werden. Konkret sind diese Methoden in vier Klassen unterteilt: Histogramme, Samples, Sketches und Wavelets. Diese Methoden sind sehr beliebt und finden vor allem im Bereich des Data Streamings viele Anwendungen, in denen die Datenmenge so massiv sein kann, dass Genauigkeit zugunsten von Rechenleistung geopfert wird. Des Weiteren haben sie einen sehr geringen Speicherbedarf, da sie die Daten komprimieren. Im Folgenden wiederholen wir drei solcher Datenstrukturen, die Sie bereits in der Vorlesung kennengelernt haben. Dabei gehören der **Bloom-Filter** und der **Count-Min Sketch** zur Klasse der Sketches, während das **Reservoir Sample** zur Klasse der Samples gehört. Wir betrachten also explizit keine Histogramme oder Wavelets.

Bei weiterem Interesse zu diesem Thema können Sie das Buch “Synopses for Massive Data: Samples, Histograms, Wavelets, Sketches” konsultieren [[1]](https://dsf.berkeley.edu/cs286/papers/synopses-fntdb2012.pdf).


下面我们将探讨**近似数据方法**，英文中称为 **Synopses（概要）**。具体来说，这些方法被划分为四类：**直方图（Histograms）**、**抽样（Samples）**、**草图（Sketches）** 和 **小波（Wavelets）**。这些方法非常流行，尤其在 **数据流（Data Streaming）** 领域中有广泛的应用——在这些场景中，数据量可能大到必须**牺牲准确性以换取计算效率**。

此外，它们的另一个优势是**极低的内存需求**，因为它们对数据进行了压缩处理。

在下文中，我们将回顾三种你在课程中已经学过的数据结构。其中：
- **Bloom Filter** 和 **Count-Min Sketch** 属于 **Sketches（草图）** 类；
- **Reservoir Sample（蓄水池抽样）** 属于 **Samples（抽样）** 类。
    

我们将不会讨论直方图（Histograms）和小波（Wavelets）。

如果你对该主题有更深入的兴趣，可以查阅书籍《**Synopses for Massive Data: Samples, Histograms, Wavelets, Sketches**》111。

## 3.1 三者比较 

| 数据结构                 | 类别       | 原因说明                         |
| -------------------- | -------- | ---------------------------- |
| **Bloom Filter**     | Sketches | 通过位图和哈希函数近似判断集合包含关系，不存储原始数据。 |
| **Count-Min Sketch** | Sketches | 通过二维数组和哈希函数近似估计频率，是频率的“草图”。  |
| **Reservoir Sample** | Samples  | 抽取原始数据中的代表性子集，直接做抽样操作。       |

| 特性     | Bloom Filter        | Count-Min Sketch |
| ------ | ------------------- | ---------------- |
| 功能     | 判断是否存在（集合）          | 估计频率（多重集）        |
| 错误类型   | 假阳性（false positive） | 频率可能被高估          |
| 是否能删除  | 普通版不能（可扩展版本可以）      | 不能（只能增加）         |
| 典型应用场景 | 缓存判断、黑名单过滤          | 数据流中找热词、统计频次     |

| 特性     | Bloom Filter        | Count-Min Sketch |
| ------ | ------------------- | ---------------- |
| 功能     | 判断是否存在（集合）          | 估计频率（多重集）        |
| 错误类型   | 假阳性（false positive） | 频率可能被高估          |
| 是否能删除  | 普通版不能（可扩展版本可以）      | 不能（只能增加）         |
| 典型应用场景 | 缓存判断、黑名单过滤          | 数据流中找热词、统计频次     |

## 3.2 Count-Min Sketch


Count-Min Sketch 是一个**估算频率**的数据结构，用于近似地统计数据流中元素出现的次数。
- 使用多行哈希函数和二维数组。
- 空间小，查询快，但结果是估算值（over-estimate）。


属于 **Sketches**：
- Count-Min Sketch 不存储所有元素，而是通过多个哈希函数和数组“画出”一个**频率的草图**。
- 是经典的 streaming sketch，用于处理海量数据流


**Count-Min Sketch** 是一种**空间高效的概率性数据结构**，用于**近似地统计数据流中元素的频率**，尤其适用于**大数据和流处理场景**。
- ✅ **近似计数**：不会返回完全精确的频率，但能以确定的误差范围内估计它。
- ✅ **低内存**：不需要为每个元素保存完整的计数。
- ✅ **速度快**：更新和查询操作是常数时间 O(1)。


它的内部结构是一个二维数组 `count[depth][width]`：
- **depth（行数）**：使用多少个 hash 函数
- **width（列数）**：每个 hash 函数映射的范围    
每个元素通过多个 hash 函数映射到数组的不同位置，每个对应位置的计数值会被累加。

---

操作过程

1. 添加元素（add(x)）：
```
for each row i:
    pos = hash_i(x)
    count[i][pos] += 1

```



2 查询频率估计（estimate(x)）：
```
minCount = +∞
for each row i:
    pos = hash_i(x)
    minCount = min(minCount, count[i][pos])
return minCount
```
由于可能有哈希冲突，CMS 返回的是多个位置中的最小值，作为元素出现频率的近似值。


假设我们有：
- `width = 5`, `depth = 2`
- 插入数据流：`[a, b, a, c, a, b]`
我们会用两个哈希函数分别计算位置，把 `a` 映射到两行的两个列，并把计数加 1。   当我们想估算 `a` 出现的次数时，取这两个位置的最小值作为估计值。


---


误差控制（理论）
- **误差 ε ≈ e / width**
- **置信度 δ ≈ e^(−depth)**

你可以通过选择较大的 `width` 和 `depth` 来减少误差并提高置信度。


| 优点               | 说明                |
| ---------------- | ----------------- |
| ✅ 空间效率高          | 远比 hashmap 占用内存小  |
| ✅ 更新和查询都很快（O(1)） | 适合处理高速数据流         |
| ✅ 可合并            | 两个 CMS 可以直接合并计数矩阵 |

| 缺点             | 说明              |
| -------------- | --------------- |
| ❌ 存在误差（不能精确统计） | 尤其是频率很低的元素容易被高估 |
| ❌ 无法删除元素       | 只能加不能减          |
| ❌ 依赖多个哈希函数     | 需设计合适的哈希函数避免冲突  |

- 🔹 网络数据流中的热门 IP 统计
- 🔹 网站用户行为统计（点击、访问等）
- 🔹 大型日志分析（找出最频繁的关键词）
- 🔹 内存受限的边设备数据处理（如 IoT）


| 特性    | Count-Min Sketch 描述  |
| ----- | -------------------- |
| 类型    | 概率性数据结构（Approximate） |
| 用途    | 元素频率估计               |
| 插入复杂度 | O(1)                 |
| 查询复杂度 | O(1)                 |
| 精确性   | 不保证，误差可控             |
| 存储效率  | 高，适合大规模数据流           |





---

Wir gucken uns zuerst den Count-Min Sketch an. Ein Count-Min Sketch besteht aus einem zweidimensionalen Array, welches über seine Breite und Höhe definiert ist. Alle Einträge des Arrays werden bei der Initialisierung des Count-Min Sketches auf 0 initialisiert.

一个 Count-Min Sketch 是一个 **二维数组**，其结构由**宽度（宽）和高度（高）**定义。在 Count-Min Sketch 初始化时，**数组中所有的元素都会被初始化为 0**。


Zuerst erstellen wir solch einen Count-Min Sketch und geben diesen aus.
```
cm = CountMinSketch(width=10, depth=3)
print(cm)
```


```
Count-Min Sketch
depth = 3 width = 10
[[0 0 0 0 0 0 0 0 0 0]
 [0 0 0 0 0 0 0 0 0 0]
 [0 0 0 0 0 0 0 0 0 0]]
```

---

Zusätzlich zum Array enthält ein Count-Min Sketch auch eine Menge von Hash-Funktionen*. Diese Hash-Funktionen bilden die Eingabe auf den Bereich der Sketch-Breite ab, d.h. auf das Intervall

. Die Anzahl der Hash-Funktionen entspricht der Anzahl der Reihen im Sketch, da wir später unterschiedliche Hash-Funktionen pro Reihe verwenden möchten.

Im Folgenden betrachten wir das `hashFunctions` Objekt, das beim Initialisieren des Sketches bereits erstellt wurde, und wenden es auf ein paar Eingaben an.

*Eine mathematische Funktion, die eine Eingabe beliebiger Größe auf eine feste Ausgabegröße abbildet.


```
hashFunctions = cm.get_hash_functions()
print(hashFunctions.hash("Hello World"))
print(hashFunctions.hash("Hello World Studis"))
print(hashFunctions.hash("Werder Bremen"))
```

```
[1 9 3]
[8 8 0]
[0 8 5]
```

---

Wir sehen, dass wir immer 3 Hashwerte erhalten, welche im Bereich zwischen 0 und 9 liegen. Die Werte zweier Hash-Funktionen können dabei per Zufall auch identisch sein.

Wenn wir die gleichen Eingaben nun an die `update`-Funktion unseres Count-Min Sketch übergeben, dann werden genau diese Einträge jeweils um eins erhöht, da wir diese Eingaben nun einmal mehr als zuvor gesehen haben.

```
cm.update("Hello World")
print(cm)
print("\n")
cm.update("Hello World Studis")
print(cm)
print("\n")
cm.update("Werder Bremen")
print(cm)
```



```
Count-Min Sketch
depth = 3 width = 10
[[0 1 0 0 0 0 0 0 0 0]
 [0 0 0 0 0 0 0 0 0 1]
 [0 0 0 1 0 0 0 0 0 0]]


Count-Min Sketch
depth = 3 width = 10
[[0 1 0 0 0 0 0 0 1 0]
 [0 0 0 0 0 0 0 0 1 1]
 [1 0 0 1 0 0 0 0 0 0]]


Count-Min Sketch
depth = 3 width = 10
[[1 1 0 0 0 0 0 0 1 0]
 [0 0 0 0 0 0 0 0 2 1]
 [1 0 0 1 0 1 0 0 0 0]]
```

---

Da es in einer Reihe zu mehreren Kollisionen zwischen Hash-Werten kommen kann, kann dies das Ergebnis beeinträchtigen. Deswegen haben wir mehrere Reihen mit unterschiedlichen Hash-Funktionen, um die Wahrscheinlichkeit zu erhöhen, dass in einer dieser Reihen die Anzahl der Kollisionen möglichst minimal ist.

Wenn wir Werte approximieren möchten, die wir bereits gesehen oder beobachtet haben, verwenden wir erneut die Hash-Funktion, betrachten die Indizes und wählen den kleinsten Wert aus, da hier die minimale Anzahl an Kollisionen vorliegt. Wir können somit nur über-approximieren.

Wenn wir den Wert “Werder Bremen” hinzufügen, gibt es zwar in der dritten Reihe eine Kollision, aber in den anderen Reihen nicht. Indem wir den Index mit der geringsten Anzahl von Kollisionen wählen, approximieren wir die Häufigkeit von “Werder Bremen” immer noch genau mit 1.


```
cm.query("Werder Bremen")

np.int32(1)
```


## 3.3 Bloom-Filter 

Bloom Filter 是一种**空间效率极高的概率型数据结构**，用于判断某个元素是否存在于集合中。
- 可能会 **误判存在**（false positive），但**不会误判不存在**。
- 使用多个哈希函数将元素映射到一个位数组中。

属于 **Sketches**：
- 它不存储实际数据，而是**通过哈希函数构建的位图 sketch**（草图）来近似表示集合。
- 所以它是对整个数据集合的“压缩表示”，符合 Sketch 的定义。


**Bloom Filter（布隆过滤器）** 是一种 **空间效率极高的概率型数据结构**，用于测试某个元素是否在一个集合中。
- ✅ **可以判断一个元素“可能在”集合中**
- ❌ **不能判断一个元素“一定不在”集合中**
- ⚠️ 存在 **假阳性（false positives）**：有时会说元素在集合中，但其实不在


|场景|说明|
|---|---|
|浏览器缓存|判断是否访问过某个 URL|
|数据库查询前过滤|判断 key 是否可能存在，减少不必要的磁盘 IO|
|分布式系统中的去重|去重传输或存储的数据|
|垃圾邮件过滤、黑名单检查等|判断邮件地址是否属于垃圾邮件来源|


Bloom Filter 使用一个：

- ✅ **位数组（bit array）**：长度为 `m`，初始为全 0
- ✅ **k 个哈希函数**：每个哈希函数将一个元素映射到 `0 ~ m-1` 范围的某个位置

---

操作过程

1 插入元素 `x`
1. 用 `k` 个哈希函数分别计算 `x` 的位置 `h1(x), h2(x), ..., hk(x)`
2. 将这些位置上的位设为 `1`

```
插入 x → 将 bit[h1(x)]、bit[h2(x)] ... bit[hk(x)] 置为 1
```

2 查询元素 `y`

1. 计算 `k` 个哈希位置 `h1(y), ..., hk(y)`
2. 查看这些位置是否 **全部为 1**
- 如果是：返回 “可能存在”    
- 如果不是：返回 “一定不存在”

```
查询 y：
    若任一 bit[hi(y)] == 0 → 一定不在
    若全部 bit[hi(y)] == 1 → 可能在（可能是假阳性）

```

---

False Positives（假阳性）

Bloom Filter 会告诉你一个元素“可能在集合中”，即使它实际并不在。这是因为
- 多个元素可能会映射到相同的 bit 位置，产生冲突
- 越多元素插入，bit 数组中 1 越多，冲突概率增加
永远没有 False Negatives（假阴性）
Bloom Filter 永远不会漏报 —— 如果某个元素在集合中，它一定会被检测为“可能在”。


---

Bloom Filter 的假阳性率与以下参数有关：

|参数|含义|
|---|---|
|`m`|位数组大小|
|`k`|哈希函数数量|
|`n`|插入的元素数量|
![](image/Pasted%20image%2020250625223305.png)

---
例子

假设：

- `m = 10` 位
    
- `k = 3` 个哈希函数
    
- 插入元素：apple、banana、cherry
    

插入后位数组可能变成：

```
bit array: [0, 1, 1, 0, 1, 0, 1, 0, 0, 0]

```

如果你现在查询 grape，它可能刚好也被映射到这些为 1 的位上，那么你会得到一个 false positive。


---

|优点|说明|
|---|---|
|✅ 空间效率极高|不存储元素本身|
|✅ 插入 & 查询速度极快|都是 O(k) 操作|
|✅ 可用于大规模数据集|很适合流式数据、分布式系统|

| 缺点                       | 说明               |
| ------------------------ | ---------------- |
| ❌ 可能有假阳性（false positive） | 查询结果不是 100% 准确   |
| ❌ 不支持删除（普通版）             | 删除元素可能会误删其他元素的数据 |
| ❌ 无法获取元素原始内容             | 仅判断是否“可能存在”      |


|变种|特点|
|---|---|
|Counting Bloom Filter|用计数数组代替 bit，可以支持删除|
|Scalable Bloom Filter|动态扩容，适应元素数量变化|
|Compressed Bloom Filter|减少网络传输成本|



---

Wir kommen nun zum Bloom-Filter. Im Gegensatz zum Count-Min Sketch besteht der Bloom-Filter nur aus einem eindimensionalen Array, das bei der Initialisierung, genauso wie der Count-Min Sketch, in jedem Eintrag auf 0 gesetzt wird. Oft wird dies auch als Bitvektor (oder Bitmap auf Englisch) bezeichnet und entsprechend implementiert, da der Wertebereich nur aus 0 und 1 besteht. Zusätzlich dazu hat der Bloom-Filter eine oder mehrere Hash-Funktionen, die ähnlich wie beim Count-Min Sketch hashen, jedoch im Gegensatz zum Count-Min Sketch ist die Anzahl der Hash-Funktionen ein Parameter.

Wir starten, indem wir einen Bloom-Filter mit 10 Bits und 2 Hash-Funktionen initialisieren.

```
bf = BloomFilter(n_bits=10, n_hash_functions=2)
print(bf)
```

```
BloomFilter
number of bits = 10 number of hash functions = 2
[0 0 0 0 0 0 0 0 0 0]
```

Nun fügen wir ein paar Werte ein. Genau wie beim Count-Min Sketch wird jeder Eingabewert pro Hash-Funktion einmal gehasht. Allerdings haben wir beim Bloom-Filter keinen Counter, sondern ein Bit pro Eintrag. Wir prüfen einfach, ob das Bit derzeit auf Null gesetzt ist, und falls ja, setzen wir es auf 1. Wenn das Bit bereits auf eins gesetzt ist, lassen wir es unverändert und fahren fort.

Lassen Sie uns also ein paar Fußballvereine der 1. Bundesliga als Datenstrom wie in den vorherigen Aufgaben hinzufügen.


```
ds = DataStream().from_collection(["Werder Bremen", "1. FC Union Berlin", "SC Freiburg"])
for tupel in ds:
    bf.update(tupel)
print(bf)
```



```
BloomFilter
number of bits = 10 number of hash functions = 2
[1 1 0 0 1 0 1 0 1 0]
```


---

Während der Count-Min Sketch unter anderem die Häufigkeit von einzelnen Instanzen approximiert (Point-Query), löst der Bloom-Filter das “Set-Problem” approximativ. Der Bloom-Filter kann zwar nie garantiert sagen, ob er ein konkretes Element schon einmal gesehen hat, kann aber mit absoluter Garantie sagen, dass er ein Element noch nie gesehen hat. Testen wir diese Eigenschaft mit ein paar Anfragen.

```
print(bf.query("Werder Bremen"))
print(bf.query("Hertha"))
```


```
True
True
```


Wir sehen, dass Werder Bremen zu den Mannschaften der 1. Bundesliga gehört, die unser Bloom-Filter möglicherweise schon einmal gesehen hat. Das Gleiche gilt jedoch auch für Hertha BSC, obwohl wir anhand unserer Daten konkret wissen, dass wir diesen Wert noch nicht gesehen haben. Dieses Verhalten liegt erneut an den Kollisionen im Array. Wenn wir uns die Hashwerte von Hertha BSC anschauen, sehen wir, dass die korrespondierenden Bits durch andere Vereine bereits auf 1 gesetzt wurden, die wir tatsächlich beobachtet haben. Somit können wir nur garantieren, dass wir etwas **möglicherweise** schon einmal gesehen haben.

```
hashFunctionsbf = bf.get_hash_functions()
print(hashFunctionsbf.hash("Hertha"))
```

```
[4 4]
```

----


Was wir jedoch definitiv ausschließen können, ist, dass der Hamburger Sport Verein zu den Vereinen der 1. Bundesliga gehört, die wir bereits gesehen haben.

```
print(hashFunctionsbf.hash("Hamburger Sport Verein"))
bf.query("Hamburger Sport Verein")
```

```
[2 1]
```


```
False
```

Da nicht alle Bits des Hamburger Sport-Verein auf 1 gesetzt sind, können wir garantieren, dass dieser noch nie beobachtet worden ist. Wäre der Hamburger Sport-Verein jemals gesehen worden, dann wäre dies der Fall.


## 3.4 Reservoir Sample

Reservoir Sampling 是一种算法，**用于从大小未知的数据流中等概率抽取固定数量的样本**，通常大小为 k。
- 即使你不能事先知道数据总量，也能保证所有样本被选中的概率相等。


属于 **Samples**：
- 它明确抽取数据中的“代表样本”，而不是构建频率图或集合草图。
- 所以是典型的“采样”方法。


| 场景                    | 描述               |
| --------------------- | ---------------- |
| 大数据处理                 | 数据量太大，无法全部加载到内存中 |
| 流式数据（Streaming）       | 数据是实时到达的，不能回头查看  |
| 在线学习（Online Learning） | 训练模型时需要从数据流中选样本  |


---


算法原理（以 k=1 为例）

假设你要从一个未知长度的流中，**随机抽取一个元素（k=1）**，使得每个元素被选中的概率是一样的（即 `1/n`）。

Step by Step：
1. 初始化一个变量 `res` 为第一个元素。
2. 从第 i（i ≥ 2）个元素开始，每次以概率 `1/i` 替换 `res`。
    - 换句话说，以 `1/i` 的概率选中第 i 个元素替换 `res`
    - 否则保留当前的 `res`

这样做的结果是：**每个元素都有 `1/n` 的概率最终被选中**，满足均匀随机的要求。



1. 第一个元素（i=1）

直接加入池中（因为池子还没满）
→ 被选中概率：1
2. 第二个元素（i=2）

以 1/2 的概率替换当前池中元素
→ 那么原来元素保留的概率是 1 * (1 - 1/2) = 1/2
→ 新元素被选中的概率是 1/2
3. 第三个元素（i=3）

以 1/3 的概率替换池中元素

    原来第一个元素保留概率：1 * (1 - 1/2) * (1 - 1/3) = 1 * 1/2 * 2/3 = 1/3

    新元素被选中概率：1/3

总结：

对第 i 个元素，选中概率是 1 / i；
而第 j 个早期元素能活到最后一轮，必须经历多轮“幸存”：
P(元素 j 被保留)=1⋅(1−1j+1)⋅(1−1j+2)⋯(1−1n)

![](image/Pasted%20image%2020250625224715.png)

→ 这个乘积等于 1/n，最终所有元素被选中的概率均为 1/n


----

通用算法（抽取 k 个样本）
当你想从数据流中抽取 **k 个元素** 时：
输入：数据流（长度未知），要选的样本数量 `k`
1. 创建大小为 `k` 的 reservoir 数组
2. 将前 `k` 个元素直接放入 reservoir 中
3. 对于第 `i` 个元素（i ≥ k）：
    - 生成一个随机数 `j`，范围为 `[0, i]`
    - 如果 `j < k`，则用当前元素替换 `reservoir[j]`
- 那么池中已有的元素，在这一轮被替换的概率是 `1/i`（因为：被选中概率 `k/i` × 被替换中某一元素的概率 `1/k`）
    - 所以原先池中每个元素保留的概率是 `1 - 1/i`
    - 那么某个前面元素 `x` 的总保留概率是： 经过数学归纳可以证明这个乘积最终等于 `k/n`
        - ![](image/Pasted%20image%2020250625224756.png)



---

这个算法的核心思想是：

- 对于前 k 个元素，先占住 k 个坑位
- 对于后续第 i 个元素，它被选中的概率是 `k/i`    
- 同时确保原先的元素被保留的概率乘积最终也是 `k/n`

因此，**最终所有 n 个元素都有相同的被选中概率：`k/n`**。


---


|优点|描述|
|---|---|
|✅ 空间复杂度为 O(k)|只需要 k 个空间保存样本|
|✅ 不需要知道总长度|非常适合处理数据流|
|✅ 样本具有均匀性|每个元素都有相等概率被抽中|

|局限|描述|
|---|---|
|❌ 随机性依赖质量好伪随机|否则样本可能不均匀|
|❌ 不支持加权采样|所有元素被采样概率一样|
|❌ 只能保留样本，不能估计频率|不是用于频率估计的结构（如 Count-Min Sketch）|



---

Zuletzt schauen wir uns das Reservoir Sample an, das zur Klasse der Samples, also der Stichproben, gehört. Im Gegensatz zu den zuvor präsentierten Datenstrukturen arbeitet das Reservoir Sample im Hintergrund nicht mit Hashfunktionen, sondern mit Zufallszahlen. Da es sich bei Samples um eine Stichprobe aus dem gesamten Datensatz handelt, sind Samples eindimensionale Arrays. Solange das Reservoir Sample noch nicht vollständig gefüllt ist, wird jedes Element einfach in das Array eingetragen.

Wir erstellen ein Reservoir Sample mit einem Array, das 20 Einträge hat, und füllen es vorerst mit genau 20 Werten.


```
rs = ReservoirSample(20)

for i in range(20):
    rs.update(i)

print(rs)
```

```
ReservoirSample
sample size = 20
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
```



Sobald das Array gefüllt ist, wird per Zufall entschieden, ob ein Element noch hinzugefügt wird und welcher Wert dafür weichen muss. Im Folgenden fügen wir weitere Werte ein, setzen dieses Mal jedoch den Debug-Parameter der Update-Funktion, um Informationen während des Updates zu erhalten.

```
rs.update(20, debug=True)
print(rs)
```

```
Random number:  0.6369616873214543
Fraction (Sample size / Number of processed elements) =  0.9523809523809523
Index:  10
ReservoirSample
sample size = 20
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 20, 11, 12, 13, 14, 15, 16, 17, 18, 19]
```


Jedes Element soll die gleiche Wahrscheinlichkeit haben, im Sample aufzutauchen. Da wir nun 21 Elemente gesehen haben, bestimmen wir die Wahrscheinlichkeit, dass das 21. Element ins Sample aufgenommen wird. Diese beträgt 95 %. Zusätzlich dazu generieren wir eine Zufallszahl im Intervall von 0 bis 1, um zu bestimmen, ob es einer der fünf Prozent der Fälle ist, in denen wir diesen Wert nicht aufnehmen sollten, oder ob wir das Element in das Sample aufnehmen sollen. Wenn wir es aufnehmen sollen, wird eine weitere Zufallszahl im Intervall [0,Breite - 1] erzeugt, um zu bestimmen, welches Element im Sample ersetzt wird.
