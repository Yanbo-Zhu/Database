


- Aspekte des praktischen betrieblichen Einsatzes von Datenbanksystemen
    - (Relationale) DB beschleunigen die Prozesse enorm!
    - (Relationale) DB erleichtern Datenanalyse enorm!
- Zwei Klassen von Datenbankanwendungen: Online Transaction Processing (OLTP) und Online Analytical Processing (OLAP)
- Einsatz von Datenbanksystemen in Decision-Support-Anwendungen
    - Data Warehouse für die Deskriptiven Fragen
    - Unterscheid zu Data Access: multidimensionales Datenmodell (Orthogonalität)
- Data Warehouse Architektur und Komponenten
    - ETL-Process: Extract Transform Load
    - Datenmodell: OLAP-Würfel (Cube)
    - DWH-Verteilung: Data Marts
    - OLAP-Operationen: Slicing, Dicing, feinere bzw. gröbere Partitionierung Drill down bzw. Roll up
- Datenbankentwurf für das Data Warehouse
    - Faktentabellen
    - Dimensionstabellen
    - Relationale Implementierung (ER, Star, Snowflake, Fullfact)


![](image/Pasted%20image%2020250609151030.png)


![](image/Pasted%20image%2020250609151043.png)

# 1 Zwei Klassen von DB-Anwendungen: OLTP vs OLAP

## 1.1 OLTP - Online Transaction Processing

− Das Konzept der Transaktion bietet die Möglichkeit, logische Verarbeitungseinheiten auf der Datenbank zu definieren. Unter einer Transaktion wird eine Folge von Datenbankoperationen (Einfügen, Löschen, Modifizieren oder Suche) verstanden, die insb. Bezüglich der Integritätsüberwachung als Einheit (atomar) angesehen wird.


− Formale Anforderungen an Transaktionen (ACID-Prinzip):
    − Atomicity (Atomarität)
    − Consistency (Konsistenz)
    − Isolation (Isolation)
    − Durability (Dauerhaftigkeit)


− Datenbanktheorie im Mehrbenutzerbetrieb
    − Ausführungsplan für Transaktionen (Schedule)
    − Serialisierbarkeit von Ausführungsplänen
    − Konfliktserialisierbarkeit
    − Sperren


![](image/Pasted%20image%2020250609143033.png)


![](image/Pasted%20image%2020250609143142.png)





## 1.2 Online Analytical Processing


− Neben der Verwaltung des Tagesgeschäftes werden Datenbanken auch zur Planung und Analyse eingesetzt:
    − Planung des Tagesgeschäfts (Welche Produkte werden wann, wo benötigt?)
    − Marketing (Was sind meine wichtigsten Kundensegmente?)
    − Marktanalyse (Wer sind meine Konkurrenten?)
    ➔ Analytische Datenbanken
− In analytischen Datenbanken liegt der Fokus auf der Auswertung aktueller und historischer Daten:
    − Annahme: Daten sind statisch (= keine Transaktionen).
    − Sammlung von Daten aus verschiedenen transaktionalen Systemen in einer Zentralen Datenbank (= Data Warehouse).
➔ Analytische Workloads werden unter dem Stichwort „Online Analytical Processing“ oder OLAP zusammengefasst.



## 1.3 OLAP vs OLTP

![](image/Pasted%20image%2020250609143214.png)


![](image/Pasted%20image%2020250609143423.png)


![](image/Pasted%20image%2020250609143437.png)

OLAP - Beispielanfragen
    − Welche Produkte hatten im letzten Jahr im Bereich Potsdam einen Umsatzrückgang um mehr als 10%?
    − Welche Produktgruppen sind davon betroffen?
