
# 1 Einführung


**Datenströme** oder auch im Englischen **Data Streams**, sind ein weiteres Paradigma, mittels dessen wir Daten verarbeiten können. Konzeptuell ist ein Datenstrom ein potenziell endloser Strom an Daten, der in ein System hineinfließt, dort verarbeitet wird, um Wissen zu extrahieren und dann wieder herausfließt. 

Das Paradigma des Data Streaming kommt unter anderem dann zum Einsatz, wenn wir sehr viele Daten haben, welche wir in Echtzeit verarbeiten wollen, da sie mit der Zeit ihre Gültigkeit oder Bedeutung verlieren. Dies kann z.B. die live Auswertung von Sensoren in einem Werk oder in einem Krankenhaus beinhalten, mittels dessen man die Maschinen oder den/die Patient*in kontrolliert.  

Da die Daten schnell ihre Gültigkeit verlieren, speichern Systeme, welche auf Datenströmen arbeiten (Stream Processing Engine (SPE)) die Daten nicht ab. Dies bedeutet nicht, dass Sie die Daten gar nicht mehr abspeichern können/sollten. Immerhin könnten manche dieser doch historisch von Interesse sein, z.B. um herauszufinden, weshalb eine Maschine kaputtgegangen ist oder welche Werte ein/eine Patient*in hatte, bevor sich sein/ihr Zustand verschlechtert hat. Stattdessen ergänzen sich **Relationale Datenbanken** und das **Data Streaming** gegenseitig, dass auch keines der beiden wichtiger ist oder ähnliches.

