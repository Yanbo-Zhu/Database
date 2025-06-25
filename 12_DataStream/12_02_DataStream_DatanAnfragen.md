
# 1 

In der vergangenen Woche haben wir angefangen, uns mit Datenströmen auseinanderzusetzen. Dabei haben wir gesagt, dass Datenströme potenziell unendlich lang sind und dass die Gültigkeit dieser Daten nur kurzlebig ist. Wenn wir jedoch Anfragen stellen wollen, bevor die Daten ihre Gültigkeit verlieren, ein Datenstrom aber potenziell unendlich lang ist, so müssen wir diesen unterteilen und Anfragen auf einzelnen Abschnitten eines Datenstroms stellen. Für diese Partitionierung führen wir heute das sogenannte Windowing ein, welches anhand der Window Definition den Datenstrom partitioniert. Die Anfragen werden dann auf einer pro Window Basis ausgeführt. Deswegen ist das Datenstrom-Paradigma durch wiederkehrende Anfragen geprägt. Somit werden im Data Streaming die Rollen der Daten und Anfragen vertauscht. Während in einer relationalen Datenbank die Daten ruhen und sich generell nicht viel verändern, sind die Anfragen “in Bewegung”, da diese nur ad-hoc/spontan gestellt werden. Im Data Streaming bewegen sich stattdessen die Daten mit einer hohen Geschwindigkeit, aber die Anfragen ruhen, da diese einmal gestellt werden und dann wiederkehrend ausgeführt werden.

Des Weiteren haben wir in der vergangenen Woche nur Tuple basierte Streams kennengelernt. In dieser Woche haben wir dies durch zeitbasierte Streams ergänzt, welche wir in diesem Tutorium entsprechend wiederholen. Zuletzt wurden dann auch noch approximative Methoden (Synopsen) vorgestellt, welche ebenfalls in diesem Tutorium wiederholt und geübt werden.


# 2 Einführung

Wir verwenden erneut die eigens definierte Streaming-API der letzten Woche. Dieses Mal ergänzen wir die imports um die zuvor erwähnten zeitbasierten Streams, als auch die Synopsen (engl. Synopses). Für eine tatsächliche Echtzeitverarbeitung bräuchte man dedizierte Systeme wie z.B. Apache Flink, die mit einer größeren Komplexität einhergehen.

Die Klassen und Methoden der API sind ausführlich dokumentiert. Siehe: [https://dima.gitlab-pages.tu-berlin.de/isda/isda-streaming/isda_streaming.html](https://dima.gitlab-pages.tu-berlin.de/isda/isda-streaming/isda_streaming.html)


```
import random

from isda_streaming.data_stream import DataStream, TimedStream
from isda_streaming.synopsis import BloomFilter, CountMinSketch, ReservoirSample
```


# 3 Windowing


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



# 4 Synopsen

Im Folgenden beschäftigen wir uns mit approximativen Datenmethoden, die im Englischen als Synopses bezeichnet werden. Konkret sind diese Methoden in vier Klassen unterteilt: Histogramme, Samples, Sketches und Wavelets. Diese Methoden sind sehr beliebt und finden vor allem im Bereich des Data Streamings viele Anwendungen, in denen die Datenmenge so massiv sein kann, dass Genauigkeit zugunsten von Rechenleistung geopfert wird. Des Weiteren haben sie einen sehr geringen Speicherbedarf, da sie die Daten komprimieren. Im Folgenden wiederholen wir drei solcher Datenstrukturen, die Sie bereits in der Vorlesung kennengelernt haben. Dabei gehören der **Bloom-Filter** und der **Count-Min Sketch** zur Klasse der Sketches, während das **Reservoir Sample** zur Klasse der Samples gehört. Wir betrachten also explizit keine Histogramme oder Wavelets.

Bei weiterem Interesse zu diesem Thema können Sie das Buch “Synopses for Massive Data: Samples, Histograms, Wavelets, Sketches” konsultieren [[1]](https://dsf.berkeley.edu/cs286/papers/synopses-fntdb2012.pdf).


下面我们将探讨**近似数据方法**，英文中称为 **Synopses（概要）**。具体来说，这些方法被划分为四类：**直方图（Histograms）**、**抽样（Samples）**、**草图（Sketches）** 和 **小波（Wavelets）**。这些方法非常流行，尤其在 **数据流（Data Streaming）** 领域中有广泛的应用——在这些场景中，数据量可能大到必须**牺牲准确性以换取计算效率**。

此外，它们的另一个优势是**极低的内存需求**，因为它们对数据进行了压缩处理。

在下文中，我们将回顾三种你在课程中已经学过的数据结构。其中：

- **Bloom Filter** 和 **Count-Min Sketch** 属于 **Sketches（草图）** 类；
    
- **Reservoir Sample（蓄水池抽样）** 属于 **Samples（抽样）** 类。
    

我们将不会讨论直方图（Histograms）和小波（Wavelets）。

如果你对该主题有更深入的兴趣，可以查阅书籍《**Synopses for Massive Data: Samples, Histograms, Wavelets, Sketches**》111。


## 4.1 Count-Min Sketch

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


## 4.2 Bloom-Filter 

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


## 4.3 Reservoir Sample

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