− Welche Lieferanten haben diese Produkte?
    − Welche Kunden haben über die letzten fünf Jahre eine Bestellung über 50 Euro innerhalb von vier
    Wochen nach einem persönlichen Anschreiben aufgegeben?
    − Wie hoch waren die Bestellungen im Durchschnitt?
    − Wie hoch waren die Bestellungen im Vergleich zu den durchschnittlichen Bestellungen des jeweiligen Kunden in einem vergleichbaren Zeitraum?
    − Lohnen sich Mailing-Aktionen?
− Haben Zweigstellen einen höheren Umsatz, die gemeinsam gekaufte Produkte nebeneinander platzieren?
    − Welche Produkte werden überhaupt zusammen gekauft – und wo?





# 2 Data Warehouse - Architektur

“A Data Warehouse is a subject-oriented, integrated, non-volatile, and time- variant collection of data in support of management‘s decisions.“ 

− Subject-oriented: Verkäufe, Personen, Produkte, etc.
− Integrated: Erstellt aus vielen Quellen
− Persistent: Hält Daten unverändert über die Zeit
− Time-Variant: Vergleich von Daten über die Zeit


## 2.1 DWH - Datenquellen
− Analytische Datenbanken (Data Warehouses) werden periodisch mit den Daten aus operationalen (OLTP) Systemen gefüllt:
    − Lieferantendatenbanken:
        − Produktinformationen: Packungsgrößen, Farben, ...
    − Lieferbedingungen, Rabatte, Lieferzeiten, ...
    − Personaldatenbank:
        − Zuordnung Kassenbuchung auf Mitarbeiter
    − Stundenabrechnung, Prämien
    − Kundendatenbank:
        − Firmennamen, Telefonnummern und Post, E-Mail-Adressen (DSGVO konform)
    − Rechnungssystem:
        − Verbuchte Verkäufe & Einkäufe.
    − Weitere Vertriebswege:
        − Internet, Katalogbestellung, Verkaufsclubs, ...


## 2.2 Extract, Transform & Load (ETL)

− Daten die in das Data Warehouse integriert werden sollen durchlaufen zunächst den sogenannten ETL Prozess:
    − Extraktion: Die Daten werden aus dem Quellsystem exportiert.
    − Transformation: Die Daten werden in das Format des Data Warehouse umgewandelt. (Selektion relevanter Spalten, Anpassen des Formates und der Datentypen, Normalisierung & De-Duplikation, Vor- Aggregationen, Datenvalidierung etc.).
    − Laden: Die transformierten Daten werden in das Data Warehouse importiert.

![](image/Pasted%20image%2020250609143934.png)


## 2.3 DWH Architektur & Komponenten

![](image/Pasted%20image%2020250609144051.png)


− Physikalische Aufteilung variabel
    − Data Marts auf eigenen Rechnern (Laptop)
    − Staging Area auf eigenen Servern
    − Metadaten auf eigenem Server (Repository)


Arbeitsbereich
− Staging Area
    − Temporärer Speicher
    − Quellnahes Schema
− Motivation
    − ETL Arbeitsschritte effizienter implementierbar
        − Mengenoperationen, SQL
    − Zugriff auf Basisdatenbank möglich (Lookups)
    − Vergleich zwischen Datenquellen möglich
    − Filterfunktion: Nur einwandfreie Daten in Basisdatenbank übernehmen


Basisdatenbank
− Zentrale Komponente des DWH
    − Begriff „DWH“ meint oft nur die Basisdatenbank.
− Speichert Daten in feinster Auflösung
    − Einzelne Verkäufe
    − Einzelne Bons
− Historische Daten
− Große Datenmengen
    − Spezielle Modellierung
    − Spezielle Optimierungsstrategien