Da uns keine Möglichkeit bekannt ist, eine bestehende Stream Processing Engine leicht in Python einzubinden, wie es für Relationale Datenbank Management Systeme (RDBMS) der Fall war, arbeiten wir in dieser und der kommenden Woche mit einer eigenen, in Python geschriebenen [Streaming API](https://pypi.org/project/isda-streaming/). Diese API wird benutzt, um Konzepte von Datenströmen zu üben. Sie simuliert eine Echtzeitverarbeitung lediglich. Für eine tatsächliche Echtzeitverarbeitung bräuchte man dedizierte Systeme wie z.B. [Apache Flink](https://flink.apache.org), die mit einer größeren Komplexität einhergehen.

Die Klassen und Methoden der API sind ausführlich dokumentiert. Siehe: [ISDA Streaming Dokumentation](https://dima.gitlab-pages.tu-berlin.de/isda/isda-streaming/isda_streaming.html)



Zuerst importieren wir die API.

from isda_streaming.data_stream import DataStream

Nun definieren wir uns einen ersten Datenstrom und geben uns diesen aus. Dabei verwenden wir die _from_collection_ Funktion, welche die eingegebene Liste in einen Datenstrom umwandelt.

```
ds = DataStream().from_collection([1, 2, 3, 4, 5])
print(ds)
```

`DataStream([1, 2, 3, 4, 5])
`
Auch mit einer Schleife können wir über den Datenstrom iterieren.

```
for x in ds:
    print(x)

1
2
3
4
5
```


# 2 Map


Als Nächstes wenden wir den in der Vorlesung vorgestellten **map**-Operator auf dem Datenstrom an. Dieser ist eine sogenannte “second order function”. Sie erhält als Argument selbst eine Funktion und wendet diese auf jedes Element des Datenstroms an. Dabei wird für jeden Eingabewert ein Ausgabewert produziert.  
Somit gibt es die Einschränkung, dass die Funktion, welche an **map** übergeben wird, nur einen **einzelnen Eingabewert** erhalten, als auch nur einen **einzelnen Ausgabewert** produzieren darf. Der **map**-Operator verändert somit nicht die Kardinalität des Datenstroms.

In diesem Beispiel wird jedes Element des Datenstroms mit 2 multipliziert.

```
def times_two(x):
    return x * 2

output_stream = ds.map(times_two)
print(output_stream)


DataStream([2, 4, 6, 8, 10])
```


# 3 Flat Map

Als Nächstes verwenden wir die **flat_map**. Diese unterscheidet sich von der **map** nur in der Hinsicht, dass sie für jedes Eingabeelement null, ein oder mehrere Elemente erzeugt. Daher kann sich die Anzahl der Elemente im Eingabe- und Ausgabe-Datenstrom beim **flat_map**-Operator unterscheiden, je nachdem, wie die jeweils übergebene Funktion definiert ist.

Um die Funktionalität zu demonstrieren, definieren wir die Funktion _split_words_, die einen übergebenen String anhand seiner Leerzeichen in einzelne Wörter aufteilt.


```

def split_words(x: str):
    if x == "":
        return []
    return x.split(" ")


input_stream = DataStream().from_collection(["Hello world!", "ISDA", ""])
output_stream = input_stream.flat_map(split_words)
print(output_stream)


```


```
DataStream(['Hello', 'world!', 'ISDA'])
```

# 4 Filter


Der **filter** ist ebenfalls eine “second order function”. Dabei gibt es erneut eine Einschränkung, die zu beachten ist: Der Rückgabewert muss ein Boolean sein. **Filter** wendet die Funktion dann auf jedes Element im Datenstrom an und sortiert Elemente aus dem Eingabedatenstrom anhand der selber implementierten booleschen Logik aus. Zurückgegeben wird von dem **Filter** dann ein Datenstrom, welcher nur Elemente beinhaltet, für die die boolesche Funktion _True_ ausgibt.

Um die Funktionalität zu demonstrieren, definieren wir die boolesche Funktion _keep_positives_. Diese Funktion gibt für alle Zahlen größer als 0 _True_ zurück und ansonsten _False_.



```
def keep_positives(x):
    if x > 0:
        return True
    return False

input_stream = DataStream().from_collection([-1, 7, 0, 4, 5, -6])
output_stream = input_stream.filter(keep_positives)
print(output_stream)


DataStream([7, 4, 5])
```

# 5 Reduce

Der **reduce**-Operator kombiniert jedes Element eines Datenstroms anhand der übergebenen “second order function” mit dem letzten reduzierten Wert und speichert das Ergebnis in einem neuen Datenstrom. Dabei wird das erste Element des Eingabe-Datenstroms als initialer reduzierter Wert betrachtet.

Um die Funktionalität zu demonstrieren, definieren wir eine Funktion namens _sum_all_, welche das aktuelle Element mit dem letzten reduzierten Wert summiert.


```
def sum_all(value1, value2):
    return value1 + value2

input_stream = DataStream().from_collection([-1, 7, 0, 4, 5, -6])
output_stream = input_stream.reduce(sum_all)
print(output_stream)


DataStream([-1, 6, 6, 10, 15, 9])
```

# 6 Key By

Das **key_by** dient dem gleichen Zweck wie das **GROUP BY** in SQL und gruppiert die Elemente des Datenstroms basierend auf dem Rückgabewert der übergebenen Funktion.

Um die Funktionalität zu demonstrieren definieren wir zuerst einen Datenstrom von Tupeln. Diesen gruppieren wir dann anhand des **key_by**, mittels der Funktion _key_first_value_, welche Werte anhand des ersten Feldes gruppiert. Es ist wichtig zu beachten, dass die Ausgabe des **key_by**-Operators kein “normaler” Datenstrom mehr ist, sondern ein _KeyedStream_.


```
def key_first_value(x):
    return x[0]

input_stream = DataStream().from_collection([("a", 1), ("a", 7), ("b", 4), ("a", 8)])
keyed_stream = input_stream.key_by(key_first_value)
print(keyed_stream)


KeyedStream({'a': [('a', 1), ('a', 7), ('a', 8)], 'b': [('b', 4)]})

```
---

Nachdem wir eine Gruppierung mithilfe des **key_by** durchgeführt haben, können wir die Daten der jeweiligen Gruppierungen mittels des **reduce**-Operators aggregieren. Da der Eingabe-Datenstrom ganze Tupel beinhaltet (welche mehrere Felder bzw. Attribute haben), müssen wir dem **reduce**-Operator auch eine Funktion namens _sum_all_tuples_ übergeben, die korrekt mit den Feldern der Tupel im Datenstrom arbeitet.

Beachte bitte auch, dass der **reduce**-Operator auf allen Partitionen des Datenstroms angewendet wird.


```
def key_first_value(x):
    return x[0]


def sum_all_tuples(tuple1, tuple2):
    return (tuple1[0], tuple1[1] + tuple2[1])


input_stream = DataStream().from_collection([("a", 1), ("a", 7), ("b", 4), ("a", 8)])
keyed_stream = input_stream.key_by(key_first_value)
print(keyed_stream)
output_stream = keyed_stream.reduce(sum_all_tuples)
print(output_stream)
```

```
KeyedStream({'a': [('a', 1), ('a', 7), ('a', 8)], 'b': [('b', 4)]})
KeyedStream({'a': [('a', 1), ('a', 8), ('a', 16)], 'b': [('b', 4)]})
```

Alternativ kann man auch eine Abfolge von Operatoren hintereinander ausführen.

```
output_stream = input_stream.key_by(key_first_value).reduce(sum_all_tuples)
print(output_stream)

KeyedStream({'a': [('a', 1), ('a', 8), ('a', 16)], 'b': [('b', 4)]})

```

# 7 Dataflow-Pipeline


Eine Abfolge von Operatoren wird als **Dataflow-Pipeline** bezeichnet. Hier ist ein Beispiel, wie man eine **Dataflow-Pipeline** als Funktion definiert (_pipeline_) und diese auf den Eingabe-Datenstrom anwendet.


```
def pipeline_aufgabe_1(data_stream: DataStream):
    return data_stream.key_by(key_first_value).reduce(sum_all_tuples)

output_stream = pipeline_aufgabe_1(input_stream)
print(output_stream)

KeyedStream({'a': [('a', 1), ('a', 8), ('a', 16)], 'b': [('b', 4)]})
```


# 8 Aufgabe 


```
from isda_streaming.data_stream import DataStream
import random

random.seed(84)  # wird benötigt, um bei allen die gleichen zufälligen Zahlen zu generieren
```


## 8.1 Aufgabe 1

Es soll ein Datenstrom initialisiert werden, der 10 zufällige Ganze Zahlen aus im Intervall `[-10,10]` enthält.

Dafür soll die Funktion _random.randint(startwert, stoppwert)_ genutzt werden, welche eine eine zufällige Zahl aus dem Intervall `[startwert, stoppwert]` generiert.

```java
# Your code here


random_number = []
for i  in in range(10):
    rand_val = random.randint(-10, 10)
    ramdom_number.append(rand_val)

// random_number = [random(-10,10) for i in range(10)]


ds = DataStream().from_collection(ramdom_number)

print(ds)

# wird benötigt um das Ergebnis mit dem erwarteten Ergebnis zu vergleichen
result_ds = DataStream().from_collection([-1, -9, 5, -10, 6, 0, -4, 4, 5, 7])
print("Does the result match the expected one?", ds == result_ds)
```

```
DataStream([])
Does the result match the expected one? False
```

## 8.2 Aufgabe 2

Ersetzen Sie jede 0 im Datenstrom durch -1 und 1.
```
ds = DataStream().from_collection([-1, -9, 5, -10, 6, 0, -4, 4, 5, 7])  
  
# Your code here  
def replace_zero(x):  
    if x == 0:  
        return [-1,1]  
    else:  
        return[x]  
  
replace_zero_ds = ds.flat_map(replace_zero)  
  
  
  
print(replace_zero_ds)  
  
# wird benötigt um das Ergebnis mit dem erwarteten Ergebnis zu vergleichen  
result_ds = DataStream().from_collection([-1, -9, 5, -10, 6, -1, 1, -4, 4, 5, 7])  
print("Does the result match the expected one?", replace_zero_ds == result_ds)
```



## 8.3 Aufgabe 3  
  
Erstellen Sie aus dem gegebenen Datenstrom zwei neue Datenströme, wobei der erste Datenstrom alle *geraden* ganzen Zahlen beinhaltet und der andere alle *ungeraden*.

```
ds = DataStream().from_collection([-1, -9, 5, -10, 6, -1, 1, -4, 4, 5, 7])  
  
# def select_odd(x):  
#     if x%2 == 0:  
#         return False  
#     else:  
#         return True  
#  
# def selesct_even(x):  
#     if x%2 == 0:  
#         return True  
#     else:  
#         return False  
  
def is_even(x):  
    return x % 2 == 0  
def is_odd(x):  
    return not is_even(x);  
  
  
# Your code here  
even_ds = ds.filter(is_even)  
odd_ds = ds.filter(is_odd)  
  
print("Datastream with even numbers:", even_ds)  
print("Datastream with odd numbers:", odd_ds)  
  
# wird benötigt um das Ergebnis mit dem erwarteten Ergebnis zu vergleichen  
result_ds = DataStream().from_collection([-10, 6, -4, 4])  
print("Does the even datastream match the expected one?", even_ds == result_ds)  
  
result_ds = DataStream().from_collection([-1, -9, 5, -1, 1, 5, 7])  
print("Does the odd datastream match the expected one?", odd_ds == result_ds)
```



## 8.4 Aufgabe 4  
  
Kategorisieren Sie die Werte des Datenstroms. Ersetzen Sie dabei alle atomaren Werte des Datenstroms mit einem Tupel. Dabei gibt das erste Feld des Tuples an ob der Wert positiv ('p') oder negativ ('n') ist und das zweite Feld beinhaltet nur noch den absoluten Betrag des ursprünglichen Wertes.   
  
**Beispiel:**    
```
even_ds = DataStream().from_collection([-10, 6, -4, 4])  
odd_ds = DataStream().from_collection([-1, -9, 5, -1, 1, 5, 7])  
  
# Your code here  
  
def create_tuple(x):  
    if x >= 0:  
        return ('p', abs(x))  
    else:  
        return ('n', abs(x))  
  
  
  
even_tuple_ds = even_ds.map(create_tuple)  
odd_tuple_ds = odd_ds.map(create_tuple)  
  
print("even_tuple_ds = ", even_tuple_ds)  
print("odd_tuple_ds = ", odd_tuple_ds)  
  
# wird benötigt um das Ergebnis mit dem erwarteten Ergebnis zu vergleichen  
result_ds = DataStream().from_collection([("n", 10), ("p", 6), ("n", 4), ("p", 4)])  
print(  
    "Does the even datastream with tuples match the expected one?",  
    even_tuple_ds == result_ds,  
)  
  
result_ds = DataStream().from_collection(  
    [("n", 1), ("n", 9), ("p", 5), ("n", 1), ("p", 1), ("p", 5), ("p", 7)]  
)  
print(  
    "Does the odd datastream with tuples match the expected one?",  
    odd_tuple_ds == result_ds,  
)


```



## 8.5 Aufgabe 5  
  
Bauen Sie nun eine **Dataflow-Pipeline**, welche die Ergebnisse der vorherigen Aufgaben kombiniert:   
  
Ersetzen Sie zunächst alle 0 Einträge im Datenstrom mit -1 und 1. Filtern Sie danach alle gerade Zahlen heraus um die übrigen Werte dann zu kategorisieren. Positive Werte sollen dabei erneut als Tupel mit 'p' kategorisiert werden und nur noch den absoluten Wert (zweites Feld) haben.

```
ds = DataStream().from_collection([-1, -9, 5, -10, 6, 0, -4, 4, 5, 7])  
  
# Your code here  
transformed_ds = ds.flat_map(replace_zero).filter(is_odd).map(create_tuple)  
  
print(transformed_ds)  
  
# wird benötigt um das Ergebnis mit dem erwarteten Ergebnis zu vergleichen  
result_ds = DataStream().from_collection(  
    [("n", 1), ("n", 9), ("p", 5), ("n", 1), ("p", 1), ("p", 5), ("p", 7)]  
)  
print(  
    "Does the odd datastream with tuples match the expected one?",  
    transformed_ds == result_ds,  
)
```


## 8.6 Aufgabe 6: Bluthochdruck  
Es steht ein Datenstrom zur Verfügung, der Informationen über den Blutdruck und den Puls eines Menschen beinhaltet. Unsere Sensoren messen dabei:  
- die Anzahl der **Herzschläge pro 10 Sekunden** und  
- den **Blutdruck in Pascal**, welcher zwei Werte annehmen kann:  
  - den systolischen Wert (wenn das Herz Blut in die Gefäße pumpt und sich zusammenzieht)  
  - den diastolischen Wert (wenn der Herzmuskel erschlafft)  
  
Weiterhin wissen wir, dass bei einem gesunden Erwachsenen:  
- Der Ruhepuls ungefähr zwischen **60 und 90 Schlägen pro Minute**,  
- der systolische Wert zwischen **110 und 130 mmHg** und  
- der diastolische Wert zwischen **80 und 90 mmHg** liegt.    
  
Leider liefern uns die Sensoren die Daten über Puls und Blutdruck in einem schlechten Format.    
- Der Blutdruck wird üblicherweise in mmHg (Millimeter Quecksilbersäulen) angegeben, **1 mmHg** entspricht dabei ungefähr **133,3 Pascal**.     
- Der Puls entspricht in der Regel den **Herzschlägen pro Minute**, hier jedoch erfolgt die Messung pro 10s.  
  
Die Daten aus dem Datenstrom sollen zuerst in das herkömmliche Format transformiert werden und danach soll jeder Wert eine der folgenden Kategorien bekommen:  
- **"puls"** für Puls-Werte im gesunden Rahmen  
- **"sys"** für systolische Werte im gesunden Rahmen  
- **"dia"** für diastolische Werte im gesunden Rahmen  
- **"anomaly"** für Blutdruck-Werte, die höher als 130mmHg sind  
- **"error"** für sonstige Werte  
  
Da der beobachtete Patient an Bluthochdruck leidet, soll der Datenstrom nur noch den Durchschnitt aller bisher gesehenen Blutdruckwerte enthalten, die über 130mmHg liegen.    
Anhand dieses Datenstroms könnte man nun Alarm schlagen, wenn der Durchschnitt auf einen zu hohen Wert ansteigt.  
  
**Hinweis:** Kategorisieren Sie zunächst die Daten, ansonsten kann es zu Überschneidungen bei den Wertebereichen kommen.


```python 
random.seed(42)  
data = [random.randint(10, 15) for _ in range(100)]  
data.extend(  
    [random.randint(14663, 17329) for _ in range(100)]  
    + [random.randint(17329, 21328) for _ in range(20)]  
)  
data.extend([random.randint(10664, 11997) for _ in range(100)])  
  
random.shuffle(data)  
  
ds = DataStream().from_collection(data)  
  
# 1) transform the measurements into the common format and assigns them a category  
  
def transform_data(x: int):  
    #transform the measurements into the common  format and assigns them a category  
    # input(value)    # output:(catagory, value)  
    if x >= 10 and x <=15:  
        return ("plus", x * 6)  
    elif x >=10664 and x <=11997:  
        return ("dia", x // 133.3)  
    elif x >= 14663 and x <= 17329:  
        return ("sys", x // 133.3)  
    elif x > 17329:  
        return ("anomly", x //133.3)  
  
    return ("error", x)  
  
categorized_ds = ds.map(transform_data)  
  
# comparison with expected result  
result_ds = DataStream().from_csv("../resources/09_data_streams/blutwerte_1.csv")  
print(  
    "Does the categorized datastream match the expected one?",  
    categorized_ds == result_ds,  
)  
  
# 2) filter the anomaly data. liefert anonmaly data aus  
  
def is_anomaly(x):  
    return x[0] == 'anomaly'  
  
filtered_ds = ds.map(transform_data).filter(is_anomaly)  # liefert anonmaly data aus  
print(filtered_ds)  
  
  
# comparison with expected result  
result_ds = DataStream().from_csv("../resources/09_data_streams/blutwerte_2.csv")  
print("Does the filtered datastream match the expected one?", filtered_ds == result_ds)  
  
# 3) calculate the average  
  
def add_counter(x):  
    return (x[1], 1)  
  
def sum_up(reduced_value, new_value):  
    return (reduced_value[0] + new_value[0], reduced_value[1] + new_value[1])  # intiale Wert von reduced_value wird im Default automatisch als wert value von inputed new_value gesetzt.   
def calc_avg(x):  
    return x[0]/x[1]  
  
avg_ds = ds.map(transform_data).filter(is_anomaly).map(add_counter).reduce(sum_up).map(calc_avg)  
  
  
  
  
# comparison with expected result  
result_ds = DataStream().from_csv("../resources/09_data_streams/blutwerte_3.csv")  
print(  
    "Does the datastream with the average values match the expected one?",  
    avg_ds == result_ds,  
)  
  
# print the latest average value in the datastream  
last_avg = list(avg_ds)[-1]  
print("The average value of all seen anomalies is", round(last_avg, 2))
```