Data Mart
− Ein Data-Mart ist eine Kopie des Teildatenbestandes eines Data Warehouse, die für einen bestimmten Organisationsbereich oder eine bestimmte Anwendung oder Analyse erstellt wird.
− Gründe für das Arbeiten mit einem Data-Mart ("Kopien" aus dem Data-Warehouse):
    − die Notwendigkeit spezieller Datenstrukturen (die in dieser Form nicht im DW vorhanden sind) für bestimmte Analysen,
    − bessere Leistung: Verlagerung von Rechnerleistung auf einen anderen Rechner,
    − Eigenständigkeit der Anwender (z. B. Mobilität, Unabhängigkeit von anderen Organisationsbereichen),
    − Zugriffsschutz: Abgrenzung gegenüber anderen Nutzern







# 3 Mehrdimensionale Modellierung: Datenmodell für DWH



Das Mehrdimensionale Datenmodell
− In Data Warehouses werden Daten im sogenannten mehrdimensionalen Datenmodell dargestellt:
    − Spezielles Datenmodell für analytische Anfragen (ungeeignet für OLTP).
        − Fokus: Schnelle Aggregation und Analyse der Daten.
        − Kein Fokus: Normalisierung, verhindern von Redundanz, „Wartbarkeit“.

数据仓库中使用所谓的“多维数据模型”来表示数据：
    这是一种专门为分析型查询设计的数据模型（不适合用于 OLTP（联机事务处理））。
        重点：快速对数据进行聚合和分析。
        不强调：范式化、冗余消除、易维护性等传统关系数据库设计原则。

---

− Die Daten sind unterteilt in:
    − Fakten: Relevante, messbare Daten (z.B. Verkäufe).
    − Dimensionen: Beschreiben die Fakten (z.B. Produkt, Ort, Zeit).
    − Dimensionen sind in der Regel hierarchisch organisiert:
        − Zeit: All ⊃ Jahr ⊃ Quartal ⊃ Monat ⊃ Woche ⊃ Tag ⊃ Stunde ⊃ Minute
        − Ort: All ⊃ Kontinent ⊃ Region ⊃ Land ⊃ Landkreis ⊃ Ort ⊃ Filiale
        − Produkt: All ⊃ Produktkategorie ⊃ Unterkategorie ⊃ Produkt


Dimensionen
− Eindeutige Strukturierung des Datenraums
− Jede Dimension hat ein Schema
    − Tag, Woche, Jahr
    − Landkreis, Land, Staat
    − Produkt, Produktgruppe, Produktklasse, Produktfamilie
− ... und Wertebereiche
    − (1, 2, 3, ..., 31), (1, ... 52), (1900, ..., 2017)
    − (...), (Berlin, NRW, Department-1, ...), (D, F, Pl ...)


---

− Typische OLAP Anfrage: Aggregiere Fakten nach Dimensionen.
    − Insbesondere: Entlang von Dimensionshierarchien.

聚合事实数据，按照维度进行分组统计。
常见操作包括：
    Drill-Down（下钻）：从“年”下钻到“月”、“日”；
    Roll-Up（上卷）：从“日”上卷到“月”、“年”；
    Slice / Dice：选取某一维度的特定值或切片分析；
    Pivot（旋转）：交换维度的展示方式。

![](image/Pasted%20image%2020250609144518.png)



# 4 OLAP - Operationen

## 4.1 Slicing und Dicing


- Slicing: Auswahl von bestimmten Einträgen in dem Cube in einer oder mehrerer Dimensionen.
- Dicing: Partitionieren des Basisdaten-Cube in einen gröberen Cube.
- Der durch Dicing erzeugte Cube enthält aggregierte Daten (hier: Summe der Profite) statt der Einzelverkäufe im Basisdaten-Cube


切条 切块

![](image/Pasted%20image%2020250609145349.png)



Hierarchische Dimensionen → unterschiedliche granulare Partitionen
![](image/Pasted%20image%2020250609145447.png)

## 4.2 Drill down und Roll up

 Drill down: Feinere Partitionierung und/oder stärkere Selektion
Roll up: Gröbere Partitionierung → Stärkere Verdichtung

![](image/Pasted%20image%2020250609145544.png)


![](image/Pasted%20image%2020250609145558.png)


### 4.2.1 ROLLUP Operator  and Cube Opeator 

Roll up Operation: hierarchische Aggregation

![](image/Pasted%20image%2020250609145615.png)

![](image/Pasted%20image%2020250609145638.png)

---

ROLLUP Operator
- Herkömmliches SQL
    - Dimension mit k Stufen – Union von k Queries
    - k Scans der Faktentabelle
        - Keine Optimierung wg. fehlender Multiple-Query Optimierung in kommerziellen RDBMS
    - Schlechte Ergebnisreihenfolge
- ROLLUP Operator
    - Hierarchische Aggregation mit Zwischensummen
    - Summen werden durch „ALL“ als Wert repräsentiert


**传统 SQL**：

- 如果你想对某个具有 `k` 个层级的维度（比如时间：年、月、日）做聚合分析，需要写 **`k` 个 UNION 查询**，每个查询对应一个聚合层级。
- 这意味着需要对**事实表扫描 k 次**，效率很低。
- 商业关系数据库管理系统（RDBMS）**通常不支持多查询（Multiple-Query）优化**，因此性能很差。
- 查询结果的行顺序也不好控制（**结果排序差**）。


![](image/Pasted%20image%2020250609145704.png)

**ROLLUP 操作符**：

- 用于在 SQL 中实现**维度层级的聚合（Hierarchische Aggregation）**，包括每一级别的**小计（Zwischensummen）**。
    
- 在结果中，自动**添加总计和小计行**，每一层的聚合用 `ALL` 或 `NULL` 来表示维度的“汇总”状态


|Jahr|Monat|Umsatz|
|---|---|---|
|2024|Jan|100|
|2024|Feb|120|
|2025|Jan|130|
使用 `ROLLUP(Jahr, Monat)` 得到的结果可能是：

|Jahr|Monat|Umsatz|
|---|---|---|
|2024|Jan|100|
|2024|Feb|120|
|2024|NULL|220 ← 小计：2024 年总和|
|2025|Jan|130|
|2025|NULL|130 ← 小计：2025 年总和|
|NULL|NULL|350 ← 总计：所有数据总和|

注意：这里的 `NULL` 就是 `ALL` 的表示方式，意味着“在这个维度上聚合所有”。

---

CUBE Operator
− d Dimensionen, jeweils eine Klassifikationsstufe
    − Jede Dimension kann in Gruppierung enthalten sein oder nicht
    − 2d Grupierungsmöglichkeiten

 基础概念：

    有 d 个维度（Dimensionen），每个维度只有一个分类层级（即没有层级结构）。

        每个维度可以被包含或不被包含在分组中。

        因此，共有 2^d（2 的 d 次方）种分组方式（包括全维度聚合、单维度聚合、无聚合等所有组合）。

---


− Herkömmliches SQL
    − Viel Schreibarbeit
    − 2d Scans der Faktentabelle (wieder keine Optimierung möglich)

在传统 SQL 中：

    编写这些组合的 SQL 查询非常繁琐（Viel Schreibarbeit）。

    为每种分组方式都要扫描一次事实表，造成 2^d 次扫描，性能极差。

    商业数据库系统通常 不支持多查询优化（Multiple-Query Optimization），所以无法提升效率。


---



− CUBE Operator
    − Berechnung der Summen von sämtlichen Kombinationen der Argumente (Klassifikationsstufen)
    − Summen werden durch „ALL“ repräsentiert
    − Keine Beachtung von Hierarchien
        − Durch Schachtelung mit ROLLUP erreichbar

CUBE 操作符的优势：

    CUBE 能一次性计算所有维度组合的聚合值（即所有分类组合的总和）。

    聚合结果中的“汇总”使用 "ALL" 来表示。

    它不考虑维度之间的层级结构（比如时间中的 年 ⊃ 月 ⊃ 日）。



![](image/Pasted%20image%2020250609150457.png)


![](image/Pasted%20image%2020250609150503.png)


![](image/Pasted%20image%2020250609150511.png)


假设你有两个维度：`Land`（国家） 和 `Produkt`（产品）：

```sql
SELECT Land, Produkt, SUM(Umsatz)
FROM Verkäufe
GROUP BY CUBE(Land, Produkt);

```

这会生成以下聚合：

- 每个国家、每个产品的销售额
    
- 每个国家的总销售额（所有产品）
    
- 每个产品的总销售额（所有国家）
    
- 所有国家 + 所有产品的总体销售额
    

结果表中，用 `NULL`（或 `ALL`）来表示聚合维度的“全部”。



---

`CUBE` 和 `ROLLUP` 是 SQL 中用于 **多维数据分析（OLAP）** 的两个聚合操作符，它们都可以在 `GROUP BY` 子句中使用，用于自动生成**分组聚合结果（aggregates）**，但它们的用途和行为有明显的不同。

| 特性                  | `ROLLUP`               | `CUBE`                 |
| ------------------- | ---------------------- | ---------------------- |
| 目的                  | 沿 **维度层级顺序** 逐级聚合      | 生成所有 **维度组合** 的聚合      |
| 聚合组合数量              | `n + 1`（n 个维度 + 总计）    | `2^n`（所有可能的组合）         |
| 是否考虑维度的顺序           | ✅ 是的（顺序重要）             | ❌ 否（顺序无关）              |
| 是否支持维度层级（hierarchy） | ✅ 支持（适用于有层次的维度）        | ❌ 不支持（所有维度平等）          |
| 聚合方式                | 顺序汇总，从明细到总计            | 所有维度的**每一种组合**都汇总      |
| 使用场景                | 财务报表、时间序列分析（年 > 月 > 日） | 多维交叉分析（如 产品 × 地区 × 时间） |


假设我们有一个销售表 `sales`，包含字段：`Region`, `Product`, `SalesAmount`


1 
```
SELECT Region, Product, SUM(SalesAmount)
FROM sales
GROUP BY ROLLUP(Region, Product);

```
结果：

    每个 Region + Product 的销售总额

    每个 Region 的总销售额（所有产品）

    所有地区和所有产品的总销售额（汇总）


```
Region   | Product   | Sum
---------|-----------|-----
East     | Apple     | 100
East     | Banana    | 150
East     | NULL      | 250   ← East 汇总
West     | Apple     | 120
West     | Banana    | 130
West     | NULL      | 250   ← West 汇总
NULL     | NULL      | 500   ← 总汇总

```


2 使用 `CUBE`：

```
SELECT Region, Product, SUM(SalesAmount)
FROM sales
GROUP BY CUBE(Region, Product);

```

结果：

    Region + Product（明细）

    Region 的总和（所有产品）

    Product 的总和（所有 Region）

    所有数据的总和（全部 Region + Product）


```
Region   | Product   | Sum
---------|-----------|-----
East     | Apple     | 100
East     | Banana    | 150
East     | NULL      | 250   ← East 汇总
West     | Apple     | 120
West     | Banana    | 130
West     | NULL      | 250   ← West 汇总
NULL     | Apple     | 220   ← Apple 汇总
NULL     | Banana    | 280   ← Banana 汇总
NULL     | NULL      | 500   ← 总汇总

```



|你想要…|使用哪个？|
|---|---|
|沿某个顺序进行层次聚合|✅ `ROLLUP`|
|所有维度组合都统计|✅ `CUBE`|
|对数据的多角度切片分析（slice）|✅ `CUBE`|
|分析某个维度随时间的变化|✅ `ROLLUP`|
|简化复杂的 `UNION ALL` 语句|两者都可以|


# 5 Relationale Implementierung von DWH


Relationale Implementierung - Schemata
− ER - Schema
    − Relationen für Fakten und für Dimensionen sind normalisiert.
    − Mehrere Relationen für die Fakten.
− Star
    − Relationen für Fakten sind normalisiert, für Dimensionen bewusst denormalisiert.
− Snowflake
    − Relationen für Dimensionen normalisiert.
    − Eine normalisierte Relation für die Fakten.
    − Zeit-Dimension wird normalerweise nicht normalisiert.
− Fullfact
    − Relation für Fakten nicht normalisiert.
    − Relation für Fakten hält Fremdschlüssel für Dimensionen jeder Hierarchiestufe.
    − Relationen für Dimensionen normalisiert.


## 5.1 ER - Schema

![](image/Pasted%20image%2020250609151315.png)

Warum kein ER-Schema
− Normalisierte ER Schemas sind für OLTP Anwendungen optimiert:
    − Ziel: Einfaches Verändern / Hinzufügen / Löschen.
    − Normalisierung verhindert Einfüge / Lösch / Update Anomalien.
    − Transaktionen müssen nur wenige Einträge sperren.
        ➔ Optimiert auf Schreiboperationen.
− In OLAP Anwendungen haben wir andere Anforderungen:
    − Aggregation benötigt de-normalisierte Daten.
        − Normalisiertes Schema ➔ Viele Joins ➔ Teure Operationen!
    − Viele Tabellen machen den ETL Prozess aufwendiger.
    − Idee: Speichere Daten de-normalisiert.
        − Keine Updates in OLAP Anwendungen ➔ Keine Anomalien!
        − Weniger Tabellen erlauben schnellere Anfragen.
        ➔ Optimiert auf Leseoperationen.

## 5.2 Star and Snowflake

![](image/Pasted%20image%2020250609151405.png)


![](image/Pasted%20image%2020250609151415.png)

![](image/Pasted%20image%2020250609151425.png)



![](image/Pasted%20image%2020250609151432.png)

![](image/Pasted%20image%2020250609151440.png)



Star-Schema:
− Vorteile:
    − Flexibles Modell, sehr einfach zu modellieren.
    − OLAP Anfragen können als einfache SQL Anfragen modelliert werden.
− Nachteile:
    − Dimensionshierarchien sind „versteckt“, nicht offensichtlich.
    − Dimensionstabellen sind de-normalisiert: Problematisch wenn sich Dimensionen über die Zeit ändern („Slowly Changing Dimensions“).
− Snowflake-Schema:
    − Vorteile:
        − Wie beim Sternschema, aber robuster gegen sich ändernde Dimensionen.
    − Nachteile:
        − Benötigt aber mehr Joins ➔ Langsamere Anfragen!





## 5.3 Fullfact 

![](image/Pasted%20image%2020250609151542.png)

− Hybrid aus Star- & Snowflake-Schema:
− Normalisierte Dimensionen / Denormalisierte Faktentabelle.
− Faktentabelle enthält PK Relation auf alle hierarchischen Dimensionstabellen

− Vorteile:
− Kombiniert Vorteile von Star- & Snowflake-Schema.
− Außerdem: Teilweise schneller als Schneeschema:
    − Direkter FK Verweis auf alle Hierarchien erlaubt Filtern nach allen Dimensionstiefen ohne Join.
    − Beispiel:
        − Star-Schema: Filtern nach Monat benötigt Join mit Zeit, da monat_id nur in Dimension vorhanden.
        − Fullfact: Direktes Filtern nach Monat ohne Join, da monat_id in der Faktentabelle vorhanden.


Aber: Zusätzliche Attribute blähen Faktenabelle auf
➔ Deutlich gesteigerter Speicherbedarf.
➔ Evtl. langsamere Anfragen, da mehr Daten geladen werden müssen


# 6 Wrap up 

![](image/Pasted%20image%2020250609151653.png)

![](image/Pasted%20image%2020250609151659.png)


![](image/Pasted%20image%2020250609151704.png)